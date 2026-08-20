# Praxis Enhancement Proposals

Centralized enhancement proposals for all [Praxis]
repositories.

## How New Features Happen

```
Discussion -> Proposal -> Experimental -> Standard
```

1. **Discussion first.** Open a [GitHub Discussion]
   (category: "Idea") describing what you want and
   why. Collect feedback, build consensus, get
   maintainer sign-off. Nothing moves forward
   without this step.

2. **Proposal.** Once a maintainer approves the
   direction, file a proposal here (What + Why
   only). Iterate on the How section after the
   direction is accepted.

3. **Experimental.** Prototype in the [experimental
   repo], then ship behind an experimental flag in
   the target repo. Experimental features may
   change or be removed at any time.

4. **Standard.** After a soak period, maintainers
   promote the feature to standard/released.

> **Nothing before standard is guaranteed.**
> A discussion does not guarantee a proposal will
> be accepted. An accepted proposal does not
> guarantee an experimental implementation. An
> experimental feature does not guarantee promotion
> to standard. Features can be changed, reworked,
> or removed at any stage before reaching standard.

See [docs/process.md](docs/process.md) for the full
lifecycle.

## Structure

```
docs/
  process.md              # Full proposal lifecycle
  statuses.md             # Status definitions
  experimental-phase.md   # Experimental repo guide
proposals/
  template.md             # Proposal template
  NNNNN_slug.md           # Individual proposals
```

## Links

- [Proposal Process](docs/process.md)
- [Status Definitions](docs/statuses.md)
- [Experimental Phase Guide](docs/experimental-phase.md)
- [Proposal Template](proposals/template.md)
- [Contributing](CONTRIBUTING.md)

[Praxis]: https://github.com/praxis-proxy
[GitHub Discussion]: https://github.com/orgs/praxis-proxy/discussions
[experimental repo]: https://github.com/praxis-proxy/experimental
