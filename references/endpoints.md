# Exa Search API

## Search

- **URL**: `https://api.exa.ai/search`
- **Method**: POST
- **Auth**: `Authorization: Bearer <EXA_API_KEY>`
- **Price**: ~$0.007–0.01 per search (varies by numResults and contents)

### Optimal request body for news

```json
{
  "query": "specific descriptive natural language about articles to find",
  "type": "auto",
  "category": "news",
  "numResults": 10,
  "startPublishedDate": "2026-05-28T09:40:00.000Z",
  "includeDomains": ["reuters.com", "bbc.com", "nytimes.com"],
  "contents": {
    "text": {
      "maxCharacters": 1400,
      "stripLinks": true
    },
    "highlights": {
      "maxCharacters": 320
    }
  }
}
```

### Key parameters

| Parameter | Type | Description |
|---|---|---|
| `query` | string | Natural language description — specific and descriptive, not keyword lists |
| `type` | string | `"auto"` (recommended), `"fast"`, `"instant"`, `"deep"`, `"deep-lite"`, `"deep-reasoning"` |
| `category` | string | `"news"`, `"company"`, `"research paper"`, `"people"`, `"personal site"`, `"financial report"` |
| `numResults` | int | 1–100, defaults to 10 |
| `startPublishedDate` | string | ISO 8601 datetime — only results published after this |
| `endPublishedDate` | string | ISO 8601 datetime — only results published before this |
| `includeDomains` | list | Restrict to these domains |
| `excludeDomains` | list | Exclude these domains |
| `maxAgeHours` | int | Cache freshness: 24 = use cache if <24hr old, 0 = always livecrawl, -1 = cache only |
| `contents.text` | object | Full article text. Options: `maxCharacters`, `stripLinks`, `includeHtmlTags` |
| `contents.highlights` | object | Key fact excerpts relevant to query. Options: `maxCharacters`, `query` |
| `contents.summary` | object | LLM-generated abstract. Supports `outputSchema` for structured extraction |

### Response shape

```json
{
  "requestId": "abc123",
  "results": [
    {
      "id": "https://...",
      "title": "Article title",
      "url": "https://...",
      "publishedDate": "2026-05-28T14:30:00.000Z",
      "score": 0.95,
      "text": "Clean article text without links...",
      "highlights": ["Key fact excerpt 1", "Key fact excerpt 2"],
      "image": "https://...",
      "favicon": "https://..."
    }
  ],
  "searchTime": 207.5,
  "costDollars": {
    "total": 0.008,
    "search": {"neural": 0.008}
  }
}
```

### Best practices (from Exa docs)

- **`type: "auto"`** — let Exa pick the best retrieval strategy. Don't use `"neural"` directly.
- **`category: "news"`** — targets a specialized news index with better relevance for current events.
- **`highlights`** — "News articles are verbose. Highlights extract the key facts." Essential for agent workflows — 10x more token-efficient than full text.
- **`text.stripLinks: true`** — removes noisy markdown links from article text.
- **Queries should be specific** — "AI startup funding announcements" beats "AI news today". Describe the articles you want to find.
- **`startPublishedDate`** — use full ISO datetime (not just date) for precise 24hr windows.
