# chaos-monkey

An [Agent Skill](https://agentskills.io/) that runs **chaos engineering on a diff**: after the ticket tests are green, spawn one subagent that tries to **disprove** the acceptance invariants with concentrated real-world inputs.

It is not a style review, not a spec audit, and not Netflix-style instance killing.

## Install

Once this repo is public on GitHub:

```bash
npx skills add avaxML/chaos-monkey-skill
```

The CLI installs into Cursor, Claude Code, Codex, and other Agent Skills clients. Until the repo is public, install from this directory:

```bash
npx skills add ~/Projects/chaos-monkey-skill
```

Or copy or symlink `skills/chaos-monkey` into the agent's skills directory:

```bash
# Cursor
ln -s "$(pwd)/skills/chaos-monkey" ~/.cursor/skills/chaos-monkey

# Claude Code
ln -s "$(pwd)/skills/chaos-monkey" ~/.claude/skills/chaos-monkey

# Codex / shared agents
ln -s "$(pwd)/skills/chaos-monkey" ~/.agents/skills/chaos-monkey
ln -s "$(pwd)/skills/chaos-monkey" ~/.codex/skills/chaos-monkey
```

## When to use

- End of an implementation, after the proving tests and lint/typecheck
- "chaos monkey", "another review round", "another subagent loop", "iterate till no more findings"

Do not use it for standards, spec completeness, performance, or SOLID. Those are other axes.

## How it works

1. **Baseline probe** — if the happy-path tests are not green, abort.
2. **Steady state** — one-line acceptance output (the control).
3. **Experiments** — `If we inject X, Y still holds.` One concentrated variable per experiment. Prefer executing the function.
4. **Fix or reject** — a disproved hypothesis becomes a regression test. Rejected items stay on the list so the next pass does not reopen them.
5. **Loop** — at least two passes if the first pass changed code.

See [`skills/chaos-monkey/SKILL.md`](skills/chaos-monkey/SKILL.md) (agent instructions) and [`skills/chaos-monkey/references/variables.md`](skills/chaos-monkey/references/variables.md) (what to inject).

## Layout

```
skills/chaos-monkey/
  SKILL.md                 Agent instructions (agentskills.io)
  references/variables.md  Injection catalog, loaded on demand
  agents/openai.yaml       Codex UI metadata
LICENSE                    MIT
```

`npx skills add` discovers `skills/*/SKILL.md`. The skill `name` matches the folder.

## Publish (skills.sh)

There is no submit form. Create a **public** GitHub repo from this directory, then install once so telemetry can index it:

```bash
cd ~/Projects/chaos-monkey-skill
git init -b main
git add .
git commit -m "Initial chaos-monkey skill"
gh repo create chaos-monkey-skill --public --source=. --remote=origin --push
npx skills add avaxML/chaos-monkey-skill
```

Suggested GitHub topics: `agent-skills`, `claude-skills`, `cursor`, `codex`, `chaos-engineering`.

Tag releases with semver (`v0.1.0`) to match `metadata.version` in `SKILL.md`.

## License

MIT. See [LICENSE](LICENSE).
