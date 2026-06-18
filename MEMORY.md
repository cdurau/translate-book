# Project Memory
## Purpose
Codex skill for translating complete PDF, DOCX, and EPUB books with parallel chunk workers.
## Main Goals
- Preserve document structure and formatting.
- Keep terminology consistent across independent translation agents.
- Support resumable, validated, multi-format builds.
## Core Features
Conversion, chunking, glossary injection, neighbor context, selective re-translation, manifest validation, and HTML/DOCX/EPUB/PDF output.
## Important Constraints
- Use only `chunk*.md` source naming.
- Keep `{baseDir}` in skill script paths.
- Keep sub-agent tasks portable across supported runtimes.
- Do not add mtime-based output rebuild logic.
## Conventions
Python scripts own deterministic state and validation; agents own translation and semantic glossary decisions.
## Key Files
- `SKILL.md`: orchestration contract
- `scripts/convert.py`: input conversion and chunking
- `scripts/run_state.py`: selective work planning
- `scripts/merge_and_build.py`: merge and output generation
- `tests/`: stdlib unit tests and baseline assets
## Quick Resume
Read `CURRENT_STATE.md` and `TASKS.md`, then inspect only the script or skill section relevant to the requested change.
