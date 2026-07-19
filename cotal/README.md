# COTAL coordination layer

COTAL is the multi-agent coordination substrate: a local [NATS](https://nats.io) + JetStream mesh where agents get signed identity, roles, presence, and a durable message log. Our two production agents (Tech Lead, Copywriter) run their hand-off here to make coordination observable.

## Prerequisites
- Node 20+
- `nats-server` on PATH (`brew install nats-server`) — COTAL ships no macOS arm64 binary.

## Setup

```bash
# 1. Configure the mesh (installs connectors, seeds an agent, launches nothing)
npx cotal-ai setup --yes

# 2. Start the mesh (nats-server + delivery daemon + manager)
npx cotal-ai up --detach

# 3. Verify
npx cotal-ai status
```

## Spawn the two role-specialized agents

```bash
npx cotal-ai spawn default --name tech-lead  --role tech-lead  --detach \
  --prompt 'You are the bitmask-stack Tech Lead on this Cotal mesh. Given a GitHub event, analyze what shipped and why it matters in 1-2 sentences, then DM your brief to the copywriter agent. Be concise.'

npx cotal-ai spawn default --name copywriter --role copywriter --detach \
  --prompt 'You are the bitmask-stack Copywriter on this Cotal mesh. When the tech-lead DMs you a brief, write one short witty dev-community announcement and reply back. Be concise.'

npx cotal-ai endpoints   # roster: manager, tech-lead, copywriter
```

## Demo the hand-off (run interactively)

The agents are Claude Code sessions; drive them in an **interactive** shell (a detached/headless spawn stalls at orientation).

```bash
# In one terminal, watch coordination live:
npx cotal-ai web            # browser dashboard: presence, channels, live feed
# (or) npx cotal-ai console  # terminal TUI

# In another, trigger the hand-off:
npx cotal-ai send dm tech-lead \
  "New merged PR in bitmask-stack/rgb-core: 'Add RGB20 fungible asset issuance flow'. Analyze it, then DM your brief to the copywriter."
```

Watch the Tech Lead analyze, DM the Copywriter, and the Copywriter reply — signed identities and a durable log on the mesh.

## Stop

```bash
npx cotal-ai down
```

## Notes
- `.cotal/` (local mesh state, may contain tokens) is git-ignored.
- Observe with `cotal web`, `cotal console`, `cotal ps`, `cotal endpoints`.
