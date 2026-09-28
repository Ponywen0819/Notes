---
name: paper-note
description: Parse a research paper from any source (arXiv URL/ID, local PDF, PDF link, or journal/publisher/blog web page) and write a detailed, segmented, easy-to-read study note into the Obsidian Research/ folder in the vault's #paper style (Traditional Chinese), with real tables/numbers, a Limitations section, and an "Inspirations" ending. Use when the user gives a paper and asks to write/撰寫/整理 a note (筆記), summarize its methodology (方法論) or limitations (研究限制), "接著這篇", or asks to expand/reformat an existing paper note.
---

# Paper → Research Note

Turn a paper into a thorough, segmented Obsidian note under `Research/`, matching the vault's existing `#paper` notes. Content must come from the paper's real text; layout must be easy to scan (see Step 3 — the user cares about this a lot).

## Step 1 — Get the real content (don't rely on summaries)

WebFetch only returns a small-model summary and big PDFs fail through it, so never write the note from WebFetch output alone. Pick the route by source, in this order:

| Source | Route |
|---|---|
| arXiv (URL, ID, or a local PDF whose filename is an arXiv ID like `2210.03629v3.pdf`) | **A. arXiv HTML** (best quality for formulas and tables) |
| Local PDF, or a downloadable PDF link (journal, ACL Anthology, OpenReview, PMC, etc.) | **B. PDF** |
| Web page with the full text in HTML (publisher page, PMC article, tech blog, report) | **C. Generic HTML** |
| Nothing above works (paywall, no full text) | **D. Fallback**: abstract page via WebFetch; state in the note header that it's based on the abstract only |

If a non-arXiv PDF turns out to also be on arXiv (search the title), prefer route A.

### A. arXiv HTML

1. **Find the arXiv ID + version** from the URL or filename (e.g. `2210.03629v3.pdf` → `2210.03629`, `v3`). Use that version, not a hardcoded `v1`.
2. **Full content**: download the HTML source and strip it to text. This gives real formulas, tables, and numbers. Use a unique temp name in the scratchpad (or `/tmp/papernote_<slug>`):
   ```bash
   D=/tmp/papernote_<slug>
   curl -sL "https://arxiv.org/html/<id><ver>" -o "$D.html"
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
   The HTML header already has title, authors, affiliations, category and date (`arXiv:<id><ver> [cs.XX] <date>`), so a separate abs-page WebFetch is only needed if that's missing.
3. **Read `$D.txt` in segments** (Read with offset/limit): ToC first, then abstract → intro → method → setup → results → related work → conclusion. Then `grep -n` for appendix headings and read the ones that matter (extra results, failure analysis, finetuning details). Skip the raw prompt/trajectory dumps unless needed for a worked example.
4. If the HTML 404s (very new or withdrawn paper), switch to route B with the arXiv PDF (`https://arxiv.org/pdf/<id><ver>`).

### B. PDF

1. If it's a URL, download it: `curl -sL "<url>" -o /tmp/papernote_<slug>.pdf`. Check that it's really a PDF (`file <path>`). If you got an HTML login or paywall page instead, go to route C or D.
2. Read it with the **Read tool** using `pages:` in chunks of at most 20 pages. Start with pages 1–3 for the title, authors, venue, and abstract, then read the method, experiments, and conclusion. Read appendix pages as needed.
3. If `pdftotext` happens to be installed (`which pdftotext`), you can run `pdftotext -layout <pdf> <txt>` to grep the text quickly. Treat that output as a search aid only: its tables and formulas are often garbled.
4. **Take tables and formulas from the rendered pages** that the Read tool shows, not from extracted text. Check every number you copy against the page.
5. Metadata usually isn't in a clean header here. Get the venue, year, and DOI from the first page, running headers, or the citation footer. If they're missing, leave the field blank rather than guess.

### C. Generic HTML (non-arXiv)

1. Run `curl -sL "<url>"` and apply the same Python stripper from route A. The `<math alttext>` rule is harmless when a page has no math.
2. If the page is mostly JavaScript and the text comes out nearly empty, look for a "PDF" or "Download" link and use route B.
3. Get metadata from `<meta name="citation_*">` tags if the page has them: `grep -o 'citation_[a-z_]*" content="[^"]*"'`. Most publisher pages do.

### All routes

- Read the text in segments. Start with the ToC or section headings, then go through abstract → method → results → conclusion.
- **Clean up** the temp files when done.

## Step 2 — Write the note

**Path**: `Research/<Short Title>.md`, or the topic subfolder the user names (e.g. `Research/Agent framework/`, `Research/Social-Network/`). Read one existing note first if unsure (`Research/Agent Laboratory.md` is the reference for depth).

**Language**: Traditional Chinese (繁中), technical terms kept in English inline.

**Skeleton** (adapt to the paper; more detail = more subsections):
```
#paper

[<Title>](<canonical url: arXiv abs / DOI link / publisher page>)

> <arXiv:<id><ver> | 期刊或會議名稱(卷期)> ｜ <date or year> ｜ <category or field, e.g. cs.CL / 生醫影像>
> 作者:<authors + affiliations>
> <DOI / code / project link if any>
> <route B/D only: 「來源:本機 PDF」或「僅根據摘要撰寫」>

---

# 一、問題與定位
# 二、方法 (Method)            ← formal definition, core mechanism, subsections, mermaid for loops/pipelines
# 三、任務設定與方法細節        ← datasets, action/input space, baselines, training/prompting details
# 四、實驗結果 (Results)        ← real tables with real numbers + per-finding analysis
# …(ablations, extra experiments from the appendix)
# N-2、限制 (Limitations)
# N-1、重點摘要 (Takeaways)
# N、這篇研究可能的啟發 (Inspirations)   ← ALWAYS include
```

**Content rules**:
- Each hard concept gets its own subsection + a `> 直覺:` blockquote explaining *why*, not just *what*. Worked examples for the trickiest mechanism (label invented ones as invented; prefer real examples from the appendix).
- Reproduce real tables/formulas/numbers — don't paraphrase into vagueness.
- **Methodology** = formal setup + what the method changes + how it is instantiated (prompting / finetuning / heuristics, with the actual thresholds and counts) + baselines built by ablation.
- **Limitations**: many papers have no dedicated section. Harvest them from the error analysis, footnotes, conclusion, ethics statement, reproducibility statement, and appendix. Give each one a concrete number or source (e.g. "23% of failures", "(Ethics Statement)"). Say explicitly when the paper has no standalone Limitations section.
- **Inspirations**: grouped by facet (design / evaluation / open questions / related notes), tied to the user's folder context (e.g. "this is the starting point for the Agent framework series").
- **Cross-link** related vault notes with `[[Name]]` — check what exists (`ls Research/**`); unresolved links are fine as placeholders.
- **Internal references** to another section of the same note must be clickable heading links, not bare `§N` text: `[[#<exact heading text without leading #s>|§四.1]]`. When citing a *paper* section, name it as the paper's (`論文 §3.3`) and link the note section that covers it.
- Survey → taxonomy is the centerpiece; benchmark → task def + metrics + ground truth; method → architecture + training + results.

## Step 3 — Readability rules (user feedback, apply every time)

The user rejects "擠成一坨" text. Leave whitespace and keep one idea per block.

- **Formulas**: never string several inline formulas through one prose sentence. Put each key formula on its own `$$…$$` line, with a short lead-in sentence before it and a blank line around it. Inline `$x$` is only for naming a symbol.
- **Paragraphs**: one idea per paragraph, with a blank line between. A "claim + 但/because detail" sentence becomes two paragraphs. An intro sentence ending in `:` gets a blank line before the list that follows.
- **Items with explanations** (limitations, key findings, properties): use a bold title on its own line, a blank line, then the explanation indented under it. Leave a blank line between items.
  ```
  1. **檢索品質是硬瓶頸**

     23% 的失敗來自搜尋結果空白或無關資訊,……
  ```
  In Takeaway-style lists, `- **小標**:一句話` with a blank line between bullets is fine.
- **Tables only for real data** (numbers, per-method results, error-type percentages). Don't use `| 特性 | 說明 |` or `| # | 貢獻 |` tables for prose. Short enumerations become a plain numbered list; label + explanation pairs become a bold header followed by bullets.
- **Dense descriptive bullets** (benchmark/dataset descriptions): write one lead sentence, then sub-bullets such as `規模 / 任務 / 評估指標 / baseline`.
- **Parallel facts packed into one bullet** with commas or semicolons get split into separate bullets.
- **Blockquotes** longer than about 2 sentences get split with an empty `>` line.
- **Never write a bare `$`** for currency (`$140`). Obsidian treats it as an unclosed math delimiter, so write `140 美元` or `\$140`.
- Put a space after `直覺:` if the next word is English, so it doesn't read as a tag.

## Step 4 — Self-check before reporting

1. Scan for crowded blocks. This prints non-table, non-code lines longer than about 120 characters:
   ```bash
   python3 -c "import sys;[print(i,len(l)) for i,l in enumerate(open(sys.argv[1]),1) if len(l)>120 and not l.lstrip().startswith(('|','\`\`\`'))]" "<note path>"
   ```
   For each hit, check whether it holds more than one idea or inline formula. If it does, split it.
2. Proofread for garbled characters and typos, and check wording against the source. Past slips: 旁邁→旁邊, 昃子→梳妝台 (dressers), 唯二→唯一.
3. Check that every number in the note appears in the source text.

## Step 5 — Report back

Give a short summary: where the note was saved, the sections it covers (and where the methodology and limitations parts are), what was skipped (e.g. the prompt appendix), which fallback was used if any, and any new placeholder `[[links]]`. Don't edit other notes unless asked.

## Notes
- To *expand or reformat* an existing note, read it first and keep the good structure. Add subsections, worked examples and `> 直覺` blocks, then apply the Step 3 rules to the whole note, not just the part the user pointed at. "詳細一點" means full detail, not a ponytail-minimal version.
- The `#paper` tag on line 1 is required (vault convention).
