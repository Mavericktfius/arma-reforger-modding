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

## PBR materials

- **Base Color → sRGB. Everything else → Non-Color Data.** Roughness, Normal,
  Metallic, Displacement. Getting this wrong causes engine misinterpretation,
  not an obvious error.
- **Always put a Normal Map node** between the purple normal texture and the
  Principled BSDF. It converts RGB into vector data; without it the normal map
  is silently wrong.
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
- Respect polycount and texture resolution caps — Enfusion runtime stability
  depends on them.
- **Never scale environmental prefabs above 1.3.** Collision meshes break.
