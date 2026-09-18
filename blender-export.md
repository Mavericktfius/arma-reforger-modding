# Blender — Modelling, Texturing, Export

Blender 5.0 LTS with Enfusion Blender Tools (EBT).

## Navigation and hotkeys

- **Orbit** MMB drag · **Pan** Shift+MMB · **Zoom** scroll or Ctrl+MMB
- **Focus selected** Numpad `.` (or `~` → View Selected)
- **Transform** G move, R rotate, S scale. Tap X/Y/Z straight after to lock an
  axis. Hold Shift while dragging for micro-adjustments.
- **Mode toggle** Tab (Object ↔ Edit)
- **Component select** in Edit Mode: 1 vertices, 2 edges, 3 faces. Alt+click an
  edge selects a continuous loop.
- **Geometry** Shift+D duplicate (Esc to leave in place), E extrude along
  normals, I inset, F fill, Ctrl+R loop cut (scroll to add cuts)
- **Proportional editing** O. Adjust falloff radius with scroll or PgUp/PgDn
  *while* grabbing.
- **Versioning** Numpad `+` / `-` in Save As auto-increments the filename.

## Modifier stack

Modifiers execute **strictly top to bottom**. For layered assets the order is:

    Shrinkwrap → Solidify → Subdivision Surface

- **Shrinkwrap** — snaps geometry to a target mesh. This is how clothing and
  accessories conform over a base body without pulling vertices by hand.
- **Solidify** — gives thickness to single-plane meshes. `Offset` controls
  direction: −1 inward, 1 outward.
- **Subdivision Surface** — doubles geometry per level. Use edge loops (Ctrl+R)
  or Edge Crease (Shift+E) to tighten corners and stop over-smoothing. High
  levels crash exports. There is no fixed level to stay under — what matters is
  the triangle count it leaves in LOD0, so judge it by the Statistics overlay
  and by whether you have built LODs, not by the level number.
- **Lattice deform** — a bounding cage for non-destructive broad shape changes.
  Use it instead of direct editing to avoid tearing triangulated faces.
- **Applying** — hover the modifier, Ctrl+A to bake it into real geometry. Must
  happen before UV unwrapping complex shapes.

## Topology and shading

- **Quads only, and evenly sized.** 4 vertices per face. Ngons (5+) and star
  junctions (poles) pinch badly under subdivision. For anything that deforms,
  even quads are what makes topology *animatable* — triangles deform in
  unwanted ways, and ngons get triangulated by the engine in ways you did not
  choose. Control the topology rather than letting an algorithm decide it.
  **Find them:** Edit Mode → Select → Select All by Trait → **Faces by Sides**,
  set to *Greater Than* 4 — anything selected is an ngon. Turn on the
  **Statistics** overlay (Viewport Overlays → Statistics) to watch face and
  triangle counts as you work, rather than discovering them at export.
- **Shade Smooth** via right-click. Flat shading is for low-poly or mechanical
  assets only.
- **Black shading artifacts → Shift+N** in Edit Mode to recalculate outside
  normals. This is almost always the fix.

## UV unwrapping

1. Select hidden or logical boundary edges → Ctrl+E → **Mark Seam**. Seams are
   where the 3D surface gets cut to lie flat.
2. Select all faces (A) → U → **Unwrap (Angle Based)**.
3. In the UV Editor, scale/rotate/position islands. Straighten curved islands
   (alignment tools or a gridify addon) so texture tiling stays uniform.
4. **Check it.** An unwrap can complete and still be wrong. Assign a checker
   material (UV Editor → New → **UV Grid**) and look at the model: squares
   reading as rectangles mean stretching, squares at different sizes across the
   mesh mean inconsistent texel density. Islands stacked on each other in the UV
   Editor mean **overlapping UVs** — intended for mirrored geometry, a bug
   everywhere else.

## PBR materials

- **Base Color → sRGB. Everything else → Non-Color Data.** Roughness, Normal,
  Metallic, Displacement. Getting this wrong causes engine misinterpretation,
  not an obvious error.
  **Verify after import:** on the `.edds` in Workbench, Details → *Image data
  format*. A correct base colour reads `TEXFMT_BC7_SRGB (99)`, same as vanilla.
  Two preconditions to land there — the source PNG must be **RGBA** (an RGB-only
  file silently takes the uncompressed path and arrives as `TEXFMT_B8G8R8X8`,
  which is also non-sRGB and renders washed out), and Import Settings →
  **Color Space** must be `ToSRGB`, not the default `ToLinear`.
- **Always put a Normal Map node** between the purple normal texture and the
  Principled BSDF. It converts RGB into vector data; without it the normal map
  is silently wrong.
  **Check the graph, not the render:** the chain must read Image Texture
  (Non-Color) → Normal Map → BSDF **Normal** input. A texture wired straight
  into Normal is the failure, and it shows up as subtly flat lighting rather
  than as anything that looks broken.
- **Subsurface scattering** for skin, food, thin fabric. Weight 1.0, Scale
  controls penetration depth. Default RGB radius is `1, 0.2, 0.1` — set it to
  `1, 1, 1` for neutral scattering.

## Rendering

- **EEVEE** — rasterised, screen-space. Fast, prone to screen-space artifacts.
  Raise **Render Steps** (≈16) for accurate shadows. Enable **Jittered Shadows**
  for soft lamps. Use **Light Probe Volumes** and bake to get indirect bounces
  from outside the camera view.
- **Cycles** — path-traced, physically accurate, expensive. Enable GPU compute
  at Edit → Preferences → System → Optix/CUDA.
- **Lamps** — Sun lamps have no falloff and ignore position; only rotation
  matters. Point lamps follow inverse square. Larger `Radius` = softer shadows.
- **Depth of field** — in Camera settings, eyedropper a focus object. Use it to
  kill distracting backgrounds in asset renders.

## Export to Enfusion

- **Freeze transforms first.** Ctrl+A → Apply All. Every time, no exceptions.
  **Confirm it took:** the N-panel → Transform should read Location `0, 0, 0`,
  Rotation `0, 0, 0`, Scale `1, 1, 1`. Check **Delta Transform** in the same
  panel while you are there — it is a separate multiplier that reads as normal
  in the main fields, and a non-1.0 delta scale is what makes an asset collapse
  to nothing the moment it is parented (see `rigging-animation.md`).
- **Never scale environmental prefabs above 1.3.** Collision meshes break.

### There is no polycount cap

Enfusion does not enforce a triangle budget, and Bohemia's own modelling
documentation states none. The LOD system switches based on the object's
on-screen size, LOD0's triangle count, FOV and graphics settings, targeting a
constant triangle-per-pixel ratio — density is managed adaptively at runtime.

So the rule is **not** "stay under N triangles." It is **build proper LODs and
let the system do its work.** A dense single-LOD asset is the actual failure
mode, not a high triangle count as such.

The one hard limit: **16 resolution LODs**. Anything beyond that is discarded.

### LODs

LODs are lower-density copies of the same topology, used at distance.

**Naming is load-bearing.** Same base name, underscore, LOD number:

    Helmet_LOD0
    Helmet_LOD1
    Helmet_LOD2

Get the naming wrong and the engine does not associate them, which presents as
an asset that never simplifies with distance rather than as an error.

### Colliders

Without a collider an asset cannot collide with anything.

- **The name's initials declare the collider type to the engine.** `UTM` marks a
  trimesh collider. This is parsed from the name — it is not a setting.
- Build the collider as a **separate, much simpler mesh**. The lower the
  topology the better; never reuse the render mesh.
- After import, set the collider's **Layer preset to `FireGeo`** in import
  settings, and set the Surface property material according to what ammunition
  it should stop.
- Clothing that is not meant to stop bullets (jackets, trousers) only needs a
  collider on the item version.

### Texture map channel packing

Two packed maps, and the channel layout is fixed:

| Map | Channels |
|---|---|
| **BCR** | Base colour in **RGB**, roughness in **alpha** |
| **NMO** | Normal in **RG**, metalness in **B**, occlusion in **alpha** |

Enfusion uses **DirectX** normal maps and reads only red (+x) and green (−y).
