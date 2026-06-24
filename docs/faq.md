# FAQ — hermes-flow

---

## General

**Q: This is just shell scripts and markdown files. Where's the actual framework?**

That's intentional. hermes-flow is a *protocol*, not a framework. The value is in the role definitions, handoff contracts, and artifact schemas — not in runtime infrastructure. Any "framework" here would add dependencies without adding correctness. Shell + markdown is auditable, portable, and not abandoned when a dependency breaks.

---

**Q: Can I use this with other agents besides Claude Code and Codex?**

Yes. The workflows assume Claude Code as Builder/Consultant A and Codex as Reviewer/Consultant B, but any agent that can read stdin and write stdout works. Substitute with `gemini`, `llama`, or any CLI agent. Adjust the prompts as needed — the output contract (the structure of the artifact) is what matters, not which agent produces it.

---

**Q: Why is the merge step always manual?**

Because merging requires whole-system judgment. An agent only sees its assigned subtask. Only you have context about naming conventions, existing patterns, shared interfaces, and what changed in the last commit. Delegating merge to an agent means trusting it with context it doesn't have — that's how silent bugs get introduced. Do it yourself.

---

**Q: Do I need to run Claude Code and Codex simultaneously?**

No. Sequential execution is fine for most workflows. Only the `parallel-build` workflow benefits from simultaneous execution (two terminals), and even then, sequential is safer for beginners. The "parallel" in parallel-build refers to logical independence, not temporal concurrency.

---

## Workflow-Specific

**Q: In code-review, why can't the Reviewer see the original task brief?**

This enforces independent review. If the Reviewer knows what the Builder was *trying* to do, they'll evaluate intent rather than output. The whole point is to catch the gap between what was intended and what was actually implemented. The Reviewer should evaluate the artifact as if it arrived without context.

---

**Q: What if both consultants give the same recommendation in tech-decision? Is the workflow useful?**

Yes — agreement increases confidence. A decision where two independent agents with different training converge is stronger than one agent's solo recommendation. Document the agreement explicitly in the decision log. Agreement also often surfaces different supporting arguments, which is valuable context.

---

**Q: The review-log says REJECTED. Do I go back to the same Claude Code session?**

You can, but you don't have to. Start a new session and paste in: (1) the original task, (2) the builder-output.md, (3) the review-log.md. Ask Claude Code to address the Required Changes. This gives a clean context and produces a traceable v2 artifact.

---

**Q: Can I use `--print` with Claude Code for all tasks?**

`--print` (non-interactive mode) works well for well-specified tasks. For complex or ambiguous tasks, interactive mode is better — you can steer mid-task. A good rule: use `--print` when you can fully specify the task in one prompt, use interactive when you expect to need clarification.

---

**Q: What goes in the `artifacts/` directory? Should I `.gitignore` it?**

`artifacts/` contains intermediate outputs: builder outputs, review logs, opinions, specs. Whether to commit them is a team choice:

- **Commit**: full audit trail, useful for post-mortems, aligns with the "auditable" positioning
- **Ignore**: cleaner repo, less noise in git history, treat artifacts as ephemeral scratch

Recommended: commit `decision-log.md` and `parallel-build-log.md` (they're human-written synthesis), gitignore raw agent outputs (`builder-output.md`, `opinion-a.md`, etc.).

---

## Troubleshooting

**Q: `codex` command not found**

Install the Codex CLI: follow instructions at the [Codex repository](https://github.com/openai/codex). Verify with `codex --version`.

---

**Q: `claude --print` hangs or produces no output**

- Verify Claude Code is authenticated: run `claude` interactively first
- Check that the prompt is not empty: `echo "$(cat prompts/builder.txt)"` should print content
- Try without stdin: `claude --print "hello"` as a smoke test

---

**Q: The Reviewer artifact is in the wrong format (no Verdict header, no checklist)**

This happens when Codex doesn't follow the output contract. Options:
1. Prepend the reviewer prompt more forcefully: `codex "CRITICAL: You MUST follow this exact output format. $(cat prompts/reviewer.txt)"  < artifacts/builder-output.md`
2. Re-run with a shorter artifact (Codex may have truncated if input was too long)
3. Run the review interactively and paste the reviewer prompt as your first message

---

**Q: I want to add a fourth workflow type. How?**

1. Add `workflows/your-workflow.md` following the same structure (roles, phases, output contract, example)
2. Add corresponding prompts to `prompts/` if the workflow needs role-specific prompt templates
3. Update the workflow table in `README.md`
4. Update the `SKILL.md` triggers section if you want a `/hermes your-workflow` command

Submit a PR if it's generally useful.

---

*繁體中文說明：這份 FAQ 涵蓋最常見的疑問——為什麼不做自動化合併、為什麼 Reviewer 看不到原始需求、artifacts 目錄該不該 commit 等。如果遇到 `codex` 找不到或 `claude --print` 沒有輸出，先看最後的 Troubleshooting 區塊。*
