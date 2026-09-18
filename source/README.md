# Upstream snapshots

Plain-text exports of ItsMeDingo's "Jarvis Framework" Google Doc, one file per
version. These exist **only to diff against** — nothing here is read as part of
the skill.

When a new framework version ships:

1. Open the upstream doc → **File → Download → Plain text (.txt)**
2. Save it here as `jarvis-framework-<version>.txt`
3. `git diff --no-index source/jarvis-framework-<old>.txt source/jarvis-framework-<new>.txt`
4. Port only the real changes into the matching reference file, then commit

Always export the same way — File → Download → Plain text. That exporter emits
clean JSON (a leading UTF-8 BOM is normal), so snapshots diff line-for-line.
Don't substitute another method: reading the doc through a Drive connector, for
instance, escapes markdown characters (`upgrade\_name`, `\[`, `-\>`) and would
turn every line into a false change.

Note: the doc title and the `"version"` string inside it have disagreed before
(title said 1.6.1 while the JSON said 1.4.0). Name files by the **doc title**,
since that is what gets bumped on release.
