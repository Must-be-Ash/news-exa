---
name: daily-news
description: >
  Daily world news briefing powered by Exa search and macOS audio narration.
  Covers science/space breakthroughs, health/medicine, economy/business, major disasters,
  and technology breakthroughs — no politics, no conflict, no diplomacy.
  Outputs a terminal briefing + self-contained HTML report + audio narration,
  all saved to ~/Documents/DailyNews/.
  Cost: ~$0.05 per run (6 Exa searches, audio is free via macOS say).
  Use when: "daily news", "news briefing", "what happened today", "world news",
  "science news", "today's headlines", "news report", "daily briefing".
allowed-tools: WebSearch, Bash, Read, Write
metadata:
  openclaw:
    requires:
      env:
        - EXA_API_KEY
    primaryEnv: EXA_API_KEY
---

# Daily News Briefing

World news briefing: 6 Exa searches → Claude analysis → HTML report + audio narration.
Cost: ~$0.05 per run (audio free via macOS `say`). Output: `~/Documents/DailyNews/`.

## Exa notes

Exa is a **neural search engine**. Queries should be specific, descriptive natural language — not keyword lists. Use `type: "auto"` and `category: "news"` to target Exa's dedicated news index. Request both `text` (with `stripLinks`) and `highlights` for best coverage. Exa returns `results[]` each with `title`, `url`, `text`, `highlights`, `publishedDate`, `score`.

## Workflow

### Step 1 — Setup

```bash
[ -d ~/Documents/DailyNews ] || mkdir -p ~/Documents/DailyNews
```

Load the Exa API key and set the 24-hour search window:

```bash
EXA_KEY=$(grep "^EXA_API_KEY" ~/.config/daily-news/.env 2>/dev/null | cut -d'=' -f2)
if [ -z "$EXA_KEY" ]; then echo "ERROR: No EXA_API_KEY in ~/.config/daily-news/.env"; fi
START_DATE=$(python3 -c "from datetime import datetime, timedelta, timezone; print((datetime.now(timezone.utc) - timedelta(hours=24)).strftime('%Y-%m-%dT%H:%M:%S.000Z'))")
echo "EXA_KEY loaded, START_DATE=${START_DATE}"
```

### Step 2 — Run 6 Exa searches

Run all 6 searches in parallel using background jobs. Each writes to a temp file.

Every search uses:
- `type: "auto"` — lets Exa pick the best retrieval strategy
- `category: "news"` — targets Exa's dedicated news index for better relevance
- `contents.text` with `maxCharacters: 1400` and `stripLinks: true` — clean, link-free article text
- `contents.highlights` with `maxCharacters: 320` — key fact excerpts for efficient processing
- `startPublishedDate` — full UTC datetime, exactly 24 hours back

```bash
EXA_KEY=$(grep "^EXA_API_KEY" ~/.config/daily-news/.env 2>/dev/null | cut -d'=' -f2)
START_DATE=$(python3 -c "from datetime import datetime, timedelta, timezone; print((datetime.now(timezone.utc) - timedelta(hours=24)).strftime('%Y-%m-%dT%H:%M:%S.000Z'))")

# 1 — Science & Space
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"new scientific discovery published today, space mission milestone, physics or biology research breakthrough announced\",
    \"numResults\": 12,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"bbc.com\", \"nytimes.com\", \"theguardian.com\", \"apnews.com\", \"nature.com\", \"newscientist.com\", \"sciencedaily.com\", \"space.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_science.json 2>/dev/null &

# 2 — Health & Medicine
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"medical breakthrough announced today, drug approval by FDA or EMA, clinical trial results showing promising new treatment for disease\",
    \"numResults\": 10,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"bbc.com\", \"nytimes.com\", \"theguardian.com\", \"apnews.com\", \"cnn.com\", \"statnews.com\", \"nature.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_health.json 2>/dev/null &

# 3 — Economy & Business
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"major economic announcement today, central bank interest rate decision, significant stock market movement, corporate earnings surprise or major acquisition\",
    \"numResults\": 10,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"ft.com\", \"bloomberg.com\", \"nytimes.com\", \"economist.com\", \"bbc.com\", \"apnews.com\", \"cnn.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_economy.json 2>/dev/null &

# 4 — Technology & AI
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"AI model release or major tech product launch announced today, semiconductor chip breakthrough, significant startup funding round\",
    \"numResults\": 10,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"bbc.com\", \"nytimes.com\", \"theguardian.com\", \"arstechnica.com\", \"wired.com\", \"ft.com\", \"bloomberg.com\", \"techcrunch.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_tech.json 2>/dev/null &

# 5 — Disasters & Major Events
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"major disaster or emergency today, earthquake with significant casualties, large wildfire or flood forcing evacuations, industrial accident\",
    \"numResults\": 8,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"bbc.com\", \"apnews.com\", \"cnn.com\", \"nytimes.com\", \"theguardian.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_disasters.json 2>/dev/null &

# 6 — Energy & Climate
curl -s https://api.exa.ai/search \
  -H "Authorization: Bearer ${EXA_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"renewable energy milestone announced today, battery technology breakthrough, nuclear fusion progress, significant climate science research findings published\",
    \"numResults\": 8,
    \"startPublishedDate\": \"${START_DATE}\",
    \"type\": \"auto\",
    \"category\": \"news\",
    \"includeDomains\": [\"reuters.com\", \"bbc.com\", \"nytimes.com\", \"theguardian.com\", \"bloomberg.com\", \"ft.com\", \"nature.com\", \"newscientist.com\"],
    \"contents\": {
      \"text\": {\"maxCharacters\": 1400, \"stripLinks\": true},
      \"highlights\": {\"maxCharacters\": 320}
    }
  }" > /tmp/exa_energy.json 2>/dev/null &

wait
echo "All 6 Exa searches complete."
```

Then parse all results into a single stream. Use `highlights` for quick triage and `text` for deeper context:

```bash
python3 -c "
import json
files = {
    'SCIENCE': '/tmp/exa_science.json',
    'HEALTH': '/tmp/exa_health.json',
    'ECONOMY': '/tmp/exa_economy.json',
    'TECH': '/tmp/exa_tech.json',
    'DISASTERS': '/tmp/exa_disasters.json',
    'ENERGY': '/tmp/exa_energy.json'
}
total_cost = 0
total_results = 0
for label, path in files.items():
    try:
        data = json.load(open(path))
        total_cost += data.get('costDollars', {}).get('total', 0)
        results = data.get('results', [])
        total_results += len(results)
        for r in results:
            domain = r.get('url','').split('/')[2].replace('www.','')
            pub = r.get('publishedDate','')[:16]
            title = r.get('title','')
            url = r.get('url','')
            highlights = r.get('highlights', [])
            text = (r.get('text') or '')[:600]
            print(f'{label}|{domain}|{pub}|{title}|{url}')
            if highlights:
                for h in highlights[:3]:
                    print(f'  HIGHLIGHT: {h}')
            elif text:
                print(f'  SNIPPET: {text}')
    except Exception as e:
        print(f'{label}|ERROR: {e}')
print(f'\n--- {total_results} results across 6 searches, cost: \${total_cost:.3f} ---')
"
```

### Step 3 — Analyze and deduplicate

From all results, extract every unique newsworthy story. For each: title, 2-3 sentence summary, source URL, topic area. Remove exact duplicates only — keep stories that cover different aspects of the same event. Rank by significance.

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
Cost: ~$0.05 (6 Exa searches, audio free)
Report: ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.html
Audio: ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff
```

Skip any topic section with zero stories.

### Step 5 — Generate HTML report

Read the template at [assets/report-template.html](assets/report-template.html) and fill in the placeholders:

- `{{DATE}}` → today's date and time (e.g. "May 27, 2026 · 6:08 PM")
- `{{SIGNAL_COUNT}}` → number of stories
- `{{COST}}` → total cost (from Exa costDollars)
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

Write a narration script covering **every story** from the briefing — not just the top 8-10. Every story that made it into the HTML report gets a line in the script. Anchor-style: concise, factual, no filler. No character limit. Group by section (science, health, economy, etc.) with smooth transitions between sections.

Script format: "Here's your daily news briefing for {date}. [Story 1]. [Story 2]. ... That's your briefing for today."

**Important:** Write the full narration script to a temp file, then use `say -f` to read from the file. Do NOT pass the script as a shell argument — long scripts get truncated by shell argument limits.

```bash
cat <<'NARRATION' > /tmp/daily-news-narration.txt
{full narration script here}
NARRATION
say -f /tmp/daily-news-narration.txt -o ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff
open ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.html
afplay ~/Documents/DailyNews/daily-news-{YYYYMMDD}-{HHMMSS}.aiff
```

### Step 7 — Report cost

Print total cost from Exa's `costDollars` fields (typically ~$0.05 for 6 searches). Audio is free (macOS say).
