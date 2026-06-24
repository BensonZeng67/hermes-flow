---
name: hermes-flow
version: 0.1.0
description: Auditable multi-agent engineering review workflows (code-review, tech-decision, parallel-build)
author: hermes-flow contributors
license: MIT
install: hermes skills install
triggers:
  - "/hermes"
  - "/hermes-review"
  - "/hermes-decision"
  - "/hermes-build"
requires:
  - codex
---

# Hermes Skill

A Claude Code skill that surfaces the hermes-flow workflow protocols inline.

## Commands

### `/hermes review`
Launch the code-review workflow.

```
Invokes: workflows/code-review.md protocol
Produces: artifacts/builder-output.md, artifacts/review-log.md
Roles: Builder (Claude Code), Reviewer (Codex), Orchestrator (you)
```

When invoked, Claude Code will:
1. Ask you to describe the task (or paste it directly)
2. Execute the Builder phase using `prompts/builder.txt`
3. Output the artifact
4. Prompt you to run the Reviewer phase with Codex
5. Wait for you to paste the review-log back for resolution

### `/hermes decision`
Launch the tech-decision workflow.

```
Invokes: workflows/tech-decision.md protocol
Produces: artifacts/brief.md, artifacts/opinion-a.md, artifacts/opinion-b.md, artifacts/decision-log.md
Roles: Consultant A (Claude Code), Consultant B (Codex), Synthesizer (you)
```

When invoked, Claude Code will:
1. Ask you to fill out the problem brief template
2. Save it to `artifacts/brief.md`
3. Execute Consultant A phase
4. Give you the Codex command to run independently
5. Provide the decision-log template for you to fill in

### `/hermes build`
Launch the parallel-build workflow.

```
Invokes: workflows/parallel-build.md protocol
Produces: artifacts/spec.md, artifacts/task-N-output.md, artifacts/parallel-build-log.md
Roles: Agents (Claude Code + Codex), Merger (you)
```

When invoked, Claude Code will:
1. Help you fill out the spec template and identify subtasks
2. Execute the tasks assigned to Claude Code
3. Give you Codex commands for tasks assigned to Codex
4. Provide the merge checklist and build log template

---

## Installation

### Option A: Reference install (recommended)
Clone the repo and point your Claude Code config at it:

```bash
git clone https://github.com/YOUR_USERNAME/hermes-flow ~/.claude/skills/hermes-flow
```

Then add to your `CLAUDE.md`:
```markdown
@~/.claude/skills/hermes-flow/SKILL.md
```

### Option B: Copy prompts only
```bash
cp hermes-flow/prompts/builder.txt ~/.claude/prompts/hermes-builder.txt
cp hermes-flow/prompts/reviewer.txt ~/.claude/prompts/hermes-reviewer.txt
```

---

## Artifact Directory Convention

hermes-flow always writes to `./artifacts/` relative to your current working directory.

```
artifacts/
├── brief.md                    # tech-decision brief
├── builder-output.md           # code-review builder artifact
├── decision-log.md             # tech-decision synthesis
├── opinion-a.md                # tech-decision consultant A
├── opinion-b.md                # tech-decision consultant B
├── parallel-build-log.md       # parallel-build log
├── review-log.md               # code-review reviewer log
├── spec.md                     # parallel-build spec
└── task-N-output.md            # parallel-build subtask outputs
```

Add `artifacts/` to your `.gitignore` if you don't want to commit intermediate outputs, or commit selectively for audit trail.

---

## Versioning

This skill follows the hermes-flow repository version. Breaking changes to the artifact schema will increment the minor version. Check `SKILL.md` version field after pulling updates.

---

*繁體中文說明：SKILL.md 是 hermes-flow 的 Claude Code skill 定義檔。透過 `@` 引用此檔案，可以在任何 Claude Code 專案中啟用 `/hermes` 指令，直接在對話中啟動三種 workflow，而不需要記住所有指令參數。*
