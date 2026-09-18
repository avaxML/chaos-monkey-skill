# chaos-monkey

An [Agent Skill](https://agentskills.io/) for AI coding agents. After a
change's focused tests pass, it runs a bounded, hypothesis-driven pass that
tries to disprove the change's invariants with real inputs. Every reproducible
failure becomes a regression test.

It is not a style review, not a spec audit, and not production chaos
engineering.

## Install

```bash
npx skills add avaxML/chaos-monkey-skill
```

Works with Claude Code, Cursor, Codex, and other Agent Skills clients.

## When to use

Ask for a "chaos monkey pass", "adversarial input experiments", or "another
isolated pass, only executable counterexamples". Not for standards, spec
completeness, performance tuning, SOLID, or debugging failing tests.

## How it works

1. Confirm the focused tests are green.
2. Name the control invariant: the observable output that must keep holding.
3. Run 3–8 experiments of the form `If we inject X, Y still holds`, each with
   a named falsifier, an executed probe, and a state reset.
4. Classify each result: FIX, REJECTED, DEFERRED, NOT_REPRODUCED, UNPROVEN, or
   SURVIVED. Fixes get a regression test. The ledger carries into the next pass.
5. Rerun the new tests, then the original baseline, then loop until a pass
   finds nothing new.

Agent instructions: [SKILL.md](skills/chaos-monkey/SKILL.md). Injection
catalog: [variables.md](skills/chaos-monkey/references/variables.md).

## Releases

Bump `metadata.version` in `SKILL.md`, add a `CHANGELOG.md` entry, and tag the
matching SemVer version. skills.sh lists and ranks skills from anonymous CLI
install telemetry; there is nothing to submit.

## License

[MIT](LICENSE)
