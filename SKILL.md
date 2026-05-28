---
name: daily-news
description: >
  Daily world news briefing powered by Tavily x402 search and Kokoro TTS audio.
  Covers science/space breakthroughs, health/medicine, economy/business, major disasters,
  and technology breakthroughs — no politics, no conflict, no diplomacy.
  Outputs a terminal briefing + self-contained HTML report + MP3 audio narration,
  all saved to ~/Documents/. Pays via Awal wallet (x402 on Base).
  Typical cost: ~$0.10 USDC per run (10 searches, audio is free via macOS say).
  Use when: "daily news", "news briefing", "what happened today", "world news",
  "science news", "today's headlines", "news report", "daily briefing".
---

# Daily News Briefing

World news briefing: 10 Tavily searches → Claude analysis → HTML report + audio narration.
All payments via Awal wallet (x402 on Base).

## Workflow

### Step 1 — Check balance

```bash
awal balance
```

Need ≥$0.15 USDC on Base (10 searches × $0.01 + audio ~$0.015). If insufficient, tell the user to fund via `awal show`.

### Step 2 — Run 10 Tavily searches

Make 10 calls via Bash. Each call:

```bash
awal x402 pay -X POST -d '{"query": "<query>", "topic": "news", "time_range": "day", "max_results": 10, "include_answer": true}' --json "https://x402.tavily.com/search"
```

The 10 queries (2 per topic):

| # | Topic | Query |
|---|---|---|
| 1 | Science | `scientific discovery breakthrough research published today` |
| 2 | Science | `space mission NASA astronomy discovery announcement` |
| 3 | Health | `health medical breakthrough drug treatment approved` |
| 4 | Health | `disease research clinical trial results public health` |
| 5 | Economy | `major economic news central bank market GDP recession` |
| 6 | Economy | `major business acquisition bankruptcy corporate earnings report` |
| 7 | Disasters | `earthquake disaster major accident casualties emergency` |
| 8 | Disasters | `extreme weather wildfire flood hurricane major event` |
| 9 | Tech | `technology breakthrough innovation scientific progress announced` |
| 10 | Tech | `AI breakthrough model release computing advance chip` |

Parse JSON response. The `data` field has `answer` (string) and `results[]` (each with `title`, `url`, `content`, `score`, `published_date`).

### Step 3 — Analyze and deduplicate

Extract unique stories. For each: title, 2-3 sentence summary, source URL, topic area. Discard duplicates and low-relevance results. Rank by significance.

Filter rules:
- NO politics, elections, legislation, diplomacy, military conflict
- NO opinion pieces, editorials, celebrity gossip
- YES science, health, economy, disasters, genuine tech breakthroughs
- Prefer reputable outlets (Reuters, BBC, AP, NYT, CNN, Guardian, FT, Nature, Science)

Aim for 6-10 top stories.

### Step 4 — Terminal briefing

Print a concise briefing:

```
## Daily News — {today's date}

### Science & Space
- **{headline}** — {1-sentence summary} [Source]

### Health & Medicine
- **{headline}** — {1-sentence summary} [Source]

### Economy & Business
- **{headline}** — {1-sentence summary} [Source]

### Major Events
- **{headline}** — {1-sentence summary} [Source]

### Technology
- **{headline}** — {1-sentence summary} [Source]

---
Cost: $0.10 USDC (10 searches, audio free)
Report: ~/Documents/daily-news-{YYYYMMDD}.html
Audio: ~/Documents/daily-news-{YYYYMMDD}.aiff
```

Skip any topic section with zero stories.

### Step 5 — Generate HTML report

Read the template at [assets/report-template.html](assets/report-template.html) and fill in the placeholders:

- `{{DATE}}` → today's date (e.g. "May 27, 2026")
- `{{SIGNAL_COUNT}}` → number of stories
- `{{COST}}` → total USDC spent
- `{{STORIES}}` → generated story HTML (see format below)

For each topic section, insert a section label, then articles:

```html
<div class="section-label">Science & Space</div>

<article>
  <div class="signal-header">
    <span class="signal-num">01</span>
    <span class="cat cat-science">Science</span>
  </div>
  <h2><a href="https://..." target="_blank">Headline here</a></h2>
  <p class="summary">Summary text here.</p>
  <div class="signal-footer">
    <a class="src" href="https://..." target="_blank">Source Name</a>
  </div>
</article>
```

Category CSS classes: `cat-science`, `cat-health`, `cat-economy`, `cat-events`, `cat-tech`.

Save the filled template to `~/Documents/daily-news-{YYYYMMDD}.html`.

### Step 6 — Generate audio narration

Write a narration script covering the top 5-6 stories. Anchor-style: concise, factual, no filler. No character limit.

Script format: "Here's your daily news briefing for {date}. [Story 1]. [Story 2]. ... That's your briefing for today."

Save the audio file and play it using macOS built-in text-to-speech (free, no API call):

```bash
say -o ~/Documents/daily-news-{YYYYMMDD}.aiff "<narration_script>"
afplay ~/Documents/daily-news-{YYYYMMDD}.aiff
```

### Step 7 — Report cost

Print total USDC spent: 10 × $0.01 (Tavily) = $0.10. Audio is free (macOS say).
