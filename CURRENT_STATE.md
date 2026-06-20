# Current State
## Working
- PDF/DOCX/EPUB conversion and chunking
- Parallel chunk translation orchestration
- Glossary, neighbor context, metadata merge, and selective re-translation
- Manifest validation and multi-format output
- Codex-compatible skill metadata and installation instructions
- Mandatory publication-quality translation and EPUB preservation policy
- Explicit structure-preserving EPUB translation and minimal-change repair instructions
## In Progress
None.
## Not Working / Risks
- EPUB generation can fail with Ubuntu Calibre 7.6.0 due to a distro packaging bug.
- Full translation requires a runtime with sub-agent support.
- The structure-preserving EPUB path is specified but does not yet have deterministic extraction/reinsertion and validation helpers; the existing scripts remain a lossy legacy rebuild path.
## Recent Changes
- Converted the skill frontmatter and UI metadata for Codex discovery.
- Added Codex installation and invocation documentation.
- Added a mandatory translation-quality policy, EPUB preservation/repair routing, address-style consistency, and a stricter subagent translation prompt.
## Current Blockers
None.
## Next Steps
- Run a real Codex translation smoke test with a small source book.
- Implement deterministic XHTML text-node extraction/reinsertion and EPUB package validation helpers; the current scripts still provide only the legacy Markdown rebuild path.
