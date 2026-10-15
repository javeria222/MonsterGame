# Monster Roommates

> Hide your monster from the landlord. Share the layout. Challenge a friend.

A cozy monster-hiding puzzle for the **run.creators hackathon** (Oct 8 to Oct 28, 2026). You run a shared apartment and must hide quirky monster roommates from a nosy landlord. Each monster leaks clues (slime trails, glowing, noise), so placement is a puzzle. Then turn your layout into a link for friends to crack, or let an **AI landlord agent** try to catch you.

**Play it:** _(public URL goes here after deploy)_ | **itch.io:** _(link here)_ | **Demo video:** _(link here)_

**Status:** hackathon build in progress.

---

## Docs (read these first)

| Doc | What it is |
|---|---|
| [`docs/PRD.md`](./docs/PRD.md) | What we are building, scope, module acceptance criteria, project structure, timeline |
| [`docs/SHARED_CONTRACT.md`](./docs/SHARED_CONTRACT.md) | Every interface between modules: data files, API endpoints, outputs. **If it disagrees with anything else, the contract wins.** |

---

## Features

- 5 monsters, each with its own look, quirk, movement and ability (stretch: fusion into new species)
- Apartment hiding puzzle with a clue system and a rule-based landlord
- Pumpkin Inspector boss level with a learnable pattern
- Share links and friend challenges (works **without a server** too)
- AI landlord agent (Google Gemini, tool calling) with a rule-based fallback
- Daily seed and leaderboard

---

## Modules (each is an independent end-to-end feature)

| Module | Feature | Folder | Owner |
|---|---|---|---|
| M1 | Monster Lab | `web/src/modules/lab` | ______ |
| M2 | Hide & Inspect (core puzzle) | `web/src/modules/hide` | ______ |
| M3 | Boss Mode | `web/src/modules/boss` | ______ |
| M4 | Challenge Arena + Backend + AI landlord | `web/src/modules/arena`, `api/` | ______ |
| M5 | Hub, polish, audio, deploy | `web/src/hub`, `web/src/shared/audio.js` | ______ |

Modules never import each other. They meet only through the contract and the hub.

---

## Tech stack

- **Game:** Phaser 3, JavaScript, Vite
- **Backend:** Python, FastAPI, Pydantic, SQLAlchemy, SQLite (Postgres when deployed)
- **AI:** Google Gemini (free tier) via the official Python SDK
- **Hosting:** Vercel or Netlify (game), a free Python host (API), itch.io (game page)

---

## Quick start

### Prerequisites

- Node.js 18 or newer
- Python 3.10 or newer
- A Gemini API key from Google AI Studio (only needed for the AI landlord; the game runs without it)

### 1. Clone

```bash
git clone <repo-url> monster-roommates
cd monster-roommates
```

### 2. Run the game (web)

```bash
cd web
cp .env.example .env        # set VITE_API_BASE_URL if you run the API
npm install
npm run dev                 # opens http://localhost:5173
```

### 3. Run the API (optional, needed for server challenges and the AI landlord)

```bash
cd api
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env        # add GEMINI_API_KEY
uvicorn app.main:app --reload --port 8000
```

Interactive API docs: <http://localhost:8000/docs>

If the API is not running, the game still works: offline share links and the rule-based landlord cover everything.

### 4. Run one module on its own

While `npm run dev` is running, open the module's test page:

| Module | URL |
|---|---|
| Monster Lab | <http://localhost:5173/test/lab.html> |
| Hide & Inspect | <http://localhost:5173/test/hide.html> |
| Boss Mode | <http://localhost:5173/test/boss.html> |
| Challenge Arena | <http://localhost:5173/test/arena.html> |

Each page boots only that module with fixture data and logs its exit result to the browser console.

---

## Environment variables

### `api/.env`

| Variable | Example | Purpose |
|---|---|---|
| `GEMINI_API_KEY` | `your-key` | Gemini access. Never commit this file |
| `GEMINI_MODEL` | _(verify the current free-tier model in Google AI Studio)_ | Model used by the AI landlord |
| `DATABASE_URL` | `sqlite:///./data/app.db` | Database connection |
| `ALLOWED_ORIGINS` | `http://localhost:5173` | CORS origins, comma separated |
| `DAILY_LLM_CALL_CAP` | `300` | Hard daily limit on Gemini calls; the fallback landlord takes over after it |
| `LLM_TIMEOUT_SECONDS` | `6` | Gemini call timeout before falling back |

### `web/.env`

| Variable | Example | Purpose |
|---|---|---|
| `VITE_API_BASE_URL` | `http://localhost:8000/v1` | Where the game finds the API |

---

## Testing

```bash
# API: includes the clue-generation golden vectors (V1 to V5)
cd api && pytest

# Web: manual check via the test pages above, plus a full playthrough from the hub
```

Both the JavaScript clue code (M2) and the Python clue code (M4) must pass the vectors in `contracts/examples/clue_vectors.json`.

---

## Project structure (short version)

```
monster-roommates/
├── docs/            PRD.md, SHARED_CONTRACT.md
├── contracts/       monsters.json, rooms.json, layout.schema.json (FROZEN)
├── web/             Phaser 3 + Vite game
│   ├── test/        one test page per module
│   └── src/         main.js, hub/, modules/, shared/
└── api/             FastAPI backend (app/, tests/)
```

The full required structure is in the PRD, section 9.

---

## Working together

1. **One branch per module:** `m1-lab`, `m2-hide`, `m3-boss`, `m4-arena`, `m5-hub`.
2. **Merge to `main` daily** with small commits and a short description. Do not push broken builds to `main`.
3. **Stay inside your folder.** Edits outside it need a heads-up in Discord.
4. **Never edit `contracts/` alone.** Changing a frozen file needs all five owners to agree, a version bump, and a line in the contract changelog.
5. **After Oct 22:** only backward-compatible contract changes.
6. **Definition of done:** your module meets its acceptance criteria in the PRD and runs from its own test page.

### Key dates

| Date | Milestone |
|---|---|
| Oct 15 | Contract frozen |
| Oct 18 | Every module demoable alone |
| **Oct 22** | **First playable end to end (hard gate)** |
| Oct 24 | Feature freeze |
| Oct 26 | Polished and deployed |
| Oct 27 | Submit |
| Oct 28 | Buffer only |

Confirm the exact submission time and time zone.

---

## Deployment

- **Game:** build with `cd web && npm run build`, deploy `web/dist` to Vercel or Netlify, then upload the same build to itch.io as an HTML5 game.
- **API:** deploy `api/` to a free Python host, set the env vars above, and add the game's URL to `ALLOWED_ORIGINS`. Test the host's cold-start behaviour before Oct 24.
- Set `VITE_API_BASE_URL` to the deployed API before building the web app.

---

## Credits and licenses

- Every asset's source and license is recorded in `web/public/assets/CREDITS.md`.
- Use only assets you are allowed to redistribute (CC0 or properly licensed).
- Code license: _(choose one, for example MIT)_.

## Team

_(Add names and module owners here.)_
