---
name: "paper-translation-en-cn"
description: "Translate academic papers from English to Chinese markdown while preserving LaTeX formulas, citation numbers, tables, and section structure. Use when the user asks to translate a paper to Chinese, or when Chinese output is needed as input for patent disclosure conversion."
version: 1
created: "2026-05-20"
updated: "2026-05-20"
---
# Academic Paper Translation: English → Chinese Markdown

## When to Use

When the user asks to translate an academic paper (English `.md`) into Chinese, typically as a prerequisite for patent disclosure conversion (`paper-to-patent-conversion` or `paper-to-patent`), or when Chinese output is explicitly requested.

## Procedure

### Step 1: Read the source markdown

Read the full English paper markdown file to understand its structure and content:

```
read(path="paper.md")
```

### Step 2: Translate section by section

Translate the full markdown from English to Chinese. Write the output as `paper_zh.md`:

```
write(path="paper_zh.md", content="...translated markdown...")
```

**Translation rules:**

1. **Section structure**: Preserve the exact heading hierarchy (`#`, `##`, `###`) and section numbering identically.

2. **LaTeX formulas**: Preserve ALL LaTeX formulas exactly as-is — never translate or modify them:
   - Block equations `$$...\tag{N}$$` — keep the LaTeX source identical
   - Inline math `$...$` — keep identical
   - Math symbols like `\mathbf`, `\mathbb{R}`, `\mathcal{L}`, `\sum` — never translate

3. **Citation numbers**: Preserve all `[N]` citation markers exactly — never modify or renumber them. These reference the original bibliography.

4. **Tables**: Translate table headers and cell content to Chinese while preserving the Markdown table structure (pipes, alignment dashes). Keep any numeric data unchanged.

5. **Figures and captions**: Translate figure captions, but preserve figure references like "Figure 1" or "图1" consistently. Use "图N" format in Chinese.

6. **Technical terminology**: 
   - Use standard Chinese academic terms consistently throughout
   - For novel terms the paper introduces (e.g., "Modality Rebalance"), first time: 中文翻译（English Original）, subsequent: 中文翻译 only
   - Model names, dataset names, and proper nouns remain in English (GPT-4o, Omni-UBench, etc.)

7. **Natural academic Chinese style**:
   - Do NOT produce literal/word-by-word translations
   - Use natural Chinese academic prose with appropriate sentence breaks
   - Adapt English long sentences into shorter, clearer Chinese sentences
   - Preserve the logical flow but use Chinese-appropriate connectors (然而, 因此, 此外, etc.)
   - Translate CCS Concepts, Keywords, ACM Reference Format sections fully

8. **Code blocks and special formatting**: Preserve any code blocks, ASCII diagrams, or special formatting identically.

### Step 3: Verify the translation

Run verification checks:

```bash
# Count lines to ensure completeness
wc -l paper.md paper_zh.md

# Verify all formulas preserved
grep -c '\\$\\$' paper.md
grep -c '\\$\\$' paper_zh.md

# Verify citation count
grep -oP '\[\d+(?:,\s*\d+)*\]' paper.md | sort -u | wc -l
grep -oP '\[\d+(?:,\s*\d+)*\]' paper_zh.md | sort -u | wc -l
```

## Verification Checklist

| Aspect | Check |
|---|---|
| Section structure | All headings preserved with same numbering |
| Tables | All tables translated, structure intact |
| LaTeX block equations | Count matches source (grep `$$`) |
| LaTeX inline expressions | Count matches source |
| Citation numbers | All `[N]` preserved, no renumbering |
| Text completeness | Line count approximately proportional |
| Readability | Chinese reads naturally, no awkward literalisms |

## Pitfalls

- **Formula translation**: Never translate mathematical notation inside `$...$` or `$$...$$` — these must be byte-identical to the source.
- **Citation drift**: When translating, citation markers `[N]` can accidentally become `[ N ]` with spaces. Check and fix if needed.
- **Table alignment**: Markdown tables with complex content may misalign after translation. Verify pipe alignment after writing.
- **Section numbering**: Sometimes papers use "1.", "1.1", "1.1.1" — preserve exactly, including any dot/no-dot conventions.
- **HTML entities**: If the source uses HTML entities (e.g., `&amp;`), preserve them.
- **Consistency**: Technical terms must be translated identically across the entire paper. Review for term consistency before finalizing.