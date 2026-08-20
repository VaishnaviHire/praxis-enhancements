---
issue: https://github.com/praxis-proxy/praxis/issues/99
discussion: https://github.com/praxis-proxy/praxis/issues/99#issuecomment-4411378263
status: proposed
repos:
  - praxis
authors:
  - nerdalert
  - shaneutt
graduation_criteria:
  - State class taxonomy agreed by stakeholders
  - How? section with requirements and design
  - Storage trait API design reviewed
  - State type hierarchy agreed by stakeholders
  - Scoping model agreed
  - Determine whether Ephemeral and Persistent storage really need to be separate and resolve
stakeholders:
  - shaneutt
  - nerdalert
  - twghu
  - rikatz
  - leseb
related:
  - 00412
origin:
  repo: praxis
  issue: https://github.com/praxis-proxy/praxis/issues/99
  file: 00099_stateful-proxy-state-management.md
---

# Stateful Proxy State Management

## What?

Praxis should define a project-wide state model before
stateful AI gateway, routing, quota, protocol, and
observability features each grow their own storage
patterns. The model should distinguish request-local
facts, local runtime state, shared hot-path state,
durable business state, configuration state, and
externalized decision state. It must provide optionality
for storage (e.g. local in-memory or disk, vs external kv-store, etc).

The spike for issue 99 found that Praxis already has
several local state patterns: request metadata, local
load-balancer counters, circuit breaker state, health
snapshots, hot-reloaded config snapshots, and filter-owned
maps. The per-IP rate limiter is the clearest existing
example: it uses a local `DashMap` with a `100_000` entry
soft cap and `200_000` entry hard cap to prevent unbounded
growth. These patterns are useful, but they are not enough
for features that must remain correct across multiple
Praxis replicas.

This proposal establishes the direction that stateful
features must use explicit storage classes and typed
domain APIs instead of exposing a raw global key-value API
as the primary filter-facing abstraction. Local in-memory
implementations should be available for tests, development,
demos, explicitly single-replica deployments, and some niche
HA scenarios. The default production guidance will be that
implementations must use an external (e.g. Valkey) storage
backend first, with strict timeouts, TTLs, key conventions,
failure-mode defaults, and bounded metrics labels.

### Goals

- Define state classes used consistently across Praxis
  features and docs.
- Keep request-derived facts in request context and
  `filter_metadata`, not in durable or shared stores.
- Make local runtime state explicitly bounded, observable,
  and documented as local-only unless proven otherwise.
- Add typed state APIs for concrete domains such as rate
  limits, token ledgers, protocol sessions, task ownership,
  policy decision caches, routing snapshots, cache indexes,
  and usage event export.
- Centralize shared backend configuration so filters do
  not create independent Redis clients, key formats,
  timeouts, or failure behavior.
- Require every shared hot-path state operation to define
  timeout behavior, fail-open or fail-closed semantics,
  key schema, TTL, cardinality bounds, and metrics.
- Keep durable billing, audit, certificate, object,
  vector, and control-plane state out of synchronous
  request processing unless a feature explicitly justifies
  that cost.
- Provide a complete experience for the storage and retrieval of metrics per-site, with
  optional and customizable storage solutions. This covers built-in metrics (core proxy,
  built-in filters, etc) but also extensions (custom filters).
- Provide an incremental implementation path that starts
  with documentation and bounded local primitives before
  adding shared backends.
- Enforce that no built-in filters outright fail without internal storage. Filters
  must tolerate lack of storage whenever possible. An explicit opt-out will be
  needed for any filter that is purely bound on external storage solutions.

### State Classes

| State class | Examples | First posture |
| --- | --- | --- |
| Request-local metadata | extracted model, JSON-RPC method, auth subject, tenant, selected route, token estimate, guardrail finding | Keep in request context and `filter_metadata`; use for later filters, logs, metrics, and headers. |
| Connection-local state | client address, TLS facts, client certificate fields, CONNECT tunnel state | Keep in connection/request context; promote only when a feature needs cross-request behavior. |
| Local runtime state | local rate buckets, circuit breakers, endpoint health, local caches, overload level | Bound with max entries, TTLs, eviction policy, reload behavior, and metrics. |
| Shared hot-path state | production quotas, token budgets, session ownership, task ownership, short-lived policy cache | Use Redis/Valkey or an external service with tight timeouts and explicit failure semantics. |
| Durable business state | billing records, usage history, subscriptions, audit logs | Export asynchronously to the owning system; do not make database writes part of normal request admission. |
| Configuration state | routes, clusters, model aliases, policies, endpoint resources, config generation | Treat as validated snapshots from files, xDS, Kubernetes, Gateway API, or another control plane. |
| Externalized decisions | authz, external processing, schedulers, guardrails, model routing providers | Use explicit timeout, retry, circuit breaker, and failure-mode rules at each call site. |

### Feature Drivers

| Area | State pressure | Direction |
| --- | --- | --- |
| Request rate limiting | Counters must remember previous requests and may need fleet-wide enforcement. | Local mode for dev/single replica; shared store for production multi-replica limits. |
| Token quotas and MaaS budgets | Budgets are ledgers over time and must be consistent across replicas. | Shared token ledger for enforcement plus async durable usage export. |
| MCP, A2A, and sticky sessions | Follow-up requests may need the backend that owns a session, task, or context. | Typed session and task stores with TTLs and config-generation awareness. |
| Intelligent routing and llm-d/GIE | Routing depends on fresh backend pressure, readiness, cost, and scheduler signals. | Local validated snapshots with freshness limits and safe fallback behavior. |
| Response and semantic caching | Cache entries, fill locks, vector refs, and purge state have cache-specific semantics. | Separate cache APIs; do not overload quota/session stores as a generic cache. |
| Auth, RBAC, guardrails, and policy | Security decisions may be expensive, cached, audited, or externally delegated. | Request-local decision facts, bounded caches, policy-version keys, and fail-closed defaults for enforcement. |
| Retry, hedging, and failover | Attempts and budgets must avoid loops, amplification, and confusing usage records. | Keep attempt state request-local first; add shared fleet budgets only when needed. |
| Observability | Metrics, logs, traces, and exporters aggregate state and can create cardinality risk. | Keep exporter state bounded and never label on raw request, user, session, task, or prompt values. |

### Non-Goals

- Do not make Praxis a database.
- Do not require every stateful feature to use Valkey.
- Do not expose a raw global mutable key-value API as the
  primary filter-facing abstraction.
- Do not put SQL, Kubernetes API, object-store, vector
  database, or other slow control-plane calls directly on
  every request path.
- Do not treat local maps as multi-replica correct.
- Do not store prompts, API keys, tokens, tool arguments,
  or PII in shared-state keys.
- Do not block MCP, A2A, MaaS, routing scorer, or cache
  work on implementing every state class at once.

## Why?

### Motivation

Praxis is an AI-native proxy framework, not just
a stateless reverse proxy. Planned features need decisions
based on facts produced before, during, and after a
request: parsed request bodies, streaming response usage,
tenant identity, token budgets, backend pressure, protocol
session IDs, task IDs, guardrail findings, retry attempts,
and dynamic configuration versions.

Without a shared state model, each feature will be tempted
to solve this locally. That creates predictable production
risks: unbounded memory maps, duplicated backend clients,
incompatible Redis key formats, hidden fail-open security
paths, hot-reload state loss, high-cardinality metrics, and
test coverage that passes in a single process but fails
across multiple replicas.

Hot reload is a concrete example of the problem. Today,
pipeline reload rebuilds stateful filter instances, so
local rate limiter and circuit breaker state resets while
the process continues serving traffic. That behavior is
acceptable for local protection, but not for
correctness-critical quota, session, or task state.

The spike also found that state should not be collapsed
into one storage layer. Request facts should stay local to
the request. Local runtime state should be fast,
bounded, and disposable. Shared hot-path state should be
reserved for correctness-sensitive counters, ledgers,
sessions, and ownership maps. Durable business records
should flow through async sinks or product systems.
Configuration should remain a validated snapshot from the
config or control plane.

Valkey is a good first shared hot-path backend
because it fits short-lived counters, TTL-backed session
maps, task ownership, policy decision caches, and
correlation maps. It is not the answer for long-term
billing history, large response bodies, vector search,
certificate management, or desired configuration state.

### User Stories

- As a proxy operator, I want local and shared state modes
  to be explicit so that I do not accidentally deploy
  single-replica counters as global production quotas.
- As an SRE, I want all hot-path state calls to have
  bounded timeouts and visible metrics so that a backend
  outage does not create unbounded request latency.
- As a security engineer, I want auth, policy, quota, and
  guardrail state failures to fail closed by default so
  that backend errors do not bypass enforcement.
- As a platform engineer, I want one Redis/Valkey backend
  configuration with shared connection, TLS, auth, timeout,
  and metric behavior so that every filter does not create
  a different operational surface.
- As a Praxis developer, I want typed state traits for
  rate limits, token ledgers, sessions, task ownership, and
  usage events so that filters do not hand-roll storage and
  key semantics.
- As an AI gateway operator, I want token counts and usage
  facts to flow from request and response processing into
  quota enforcement and billing export without turning
  request metadata into the ledger of record.
- As an SRE, I want storage-related degradation to be
  monitored with configurable performance guards so that
  Praxis can detect trends, alert on threshold breaches,
  and support graceful degradation with recovery rather
  than silent latency creep or hard failures.

> **Note:** Detailed requirements and design should be
> added in a follow-up proposal update after the state
> model and motivation are accepted.

## Unified State Interface

This section incorporates the concrete interface design
from proposal #412 (Unified State Interface), which
complements the state model above with traits, configuration
schema, scoping rules, and backend implementations.

### What?

Create a standard interface and machinery for State
in Praxis core. State is the single top-level
abstraction. It has two categories:

- **Ephemeral state** covers backends like Valkey
  and in-memory stores. Data may be lost on restart.
  Suitable for caches, counters, and session
  affinity across requests and replicas.

- **Storage state** covers persistent backends like
  PostgreSQL, SQLite, and file stores. Data survives
  restarts. Suitable for conversation history,
  response records, and durable business data.

Filters and other consumers (such as probes)
declare their state needs in configuration. Praxis
manages backend lifecycle, configuration, and
scoped access. State backends are scoped to a
specific filter, a named chain, or made globally
available with explicit opt-in.

#### Goals

- A single interface that covers both ephemeral
  and persistent state under one abstraction.
- Config-driven backend lifecycle: operators
  declare backends in YAML, Praxis provisions
  them at startup.
- Scoped access by default: backends are bound to
  a filter or chain. Global access requires
  explicit configuration.
- Pluggable backends: core defines traits and ships
  common defaults (in-memory, Valkey, SQLite,
  OpenAI Files API). External crates provide
  additional
  backends (e.g. PostgreSQL) without modifying
  core.
- Accessible to filters and to probes (background
  processes), decoupled from the HTTP request
  lifecycle.

### Why?

#### Ephemeral State

Praxis needs ephemeral state to function as a
stateful proxy.

**Multi-instance deployments.** Distributed
ephemeral state (Valkey) enables session affinity,
rate limiting, and feature flags that are consistent
across proxy replicas. Without a standard backend,
each feature that needs distributed ephemeral state
would build its own Redis client, key format,
timeout policy, and failure semantics.

**Stateful protocols.** MCP uses session IDs that
bind follow-up requests to the backend that owns
the session context. A2A tracks long-running tasks
across multiple request/response exchanges.
WebSocket and gRPC streaming sessions need
connection-to-session mapping. Without shared
ephemeral state, these protocols cannot be proxied
correctly across multiple requests.

**Cross-request state.** Rate limiters track
request counts across calls. Circuit breakers
accumulate failure counts over time. Health
snapshots inform load balancing decisions. These
patterns need state that persists across requests
and survives configuration reloads, but does not
need to survive a process restart.
Today each builds its own in-process store
(DashMap, atomics) with ad-hoc capacity limits
and no shared lifecycle management.

#### Storage State

Praxis needs persistent storage for the Responses
API, Conversations API, and the agentic loop.

**Responses and Conversations APIs.** The OpenAI
Responses API is stateful by design. Each response
can reference a `previous_response_id` to continue
a conversation. The Conversations API maintains
accumulated message history across turns. Both
must persist records so that subsequent requests
can rehydrate context from earlier turns. This
already works today via `ResponseStore` and
`ConversationItemStore` with SQLite and PostgreSQL
backends, but the implementations are
self-contained in the ai/ repository with their
own traits, registries, and configuration. Nothing
else can reuse them.

**Agentic loop orchestration.** The agentic loop
executes multiple inference rounds, accumulating
tool call results, conversation messages, and token
usage across iterations. When `store: true` is set,
the full conversation state must be persisted
durably so that clients can retrieve or continue it
later. Conversation compare-and-swap (already
implemented in the ai/ repo) shows the need for
concurrent-safe persistent state operations.

**Near-term consumers.** MCP session persistence
requires durable session-to-backend mappings that
survive proxy restarts and are shareable across
replicas. A2A task state tracks long-running task
lifecycle (submitted, working, completed, failed)
across multiple request/response exchanges. Both
are protocol-level concerns, not OpenAI-specific.
The Files API needs local file storage for
development environments (praxis-proxy/ai#494).
Each of these would otherwise build its own
trait, registry, and configuration - the same
scaffolding that `ResponseStore` and
`ConversationItemStore` already duplicated.

**Shared need.** Both ephemeral and storage state
need the same infrastructure: named backends
declared in configuration, scoped access control,
managed lifecycle, pluggable implementations, and
availability outside the HTTP request path (for
probes and background processes). Building this
once as a unified interface avoids duplicating the
machinery for each new feature.

#### User Stories

- As a proxy operator, I want to declare state
  backends in YAML so that I manage connectivity,
  credentials, and TLS centrally instead of per
  filter.
- As a filter author, I want to request a named
  state backend through the filter context so that
  I do not manage backend lifecycle or connection
  pooling myself.
- As a platform engineer, I want state scoped to
  specific filters by default so that one filter
  cannot corrupt another's state.
- As an AI gateway operator, I want the same
  interface for conversation persistence, file
  caching, and session tracking so that each
  feature does not build its own storage layer.
- As a Praxis developer, I want to add a new
  storage backend in an external crate without
  modifying core.
- As a probe author, I want access to the same
  state backends that filters use so that
  background processes share state without a
  separate configuration path.
