# Changelog

## 0.2.0 — 2026-09-18

- Narrowed the trigger description; the skill no longer claims every implementation.
- Replaced the single reject list with six dispositions (FIX, REJECTED, DEFERRED, NOT_REPRODUCED, UNPROVEN, SURVIVED) so valid-but-deferred and unproven findings are no longer suppressed as rejections.
- Loop now reruns the original focused baseline after fixes, not only the new regression tests.
- Experiments gain explicit falsifier, probe, reset, timeout, and side-effect bounds; experimenter must not persist changes to tracked files.
- Report contract uses PASS/FAIL/UNPROVEN per experiment with a 400-word budget.
- `references/variables.md` rewritten as 19 general failure classes; project-specific nouns removed.
- Bundled `LICENSE` inside the skill directory; CI now runs the `skills-ref` validator instead of a substring check.
- Added `evals/trigger-queries.json` with should-trigger and near-miss queries.

## 0.1.0 — 2026-09-18

- First publishable package: Agent Skills layout, MIT license, Codex `agents/openai.yaml`.
- Experiments follow [Principles of Chaos](https://principlesofchaos.org/): steady state, hypothesis, concentrated variable, try to disprove.
- Variable catalog lives in `skills/chaos-monkey/references/variables.md`.
