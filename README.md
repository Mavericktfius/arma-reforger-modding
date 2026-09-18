# Arma Reforger Modding Reference

Field-tested notes on building mods for Arma Reforger — the Blender-to-Enfusion
asset pipeline, terrain and surface materials, AI behaviour trees, rigging and
animation, and the validation pass before a Workshop release.

This is the stuff that isn't in the official documentation: the ordering
constraints, the silent failures, and the traps that cost people a weekend
before they find the one forum post explaining them.

## Who it's for

Anyone doing asset-level Reforger work — custom factions, weapons, vehicles,
terrain, characters. It assumes you've opened Workbench before. It does not
assume you use any AI tool, and most of it does not involve one.

## Three ways to use it

**Just read it.** Every file below is plain text with no setup. Start with
whichever one matches what you're stuck on. On GitHub they render as web pages;
locally, any text editor or markdown viewer works.

**With any AI assistant.** Paste or upload the file covering your topic
(ChatGPT, Claude, Grok, Cursor — all fine). You get answers grounded in this
rather than in whatever the model half-remembers about a niche engine.

**As a Claude Code skill.** Drop the whole folder into `~/.claude/skills/`
(`C:\Users\<you>\.claude\skills\` on Windows) and start a new session. It loads
itself and pulls in only the section relevant to what you're working on.
`SKILL.md` exists for this; ignore it otherwise.

## What's in it

| File | Covers |
|---|---|
| [blender-export.md](blender-export.md) | Modifier stack order, topology rules, UV unwrapping, PBR colour space, EEVEE vs Cycles, FBX export discipline |
| [terrain-surfaces.md](terrain-surfaces.md) | Terrain setup, heightmaps, world layer files, surface material ordering and limits, atmosphere stack, Game Master scenes |
| [ai-behaviour.md](ai-behaviour.md) | Behaviour trees vs FSMs, scripted nodes, agents and waypoints, utility reactions and priority overrides |
| [rigging-animation.md](rigging-animation.md) | Armatures, weapon attachment, the delta-scale trap, NLA retargeting, additive actions, AnimEvents |
| [qa-validation.md](qa-validation.md) | Seven-level validation pass, packing and publishing, recurring release traps |
| [mcp-tooling.md](mcp-tooling.md) | Optional: three third-party tools that speed some of this up, how to set them up, and where they're unreliable |
| [SKILL.md](SKILL.md) | Core rules and evidence rules — worth reading on its own even if you never use it as a skill |

`mcp-tooling.md` is the only file that assumes an AI setup, and it's optional.
Everything else is engine and Blender knowledge that stands on its own.

## A note on evidence

Reforger fails quietly. Things that look like success — units spawning, a clean
log, no errors — routinely coexist with a feature that never registered at all.
Several rules here are specifically about telling real success from the
appearance of it. `SKILL.md` collects those under "Evidence rules," and they're
probably the most useful part of this.

## Credit

Built on the **Jarvis Framework** by **ItsMeDingo**, restructured from its
original form with MCP setup notes and field corrections added.

Corrections welcome — anything here that's wrong cost somebody time, and
leaving it wrong will cost somebody else the same.
