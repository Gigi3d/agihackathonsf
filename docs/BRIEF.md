# OpenPulse — context brief

Paste this into a new claude.ai chat as your first message, then brainstorm.

---

I'm building a product called **OpenPulse** and I want to brainstorm it with you. Here's the full context.

## What it is
OpenPulse gives a code repo a heartbeat. When a commit lands or a PR merges in a GitHub org, a multi-agent swarm turns the event into a witty, factual announcement with a meme and broadcasts it to X, Telegram, Discord, and Slack. A weekly job posts a top-10 contributor leaderboard. It is a developer-community automation tool: the people who care most about a project (its community) usually never hear about the work shipping, because nobody hand-posts every commit to four channels. OpenPulse closes that gap automatically.

## Why I'm building it
I run BitMask's developer community: 160,000+ people across four social platforms. BitMask is backed by legendary investor Tim Draper. Our engineers ship constantly, but that work stays invisible to the community, so momentum quietly dies. I felt this pain firsthand, so BitMask is the first real deployment and the case study, but the product is built for any repo, open or closed.

## How it works (architecture)
Two layers, each doing what it is best at:

- **Runtype (execution layer):** hosts the agents, injects all secrets, runs the deterministic flow (dedupe via collections, first-timer check, Giphy meme lookup, per-platform rendering), and fans out to every channel. A Tech Lead agent reads the normalized event and writes a strict-JSON brief (what shipped, why it matters, significance, vibe keywords). A Copywriter agent turns that brief into a <=200 char announcement grounded only in the brief. Both run on Claude Haiku 4.5. An eval suite grades the Copywriter (valid output, no invented facts, mentions author + repo, welcomes first-timers) and already caught a real model regression before it shipped.
- **Cotal (coordination layer):** the same Tech Lead and Copywriter also live on a local NATS mesh as signed, role-scoped identities, and the brief moves between them over a durable message log instead of a raw function call. That gives the agents identity, roles, presence, and auditable hand-offs. I exposed the Runtype agents as A2A so the Cotal nodes talk to them without a native integration.

Per-platform rendering: Telegram uses sendAnimation so the gif plays inline; Discord and Slack embed the gif URL; X posts a clean text tweet via an OAuth2 auto-refresh flow (public client, rotating refresh token stored in a record).

## Current status
- Runtype 2-agent swarm: working
- Ingestion flow (dedupe + first-timer + meme + broadcast + state): built and live-tested
- Telegram (inline gif), Discord, Slack: live, posting real messages end-to-end
- X: flow is built and the token rotates correctly, but the X app is not yet attached to a developer Project (403 client-not-enrolled), so live tweets are pending that one account fix
- Eval suite: built, caught a real bug
- Weekly leaderboard flow: built (cron schedule blocked by plan quota)
- Cotal mesh + 2 role agents: running, hand-off demoed interactively
- Landing page + pitch deck: live on GitHub Pages
- Built for the AGI Hackathon SF (tracks: Autonomous Agents & Workflows, Multi-Agent Systems & Coordination, Applied AI Products)

## Business thesis (rough)
- Market: millions of repos and orgs; developer marketing is a multi-billion-dollar spend, and agentic coding ships more code every day
- Model: free for one repo, paid per org, then upsell agentic devrel (release threads, changelog digests, recap clips). Every post carries a subtle "via OpenPulse" so it markets itself
- Moat: it becomes your community's voice and owns your distribution, so switching cost compounds
- Math: 10,000 orgs x $1k/yr = $10M ARR, and 10x from there to $100M
- Proof: first deployed for BitMask (160,000+ community, backed by Tim Draper)

## What I want to brainstorm
(fill in what's on your mind, e.g.)
- Positioning and naming
- Pricing and the free-to-paid line
- The wedge: which segment to win first (open-source maintainers? funded startups? DAOs?)
- The agentic devrel upsell roadmap
- Go-to-market and the self-marketing "via OpenPulse" loop
- Risks and moats

Please act as a sharp product and go-to-market partner. Ask me clarifying questions before giving generic advice.
