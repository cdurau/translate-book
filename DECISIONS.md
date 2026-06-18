# Decisions
## Decision Log
- Date: 2026-06-18
- Decision: Use Codex-standard `SKILL.md` frontmatter with only `name` and `description`, plus `agents/openai.yaml` for UI metadata.
- Reason: Codex uses frontmatter for triggering and product-specific UI metadata from the `agents` directory.
- Alternatives Considered: Keep Claude/OpenClaw-only tool and dependency fields in frontmatter; create a duplicated Codex-only skill directory.
- Consequences: The repository root is directly installable by Codex without duplicating the pipeline; runtime prerequisites remain documented in the README.

- Date: 2026-06-18
- Decision: Require an explicit request for parallel/sub-agent work before Codex spawns chunk translators.
- Reason: Codex sub-agent workflows require explicit user opt-in.
- Alternatives Considered: Spawn agents whenever the skill triggers implicitly.
- Consequences: Example prompts mention parallel subagents, while implicit invocations ask for confirmation before translation workers start.
