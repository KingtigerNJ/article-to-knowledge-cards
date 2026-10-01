---
name: make-knowledge-cards
description: Turn a pasted article or a local Markdown/TXT file into 5-8 self-contained knowledge cards. This skill should be used when the user wants to extract and distill the key knowledge from a text into study or review cards, e.g. "把这些内容做成知识卡片", "提炼成卡片", "生成知识卡片", "make knowledge cards", "turn this into study cards". Each card contains a title, the core knowledge point, a concise explanation, and an example or a self-test question. Input is pasted text or a .md/.txt file only; this skill does not fetch web pages, parse PDFs, export to Anki, or provide a GUI.
agent_created: true
---

# Make Knowledge Cards

## Overview

Convert an article (pasted text, or a local `.md`/`.markdown`/`.txt` file) into a small set of high-value knowledge cards. Each card isolates exactly one knowledge point so it can be studied and reviewed on its own.

The value of this skill is *selection*: keep only what is truly important, drop everything else, and never invent what the source does not say.

## When to use

Trigger when the user asks to distill text into knowledge cards, study cards, flashcards, or review notes. Typical phrasings:

- "把这些内容做成知识卡片" / "帮我提炼成卡片" / "生成知识卡片"
- "make knowledge cards from this" / "turn this article into study cards"

## Scope

Supported input:

- Text pasted directly in the conversation.
- A local file path ending in `.md`, `.markdown`, or `.txt` — read it before generating.

Explicitly out of scope — do not silently attempt these:

- Web pages or URLs (no fetching).
- PDFs and other binary documents (`.pdf`, `.docx`, `.pptx`, images).
- Anki export (`.apkg`) or any spaced-repetition software format.
- Any graphical interface.

When the request needs something out of scope, state the limitation in one line and offer the supported path (e.g. ask the user to paste the text or save it as `.md`/`.txt`).

## Workflow

### Step 1 — Acquire and normalize the source

- Pasted text → use it directly.
- A file path → read the file. Accept only `.md`, `.markdown`, `.txt`. If the extension is unsupported, say so and stop; do not guess a parser.
- If the path does not exist, say the file was not found and stop.
- If several files are given, say you will handle one text at a time and start with the first, unless the user asks to merge them.
- If the input is empty, or too thin to contain reusable knowledge (e.g. a single slogan with no idea behind it), say the material is insufficient and return fewer cards — or none — rather than padding.

### Step 2 — Identify candidate knowledge points

Scan the whole text and pull out candidates that carry durable, reusable knowledge. Prefer, roughly in order:

1. Definitions and key terms ("X is …", "X 指的是 …").
2. Core mechanisms — how something works and why.
3. Causal relations and conditions (causes, requires, depends on, "if … then …").
4. Rules, formulas, thresholds, classifications.
5. Meaningful distinctions and common misconceptions.
6. Actionable steps, when the article is a how-to.

Exclude: anecdotes, tone or marketing filler, restatements of an idea already captured, examples that only illustrate a point already counted, and headings/transitions with no content of their own.

Adapt to the genre (see Selection calibration): a how-to yields one card per independently applicable guideline, a persuasive essay yields the thesis plus its distinct reasons, an explainer yields the causal chain.

### Step 3 — Deduplicate and merge

- Collapse candidates that express the same knowledge point, even when worded differently.
- Merge candidates that are facets of one idea; keep them separate only when they are genuinely distinct.
- Merge test: if two candidates would be answered by the same self-test question, they are one card; if they need different answers, they are two cards. Use this as the deciding rule when merging feels uncertain.
- Never output two cards whose core knowledge overlaps.

### Step 4 — Decide the card count

- Target 5-8 cards for a normal article.
- Rank candidates by importance and keep the strongest.
- If fewer than 5 distinct, non-overlapping points exist, output fewer. Do not pad, split, or invent to reach 5 — returning 3 cards for a thin source is correct.
- If a large source yields more than 8 strong points, keep the 8 most important and briefly note that more material exists.

### Step 5 — Write each card

Fill four fields for every kept knowledge point:

1. **Title (标题)** — a short noun phrase naming the single point (roughly ≤ 12 words). Not a full sentence, not a question, no ordering words like "第一点" / "point 1".
2. **Core knowledge (核心知识)** — the single takeaway in one sentence: the definition, rule, or claim to remember. Prefer the source's own wording for definitions.
3. **Concise explanation (简明解释)** — 2-4 sentences: what it means, why it matters, how it works, or how it connects. Use only context from the source.
4. **Example or self-test (例子或自测问题)** — exactly one of:
   - An **example**, when the source contains a concrete case, figure, or illustration for this point — reproduce it faithfully, do not invent a new one.
   - Otherwise a **self-test question**, whose answer is fully derivable from this card's own content.

Order the cards from foundational to derived, following the source's logical flow.

### Step 6 — Output

- Return a numbered list of cards using the template below. Do not add per-card commentary outside the four fields.
- Write all card content in the same language as the source text.
- Keep the source's original terms, including loanwords and abbreviations (e.g. keep `feat`/`fix`, `GET`/`PUT`) rather than translating technical terms.
- Add at most one short line before the cards when the card count falls outside 5-8 or when substantial material was dropped, explaining why. Otherwise start directly with the cards.

## Card template

Localize the labels to the source language (Chinese labels shown):

```
### 卡片 N：<标题>
- **核心知识**：<一句话>
- **简明解释**：<2-4 句>
- **例子 / 自测**：<具体例子，或一个自测问题>
```

## Selection calibration

- **Merge test first.** One self-test answer → one card; different answers → separate cards.
- **How-to articles.** One card per independently applicable guideline. If the article instead teaches a single linear procedure (ordered steps that only make sense together), make one card for the procedure itself and separate cards only for its key principles, not one card per step.
- **Persuasive / opinion articles.** Capture the central thesis as one card, then one card per distinct supporting reason or standalone recommendation. Keep the author's explicit caveats as part of the related card, or as their own card if they change the conclusion. Do not turn rhetorical framing or the author's tone into a card.
- **Explainers.** Follow the causal chain: each link that carries a mechanism earns a card.
- **Numbers and figures.** Keep exact quantities, thresholds, and units (e.g. 50 字符, 72 字符, 60%, 1/λ⁴) attached to the card they belong to. Never round or approximate them.

## Hard rules

- **One card = one knowledge point.** If a card's core knowledge needs "and" to join two independent ideas, split it. A single idea plus its direct consequence is still one card.
- **No duplication.** No two cards may share the same core knowledge.
- **No fabrication.** Every statement must be traceable to the source. Facts and recommendations the source states explicitly may each be a card. Anything you infer or add from outside the source must never appear as a card or as the source's claim — not even if it is correct.
- **Do not force the count.** Fewer than 5 cards is acceptable when the source is thin; more than 8 is not.
- **Stay faithful.** Preserve the source's meaning, numbers, and terminology, including qualifiers such as "通常在…" / "usually" and "在…条件下".
- **Do not resolve ambiguity.** If the source is vague or self-contradictory, reflect that instead of inventing detail; you may write "原文未说明" / "not specified in the source".
- **Cards capture knowledge, not the article.** Do not summarize the article's structure, author, or narrative.

## Worked example

Source excerpt:

> 光合作用是植物利用光能，把二氧化碳和水转化成有机物并释放氧气的过程。它发生在叶绿体中，叶绿体里的叶绿素负责吸收光能。光合作用分为光反应和暗反应两个阶段：光反应在类囊体薄膜上进行，把光能转成活跃的化学能；暗反应在叶绿体基质中进行，利用这些能量固定二氧化碳，生成糖类。

Resulting cards (2 of the 3 kept):

```
### 卡片 1：光合作用的定义
- **核心知识**：光合作用是植物利用光能，把二氧化碳和水转化为有机物并释放氧气的过程。
- **简明解释**：它既是植物制造自身养分的方式，也是大气氧气的来源，是把光能转成化学能的关键过程。
- **例子 / 自测**：自测——光合作用的原料和产物分别是什么？

### 卡片 2：光合作用的两阶段分工
- **核心知识**：光合作用分光反应和暗反应，前者把光能转为活跃化学能，后者用这些能量固定二氧化碳生成糖类。
- **简明解释**：两阶段在叶绿体的不同部位发生：光反应在类囊体薄膜，暗反应在叶绿体基质。能量先被捕获，再被用于合成有机物。
- **例子 / 自测**：自测——光反应和暗反应分别在叶绿体的哪个部位进行？
```

The third card covers "叶绿素吸收光能": it is a distinct point needing a different answer, so it earns its own card instead of being folded in. A short source like this yields 3 cards, not 5 — that is the correct behavior.

## Common failure modes

Check the output against these before returning it:

- **Over-splitting.** Turning one idea and its cause/consequence into two cards. Apply the merge test.
- **Under-splitting.** Packing several independent facts into one card. If the self-test has multiple unrelated answers, split.
- **Padding.** Inventing or stretching points to reach 5 cards. Three correct cards beat five padded ones.
- **Paraphrase drift.** Rewriting the source so freely that the meaning shifts. Keep the source's terms and qualifiers.
- **Borrowed knowledge.** Adding well-known facts the article never states. Delete them.

## Edge cases

- **URL or "fetch this page"** → out of scope; ask the user to paste the text.
- **PDF / Word / PPT / image** → out of scope; ask for `.md`/`.txt` or pasted text.
- **"Export these to Anki"** → out of scope; deliver the cards only.
- **File path not found** → report it and stop.
- **A list of unrelated tips** → one card per tip, merged where they overlap, capped at 8.
- **A single linear procedure** → one card for the procedure plus its key principles, not one card per step.
- **Mostly narrative, opinion, or non-knowledge text** (diary, poem, shopping list) → return few cards or none, and say why.
