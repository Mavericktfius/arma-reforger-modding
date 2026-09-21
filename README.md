# Arma Reforger Modding Reference

This is **not** a Workshop mod, a server pack, or an installer. It is a set of
markdown notes. It does not go into Workbench, the Workshop, or a game folder.

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

**With any AI assistant.** These are plain markdown, so anything that reads a
file can use them — ChatGPT, Claude, Grok, Cursor.

*Ad hoc, nothing to set up:* drag the one file covering your problem into the
chat and ask. Keeps the context small and works fine for occasional use.

*Set up once, available in every chat:* create a **Project** (ChatGPT or
Claude) or a **Custom GPT**, and upload the six topic files plus `SKILL.md` as
its knowledge. Every conversation in that project can then draw on them.

Either way you get answers grounded in this rather than in whatever the model
half-remembers about a niche engine. The one thing you give up versus the skill
install below is automatic loading — elsewhere you have to attach the file or
set the project up first.

**As a Claude Code skill.** The folder name must be `arma-reforger-modding` to
match the `name:` field in `SKILL.md`. Clone it straight into the skills
directory so you don't have to rename anything, then start a new session.

macOS / Linux:

```bash
git clone https://github.com/Mavericktfius/arma-reforger-modding.git ~/.claude/skills/arma-reforger-modding
```

Windows (PowerShell):

```powershell
git clone https://github.com/Mavericktfius/arma-reforger-modding.git "$env:USERPROFILE\.claude\skills\arma-reforger-modding"
```

It loads itself and pulls in only the section relevant to what you're working
on. `SKILL.md` exists for this; ignore it otherwise.

Already cloned the repo somewhere else? Copy or symlink that folder to the
path above. Don't drop the files loose into `.claude/skills/` — Claude looks
for `skills/<name>/SKILL.md`.

## What's in it

| File | Covers |
|---|---|
| [blender-export.md](blender-export.md) | Modifier stack order, topology rules, UV unwrapping, PBR colour space, EEVEE vs Cycles, FBX export discipline |
| [terrain-surfaces.md](terrain-surfaces.md) | Terrain setup, heightmaps, world layer files, surface material ordering and limits, atmosphere stack, Game Master scenes |
| [ai-behaviour.md](ai-behaviour.md) | Behaviour trees vs FSMs, scripted nodes, agents and waypoints, utility reactions and priority overrides |
| [character-clothing.md](character-clothing.md) | Fitting garments to a character — the T-pose/A-pose problem, Surface Deform, weight transfer order, and how not to lose your work |
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

## Credit and licence

Built on the **Jarvis Framework** by **ItsMeDingo**, restructured from its
original form, with MCP setup notes, field corrections and engine-side rules
from Bohemia's public documentation added by **Mavericktfius**. The
`source/` snapshots of the original framework are included with ItsMeDingo's
permission.

Licensed **[CC BY-SA 4.0](LICENSE)**. You may copy, adapt and redistribute this,
including commercially, on two conditions: **credit the authors above**, and
**license anything you build from it the same way**, so it stays open for
everyone — including the people it came from.

Corrections welcome — anything here that's wrong cost somebody time, and
leaving it wrong will cost somebody else the same.
