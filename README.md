# Daily News Briefing (Exa)

A Claude Code skill that delivers a curated daily news briefing — covering science, health, economy, disasters, and tech breakthroughs. No politics, no conflict, no gossip.

Powered by [Exa](https://exa.ai) neural search. Outputs a terminal summary, a styled HTML report, and an audio narration — all saved to `~/Documents/DailyNews/`.

## Install

```bash
npx skills add https://github.com/Must-be-Ash/news-exa
```

## Prerequisites

- **Exa API key** — sign up and get your key at [exa.ai](https://dashboard.exa.ai/api-keys)
- Save your key to `~/.config/daily-news/.env`:
  ```bash
  mkdir -p ~/.config/daily-news
  echo "EXA_API_KEY=your-key-here" > ~/.config/daily-news/.env
  ```
- macOS (uses built-in `say` for audio narration)

## Usage

In Claude Code, run:

```
/daily-news
```

Or just ask naturally: *"What's in the news today?"*, *"Give me a daily briefing"*, etc.

## What it does

1. Runs 6 targeted Exa neural searches across reputable outlets
2. Filters and deduplicates results across 6 topic areas
3. Prints a concise terminal briefing
4. Generates a self-contained HTML report at `~/Documents/DailyNews/daily-news-YYYYMMDD.html`
5. Creates an audio narration at `~/Documents/DailyNews/daily-news-YYYYMMDD.aiff`

## Topics covered

| Category | Examples |
|---|---|
| Science & Space | Discoveries, missions, research papers |
| Health & Medicine | Drug approvals, clinical trials, public health |
| Economy & Business | Markets, earnings, acquisitions, central bank moves |
| Major Events | Natural disasters, extreme weather, emergencies |
| Technology | AI breakthroughs, chip advances, new releases |
| Energy & Climate | Renewable energy, battery tech, climate research |

## Cost

~$0.04 per run (6 Exa searches at ~$0.007 each). Audio is free via macOS `say`.
