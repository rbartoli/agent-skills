---
name: serpapi-search
description: Search Google Flights, Google Shopping, Amazon, Google Maps, Google Scholar, Google News, YouTube, and 30+ other search verticals via the SerpApi REST API. Use this skill whenever the user asks to check flight prices, compare products, find local businesses, research academic papers, look up stock/finance data, search job listings, compare hotels, or needs any structured search-engine data beyond what a generic web search returns. ESPECIALLY use for flights (Google Flights API with departure/arrival airports, dates, cabin class, stops) and product price comparisons (Amazon, Walmart, Google Shopping). Also trigger when the user says "search for", "look up prices", "find flights", "compare", or asks you to check availability of anything searchable. Prefer this over tavily/WebSearch when structured vertical data (prices, schedules, ratings, coordinates) is needed rather than article content.
allowed-tools: Bash(curl *) Read
---

# SerpApi Search

Search any engine via a single REST endpoint. Each API call costs one search credit.

**Important:** Always use `Bash(curl ...)` to call the SerpApi REST API. Do not use WebFetch, WebSearch, or any other HTTP tool.

## Setup

The API key must be set as an environment variable:

```bash
export SERPAPI_API_KEY="your_key_here"
```

If `SERPAPI_API_KEY` is not set, tell the user:
> Set your SerpApi API key: `export SERPAPI_API_KEY="your_key"` — get one free at https://serpapi.com/manage-api-key

## API Pattern

Every search follows the same pattern regardless of engine:

```bash
curl -s "https://serpapi.com/search.json?engine=ENGINE&QUERY_PARAM=QUERY&api_key=$SERPAPI_API_KEY"
```

The only things that vary per engine are:
1. The `engine` value
2. The query parameter name (`q`, `k`, `query`, etc.)
3. Engine-specific optional parameters

## Engine Selection

Pick the engine based on user intent:

| Intent | Engine | Query param | Docs |
|--------|--------|-------------|------|
| **Web search** | `google_light` | `q` | https://serpapi.com/google-light-api |
| Web search (full features) | `google` | `q` | https://serpapi.com/search-api |
| Bing search | `bing` | `q` | https://serpapi.com/bing-search-api |
| DuckDuckGo search | `duckduckgo` | `q` | https://serpapi.com/duckduckgo-search-api |
| Yahoo search | `yahoo` | `p` | https://serpapi.com/yahoo-search-api |
| Yandex search | `yandex` | `text` | https://serpapi.com/yandex-search-api |
| Baidu search | `baidu` | `q` | https://serpapi.com/baidu-search-api |
| **AI search** | `google_ai_mode` | `q` | https://serpapi.com/google-ai-mode-api |
| AI overview | `google_ai_overview` | `q` | https://serpapi.com/google-ai-overview-api |
| Bing Copilot | `bing_copilot` | `q` | https://serpapi.com/bing-copilot-api |
| Brave AI | `brave_ai_mode` | `q` | https://serpapi.com/brave-ai-mode-api |
| **Amazon products** | `amazon` | `k` | https://serpapi.com/amazon-search-api |
| Walmart products | `walmart` | `query` | https://serpapi.com/walmart-search-api |
| eBay products | `ebay` | `_nkw` | https://serpapi.com/ebay-search-api |
| Google Shopping | `google_shopping` | `q` | https://serpapi.com/google-shopping-api |
| Home Depot | `home_depot` | `q` | https://serpapi.com/home-depot-search-api |
| **Google Maps / local** | `google_maps` | `q` | https://serpapi.com/google-maps-api |
| Google Local | `google_local` | `q` | https://serpapi.com/google-local-api |
| Yelp | `yelp` | `find_desc` | https://serpapi.com/yelp-search-api |
| TripAdvisor | `tripadvisor` | `q` | https://serpapi.com/tripadvisor-search-api |
| OpenTable reviews | `open_table_reviews` | `restaurant_id` | https://serpapi.com/open-table-reviews-api |
| **Google Scholar** | `google_scholar` | `q` | https://serpapi.com/google-scholar-api |
| Google Patents | `google_patents` | `q` | https://serpapi.com/google-patents-api |
| **Google News** | `google_news` | `q` | https://serpapi.com/google-news-api |
| Google Trends | `google_trends` | `q` | https://serpapi.com/google-trends-api |
| **Google Images** | `google_images` | `q` | https://serpapi.com/google-images-api |
| Google Videos | `google_videos` | `q` | https://serpapi.com/google-videos-api |
| Google Lens | `google_lens` | `url` | https://serpapi.com/google-lens-api |
| YouTube | `youtube` | `search_query` | https://serpapi.com/youtube-search-api |
| **Google Flights** | `google_flights` | (see params) | https://serpapi.com/google-flights-api |
| Google Hotels | `google_hotels` | `q` | https://serpapi.com/google-hotels-api |
| Google Travel | `google_travel_explore` | (see params) | https://serpapi.com/google-travel-explore-api |
| **Google Jobs** | `google_jobs` | `q` | https://serpapi.com/google-jobs-api |
| **Google Finance** | `google_finance` | `q` | https://serpapi.com/google-finance-api |
| Google Play | `google_play` | `q` | https://serpapi.com/google-play-api |
| Apple App Store | `apple_app_store` | `term` | https://serpapi.com/apple-app-store |
| **Google Autocomplete** | `google_autocomplete` | `q` | https://serpapi.com/google-autocomplete-api |
| Naver | `naver` | `query` | https://serpapi.com/naver-search-api |

**Default to `google_light` for general web searches** — it's faster and cheaper than `google`. Use `google` only when you need knowledge graph, ads, or advanced SERP features.

## Engine Parameters

Each engine has its own parameters documented in a JSON schema file at `engines/<engine_name>.json` relative to this plugin's root directory. Read the relevant file when you need engine-specific parameter details.

The JSON schema structure:
- `params`: engine-specific parameters (query, filters, pagination, etc.)
- `common_params`: shared SerpApi parameters (api_key, device, no_cache, etc.)
- Each param has `description`, optional `required`, `type`, `options`, and `group` fields.

## Common Parameters

These work across most engines:

| Param | Description |
|-------|-------------|
| `location` | Search origin (city-level recommended). E.g., `Austin, Texas, United States` |
| `gl` | Country code. E.g., `us`, `uk`, `fr` |
| `hl` | Language code. E.g., `en`, `es`, `de` |
| `device` | `desktop` (default), `tablet`, or `mobile` |
| `no_cache` | `true` to force fresh results (costs a credit; cached results are free) |
| `num` | Number of results (where supported) |
| `start` / `page` | Pagination. Google uses `start` (0, 10, 20...), Amazon/Walmart use `page` (1, 2, 3...) |
| `json_restrictor` | Restrict response fields for smaller payloads. E.g., `organic_results.title,organic_results.link` |

## Response Handling

SerpApi returns structured JSON. Key top-level fields vary by engine:

- **Web search**: `organic_results`, `knowledge_graph`, `answer_box`, `related_questions`, `local_results`
- **Shopping**: `shopping_results` or `organic_results` (with price, rating, etc.)
- **Maps**: `local_results` (with address, rating, GPS coordinates, phone)
- **Scholar**: `organic_results` (with citation_id, cited_by count, PDF links)
- **News**: `news_results`
- **Images**: `images_results`
- **Flights**: `best_flights`, `other_flights`, `price_insights`
- **Jobs**: `jobs_results`
- **Finance**: `summary`, `financials`, `graph`

Always summarize results for the user. Never dump raw JSON unless explicitly asked.

Use `json_restrictor` to reduce response size when you only need specific fields.

## Google Flights

**Default to round-trip (`type=1`).** Only use one-way (`type=2`) if the user explicitly asks for one-way.

Key parameters:

| Param | Required | Description |
|-------|----------|-------------|
| `departure_id` | yes | IATA airport code (e.g., `LGW`, `LTN`) |
| `arrival_id` | yes | IATA airport code (e.g., `FLR`, `PSA`) |
| `outbound_date` | yes | `YYYY-MM-DD` |
| `return_date` | round-trip | `YYYY-MM-DD` (required when `type=1`) |
| `type` | no | `1` = round-trip (default), `2` = one-way, `3` = multi-city |
| `stops` | no | `0` = direct only, `1` = 1 stop max, `2` = any |
| `travel_class` | no | `1` = economy (default), `2` = premium economy, `3` = business, `4` = first |
| `adults` | no | Number of adults (default `1`) |
| `children` | no | Number of children |
| `currency` | no | e.g., `GBP`, `USD`, `EUR` |
| `hl` | no | Language, e.g., `en` |
| `gl` | no | Country code, e.g., `uk` |

Example — round-trip, direct only:
```bash
curl -s "https://serpapi.com/search.json?engine=google_flights&departure_id=LGW&arrival_id=FLR&outbound_date=2026-05-23&return_date=2026-05-30&type=1&stops=0&currency=GBP&gl=uk&api_key=$SERPAPI_API_KEY"
```

Response fields: `best_flights`, `other_flights` (each with `.flights[]`, `.price`, `.total_duration`), `price_insights`.

When the user asks to "check flights" without specifying one-way, always search round-trip. If no return date is given, ask for one before searching.

### Presenting results

- **Filter by user's airline/airport constraints** before presenting. Don't show airlines or airports they've excluded — apply memory preferences automatically.
- **Discard impractical options.** If outbound arrives late evening and the return is the next day, drop it — the user gets no usable time at the destination. Rule of thumb: if arrival time + transit to city leaves fewer than ~12 waking hours before the return journey, flag it as impractical or omit it.
- **Sort by price**, cheapest first.
- **Show round-trip total**, not per-leg — the user wants to know what they'll pay.
- Present as a clean table: route, airline, outbound times, round-trip price.

## Multi-Engine Comparison

For price comparisons or cross-engine research, query multiple engines sequentially:

```bash
# Compare prices across Amazon, Walmart, and Google Shopping
curl -s "https://serpapi.com/search.json?engine=amazon&k=airpods+pro&api_key=$SERPAPI_API_KEY"
curl -s "https://serpapi.com/search.json?engine=walmart&query=airpods+pro&api_key=$SERPAPI_API_KEY"
curl -s "https://serpapi.com/search.json?engine=google_shopping&q=airpods+pro&api_key=$SERPAPI_API_KEY"
```

Consolidate and compare the results for the user.

## Rules

1. **Always use `curl` via Bash.** Never use WebFetch, WebSearch, or other HTTP tools. The `allowed-tools` header restricts this skill to `Bash(curl *)` and `Read`.
2. **Confirm before searching** when the query or engine choice is ambiguous. Each non-cached call costs one credit.
3. **Show the curl command** before executing so the user sees exactly what's being called.
4. **Prefer `google_light`** over `google` for simple web searches.
5. **Use `no_cache=false`** (the default) to benefit from free cached results.
6. **URL-encode query parameters** properly. Spaces become `+` or `%20`.
7. **Read the engine schema** from `engines/<engine>.json` when you need to look up available parameters for a specific engine.

## Additional Resources

- For practical curl examples, see [examples.md](examples.md)
- Interactive playground: https://serpapi.com/playground
- Full API docs: https://serpapi.com/search-api
- Account & usage: https://serpapi.com/account-api
