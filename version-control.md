# Git with an Enfusion Project

Worth doing: the world layer is a single text file holding every entity, and a
stale Workbench tab or a partial load silently overwrites it. Git makes that
recoverable. It is also required for Claude Code cloud sessions, which push a
branch and refuse uncommitted changes.

## Never let git rewrite line endings

Enfusion writes `.et`, `.conf`, `.layer` and `.emat` with **LF**. Git's default on
Windows converts to CRLF on checkout, so a Linux cloud session and a local
checkout then disagree about every file in the project.

```
# .gitattributes
* -text
```

```
git config core.autocrlf false
```

Do this before the first commit. Retrofitting means a noisy normalising commit
across the whole tree.

## Do not put the project in OneDrive

OneDrive syncs `.git` mid-operation and corrupts repositories. Since **Workbench
keeps an explicit project registry rather than scanning the addons folder**, the
project can live anywhere — move it somewhere outside any sync client.

The registry lives at `profile/.projectList_app<appid>_user<steamid>.conf` and
holds a plain list of `WBProjectListItem { FilePath "...addon.gproj" }`. Edit it
with **Workbench closed**; it rewrites the file on exit. There is usually one per
app id — update all of them.

Copy rather than move on the first migration, and keep the original until the new
location is proven to open.

## What to ignore

```
Worlds/<WorldName>/          # baked terrain, hundreds of MB of binaries
resourceDatabase.rdb         # rebuilt by Workbench
**/.EditorData/
**/.Data/
data.pak                     # publish output
*_manifest.json
ServerData.json
temp/
```

**This means the repo is not a full restore.** A fresh clone has no terrain and
the world will not open until that folder is put back. Say so out loud rather than
letting someone assume the repo is a backup. Keep the terrain somewhere else, or
accept that it is regenerable from the heightmap source.

## Committing

Commit after something *works*, so every commit is a state worth returning to.
Write what changed — "squad spawns correctly with six men" beats "update", because
the value of the log is finding the commit before something broke.

Workbench must not be open on files being changed externally, and a stale tab must
never be saved over a newer file on disk.
