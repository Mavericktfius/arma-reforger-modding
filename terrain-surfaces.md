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
- **Hard cap of 5 surface materials per block**, and the default surface is
  one of them — so a terrain gets a default plus four painted layers. BI's own
  tutorial recommends three. (The "raisable to 7" figure that circulates has no
  source behind it.)
- **Middle maps** blend close-range ground detail (GDT) into distant satellite
  imagery. Without one the transition is visible.
- **Shores** via the terrain tool shore map generator. Ocean wave simulators
  only spawn waves on shores physically facing open sea.
- **Q key** snaps prefabs along terrain normals.

## Registering a surface material — the drag rule

**Right-click the layer list -> "Add Material(s)" does not persist.** The
material appears in the Paint list, survives a save, and is silently gone on
reload. Nothing warns you.

**Drag the `.emat` from the Resource Browser onto the Paint layer list
instead.** That is what writes it into `<World>.terr`, and it sticks.

This cost five hours across two machines. Nothing in Bohemia's documentation
mentions the distinction, and every downstream symptom points somewhere else:
the Change Layers Order dialog lists only materials already in the `.terr`, so
newly "added" ones never appear and the dialog looks broken; and the mask
importer reports **"5 surface masks imported successfully"** while doing
nothing, because it silently skips masks whose material is not registered.

**Verify against the file, not the UI.** `<World>.terr` is small and binary,
but its `.emat` paths are plain readable strings:

```
grep -a '\.emat' Worlds/<World>/<World>.terr
```

Every material you expect must be in there before importing anything. The Paint
panel's list is not evidence.

## Importing surface masks

- **The default surface is the first layer.** It covers every block 100% by
  definition, cannot be removed, and **cannot take a mask**. Import masks only
  for the other layers. Make the most widespread surface the default and save a
  slot.
- **"Priority surface mask import"** (right-click a single layer) is the
  per-layer route and has **no file picker** — it reads the `Import/export
  directory` field and looks for `<LayerName>.png`. So mask filenames must match
  layer names exactly, and a stale file of the right name in that folder will be
  imported in preference to the one you meant.
- **Export first to learn the target resolution.** Export writes the terrain's
  current masks at exactly the size the importer expects. Generate to match that
  number rather than to the heightmap's raster size.
- Masks are 8-bit greyscale PNG, colour type 0, one per material.
- **Import is memory-hungry beyond any obvious proportion.** A single 0.66 MB
  mask was enough to crash an 8 GB integrated-graphics laptop. Both the batch
  and per-layer routes died identically. This step genuinely requires a real
  GPU; there is no incremental workaround.

**Confirming a bake actually happened.** Check `Worlds/<World>/.Data/*_layer.edds`.
An unpainted tile is **411 bytes** — and Explorer rounds that to "1 KB", which
reads as growth if you are hoping for it. Compare real byte counts, and check
file timestamps against when the import ran.

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
