# Fitting Clothing to a Character

The most common character-modding task, and the one where people lose the most
work. Read this before touching a garment mesh.

## Where scripts run

Blender's own **Scripting** tab — top row of the window, scroll the tab strip
right if it is off screen, or click `+` → General → Scripting. Click **New** in
the Text Editor, paste, press the **▶** button (or Alt+P). Turn on
Window → Toggle System Console to see errors and `print()` output.

No external editor, add-on, or install is needed for any of this. If you are
being told to install VS Code to run a Blender script, that advice is wrong.

## The core problem

Clothing is almost always modelled in a **T-pose** (arms straight out).
Reforger's skeleton rests in an **A-pose** (arms angled down). Transferring
weights across that gap makes sleeve vertices sample from the torso instead of
the arm, and the garment tears.

**Do not fix this by moving vertices.** Rotating sleeves with hand-written
rotation matrices means guessing the shoulder pivot from a bounding box, and a
guess that is 2 cm off wrecks the deformation. Each correction compounds on the
last and it never converges. If you find yourself on a third rotation script,
the method is wrong.

## The fix: Surface Deform

Bind while the poses match, then let the body carry the garment into rest
position.

1. Remove **every** modifier from the garment first. Posing an armature while
   modifiers are live is what tears meshes.
2. Select the **Armature** → Pose Mode → rotate the arm bones **up** into
   T-pose, matching the garment.
3. Select the **garment** → add a **Surface Deform** modifier → Target:
   the body mesh → click **Bind**.
4. Back to the Armature → Pose Mode → **Pose → Clear Transforms**, returning to
   the A-pose rest.
5. The garment follows the body down into A-pose on its own.
6. Apply the Surface Deform modifier. The mesh is now physically A-pose.

No vertex maths, no pivot guessing. Only then do the weights.

## Weight transfer, in order

Order matters — bake before you bind:

1. Garment selected, add a **Data Transfer** modifier
2. Source: the body mesh · Vertex Data on · **Vertex Groups** ·
   mapping **Nearest Face Interpolated** (`POLYINTERP_NEAREST` in Python)
3. **Generate Data Layers**, then **apply** the modifier
4. *Then* add the **Armature** modifier pointing at the skeleton

Leaving an unapplied Data Transfer in the stack beneath a live Armature
modifier is a reliable way to shred the mesh.

Clamp the search distance (**Max Distance**, ~0.15 m) if sleeve tips are
grabbing torso weights across a gap.

## Never parent and modify at once

```python
obj.parent = armature                              # WRONG on its own
obj.modifiers.new(name="Armature", type='ARMATURE') # ...and wrong together
```

Two separate faults in those two lines.

**Assigning `.parent` in Python does not set `matrix_parent_inverse`.** The UI
route (Ctrl+P) stores the inverse of the parent's world matrix so the child
stays put. Python assignment skips that, so the child immediately inherits the
parent's full transform and jumps.

**And a parent relationship plus an Armature modifier applies the skeleton
twice.** Combined with weights sampled from a body that is offset in space,
limbs collapse toward the origin — a flat, stumpy, toy-like silhouette that
looks nothing like either input mesh.

**The Armature modifier alone is enough.** Do not parent as well. If you do need
a parent relationship for some other reason, set the inverse yourself:

```python
obj.matrix_parent_inverse = armature.matrix_world.inverted()
```

## Local space is not world space

The single most expensive trap in this whole area, and it bites twice: once when
fitting, and again when you try to measure whether the fit worked.

**A character mesh is usually parented to its armature.** That means its *world*
transform carries the armature's rotation and offset, while its *local* vertex
coordinates do not. A garment you imported separately is unparented, so for it
local and world are the same thing. The two meshes are then described in
different frames even though they appear in the same place on screen.

The symptom is a measurement that does not respond to the input. Compare
garment-local coordinates against body-local ones and you get a distance that
stays roughly constant no matter how you pose the skeleton — because you are
comparing points that were never in the same space.

Concretely, on a Reforger character whose armature carries a −102° Z rotation:
the body's arms run along **local X** and along **world Y**. Sampling "sleeve
vertices" by local Y picks out the front and back of the torso instead.

**Convert explicitly, every time:**

```python
world       = obj.matrix_world @ v.co
body_local  = body.matrix_world.inverted() @ world
```

`closest_point_on_mesh` and `BVHTree.FromObject` both work in the target's
**local** space. `obj.dimensions` is local too, while a bounding box you build
from `matrix_world @ bound_box` is world — which is exactly why those two can
disagree wildly on the same object and both be right.

**Before trusting any measurement, check that it moves when you move the
input.** A value that stays put while the pose changes is broken, not stable.

## Object transforms

A garment at `(0,0,0)` with a body and armature at some offset is a real
problem — the Armature modifier evaluates relative to the skeleton's origin, so
a mismatch is applied to every vertex.

**But do not "fix" it by copying the armature's location onto the garment.**

```python
obj.location = ARM.location   # WRONG — this MOVES the mesh
```

That translates the garment before anything is applied, which is where sudden
30 cm jumps come from. What you want is the visual position preserved while the
origins agree: apply transforms on the **body and armature** so they become
`(0,0,0)` without moving, rather than dragging the clothing to meet them.

Verify in the N-panel: Location `0,0,0`, Rotation `0,0,0`, Scale `1,1,1` on
everything, with nothing having visibly moved.

## Protecting your work

- **Never let a script delete objects.** Hide them (`hide_viewport = True`).
  A deletion inside a loop takes collision hulls and mesh variants with it, and
  hand-built `UCX_`/`UTM_` colliders are hours to rebuild.
- **Save incrementally** before running anything that edits geometry — Numpad
  `+` in Save As increments the filename.
- If something does get destroyed: **File → Recover → Auto Save…** Blender
  keeps rolling backups, and this has saved entire sessions.
- Work on one garment first. Batch loops across five pieces turn one mistake
  into five.
