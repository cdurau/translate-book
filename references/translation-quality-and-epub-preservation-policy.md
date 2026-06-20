# Translation Quality and EPUB Preservation Policy

## Contents

- [Purpose and priorities](#purpose-and-priorities)
- [Core translation principles](#core-translation-principles)
- [Glossary and consistency](#glossary-and-consistency)
- [Structural preservation](#structural-preservation)
- [EPUB cover handling](#epub-cover-handling)
- [Title, subtitle, and metadata](#title-subtitle-and-metadata)
- [Visible table of contents](#visible-table-of-contents)
- [EPUB reader navigation](#epub-reader-navigation)
- [Internal links and anchors](#internal-links-and-anchors)
- [Artifact cleanup](#artifact-cleanup)
- [Chunking and subagent rules](#chunking-and-subagent-rules)
- [Output requirements](#output-requirements)
- [Quality control checklist](#quality-control-checklist)
- [Repair mode](#repair-mode)
- [Priority order](#priority-order)
- [Safety and legality](#safety-and-legality)

## Purpose and priorities

When translating books, create a natural, publication-quality translated ebook that preserves the original reading experience. Prioritize:

1. High-quality natural translation.
2. Preservation of the author's tone, intent, rhythm, and stylistic voice.
3. Preservation of the original book structure and formatting.
4. Valid ebook navigation and linked tables of contents.
5. Minimal structural changes unless required for correctness.

## Core translation principles

Translate into fluent, natural, idiomatic target-language prose. Preserve the author's meaning, emotional nuance, tone, rhythm, level of formality, humor, domain-specific style, chapter and section intent, rhetorical emphasis, and recurring terms and concepts.

Do not produce literal word-for-word prose when it sounds unnatural. Prefer a faithful translation that reads as if originally written in the target language.

Do not summarize, shorten, expand, censor, modernize, simplify, or freely rewrite unless explicitly requested. Infer the author's style from surrounding text before translating large sections and maintain it consistently.

Preserve the intended relationship in direct reader address. For German output, use the user's requested style consistently:

- Informal: `du`, `dir`, `dein`
- Formal: `Sie`, `Ihnen`, `Ihr`

If the user does not specify a German address style, infer it from the book type and existing translated title, record the choice, and use it consistently throughout the book.

## Glossary and consistency

Create and maintain a glossary for:

- Character names, place names, and organizations
- Recurring technical, spiritual, or conceptual terms
- Chapter titles and book-specific phrases
- Repeated metaphors and abbreviations
- Named exercises or methods

Keep translations consistent across all chunks and subagents. Do not translate proper names, trademarks, URLs, ISBNs, publisher names, copyright notices, filenames, code snippets, or identifiers unless clearly intended. Prefer consistent translations for recurring phrases unless context requires variation.

## Structural preservation

Preserve the original structure as much as possible. For EPUB input, treat the original EPUB as the source of truth for:

- File and XHTML/HTML structure
- CSS, classes, IDs, and anchors
- Internal links and image references
- Cover references
- Page, chapter, and section breaks
- Reading order
- OPF manifest and spine
- `nav.xhtml`
- `toc.ncx`, if present

Do not flatten an EPUB to plain text or regenerate it from scratch when its structure can be preserved. Do not redesign, merge, split, reorder, or rename files unless required for EPUB validity or explicitly requested.

Translate only human-readable text nodes when possible and preserve surrounding markup exactly. Preserve paragraphs, headings, lists, blockquotes, italics, bold, small caps, alignment, indentation, spacing, scene breaks, footnotes, endnotes, captions, tables, image placement, front matter, back matter, title page, copyright page, dedication, foreword, introduction, acknowledgements, appendices, bibliography, and index.

Do not remove, crop, compress, replace, or redesign images unless explicitly requested.

## EPUB cover handling

For EPUB output, preserve or restore the cover. If the input contains a cover image:

- Keep the original image.
- Preserve or repair its OPF manifest entry.
- Mark it as the cover image where appropriate.
- Ensure ebook readers recognize it as the cover.
- Preserve a visible cover page at the beginning when the original had one; create one only when requested or required to restore the original reading experience.

Do not create or alter a cover image unless explicitly requested.

## Title, subtitle, and metadata

Translate visible title and subtitle text unless requested otherwise. Update the EPUB metadata title and language where appropriate.

Preserve author, publisher, copyright information, ISBN, identifiers, source metadata, rights statements, edition notes, URLs, credits, and legal notices unless validity or an explicit request requires a change. Use natural target-language title style; avoid overly literal German titles.

## Visible table of contents

Every translated ebook should have a clean, linked visible table of contents near the beginning unless explicitly declined.

If one exists, preserve its location, hierarchy, and formatting; translate its entries; remove conversion artifacts; and verify every link.

If none exists, create one after appropriate cover/title/front matter using the detected chapter hierarchy. Match the book's visual style and avoid a generic or disruptive design.

Do not add unwanted bullets. If semantic list markup is appropriate, hide unintended markers with targeted CSS such as `list-style-type: none` without changing links or hierarchy.

## EPUB reader navigation

Every EPUB output must have valid reader navigation:

- For EPUB 3, preserve, create, or repair `nav.xhtml`; use translated labels and valid targets.
- For EPUB 2 or mixed EPUBs, preserve, create, or repair `toc.ncx`; keep hierarchy and `playOrder` valid.
- If both exist, keep them mutually consistent.
- Keep visible TOC labels, navigation labels, and actual headings consistent.

## Internal links and anchors

Preserve existing links and anchors wherever possible. After rebuilding, verify visible TOC links, reader navigation, footnotes, endnotes, and cross-references. No link may point to a missing file or ID.

If a navigation target requires a missing anchor, add a stable ID to the corresponding heading or section without altering layout.

## Artifact cleanup

Remove only artifacts not genuinely present in the book, including:

- `[Chapter]`, `[Kapitel]`, `[Section]`, and `[Abschnitt]`
- Artificial square-bracket labels and placeholder headings
- Duplicate chapter labels or headings introduced by conversion
- Broken Markdown remnants and stray code fences
- Accidental TOC bullets
- Chunk or page-extraction markers
- Unrequested translator notes or AI comments

Perform cleanup contextually. Do not remove genuine content.

## Chunking and subagent rules

Give every translation subagent:

- Relevant glossary
- Target language and requested address style
- Book title and author
- Available surrounding chapter context
- Formatting-preservation requirements
- Instructions to preserve markup and placeholders
- Instructions not to introduce headings, labels, summaries, or structural changes

Subagents must not invent chapter titles, add bracket labels, summarize, or change structure. After merging, run a complete-book consistency pass.

## Output requirements

Produce a valid ebook in the requested format. For EPUB output:

- Preserve or repair cover and metadata.
- Preserve or repair OPF manifest and spine.
- Preserve or repair `nav.xhtml` and `toc.ncx` when present or required.
- Include a linked visible TOC near the beginning unless explicitly declined.
- Preserve images, CSS, and original formatting.
- Validate the EPUB and confirm it opens correctly in available common readers.

Use the requested filename or a clear translated name such as `<original_name>_de.epub` or `<original_name>_translated.epub`.

## Quality control checklist

### Translation quality

- Translation is natural and fluent while preserving style and tone.
- Address style is consistent.
- No accidental untranslated body text remains.
- Proper names, key terms, and chapter titles are consistent.

### Formatting

- Original layout is preserved as closely as possible.
- Paragraphs, headings, lists, blockquotes, emphasis, images, captions, footnotes, and breaks remain intact.
- No unwanted Markdown, conversion artifacts, or artificial labels remain.

### EPUB structure

- EPUB validates successfully.
- OPF manifest and spine are correct.
- Cover image is present, recognized, and visibly represented when appropriate.
- Visible and reader-navigation TOCs exist and are linked.
- `nav.xhtml` and `toc.ncx`, if present, are consistent.
- Internal links and anchors work.
- All images are present.
- The book opens correctly in available common ebook readers.

## Repair mode

When repairing an already translated ebook, do not translate it again unless explicitly requested.

- Treat the original ebook as the source of truth for structure and formatting.
- Treat the translated ebook as the source of truth for translated text.
- Make the smallest possible changes.
- Repair only requested issues.
- Preserve translated content unless it contains obvious artifacts or address-style inconsistencies covered by the request.

Common repairs include restoring the cover page or cover metadata, removing unintended TOC bullets, repairing visible TOC links, rebuilding `nav.xhtml` or `toc.ncx`, removing artificial bracket labels, restoring original formatting, and changing German formal address to informal address when requested.

Do not regenerate a repair target from Markdown unless no safer alternative exists.

## Priority order

1. Preserve original structure and formatting.
2. Preserve author meaning, tone, and style.
3. Produce natural, fluent target-language prose.
4. Preserve or repair cover, metadata, images, links, and navigation.
5. Create or repair a linked visible TOC.
6. Create or repair reader navigation.
7. Remove conversion and translation artifacts.
8. Validate the final ebook.

If priorities conflict, preserve structure and links first, then improve language inside that structure.

## Safety and legality

Translate or modify books only when the user confirms they have the legal right to create the translated copy for the intended use. Do not remove copyright notices, publisher information, licensing text, or attribution. Do not claim a translation is official without authorization.
