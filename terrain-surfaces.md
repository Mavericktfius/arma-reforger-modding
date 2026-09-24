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

## Georeferencing a real-world terrain

A terrain built from real survey data is georeferenced, so real-world coordinates
can be converted into world X/Z without opening Workbench. Useful for placing a
town where the real one is, and for checking that the terrain is oriented right.

You need the box the heightmap was cut from: grid size, cell size, and the centre
latitude/longitude with its UTM zone.

```
world X = SIZE/2 + (easting  - centre_easting)     # X increases east
world Z = SIZE/2 + (northing - centre_northing)    # Z increases north
```

Convert lat/lon to UTM with the standard transverse Mercator formulas — no library
needed, it is about forty lines. NAD83 and WGS84 differ by about a metre in North
America, which does not matter at this scale.

**Cross-check the result against something placed by hand.** Converting real town
data and finding its centroid lands on a marker placed months earlier by eye is
strong independent confirmation the terrain is correctly oriented — and it costs
nothing, because you already have both numbers.

## Sampling ground height without Workbench

An ASCII grid (`.asc`) heightmap can be read directly to find terrain height at
any world position:

```
row = (SIZE - worldZ) / cellsize     # row 0 is the NORTH edge
col = worldX / cellsize
```

Stream the file and read only the row you need — these are hundreds of MB and
parsing the whole grid wastes gigabytes.

**Check whether an elevation offset is already baked in.** A heightmap prepared for
Enfusion has usually had one subtracted to fit the engine's height scale, so the
file may already be in world space. Do not assume, and do not trust a note that
says otherwise. Verify by sampling at a position where something has been snapped
to the terrain in Workbench and comparing — agreement within a metre confirms it.

## Real-world layout data

OpenStreetMap gives building footprints and road centrelines for most places,
including small towns. Fetch it from the standard API rather than Overpass, which
is frequently overloaded:

```
https://api.openstreetmap.org/api/0.6/map?bbox=west,south,east,north
```

Send a real User-Agent or you get HTTP 406. Returns XML. Data is ODbL — attribute
it.

Converted to world coordinates, this turns "build a town somewhere that looks
right" into "place these buildings at these coordinates", which is the difference
between a judgement call and a task someone else can pick up.
