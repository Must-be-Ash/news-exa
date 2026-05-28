---
name: daily-news
description: >
  Daily world news briefing powered by Tavily x402 search and macOS audio narration.
  Covers science/space breakthroughs, health/medicine, economy/business, major disasters,
  and technology breakthroughs — no politics, no conflict, no diplomacy.
  Outputs a terminal briefing + self-contained HTML report + audio narration,
  all saved to ~/Documents/DailyNews/. Pays via Awal wallet (x402 on Base).
  Typical cost: ~$0.10 USDC per run (10 searches, audio is free via macOS say).
  Use when: "daily news", "news briefing", "what happened today", "world news",
  "science news", "today's headlines", "news report", "daily briefing".
---

# Daily News Briefing

World news briefing: 10 Tavily searches → Claude analysis → HTML report + audio narration.
All payments via Awal wallet (x402 on Base). Output: `~/Documents/DailyNews/`.

## Workflow

### Step 1 — Check balance

```bash
npx awal balance
```

Need ≥$0.15 USDC on Base (10 searches × $0.01). If insufficient, tell the user to fund via `npx awal show`.

### Step 2 — Run 10 Tavily searches

Ensure `~/Documents/DailyNews/` exists:

```bash
mkdir -p ~/Documents/DailyNews
```

Make 10 calls via Bash. Each call:

```bash
npx awal x402 pay -X POST -d '{"query": "<query>", "topic": "news", "time_range": "day", "max_results": 10, "include_answer": true}' --json "https://x402.tavily.com/search"
```

The 10 queries:

| # | Topic | Query |
|---|---|---|
| 1 | Science | `scientific discovery breakthrough research published today` |
| 2 | Science | `space mission NASA astronomy discovery announcement today` |
| 3 | Health | `health medical breakthrough drug treatment approved today` |
| 4 | Health | `disease research clinical trial results public health today` |
| 5 | Economy | `major economic news central bank interest rate market today` |
| 6 | Economy | `major business acquisition IPO corporate earnings report today` |
| 7 | Disasters | `natural disaster earthquake wildfire flood hurricane casualties today` |
| 8 | Tech | `technology breakthrough innovation scientific progress today` |
| 9 | Tech | `AI breakthrough model release computing chip semiconductor today` |
| 10 | Tech | `renewable energy climate breakthrough battery solar today` |

Parse JSON response. The `data` field has `answer` (string) and `results[]` (each with `title`, `url`, `content`, `score`, `published_date`).

### Step 3 — Analyze and deduplicate

From all ~100 results, extract every unique newsworthy story. For each: title, 2-3 sentence summary, source URL, topic area. Remove exact duplicates only — keep stories that cover different aspects of the same event. Rank by significance.

Filter rules:
- NO politics, elections, legislation, diplomacy, military conflict
- NO opinion pieces, editorials, celebrity gossip
- YES science, health, economy, disasters, genuine tech breakthroughs, energy/climate
- Prefer reputable outlets (Reuters, BBC, AP, NYT, CNN, Guardian, FT, Nature, Science)

Surface as many quality stories as possible — aim for 15-25 stories. Do not artificially cap the output. If there are 20 good stories, include all 20.

### Step 4 — Terminal briefing

Print a comprehensive briefing:

```
## Daily News — {today's date, time}

### Science & Space
- **{headline}** — {1-sentence summary} [Source]
- **{headline}** — {1-sentence summary} [Source]

### Health & Medicine
- **{headline}** — {1-sentence summary} [Source]

### Economy & Business
- **{headline}** — {1-sentence summary} [Source]

### Major Events
- **{headline}** — {1-sentence summary} [Source]

### Technology
- **{headline}** — {1-sentence summary} [Source]

### Energy & Climate
- **{headline}** — {1-sentence summary} [Source]

---
Cost: $0.10 USDC (10 searches, audio free)
Report: ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.html
Audio: ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff
```

Skip any topic section with zero stories.

### Step 5 — Generate HTML report

Read the template at [assets/report-template.html](assets/report-template.html) and fill in the placeholders:

- `{{DATE}}` → today's date and time (e.g. "May 27, 2026 · 6:08 PM")
- `{{SIGNAL_COUNT}}` → number of stories
- `{{COST}}` → total USDC spent
- `{{STORIES}}` → generated story HTML (see format below)

For each topic section, insert a section label, then articles:

```html
<div class="section-label">Science & Space</div>

<article>
  <div class="title-row">
    <h2><a href="https://..." target="_blank">Headline here</a></h2>
  </div>
  <p class="summary">Summary text here.</p>
  <div class="signal-footer">
    <a class="src" href="https://..." target="_blank">Source Name</a>
  </div>
</article>
```

No numbers, no category tags. Section labels handle the grouping.

Save to `~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.html`.

### Step 6 — Generate audio narration

Write a narration script covering the top 8-10 stories. Anchor-style: concise, factual, no filler. No character limit.

Script format: "Here's your daily news briefing for {date}. [Story 1]. [Story 2]. ... That's your briefing for today."

Save the audio file and play it using macOS built-in text-to-speech (free, no API call):

```bash
say -o ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff "<narration_script>"
afplay ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff
```

### Step 7 — Report cost

Print total USDC spent: 10 × $0.01 (Tavily) = $0.10. Audio is free (macOS say).
