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
  or Edge Crease (Shift+E) to tighten corners and stop over-smoothing. **High
  levels crash exports** — keep viewport and render levels low.
- **Lattice deform** — a bounding cage for non-destructive broad shape changes.
  Use it instead of direct editing to avoid tearing triangulated faces.
- **Applying** — hover the modifier, Ctrl+A to bake it into real geometry. Must
  happen before UV unwrapping complex shapes.

## Topology and shading

- **Quads only.** 4 vertices per face. Ngons (5+) and star junctions (poles)
  pinch badly under subdivision.
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
- Respect polycount and texture resolution caps — Enfusion runtime stability
  depends on them.
- **Never scale environmental prefabs above 1.3.** Collision meshes break.
