# OpenPulse

**Give your repo a heartbeat.** OpenPulse is a multi-agent orchestration and developer-community automation tool. When a commit lands or a PR merges, a **multi-agent swarm** turns it into a witty, grounded announcement with a meme and broadcasts it to **X, Telegram, Discord, and Slack**. A weekly job posts a top‑10 contributor leaderboard.

First deployed for **BitMask** (a 160,000+ developer community across four platforms, backed by Tim Draper); built for any repo, open or closed.

> **Why I'm building this:** I run BitMask's 160,000-person developer community across four social platforms, and BitMask is backed by legendary investor Tim Draper. Our engineers ship constantly, but the people who care most, our community, never hear about it. Nobody hand-posts every commit to four channels, so the work stays invisible and momentum quietly dies. OpenPulse closes that gap automatically.

Built for the AGI Hackathon SF on a **Dual‑Core** stack:

- **[Runtype](https://runtype.com)**, execution: agents, flows, tools, secret injection, evals, broadcasting
- **[COTAL](https://cotal.ai)**, coordination: a local NATS mesh where signed, role‑specialized agents hand off work

---

## Architecture

```
   GitHub push / merged PR
            │
            ▼
   ┌───────────────────────────┐
   │   Runtype Webhook Surface  │   verifies GitHub HMAC, spreads payload
   └─────────────┬─────────────┘
                 ▼
   ┌───────────────────────────┐
   │  Dedupe  (Collections)     │   processed_commits · processed_prs
   │  First-timer check         │   known_contributors
   └─────────────┬─────────────┘
                 ▼
  ┌──────────── Multi-Agent Swarm (Runtype) ─────────────┐
  │                                                       │
  │   🧑‍💻 Tech Lead  ──── brief (shared state) ────►  📣 Copywriter  │
  │   Claude Haiku 4.5                         Claude Haiku 4.5      │
  │   "what shipped / why it                   "witty announcement, │
  │    matters / vibe keywords"                 grounded in brief"   │
  └─────────────────────────┬─────────────────────────────┘
                            ▼
                 ┌───────────────────────┐
                 │  Giphy meme lookup     │   (deterministic, by vibeKeywords)
                 └───────────┬───────────┘
                             ▼
        ┌──────────┬─────────┼─────────┬──────────┐
        ▼          ▼         ▼         ▼          
     X / Twitter  Telegram  Discord   Slack       
     auto-OAuth   sendAnim  webhook   webhook      
     (rotating)   (inline   (embeds   (unfurls     
                   gif)      gif)      gif)         

   ── weekly (Sun 09:00 UTC) ───────────────────────────
   GitHub commit search → aggregate top 10 → Claude header → broadcast


  COTAL coordination substrate (local mesh, NATS/JetStream)
  ┌──────────────────────────────────────────────────────┐
  │   manager ──supervises──┐                             │
  │                         ▼                             │
  │     tech-lead  ◄── signed DM / durable log ──►  copywriter
  │     (role)                                    (role)   │
  └──────────────────────────────────────────────────────┘
```

Why two layers? **COTAL** gives the agents identity, roles, presence, and a durable message log for secure hand‑offs; **Runtype** executes the deterministic flow, injects secrets, and fans out to every platform. The same Tech Lead → Copywriter hand‑off exists in both: as a Runtype flow (production path) and on the COTAL mesh (coordination showcase).

---

## Repo layout

```
runtype/
  agents.json            Tech Lead + Copywriter agent definitions
  tools.json             8 custom tools (Giphy, GitHub, 4 broadcasters, X helpers)
  collections.json       processed_commits / processed_prs / known_contributors
  flows/
    ingestion.json       GitHub event → swarm → meme → broadcast → state
    leaderboard.json     weekly top-10 contributor leaderboard
    x-post-auto.json     post to X with automatic OAuth2 token refresh + rotation
    x-seed-exchange.json one-time PKCE code → token exchange (seeds the record)
cotal/
  README.md              local mesh setup + the interactive demo script
.env.example             secret names + how to obtain each
```

Every secret is referenced as `{{secret:NAME}}`, **no credential values are committed.**

---

## How it works

### Multi-agent hand-off
1. **Tech Lead** (`claude-haiku-4-5`, no tools) reads the normalized event and emits a strict‑JSON brief: `whatShipped`, `whyItMatters`, `significance`, `vibeKeywords`.
2. **Copywriter** (`claude-haiku-4-5`, no tools) turns the brief into a ≤200‑char announcement grounded only in the brief, never inventing facts.
3. The flow does the **Giphy** lookup deterministically (using `vibeKeywords`) so the agents stay fast and cheap, then routes per platform.

### Per-platform rendering
- **Telegram** → `sendAnimation` so the gif plays inline (a URL in text does not).
- **Discord / Slack** → gif URL in the message body (auto‑embeds / unfurls).
- **X** → clean text tweet via the auto‑refresh flow.

### X unattended posting
X OAuth2 access tokens expire in ~2h and the refresh token rotates on every use. `x-post-auto` reads the token from a Runtype **Record** (`x_oauth/x_token`), refreshes when near expiry, **persists the rotated refresh token**, and posts, all with a **public** OAuth2 client (no client secret). See [`runtype/flows/x-post-auto.json`](runtype/flows/x-post-auto.json).

### Reliability
A Runtype **eval suite** grades the Copywriter (valid output, no invented facts, mentions author + repo, welcomes first‑timers). It already caught a real model regression (Gemini 3.x truncating outputs) before it shipped, which is why the agents run on Claude Haiku 4.5.

---

## Setup

### 1. Runtype
Import the definitions in `runtype/` (via the Runtype dashboard or MCP tools). Then in **Settings → Secrets**, add the values from [`.env.example`](.env.example). Attach the webhook surface's URL to `bitmask-stack → Settings → Webhooks` (events: Pushes + Pull requests; content‑type `application/json`) and send one delivery to activate it.

### 2. X (public OAuth2 client)
Create an X app as a **Native/Public client** (OAuth 2.0, Read+Write). Run the PKCE flow once (scopes `tweet.read tweet.write users.read offline.access`), then run `x-seed-exchange` with the returned `code` to seed the `x_oauth/x_token` record with `clientId` + `refreshToken`.

### 3. COTAL
See [`cotal/README.md`](cotal/README.md).

---

## Status

| Piece | State |
|---|---|
| Runtype 2-agent swarm (Claude Haiku 4.5) | ✅ working |
| Ingestion flow + dedupe + first-timer | ✅ built |
| Giphy / GitHub / Telegram / Discord / Slack | ✅ live-tested |
| X auto-refresh (token seeded & rotating) | ✅ built; live tweet pending |
| Eval suite | ✅ built (caught a real bug) |
| Weekly leaderboard flow | ✅ built; schedule cron blocked by plan quota (402) |
| COTAL mesh + 2 role agents | ✅ running; hand-off demoed interactively |

## Hackathon tracks
Autonomous Agents & Workflows · Multi-Agent Systems & Coordination · Applied AI Products.

## License
MIT
