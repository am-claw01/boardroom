# The Boardroom

A seven-agent Hermes team themed on Final Fantasy VI characters. Each agent is
an independent Hermes profile with its own model, toolset, memory, and gateway.
They coordinate through Hermes's native Kanban board.

## The Roster

| Profile  | Character     | Role                  | Model              |
|----------|---------------|-----------------------|--------------------|
| setzer   | Setzer Gabbiani | Orchestrator        | Grok 4.3 (OAuth)   |
| celes    | Celes Chere   | Strategist            | Claude Sonnet 4    |
| edgar    | Edgar Figaro  | Engineer              | Grok 4.3 (OAuth)   |
| locke    | Locke Cole    | Researcher            | Grok 4.3 (OAuth)   |
| relm     | Relm Arrowny  | Designer              | Grok 4.3 (OAuth)   |
| kefka    | Kefka Palazzo | Critic (zoomed-out)   | Claude Sonnet 4    |
| ultros   | Ultros        | Critic (zoomed-in)    | Grok 4.3 (OAuth)   |

Celes handles deep architectural decisions; Kefka handles deep analysis.
All other agents run on Grok 4.3 via SuperGrok OAuth subscription.

## Prerequisites

1. Hermes Agent v0.14.0 or newer
2. SuperGrok subscription (for Grok agents)
3. Anthropic API key in `~/.hermes/.env` (for Celes and Kefka)

## SuperGrok OAuth Setup (do this first)

```
hermes auth add xai-oauth
```

Approve in your browser. Tokens are saved and auto-refreshed.

## Install All Agents

```
hermes profile install ./setzer --alias
hermes profile install ./celes --alias
hermes profile install ./edgar --alias
hermes profile install ./locke --alias
hermes profile install ./relm --alias
hermes profile install ./kefka --alias
hermes profile install ./ultros --alias
```

## Initialize the Kanban Board

```
hermes kanban init
```

## Start the Gateway (Setzer is the user-facing agent)

```
setzer gateway setup    # configure Discord/Telegram for Setzer
setzer gateway start
```

## Banter Mode

In any agent session:

```
/personality banter    # activate FFVI character voice
/personality           # return to work mode
```

## Discord Channel Layout (recommended)

- `#war-room` — brief Setzer here
- `#strategy-hall` — Celes plan outputs
- `#forge` — Edgar build outputs
- `#archive` — Locke research outputs
- `#studio` — Relm design outputs
- `#critique-pit` — Kefka and Ultros critique outputs
- `#below-decks` — banter mode, off-duty character chatter

## Sprites

Drop FFVI character sprites into `shared/sprites/`. Recommended filenames:

- `setzer.png`, `celes.png`, `edgar.png`, `locke.png`
- `relm.png`, `kefka.png`, `ultros.png`

Use these as Discord bot avatars when setting up each agent's bot application.

## See Also

- `BOARDROOM_BUILD_SPEC.md` — full build specification
- Per-agent `README.md` in each subdirectory
- Hermes docs: https://hermes-agent.nousresearch.com/docs/
