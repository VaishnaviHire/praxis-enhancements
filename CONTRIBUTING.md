# Contributing Proposals

Thank you for your interest in improving Praxis.

## Before You Start

Enhancement proposals are for features that span
multiple PRs, introduce new architectural patterns,
affect a project's public interface, or are complex
enough to warrant more process.

Small changes (bug fixes, minor enhancements,
documentation updates) do not require proposals.

## How New Features Happen

```
Discussion -> Proposal -> Experimental -> Standard
```

**Start with a Discussion.** This is the only way
to get a new feature started. Open a [GitHub
Discussion] (category: "Idea") describing your idea.
Focus on *what* and *why*. Build consensus.

> **Nothing before standard is guaranteed.** A
> discussion does not mean a proposal will be
> accepted. An accepted proposal does not mean it
> will ship. An experimental feature does not mean
> it will reach standard. Features can be changed,
> reworked, or removed at any stage.

[GitHub Discussion]: https://github.com/orgs/praxis-proxy/discussions

## Step by Step

1. **Open a Discussion** at the [org-level
   discussions] (category: "Idea"). Focus on *what*
   and *why*, not implementation details. Build
   consensus.

2. **Get sign-off.** A maintainer reviews the
   discussion and marks it approved.

3. **Create an EPIC issue.** Open an issue in this
   repo from the approved discussion. Include a
   link to the discussion and a high-level summary.

4. **Submit a proposal PR.** Create a file in
   `proposals/` using the [template]. The filename
   must follow the convention:

   ```
   <5-digit-issue-number>_<kebab-case-slug>.md
   ```

   The first PR must contain only the **What?** and
   **Why?** sections. Do not include **How?** yet.

5. **Iterate.** Add the **How?** section in follow-up
   PRs once the direction is accepted.

6. **Prototype.** Implement a working prototype in
   the [experimental repo] before the proposal can
   advance to experimental status. See
   [experimental-phase.md] for details and
   exemptions.

## Requirements

All proposals must include:

- A `discussion` link (PRs without one are
  auto-closed)
- An `issue` link to the EPIC issue
- At least one author
- At least one stakeholder
- A `repos` list of affected repositories
- At least one graduation criterion

See the [template] for the full frontmatter schema.

## Affected Repositories

Valid `repos` values:

`praxis`, `ai`, `operator`, `extproc`,
`conventions`, `experiments`, `grid`, `forge`,
`policy`, `demos`, `pingora`

[org-level discussions]: https://github.com/orgs/praxis-proxy/discussions
[template]: proposals/template.md
[experimental repo]: https://github.com/praxis-proxy/experimental
[experimental-phase.md]: docs/experimental-phase.md
