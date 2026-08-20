# Proposal Process

Enhancement proposals for new capabilities and
refinements across [Praxis] repositories.

Small changes (bug fixes, minor enhancements,
documentation updates) do not require proposals.
Proposals are for features that span multiple PRs,
introduce new architectural patterns, affect a
project's public interface, or are complex enough
to warrant more process.

## How New Features Happen

```
Discussion -> Proposal -> Experimental -> Standard
```

**Discussion first.** This is the only entry point.
Open a [GitHub Discussion] (category: "Idea")
describing what you want and why. Collect feedback,
build consensus, get maintainer sign-off.

**Nothing moves forward without a Discussion.**
PRs that add proposals without a valid discussion
link are automatically closed.

> **Nothing before standard is guaranteed.** A
> discussion does not guarantee a proposal will be
> accepted. An accepted proposal does not guarantee
> an experimental implementation. An experimental
> feature does not guarantee promotion to standard.
> Features can be changed, reworked, or removed at
> any stage before reaching standard.

## Lifecycle

### 1. Discussion

Open a [GitHub Discussion] (category: "Idea")
describing the change at a high level. Focus on
*what* and *why*, not implementation details.

Build consensus with community members.

> **Note**: Some implementation details at this
> stage can be OK, depending on the situation. The
> point of the discussion phase is to get consensus
> that what you're bringing up is a real concern
> that needs to be addressed, regardless of how it
> is addressed.

**This step is mandatory.** Proposals without a
valid discussion link will have their PRs
automatically closed.

[GitHub Discussion]: https://github.com/orgs/praxis-proxy/discussions

### 2. Sign-off

A maintainer reviews the discussion and marks it
as approved. This confirms the project is open to
the proposed direction.

> **Note**: It's fair to directly ping maintainers
> asking for review and approval consideration when
> things get stuck.

### 3. Issue

Once the discussion is approved by a maintainer
and resolved, create an `EPIC` issue in this repo.
Include first a link to the originating discussion,
followed by a high-level summary. This is where
all implementation work will be organized (as
sub-tasks).

> **Note**: Maintainers will assign epic and
> sub-task owners.

### 4. Proposal PR

Create a proposal file in `proposals/` and submit
it as a PR. File naming convention:

```console
<5-digit-issue-number>_<kebab-case-slug>.md
```

The first PR must contain only the **What?** and
**Why?** sections. The **How?** section must be
added after the goals and motivation are accepted.
See the [template] for the full structure.

> **v0.x.x simplification**: During pre-1.0
> development, the **How?** section does not
> require an upfront design document. Once the
> **What?** and **Why?** are agreed on, the
> **How?** can simply list the PRs that implement
> the solution. A full requirements and design
> writeup is welcome but not required until 1.0.

CI will auto-close PRs that:

- Are missing a `discussion` link
- Are missing an `issue` link
- Have no `authors` listed
- Have no `stakeholders` listed
- Have no `repos` listed
- Include the `How?` section in a new proposal

[template]: ../proposals/template.md

### 5. Iteration

Iterate on the proposal in subsequent PRs. Add
the **How?** section: either a list of implementing
PRs or a full requirements and design writeup.
Refine until a maintainer marks the proposal as
accepted.

### 6. Prototype

Before implementation begins in the target
repository, most proposals must be prototyped in
the [experimental repo].

**Default path** (filters, features, new
capabilities): implement a working prototype in the
experimental repo. Update the proposal's
`experimental_impl` frontmatter field with a link to
the implementation. Proposals cannot advance to
`experimental` status without a prototype or an
exemption.

**Exempt path** (core infrastructure, operator
changes, cross-cutting concerns): set
`experimental_exempt: true` and provide a reason in
`experimental_exempt_reason`. Use a feature flag or
experimental build flag in the target repo instead.

See [experimental-phase.md] for full details,
exemption categories, and what constitutes a
sufficient prototype.

[experimental repo]: https://github.com/praxis-proxy/experimental
[experimental-phase.md]: experimental-phase.md

### 7. Experimental

Once prototyped and accepted, someone (perhaps the
authors of the proposal) will be tasked with
implementing the feature in the target repository
and shipping it as experimental. Code moves from
the experimental repo to the real repo behind an
experimental flag. Experimental features are
functional but may change based on user feedback,
and nothing about them is guaranteed.

> **Note**: Updates to experimental features may
> make breaking, backwards-incompatible changes. An
> experimental feature may be removed at any time.

### 8. Standard (Release)

After a soak period determined by maintainers, a
maintainer may promote the feature from experimental
to standard/released. The proposal status is updated
to `released`.

## Affected Repositories

Proposals must declare which repositories they
affect using the `repos` frontmatter field. Valid
values:

| Value | Repository |
|-------|-----------|
| `praxis` | Core proxy server and framework |
| `ai` | AI inference proxy and filters |
| `operator` | Kubernetes Gateway API operator |
| `extproc` | Envoy ExtProc gRPC server |
| `conventions` | Shared lint/config template |
| `experiments` | Experimental filters and features |
| `grid` | AI Grid control plane |
| `forge` | Declarative dev environments |
| `policy` | OPA/Rego policy engine |
| `demos` | Demo configurations |
| `pingora` | Vendored Pingora fork |

## Stakeholders

Every proposal must list its stakeholders in the
frontmatter. Stakeholders are people with a vested
interest in the outcome of a proposal: maintainers,
domain experts, downstream consumers, or anyone
whose work is directly affected by the change.
Stakeholders are expected to review and provide
feedback throughout the proposal lifecycle.

Authors are the people writing and driving the
proposal. Stakeholders are the people who need to
be kept informed and whose input is essential for
the proposal to succeed. An author may also be a
stakeholder.

## Graduation Criteria

Every proposal must list graduation criteria in
the frontmatter. These are the conditions that must
be satisfied before a maintainer will advance the
proposal's status (e.g. `proposed` to `accepted`,
`experimental` to `released`).

Graduation criteria serve as a TODO list for the
proposal. They capture important open items that
must be resolved before the proposal can graduate,
without necessarily blocking the current PR. If a
concern is real but can be addressed in a follow-up
iteration, add it as a graduation criterion and
merge the PR. The criterion holds up the status
change, not the pull request.

Good graduation criteria are specific and
verifiable:

- "How? section with requirements and design"
- "Benchmark results for candidate implementations"
- "Storage trait API reviewed by stakeholders"
- "Prototype implemented in experimental repo"

Avoid vague criteria like "general agreement" or
"feels ready."

Released proposals should have an empty
`graduation_criteria` list, since all criteria were
met when the status advanced to `released`.

## Status Values

See [statuses.md] for the full list of status
values, their definitions, and allowed transitions.

[statuses.md]: statuses.md

[Praxis]: https://github.com/praxis-proxy
