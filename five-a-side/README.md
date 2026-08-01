# five-a-side

A risk-budgeted adversarial code-review skill for [Claude Code](https://claude.com/claude-code).
It asks a fixed set of questions, but only pays for the reviewers and evidence
that the changed paths justify.

## The team

| Role | Question |
| --- | --- |
| `standards` | Does this match the repository's written rules? |
| `spec` | Does it do what was asked—no less, no more? |
| `adversary` | Can it be abused or exploited? |
| `operator` | Can failure be seen and reversed? |
| `prover` | Do tests go red when the behaviour is wrong? |
| `steward` | Were we allowed to do this to a person? |

The roles are fixed; repository-owned packs decide which roles play for a
changed path. A UI-only change can stay standard, while a migration, auth
change, deployment workflow, or user-data path can enter the critical lane.

## What changed from the always-on version

The old shape spent the full squad and a refute pass on every change. That is
expensive and encourages bypasses. The current shape is explicit about cost:

- no matched pack is `EXEMPT`;
- standard review uses at most three mutations and one adjudicator batch;
- critical review uses at most six mutations and two adjudicator batches;
- one automated remediation is allowed, then focused verification or a human;
- a full second review is reserved for a new risk domain, a changed public
  contract/schema/deploy path, or an explicit request.

`review_plan.py` is the single source of truth for pack matching, lane,
reviewers, budgets, and human acknowledgement. `review_state.py` makes the
remediation limit and metrics auditable. `review_gate.py` can require a recorded
decision only for non-exempt paths.

## Arsenal mode

The optional presentation layer brings a little Arsenal matchday energy to the
workflow: the selected reviewers are the **Starting XI**, objective evidence is
a **VAR check**, focused verification is a **halftime reset**, and the final
decision is the **final whistle**. Stable role slugs always remain visible, and
the mode uses no logos, club marks, player likenesses, or private facts.

## Packs

Packs live in the repository being reviewed:

```text
.claude/five-a-side/packs/<domain>.md
```

They contain path globs, a lane, the roles that own the domain's questions,
and optional human-acknowledgement reasons. A missing pack is a signal to fix
the repository policy, not an excuse to run every model on every diff.

See [`references/pack-format.md`](references/pack-format.md) and the generic
[`examples/packs/design-system.md`](examples/packs/design-system.md).

## Known limits

- One model family can still share blind spots; the adjudicator only narrows
  that risk.
- The review is diff-scoped and cannot replace an architectural audit.
- Packs are the ceiling: unwritten rules cannot be cited.
- A CI marker is detective unless the forge's required status checks make it
  preventive.

## Licence

MIT. See [LICENSE](../LICENSE).
