# five-a-side

A six-reviewer adversarial code-review skill for [Claude Code](https://claude.com/claude-code). Five roles that always play, plus a substitute for anything touching a real user — each reading your repo's own rules, aggregated into one merge decision.

It replaced a two-axis review (standards + spec) that kept passing changes which were wrong.

## Why two axes weren't enough

Both original axes were **internally referenced**: Standards checked code against our own docs, Spec against our own issue. A change could satisfy both and still be broken — and one was.

An atomic-deploy rewrite conformed to every documented standard and implemented its issue exactly, while containing a leaked process holding a port, a `set -e` path that skipped rollback entirely, uncollected build directories, a command-injection vector, and three test assertions that passed against deliberately broken code.

None of that was reachable from either axis. What found it was adversarial red-teaming, executing the scripts against a fixture, and mutation-testing the assertions. Those became roles.

## The squad

| Role | Question it owns | May block on |
| --- | --- | --- |
| `standards` | Does this match how we build here? | Documented-standard breaches |
| `spec` | Does it do what was asked — no less, no more? | Missing or wrong requirements |
| `adversary` | Can it be broken? | Anything exploitable |
| `operator` | Can I see it fail, and can I undo it? | Unrecoverable or invisible failure |
| `prover` | Do the tests go red when the code is wrong? | Behaviour change no test catches |
| `steward` | *Substitute.* Were we allowed to do this to a user? | Consent, retention, exclusion, false promises |

`prover` is the only one that runs anything. It mutates the change and requires the suite to go red — reading a test tells you what it claims, only breaking the code tells you what it catches.

## Roles are fixed; domain is a variable

The obvious way to cover a full stack is one reviewer per domain — frontend, backend, database, design system. That turns five into eleven, each idle most of the time.

Instead the roles are fixed by **the question they ask**, and the domain changes **which rules they load**. Every domain has all five questions: a migration can be non-conventional, wrong versus spec, exploitable, unrecoverable, or untested. So can a button.

```
skill (this repo)   = the roles, the process, the fan-out, the aggregation
packs (your repo)   = your actual rules, in .claude/five-a-side/packs/*.md
```

A section's presence in a pack is the trigger. A pack with no `adversary` section benches the security reviewer for that domain — which is how a UI-only diff avoids being reviewed for rollback safety. See [`references/pack-format.md`](references/pack-format.md) and [`examples/packs/`](examples/packs/).

## Evidence

Three trials, because a review system that has never been measured is a ritual.

**1. Known-answer recall.** Pointed at a commit whose bugs were already documented, in a fixture stripped of git history — one `git log` would have shown a commit titled *"close four holes found red-teaming the release script"*. It found **all four**, plus four nobody had found, one of which was still live in production.

**2. Merged, deployed code.** A consent-gating change that had already been reviewed and shipped. **Five confirmed defects**, including analytics contacting a third party before consent, and "turn analytics off" not turning it off. Each verified by a **different model** than the one that found it.

**3. Its own pull request.** The squad reviewed the PR that introduced the squad. **Nine blocking findings**, including that the CI gate could not block a merge on that GitHub plan, and that it watched pull requests in a repo where people push directly to the protected branch. Two of the nine contradicted claims made in the PR description.

That third result is the one worth trusting. A review system that cannot find fault with its own author is measuring agreement, not quality.

## Design decisions that came from the trials

- **Silence fails safe.** A reviewer that returns nothing is `DID NOT REPORT` and the run is `INCOMPLETE` — never `CLEAR`. In the refute stage it is the opposite direction: a challenger returning no verdict leaves the finding **blocking**. Deleting a finding requires an affirmative refutation, never the absence of one.
- **Never economise on the refute pass.** It is the only stage that removes findings, and the failure modes are asymmetric: a weak finder produces a thin report someone notices; a weak refuter deletes a real finding and nobody learns it existed.
- **Challenge cross-model.** Reviewers sharing a model share its blind spots, so agreement between them can be one opinion wearing several shirts. In trial 2 four reviewers agreed — a different model verifying against sources they never opened is what turned that into evidence.
- **Uncertainty defaults differ by role.** `adversary` and `steward` stand when unsure; everyone else refutes. Those two produce the findings hardest to be certain about, so a refute-on-doubt default discards exactly the most valuable ones.
- **Declare truncation.** A silent finding cap reads as "that was everything", which is the one thing it must never mean.

## Known limits

Stated here because a tool that hides these is worse than one that has them.

- **Same model on both sides.** Builder and reviewers are separate agents but one model with one set of blind spots. Five roles are five prompts over one prior, not five independent judgements. The cross-model challenge narrows this at one point; it does not remove it.
- **Diff-scoped.** Nobody sees that this is the fourth implementation of the same thing, or that a route is now unreachable.
- **Packs are the ceiling.** Every reviewer is exactly as good as the rules it was handed. An unwritten rule is an unreviewable one.
- **A CI gate is only preventive if required status checks are configured** — which needs a paid GitHub plan. Otherwise a red check records that review was skipped and cannot stop the merge. Check with `gh api repos/{owner}/{repo}/branches/{branch}/protection` before claiming a gate blocks anything. This was learned by claiming it and being wrong.

## Install

```bash
git clone https://github.com/adigunners/five-a-side.git
cp -R five-a-side <your-repo>/.claude/skills/five-a-side
mkdir -p <your-repo>/.claude/five-a-side/packs
```

Then write packs for your domains — see [`references/pack-format.md`](references/pack-format.md). The skill needs no configuration; the packs are the whole adaptation surface.

Invoke with `/five-a-side <fixed-point>`, or call it as the review stage of another skill.

## Licence

MIT. The code-smell baseline in [`references/standards.md`](references/standards.md) is adapted from the `two-axis-review` skill by [Saurabh Sarin](https://github.com/sarinsaurabh/claude-skills), also MIT. See [LICENSE](LICENSE).
