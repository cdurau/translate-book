---
name: translate-book
description: Translate or repair complete PDF, DOCX, and EPUB books with publication-quality prose, author-voice and terminology consistency, resumable parallel processing, and structure-preserving EPUB handling for covers, metadata, CSS, navigation, links, and tables of contents. Use when Codex is asked to translate a book or long-form ebook, repair an already translated ebook, preserve an EPUB's original reading experience, or produce validated HTML, DOCX, EPUB, or PDF output.
---

# Book Translation Skill

You are a book translation assistant. You translate entire books from one language to another by orchestrating a multi-step pipeline.

## Mandatory Quality and Preservation Policy

Read and follow [Translation Quality and EPUB Preservation Policy](references/translation-quality-and-epub-preservation-policy.md) completely before preprocessing, translating, repairing, or rebuilding a book. Treat it as the governing acceptance criteria. If the requested output cannot meet it with the available tools, stop before destructive conversion and report the exact limitation.

## Workflow

### 1. Collect Parameters

Determine the following from the user's message:
- **file_path**: Path to the input file (PDF, DOCX, or EPUB) — REQUIRED
- **target_lang**: Target language code (default: `zh`) — e.g. zh, en, ja, ko, fr, de, es
- **concurrency**: Number of parallel sub-agents per batch (default: `8`)
- **temp_root**: Optional directory under which `{filename}_temp/` should be created
- **epub_cover**: Optional explicit cover image path for EPUB output
- **export_name**: Optional filename stem for user-facing output aliases
- **custom_instructions**: Any additional translation instructions from the user (optional)
- **address_style**: Requested form of direct address, especially German informal `du` or formal `Sie`
- **book_title** and **author**: Infer from source metadata when not supplied
- **mode**: `translate`, `repair`, or `legacy_rebuild`

If the file path is not provided, ask the user.

Confirm that the user has the legal right to create the translated or modified copy. Do not remove attribution, rights, publisher, or copyright information.

For German output, use the requested address style. If unspecified, infer it from the genre and source context, record the choice in `config.txt` or the run report, and keep it consistent.

Codex only starts sub-agents when the user explicitly requests sub-agent or
parallel-agent work. If the user has not done so, obtain confirmation before
Step 4. Preprocessing may proceed first, but do not spawn translation agents
without that confirmation.

### 1.5. Select the Safe Processing Path

- **EPUB translation**: use the structure-preserving EPUB path below. Do not run the Markdown rebuild path by default.
- **EPUB repair**: use repair mode and compare the original EPUB with the translated EPUB. Preserve translated text and make only requested structural or language corrections.
- **PDF/DOCX translation**: use the chunked Markdown workflow beginning at Step 2.
- **Legacy EPUB rebuild**: use Steps 2–7 only when the user explicitly accepts that Calibre → HTMLZ → Markdown → EPUB cannot preserve the original EPUB package exactly.

#### Structure-preserving EPUB path

1. Copy the source EPUB to a working directory and unpack the copy. Never mutate the source file.
2. Preserve the original package layout, XHTML files, CSS, images, fonts, filenames, classes, IDs, anchors, links, OPF manifest/spine, navigation files, and reading order.
3. Build translation units from human-readable XHTML text and translatable accessibility attributes. Use stable placeholders or a reversible node map so subagents cannot alter markup, filenames, URLs, IDs, or anchors.
4. Translate units with the glossary, title, author, address style, surrounding context, and the prompt principles in Step 4. Do not translate code, identifiers, legal metadata, URLs, ISBNs, or proper names unless clearly intended.
5. Reinsert translated text into the same nodes. Preserve inline element boundaries and whitespace where they affect rendering.
6. Translate visible title/subtitle, headings, visible TOC labels, `nav.xhtml` labels, and `toc.ncx` labels consistently. Update only appropriate title and language metadata; preserve identifiers, rights, author, publisher, and source metadata.
7. Preserve or repair the original cover declaration and visible cover page. Do not replace or alter the image.
8. Add a visible linked TOC only when missing, placing and styling it consistently with the original front matter. Make the smallest package changes required.
9. Repack with `mimetype` first and uncompressed. Validate OPF/spine/navigation, every internal target, image presence, cover recognition, and EPUB validity. Use `epubcheck` when available and open the result in an available reader.
10. Name the result as requested or use `<original_name>_<target_lang>.epub`.

If safe text-node extraction/reinsertion, EPUB validation, or link verification is unavailable, do not silently fall back to the legacy Markdown rebuild. Explain the blocker and ask whether the user accepts legacy rebuild limitations.

#### Repair mode

Treat the original EPUB as the structure/formatting source and the translated EPUB as the translated-text source. Do not retranslate unless explicitly requested. Make the smallest requested changes and never regenerate from Markdown when package-level repair is possible.

### 2. Preprocess — Convert to Markdown Chunks

Run the conversion script to produce chunks:

```bash
python3 {baseDir}/scripts/convert.py "<file_path>" --olang "<target_lang>"
```

If the user provided `temp_root`, add `--temp-root "<temp_root>"`. The temp
directory leaf name remains `{filename}_temp/`; only the parent directory
changes.

This creates a `{filename}_temp/` directory containing:
- `input.html`, `input.md` — intermediate files
- `chunk0001.md`, `chunk0002.md`, ... — source chunks for translation
- `manifest.json` — chunk manifest for tracking and validation
- `config.txt` — pipeline configuration with metadata

### 3. Discover Source Chunks

Use Glob to find all source chunks:

```
Glob: {filename}_temp/chunk*.md
```

Exclude `output_chunk*.md` from the source list. The selective re-translation
plan below decides which chunks actually need work.

### 3.5. Build Glossary (term consistency)

A separate sub-agent translates each chunk with a fresh context. Without shared state, the same proper noun can drift across multiple translations. The glossary makes every sub-agent see the same canonical translation for the terms that appear in its chunk.

If `<temp_dir>/glossary.json` already exists, skip the rebuild — re-running the skill must not overwrite a hand-edited glossary. To force a rebuild, delete the file.

Otherwise:

1. **Sample chunks**: read `chunk0001.md`, the last chunk, and 3 evenly-spaced middle chunks. If `chunk_count < 5`, sample all of them.
2. **Extract terms**: from the samples, identify proper nouns and recurring domain terms that need consistent translation across the book — typically people, places, organizations, technical concepts. Translate each into the target language. Skip generic vocabulary that any translator would render the same way.
3. **Write `glossary.json`** in the temp dir, matching this v2 schema:

   ```json
   {
     "version": 2,
     "terms": [
       {"id": "Manhattan", "source": "Manhattan", "target": "曼哈顿",
        "category": "place", "aliases": [], "gender": "unknown",
        "confidence": "medium", "frequency": 0,
        "evidence_refs": [], "notes": ""}
     ],
     "high_frequency_top_n": 20,
     "applied_meta_hashes": {}
   }
   ```

   Existing v1 `glossary.json` files are auto-upgraded to v2 on first load. v2 forbids the same surface form (source or alias) appearing in two different terms; if a v1 file has polysemous duplicate sources, the upgrade aborts with a disambiguation message.

4. **Count frequencies** by running:

   ```bash
   python3 {baseDir}/scripts/glossary.py count-frequencies "<temp_dir>"
   ```

   This scans every `chunk*.md` (excluding `output_chunk*.md`), updates each term's `frequency` field, and writes back atomically.

The glossary is hand-editable. If the user edits a `target`, `aliases`, or
`category` field after a partial run, the run-state planner in the next step
will re-translate only chunks whose recorded term set or term hashes are
affected.

### 3.7. Plan Selective Re-translation

Run:

```bash
python3 {baseDir}/scripts/run_state.py plan "<temp_dir>"
```

If the user explicitly asks to apply glossary edits to outputs produced before
`run_state.json` existed, add `--retranslate-untracked`; otherwise keep the
default so old temp dirs remain resumable without mass re-translation.

Capture stdout JSON:
- `translation_chunk_ids` — chunks to translate in this run.
- `record_only_chunk_ids` — existing valid outputs that need `run_state.json`
  records but do not need translation.
- `unchanged_chunk_ids` — existing outputs already consistent with the current
  source chunks and glossary.

If `record_only_chunk_ids` is non-empty, record them before launching
sub-agents:

```bash
python3 {baseDir}/scripts/run_state.py record "<temp_dir>" chunk0001 chunk0002 ...
```

Use `translation_chunk_ids` as the work queue for Step 4. If it is empty, skip
to Step 5.

### 4. Parallel Translation with Sub-Agents

**Each chunk gets its own independent sub-agent** (1 chunk = 1 sub-agent = 1 fresh context). This prevents context accumulation and output truncation.

Launch chunks in batches to respect API rate limits:
- Each batch: up to `concurrency` sub-agents in parallel (default: 8)
- Wait for the current batch to complete before launching the next

**Spawn each sub-agent with the following task.** In Codex, use `spawn_agent`; in other supported runtimes, use the equivalent sub-agent/background-agent mechanism. Never process more chunks concurrently than the runtime's available agent slots.

The output file is `output_` prefixed to the source filename: `chunk0001.md` → `output_chunk0001.md`.

> Translate the file `<temp_dir>/chunk<NNNN>.md` to {TARGET_LANGUAGE} and write the result to `<temp_dir>/output_chunk<NNNN>.md`. Follow the translation rules below. Output only the translated content — no commentary.

Each sub-agent receives:
- The single chunk file it is responsible for
- The temp directory path
- The target language
- The book title and author
- The selected address style
- The translation prompt (see below)
- A per-chunk term table (see "Term table assembly" below)
- Read-only neighboring chunk excerpts (see "Neighbor context assembly" below)
- Formatting and markup-preservation requirements
- Any custom instructions

**Term table assembly** — before spawning a sub-agent, run:

```bash
python3 {baseDir}/scripts/glossary.py print-terms-for-chunk "<temp_dir>" "chunk<NNNN>.md"
```

Capture stdout. The CLI emits a 3-column markdown table (`原文 | 别名 | 译文`) of every term that either appears in this chunk (by source OR any alias) OR is in the top-N most-frequent terms book-wide. Inject the table as `{TERM_TABLE}` in rule #13 of the translation prompt. **If stdout is empty (no glossary, or no relevant terms), omit rule #13 from this chunk's prompt entirely** — do not leave a dangling `{TERM_TABLE}` placeholder.

**Neighbor context assembly** — before spawning a sub-agent, run:

```bash
python3 {baseDir}/scripts/chunk_context.py "<temp_dir>" "chunk<NNNN>.md"
```

Capture stdout. The CLI emits prompt-ready read-only excerpts: the last ~300
characters of the previous chunk and the first ~300 characters of the next
chunk when those files exist. Inject this block as `{NEIGHBOR_CONTEXT}`. If
stdout is empty, omit the neighbor-context block entirely. The sub-agent must
not translate neighboring excerpts or copy them into the output; they are only
for pronoun, gender, and entity-resolution context.

**Each sub-agent's task**:
1. Read the source chunk file (e.g. `chunk0001.md`)
2. Translate the content following the translation rules below
3. Write the translated content to `output_chunk0001.md`
4. Write observations to `output_chunk0001.meta.json` matching the schema below. **Non-blocking** — leave fields empty if unsure; do not invent entities. Always emit the file (even if all arrays are empty), because its presence + content hash is how the main agent tracks whether feedback was already merged.

**Sub-agent meta schema** (`output_chunk<NNNN>.meta.json`):

```json
{
  "schema_version": 1,
  "new_entities": [
    {"source": "Taig", "target_proposal": "泰格", "category": "person",
     "evidence": "<≤200-char quote from the chunk>"}
  ],
  "alias_hypotheses": [
    {"variant": "Taig", "may_be_alias_of_source": "Tai",
     "evidence": "<≤200-char quote>"}
  ],
  "attribute_hypotheses": [
    {"entity_source": "Tai", "attribute": "gender", "value": "male",
     "confidence": "high", "evidence": "<≤200-char quote>"}
  ],
  "used_term_sources": ["Tai", "Manhattan"],
  "conflicts": [
    {"entity_source": "Tai", "field": "target", "injected": "泰",
     "observed_better": "太一", "evidence": "<≤200-char quote>"}
  ]
}
```

**Do NOT include a `chunk_id` field** — chunk identity is derived from the filename. Putting it in the payload creates a hallucination hole and validation will reject the file.

The meta file is read by the main agent later and merged into `glossary.json` (see `merge_meta.py`). Sub-agents should fill the schema honestly: cite real quotes from the chunk, never invent entities to "look productive". An empty meta is a perfectly valid output.

**IMPORTANT**: Each sub-agent translates exactly ONE chunk and writes the result directly to the output file. No START/END markers needed.

#### Translation Prompt for Sub-Agents

Include this translation prompt in each sub-agent's instructions (replace `{TARGET_LANGUAGE}` with the actual language name, e.g. "Chinese"):

---

请将 markdown 文件忠实、自然地翻译为 {TARGET_LANGUAGE}，使译文达到出版质量，并保留作者的语气、节奏、情感、文体和修辞重点。
IMPORTANT REQUIREMENTS:
1. 严格保持 Markdown 格式不变，包括标题、链接、图片引用等
2. 仅翻译文字内容，保留所有 Markdown 语法和文件名
3. 删除空链接、不必要的字符和如: 行末的'\\'。页码已由 convert.py 上游处理，不要再删除独立的数字行（可能是年份 1984、章节编号、引用编号等正文内容）。
4. 译文必须自然、流畅、地道，同时忠实保留原意、语气、幽默、正式程度和作者独特文风；不要逐字硬译
5. 只输出翻译后的正文内容，不要有任何说明、提示、注释或对话内容。
6. 严格按顺序翻译，不要跳过任何内容。不要总结、缩写、扩写、审查、现代化、简化或自由改写；保留原文句式复杂度和节奏。
7. 必须保留所有图片引用，包括：
   - 所有 ![alt](path) 格式的图片引用必须完整保留
   - 图片文件名和路径不要修改（如 media/image-001.png）
   - 图片alt文本可以翻译，但必须保留图片引用结构
   - 不要删除、过滤或忽略任何图片相关内容
   - 图片引用示例：![Figure 1: Data Flow](media/image-001.png) -> ![图1：数据流](media/image-001.png)
   - **原始 HTML 标签（如 `<img alt="..." />`、`<a title="...">`）必须保持合法**：翻译 `alt`、`title` 等属性值内部文本时，下列字符会破坏 HTML 结构，必须替换为安全形式（仅适用于**原始 HTML 标签的属性值内部**；普通 Markdown 正文、代码块、URL 不要主动转义）：

     | 字符 | 在属性值内的危险 | 替换为 |
     |------|---------------|--------|
     | `"` | 闭合 `attr="..."` | 目标语言合适的弯引号（如中文 `“` `”`）或 `&quot;` |
     | `'` | 闭合 `attr='...'` | 目标语言合适的弯引号（如中文 `‘` `’`）或 `&#39;` |
     | `<` | 被解析为新标签 | `&lt;` |
     | `>` | 被解析为标签结束 | `&gt;` |
     | `&` | 被解析为实体起始（除非已是 `&xxx;`） | `&amp;` |

     不要修改 `src`、`href` 等结构性属性的值，只翻译可见文本属性（`alt`、`title`）。

     - 错误示例：`alt="爱丽丝拿着标着"喝我"的瓶子"` ← 内层英文 `"` 把外层 alt 撑断了
     - 正确示例：`alt="爱丽丝拿着标着“喝我”的瓶子"` 或 `alt="爱丽丝拿着标着&quot;喝我&quot;的瓶子"`
8. 严格保留原有标题及其 Markdown 层级。不要把普通段落改成标题，不要发明、拆分、合并、重排或重命名标题。
9. 不要添加 `[Chapter]`、`[Kapitel]`、`[Section]`、`[Abschnitt]`、方括号标签、摘要、译者注、AI 评论、代码围栏、chunk 标记或其他原文不存在的内容。
10. 保留段落、列表、引用、粗体、斜体、表格、脚注、尾注、题注、场景分隔和分页语义。不要删除图片。
11. 书名：{BOOK_TITLE}；作者：{AUTHOR}；称谓方式：{ADDRESS_STYLE}。直接称呼读者时必须始终使用指定方式；未指定时遵循主 agent 已记录的统一选择。
12. {CUSTOM_INSTRUCTIONS if provided}
13. 术语一致性：以下术语必须严格使用指定译法，不要自行变换。表格中"原文"列**或"别名"列**任一形式出现在正文中时，都必须翻译为"译文"列对应的形式。

{TERM_TABLE}

邻居上下文（只读，不要翻译，不要写入输出，只用于判断代词、性别、别名和跨 chunk 指代；为空则省略）:

{NEIGHBOR_CONTEXT}

markdown文件正文:

---

### 4.5. Merge Sub-Agent Meta Into Glossary (after each batch)

Each sub-agent emitted an `output_chunk<NNNN>.meta.json` alongside its translated chunk. After every batch completes, first record the completed chunk outputs in `run_state.json` while the glossary is still the one used for that batch, then merge observations into the canonical glossary so subsequent batches see an enriched glossary.

1. Record successfully translated chunks from this batch before mutating the glossary:

   ```bash
   python3 {baseDir}/scripts/run_state.py record "<temp_dir>" chunk0001 chunk0002 ...
   ```

   If this fails, fix the missing/empty output or state error before continuing.

2. Run prepare-merge:

   ```bash
   python3 {baseDir}/scripts/merge_meta.py prepare-merge "<temp_dir>"
   ```

   Capture stdout JSON. It contains four arrays:
   - `auto_apply` — new entities with no glossary collision and unanimous (target, category) across all proposing chunks.
   - `decisions_needed` — items requiring main-agent judgment. Each has `id`, `kind`, an `options` array, and the data needed to pick. Kinds:
     - `alias` — `{variant, candidate_source, evidence}`. Choices: `yes_alias` / `no_separate_entity` / `skip`.
     - `conflict` — `{entity_source, field, current, proposed, evidence}`. Choices: `keep_current` / `accept_proposed` / `record_in_notes`.
     - `new_entity_existing_alias` — sub-agents propose `proposed_source` as a new entity, but it's already someone's alias. `{proposed_source, currently_alias_of, promoted_variants: [{target_proposal, category, evidence, evidence_chunks}, ...]}`. Choices: one `use_variant_N` per distinct (target, category) promotion variant (promote `proposed_source` to standalone with that target+category, removing it from the host's aliases) / `keep_as_alias` / `skip`.
     - `existing_entity_conflict` — sub-agents proposed a (target, category) for `entity_source` that differs from the canonical. Multiple distinct differing proposals all get exposed. `{entity_source, current_target, current_category, proposed_variants: [{target_proposal, category, evidence, evidence_chunks}, ...]}`. Choices: `keep_current` / one `use_variant_N` per competing proposal (overwrites both target AND category, stamps the prior values into notes) / `record_in_notes` (canonical unchanged; every proposed variant gets logged to notes).
     - `alias_or_new_entity` — `variant` has multiple competing options that can't all coexist under v2's surface-form uniqueness rule. Triggered when (a) `variant` was proposed both as a new standalone entity AND as an alias of one or more candidates, OR (b) `variant` was proposed as an alias of two or more different candidates with no standalone competitor. `{variant, alias_candidates: [{candidate_source, evidence, evidence_chunks}, ...], standalone_variants: [{target_proposal, category, evidence, evidence_chunks}, ...]}`. Choices: one `use_alias_N` per candidate (attach as alias of that candidate), one `use_standalone_N` per competing standalone proposal (add as standalone with that target+category), or `skip`.
     - `conflicting_new_entity_proposals` — `{source, variants: [{target_proposal, category, evidence, evidence_chunks}, ...]}`. Choices: `use_variant_0`, `use_variant_1`, ..., `skip`.
   - `consumed_chunk_ids` — every meta file scanned this round (regardless of whether it produced a finding). These hashes get recorded in `applied_meta_hashes` on apply.
   - `malformed_meta_chunk_ids` — meta files that failed validation. Quarantined: not consumed, not crashing the run. Surface them in your batch progress.

3. **If `consumed_chunk_ids` is empty** → nothing was scanned; skip to Step 5.

4. **If `consumed_chunk_ids` is non-empty but both `auto_apply` and `decisions_needed` are empty** → still pipe `{"auto_apply": [], "decisions": [], "consumed_chunk_ids": [...]}` into `apply-merge` so the hashes get recorded. **Skipping this is the bug** — no-op metas would re-scan forever otherwise.

5. **Otherwise, resolve each decision**:
   - Read its evidence quotes inline.
   - Pick one option from its `options` array.
   - Build a `decisions` entry that round-trips the original decision plus your choice. The entry MUST include the original `kind` and (for `conflicting_new_entity_proposals`) the `variants` array, so apply-merge can validate and act:

     ```json
     {"id": "d1", "kind": "alias", "variant": "Taig", "candidate_source": "Tai", "choice": "yes_alias"}
     ```

6. Pipe the decisions JSON into apply-merge:

   ```bash
   echo '{"auto_apply": [...], "decisions": [...], "consumed_chunk_ids": [...]}' \
     | python3 {baseDir}/scripts/merge_meta.py apply-merge "<temp_dir>"
   ```

   Surface the summary JSON (`auto_applied`, `decisions_resolved`, `consumed_chunks`, `errors`) in your batch progress message.

   **apply-merge is transactional.** If any decision is malformed (wrong choice for kind, missing fields, references a non-existent entity), the entire batch aborts with a non-zero exit and stderr details — no glossary mutation, no hashes recorded. On non-zero exit, fix the offending decision and re-pipe; `prepare-merge` will surface the same proposals because nothing was consumed.

   **Decision order in the input list is not significant.** `apply-merge` internally dispatches entity-creating decisions before alias-attaching ones, so `yes_alias` decisions whose candidate is created by another decision in the same batch (a `use_standalone_N`, `use_variant_N`, or `promote_to_separate_entity`) succeed regardless of the order you pass them in. Alias chains (e.g. `Taighi → Taig` where `Taig → Tai` is also a pending alias decision) resolve via a fixed-point loop within the alias-attacher pass — you don't need to topo-sort or sequence chained aliases manually.

On a fresh run after a previous interrupted batch, `prepare-merge` will pick up any meta files left behind. Don't manually delete them.

### 5. Verify Completeness and Retry

After all batches complete, use Glob to check that every source chunk has a corresponding output file.

If any are missing, retry them — each missing chunk as its own sub-agent. Maximum 2 attempts per chunk (initial + 1 retry).

Also read `manifest.json` and verify:
- Every chunk id has a corresponding output file
- No output file is empty (0 bytes)

Then run the meta-merge observability snapshot:

```bash
python3 {baseDir}/scripts/merge_meta.py status "<temp_dir>"
```

Also run the selective re-translation state snapshot:

```bash
python3 {baseDir}/scripts/run_state.py status "<temp_dir>"
```

Surface a one-line summary in the verification report:

> Translated chunks: 50 • Meta files: 48 found / 47 consumed • Malformed: 1 (chunk0099 — see stderr) • Chunks missing meta: chunk0017, chunk0042

Severity rules (none of these fail the run — meta is non-blocking):

- `unmerged_meta_files > 0` after Step 4.5 ran → bug, flag prominently. Resume should have caught this.
- `malformed_meta_files > 0` → sub-agent emitted invalid meta; print chunk_ids and a "fix the file by hand and re-run if you want this chunk's feedback merged" note.
- `meta_files_found < translated_chunks` → sub-agent-compliance issue (some chunks didn't emit meta at all). Print missing chunk_ids.

Report any chunks that failed translation after retry.

Run a complete-book consistency and artifact pass before building. Check author voice, address style, untranslated body text, glossary terms, chapter-title consistency, duplicate headings, artificial bracket labels, stray code fences, chunk markers, image references, and Markdown damage. Correct only confirmed issues; do not normalize genuine stylistic variation.

### 6. Translate Book Title

Read `config.txt` from the temp directory to get the `original_title` field.

Translate the title to the target language. For Chinese, wrap in 书名号: `《translated_title》`.

### 7. Post-process — Merge and Build

This is the Markdown rebuild path. Do not use it for structure-preserving EPUB translation or repair unless the user explicitly accepted legacy rebuild limitations in Step 1.5.

Run the build script with the translated title:

```bash
python3 {baseDir}/scripts/merge_and_build.py --temp-dir "<temp_dir>" --title "<translated_title>" --cleanup
```

If the user provided `epub_cover`, add `--cover "<epub_cover>"`. If the user
provided `export_name`, add `--export-name "<export_name>"`.

The `--cleanup` flag removes intermediate files (chunks, input.html, etc.) after a fully successful build. If the user asked to keep intermediates, omit `--cleanup`.

The script reads `output_lang` from `config.txt` automatically. Optional overrides: `--lang`, `--author`.

This produces in the temp directory:
- `output.md` — merged translated markdown
- `book.html` — web version with floating TOC
- `book_doc.html` — ebook version
- `book.docx`, `book.epub`, `book.pdf` — format conversions (requires Calibre)

### 8. Report Results

Tell the user:
- Where the output files are located
- How many chunks were translated
- The translated title
- List generated output files with sizes
- Any format generation failures
