# Retexturing Vanilla Assets

Inherit the vanilla prefab and override its material. Never modify base-game
resources. The hard part is knowing *which* material slot actually renders —
getting that wrong costs hours, because a wrong override looks identical to a
right one in the editor.

## Clothing does not render through MeshObject

This is the one that wastes an afternoon.

A `MaterialAssignClass` on `MeshObject` binds correctly — bold label, revert
arrow, matching `SourceMaterial` — and is **never consulted at render time for a
wearable**. Clothing draws through `BaseLoadoutClothComponent`, which carries its
own model references and its own material arrays:

```
BaseLoadoutClothComponent "{componentId}" {
 WornMaterialsOverride {
  "{guid}path/Your.emat"
 }
 ItemMaterialsOverride {
  "{guid}path/Your.emat"
 }
}
```

- `WornMaterialsOverride` — on the body. `ItemMaterialsOverride` — dropped on the
  ground. **Fill both.**
- These are **positional arrays indexed against the mesh's material slots**, not
  source-to-assign maps. That is why the GUI shows no "source" field and why they
  cannot safely be hand-authored. Do them in the editor.
- To learn how many slots a mesh has, double-click the `.xob` and read the
  Materials panel. A peaked cap has exactly one, so each array needs one entry at
  index 0.

Non-clothing assets *do* use `MeshObject` → `Materials` → `MaterialAssignClass`
with `SourceMaterial` naming the slot baked into the `.xob`.

## Inherit the concrete prefab, not the `_base`

Vanilla items usually ship as a pair: `Thing_base.et` and `Thing.et`. The
concrete one adds the inventory and item components that make it loot-able. A
child of `_base` is a *sibling* of the real item, not a replacement — it renders
on the character and then behaves oddly in inventory.

Check what the loadout slot actually points at and inherit **that**.

## The magenta test

When a retexture does not show, the trap is that your edited texture usually
differs from vanilla in only a small area. "The asset looks right" is then
equally consistent with the override working and with it being ignored, so every
check comes back inconclusive.

**Stop reasoning and paint diagonal magenta stripes across the whole texture**,
reimport, and look.

- Striped → the override is live, and your edits are somewhere you did not expect
- Unchanged → the override is inert, and no amount of repainting will help

Back the texture up first; restoring is one file copy. This settles in one pass
what several plausible theories each fail to resolve, and it generalises to any
"my change is not showing" problem.

## Insignia can live in the normal map

A painted emblem may exist **twice** — as colour in the BCR and as embossed relief
in the NMO. Cleaning the colour map leaves the relief, which still reads as the
emblem under lighting.

Test it by deleting the `NMOMap` line from your `.emat` and reloading. If the
emblem vanishes, it was relief. The asset renders flat and plasticky meanwhile —
restore afterwards.

Removing it properly means rebuilding the normal map, and since vanilla `.edds`
files cannot be exported, that means reconstructing the whole map from a
screenshot — degrading fabric weave and stitching everywhere to remove a few
pixels. For small details, leaving the relief is usually the better trade.

## Texture import

A base-colour PNG must be **RGBA**; an RGB-only file silently takes the
uncompressed path and lands as `TEXFMT_B8G8R8X8`, which renders washed out. Import
Settings → Color Space must be **`ToSRGB`**, not the default `ToLinear`. Verify on
the `.edds` via Details → Image data format; correct base-colour textures read
`TEXFMT_BC7_SRGB (99)`.

Reimport does nothing if the `.edds` is already newer than the source. Check the
timestamps before concluding a reimport failed.

## Gizmo drift

Dragging in the prefab edit viewport writes a root `coords` line into the `.et`.
On headgear that offsets the item on the head. The fix is to **remove** the line
so the transform is inherited again, not to set it to `0 0 0` — the parent may
define a deliberate fitting offset that zeroing would clobber.
