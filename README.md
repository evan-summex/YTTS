# YTT Summarizer

> Paste a YouTube link, get an AI-generated summary of the video. A small, focused web app built as a hands-on way to learn backend development in Python.

---

## About

This is a lightweight web app: you paste a YouTube URL, hit **Summarize**, and the backend fetches the video's transcript and returns an AI-written summary.

It's intentionally small. The point isn't novelty — it's a **learning project focused specifically on backend development**. The backend (a FastAPI app) is written by hand as a learning exercise; the frontend and project scaffolding were generated with AI assistance so the effort stays concentrated on the backend skill being practiced. If you're browsing this repo, that context explains why the structure is deliberately minimal and why there's a `CLAUDE.md` with unusual tutoring constraints (more on that below).

The hard problem in any "summarize a YouTube video" app — actually *getting* the transcript past YouTube's anti-scraping defenses — is handed off to a third-party API ([Supadata](https://supadata.ai/)), which keeps the backend approachable.

## How It Works

```
┌─────────────────────────────┐
│  Frontend (HTML/CSS/JS)     │
│  URL input + Summarize btn  │
└──────────────┬──────────────┘
               │  POST /summarize  { "url": "..." }
               ▼
┌─────────────────────────────┐
│  FastAPI backend            │
│  1. Receive the URL         │
│  2. Validate it's YouTube   │  ← guardrail: non-YouTube links rejected
│  3. Supadata → transcript   │  ← outbound API call
│  4. Anthropic → summary     │  ← outbound API call
│  5. Return { "summary" }    │
└─────────────────────────────┘
```

1. The frontend sends the pasted URL to the backend.
2. The backend validates that it's a genuine YouTube link and rejects anything else.
3. It calls the Supadata API to retrieve the transcript text.
4. It sends that transcript to the Anthropic API (Claude) to produce a summary.
5. It returns the summary, which the frontend displays.

**API contract** between frontend and backend: the frontend sends `POST /summarize` with a JSON body `{ "url": "..." }`. The backend responds with `{ "summary": "..." }` on success (HTTP 200), or `{ "error": "..." }` with an appropriate error status on failure.

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python + [FastAPI](https://fastapi.tiangolo.com/) |
| ASGI server | [`uvicorn`](https://www.uvicorn.org/) |
| HTTP requests | [`requests`](https://requests.readthedocs.io/) |
| Config / secrets | [`python-dotenv`](https://pypi.org/project/python-dotenv/) |
| Transcripts | [Supadata YouTube Transcript API](https://supadata.ai/) |
| Summarization | [Anthropic API](https://docs.anthropic.com/) (Claude) |
| Frontend | Vanilla HTML / CSS / JavaScript |

## Features

- Paste-and-summarize flow for any public YouTube video with available captions.
- YouTube-only URL guardrail — non-YouTube links are rejected before any API calls are made.
- Transcript fetching handled by Supadata, including AI fallback for videos without native captions.
- API keys kept server-side only; never exposed to the browser.

## Prerequisites

- **Python 3.10+**
- A **Supadata API key** — sign up at [supadata.ai](https://supadata.ai/); a free tier (100 requests/month, no credit card) is available to get started.
- An **Anthropic API key** — from the [Anthropic Console](https://console.anthropic.com/).

## Getting Started

```bash
# 1. Clone the repo (using SSH)
git clone git@github.com:evan-summex/YTTS.git
cd YTTS

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set up your environment variables
cp .env.example .env            # then edit .env and add your keys

# 5. Run the app
uvicorn app:app --reload
```

Then open the app in your browser (default `http://127.0.0.1:8000`).

## Environment Variables

Copy `.env.example` to `.env` and fill in your own values. **The `.env` file is git-ignored and must never be committed.**

| Variable | Description |
|---|---|
| `SUPADATA_API_KEY` | Your Supadata API key (passed to Supadata as the `x-api-key` header). |
| `ANTHROPIC_API_KEY` | Your Anthropic API key. |

## Project Structure

```
YTTS/
├── app.py              # FastAPI backend (the hand-written learning core)
├── requirements.txt    # Python dependencies
├── .env.example        # Template for required environment variables
├── .env                # Your real keys — git-ignored, never committed
├── .gitignore
├── static/             # Frontend assets (HTML/CSS/JS)
├── CLAUDE.md           # Tutoring config for Claude Code (see below)
└── README.md
```

*(Structure may shift as the backend is built out.)*

## API Notes

- **Supadata** exposes a REST endpoint at `https://api.supadata.ai/v1/youtube/transcript` and authenticates via an `x-api-key` request header. It's rate-limited (roughly 5 requests per 10 seconds on the free tier) and falls back to AI transcription for videos that lack native captions. Always check the [current Supadata docs](https://docs.supadata.ai/) — limits and pricing change.
- **Anthropic** is called server-side with your API key; the transcript text is sent as the input and a summary is returned. See the [Anthropic API docs](https://docs.anthropic.com/).

## Security Notes

- Both API keys live **only** on the backend, loaded from environment variables. They are never shipped to or referenced by the frontend.
- `.env` is git-ignored. Only `.env.example` (with placeholder values) is committed.
- If you fork or clone this, **use your own keys** and never commit them. If a key is ever exposed, rotate it immediately.

## Development Approach

This repo includes a `CLAUDE.md` configured to make [Claude Code](https://www.anthropic.com/claude-code) act as a **backend tutor rather than an autocomplete**. It's set up to build the frontend, scaffolding, and config, but to *guide* the backend with explanations and escalating hints instead of writing the backend code. That's a deliberate choice for the learning goals of this project — the backend is meant to be typed by hand.

## Status & Roadmap

🚧 **Learning in progress.** This is a personal learning build, not a polished product. There's no urgency to ship — the existing tool it's modeled on already works.

**Current state:** The frontend mockup, project scaffolding (`requirements.txt`, `.env.example`), and the API data contract are in place. The frontend currently runs in **mockup mode** — it displays hardcoded sample output so the finished look and feel is visible, and is not yet wired to a live backend. The FastAPI backend (`app.py`) is the next piece, hand-written as the core learning exercise.

Possible future additions:

- [ ] Error handling for missing transcripts, invalid URLs, and API failures
- [ ] Selectable summary length / style
- [ ] Support for additional transcript sources or platforms
- [ ] Basic caching to avoid re-summarizing the same video

## License

This project is licensed under the **GNU Affero General Public License v3.0** (AGPL-3.0), 19 November 2007.

The AGPL-3.0 is a strong copyleft license. In short: you're free to use, study, modify, and redistribute this software, but any derivative work must also be released under the AGPL-3.0 — and crucially, if you run a modified version as a network service, you must make the corresponding source available to its users. See the full text at <https://www.gnu.org/licenses/agpl-3.0.html>.

A complete copy of the license should accompany this repository in a `LICENSE` file.

## Acknowledgments

- [Supadata](https://supadata.ai/) for transcript retrieval.
- [Anthropic](https://www.anthropic.com/) for the Claude API used to generate summaries.
