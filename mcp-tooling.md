# MCP Tooling Setup

Last verified: 2026-09-18.

Optional. Nothing else in this repo depends on these tools. Paths, package
versions, and extension folder names rot — if a step fails, assume this file
is stale before assuming you did it wrong.

Three servers, all registered in the same file:

    %APPDATA%\Claude\claude_desktop_config.json

Merge into the existing top-level `"mcpServers"` object — don't replace the
file. **A single missing comma stops every server loading, not just the new
one.** Validate the JSON before restarting.

**Notepad is fine for this** — no code editor needed. One trap: in Notepad's
Save As dialog set **Save as type: All Files**, or it silently appends `.txt`
and you end up editing a `claude_desktop_config.json.txt` that nothing reads.
Notepad++ or VS Code are nicer but change nothing about the outcome.

**Fully quit and reopen Claude Desktop** after editing. Closing the window is
not enough.

## 1. `enfusion-mcp` — live Workbench control

Repo: `steffenbk/enfusion-mcp-BK` or `Articulated7/enfusion-mcp`

```json
"enfusion-mcp": {
  "command": "cmd",
  "args": ["/c", "npx", "-y", "enfusion-mcp"]
}
```

One-time setup per project:

1. Workbench → File → Options → General → **Net API** → Enabled. Default port
   `5775` matches the connector default.
2. Run `wb_launch` pointed **explicitly** at the mod's `.gproj` — this copies
   handler scripts into `<mod>\Scripts\WorkbenchGame\EnfusionMCP\`.
3. **Restart Workbench.** Scripts copied into a running session don't compile
   until restart. This trips nearly everyone.
4. Confirm with `wb_connect` → "Workbench Connected."

`NETWORK (E): Failed to call not existing Net API function 'EMCP_WB_Ping'` means
the connection is fine but the handlers aren't compiled in — restart again.

## 2. `arma-reforger-api` — offline API/wiki search

Repo: `ViVi141/Arma_Reforger_Tools_MCP` (README in Chinese). **AGPL-3.0** —
relevant if this is ever redistributed or hosted.

1. Clone somewhere permanent, then `pip install -e .`
2. **Pin `mcp` to 1.x.** The code isn't 2.x-compatible and its dependency list
   has no upper bound, so a fresh install grabs the broken newest:
   ```
   pip uninstall mcp -y
   pip install "mcp>=1.0.0,<2.0.0"
   ```
3. Copy the API docs from the Steam install. **The files sit one level deeper
   than expected, inside `html\`** — copy that subfolder as the destination:
   ```
   <Steam>\Arma Reforger Tools\Workbench\docs\ArmaReforgerScriptAPIPublic\html
   <Steam>\Arma Reforger Tools\Workbench\docs\EnfusionScriptAPI\html
   ```
   Name the destinations `ArmaReforgerScriptAPIPublic` and
   `EnfusionScriptAPIPublic` — the source folder is now `EnfusionScriptAPI`
   without the suffix, but the code expects the old name. Thousands of small
   files; into a cloud-synced folder this takes minutes and is not a hang.
4. Build the index: `python -m src.parser.build_index --api-source both --fast`
   Should report thousands of files. **0 files found = docs are in the wrong
   place**, see step 3.
5. Register it (`cwd`, `API_DATA_PATH` and `PYTHONPATH` all point at the clone):
   ```json
   "arma-reforger-api": {
     "command": "python",
     "args": ["-m", "src.mcp_server.server"],
     "cwd": "<clone>",
     "env": {
       "API_DATA_PATH": "<clone>/data",
       "LOG_LEVEL": "INFO",
       "PYTHONPATH": "<clone>"
     }
   }
   ```
   The repo's own `generate_config.py` / `setup_cursor.ps1` only write Cursor's
   config — they won't touch `claude_desktop_config.json`. Add this by hand.

Note: the indexed API data is derived from Bohemia's documentation and is not
redistributable. Each machine builds its own index from its own Steam install.

## 3. `reforger-script-tools` — VS Code extension

Marketplace: `Burn0ut7.reforger-script-tools`. Enforce Script autocomplete and
compiler validation work in VS Code without any Claude registration.

**This is the only part of any of this that needs VS Code**, and it is entirely
optional. It helps when writing Enforce Script (`.c`) game code — autocomplete
and errors as you type. It does nothing for Blender work, terrain, or asset
import. Skip it without consequence if you are not writing scripts.

To let Claude call its `workbench_*` tools, register the bundled server:

```json
"reforger-script-tools": {
  "command": "C:\Users\<user>\.vscode\extensions\burn0ut7.reforger-script-tools-<version>\dist\server\win32-x64\reforger_language_server.exe",
  "args": ["mcp"]
}
```

**The `<version>` folder name changes on extension update** — if it stops
connecting after an update, check `%USERPROFILE%\.vscode\extensions\` for the
current folder name.

## Troubleshooting

- Tool absent entirely → did Claude Desktop fully quit and reopen?
- `wb_connect` fails → is Workbench running? Restarted since handlers copied?
- API search empty → did the index build report real numbers, or "0 files"?
- Everything broke at once → JSON syntax error in the config.
