# Tavily x402 Endpoint

## Search

- **URL**: `https://x402.tavily.com/search`
- **Method**: POST
- **Protocol**: x402 on Base
- **Price**: $0.01 USDC per call
- **Auth**: None — x402 payment replaces API keys

### Request body

```json
{
  "query": "scientific breakthroughs announced today",
  "topic": "news",
  "time_range": "day",
  "max_results": 10,
  "include_answer": true
}
```

### Key parameters

| Parameter | Type | Description |
|---|---|---|
| `query` | string | Search query (1-1000 chars) |
| `topic` | string | `"news"`, `"general"`, or `"finance"` |
| `time_range` | string | `"day"`, `"week"`, `"month"`, `"year"` |
| `max_results` | int | 1-50 results |
| `include_answer` | bool | Include AI-generated answer summary |
| `search_depth` | string | Always `"advanced"` on x402 |
| `include_domains` | list | Restrict to specific domains |
| `exclude_domains` | list | Exclude specific domains |

### Response shape

```json
{
  "query": "...",
  "answer": "AI-generated summary of results",
  "results": [
    {
      "title": "Article title",
      "url": "https://...",
      "content": "Extracted article text...",
      "score": 0.95,
      "published_date": "Wed, 27 May 2026 11:15:35 GMT"
    }
  ],
  "response_time": 1.9
}
```

### Awal CLI example

```bash
awal x402 pay -X POST \
  -d '{"query": "scientific breakthroughs today", "topic": "news", "time_range": "day", "max_results": 10, "include_answer": true}' \
  --json "https://x402.tavily.com/search"
```

With `--json`, the response wraps in `{"status": 200, "data": {...}}`. The Tavily results are in `data`.

---

# Dellbot TTS-Nano Endpoint

## Text-to-Speech (Kokoro)

- **URL**: `https://x402.dellbot.win/tts-nano`
- **Method**: POST
- **Protocol**: x402 on Base
- **Price**: $0.005 USDC per call
- **Max input**: 1000 characters
- **Returns**: Raw MP3 binary (`audio/mpeg`) — no JSON wrapper

### Request body

```json
{
  "input": "Text to speak...",
  "model": "tts-kokoro",
  "voice": "af_sky",
  "response_format": "mp3"
}
```

### Parameters

| Parameter | Type | Description |
|---|---|---|
| `input` | string | Text to synthesize (max 1000 chars) |
| `model` | string | `"tts-kokoro"` (only option) |
| `voice` | string | Voice ID (e.g. `"af_sky"`) |
| `response_format` | string | `"mp3"`, `"wav"`, `"opus"`, `"aac"`, `"flac"` |
| `speed` | number | Speech speed multiplier |

### Awal CLI example

```bash
awal x402 pay -X POST \
  -d '{"input": "Hello world.", "model": "tts-kokoro", "voice": "af_sky", "response_format": "mp3"}' \
  "https://x402.dellbot.win/tts-nano" > output.mp3
```

Response is raw binary — pipe directly to file.
