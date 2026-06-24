# hermes-flow

> **Auditable multi-agent engineering review workflows for Claude Code + Codex.**

Not a chat room. Not a tool switcher.  
A structured orchestrator pattern where Claude Code and Codex collaborate on real engineering tasks — with every decision logged and reproducible.

---

## Why hermes-flow?

Most "multi-agent" setups are glorified round-robin chat. hermes-flow enforces **roles**, **handoff contracts**, and **artifact outputs** so you get a reviewable paper trail, not a conversation history.

> 💡 **Origin**: This architecture was proposed by **Codex** during a cross-AI collaboration session — the orchestrator pattern, blind-review design, and artifact contract. Claude Code generated the implementation files. See [`docs/origin.md`](docs/origin.md) for the session log.

| | hermes-flow | Generic multi-agent chat | Claude Code solo |
|---|---|---|---|
| Auditable decision log | ✅ | ❌ | ❌ |
| Parallel agent execution | ✅ | partial | ❌ |
| Role-locked prompts | ✅ | ❌ | ❌ |
| No infra required | ✅ | varies | ✅ |
| Codex integration | ✅ | ❌ | ❌ |

---

## Three Workflows

### 1. `code-review` — Builder → Reviewer
Claude Code writes or modifies code. Codex reviews with a structured checklist. Output: annotated diff + approval/rejection verdict.

```
Builder (Claude Code) ──writes──▶ artifact ──handoff──▶ Reviewer (Codex) ──▶ review-log.md
```

### 2. `tech-decision` — Parallel Consultation
Same problem brief sent to Claude Code and Codex independently. You synthesize a decision log from two perspectives.

```
Problem Brief ──▶ Claude Code ──▶ opinion-A.md
              └──▶ Codex      ──▶ opinion-B.md
                                        ▼
                              decision-log.md (you synthesize)
```

### 3. `parallel-build` — Task Distribution
Break a large task into independent subtasks. Assign each to an agent. Manually merge outputs.

```
Spec ──▶ [subtask-1 → Agent A]
     ├──▶ [subtask-2 → Agent B]
     └──▶ [subtask-3 → Agent A]
                  ▼
            merge + integration test
```

---

## 5-Minute Quickstart

### Prerequisites
- [Claude Code](https://claude.ai/code) installed and authenticated
- [Codex CLI](https://github.com/openai/codex) installed (`codex --version` works)
- A git repo to work in

> ⚠️ **Both tools require paid access**: Claude Code needs a [Claude Pro or Team subscription](https://claude.ai) (or Anthropic API key). Codex CLI needs an [OpenAI API key with billing enabled](https://platform.openai.com/settings/organization/billing) or a Codex Pro subscription. Make sure billing is set up on both before starting.

### Run your first code-review workflow

**Step 1 — Builder phase (Claude Code)**
```bash
claude --print < prompts/builder.txt > artifacts/builder-output.md
```

**Step 2 — Reviewer phase (Codex)**
```bash
codex "$(cat prompts/reviewer.txt)" < artifacts/builder-output.md > artifacts/review-log.md
```

**Step 3 — Inspect the log**
```bash
cat artifacts/review-log.md
```

See [`workflows/code-review.md`](workflows/code-review.md) for the full protocol.

---

## Project Structure

```
hermes-flow/
├── workflows/
│   ├── code-review.md       # Builder→Reviewer protocol
│   ├── tech-decision.md     # Parallel consultation protocol
│   └── parallel-build.md    # Task distribution protocol
├── prompts/
│   ├── builder.txt          # Claude Code prompt template
│   └── reviewer.txt         # Codex prompt template
├── docs/
│   └── faq.md
├── SKILL.md                 # Hermes skill definition (installable)
└── README.md
```

---

## Similar Projects

| Project | Difference |
|---|---|
| [AutoGen](https://github.com/microsoft/autogen) | Framework-heavy, Python runtime required, agents are autonomous loops |
| [CrewAI](https://github.com/crewAIInc/crewAI) | Similar role concept but tightly coupled to LangChain, no Claude Code native support |
| [OpenHands](https://github.com/All-Hands-AI/OpenHands) | Full agent runtime with browser/container, much heavier than hermes-flow |
| Claude Code `/review` skill | Single-agent, no Codex handoff, no decision log artifact |

hermes-flow is intentionally **thin**: shell commands, markdown files, no framework lock-in.

---

## License

MIT © 2026 hermes-flow contributors

---

*繁體中文說明：hermes-flow 是一套以 orchestrator pattern 運作的 multi-agent 工程審查工作流，核心概念是讓 Claude Code（Builder 角色）與 Codex（Reviewer 角色）以結構化方式協作，每次決策都留有可追溯的 artifact，不是聊天室，是有紀律的工程流程。*
