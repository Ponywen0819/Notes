---
name: paper-note
description: Fetch a research paper (arXiv URL/ID or PDF link) and write a detailed, segmented study note into the Obsidian Research/ folder in the vault's #paper style (Traditional Chinese), ending with an "Inspirations" section. Use when the user gives a paper link/ID and asks to write/撰寫 a note (筆記), summarize a paper into the vault, or "接著這篇". Also handles requests to expand an existing paper note.
---

# Paper → Research Note

Turn a paper into a thorough, segmented Obsidian note under `Research/`, matching the vault's existing `#paper` notes.

## Step 1 — Get the real content (don't rely on summaries)

WebFetch runs the page through a small model and only returns a summary; the PDF is usually >10MB and fails. So:

1. **Metadata + abstract**: `WebFetch` the **abs** page (`https://arxiv.org/abs/<id>`) for exact title, authors+affiliations, category, date, verbatim abstract.
2. **Full content**: download the HTML source and read it directly — this gives real formulas, tables, and numbers:
   ```bash
   D=/tmp/papernote
   curl -sL "https://arxiv.org/html/<id>v1" -o "$D.html"
   python3 - "$D.html" "$D.txt" <<'PY'
   import re, html, sys
   src = open(sys.argv[1], encoding='utf-8', errors='ignore').read()
   src = re.sub(r'<(script|style)[^>]*>.*?</\1>', ' ', src, flags=re.S|re.I)
   src = re.sub(r'<math[^>]*alttext="([^"]*)"[^>]*>.*?</math>', r' $\1$ ', src, flags=re.S|re.I)
   src = re.sub(r'</(p|div|li|h1|h2|h3|h4|section|tr|figcaption|caption)>', '\n', src, flags=re.I)
   src = re.sub(r'<br[^>]*>', '\n', src, flags=re.I)
   src = re.sub(r'</td>', ' | ', src, flags=re.I)
   src = re.sub(r'<[^>]+>', ' ', src)
   src = html.unescape(src); src = re.sub(r'[ \t]+', ' ', src)
   src = re.sub(r'\n\s*\n\s*\n+', '\n\n', src)
   open(sys.argv[2], 'w').write(src); print("chars:", len(src))
   PY
   ```
3. **Read `$D.txt` in segments** (Read with offset/limit): ToC first, then intro/method/metrics/experiments/conclusion. Pull the actual tables and numbers — this is the point of grabbing the source.
4. If the HTML 404s (very new/withdrawn), fall back to abs + `arxiv.org/pdf/<id>` via WebFetch and say so in the note.
5. **Clean up**: `rm -f /tmp/papernote.*` when done.

## Step 2 — Write the note

Path: `Research/<Short Title>.md` (or a relevant subfolder like `Research/Social-Network/` for graph papers). Match the existing notes — read one first if unsure (`Research/Agent Laboratory.md` is the reference for depth).

**Language**: Traditional Chinese (繁中), technical terms kept in English inline.

**Skeleton** (adapt/segment to the paper; more detail = more subsections):
```
#paper

[<Title with link>](<abs url>)

> arXiv:<id> ｜ <date> ｜ <category>
> 作者:<authors + affiliations>
> <code/project link if any>

---

# 一、問題與定位
# 二、方法 (Method)          ← break into subsections; formulas in $$…$$; mermaid for pipelines
# 三、實驗 (Experiments)      ← real tables with real numbers; per-dimension analysis
# …
# N-2、限制 (Limitations)
# N-1、重點摘要 (Takeaways)
# N、這篇研究可能的啟發 (Inspirations)   ← ALWAYS include this section
```

**Style rules**:
- Segment heavily. Each hard concept gets its own subsection + a `> 直覺:` blockquote explaining *why*, not just *what*. Worked examples for the trickiest mechanism (label them if invented).
- Reproduce real tables/formulas/numbers from the source — don't paraphrase into vagueness.
- **Always end with the Inspirations section** (`# …、這篇研究可能的啟發`): what it means for future work / the user's own work, grouped by facet, plus open questions.
- **Cross-link** related vault notes with `[[Name]]` or `[[Name|alias]]` — check what exists (`ls Research/**`) and link liberally; unresolved links are fine as placeholders.
- Survey papers: centerpiece is the taxonomy (a table); benchmark papers: task def + metrics + ground-truth construction; method papers: architecture + training + results.

## Step 3 — Report back

Short summary of sections covered + what was skipped (e.g. "PDF too big, used abs+HTML"). Note any newly-created placeholder `[[links]]`.

## Notes
- To *expand* an existing note: read it, keep the good structure, add subsections + worked examples + `> 直覺` blocks. User asking for "詳細一點" = full detail, not ponytail-minimal.
- The `#paper` tag on line 1 is required (vault convention).
