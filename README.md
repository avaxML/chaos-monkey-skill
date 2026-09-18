# chaos-monkey

An [Agent Skill](https://agentskills.io/) that runs a bounded,
hypothesis-driven adversarial pass over changed behavior after its focused tests
are green.

It borrows the experiment discipline of chaos engineering (control invariant,
falsifiable hypothesis, realistic variable, observed disproof, bounded side
effects, automation) and applies it to a diff. It is not a style review, not a
spec audit, and not production chaos engineering.

## Install

```bash
npx skills add avaxML/chaos-monkey-skill
```

For a local checkout:

```bash
npx skills add .
```

The CLI installs into Claude Code, Cursor, Codex, and other Agent Skills clients.

## When to use

- After the focused tests for a change are green, when you want executable
  counterexamples rather than another read-through
- "chaos monkey pass", "adversarial input experiments", "another isolated pass,
  only report executable counterexamples", "iterate till no new defects"

Do not use it for standards, spec completeness, performance tuning, SOLID, or
failing-test debugging. Those are other axes.

## How it works

1. **Baseline** — the focused acceptance tests must be green, or abort.
2. **Control invariant** — one-line observable output that must keep holding.
3. **Experiments** — `If we inject X, Y still holds.` One variable per
   experiment, a named falsifier, an executed probe, and a state reset.
4. **Dispositions** — every result is FIX, REJECTED, DEFERRED, NOT_REPRODUCED,
   UNPROVEN, or SURVIVED. Fixes get a regression test. The ledger carries into
   the next pass so items are not reopened or silently lost.
5. **Loop** — rerun new tests, then the original baseline, then the next pass.
   Stop when a pass finds no new reproducible in-scope violations.

See [`skills/chaos-monkey/SKILL.md`](skills/chaos-monkey/SKILL.md) (agent
instructions) and
[`skills/chaos-monkey/references/variables.md`](skills/chaos-monkey/references/variables.md)
(what to inject).

## Layout

```
skills/chaos-monkey/
  SKILL.md                 Agent instructions (agentskills.io)
  references/variables.md  Injection catalog, loaded on demand
  agents/openai.yaml       Codex UI metadata
  LICENSE                  MIT, bundled with the installed skill
evals/trigger-queries.json Should-trigger and near-miss queries
```

This repository uses a supported catalog layout:
`skills/chaos-monkey/SKILL.md`. The frontmatter `name` matches the containing
directory as required by the Agent Skills specification.

## Releases

Update `CHANGELOG.md` and `metadata.version` for behavior changes, then create a
matching SemVer tag. `metadata.version` is repository metadata; Agent Skills
clients do not use it for dependency resolution.

skills.sh listing and ranking are generated automatically from anonymous CLI
install telemetry; there is no manual submission form.

## License

MIT. See [LICENSE](LICENSE).
