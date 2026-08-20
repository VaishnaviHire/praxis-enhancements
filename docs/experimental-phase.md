# Experimental Phase Guide

The [experimental repo] is a required phase in the
proposal lifecycle. Most new filters and features
must be prototyped there before advancing to
`experimental` status in their target repository.

[experimental repo]: https://github.com/praxis-proxy/experimental

## Why Prototype First?

Prototyping in a separate repo provides:

- **Low-risk iteration.** Experimental code does
  not affect the stability of production repos.
- **Fast feedback.** Authors can iterate on design
  without the overhead of full CI and review in
  the target repo.
- **Proof of feasibility.** A working prototype
  demonstrates the proposal is implementable, not
  just theoretically sound.
- **Informed design.** The How? section is stronger
  when informed by a working prototype.

## Default Path

For proposals adding filters, features, or new
capabilities:

1. Implement a working prototype in the
   experimental repo.
2. The prototype must demonstrate the core
   functionality described in the proposal's
   What? section. It does not need to be
   production-ready.
3. Update the proposal's `experimental_impl`
   frontmatter field with a link to the
   implementation (PR or branch URL).
4. Once the prototype exists and the proposal is
   accepted, code moves to the target repository
   behind an experimental flag.

### What Counts as a Sufficient Prototype?

- Core functionality works end-to-end
- Basic happy path is exercised
- Key design decisions are validated
- It does NOT need: production error handling,
  full test coverage, documentation, or
  performance optimization

## Exempt Path

Some proposals structurally cannot use the
experimental repo. These must set
`experimental_exempt: true` and provide a reason
in `experimental_exempt_reason`.

### Valid Exemption Categories

| Category | Examples |
|----------|---------|
| Operator CRD / control plane changes | New CRD fields, controller logic, Gateway API support |
| Core pipeline architecture changes | Pipeline execution model, filter trait changes |
| Build system / tooling changes | Build-time filter registry, workspace configuration |
| Cross-repo infrastructure | Shared crate changes, workspace dependency updates |
| Configuration schema changes | New config file format, YAML schema additions |
| Convention / process changes | Linting rules, CI workflow changes |

### Alternative for Exempt Proposals

Instead of prototyping in the experimental repo,
exempt proposals must use one of:

- **Feature flag in the target repo.** Gate the new
  functionality behind a compile-time or runtime
  feature flag for a soak period.
- **Experimental CRD version.** For operator
  changes, use a `v1alpha1` API version or
  equivalent.
- **Experimental build profile.** Use a Cargo
  feature or build profile that is not enabled by
  default.

The `experimental_impl` field should point to the
feature-flagged implementation in the target repo
instead of the experimental repo.

## Frontmatter Fields

```yaml
# Default path - link to experimental repo impl
experimental_impl: https://github.com/praxis-proxy/experimental/pull/42

# Exempt path - explain why
experimental_exempt: true
experimental_exempt_reason: "Core pipeline architecture change"
```
