---
name: chaos-monkey
description: >-
  Run a post-green-test, hypothesis-driven robustness pass over changed
  behavior. Use when the user asks for a chaos-monkey pass, adversarial input
  experiments, another isolated runtime-counterexample review, or iteration
  until no new input-driven defects remain. Do not use for ordinary
  implementation completion, production infrastructure chaos, failing-test
  debugging, style, architecture, spec completeness, performance tuning, or
  SOLID review.
license: MIT; see LICENSE
compatibility: Requires access to the changed repository and a runnable focused test command. Subagent support is optional.
metadata:
  author: "alex-vasinca"
  version: "0.2.0"
  avaxml.tags: "chaos-engineering,code-review,invariants,regression-tests"
---

# Chaos monkey

Green tests show that the written examples passed; they do not establish the
changed behavior for other plausible inputs.

This is a **chaos-inspired adversarial experiment protocol for changed
behavior**. It borrows from [Principles of Chaos](https://principlesofchaos.org/)
a control invariant, falsifiable hypotheses, realistic variables, observed
disproof, automation, and bounded side effects. It is not production chaos
engineering or a general code-quality review. A finding is a target for a
regression test, not a style note.

## When the parent runs this

Run after the focused acceptance tests for the changed behavior are green.
Relevant lint, typecheck, and build checks are execution preconditions, not the
behavioral control. Record the exact focused test command and result. If the
focused tests are not green, abort: you cannot learn from an unknown state.

Not mid-coding. Not instead of a standards-vs-spec review. Run it even if that
review already ran. Keep the axes separate.

## What it is not

- **Standards** (layering, types, code smells)
- **Spec** (missing requirement, scope creep)
- **Performance tuning** (throughput, prompt growth, N+1)
- **SOLID/DRY** (second parser, discarded merge)
- **Random vandalism.** Planned experiments with a bounded blast radius only.

A chaos finding can *look* like another axis. Keep it if a concrete input
produces the wrong **observable output**. Hand style-only notes back.

## Experiments

Focus on observable output (what the caller, stream, or user sees), not
internal arrangement. For each experiment record:

1. **Control invariant** — the externally observable output that must remain
   true. For asynchronous or system behavior, include the metric and observation
   window.
2. **Variable** — one plausible changed condition applied through an existing
   public or test seam. Do not invent parameters or interfaces.
3. **Hypothesis** — `If we inject [variable], [control invariant] still holds.`
4. **Falsifier** — the exact observable result that would disprove it.
5. **Probe** — the command or focused test and its literal inputs.
6. **Reset** — how mutable state is restored before the next experiment.

Start with one variable at a time. Add at most one pairwise interaction when the
diff or failure history gives a concrete reason to suspect coupling.

Prefer executable probes over source inspection. A surviving probe increases
confidence only for that probe; it does not prove the invariant.

Before execution, bound the experiment: set a timeout, name permitted side
effects, and identify mutable state. Default to local or sandboxed execution.
Do not use live cloud mutation, deployment, destructive commands, real OAuth, or
external writes unless explicitly authorized. Stop on unexpected side effects.

Every reproducible, in-scope disproof becomes a regression test at the
narrowest stable seam. When no stable automated test is possible, record the
reason rather than claiming the defect is permanently fixed.

## Derive experiments from this ticket

Build a numbered list from:

1. The acceptance examples (the controls).
2. The deleted or replaced path (did the dead short-circuit survive?).
3. Adjacent identity the ticket said not to confuse.
4. Applicable generators from [references/variables.md](references/variables.md),
   instantiated with this ticket's names and written as hypotheses.

Select the 3–8 highest-risk experiments. Do not exhaust the catalog
mechanically. Skip a generator that cannot apply to this diff. Do not hunt
deferred sibling work.

## Drive the experiment pass

Run one isolated experimenter at a time per pass against a stable snapshot.
If the host can spawn a subagent, spawn one general-purpose subagent; otherwise
run the same prompt contract in this turn without mixing it into a standards or
spec review. The experimenter may create temporary probes but must not persist
changes to tracked files; the parent applies accepted fixes. Parallel
experimenters are allowed only when code, test state, caches, databases, and
temporary resources are isolated.

Later passes must carry the fixed list and the disposition ledger. A pass
without them reopens settled items and wastes the round.

### Subagent report contract

For each experiment return:

- `ID — PASS | FAIL | UNPROVEN`
- Variable and literal input
- Command or probe
- Expected observable output
- Observed output
- Evidence

For `FAIL`, also give the likely `file:line` when traced and the narrowest
stable regression-test seam. For `UNPROVEN`, state exactly what prevented
execution.

Do not modify tracked files. Do not report style, architecture, requirement
gaps, or performance-tuning advice. Keep the report under 400 words; summarize
successful experiments in one line each. Quote only the smallest relevant
expression when it materially supports the diagnosis.

### Prompt skeleton

```text
You are an isolated robustness experimenter for <ticket/change>.

Repository: <path>
Branch/commits: <branch, uncommitted state, or commit range>
Changed behavior: <bounded diff or files>
Read initially: <owning code, wiring, focused tests>
You may follow direct imports and callers needed to execute a probe, but do not
perform repo-wide review.

Baseline command: <command>
Baseline result: <green result>

Control invariant(s):
C1. <observable behavior>

Experiments:
E1. If we inject <variable>, <C1> still holds.
    Falsifier: <exact wrong output>
E2. ...

Already fixed:
- <ID, exact scenario, regression test>

Prior dispositions:
- <ID, scenario, disposition, reason, reopen condition>

Constraints:
- Do not modify tracked files.
- No deploy, destructive command, external write, real OAuth, or live-cloud
  mutation unless explicitly authorized.
- Reset mutable state between experiments.
- Stop an experiment on timeout or unexpected side effects.

Report each experiment as PASS, FAIL, or UNPROVEN with the literal input,
command, expected output, observed output, and evidence. For FAIL, identify the
likely location and regression-test seam.
```

## Parent judgment

The experimenter is advisory. The parent decides. Classify every experiment:

- **FIX** — the probe reproducibly violates an in-scope control invariant on
  current code. Add a regression test at the narrowest stable seam in the same
  change.
- **REJECTED** — the probe's expected output conflicts with a cited ticket,
  contract, or explicit product decision.
- **DEFERRED** — the violation is reproducible and valid but outside the
  authorized ticket scope. Preserve it as a real issue; do not describe it as
  not-a-defect.
- **NOT_REPRODUCED** — the exact probe does not produce the claimed result on
  current code. Record the command and observed output.
- **UNPROVEN** — the experiment could not be executed safely or cheaply.
- **SURVIVED** — the experiment ran and did not disprove the hypothesis.

For every non-SURVIVED item, carry forward its ID, exact scenario, evidence,
disposition, reason, and reopen condition. Suppress only an exact duplicate.
Reopen when the code, contract, environment, or supporting assumption changes.

Prefer repairing the existing decision point over adding a parallel parser,
matcher, or policy. Add a new decision point only when the current architecture
has no suitable owner and the ticket authorizes that change. Do not expand the
ticket.

## Loop

1. Run the original focused baseline; abort if it is not green.
2. Run one experiment pass.
3. Classify every result.
4. Apply accepted fixes and add regression tests.
5. Run the new regression tests first.
6. Rerun the original focused baseline and affected lint, typecheck, or build
   checks.
7. Run the next pass with the updated ledger and new applicable variables.
8. Stop when a pass finds no new reproducible in-scope violations.

Unproven or deferred items do not block stopping, but they must remain visible
in the final report.

## Output to the user

Keep standards and spec out of this report. For chaos:

- **Control invariant(s)** — the observable behaviors used as controls.
- **Fixed** — failed experiments and the regression tests added.
- **Rejected** — invalid expectations and the cited contract evidence.
- **Deferred** — valid reproducible issues outside the authorized scope.
- **Not reproduced** — exact probes that did not fail.
- **Unproven** — experiments not executed and why.
- **Survived** — executed hypotheses that were not disproved.

Do not commit, push, or move the issue tracker unless the user asked.
