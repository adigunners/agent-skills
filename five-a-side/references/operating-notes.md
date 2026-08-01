# Operating notes

This page is for maintainers of the skill, not for every review run.

## Why the lane exists

An always-on review creates a predictable failure mode: reviewers spend time
re-reading low-risk changes, people wait for an expensive run, and the gate is
eventually bypassed. Risk lanes make the control proportional:

- `exempt`: no model review; keep deterministic CI and ordinary human review;
- `standard`: a small, pack-selected team and one bounded adjudication batch;
- `critical`: the declared risk paths, a larger mutation budget, and a human
  acknowledgement where the pack requires it.

The lane is selected from changed paths and explicit signals by
`scripts/review_plan.py`. Keep the risk set short and evidence-based. If a
path is not genuinely expensive, slow to detect, or hard to reverse, do not add
it merely because it belongs to a familiar category.

## Measure before tuning

Record each run with `review_state.py`: lane, selected roles, duration, model,
mutations, blockers, notes, and refutations. Review a small sample of merged
changes periodically and compare:

1. time spent and tokens consumed;
2. proportion of changes that were exempt, standard, or critical;
3. surviving findings confirmed by a human or a deterministic reproduction;
4. false blocks, retries, and missing reports;
5. remediation cycles per change.

If standard runs rarely produce a confirmed blocker, shrink the pack or move a
path to `exempt`; do not simply lower the evidence bar. If critical runs are
slow, reduce the team from the pack first, then the mutation scope, before
reducing the adjudicator's independence.

## Stop conditions

One automated remediation is allowed. Focused verification may inspect the
delta and the roles that owned the surviving blockers. A second automated fix
cycle stops for a human decision. Repeating the same full review/fix loop hides
whether the issue is code, the pack, or the spec and is the main source of
runaway cost.

## Publishing safely

This public snapshot must remain generic. Before publishing, scan for:

- private repository names, local filesystem paths, issue numbers, URLs, and
  customer or product identifiers;
- copied incident details that identify a team or deployment;
- credentials, tokens, internal hostnames, or private model configuration;
- examples that imply the public repo's author has access to a consumer's data.

Use placeholders such as `src/api/**`, `resource_id`, and `example.invalid`.
The only brand layer in this repository is the optional Arsenal-inspired
football vocabulary; it must not include club assets, player likenesses, or
private facts.
