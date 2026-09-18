---
name: chaos-monkey
description: >-
  Drive a chaos-monkey subagent that runs small experiments to disprove a
  ticket's steady-state invariants after green tests. Use at the end of every
  implementation before claiming done, and whenever the user says chaos monkey,
  another review round, another subagent loop, or iterate till no more
  findings. Do not use for style, spec completeness, performance, or SOLID.
  Do not skip this pass because the test suite was green.
license: MIT
compatibility: Needs a test runner for the changed code and either a subagent API or a second isolated pass. Does not need cloud mutation, Docker push, or provider OAuth.
metadata:
  author: alex-vasinca
  version: "0.1.0"
  tags: chaos-engineering,code-review,invariants,regression-tests
---

# Chaos monkey

Green tests mean the cases you wrote passed. They do not mean the invariant holds. This pass has caught sibling prefix matches, item-id-as-drive, cache poisoning of truncated walks, capability aliases matching inside longer tokens, and unsuffixed `in the last quarter` becoming NotFound — all after the ticket suite was already green.

This is **chaos engineering applied to a diff**, not a second code review and not instance-killing. [Principles of Chaos](https://principlesofchaos.org/) are experiments: name the steady state, hypothesize it holds under a real-world variable, try to **disprove** that hypothesis. The harder it is to disrupt the steady state, the more confidence you have. A finding is a target for a regression test, not a style note.

## When the parent runs this

After implementation **and** the ticket's proving tests plus the project's lint and typecheck (and UI check/build if the ticket touched UI). Those tests are the **baseline probe**. If the happy-path acceptance cases are not green, abort: you cannot learn from an unknown state.

Not mid-coding. Not instead of a standards-vs-spec review. Run it **even if** that review already ran. Keep the axes separate.

If the user says "another subagent loop" or "iterate till no more findings", this is the loop they mean. Minimum two chaos passes whenever the first pass found anything you then changed.

## What it is not

- **Standards** (layering, types, Fowler smells)
- **Spec** (missing requirement, scope creep)
- **Performance** (O(n²), prompt growth, N+1)
- **SOLID/DRY** (second parser, discarded merge)
- **Random vandalism.** Terminating instances was a primitive. This pass is planned experiments with a blast radius.

A chaos finding can *look* like another axis. Keep it if a concrete input produces the wrong **output**. Hand style-only notes back.

## Experiments, not hunts-without-a-hypothesis

Focus on **measurable output** (what the caller, stream, or user sees), not internal attributes (whether an if-chain "looks ordered"). Chaos verifies that the system *does* work, not how the code is arranged.

Before the pass, write:

1. **Steady state** — the ticket's acceptance output in one line (the control).
2. **Hypothesis** — `If we inject [variable], [steady-state output] still holds.`
3. **Variable** — a real-world input, not an invented API. Prioritize by **impact or frequency**.
4. **Disproof** — the exact wrong output that would falsify the hypothesis.

A hunt that cannot name control output, variable, and wrong experimental output is not an experiment. Skip it.

**Control vs experiment.** The control is the acceptance case already in tests. The experiment is the same path with one concentrated variable. Diffuse "maybe this is fragile" notes do not trip the threshold; make the variable 100% of that call.

**Prefer running the code.** Execute the pure function or a focused test with literal inputs. If you cannot run it cheaply, say the finding is unproven.

**Stop conditions (blast radius).** Drop a hunt that needs live cloud mutation, Terraform apply, Docker push, or real OAuth unless the user asked. Do not expand the ticket. Do not add a second parser "while we're here." If a claimed hang cannot be shown on current code, stop that experiment.

**Automate the ones that fail.** A disproved hypothesis becomes a regression test at the ticket seam. That is the continuous experiment. A finding without a test will regress.

## Derive experiments from this ticket

Build a numbered list from:

1. Steady-state acceptance examples.
2. The deleted or replaced path (did the dead short-circuit survive?).
3. Adjacent identity the ticket said not to confuse.
4. The generators below, instantiated with **this** ticket's names, each written as a hypothesis.

Read [references/variables.md](references/variables.md) when choosing what to inject.

Skip a generator that cannot apply to this diff. Do not hunt deferred sibling work.

## Drive the experiment pass

Run **one** isolated pass. If the host can spawn a subagent, spawn one general-purpose subagent (use the model the user named, otherwise inherit). If it cannot, run the same prompt contract in this turn without mixing it into a standards or spec review. Parallel chaos agents on the same tree contaminate each other's experiments.

The prompt must include:

- Repo path, branch, that the work is **uncommitted** (or name the commits if it is not).
- Files to read (owning module, wiring, tests). Do not ask it to "explore the repo".
- **Steady state** in one line.
- Numbered experiments: each is `If we inject X, Y still holds` plus the wrong output that would disprove it. Prefer executing the function.
- **Already fixed** this session: do not re-report unless still broken.
- **Explicitly rejected** last round: do not reopen unless it can quote a **new** failing scenario against current code and the ticket.
- Report contract (below).

### Subagent report contract

```
Each finding is a disproved hypothesis: variable, expected steady state,
observed output, file:line, one-line fix.
If none: "no remaining chaos defects" plus the experiments you ran
(control still held).
Under 250 words. Quote code. Do not grade style, spec gaps, or performance.
Unproven (could not run the function) stays unproven — do not report it as a defect.
```

### Prompt skeleton

```
You are a chaos-monkey experimenter on UNCOMMITTED <ticket> work.
Repo: <path>  Branch: <branch>

Steady state (control): <one-line acceptance output>

Try to disprove these hypotheses. Prefer executing the function with
literal inputs. Each experiment names the variable and the wrong output:
1. If we inject …, … still holds. Disproof: …
2. …

Already fixed (do not re-report unless still broken):
- ...

Explicitly REJECTED last round (do not reopen unless you have a new
failing scenario against the ticket):
- ...

Read: <paths>

Report only disproved hypotheses with file:line and a one-line fix.
If none: "no remaining chaos defects" plus the experiments you ran.
Unproven stays unproven. Under 250 words.
```

Later passes **must** carry the fix list and the reject list. A second pass without them reopens settled items and wastes the round.

## Parent judgment

The subagent is advisory. The parent decides.

**Fix** when the experiment disproved the hypothesis on current code and a test at the ticket seam can lock the control. Add that test in the same change (automate the experiment).

**Reject** when:

- The ticket's wiring explicitly chose that output.
- It belongs to deferred sibling work.
- The claimed hang/bug does not happen on current code (prove it).
- The hunt is a second matching policy the ticket forbade.
- It is a latent bug in a function this ticket did not own **and** did not touch. If this ticket now owns the function, fix the latent bug.
- The finding is unproven (internals read, function not run) and you cannot cheaply run it.

Write the reject reason next to the finding. The next chaos prompt gets that list.

Do not add speculative helpers, extra regexes, or "while we're here" identity parsers. One owner per policy already exists; chaos monkey does not grow a second.

## Loop

1. Baseline probe: ticket tests green, or abort.
2. First experiment pass.
3. Fix or reject. Lock fixes with tests.
4. Re-run **only** the tests that lock the fixes.
5. Next pass with updated fixed/rejected lists, new variables (not closed ones).
6. Stop when a pass reports no remaining chaos defects, or only items already rejected.

Do not declare the implementation done after the first green test run. Do not stop after the first chaos pass if you changed code.

## Output to the user

Keep standards and spec out of this report. For chaos:

- **Steady state** — the one-line control you used.
- **Fixed** — disproved hypothesis, and the test that now runs continuously.
- **Rejected** — hypothesis, and why it was not a defect.
- **Unproven / still true** — live gaps, deferred work, things tests cannot prove.

Do not commit, push, or move the issue tracker unless the user asked.
