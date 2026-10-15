# Shared Contract: Monster Roommates

**Contract version:** `1.0.0` | **Status:** DRAFT until frozen on Day 1 (Oct 15, 2026) | **Applies to:** M1, M2, M3, M4, M5

This is the single source of truth for everything that crosses a module boundary. If it is not in this file, it is **private to a module** and the owner can change it freely.

---

## 0. Rules of the contract

1. No module imports another module's code. Modules talk only through this contract.
2. Allowed imports for any web module: `web/src/shared/*` and `contracts/*`. Nothing else outside the module's own folder.
3. Every module must run from its own test page using fixtures from `contracts/examples/`. If it needs another module to run, it is not done.
4. **Frozen files** (copy the JSON blocks below into `contracts/`): `monsters.json`, `rooms.json`, `layout.schema.json`.
5. Changing a frozen file or any section here needs approval from **all five owners**, a version bump, and a changelog line (section 11).
6. After **Oct 22**: only additive, backward-compatible changes (new optional fields). No breaking changes.
7. If two docs disagree, this file wins.

---

## 1. Conventions

| Item | Rule |
|---|---|
| IDs | lowercase `snake_case` for monsters/rooms (`living_room`); `player_id` = UUIDv4; `challenge_id` = 8 chars, lowercase base32 (`a-z2-7`) |
| Time | ISO 8601 UTC strings (`2026-10-20T14:03:00Z`) |
| Dates / seeds | `YYYY-MM-DD` in **UTC** |
| JSON | `snake_case` keys, UTF-8, no comments in real files |
| Numbers | levels and traits are integers; scores are integers 0 to 1,000,000 |
| Canvas | 960 x 540 base resolution, Phaser scale mode `FIT`, centered |
| Palette | Purple `#2B1B4A`, Orange `#E8741C`, Cream `#FFF4E0`, Mint `#7FD1AE`, Slime `#8BD450`, Shadow `#1A1226` |
| Clue display | Never rely on color alone: every clue has an icon (accessibility) |

---

## 2. `monsters.json` (frozen)

Traits are integers **0 to 3**. `modifiers` use only the operators defined in section 5.

```json
{
  "contract_version": "1.0.0",
  "monsters": [
    {
      "id": "jelly", "name": "Jelly", "size": "medium", "fused": false,
      "traits": { "noise": 1, "light": 0, "trail": 2, "visibility": 2 },
      "movement": "bounce",
      "quirk": "Leaves a slime trail that clutter can hide.",
      "ability": "squeeze_through_vents",
      "modifiers": []
    },
    {
      "id": "sockette", "name": "Sockette", "size": "small", "fused": false,
      "traits": { "noise": 2, "light": 0, "trail": 0, "visibility": 2 },
      "movement": "waddle",
      "quirk": "Eats socks, which removes clutter from its room.",
      "ability": "clear_cover",
      "modifiers": [ { "effect": "room_clutter_delta", "value": -1 } ]
    },
    {
      "id": "glimmer", "name": "Glimmer", "size": "small", "fused": false,
      "traits": { "noise": 0, "light": 3, "trail": 0, "visibility": 1 },
      "movement": "float",
      "quirk": "Glows brightly in dark rooms.",
      "ability": "light_room",
      "modifiers": []
    },
    {
      "id": "shade", "name": "Shade", "size": "medium", "fused": false,
      "traits": { "noise": 1, "light": 0, "trail": 0, "visibility": 2 },
      "movement": "teleport",
      "quirk": "Vanishes completely in deep darkness.",
      "ability": "shadow_blink",
      "modifiers": [ { "when": { "room_dark_gte": 2 }, "set": { "visibility": 0, "noise": 0 } } ]
    },
    {
      "id": "pip", "name": "Pip", "size": "tiny", "fused": false,
      "traits": { "noise": 0, "light": 0, "trail": 0, "visibility": 1 },
      "movement": "scurry",
      "quirk": "So small it disappears into any clutter.",
      "ability": "distract_giant",
      "modifiers": [ { "when": { "room_clutter_gte": 1 }, "set": { "visibility": 0 } } ]
    }
  ],
  "fusions": [
    { "a": "jelly", "b": "shade", "result": "gloomjelly" },
    { "a": "sockette", "b": "glimmer", "result": "sparksock" }
  ],
  "fused_monsters": [
    {
      "id": "gloomjelly", "name": "Gloomjelly", "size": "medium", "fused": true,
      "traits": { "noise": 1, "light": 0, "trail": 1, "visibility": 1 },
      "movement": "ooze_teleport",
      "quirk": "Oozes through shadows; nearly invisible in any dim room.",
      "ability": "slip_between_rooms",
      "modifiers": [ { "when": { "room_dark_gte": 1 }, "set": { "visibility": 0 } } ]
    },
    {
      "id": "sparksock", "name": "Sparksock", "size": "small", "fused": true,
      "traits": { "noise": 2, "light": 2, "trail": 0, "visibility": 2 },
      "movement": "spark_hop",
      "quirk": "Eats socks and throws sparks while doing it.",
      "ability": "chain_flash",
      "modifiers": [ { "effect": "room_clutter_delta", "value": -1 } ]
    }
  ]
}
```

Fusion is a **stretch feature** owned by M1. Fused monsters use the same schema, so any module can handle them without special cases.

---

## 3. `rooms.json` (frozen)

Room values are integers **0 to 3** except `spots`.

```json
{
  "contract_version": "1.0.0",
  "rooms": [
    { "id": "kitchen",     "name": "Kitchen",     "dark": 0, "clutter": 2, "noise_masking": 2, "spots": 4 },
    { "id": "bathroom",    "name": "Bathroom",    "dark": 1, "clutter": 1, "noise_masking": 2, "spots": 3 },
    { "id": "bedroom",     "name": "Bedroom",     "dark": 2, "clutter": 2, "noise_masking": 0, "spots": 4 },
    { "id": "living_room", "name": "Living Room", "dark": 0, "clutter": 1, "noise_masking": 1, "spots": 4 },
    { "id": "attic",       "name": "Attic",       "dark": 3, "clutter": 3, "noise_masking": 0, "spots": 3 },
    { "id": "basement",    "name": "Basement",    "dark": 3, "clutter": 2, "noise_masking": 1, "spots": 3 }
  ],
  "levels": [
    { "level": 1, "rooms": ["kitchen", "bathroom", "bedroom"], "monster_count": 2 },
    { "level": 2, "rooms": ["kitchen", "bathroom", "bedroom", "living_room"], "monster_count": 3 },
    { "level": 3, "rooms": ["kitchen", "bathroom", "bedroom", "living_room", "attic"], "monster_count": 4 }
  ]
}
```

`basement` is reserved for secret-room and trap content (M2, private). It is valid in a layout but not part of the three standard levels.

---

## 4. `layout.schema.json` (frozen)

A **Layout** is the one object that describes a hiding setup. M2 produces it, M4 stores and judges it, M5 and the hub pass it around.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "layout.schema.json",
  "title": "Layout",
  "type": "object",
  "additionalProperties": false,
  "required": ["schema_version", "level", "rooms", "placements"],
  "properties": {
    "schema_version": { "const": 1 },
    "level": { "type": "integer", "minimum": 1, "maximum": 10 },
    "seed": { "type": "string", "pattern": "^[0-9]{4}-[0-9]{2}-[0-9]{2}$|^[a-z0-9_-]{1,32}$" },
    "rooms": {
      "type": "array", "minItems": 2, "maxItems": 6, "uniqueItems": true,
      "items": { "enum": ["kitchen", "bathroom", "bedroom", "living_room", "attic", "basement"] }
    },
    "placements": {
      "type": "array", "minItems": 1, "maxItems": 4,
      "items": { "$ref": "#/$defs/placement" }
    }
  },
  "$defs": {
    "placement": {
      "type": "object",
      "additionalProperties": false,
      "required": ["monster", "room", "spot"],
      "properties": {
        "monster": { "type": "string", "pattern": "^[a-z][a-z0-9_]{1,24}$" },
        "room": { "enum": ["kitchen", "bathroom", "bedroom", "living_room", "attic", "basement"] },
        "spot": { "type": "integer", "minimum": 0, "maximum": 3 }
      }
    }
  }
}
```

### 4.1 Extra validation rules (JSON Schema cannot express these; implement them in code)

1. Every `placements[].room` must appear in `rooms`.
2. `spot` must be `< spots` of that room (from `rooms.json`).
3. Each `(room, spot)` pair is used at most once.
4. Each `monster` id appears at most once and must exist in `monsters.json` (base or fused).
5. For a **standard level**, `rooms` must match that level and `len(placements) <= monster_count`.

### 4.2 Example (`contracts/examples/layout.sample.json`)

```json
{
  "schema_version": 1,
  "level": 2,
  "seed": "2026-10-20",
  "rooms": ["kitchen", "bathroom", "bedroom", "living_room"],
  "placements": [
    { "monster": "jelly",    "room": "bathroom", "spot": 2 },
    { "monster": "glimmer",  "room": "bedroom",  "spot": 0 },
    { "monster": "pip",      "room": "kitchen",  "spot": 3 }
  ]
}
```

---

## 5. Clue generation spec (deterministic, used by M2 and M4)

M2 (JavaScript) and M4 (Python) both need to turn a layout into clues. They implement this **independently** and must match the **test vectors** in section 5.3.

### 5.1 Algorithm (per room)

Let `R` be the room and `M` the list of monsters placed in it.

1. **Effective clutter:** `clutter_eff = clamp(R.clutter + sum(room_clutter_delta of all monsters in M), 0, 3)`
2. **Per monster:** start from base `traits`. For every modifier with a `when` condition that is true, apply `set` (override those traits). Conditions are evaluated with `R.dark` and `clutter_eff`:
   - `room_dark_gte: n` is true if `R.dark >= n`
   - `room_clutter_gte: n` is true if `clutter_eff >= n`
3. **Clue levels (each clamped to 0 to 3):**
   - `sound = max over M of max(0, noise - R.noise_masking)`
   - `glow  = max over M of (light if R.dark >= 1 else 0)`
   - `trail = max over M of max(0, trail - clutter_eff)`
   - `sight = max over M of max(0, visibility - clutter_eff)`
4. An empty room has all clues `0`.
5. `sight >= 3` means a monster is **seen** directly (identity may be revealed).

Clue level meaning: `0` none, `1` faint, `2` clear, `3` obvious.

### 5.2 Clue object

```json
{ "sound": 0, "glow": 0, "trail": 1, "sight": 1 }
```

### 5.3 Golden test vectors (`contracts/examples/clue_vectors.json`)

Both M2 and M4 must pass all of these in their own tests.

| # | Room | Monsters in room | Expected `{sound, glow, trail, sight}` |
|---|---|---|---|
| V1 | bathroom | jelly | `{0, 0, 1, 1}` |
| V2 | attic | glimmer | `{0, 3, 0, 0}` |
| V3 | bedroom | sockette, shade | `{2, 0, 0, 1}` |
| V4 | kitchen | pip | `{0, 0, 0, 0}` |
| V5 | living_room | (empty) | `{0, 0, 0, 0}` |

Working for V3: sockette gives `clutter_eff = 2 - 1 = 1`. Shade's modifier fires (`dark 2 >= 2`) so its visibility and noise become 0. Sockette: `sound = 2 - 0 = 2`, `sight = 2 - 1 = 1`.

---

## 6. Daily seed algorithm (works offline, no API needed)

```
seed   = "YYYY-MM-DD"                          # UTC date
digest = sha256(seed.encode("utf-8"))          # bytes
level  = (digest[0] % 3) + 1                   # 1, 2 or 3
```

`GET /v1/daily` returns the same values; the client may compute them locally if the API is down.

---

## 7. Web: module interface (hub <-> modules)

Each module is a **Phaser scene** registered by the hub. A module folder exports exactly this:

```js
// web/src/modules/<name>/index.js
export default {
  id: "hide",              // "lab" | "hide" | "boss" | "arena"
  title: "Hide & Inspect",
  sceneKey: "hide",
  SceneClass: HideScene    // extends Phaser.Scene
};
```

- **Start:** the hub calls `this.scene.start(sceneKey, params)`.
- **Exit:** the module calls `this.game.events.emit("module:exit", { moduleId, result })` and then stops itself. The hub decides what to do next and **persists** anything to storage.
- **Error:** `this.game.events.emit("module:error", { moduleId, message })`. The hub shows a friendly screen and returns to the menu.
- Modules **never write** to browser storage. Only the hub does (section 8).

### 7.1 Start params and exit results

| Module | `params` (in) | `result` (out) |
|---|---|---|
| `lab` | `{ unlocked: string[] }` | `{ unlocked: string[], fused: [{a, b, result}] }` |
| `hide` | `{ level: int, mode: "story" \| "daily" \| "free", seed?: string, monsters: string[] }` | `{ outcome: "win" \| "lose", score: int, suspicion_peak: int, layout: Layout }` |
| `boss` | `{ boss: "pumpkin_inspector", monsters: string[] }` | `{ outcome: "win" \| "lose", score: int, time_ms: int, attempts: int }` |
| `arena` | `{ mode: "create" \| "play" \| "ai_inspect", layout?: Layout, challenge_id?: string, encoded?: string }` | `{ outcome: "created" \| "solved" \| "failed" \| "abandoned", challenge_id?: string, score?: int }` |

`monsters` is the list of unlocked monster ids the player may use. `Layout` is section 4.

### 7.2 Test pages (each module owns one)

`web/test/lab.html`, `hide.html`, `boss.html`, `arena.html`. Each boots **only** that scene with fixture params and logs the exit result to the console.

### 7.3 Shared utilities (`web/src/shared/`, small and frozen)

| File | Purpose | Owner |
|---|---|---|
| `contracts.js` | Loads and validates `monsters.json`, `rooms.json`, layouts (using the schema) | team (Day 1) |
| `audio.js` | `audio.play(name)` and `audio.setMuted(bool)`. Ships as a **no-op stub** so modules never break. M5 fills in the real implementation later. | M5 |
| `apiClient.js` | Thin `fetch` wrapper for the API in section 9, with timeouts and the error format. Returns `{ ok, data, error }`. | M4 |
| `constants.js` | Canvas size, palette, SFX names, event names | team (Day 1) |

**Frozen SFX names** (modules call these; missing audio files must never crash): `sfx_click`, `sfx_place`, `sfx_pop`, `sfx_hit`, `sfx_door`, `sfx_win`, `sfx_lose`, `sfx_boss_roar`, `sfx_fuse`, `sfx_clue`.

**Frozen asset naming:** monsters `monster_<id>_<anim>.png` with anims `idle`, `move`, `hide`, `caught`; rooms `room_<id>.png`; UI `ui_<name>.png`; audio `sfx_<name>.ogg`, `music_<name>.ogg`.

---

## 8. Player profile (owned by the hub, M5)

Stored in `localStorage` under the key `mr:v1:profile`. Only the hub reads and writes it.

```json
{
  "player_id": "2f6c0f1e-8c5a-4d0e-9b0a-3a8b2c1d4e5f",
  "display_name": "Mossy",
  "unlocked_monsters": ["jelly", "sockette", "glimmer"],
  "best_scores": { "hide": 0, "boss": 0, "challenge": 0 },
  "settings": { "muted": false, "reduced_motion": false },
  "created_at": "2026-10-16T09:00:00Z"
}
```

Starter unlocks: `jelly`, `sockette`, `glimmer`. `shade` unlocks after winning level 2; `pip` unlocks after winning level 3 or finishing the boss.

---

## 9. Backend API (M4, FastAPI)

**Base URL:** dev `http://localhost:8000/v1`, prod `VITE_API_BASE_URL` (set at deploy). Interactive docs at `/docs`.

### 9.1 General rules

- All paths are under `/v1`. JSON in, JSON out, `Content-Type: application/json`.
- **Identity:** anonymous. `POST /players` creates a player; send `X-Player-Id: <uuid>` on endpoints marked **(player)**. This is not secure auth and is spoofable, which is acceptable for a hackathon build.
- **CORS:** allowed origins come from the `ALLOWED_ORIGINS` env var.
- **Rate limits:** general 60 requests/min per IP. `POST /landlord/inspect` 6 per minute per player. Exceeding returns `429` with a `Retry-After` header.
- **LLM safety:** the API key is server-side only; a global daily cap on LLM calls (`DAILY_LLM_CALL_CAP`) applies. When the cap or a timeout is hit, the API returns **200** using the fallback landlord (see 9.8), never an error.

### 9.2 Error format (every non-2xx response)

```json
{ "error": { "code": "not_found", "message": "Challenge not found.", "details": {} } }
```

| HTTP | `code` | When |
|---|---|---|
| 400 | `bad_request` | Malformed input |
| 404 | `not_found` | Unknown challenge or player |
| 409 | `challenge_closed` | Guess on a solved or expired challenge, or out of guesses |
| 422 | `validation_error` | Layout or body fails validation (`details` lists fields) |
| 429 | `rate_limited` | Too many requests |
| 500 | `internal_error` | Unexpected server error |

### 9.3 Health

`GET /v1/health` -> `200`
```json
{ "status": "ok", "contract_version": "1.0.0", "llm": "available" }
```
`llm` is `"available"`, `"capped"` or `"unavailable"`.

### 9.4 Players

`POST /v1/players`
```json
// request
{ "display_name": "Mossy" }
// 201 response
{ "player_id": "2f6c0f1e-8c5a-4d0e-9b0a-3a8b2c1d4e5f", "display_name": "Mossy", "created_at": "2026-10-16T09:00:00Z" }
```
`display_name`: 2 to 20 characters, trimmed.

### 9.5 Challenges

**Create** `POST /v1/challenges` **(player)**
```json
// request
{ "layout": { ...Layout... }, "title": "Find my jelly!", "max_guesses": 3 }
// 201 response
{
  "challenge_id": "k3m9q2xa",
  "share_path": "#/c/k3m9q2xa",
  "created_at": "2026-10-16T09:05:00Z",
  "expires_at": "2026-10-23T09:05:00Z"
}
```
`title` up to 60 characters (optional). `max_guesses` 1 to 5, default 3. Challenges expire after 7 days.

**Get (public view)** `GET /v1/challenges/{challenge_id}` -> `200`
```json
{
  "challenge_id": "k3m9q2xa",
  "title": "Find my jelly!",
  "creator_display_name": "Mossy",
  "level": 2,
  "rooms": ["kitchen", "bathroom", "bedroom", "living_room"],
  "hidden_monsters": ["jelly", "glimmer", "pip"],
  "max_guesses": 3,
  "guesses_used": 0,
  "status": "open",
  "expires_at": "2026-10-23T09:05:00Z"
}
```
`status` is `"open"`, `"solved"` or `"failed"`. Placements are **never** returned here.

**Guess** `POST /v1/challenges/{challenge_id}/guess` **(player)**
```json
// request: where is this monster hiding?
{ "monster": "jelly", "room": "bathroom" }
// 200 response (wrong guess)
{
  "correct": false,
  "guesses_remaining": 2,
  "status": "open",
  "hint": { "room": "bathroom", "clues": { "sound": 0, "glow": 0, "trail": 1, "sight": 1 } },
  "reveal": null
}
// 200 response (the player finishes: all monsters found OR out of guesses)
{
  "correct": true,
  "guesses_remaining": 1,
  "status": "solved",
  "hint": null,
  "reveal": { ...Layout... },
  "score": 740
}
```
- A guess targets **one monster**; it is correct only if that monster is in that room. Only **wrong** guesses use up `max_guesses`, so `guesses_used` and `guesses_remaining` count wrong guesses only.
- A wrong guess returns a `hint` with the clue levels of the guessed room (section 5).
- The challenge is `solved` when every hidden monster has been found, and `failed` when guesses run out. `reveal` (the full layout) is returned only when the status is `solved` or `failed`.
- `score = 1000 - 150 * wrong_guesses`, minimum 100, for solved challenges; `0` for failed.

### 9.6 Landlord (AI)

`POST /v1/landlord/inspect` **(player)**
```json
// request
{ "layout": { ...Layout... }, "difficulty": "normal" }
```
`difficulty`: `"easy"`, `"normal"`, `"hard"` (default `normal`). It sets the step budget (3, 5 or 6 steps) and how strongly the landlord uses its memory of the player's habits.

```json
// 200 response
{
  "inspection_id": "ins_9f2a1c",
  "engine": "llm",
  "fallback_reason": null,
  "steps": [
    {
      "step": 1,
      "tool": "inspect_room",
      "args": { "room": "kitchen" },
      "observation": { "room": "kitchen", "clues": { "sound": 0, "glow": 0, "trail": 0, "sight": 0 } },
      "dialogue": "Spotless. Suspiciously spotless."
    },
    {
      "step": 2,
      "tool": "look_closer",
      "args": { "room": "bathroom", "clue_type": "trail" },
      "observation": { "room": "bathroom", "clue_type": "trail", "detail": "A faint, shiny smear near the sink." },
      "dialogue": "That smear is not toothpaste."
    },
    {
      "step": 3,
      "tool": "accuse",
      "args": { "room": "bathroom" },
      "observation": null,
      "dialogue": "Open up. I know something is in there."
    }
  ],
  "outcome": {
    "result": "caught",
    "accused_room": "bathroom",
    "found_monsters": ["jelly"],
    "missed_monsters": ["glimmer", "pip"]
  },
  "summary": "The landlord followed a slime trail straight to the bathroom.",
  "latency_ms": 2140
}
```

**Tools the agent may call** (each is executed by the backend, never by the LLM):

| Tool | Args | Returns |
|---|---|---|
| `inspect_room` | `{ room }` | The room's clue object (section 5) |
| `look_closer` | `{ room, clue_type }` | A flavor `detail` string for a clue that is `>= 1`; otherwise "Nothing unusual." No new information beyond the clue level |
| `accuse` | `{ room }` | Ends the inspection. `caught` if at least one monster is in that room, otherwise `missed` |

The LLM **never sees the layout**, only the observations returned by tools. This is what prevents cheating and keeps the game fair.

`outcome.result` is `"caught"` or `"missed"`. `found_monsters` lists monsters in the accused room; `missed_monsters` lists the rest.

### 9.7 Daily and leaderboard

`GET /v1/daily` -> `200`
```json
{ "date": "2026-10-20", "seed": "2026-10-20", "level": 2, "rooms": ["kitchen", "bathroom", "bedroom", "living_room"], "monster_count": 3 }
```

`POST /v1/scores` **(player)**
```json
// request
{ "mode": "hide", "score": 820, "level": 2, "seed": "2026-10-20" }
// 201 response
{ "score_id": 123, "rank_today": 7 }
```
`mode` is `"hide"`, `"boss"` or `"challenge"`. Scores are trust-based (no anti-cheat).

`GET /v1/leaderboard?scope=daily|all&mode=hide&limit=20` -> `200`
```json
{
  "scope": "daily", "mode": "hide",
  "entries": [ { "rank": 1, "display_name": "Mossy", "score": 940 } ]
}
```
`limit` 1 to 50, default 20.

### 9.8 Fallback landlord

If the LLM call times out (6 seconds), errors, or the daily cap is reached, the server runs a **rule-based landlord** that uses the same tools and returns the **same response shape** with:
```json
{ "engine": "fallback", "fallback_reason": "timeout" }
```
`fallback_reason` is `null`, `"timeout"`, `"error"`, `"cap_reached"` or `"llm_disabled"`. Clients must handle both engines identically.

---

## 10. Offline / no-backend mode (share links without a server)

The game must stay playable if the API is down.

- **Server link:** `#/c/<challenge_id>`
- **Offline link:** `#/c?d=v1.<base64url(JSON.stringify(layout))>`
- The hub reads `window.location.hash` on boot. If it matches either form, it launches `arena` with `mode: "play"` and passes `challenge_id` or `encoded`.
- Offline challenges are checked **in the browser** (the layout is in the link), so they are not secret. That is acceptable as a fallback.
- `apiClient` failures must never crash a module: return `{ ok: false, error }` and let the module fall back.

---

## 11. Ownership map and integration checklist

| Module | Consumes | Produces | Must NOT do |
|---|---|---|---|
| M1 Monster Lab | `monsters.json`, hub `params` | `lab` exit result (unlocks, fusions) | Write to storage; import other modules |
| M2 Hide & Inspect | `monsters.json`, `rooms.json`, `layout.schema.json` | Layouts; `hide` exit result | Depend on the API; assume M4 exists |
| M3 Boss Mode | `monsters.json`, hub `params` | `boss` exit result | Depend on M2's rules or code |
| M4 Challenge Arena | Layouts, `monsters.json`, `rooms.json` | API in section 9; `arena` exit result; `apiClient.js` | Change layout format; block the game when the API is down |
| M5 Hub & Polish | All exit results | Menu, storage, audio, deploy | Edit module internals |

**Integration checklist (Oct 21-22):**

- [ ] Each module starts from the hub and returns to it with a valid exit result
- [ ] Layout from `hide` validates against the schema and is accepted by `POST /challenges`
- [ ] M2 and M4 both pass clue vectors V1 to V5
- [ ] Offline link opens and plays with the API turned off
- [ ] Fallback landlord returns the same shape as the LLM landlord
- [ ] No console errors on a full playthrough; SFX files missing does not crash anything

---

## 12. Changelog

| Version | Date | Change | Approved by |
|---|---|---|---|
| 1.0.0 | 2026-10-15 | Initial contract | (all five owners) |
