# Upstream snapshots

Plain-text exports of ItsMeDingo's "Jarvis Framework" Google Doc, one file per
version. These exist **only to diff against** — nothing here is read as part of
the skill.

When a new framework version ships:

1. Open the upstream doc → **File → Download → Plain text (.txt)**
2. Save it here as `jarvis-framework-<version>.txt`
3. `git diff --no-index source/jarvis-framework-<old>.txt source/jarvis-framework-<new>.txt`
4. Port only the real changes into the matching reference file, then commit

Always export with the same method. Google Docs' plain-text exporter adds its
own escaping (`upgrade\_name`, `\[`, `-\>`) — consistent across exports, so it
diffs cleanly, but mixing export methods turns every line into a false change.

Note: the doc title and the `"version"` string inside it have disagreed before
(title said 1.6.1 while the JSON said 1.4.0). Name files by the **doc title**,
since that is what gets bumped on release.
