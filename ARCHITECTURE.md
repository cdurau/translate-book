# Architecture
## Stack
Codex Skill Markdown, Python 3, Pandoc/pypandoc, and Calibre.
## Project Structure
`SKILL.md` defines two processing paths and orchestrates deterministic helpers in `scripts/`; `tests/` covers existing script behavior. `references/translation-quality-and-epub-preservation-policy.md` is the mandatory acceptance contract for translation quality, EPUB preservation, repair, and validation.

- The implemented PDF/DOCX and legacy EPUB path converts through Calibre HTMLZ and Markdown chunks, translates through subagents, and rebuilds output formats from merged Markdown.
- The structure-preserving EPUB path operates on the original package and must retain XHTML structure, CSS, assets, OPF manifest/spine, navigation, covers, anchors, and links while replacing human-readable text nodes.
- Deterministic extraction, reinsertion, link validation, and package rebuilding helpers for the preservation path are not implemented yet. The skill must report this boundary and must not silently fall back to the legacy rebuild.
## State Management
`manifest.json` tracks source chunks, `glossary.json` stores canonical terms, per-chunk meta files store observations, and `run_state.json` tracks translation state.
## Routing / Navigation
The skill routes PDF/DOCX translation to the implemented chunked Markdown pipeline. EPUB translation and repair route to the structure-preserving policy path; legacy EPUB rebuild requires explicit acceptance of its structural limitations.
## Data Layer
Filesystem-based Markdown, JSON, HTML, and ebook artifacts. The legacy path uses `manifest.json`, `glossary.json`, per-chunk metadata, and `run_state.json`; a future preservation implementation must additionally maintain a reversible XHTML text-node map without changing package identifiers or paths.
## Persistence
All run state and generated outputs live in `{book_name}_temp/` or a configured temp root.
## Error Handling
Scripts use validation and non-zero exits; metadata failures are quarantined and reported without failing translations.
## Testing
Run `python3 -m unittest discover -s tests -p 'test_*.py' -v` and `python3 -m compileall scripts tests`.

Current tests cover the Markdown conversion/rebuild helpers. They do not prove original EPUB package preservation; future preservation helpers require fixtures for OPF/spine, EPUB 2/3 navigation, covers, internal links, anchors, CSS/assets, and standards-compliant packaging.
## Build / Deployment Notes
Install the repository under `~/.agents/skills/translate-book`. Legacy output conversion requires Pandoc and Calibre. EPUBCheck is recommended for structure-preserving EPUB validation.
