# PRD: Monster Roommates

**Working title:** Monster Roommates | **Event:** run.creators hackathon (Oct 8 to Oct 28, 2026) | **Team:** 5 builders | **Doc status:** DRAFT v1.0 (Oct 14, 2026)

> Companion docs: [`README.md`](./README.md) (how to run it) and [`SHARED_CONTRACT.md`](./SHARED_CONTRACT.md) (every interface between modules). If this PRD and the contract disagree on an interface, the contract wins.

---

## 1. Overview

**One-line pitch:** Hide your monster from the landlord, share your layout, and challenge a friend to find it.

You run a shared apartment where you must hide monster roommates from a nosy landlord. Each monster has quirks that leak clues (slime trails, glowing, noise). You place monsters to survive the inspection, then turn your hiding layout into a shareable puzzle that friends try to crack, or let an AI landlord try to catch you.

### 1.1 Goals

1. **Win the hackathon:** one finished, polished loop with a clear differentiator and a 60-second demo.
2. **Publish it:** a public link anyone can play, even if we do not win.
3. **Show real engineering depth:** modular architecture, a real backend, and an agentic AI opponent with a safe fallback.

### 1.2 Non-goals (explicitly out of scope)

- Real payments (the cosmetic shop is a mock UI only)
- User accounts, passwords, or social login
- Real-time multiplayer
- RAG or a knowledge base (decided: no natural fit)
- Native mobile apps (browser only; mobile must be playable but is not the priority)

### 1.3 Differentiator

The hiding layout becomes a **shareable puzzle**, and the landlord can be an **AI agent that adapts to your habits**. `[Guessing]` The claim that most cozy monster games are single-player is unverified. Search for similar games before using that line in the pitch.

---

## 2. Users

| User | Needs | How we serve them |
|---|---|---|
| Casual player (friends sharing links) | Quick, charming, no sign-up | Browser game, instant play, share link |
| Hackathon judge | Clear idea, working demo, polish, technical depth | 60-second demo path, public URL, visible AI agent reasoning |
| Returning player | A reason to come back | Daily layout seed, leaderboard, unlockable monsters |

Judging criteria and submission rules have been read by the team and nothing blocks this plan. Confirm the exact submission time and time zone.

---

## 3. Core loop

1. **Plan:** place 2 to 4 monsters in rooms of the apartment. Each monster has a quirk that produces clues.
2. **Inspect:** the landlord walks the apartment. Clues raise the suspicion bar.
3. **Survive:** win the day to unlock monsters and rooms; lose too much suspicion and it is game over.
4. **Share:** your layout becomes a link. A friend gets 3 wrong guesses to find each hidden monster, or you watch the AI landlord try.

---

## 4. Features and priorities

Priority: **MUST** (game is broken without it), **SHOULD** (strongly wanted), **COULD** (stretch, cut first).

| # | Feature | Priority | Module |
|---|---|---|---|
| F1 | 5 monsters with distinct design, quirk, movement and ability | MUST | M1 |
| F2 | Apartment with 3 to 5 rooms; place monsters in spots | MUST | M2 |
| F3 | Rule-based landlord inspection, suspicion bar, win/lose screens | MUST | M2 |
| F4 | 3 levels, each with 1 secret room or trap | SHOULD | M2 |
| F5 | Pumpkin Inspector boss level with a learnable pattern and finishing moment | SHOULD | M3 |
| F6 | Share link and friend challenge (3 wrong guesses, hints) | MUST | M4 |
| F7 | Offline share link (no server needed) | MUST | M4 |
| F8 | AI landlord agent (Gemini, tool calling) with rule-based fallback | SHOULD | M4 |
| F9 | Daily seed and leaderboard | SHOULD | M4 / M5 |
| F10 | Monster fusion (2 monsters become a new species with different powers) | COULD | M1 |
| F11 | AI landlord memory of the player's hiding habits | COULD | M4 |
| F12 | Hub, tutorial, audio, settings, polish, deploy | MUST | M5 |
| F13 | Mock cosmetic shop | COULD | M5 |

### 4.1 How the five "monster game" points map to this game

| Point | Where it lives |
|---|---|
| Memorable monsters | F1: roster of 5, each unique (section 5.1) |
| Unexpected transformations | F10: fusion changes powers, not just colour (contract section 2) |
| Epic boss battles | F5: Pumpkin Inspector, learnable pattern, big finishing moment |
| New surprise per level | F4: one secret room or trap per level; hidden monsters on replay |
| Professional finishing | F12: menus, health/suspicion bar, particles, screen shake, SFX, win/lose screens |

---

## 5. Game design summary

### 5.1 Monsters (full data in the contract)

| Monster | Look | Quirk | Movement | Ability |
|---|---|---|---|---|
| Jelly | Green blob | Leaves a slime trail | Bounces | Squeezes through vents |
| Sockette | Fuzzy sock-eater | Eats socks, removing clutter | Waddles | Clears cover |
| Glimmer | Glow-worm | Glows in dark rooms | Floats | Lights a room |
| Shade | Shadow | Vanishes in deep darkness | Teleports | Shadow blink |
| Pip | Tiny | Disappears into clutter | Scurries | Distracts the boss |

### 5.2 Rooms and clues

Rooms have `dark`, `clutter`, `noise_masking` and a number of `spots`. Monsters emit four clue types (**sound, glow, trail, sight**), each level 0 to 3, computed by the deterministic algorithm in contract section 5.

### 5.3 Boss: Pumpkin Inspector

A standalone level. The boss cycles through an inspection pattern the player must learn (for example: checks left door, then window, then closet). Hiding at the right moment avoids detection. Pip can distract the boss to open a window. Ending: dramatic door slam, screen shake, monster pop-out.

### 5.4 Challenge Arena

- **Create:** pick a layout, get a link.
- **Play:** a friend guesses where each monster is. A wrong guess reveals the guessed room's clues as a hint.
- **AI inspect:** an LLM-driven landlord investigates your layout using tools and tells you what it found and why.

---

## 6. Modules, ownership and acceptance criteria

Each module is an **independent, end-to-end feature** (UI, logic and data), runnable from its own test page. Modules interact only through the contract.

### M1: Monster Lab | Owner: ______

Roster screen, sprites and animations, quirk previews, fusion (stretch). **Done when:**
- [ ] 5 monsters have idle, move, hide and caught animations
- [ ] Each monster's quirk preview works in the Lab
- [ ] Roster data is read from `monsters.json` (no hardcoded stats)
- [ ] (Stretch) One fusion produces a new species with changed powers
- [ ] Test page `web/test/lab.html` runs with no other module

### M2: Hide & Inspect | Owner: ______

The core puzzle. **Done when:**
- [ ] Apartment renders rooms with spots; monsters can be placed and moved
- [ ] Rule-based landlord inspects using the contract clue algorithm; passes vectors V1 to V5
- [ ] Suspicion bar, win and lose screens work; 3 levels exist
- [ ] Produces a valid Layout and returns the `hide` exit result
- [ ] Test page `web/test/hide.html` runs with no other module and no API

### M3: Boss Mode | Owner: ______

Pumpkin Inspector. **Done when:**
- [ ] Boss has a readable, learnable pattern and a health/suspicion indicator
- [ ] Player can win and lose; finishing moment includes screen shake and particles
- [ ] Pip's distraction ability works
- [ ] Returns the `boss` exit result; test page `web/test/boss.html` runs alone

### M4: Challenge Arena (backend + AI landlord) | Owner: ______

FastAPI service plus the challenge screen. **Done when:**
- [ ] Every endpoint in contract section 9 works as specified, with the error format
- [ ] A challenge created in one browser can be played in another
- [ ] Offline links (contract section 10) work with the API stopped
- [ ] AI landlord uses tool calling (`inspect_room`, `look_closer`, `accuse`) and never sees the layout
- [ ] Fallback landlord returns the identical response shape on timeout, error or cap
- [ ] Passes clue vectors V1 to V5 in `pytest`
- [ ] Rate limits and the daily LLM cap are active
- [ ] `apiClient.js` never throws; the game works with the API down

**Build order inside M4 (protect the differentiator first):**

1. Offline share link and arena screen (no backend), days 1 to 4
2. FastAPI: players, challenges, guesses, daily, leaderboard
3. Fallback landlord (rule-based), then the Gemini agent loop
4. Memory of player habits (COULD)

**Backend design notes (private to M4):**

- **Stack:** Python, FastAPI, Pydantic, SQLAlchemy, SQLite (dev) or Postgres (deploy). Flask is acceptable if the team prefers it, but FastAPI is recommended for typed schemas and auto docs.
- **Agent loop:** a plain tool-calling loop around the Gemini SDK, max 6 steps, JSON-schema validation of every tool call, 6-second timeout. No agent framework required.
- **Tables:** `players`, `challenges`, `guesses`, `scores`, `inspections` (log with engine, result, latency), `landlord_memory` (stretch).
- **Metrics to log:** landlord win rate per monster and per layout. This gives a pitch line such as "Pip beats the AI landlord 70% of the time".
- **Free-tier note:** `[Guessing]` Gemini's free tier has request limits and may allow Google to use submitted data to improve its products. Verify the current limits and terms, never send personal data, and design for the cap (fallback engine, caching).

### M5: Hub & Polish | Owner: ______

**Done when:**
- [ ] Main menu launches all four modules via the contract interface and handles `module:exit` and `module:error`
- [ ] Hub owns the profile in `localStorage` (contract section 8) and unlock logic
- [ ] Hub handles share-link hashes on boot (contract section 10)
- [ ] Real `audio.js` implementation; tutorial; settings (mute, reduced motion)
- [ ] (Stretch) Mock cosmetic shop; daily seed screen
- [ ] Public deployment of the game and the API; itch.io page; 60 to 90 second demo video

---

## 7. Architecture

```
            +--------------------------- Browser (Phaser 3 + Vite) ---------------------------+
            |  Hub (M5)  --launch/exit-->  Lab (M1) | Hide (M2) | Boss (M3) | Arena UI (M4)   |
            |       |                                                      |                  |
            |  localStorage (profile, hub only)                       apiClient.js            |
            +----------------------------------------------------------------|---------------+
                                                                             | HTTPS (JSON)
                                                          +------------------v---------------+
                                                          |   FastAPI service (M4) /v1       |
                                                          |  routers -> services -> database |
                                                          |        |                         |
                                                          |   Landlord agent --> Gemini API  |
                                                          |   Fallback landlord (rules)      |
                                                          +----------------------------------+
```

- **Data flow:** `hide` produces a Layout, the hub passes it to `arena`, `arena` sends it to the API (or encodes it into a URL).
- **Failure mode:** if the API or Gemini is down, the game still plays: offline links and the rule-based landlord cover everything.

---

## 8. Tech stack

| Layer | Choice | Notes |
|---|---|---|
| Game | Phaser 3, JavaScript, Vite | Tweens, particles, camera shake, audio built in |
| UI overlays | Plain HTML + CSS | No React |
| Backend | Python, FastAPI, Pydantic | Flask allowed as an alternative |
| Database | SQLite (dev), Postgres/Supabase (deploy) | No MongoDB needed |
| LLM | Google Gemini (free tier) via official Python SDK | Model name set by env var; verify current free-tier model and limits |
| Rate limiting | slowapi or equivalent | Per IP and per player |
| Hosting | Vercel or Netlify (game), free Python host (API), itch.io page | Verify host terms and cold-start behaviour |
| Collaboration | GitHub (branch per module), Discord | Daily merge to `main` |
| Art and audio | CC0 packs, Piskel/LibreSprite, sfxr-style SFX | Check licenses and the hackathon's AI-asset rules |

---

## 9. Minimum project structure

This is the **minimum required** layout. Files marked `*` are required for the app to run; everything else can be added later. Do not rename top-level folders: the README, contract and deploy scripts depend on them.

```
monster-roommates/
├── README.md                      *
├── .gitignore                     *   (node_modules, .venv, .env, dist, data/*.db)
├── docs/
│   ├── PRD.md                     *
│   └── SHARED_CONTRACT.md         *
├── contracts/                     *   FROZEN (see contract rules)
│   ├── monsters.json              *
│   ├── rooms.json                 *
│   ├── layout.schema.json         *
│   └── examples/
│       ├── layout.sample.json     *
│       └── clue_vectors.json      *
├── web/                           *   Phaser 3 + Vite (JavaScript)
│   ├── index.html                 *
│   ├── package.json               *
│   ├── vite.config.js             *   (multi-page: index + test pages)
│   ├── .env.example                   (VITE_API_BASE_URL)
│   ├── public/assets/
│   │   ├── sprites/                   monster_<id>_<anim>.png, room_<id>.png
│   │   └── audio/                     sfx_<name>.ogg, music_<name>.ogg
│   ├── test/                      *   one page per module
│   │   ├── lab.html               *
│   │   ├── hide.html              *
│   │   ├── boss.html              *
│   │   └── arena.html             *
│   └── src/
│       ├── main.js                *   boots Phaser, registers module scenes
│       ├── shared/                *   small, frozen utilities
│       │   ├── contracts.js       *
│       │   ├── constants.js       *
│       │   ├── audio.js           *   no-op stub until M5 fills it in
│       │   └── apiClient.js       *
│       ├── hub/                   *   M5: menu, router, profile, settings
│       │   └── HubScene.js        *
│       └── modules/
│           ├── lab/    (index.js *, LabScene.js *)      M1
│           ├── hide/   (index.js *, HideScene.js *)     M2
│           ├── boss/   (index.js *, BossScene.js *)     M3
│           └── arena/  (index.js *, ArenaScene.js *)    M4 UI
├── api/                           *   FastAPI (Python)  M4
│   ├── requirements.txt           *
│   ├── .env.example               *   (see README)
│   ├── app/
│   │   ├── main.py                *   app, CORS, error handlers, routers
│   │   ├── config.py              *   env settings
│   │   ├── db.py                  *   engine and session
│   │   ├── models.py              *   SQLAlchemy tables
│   │   ├── schemas.py             *   Pydantic models matching the contract
│   │   ├── routers/
│   │   │   ├── health.py          *
│   │   │   ├── players.py         *
│   │   │   ├── challenges.py      *
│   │   │   ├── landlord.py        *
│   │   │   ├── daily.py               (can live in leaderboard.py)
│   │   │   └── leaderboard.py     *
│   │   └── services/
│   │       ├── clues.py           *   implements contract section 5
│   │       ├── fallback_landlord.py *
│   │       ├── landlord_agent.py      Gemini tool-calling loop
│   │       └── memory.py              (stretch)
│   └── tests/
│       └── test_clues.py          *   runs vectors V1 to V5
└── .github/                           (optional)
    └── pull_request_template.md
```

**Module boundary rules (enforced in code review):**

- A file in `web/src/modules/<x>/` may import only from its own folder, `web/src/shared/`, and `contracts/`.
- Only `web/src/hub/` reads or writes `localStorage`.
- Only `api/` talks to Gemini. The browser never sees the API key.

---

## 10. Non-functional requirements

| Area | Requirement |
|---|---|
| Performance | 60 fps on a mid-range laptop; first load under 5 seconds on broadband; total initial download under 10 MB |
| Compatibility | Latest Chrome, Edge, Firefox, Safari; playable on a phone in landscape |
| Accessibility | Clues use icons plus color; mute toggle; reduced-motion setting disables screen shake |
| Resilience | Game fully playable if the API or Gemini is down |
| Security and cost | API key server-side only; rate limits; response caching; hard daily LLM cap; validate every layout against the schema |
| Privacy | Anonymous players; no personal data stored or sent to the LLM |
| Licensing | Every asset has a recorded license in `web/public/assets/CREDITS.md` |

---

## 11. Success metrics

| Metric | Target |
|---|---|
| Public playable URL working on desktop and phone | Yes, by Oct 26 |
| Demo path completes in under 60 seconds | Yes |
| Outside playtesters complete one full day | 5 people |
| Challenge links created and solved in playtests | At least 3 |
| AI landlord returns a valid response, or falls back | 100% (never an error screen) |
| Console errors on a full playthrough | 0 |

---

## 12. Timeline and gates (Oct 14 to Oct 28)

| Dates | Goal | Gate |
|---|---|---|
| Oct 14-15 | Lock the idea; freeze the contract; create repo; each module's test page runs | Everyone runs their test page |
| Oct 16-20 | Build modules independently; Oct 18 checkpoint: each demoable alone | Module "done when" items met |
| Oct 21-22 | Hub integration | **First playable end to end by Oct 22 (hard gate)** |
| Oct 23-24 | AI landlord wired in; boss polish; stretch only if stable | Feature freeze, end of Oct 24 |
| Oct 25-26 | Polish, bug bash, 5 outside playtests, deploy game and API | Public link works |
| Oct 27 | Demo video, screenshots, submission text, submit | Submitted |
| Oct 28 | Buffer only | |

---

## 13. Risks and mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Integration failures | High | Frozen contract, test pages, Oct 22 hard gate |
| Gemini free-tier limits or outage | Medium | Fallback landlord, caching, daily cap, offline links |
| Free API host sleeps or changes terms | Medium | Test host before Oct 24; offline mode keeps the game playable |
| Beginner velocity | Medium | Small scope per module; stretch features cut first |
| Scope creep | High | Stretch only after the first playable exists |
| Overlap between M2's landlord and M4's AI landlord | Medium | M2 owns the rule-based landlord; M4's is an optional opponent behind the contract |
| Asset licensing problems | Low | Use CC0 assets; log credits |

**Cut order if behind:** mock shop, fusion polish, secret rooms, daily seed, AI landlord memory. **Never cut:** 5 monsters with personality, the hiding puzzle, the inspection, the share link, the boss, polish.

---

## 14. Open questions and assumptions

1. Exact submission deadline time and time zone (confirm today).
2. Differentiator uniqueness needs a quick competitor search before it goes in the pitch.
3. Current Gemini free-tier model name, rate limits, and data-use terms (verify in Google AI Studio).
4. Which free host will serve the API, and how it handles cold starts.
5. Module owners for M1 to M5 (fill in the headings above).

---

## 15. Demo script (60 seconds)

1. Hook: "Hide your monster. Share the layout. Challenge a friend."
2. Place two monsters and show their quirks (slime trail, glow).
3. Run the inspection, then the Pumpkin Inspector: suspicion bar, screen shake, finishing moment.
4. Create a challenge link, open it in a second window, make a guess and show the hint.
5. Show the AI landlord inspecting your layout with its reasoning visible.
6. End on the public URL: "Play it right now."
