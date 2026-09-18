# Contributing

- Open a pull request against `main`. Direct pushes are blocked.
- Keep `SKILL.md` under the agentskills.io conventions: `name` must equal the
  folder name, the `description` is what agents use to decide when to trigger.
- Bump `metadata.version` in `SKILL.md` and add a `CHANGELOG.md` entry for any
  change that alters what the agent does.
- Do not add instructions that push, deploy, mutate cloud resources, or read
  secrets. The skill's blast-radius rules are deliberate.
- The `validate` workflow must pass before merge.
