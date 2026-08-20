# Proposal Statuses

## Status Values

| Status | Meaning |
|--------|---------|
| `proposed` | Under discussion, not yet accepted |
| `blocked` | Cannot proceed; requires prerequisite work |
| `deferred` | Intentionally postponed; not actively pursued |
| `accepted` | Approved for implementation |
| `experimental` | Implemented, shipping as experimental |
| `released` | Stable, fully shipped |
| `withdrawn` | Not proceeding |
| `retroactive` | Documents a feature that shipped before the proposal process existed |

## Status Notes

- A `blocked` proposal must describe what must
  happen for it to become unblocked.

- A `deferred` proposal is not withdrawn but is
  not actively being pursued. It may include a
  `superseded_by` field if another proposal
  replaces it.

- A `withdrawn` proposal must include a clear,
  detailed explanation of why it was withdrawn.

- A `retroactive` proposal documents existing
  functionality. It is a terminal status.

## Transition Rules

```
proposed --> accepted
proposed --> blocked
proposed --> deferred
proposed --> withdrawn

blocked --> proposed
blocked --> accepted
blocked --> withdrawn

deferred --> proposed
deferred --> withdrawn

accepted --> experimental
accepted --> withdrawn

experimental --> released
experimental --> withdrawn

released --> (terminal)
withdrawn --> (terminal)
retroactive --> (terminal)
```

CI enforces these transitions. A PR that attempts
a transition not listed above will fail validation
unless a maintainer applies the
`skip/status-transition` label.

## Supersession

When one proposal replaces another, the replaced
proposal should be set to `deferred` or `withdrawn`
with a `superseded_by` field pointing to the
replacement. The replacement proposal should have a
`supersedes` field pointing back.

Example:

```yaml
# Old proposal
status: deferred
superseded_by: 00042

# New proposal
status: proposed
supersedes: 00017
```
