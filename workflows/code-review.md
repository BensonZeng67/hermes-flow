# Workflow: code-review

**Pattern**: Builder (Claude Code) → artifact handoff → Reviewer (Codex)  
**Output**: `artifacts/review-log.md` with structured verdict

---

## Role Definitions

| Role | Agent | Responsibility |
|---|---|---|
| Builder | Claude Code | Implement the requested change, produce a diff or file output |
| Reviewer | Codex | Review the artifact against a checklist, issue verdict |
| Orchestrator | You | Trigger each phase, inspect artifacts, decide on merge |

---

## Phase 1 — Builder

**Goal**: Produce a concrete code artifact (new file, patch, or implementation).

```bash
# Option A: interactive (recommended for complex tasks)
claude

# Option B: non-interactive with --print (pipe-friendly)
claude --print "$(cat prompts/builder.txt)" > artifacts/builder-output.md
```

**Builder prompt contract** (see `prompts/builder.txt`):
- Must state what was changed and why
- Must include the full diff or file content in a fenced code block
- Must list assumptions made

**Expected output structure**:
```
## Summary
<one paragraph: what was changed, why>

## Artifact
```diff
<full diff or file>
```

## Assumptions
- <assumption 1>
- <assumption 2>
```

---

## Phase 2 — Handoff

Verify the artifact is complete before passing to Reviewer.

```bash
# Sanity check: artifact exists and is non-empty
test -s artifacts/builder-output.md && echo "OK: artifact ready" || echo "FAIL: artifact missing or empty"
```

If the Builder output is incomplete, go back to Phase 1.

---

## Phase 3 — Reviewer

**Goal**: Independent review of the artifact. Reviewer must not have access to the original task brief — only the artifact.

```bash
# Codex reviews the artifact
codex "$(cat prompts/reviewer.txt)" < artifacts/builder-output.md > artifacts/review-log.md
```

**Reviewer checklist** (enforced in `prompts/reviewer.txt`):
1. Correctness — does the code do what the summary claims?
2. Edge cases — what inputs would break this?
3. Security — any injection, auth bypass, or data exposure risk?
4. Readability — would a new engineer understand this in 5 minutes?
5. Test coverage — are there tests? Should there be?

**Expected review-log structure**:
```
## Verdict
APPROVED | APPROVED_WITH_NOTES | REJECTED

## Checklist
- [ ] Correctness: ...
- [ ] Edge cases: ...
- [ ] Security: ...
- [ ] Readability: ...
- [ ] Tests: ...

## Required Changes (if REJECTED or APPROVED_WITH_NOTES)
1. ...

## Notes
...
```

---

## Phase 4 — Resolution

```bash
# Read the verdict
cat artifacts/review-log.md | head -5
```

| Verdict | Next step |
|---|---|
| `APPROVED` | Merge / commit the artifact |
| `APPROVED_WITH_NOTES` | Apply notes, commit with review-log linked |
| `REJECTED` | Return to Phase 1 with review-log as context |

---

## Full Example (one-liner chain)

```bash
# Create artifacts dir if not exists
mkdir -p artifacts

# Phase 1
claude --print "$(cat prompts/builder.txt)" > artifacts/builder-output.md

# Phase 2 check
test -s artifacts/builder-output.md || exit 1

# Phase 3
codex "$(cat prompts/reviewer.txt)" < artifacts/builder-output.md > artifacts/review-log.md

# Phase 4
echo "=== VERDICT ===" && grep "^## Verdict" -A1 artifacts/review-log.md
```

---

## Artifact Naming Convention

```
artifacts/
├── builder-output.md          # Phase 1 output
├── review-log.md              # Phase 3 output
└── YYYYMMDD-task-name/        # archive by date+task for multi-iteration
    ├── builder-output-v1.md
    ├── review-log-v1.md
    └── builder-output-v2.md
```

---

*繁體中文說明：code-review workflow 分三個階段——Builder（Claude Code）負責實作並輸出 artifact；Reviewer（Codex）獨立審查，不看原始需求；Orchestrator（你）根據 verdict 決定下一步。每個 artifact 都留存，可追溯每次迭代的決策過程。*
