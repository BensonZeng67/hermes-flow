# Workflow: tech-decision

**Pattern**: Same brief → parallel independent consultation → human synthesis  
**Output**: `artifacts/decision-log.md` — a structured, auditable architectural decision record

---

## When to Use This

- Choosing between two architectural approaches (e.g., REST vs GraphQL, SQL vs NoSQL)
- Evaluating a library or tool (e.g., which state management solution)
- Risk assessment before a significant refactor
- Any decision where you want independent perspectives before committing

**Do NOT use this for**: implementation tasks, bug fixes, or anything with a clear single answer.

---

## Role Definitions

| Role | Agent | Constraint |
|---|---|---|
| Consultant A | Claude Code | Responds independently, no access to Consultant B's output |
| Consultant B | Codex | Responds independently, no access to Consultant A's output |
| Synthesizer | You | Read both opinions, write the final decision log |

The independence constraint is critical. Do not show one agent's output to the other before synthesis.

---

## Phase 1 — Write the Problem Brief

Create `artifacts/brief.md`:

```markdown
## Problem
<one paragraph: what decision needs to be made and why now>

## Constraints
- <hard constraint 1 (non-negotiable)>
- <hard constraint 2>

## Options Under Consideration
- Option A: <name + one-line description>
- Option B: <name + one-line description>

## Context
<relevant codebase info, team size, timeline, existing stack>

## Question
Which option do you recommend, and why? What are the risks of your recommendation?
```

---

## Phase 2 — Parallel Consultation

Run both agents against the same brief. Do this in two separate terminals or sequentially — do NOT pipe one output into the other.

**Consultant A (Claude Code)**:
```bash
claude --print "You are a technical consultant. Read the following problem brief and give your independent recommendation. Be concrete, cite tradeoffs, and end with a clear verdict.\n\n$(cat artifacts/brief.md)" > artifacts/opinion-a.md
```

**Consultant B (Codex)**:
```bash
codex "You are a technical consultant. Read the following problem brief and give your independent recommendation. Be concrete, cite tradeoffs, and end with a clear verdict." < artifacts/brief.md > artifacts/opinion-b.md
```

---

## Phase 3 — Read Both Opinions

```bash
echo "=== CONSULTANT A (Claude Code) ===" && cat artifacts/opinion-a.md
echo ""
echo "=== CONSULTANT B (Codex) ===" && cat artifacts/opinion-b.md
```

Look for:
- **Agreement**: If both recommend the same option → high confidence, proceed
- **Disagreement**: Note which tradeoffs each agent weighted differently
- **Gaps**: Did either agent miss a constraint from the brief?

---

## Phase 4 — Synthesize Decision Log

Write `artifacts/decision-log.md` yourself (do not delegate this):

```markdown
# Decision Log — <topic>

**Date**: YYYY-MM-DD  
**Decision**: <Option A | Option B | Neither | Hybrid>  
**Status**: DECIDED | PENDING | SUPERSEDED

## Brief Summary
<one paragraph of the problem>

## What the Agents Said

### Consultant A (Claude Code)
Recommendation: <option>  
Key argument: <one sentence>  
Risk flagged: <one sentence>

### Consultant B (Codex)
Recommendation: <option>  
Key argument: <one sentence>  
Risk flagged: <one sentence>

## Where They Agreed
-

## Where They Differed
-

## Final Decision & Rationale
<Your reasoning. Why did you pick this? What did you override and why?>

## Risks Accepted
-

## Review Trigger
<What condition would make you revisit this decision? e.g., "if team grows past 5 engineers" or "if latency exceeds 200ms at p99">
```

---

## Example: Choosing a Database

```bash
mkdir -p artifacts

cat > artifacts/brief.md << 'EOF'
## Problem
We need to choose a primary database for a new internal tool that tracks equipment calibration records.

## Constraints
- Must run on-prem (no managed cloud services)
- Team has zero MongoDB experience
- Data is highly relational (equipment → calibration → engineer → approval)

## Options Under Consideration
- Option A: PostgreSQL
- Option B: SQLite (for simplicity, single-machine deployment)

## Context
Single server, ~10 concurrent users, ~50k records/year, no real-time requirements.

## Question
Which option do you recommend, and why? What are the risks of your recommendation?
EOF

# Run consultations
claude --print "You are a technical consultant. Read the following problem brief and give your independent recommendation.\n\n$(cat artifacts/brief.md)" > artifacts/opinion-a.md
codex "You are a technical consultant. Give your independent recommendation for this problem." < artifacts/brief.md > artifacts/opinion-b.md

# Review
cat artifacts/opinion-a.md
echo "---"
cat artifacts/opinion-b.md
```

---

*繁體中文說明：tech-decision workflow 的核心是「平行獨立諮詢」——兩個 AI 各自看同一份 brief，不互相參考對方答案，最後由你（人類）統整成決策日誌。這樣做能避免一個 AI 的偏見影響另一個，並留下可追溯的決策紀錄。*
