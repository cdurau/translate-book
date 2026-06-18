# Architecture
## Stack
Codex Skill Markdown, Python 3, Pandoc/pypandoc, and Calibre.
## Project Structure
`SKILL.md` orchestrates deterministic helpers in `scripts/`; `tests/` covers script behavior.
## State Management
`manifest.json` tracks source chunks, `glossary.json` stores canonical terms, per-chunk meta files store observations, and `run_state.json` tracks translation state.
## Routing / Navigation
Not applicable.
## Data Layer
Filesystem-based Markdown, JSON, HTML, and ebook artifacts.
## Persistence
All run state and generated outputs live in `{book_name}_temp/` or a configured temp root.
## Error Handling
Scripts use validation and non-zero exits; metadata failures are quarantined and reported without failing translations.
## Testing
Run `python3 -m unittest discover -s tests -p 'test_*.py' -v` and `python3 -m compileall scripts tests`.
## Build / Deployment Notes
Install the repository under `~/.agents/skills/translate-book`. Output conversion requires Pandoc and Calibre.
