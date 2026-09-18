# Rigging and Animation

## Environment setup

- **Enfusion Blender Tools (EBT)** — install the `.zip` from the Workbench
  install directory into **Blender 5.0 LTS**.
- Load **TXA animation export profiles** into Enfusion Tool Preferences from the
  `data.zip` package.
- Connect Workbench to Blender over **Net API** — enable it in Workbench
  Options and make sure the ports match on both sides.

## Armatures

- Use **Rigify** base armatures. Attach donor models with
  Make Parent → Object (**Keep Transform**).
- Align weapons to **`RightHandProp`** and magazines to **`LeftHandProp`** using
  bone constraints — Copy Transforms or Child Of.

**Delta scale trap:** always inspect Delta Transform scales on donor objects. If
an asset shrinks infinitely the moment you parent it, that's a non-1.0 delta
scale compounding. Fix with Apply Scale (Ctrl+A) in Blender.

## NLA retargeting

Retargeting from base-game actions runs through the **Non-Linear Animation
editor**:

1. Push base actions onto the NLA stack.
2. Create a combined blending action.
3. Apply weapon base poses.

**Additive actions** — fire mode switches, magazine manipulation — must be
exported with `Additive = true`, referenced against **frame 0**. Getting the
reference frame wrong makes the additive layer fight the base pose.

## Hand ownership during reloads

Weapon ownership shifts between hands mid-reload. Handle it by **keyframing
Child Of constraint influence from 0.0 to 1.0** across the hand attachment
points, so the weapon hands off cleanly rather than snapping.

## FBX export and import settings

Two settings that fail silently — the asset imports, nothing errors, and the
skeleton simply is not there:

- **On export:** under the Armature tab, **uncheck "Add lead bones."**
- **On import:** in the FBX's import settings, **check "Export Skinning"** or
  the skeleton does not come through.

Select everything the asset needs when exporting: the rig, the model, the
collider, and the empties that came with the skeleton. A missing empty is not
reported.

Bohemia ship character templates worth starting from rather than building
blind — `Head_Template.fbx` and `Character_Weights_Template.blend`, in the
`Arma-Reforger-Samples` repository under `SampleMod_NewCharacter`.

## AnimEvents

Use the Workbench script/animation tools to **transfer AnimEvent tracks from
donor base-game graphs**. Without them, sound triggers, magazine attachment
states and reload completion checks all desync — the animation plays but nothing
downstream fires.
