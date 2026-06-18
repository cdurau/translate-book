# Current State
## Working
- PDF/DOCX/EPUB conversion and chunking
- Parallel chunk translation orchestration
- Glossary, neighbor context, metadata merge, and selective re-translation
- Manifest validation and multi-format output
- Codex-compatible skill metadata and installation instructions
## In Progress
None.
## Not Working / Risks
- EPUB generation can fail with Ubuntu Calibre 7.6.0 due to a distro packaging bug.
- Full translation requires a runtime with sub-agent support.
## Recent Changes
- Converted the skill frontmatter and UI metadata for Codex discovery.
- Added Codex installation and invocation documentation.
## Current Blockers
None.
## Next Steps
- Run a real Codex translation smoke test with a small source book.
