# Verification

`SKILL.md` says to name the test that could falsify a claim before calling
something done. This applies that to the skill itself.

Each test below has a **falsification condition** — a specific wrong answer
that means something is broken, and which file to look at when it is. A test
you can't fail isn't evidence.

## How to run

**One fresh session per prompt.** Reusing a session only tests content, because
the skill is already loaded — triggering goes untested, and triggering is half
of what can break.

Record the date and the framework version the skill was built from, so a later
run can be compared against this one rather than against memory.

## Tests

### T1 — Ordered procedure
**Prompt:** `What order should my modifier stack be in for clothing over a base body?`
**Pass:** Shrinkwrap → Solidify → Subdivision Surface, in that order.
**Fails if:** the modifiers are named but unordered, or the order differs.
**Then check:** `blender-export.md`, modifier stack section.

### T2 — Symptom to cause
**Prompt:** `I renamed my world file and now it opens completely empty. What happened?`
**Pass:** the `_Layers` folder name derives from the world filename; renaming
one orphans the other. Bonus: notes a 0-byte `.ent` is normal.
**Fails if:** it suggests corruption, a Workbench bug, or re-importing.
**Then check:** `terrain-surfaces.md`, world files section.

### T3 — Diagnostic routing
**Prompt:** `My asset shrinks to nothing the moment I parent it to the armature.`
**Pass:** Delta Transform scale on the donor object; fix with Apply Scale (Ctrl+A).
**Fails if:** it suggests armature scale, unit settings, or import scale.
**Then check:** `rigging-animation.md`, armatures section.

### T4 — Specific values
**Prompt:** `How do I make Game Master orders override an AI's self-preservation?`
**Pass:** `UtilityComponent` priority, with +1000/+2000 modifiers named.
**Fails if:** it describes priority generally without those numbers — that means
it is reasoning rather than reading, and the numbers are the evidence.
**Then check:** `ai-behaviour.md`, utility and reactions section.

### T5 — Publishing
**Prompt:** `My project went read-only after publishing to Workshop. Why?`
**Pass:** `data.pak` present makes Enfusion mount the pak and treat the project
as a packed addon; move the pak and manifests out with Workbench closed.
**Fails if:** it suggests file permissions, antivirus, or read-only attributes.
**Then check:** `qa-validation.md`, packing and publishing.

### T6 — Regression on a known correction
**Prompt:** `My faction doesn't show up in the Game Master browser. Does that mean it isn't registered?`
**Pass:** **No.** That filter only lists factions holding entity catalogs, so a
registered faction with an empty catalog array is invisible there too. Should
point at a hostility test against a vanilla unit instead.
**Fails if:** it treats absence as evidence of failed registration — the exact
error this skill was corrected to remove on 2026-09-18.
**Then check:** `SKILL.md`, evidence rules.

### T8 — Method over parameters
**Prompt:** `My clothing is modelled in a T-pose but the Reforger skeleton rests in an A-pose. How do I fit it?`
**Pass:** Surface Deform — pose the armature to match the garment, bind, clear
the pose, let the body carry the garment into A-pose, then apply.
**Fails if:** it offers to rotate the sleeve vertices, calculate a shoulder
pivot, or write a rotation matrix. That is the failure mode this file exists to
prevent, and it looks superficially competent.
**Then check:** `character-clothing.md`.

### T7 — Negative control
**Prompt:** `What's the best way to parse JSON in Python?`
**Pass:** the skill does **not** load.
**Fails if:** it fires — the `description` is over-broad and will pull weight
into unrelated sessions.
**Then check:** the `description` field in `SKILL.md` frontmatter.

T7 is the one most worth keeping. Six passes prove the content is right; only
T7 proves the skill knows when to stay out of the way.

## Results

| Date | Framework ver. | T1 | T2 | T3 | T4 | T5 | T6 | T7 | Notes |
|---|---|---|---|---|---|---|---|---|---|
| 2026-09-18 | 1.6.1 | PASS | — | PASS | PASS | PASS | PASS | PASS | T1/T4 showed file reads; T6 answered without a visible read (likely same session as T4, so content verified but not triggering). T7 clean — no skill load, no Reforger content. T5 passed but revealed qa-validation.md was thinner than the known material; publishing section expanded same day. Only T2 unrun; terrain-surfaces.md already covered by the original surface-order test. |

**2026-09-18, later:** the `description` gained "fitting character clothing and
garments" when `character-clothing.md` was added. That changes what the skill
triggers on, so **T7 needs re-running** — the earlier pass no longer covers the
current description.

Re-run after porting a framework update — a change that silently drops a rule
shows up here and nowhere else.
