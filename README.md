# Heartline

Heartline is a source-bounded compatibility app that turns public LinkedIn and
Instagram profiles into structured conversation starters and AI-assisted
match evaluations.

It is designed around a simple rule: the analysis agents may use only the
profile data supplied by the configured source scrapers. They do not search
the wider web for additional information about a person.

## What it does

- Imports a verified LinkedIn and Instagram profile pair.
- Scrapes the two public profiles through configured Apify actors.
- Builds a structured profile containing facts, interests, lifestyle signals,
  conversation topics, and clearly marked inferences.
- Lets two independent agents have a five-round conversation.
- Scores the completed date across interests, lifestyle, needs, values,
  communication, and chemistry.
- Caches analyzed profiles and completed dates in `data/cache.json` so repeat
  requests do not need to reprocess the same pair.
- Provides a browser UI for discovery, profile inspection, live dates, and
  completed-match rankings.

## Architecture

```text
data/people.csv
        |
        v
  Apify LinkedIn actor + Apify Instagram actor
        |
        v
  Source normalization
        |
        v
  Gemini or OpenRouter structured analysis
        |
        v
  data/cache.json  <---- browser UI
        |
        v
  Independent-agent date + compatibility score
```

The application is a small Node.js server with a static frontend:

- `server.mjs` handles configuration, source retrieval, model calls, caching,
  validation, and HTTP endpoints.
- `src/main.js` renders the browser experience and consumes the API.
- `src/style.css` contains the Heartline visual system and responsive layout.
- `data/people.csv` contains the demo profile pairs.
- `data/cache.json` is generated locally and is intentionally ignored by Git.

## Requirements

- Node.js 18 or newer
- An Apify API token
- A Gemini API key, or an OpenRouter API key
- Public LinkedIn and Instagram profile URLs for the records being analyzed

Node 18+ is recommended because the server uses the built-in `fetch`,
`AbortSignal.timeout`, and ES modules.

## Installation

Clone the repository and install the project dependencies:

```bash
git clone https://github.com/AmanSingh-tech/cutie.git
cd cutie
npm install
```

This project currently has no runtime npm dependencies, but running
`npm install` keeps the normal Node project workflow available.

## Configuration

Create a local `.env` file in the project root. Environment files are ignored
and must not be committed.

```dotenv
APIFY_TOKEN=your_apify_token
GEMINI_API_KEY=your_gemini_api_key

APIFY_LINKEDIN_ACTOR_ID=automation-lab/linkedin-profile-scraper
APIFY_INSTAGRAM_ACTOR_ID=eiv/instagram-profile-scraper

LLM_PROVIDER=gemini
GEMINI_PROFILE_MODEL=auto
GEMINI_SCORE_MODEL=auto
GEMINI_DATE_MODEL=auto

PORT=3000
```

### Supported model providers

The default provider is Gemini:

```dotenv
LLM_PROVIDER=gemini
GEMINI_API_KEY=your_gemini_api_key
GEMINI_PROFILE_MODEL=auto
GEMINI_SCORE_MODEL=auto
GEMINI_DATE_MODEL=auto
```

Set the model names explicitly when required. With `auto`, Heartline asks the
Gemini Models API for models available to the configured key and chooses the
strongest supported generation model from that response.

OpenRouter is also supported:

```dotenv
LLM_PROVIDER=openrouter
OPENROUTER_API_KEY=your_openrouter_api_key
OPENROUTER_PROFILE_MODEL=openrouter/free
OPENROUTER_SCORE_MODEL=openrouter/free
OPENROUTER_DATE_MODEL=openrouter/free
```

The actor IDs can be changed if you use compatible Apify actors. The server
passes only the supplied profile URL to each actor.

## Demo data

`data/people.csv` must use exactly these headers:

```csv
name,linkedin_url,instagram_url
```

Each row represents one verified profile pair. LinkedIn regional domains such
as `in.linkedin.com`, `ng.linkedin.com`, and `my.linkedin.com` are supported.
The URL validator requires:

- HTTPS
- A LinkedIn `/in/<profile>` URL
- An Instagram profile URL
- No unsupported search, post, or arbitrary page URLs

## Running the application

Start the local server:

```bash
npm run dev
```

Then open [http://localhost:3000](http://localhost:3000).

The server binds to `127.0.0.1` by default. Set `PORT` in `.env` to use a
different local port.

Other available commands:

```bash
# Start the server in production-style mode
npm start

# Check JavaScript syntax
npm run build

# Analyze all 25 demo rows and populate the local cache
npm run validate:demo
```

`npm run validate:demo` expects exactly 25 usable rows in
`data/people.csv`. It retrieves each pair, performs source-bounded analysis,
and writes successful results to `data/cache.json`. It may take time and
consume Apify and model quota.

## API endpoints

### `GET /api/health`

Returns basic configuration status and the selected model names. It does not
return API keys.

### `POST /api/analyze`

Analyzes one profile pair and returns the cached or newly generated profile.

```json
{
  "linkedinUrl": "https://www.linkedin.com/in/example/",
  "instagramUrl": "https://www.instagram.com/example/"
}
```

### `POST /api/date/stream`

Runs a date between two analyzed profile IDs and streams progress as
Server-Sent Events. The completed transcript and evaluation are saved to the
cache for both participants.

```json
{
  "a": "profile-cache-key-a",
  "b": "profile-cache-key-b"
}
```

## Data and privacy boundaries

- API keys are read from the server environment and are never sent to the
  browser.
- `.env`, `.env.example`, and `data/cache.json` are ignored by Git.
- Only the supplied LinkedIn and Instagram URLs are sent to the configured
  Apify actors.
- Model prompts contain normalized data returned by those two sources.
- Facts and inferences are stored separately in the structured profile.
- Unsupported relationship needs or dealbreakers are returned as unknown or
  empty rather than invented.
- The cache contains source snapshots, normalized fields, generated profiles,
  date transcripts, and scores. Treat it as sensitive local application data.

Do not commit API keys, populated environment files, or generated cache data.
If a key is exposed, revoke and replace it immediately.

## Project status and limitations

This is a local/demo application rather than a production deployment. It does
not currently provide user accounts, persistent database storage, background
job orchestration, rate limiting, or production authentication.

The CSV parser is intentionally lightweight and expects simple comma-separated
rows. Values containing commas should be normalized before adding them to the
demo file.

## License

No license has been declared yet. Add a license before distributing the
project or accepting external contributions.
