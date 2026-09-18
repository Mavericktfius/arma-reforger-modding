# Terrain, Surfaces, Environment

## Terrain creation

- Create via `GenericTerrainEntity` → Create New Terrain. **Use the
  `GenericTerrain_Default.et` prefab, not a bare `GenericTerrainEntity`** — the
  prefab sets the Terrain Layer Preset, without which vehicles handle badly.
- **Zero the terrain entity's coordinates.** Non-zero coords cause pathing
  failures and player collision errors.
- **Grid Size** sets vertex spacing — it is the detail-versus-draw-calls dial.
- **Heightmap import**: manage Z-axis inversion per source format; it differs
  between exporters.

## World files

- A world's layer folder name is **derived from the world filename**. Renaming
  `X.ent` orphans `X.ent_Layers` and the world opens completely empty. Rename
  both together or neither.
- **A 0-byte `.ent` is normal.** Content lives in `<world>_Layers/default.layer`.
- `default.layer` is **plain text and directly editable** — entities are
  `$grp <Class> : "{guid}path.et" {` blocks containing instances of five lines
  (`{`, `coords X Y Z`, `angles`, `scale`, `}`). This is the escape hatch when a
  world is too heavy to open in Workbench. Back it up first.

## Surface materials

- **Uncheck "linear color space" when importing satellite imagery**, or the
  world map renders far too dark.
- **Top-to-bottom parallax sorting.** Base layers such as dirt must sit at the
  **top** of the material list to render beneath grass and debris. This reads
  backwards and catches people every time.
- **Hard cap of 5 simultaneous surface materials** per block (raisable to 7).
- **Middle maps** blend close-range ground detail (GDT) into distant satellite
  imagery. Without one the transition is visible.
- **Shores** via the terrain tool shore map generator. Ocean wave simulators
  only spawn waves on shores physically facing open sea.
- **Q key** snaps prefabs along terrain normals.

## Atmosphere stack

Add in roughly this order:

- `WorldLightEntity` — local sun and moon pitch/yaw
- Generic Post-Processing Default prefab — HDR, god rays, ambient occlusion
- Planet entity — sun, moon, stars; assign **real geographic coordinates** for
  an accurate day/night cycle
- Sky Preset (`atmosphere.emat`), volumetric clouds, ocean simulation,
  Fog Haze Default, Time and Weather Manager

## Game Master

- **Base worlds stay pure.** Put Game Master in a SubScene:
  File → New World → SubScene.
- Run Plugins → Game Mode Setup → **Template Game Master**.
- Add an `SCR_CameraManager` to the base scene so GM gets camera navigation.
- Dropping `GameMode_Editor_Full.et` straight into the base world works for a
  single scenario, but redo it as a SubScene before adding a second game mode on
  the same terrain.

## Performance

Heavy terrain operations and heavy worlds do not coexist on low-VRAM hardware.
Sequence big jobs — delete entities, do the job, re-add. Keep terrain tile
texture size at **128**; 256 crashes generation on integrated graphics.
