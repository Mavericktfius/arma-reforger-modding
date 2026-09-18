# QA, Packing, Release

## The seven validation levels

Run in order. Each assumes the one before it passed.

| # | Level | Checks |
|---|---|---|
| 1 | **Structure** | `.gproj` setup, target folder hierarchy |
| 2 | **Resources** | Missing paths, texture formats, material compliance (BCR/NMO — see `blender-export.md` for the channel layout), audio pointers |
| 3 | **Prefabs** | Inheritance chains, component structure, property overrides |
| 4 | **Scripts** | Enforce Script syntax, API class binding, valid class overrides |
| 5 | **Configs** | Entity catalogs, faction registration, game mode config |
| 6 | **Runtime** | World loads, scripts execute, spawns work, AI paths |
| 7 | **Workshop** | Manifest accuracy, developer artifacts purged, release build |

## Packing and publishing

**The game only loads packed addons.** A raw Workbench source project will not
load no matter what `-addonsDir` points at. A packed addon means
`addon.gproj` + `data.pak` + `resourceDatabase.rdb` under
`My Games/ArmaReforger/addons/<Name>_<GUID>/`.

- Some Workbench builds have **no "Pack Project"** — then
  `File → Publish to Workshop` is the only route.
- Launch args that work once published:
  `-addonsDir "<dir>" -addons <GUID>` — use the **GUID**, not the addon ID.
## Publishing locks the project

`File → Publish to Workshop` writes its build output **into the project folder
itself**: `data.pak`, plus `addon.gproj_<ver>_manifest.json`,
`data.pak_<ver>_manifest.json`, `resourceDatabase.rdb_<ver>_manifest.json` and
`ServerData.json`.

Once `data.pak` exists, Enfusion **mounts the pak instead of the loose files**
and treats the project as a packed addon — read-only, with a padlock on every
resource including your own. The game log confirms it by listing the addon as
`(packed)`.

To edit again: close Workbench, move `data.pak` and the manifests out. The
Workshop copy is unaffected. The working cycle is **edit → publish → test →
remove the pak → edit**.

Two traps when removing it:

- `data.pak` is **locked while Workbench is running**.
- Workbench can linger as `ArmaReforgerWorkbenchSteamDiag` after its window
  closes. Check for the process before assuming it's gone.

Publishing also shrinks `resourceDatabase.rdb` to the packed version (~4 KB).
Workbench rebuilds it on next open — that is not damage.

## Recurring traps

- **Duplicate addon IDs collide.** Two project folders declaring the same addon
  ID will fight if both are present. Check for abandoned copies before
  publishing.
- **Cloud-synced project folders get dehydrated.** OneDrive/Drive Files
  On-Demand will evict files, and an evicted file reads as *missing* to
  Workbench and to the game. Set the project folder to "Always keep on this
  device."
- **Purge developer artifacts before a release build** — `.bak` files, test
  textures, abandoned prefabs.
- **Inventory items need `SCR_ItemAttributeCollection`**, not the engine base
  `ItemAttributeCollection`. Set this on the `InventoryItemComponent` when
  creating an item prefab — Bohemia flag it as important in their own material,
  which usually means people keep getting it wrong.

## Before calling anything done

Name the test that would falsify the claim, then run it. Reforger fails quietly:
things that look like success (units spawning, no errors in the log) routinely
coexist with a feature that was never registered at all.
