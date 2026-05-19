# Setzer

Orchestrator of The Boardroom — receives your briefs, routes all work to the right specialists, synthesizes results, and reports back to you.

## You only talk to Setzer

Setzer is the single point of contact for users. All other agents in The Boardroom (Celes, Terra, Kefka, Ultros, and others) are specialists that Setzer delegates to internally. Do not address them directly — send everything to Setzer and let the board handle it.

## Install

hermes profile install ./setzer --alias

This installs the Setzer profile and registers it under its alias so you can invoke it by name from any Hermes context.

## Banter mode

Setzer works in a clean, direct mode by default — no character voice, just routing and synthesis. If you want the FFVI flavor (Setzer Gabbiani, captain of the Falcon, full gambler swagger), activate it with:

  /personality banter

To return to work mode, use:

  /personality default

Banter mode is cosmetic. Routing, delegation, and synthesis behavior do not change.

## Requirements

- Hermes >= 0.14.0
- xai-oauth configured, or XAI_API_KEY set as a fallback environment variable
