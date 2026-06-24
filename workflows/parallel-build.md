# Workflow: parallel-build

**Pattern**: Spec → task decomposition → parallel agent execution → manual merge  
**Output**: Independent subtask artifacts, merged by you into a coherent whole

---

## When to Use This

- Large features that decompose cleanly into independent modules
- Generating boilerplate for multiple similar components
- Writing tests for multiple functions in parallel
- Documentation passes over large codebases

**Prerequisite for parallelism**: subtasks must have **no runtime dependency on each other**. If Task B needs Task A's output as input, use sequential execution, not parallel.

---

## Phase 1 — Write the Spec

Create `artifacts/spec.md`:

```markdown
## Goal
<one sentence: what is the end state>

## Decomposition Rule
<how you split this: by module, by layer, by file, by feature flag, etc.>

## Subtasks

### Task 1: <name>
- Input: <what the agent needs to know>
- Output: <what file or artifact is expected>
- Constraints: <anything it must not do>

### Task 2: <name>
- Input: ...
- Output: ...
- Constraints: ...

### Task 3: <name>
- Input: ...
- Output: ...
- Constraints: ...

## Integration Notes
<how you plan to merge the outputs — what naming conventions to use, what the shared interface is>
```

---

## Phase 2 — Assign Agents

Each subtask gets its own prompt. Agents run independently.

**Format for each assignment**:
```bash
# Task 1 → Claude Code
claude --print "You are working on subtask 1 of a parallel build.

TASK: <task 1 description>
OUTPUT FORMAT: <expected file/format>
CONSTRAINTS: <what not to do>

Do not reference or wait for other tasks. Produce only your assigned output." > artifacts/task-1-output.md

# Task 2 → Codex (in a separate terminal or sequentially)
codex "You are working on subtask 2 of a parallel build.

TASK: <task 2 description>
OUTPUT FORMAT: <expected file/format>
CONSTRAINTS: <what not to do>" > artifacts/task-2-output.md

# Task 3 → Claude Code (can reuse same agent for non-conflicting tasks)
claude --print "You are working on subtask 3 of a parallel build. ..." > artifacts/task-3-output.md
```

---

## Phase 3 — Verify Outputs

Before merging, verify each output meets its contract:

```bash
# Check all task outputs exist and are non-empty
for i in 1 2 3; do
  if test -s "artifacts/task-$i-output.md"; then
    echo "Task $i: OK"
  else
    echo "Task $i: MISSING or EMPTY — re-run before merging"
  fi
done
```

Common failure modes:
- Agent refused the task (check for refusal language at the start of output)
- Output is incomplete (truncated mid-function)
- Output violates a constraint (e.g., introduced a dependency it wasn't supposed to)

---

## Phase 4 — Manual Merge

**Do not ask an agent to merge.** You own this step.

Merge checklist:
- [ ] Each output placed in its correct file/location
- [ ] Naming collisions resolved (check for duplicate function names, variable names)
- [ ] Shared interfaces are consistent (e.g., function signatures that call each other)
- [ ] No duplicated logic across subtask outputs
- [ ] Integration test or smoke test passes

```bash
# Example: copy subtask outputs into the actual project
cp artifacts/task-1-output.md src/module-a.ts   # adjust as needed
cp artifacts/task-2-output.md src/module-b.ts
cp artifacts/task-3-output.md src/module-c.ts

# Run whatever test/check applies to your project
# e.g.: cargo check, tsc --noEmit, pytest, etc.
```

---

## Phase 5 — Log the Build

Write `artifacts/parallel-build-log.md`:

```markdown
# Parallel Build Log — <feature name>

**Date**: YYYY-MM-DD  
**Spec**: artifacts/spec.md

## Tasks

| Task | Agent | Status | Output |
|---|---|---|---|
| task-1 | Claude Code | MERGED | src/module-a.ts |
| task-2 | Codex | MERGED | src/module-b.ts |
| task-3 | Claude Code | MERGED | src/module-c.ts |

## Merge Notes
<anything surprising, conflicts resolved, decisions made during merge>

## Integration Result
<did the smoke test pass? any follow-up tasks?>
```

---

## Example: Writing Tests for Three Modules

```bash
mkdir -p artifacts

# Task 1: tests for auth module
claude --print "Write unit tests for the following auth module. Use the existing test style in the codebase. Output only the test file content, no explanation.\n\n$(cat src/auth.ts)" > artifacts/task-1-auth-tests.md

# Task 2: tests for payment module
codex "Write unit tests for the following payment module. Output only the test file content." < src/payment.ts > artifacts/task-2-payment-tests.md

# Task 3: tests for notification module
claude --print "Write unit tests for the following notification module. Output only the test file content.\n\n$(cat src/notification.ts)" > artifacts/task-3-notification-tests.md

# Verify
for i in 1 2 3; do test -s "artifacts/task-$i-*.md" && echo "OK" || echo "FAIL"; done

# Merge (manually copy content to actual test files)
```

---

## Decomposition Heuristics

| Situation | Good split | Bad split |
|---|---|---|
| 3 independent API endpoints | 1 task per endpoint | Split by layer (all models in one task) |
| Generate tests for 10 functions | 5 functions per task | All 10 in one task |
| Write docs for 4 modules | 1 task per module | Split by section type |
| Refactor with shared state | ❌ Don't parallelize | ❌ Don't parallelize |

---

*繁體中文說明：parallel-build 的關鍵是「任務必須真正獨立」——如果兩個子任務的輸出互相依賴，就不能並行，應改成循序執行。合併步驟永遠由人類執行，不委託 AI，因為衝突解決需要整體判斷，不是局部知識。*
