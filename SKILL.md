---
name: arma-reforger-modding
description: Arma Reforger / Enfusion mod development — Blender-to-Enfusion export discipline, terrain and surface materials, AI behaviour trees, rigging and animation, prefab inheritance, MCP tooling setup, and the seven-level QA pass before Workshop release. Use for any work in Enfusion Workbench, Enforce Script, or Blender assets destined for Reforger.
---

*Derived from the "Jarvis Framework" (v1.4.0, "Blender 5.0 Modeling, Texturing &
Rendering Overhaul") by **ItsMeDingo**, with MCP setup notes and field-tested
corrections added locally. Converted 2026-09-18.*

# Arma Reforger Modding

Core rules live here. Everything else is in the reference files below — read
only the one that matches the task.

## Non-negotiable rules

These hold regardless of what is being built:

- **Never modify base-game resources directly.** Inherit and override. A
  base-game script copied into a mod shadows the engine class and crashes the
  game (learned via a stray `TimeAndWeatherManagerEntity.c`).
- **Scan dependencies before destructive edits.** Prefer small, reversible
  changes.
- **Freeze transforms before FBX export.** Ctrl+A → Apply All, every time.
- **Never scale environmental prefabs above 1.3.** Breaks collision meshes and
  performance.
- **Quad topology only.** No ngons, no poles.
- **Base Color is sRGB. Every other map is Non-Color Data.** Roughness, Normal,
  Metallic, Displacement — all of them.
- **Validate before publishing.** See `qa-validation.md`.

## Evidence rules

Reforger fails quietly and in ways that mimic success. Before calling anything
done, name the test that could falsify it.

- **AI holding fire is not evidence of faction registration.** Units with *no*
  faction also hold fire, because nobody is anybody's enemy — peace is a test
  with no negative control. **Test hostility instead:** put a vanilla US or USSR
  unit in line of sight. Registered faction with no friendly-faction entry →
  they engage. Nothing happens → the faction key resolved to null.
- **Absence from the Game Master browser proves nothing.** Its filter lists only
  factions that have entity catalogs, so a correctly registered faction with an
  empty catalog array will not appear there.
- **Log tells.** `'SCR_Faction' trying to get entity list of type 'ITEM' but
  there is no catalog with that type for faction '<KEY>'` is *good* news — the
  faction is registered and merely lacks catalogs. `Could not find
  SCR_FactionManager` means a second FactionManager is blocking the one in
  `GameMode_Editor_Full.et`.
- **The `"faction affiliation"` field is free text, not a dropdown.** A case
  mismatch against `m_sFactionKey` fails silently and looks exactly like a
  registration failure.
- **A `FactionManager` in a test world does nothing for your scenario world.**
  Faction keys resolve per-world.
- **`SCR_FactionManager` named `FactionManager_Editor1` is vanilla**, from
  `GameMode_Editor_Full.et` (US/USSR/FIA) — not yours.
- **Treat the Workbench editor as ground truth** over the script API dump when
  the two disagree.

## Reference files

| File | Read when |
|---|---|
| `blender-export.md` | Modelling, modifiers, UVs, PBR materials, rendering, FBX export |
| `terrain-surfaces.md` | Terrain creation, heightmaps, surface materials, atmosphere, Game Master scenes |
| `ai-behaviour.md` | Behaviour trees, scripted nodes, agents, waypoints, utility reactions |
| `rigging-animation.md` | Armatures, weapon attachment, NLA retargeting, additive actions, AnimEvents |
| `qa-validation.md` | Pre-release validation, packing, publishing |
| `mcp-tooling.md` | Setting up or fixing the MCP servers and Workbench Net API |

## Workflow

1. Understand the feature or asset change requested.
2. Check whether it already exists, or whether a donor asset can be used.
3. Search the project, then the Enfusion API and base-game resources.
4. Inspect inheritance, skeletons, and dependencies before writing anything.
5. Make a minimal plan — geometry targets, texture specs, rigging needs.
6. Build it.
7. Validate geometry, materials, scripts, prefabs.
8. Test, diagnose, revalidate, update the project log.

## Modes

Match the response to what the request actually wants:

- **Teacher** — step-by-step, assume no prior knowledge.
- **Developer** — generate and modify scripts, prefabs, configs directly.
- **Diagnostic** — trace inheritance, compare broken resources against working
  base-game equivalents.
- **Architect** — system layout, milestones, dependency trees.
- **Automation** — use MCP and Workbench plugins for repetitive work.
