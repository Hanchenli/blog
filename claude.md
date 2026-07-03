# Blog Writing Guide (for Claude)

Hugo blog ("Hanchen's Space Bar", PaperMod theme). Posts live in `content/posts/*.md`.

## Workflow (how we write together)
Hanchen drafts, Claude fills in. A draft arrives as:
- A **TLDR** at the top (the core thesis / takeaways).
- An **outline** below it — section headers plus bullet-point ideas.

Your job is to **fill in the middle**: turn the outline bullets into full prose while keeping Hanchen's argument and ordering intact.

Rules for filling in:
- Preserve the TLDR and every outline point. Don't drop, reorder, or contradict ideas — expand them.
- Match the existing voice: first-person, technical but accessible, explains jargon, links prerequisites and sources inline via `[text](url)`.
- Prefer concrete numbers, examples, and mechanisms over hand-waving. Flag with a comment where a real figure is needed if you don't have one.
- Don't invent facts, citations, or benchmark numbers. If something needs a source Hanchen has, leave a `<!-- TODO: cite ... -->`.

## Graphs (you insert placeholders, Hanchen creates them)
Wherever a figure would help, insert a placeholder image reference **immediately followed by an HTML comment containing a generation prompt** describing the graph. Hanchen creates the actual image later.

```markdown
![Descriptive alt](../../images/<post_slug>/<figure_name>.png)
<!-- GRAPH: describe what the figure should show — axes, series, comparison, key
     takeaway the reader should get. Enough detail for Hanchen to build it. -->
```

- Store images in `static/images/<post_slug>/`; body refs use `../../images/<slug>/file.png`.
- Add filler graphs where they aid the argument (breakdowns, comparisons, architecture sketches, before/after), but don't overdo it.

## Front matter
Every post starts with this YAML block:

```yaml
---
title: "Title Case, in Quotes"
date: 2026-02-16          # YYYY-MM-DD; use a 2069-* date for unpublished drafts
description: "One-sentence summary shown in listings and previews."
author: "Hanchen Li"      # add ", and Collaborators" when relevant
tags: ["LLM", "Agents"]   # 3-8 topical tags
categories: ["general"]
cover:
    image: images/<post_slug>/main.png   # note: no ../../ for the cover field
    alt: "Short alt text"
    relative: false
ShowToc: true
TocOpen: false
---
```

## Structure
- Open with a short intro paragraph (no heading) stating what the post covers.
- `##` for main sections, `###` for subsections; `### Acknowledgement` at the end when relevant.
- Use `<!-- ... -->` to hold back unfinished sections.

## Don't touch
`public/`, `resources/`, and theme files are generated/vendored. Reference `continual_learning.md` and `fast_agent.md` for tone.
