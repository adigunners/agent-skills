# agent-skills

Skills for [Claude Code](https://claude.com/claude-code) — packaged workflows an agent invokes by name. Built for a production consumer product, then generalised and measured before being published here.

---

## [`five-a-side`](five-a-side/) — adversarial code review

Five reviewers that always play, plus a substitute for anything touching a real user. Each asks one question and loads your repo's own rules; one merge decision comes out.

| Role | Question it owns |
| --- | --- |
| `standards` | Does this match how we build here? |
| `spec` | Does it do what was asked — no less, no more? |
| `adversary` | Can it be broken? |
| `operator` | Can I see it fail, and can I undo it? |
| `prover` | Do the tests go red when the code is wrong? |
| `steward` | *Substitute.* Were we allowed to do this to a user? |

**It replaced a two-axis review that kept passing changes which were wrong.** Both original axes were internally referenced — standards against our own docs, spec against our own issue — so a change could satisfy both and still ship a leaked process holding a port, a `set -e` path that skipped rollback, a command-injection vector, and three assertions that passed against deliberately broken code.

### Measured, not asserted

A review system nobody has measured is a ritual. Three trials:

1. **Known-answer recall** — pointed at a commit whose bugs were already documented, in a fixture stripped of git history so it couldn't read the answer. Found **all four**, plus four nobody had found, one still live in production.
2. **Merged, deployed code** — a consent-gating change already reviewed and shipped. **Five confirmed defects**, each verified by a *different model* than the one that found it.
3. **Its own pull request** — **nine blocking findings**, including that the CI gate couldn't block a merge on that GitHub plan. Two of the nine contradicted claims made in the PR description.
4. **Real use — and it failed.** The gate shipped requiring review on every change into a protected branch. Within a day it had blocked a production promotion and a colleague could not merge a single small fix.

The third result says the reviewers work. **The fourth is the one to learn from:** a review system can be correct and still unusable, and its author is the last person able to tell. Every PR I had tested it on was mine, and all of them were the one category that genuinely needs review.

**[Full documentation, design decisions and known limits →](five-a-side/README.md)**

---

## Design principles these were built on

Learned from the trials above, and they generalise past this one skill.

- **Silence fails safe, in the direction that costs least.** A reviewer returning nothing means the run is incomplete — never clean. But a *challenger* returning nothing leaves the finding standing. Deleting a finding requires an affirmative refutation, never the absence of one.
- **Don't economise on the stage that removes work.** A weak finder produces a thin report someone notices. A weak refuter deletes a real finding and nobody learns it existed.
- **Agreement between agents on one model is one opinion wearing several shirts.** Cross-model challenge is what turns it into evidence.
- **Declare what you truncated.** A silent cap reads as "that was everything", which is the one thing it must never mean.
- **Publish the limits.** Every skill here documents where it is blind. A tool that hides that is worse than one that has it.
- **Gate on risk, not on everything — and advertise the override.** Friction has to be proportional to what a mistake costs. A gate a competent engineer cannot get past is a gate that gets deleted, after which nothing is checked at all.
- **Backtest a policy before trusting it.** Replay recent merged work through the rule and count what it would have stopped. Cheaper than waiting for a colleague to be annoyed enough to tell you.

## About this repo

This is a **release snapshot**, not a live mirror. The skills are developed in a private repo against a production codebase and generalised on publish — incident specifics and product decisions are stripped by a script that aborts rather than publish an un-scrubbed line. Expect this copy to lag the source between releases.

## Install

Each skill is a directory. Copy the one you want:

```bash
git clone https://github.com/adigunners/agent-skills.git
cp -R agent-skills/five-a-side <your-repo>/.claude/skills/five-a-side
```

## Licence

MIT — see [LICENSE](LICENSE). The code-smell baseline in `five-a-side/references/standards.md` is adapted from the `two-axis-review` skill by [Saurabh Sarin](https://github.com/sarinsaurabh/claude-skills), also MIT.
