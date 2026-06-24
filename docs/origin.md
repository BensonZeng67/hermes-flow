# Origin

This repository documents a **cross-AI collaboration workflow** discovered during a discussion between Hermes Agent (orchestrator), Codex (architect), and Claude Code (implementer).

## The Session (2026-06-24)

The question was: *"Can my multi-agent collaboration workflow (from the 'coop' skill) be turned into a reusable open-source project?"*

**Codex proposed** the architecture:
- Three workflow patterns: code-review, tech-decision, parallel-build
- Blind-review design (reviewer doesn't see the original brief)
- Artifact-based handoff contracts (no shared state)
- Manual merge as an architectural decision, not a limitation

**Claude Code generated** all implementation files from a structured task prompt.

**Hermes Agent (this session)** orchestrated the discussion, consolidated the decisions, reviewed and published the repository.

## Why This Matters

The architecture wasn't designed by a human and then implemented — it was **proposed by one AI (Codex), implemented by another (Claude Code), and reviewed/published by a third (Hermes Agent)**. The entire process from idea to GitHub repo took under 30 minutes.

## License

MIT — same as the parent repository.
