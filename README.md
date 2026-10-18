# Heartline

## Setup

1. Create a local `.env` file (environment files are intentionally not published) and set `APIFY_TOKEN` and `GEMINI_API_KEY`.
2. Put the verified pairs into `data/people.csv` with exactly these headers: `name,linkedin_url,instagram_url`. Regional LinkedIn subdomains such as `in.linkedin.com` are supported.
3. Run `npm run validate:demo`. This runs each row through only the configured LinkedIn and Instagram actors, performs source-bounded analysis, and writes durable processed entries to `data/cache.json`.
4. Run `npm run dev`, then open `http://localhost:3000`.

`GEMINI_DATE_MODEL=auto` asks the Gemini Models API for models available to the key and selects the strongest supported candidate from that returned list. Set a specific model name to override it.

The cache contains raw source snapshots, normalized source fields, the structured profile, date transcripts, and scores. No source beyond the supplied LinkedIn and Instagram URL is fetched or supplied to the models.
