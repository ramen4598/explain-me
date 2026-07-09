# explain-me

A portable, explicitly-invoked agent skill for rigorous code-change explanations.

`explain-me` is designed for Hermes Agent, Codex, OpenCode, Claude, and other coding agents. It tells an agent how to inspect every changed line internally, map every diff hunk / changed block, prepare explanation artifacts, and then teach the change incrementally without running tests.

The skill is intentionally **explicit invocation only**. Agents should use it only when the user names `explain-me`, for example `explain-me 해줘`, `use explain-me`, or `/skill explain-me`. Ordinary requests like "이 PR 설명해줘" or "walk me through this diff" should not automatically trigger this skill.

## What it does

- Explains recent agent work, PRs, diffs, commits, branches, files, or explicit ranges.
- Creates a workspace under `docs/explains/YYYYMMDD-topic/`.
- Always writes:
  - `plan.md` — internal audit trail and changed-block mapping.
  - `EXPLANATION.md` — user-facing explanation document.
  - `FIXME.md` — user-decided follow-up changes only.
- Reads relevant tests/docs/config for understanding, but does **not** run tests.
- Explains in small interactive units and pauses for user questions.

## Files

- [`SKILL.md`](./SKILL.md) — the skill itself.

## Use with Hermes Agent

Copy this repository's `SKILL.md` into a Hermes skill directory, for example:

```text
~/.hermes/skills/software-development/explain-me/SKILL.md
```

Then start a new Hermes session so the skill loader can pick it up.

Invoke it explicitly when needed, for example:

```text
/skill explain-me
```

or:

```text
explain-me로 이 PR 설명해줘
```

## Use with other agents

Paste or reference `SKILL.md` in your agent instructions. The body is intentionally written as a portable Markdown procedure and avoids relying on Hermes-specific tool names.

## License

MIT
