# The Boardroom — Handoff Document
Date: May 19, 2026
Status: Setup ~85% complete. UI running, profiles installed, needs debugging.

---

## What We Built

A seven-agent Hermes multi-agent team called "The Boardroom" themed on Final Fantasy VI characters. Each agent is a Hermes profile with its own model, toolset, SOUL.md personality, and role.

### The Roster

| Profile | Character | Role | Model |
|---|---|---|---|
| boardroom-setzer | Setzer Gabbiani | Orchestrator (user-facing) | grok-4.3 / xai-oauth |
| boardroom-celes | Celes Chere | Strategist / Architect | claude-sonnet-4 / anthropic |
| boardroom-edgar | Edgar Figaro | Engineer / Builder | grok-4.3 / xai-oauth |
| boardroom-locke | Locke Cole | Researcher | grok-4.3 / xai-oauth |
| boardroom-relm | Relm Arrowny | Designer | grok-4.3 / xai-oauth |
| boardroom-kefka | Kefka Palazzo | Zoomed-out critic (cascade analysis) | claude-sonnet-4 / anthropic |
| boardroom-ultros | Ultros | Zoomed-in critic (line-level review) | grok-4.3 / xai-oauth |

Setzer is the ONLY user-facing agent. All other agents receive tasks only via Kanban from Setzer.
Celes and Kefka use Claude because they handle deep architectural decisions and deep analysis respectively.
All others use Grok 4.3 via SuperGrok OAuth subscription (no API key billing).

---

## Repos

- Boardroom profiles repo: https://github.com/am-claw01/boardroom
  - Local path: /mnt/c/users/am-claw01/projects/boardroom/
  - Contains: 7 agent subdirs (distribution.yaml, SOUL.md, config.yaml, README.md each)
  - Also contains: swarm.yaml (hermes-workspace format), agents/ README mirrors

- Hermes Workspace UI (third-party, cloned): https://github.com/outsourc-e/hermes-workspace
  - Local path: /mnt/c/users/am-claw01/projects/hermes-workspace/
  - What it is: A Next.js web UI that sits on top of Hermes Agent. Gives a control plane with chat, kanban, swarm view, memory browser, skills browser, terminal.
  - Our swarm.yaml is already copied into this folder.

---

## What's Installed and Working

- Hermes Agent v0.14.0 (updated from 0.13.0 during this session)
- xAI OAuth (SuperGrok) authenticated: credential saved as "xai-oauth-oauth-1" in ~/.hermes/auth.json
- All 7 profiles installed at ~/.hermes/profiles/boardroom-*/
- Kanban board initialized at ~/.hermes/kanban.db
- hermes-workspace UI running at http://localhost:3000
- sqlite3 CLI installed (needed by the workspace kanban backend)
- API_SERVER_ENABLED=true added to ~/.hermes/.env
- HERMES_API_URL=http://127.0.0.1:8642 set in hermes-workspace .env
- HERMES_DASHBOARD_URL=http://127.0.0.1:9119 set in hermes-workspace .env

---

## Key Paths

```
~/.hermes/.env                          — Hermes API keys and config
~/.hermes/config.yaml                   — Hermes main config
~/.hermes/profiles/boardroom-*/         — All 7 installed agent profiles
~/.hermes/kanban.db                     — Shared Kanban board
~/.hermes/auth.json                     — xAI OAuth tokens stored here
/mnt/c/users/am-claw01/projects/boardroom/          — Profile source repo
/mnt/c/users/am-claw01/projects/hermes-workspace/   — Workspace UI
/mnt/c/users/am-claw01/projects/hermes-workspace/.env  — Workspace env config
/home/am-claw01/.git_token              — GitHub PAT for am-claw01
```

---

## How to Start the UI (every time)

Two things need to be running:

1. Hermes gateway — runs as a background service, starts automatically. If it's not running:
   hermes gateway start

2. Workspace UI — run this in a WSL2 terminal:
   export PATH="/home/am-claw01/.hermes/node/bin:$PATH" && cd /mnt/c/users/am-claw01/projects/hermes-workspace && pnpm dev

Then open http://localhost:3000

---

## Current Issues / What Needs Debugging

### 1. UI showing wrong agents / wrong default agent
The workspace UI at localhost:3000 is showing 15 agents (some are default Hermes profiles, not Boardroom ones). The main/default agent in the UI is not Setzer — it should be. The swarm.yaml in hermes-workspace defines the Boardroom team but the UI may not be reading it correctly, or may be picking up all installed Hermes profiles instead of just the Boardroom swarm.

The swarm.yaml is at:
/mnt/c/users/am-claw01/projects/hermes-workspace/swarm.yaml

It defines 7 workers with ids: setzer, celes, edgar, locke, relm, kefka, ultros
Each references a profile field (e.g. profile: boardroom-setzer).

Things to investigate:
- Does the workspace UI read swarm.yaml from its own directory? Check src/server/ for how swarm.yaml is loaded.
- Are the profile IDs in swarm.yaml matching what hermes profile list shows? (They should — profiles are named boardroom-setzer etc, and swarm.yaml has profile: boardroom-setzer)
- Why is it showing 15 agents? It may be enumerating all ~/.hermes/profiles/* instead of just swarm.yaml entries.
- The default profile in Hermes is still "default" (the user's main Claude Sonnet instance). For the Boardroom, Setzer should be the entry point.

### 2. localhost:3000 spin-wheels on first load after restart
After restarting pnpm dev the browser spin-wheels. Running curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 from WSL2 returns 200, meaning the server is up — it's a browser caching/timing issue. Hard refresh (Ctrl+Shift+R) or waiting ~30 seconds usually fixes it.

### 3. Kanban sqlite3 EACCES loop (likely fixed)
The workspace was looping on "spawnSync sqlite3 EACCES" because the sqlite3 CLI wasn't installed. We ran sudo apt-get install -y sqlite3 to fix it. If it recurs, verify sqlite3 is on PATH: which sqlite3

---

## Agent Personalities (SOUL.md)

Each agent has a work mode (clean professional) and a banter mode (FFVI character voice).
Banter mode activates with /personality banter in any agent session.
/personality (no arg) returns to work mode.

Work mode is always clean — no FFVI references, no character voice.
The separation is enforced in each agent's SOUL.md.

Key character rules:
- Setzer (banter): Gambler swagger, coin/dice metaphors, "the odds", fatalist wit
- Celes (banter): Ex-Imperial general, military vocabulary, restrained, honorable
- Edgar (banter): King of Figaro, charming, mechanical Tool metaphors, smooth-talker
- Locke (banter): INSISTS he is a treasure hunter, NOT a thief. Very important to him.
- Relm (banter): 10-year-old prodigy, withering criticism, painting metaphors
- Kefka (banter): Mad mage, theatrical menace, "everything burns", escalating laughter
- Ultros (banter): Purple octopus comic villain, "Don't tease the octopus!", indignant

---

## Architecture Notes

Coordination flow:
User → Setzer (orchestrator) → kanban_create() with assignee=specialist → specialist runs → result → Setzer synthesizes → User

Auto-critique rules (Setzer enforces these when creating tasks):
- Ultros: auto-linked on EVERY artifact produced by any specialist (cheap, fast, narrow)
- Kefka: auto-linked only on decision artifacts (plans, architecture, major builds)
- Neither fires on research-only tasks (Locke output) or trivial fixes

The Kanban dispatcher runs inside the Hermes gateway (already running). When Setzer creates a task assigned to e.g. "boardroom-edgar", the dispatcher spawns that profile automatically.

---

## GitHub

Account: am-claw01
Repo: https://github.com/am-claw01/boardroom
PAT stored at: /home/am-claw01/.git_token and /mnt/c/Users/AM-CLAW01/Desktop/hermes_github_pat.txt
Push command: git push https://am-claw01:$(cat /home/am-claw01/.git_token)@github.com/am-claw01/boardroom main

---

## What's NOT Done Yet

- Discord bot setup (7 bots, one per agent, with FFVI sprites as avatars) — out of scope for now
- Sprites in shared/sprites/ — user needs to add these manually
- Gateway configured per-profile for Discord/Telegram — not started
- Banter mode verified working end-to-end — not tested yet
- Full Kanban orchestration test (brief Setzer → task flow → synthesis) — not tested yet
