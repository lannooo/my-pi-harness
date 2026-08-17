---
name: "pdf-to-markdown-with-math"
description: "Extract academic PDFs to markdown and fix LaTeX math formulas so they render properly. Use when the user wants to extract a paper/conference article to .md, especially when formulas are involved."
version: 7
created: "2026-05-18"
updated: "2026-05-20"
---
## When to Use

When the user asks to extract a PDF (especially academic papers with math formulas) to markdown format.

## Procedure

### Step 1: Parse the PDF

Use `document_parse` with `format="text"` to extract content:

```
document_parse(path="paper.pdf", format="text")
```

The output is saved to a temp file at `/var/folders/.../pi-document-parse-*/parsed.txt`. Note the exact path from the result output.

### Step 2: Read the parsed output

Use `read` to get the full parsed content from the temp file:

```
read(path="/var/folders/.../pi-document-parse-*/parsed.txt")
```

This avoids truncation issues that can occur with `cat` or `bash` on large files.

### Step 3: Manually structure as Markdown

The raw parsed output needs manual reorganization. Do NOT just copy-paste:

- **Headings**: Rebuild the heading hierarchy (`#`, `##`, `###`) based on the paper's section structure
- **Tables**: Reformat tabular data as proper Markdown tables
- **Lists**: Convert plain-text enumerations to `-` or `1.` list items
- **References**: Preserve original citation numbers `[N]` exactly as they appear — do not modify or renumber them

Save the structured markdown with `write`:

```
write(path="paper.md", content="...structured markdown...")
```

### Step 4: Fix LaTeX Formulas

`document_parse` extracts formulas as plain ASCII in code blocks (` ``` `), which won't render. They need proper LaTeX delimiters:

**Block equations** (numbered, multi-line): Replace code blocks with `$$...$$`:

```latex
$$\mathbf{x}' = \mathcal{R}_x(...) = [...], \tag{2}$$
```

**Inline math**: Replace code blocks or bold markers with `$...$`:

```latex
probability $p_i$ and dimension $D_c$
```

**Common math symbols to use:**

- `\mathbf{x}` for bold vectors
- `\mathbb{R}`, `\mathbb{I}` for number sets
- `\mathcal{L}` for calligraphic (loss functions, etc.)
- `\sum` for summation
- `\sim` for "distributed as"
- `\in` for set membership
- `\ll` for "much less than"

**Procedure for fixing formulas:**

1. `grep -n '```' paper.md` to find formula locations
2. Use `edit` with targeted replacements — each `oldText` must be unique and non-overlapping
3. Prioritize block equations first (they're larger, more likely to overlap with inline ones)
4. After edits, verify with `grep -n '\\$\\$' paper.md` and `grep -n '\\$[^$]' paper.md` to check for proper LaTeX delimiters

### Step 5: Translate to Chinese (Optional)

If the user asks for Chinese translation, follow the `paper-translation-en-cn` skill procedure. Key points:

1. **Do NOT use bash translation tools** (e.g., `trans`, `translate-shell`). These produce literal translations that break LaTeX formulas and produce awkward academic prose.
2. **Manually translate the full markdown** using `write`:

   - Preserve all LaTeX formulas exactly (block `$$...$$` and inline `$...$`)
   - Preserve all citation numbers `[N]` unchanged
   - Translate table headers and cell content while keeping structure
   - Use natural Chinese academic style, not literal translation
   - Model/dataset names remain in English
3. Save as `paper_zh.md` and verify with `wc -l` and `grep` counts for formulas/citations.

## Verification

- Open the .md in a LaTeX-aware viewer (VS Code, Typora, Obsidian) and confirm all formulas render as math
- Verify equation numbers are preserved
- Confirm no remaining code blocks contain formulas

## Pitfalls

- **Overlapping edits**: If two changes touch the same block, merge them into one edit. If an edit fails with "Could not find exact text", it was likely already fixed — skip it.
- **Formula detection**: Some formulas may appear as bold markdown (`**...**`) instead of code blocks. Check both.
- **Numbered equations**: Preserve equation numbers with `\tag{1}`, `\tag{2}`, etc. inside `$$...$$`.
- **Citation preservation**: Never modify or renumber citation markers like `[1]`, `[23]` — they reference the original paper's bibliography.
- **Temp file path**: Parsed output goes to `/var/folders/.../pi-document-parse-*/parsed.txt`. Use `read` (not `cat` via bash) to avoid large-file truncation.
- **Manual structuring required**: The raw `document_parse` output flattens structure. You must rebuild headings, tables, and lists manually — do not ship the raw output as-is.
- **Dual-extraction comparison**: For important papers, use `markdown-converter` (markitdown) for a quick first-pass raw dump as `paper_copilot.md`. This raw version preserves ALL content including page artifacts and is useful for cross-referencing completeness against the clean `document_parse` version. The clean version (`paper.md`) will be substantially shorter — this is expected (page headers/footers/copyright text are stripped).

## Step 5: Optional — Translate to Chinese

If the user asks for Chinese translation, use the dedicated `paper-translation-en-cn` skill which provides a complete, verified translation workflow with LaTeX/math preservation, citation integrity checks, and consistency rules.

For a quick summary:

1. Read the full English `paper.md`
2. Translate to Chinese following the `paper-translation-en-cn` skill rules (preserve formulas, citations, tables; use natural academic Chinese)
3. Write as `paper_zh.md`
4. Verify line counts and formula/citation counts match
