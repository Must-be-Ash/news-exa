# Daily News Briefing

A Claude Code skill that delivers a curated daily news briefing — covering science, health, economy, disasters, and tech breakthroughs. No politics, no conflict, no gossip.

Outputs a terminal summary, a styled HTML report, and an audio narration — all saved to `~/Documents/`.

## Install

```bash
npx skills add https://github.com/Must-be-Ash/daily-news
```

## Prerequisites

- [Awal](https://github.com/anthropics/awal) CLI installed (`npm install -g awal`)
- Awal wallet funded with USDC on Base (each run costs ~$0.10)
- macOS (uses built-in `say` for audio narration)

## Usage

In Claude Code, run:

```
/daily-news
```

Or just ask naturally: *"What's in the news today?"*, *"Give me a daily briefing"*, etc.

## What it does

1. Runs 10 targeted Tavily searches via x402 micropayments
2. Filters and deduplicates results across 5 topic areas
3. Prints a concise terminal briefing
4. Generates a self-contained HTML report at `~/Documents/daily-news-YYYYMMDD.html`
5. Creates an audio narration at `~/Documents/daily-news-YYYYMMDD.aiff`

## Topics covered

| Category | Examples |
|---|---|
| Science & Space | Discoveries, missions, research papers |
| Health & Medicine | Drug approvals, clinical trials, public health |
| Economy & Business | Markets, earnings, acquisitions, central bank moves |
| Major Events | Natural disasters, extreme weather, emergencies |
| Technology | AI breakthroughs, chip advances, new releases |

## Cost

~$0.10 USDC per run (10 Tavily searches at $0.01 each). Audio is free via macOS `say`.
