---
name: explain-me
description: Use when the user asks to understand code changes, PRs, diffs, commits, or recent agent work. Prepare a rigorous explanation workspace, inspect every changed line internally, map every changed block to user-facing chapters, then teach the change incrementally without running tests.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [code-review, explanation, diff-analysis, pull-requests, teaching, software-development]
---

# explain-me

## Overview

`explain-me` is a rigorous workflow for explaining code changes so the user can understand and review them without manually reading every file from scratch.

This is **not** a lightweight summary skill. The agent must inspect every changed line internally, understand the relevant surrounding code, map every changed block, write explanation artifacts, and then teach the change in small interactive units until the user understands it.

The primary responsibility is explanation. The agent may add review-oriented concerns or improvement suggestions at the end of an explanation unit, but it must not approve, reject, merge, request changes, or make the final decision for the user.

## When to Use

Use this skill when:

- The user asks to understand code changes, e.g. "코드 설명해줘", "이 PR 설명해줘", "explain-me 해줘", "walk me through this diff".
- Project instructions such as `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or similar say to use `explain-me` after work.
- The user wants to understand agent-written changes before reviewing or accepting them.
- The user wants to understand an external PR, branch comparison, patch, or diff written by someone else.

Do **not** auto-trigger this skill unless the user asks for code explanation or project instructions explicitly require it after a task. Do not use this as a general runtime verification workflow; tests are read for understanding but not executed.

## Core Rules

1. **Do not skip any changed lines.** Inspect every added, removed, or modified line internally.
2. **Do not reduce scope to make explanation easier.** Keep the full confirmed scope and split the explanation into smaller units.
3. **Track completeness by diff hunk / changed block, not just by file.** Every changed block must be inventoried and mapped.
4. **Do not rely on the diff alone.** Inspect enough surrounding code, call paths, tests, docs, and configuration to explain meaning and impact.
5. **Do not run tests.** Read relevant tests to understand protection/coverage, but do not execute test commands as part of this skill.
6. **Write artifacts before teaching.** Always prepare `plan.md`, `README.md`, and `FIXME.md` under one explanation workspace.
7. **Teach incrementally.** Explain one small unit at a time, pause, and continue only when the user is ready unless they explicitly ask for an uninterrupted full explanation.
8. **Record only user-decided fixes.** Suggestions become `FIXME.md` entries only after the user decides what to do.

## Input Modes and Scope Rules

### 1. Recent-Work Mode

Use when the user asks to explain work just completed by the current agent.

Scope:

- All files and changed blocks actually modified by the recent task.
- Relevant uncommitted changes or recent commits needed to reconstruct the work.
- Surrounding code, call paths, tests, docs, and configuration related to those changes.

### 2. PR / Diff Mode

Use when the user provides a PR URL/number, branch comparison, commit range, patch, or diff.

Scope:

- The full PR/diff/commit range.
- PR title/body, commit messages, and tests when available.
- Surrounding code, call paths, docs, and configuration related to the changed blocks.

For external PRs/diffs, do not pretend to know the author's private intent. Explain likely intent from code evidence, PR text, commit messages, tests, and surrounding context. Mark uncertainty when needed:

- "The code suggests this change is trying to ..."
- "Based on the PR description and tests, this appears intended to ..."
- "The actual author intent may need confirmation."

### 3. Explicit-Range Mode

Use when the user specifies files, directories, commits, modules, or a feature area.

Scope:

- Everything inside the user-provided range.
- Every changed block in that range.
- Related surrounding code and tests needed to explain the range correctly.

### 4. Ambiguous Request Mode

If the user says only "this code", "this", "이거", "방금 한 것", or another ambiguous phrase:

- Inspect repository state, current branch, changed files, and recent context first.
- If the scope is clear from context, declare it briefly and proceed.
- If multiple materially different scopes are plausible, ask one clarification question before preparing artifacts.

## Explanation Workspace

Always create one explanation workspace:

```text
docs/explains/YYYYMMDD-topic/
├── plan.md
├── README.md
└── FIXME.md
```

Rules:

- Use the current local date for `YYYYMMDD`.
- Generate `topic` automatically from the PR title, branch name, dominant changed feature, or major changed files.
- The exact title is not important. Prefer short kebab-case.
- Do not interrupt the user just to ask for a title.
- Never overwrite an existing workspace.
- If `docs/explains/YYYYMMDD-topic/` already exists, create a unique sibling by appending `-2`, `-3`, etc.

## Artifact Roles

### `plan.md` — Internal Audit Trail and Explanation Plan

Optimize `plan.md` for completeness and traceability. It is the agent's work ledger proving that all changed blocks were inspected and mapped.

Include:

- Input mode and chosen scope.
- Source references: PR URL, branch comparison, commit range, working-tree diff, or user-provided range.
- Included and excluded ranges, if any.
- Changed-file inventory.
- Diff hunk / changed block inventory.
- Line/block-level analysis notes.
- Surrounding code, call paths, tests, docs, and config inspected.
- Mapping from every changed block to one or more explanation chapters.
- Completeness checklist.
- Interactive explanation sequence checklist.

Suggested changed block fields:

```md
- Block ID: B001
- File: `src/example.ts`
- Range: `42-61`
- Change type: added | removed | modified | moved | renamed
- Raw purpose observed from diff:
- Surrounding code inspected:
- Related tests/docs/config inspected:
- Explanation chapter mapping:
```

### `README.md` — User-Facing Explanation Document

Optimize `README.md` for understanding and readability. Always create it. It is the final explanation document used as the basis for chat teaching.

Use this structure:

```md
# Change Explanation: <topic>

## 1. One-Line Conclusion

## 2. Background You Need First

## 3. Overall Change Map

## 4. Chapter-by-Chapter Detailed Explanation

### Chapter N. <meaning-oriented title>
- Included scope:
  - `path/file.ext:line-line`
  - Changed block IDs from `plan.md`
- Why this changed
- What changed
- How the runtime/user flow works now
- Code block / line meaning
- Test perspective
- Caution or possible improvement, if any

## 5. End-to-End Flow Example

## 6. Tests / Protection Scope

## 7. What to Review Carefully

## 8. Explanation Completion Checklist
```

Do not mechanically list every line for the user. Internally inspect every changed line, then explain in meaningful blocks. Each chapter must state which changed blocks/files/line ranges it covers so completeness remains checkable.

### `FIXME.md` — User-Decided Follow-Up Changes

Always create `FIXME.md`, even if it starts empty.

Initial content may be:

```md
# FIXME

No user-decided follow-up changes yet.
```

Only record an item after the user makes a decision. A possible issue identified by the agent is not enough by itself.

Record decisions clearly:

```md
## Decided Follow-Ups

- [ ] <what will change>
  - Reason: <why>
  - Source: <agent suggestion | user request | discussion>
  - Related explanation section: <chapter/block>
```

## Required Analysis Procedure

1. **Establish scope.** Determine input mode and exact scope. Ask only if ambiguity materially changes the work.
2. **Collect changed blocks.** Gather all changed files and diff hunks / changed blocks. Do not rely only on filenames.
3. **Inspect context beyond the diff.** For every changed block, inspect enough surrounding code, callers/callees, tests, docs, and config to explain the change accurately.
4. **Read tests, but do not run tests.** Explain what tests protect or document; do not claim runtime correctness.
5. **Build `plan.md`.** Inventory every changed block and map each one to explanation chapters.
6. **Build `README.md`.** Reorganize by intent and understanding, not raw file order.
7. **Create `FIXME.md`.** Leave it neutral until the user decides follow-up changes.
8. **Brief the user before teaching.** Show a short briefing with:
   - chosen scope,
   - output directory,
   - number of changed files,
   - number of changed blocks,
   - explanation chapter list.
9. **Teach incrementally.** Present one chapter or sub-block at a time, then pause for the user.

## How to Express Reasoning

When the source material asks for "thinking" or "reasoning", do not reveal raw hidden chain-of-thought or produce a long private reasoning log.

Instead, explain design judgment and trade-offs:

- Why this change was needed.
- Why this location or structure changed.
- Why this implementation shape was chosen.
- What alternatives or trade-offs matter.
- What risks or future maintenance points exist.
- How this connects to user-visible or system behavior.

Use a natural narrative style, but keep each unit concise.

## Interactive Teaching Workflow

Do not paste the entire `README.md` into chat unless the user explicitly asks for a full uninterrupted explanation.

For each explanation unit:

1. Explain the unit in an easy order.
2. Mention included file/line/block scope when useful.
3. Add a review-oriented concern only if it naturally follows from the explanation:
   - "다만 `...`한 문제가 있을 수 있을 것 같습니다."
   - "그래서 `...`게 수정하는 것이 좋아 보입니다."
4. Ask whether the user understands or has questions.
5. Pause.

If the user asks a question:

1. Answer it directly.
2. Ask whether that resolves the question if needed.
3. Return to the planned explanation sequence.

Continue until every mapped changed block has been explained, unless the user explicitly stops.

## Review Boundaries

The primary responsibility is explanation, not final review judgment.

Do not:

- Approve a PR.
- Request changes.
- Merge.
- Change labels, reviewers, checks, or PR state.
- Present a suggestion as a decided FIXME without user confirmation.
- Run tests or claim that the change passed tests as part of this skill.

## Common Pitfalls

1. **Treating completeness and small explanations as conflicting.** They are complementary: keep full scope, split delivery.
2. **Summarizing instead of explaining.** The user needs understanding, not just a compressed changelog.
3. **Tracking only files.** A file can contain multiple unrelated hunks. Track changed blocks.
4. **Explaining in file order.** Use intent/meaning order.
5. **Skipping tests because tests are not executed.** Read relevant tests even though you do not run them.
6. **Inventing external author intent.** Mark uncertainty and ground claims in code/PR/test evidence.
7. **Dumping the whole README into chat.** Use it as preparation and reference; teach incrementally.
8. **Writing agent suggestions directly into FIXME.** FIXME is for user-decided follow-ups only.
9. **Overwriting old explanation workspaces.** Always create a unique directory.
10. **Treating this as a verification skill.** It explains code and review implications; it does not verify runtime correctness.

## Verification Checklist

Before starting interactive teaching, verify:

- [ ] Input mode is identified.
- [ ] Scope is clear or clarified.
- [ ] Workspace exists under `docs/explains/YYYYMMDD-topic/`.
- [ ] Existing workspace was not overwritten.
- [ ] `plan.md` exists.
- [ ] `README.md` exists.
- [ ] `FIXME.md` exists.
- [ ] Every changed file is inventoried.
- [ ] Every diff hunk / changed block is inventoried.
- [ ] Every changed block maps to at least one README chapter.
- [ ] Surrounding code and relevant call paths were inspected.
- [ ] Relevant tests/docs/config were read when applicable.
- [ ] No tests were run as part of this skill.
- [ ] README uses the required structure.
- [ ] User briefing includes scope, output directory, changed file count, changed block count, and chapter list.

Before finishing the whole explanation, verify:

- [ ] Every mapped changed block has been explained in chat or the user explicitly requested to stop.
- [ ] User questions were answered and the explanation returned to the plan.
- [ ] Possible issues were presented as suggestions, not decisions.
- [ ] User-decided follow-ups, if any, were recorded in `FIXME.md`.
