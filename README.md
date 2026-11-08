# Hermes Integration Plugin

Embed Nous Research's Hermes Agent web UI directly in the UE5 editor. Access your local Hermes AI assistant without leaving Unreal Engine.

**Latest: v6 (plugin v1.0.2)** — `hermesintegration-ue5.8-v6.zip`

## What's New in v6 (plugin v1.0.2)

- **Rebuilt for UE 5.8.3** — the shipped Win64 DLL was rebuilt with the current toolchain and verified loading from `Engine/Plugins/Marketplace/` (the path Fab installs to), not just from a project `Plugins/` folder.
- **Always-visible panel header** — a Nous Portal account link plus **Open in browser** and **Reload** buttons, so a stale or blank panel can be recovered without restarting the editor.
- **Honest account wording** — the header link opens the Nous Portal in your browser for account/subscription management (Google, Microsoft, GitHub). The panel itself talks to your local dashboard and needs no account.
- **New troubleshooting entry** for the dashboard sign-in page: it only appears when `dashboard.public_url` is set (auth gate on), and `hermes config set dashboard.public_url ""` clears it.
- **Config/FilterPlugin.ini** now included in the package so documentation survives packaging, and a **CHANGELOG.md** was added.
- `PlatformAllowList` narrowed to Win64 to match the shipped binary.

## What Was New in v5 (plugin v1.0.1)

- **Blank-panel fix** — the plugin now launches the Hermes **dashboard** (the browser UI) instead of `hermes serve` (the headless backend, which serves no web page). This was the cause of the empty panel.
- **Finds Hermes automatically** — resolves the Hermes executable via PATH *or* the standard local install, so auto-launch works even when `hermes` isn't on your PATH.
- **Reliable first load** — waits for the dashboard's one-time web-UI build and reloads when ready; shows an actionable message instead of a blank window on timeout.
- **Blueprint API** — `UHermesExportLibrary` exposes the asset exporter to Editor Utility Blueprints (*Export Assets To Folder*, *Export Selected Assets To Folder*).
- **Pre-compiled binaries included** — ready-to-use `UnrealEditor-HermesIntegration.dll` for UE 5.8 (no build step required).
- Added a PDF manual and a 5.4–5.8 packaging script.

## What Was New in v4

- **Asset Exporter** — Select assets in the Content Browser, then run `Hermes.ExportSelected` (or use the menu: *Tools → Hermes → Export Selected Assets…*) to copy them as `.uasset` files along with their hard package dependencies to any folder. Re-import the bundle into another Unreal project, or use it as a hand-off package.
- This unblocks the "exporting Unreal Engine files" capability on the Fab listing.

## Features

- **Embedded Web UI** — Full Hermes web interface runs inside an editor panel
- **Asset Exporter** — Copy selected Content Browser assets and their hard package deps as `.uasset` files to a folder
- **Auto-Detection** — Detects existing Hermes server on port 9119
- **Auto-Launch** — Starts the Hermes dashboard if not already running (uses `hermes` from PATH, or the default local install)
- **Blueprint API** — Drive the asset exporter from Editor Utility Blueprints (`UHermesExportLibrary`)
- **Zoom Controls** — Ctrl+= and Ctrl+− to adjust zoom level (remembered across restarts)
- **Header Controls** — Open in browser / Reload, plus a Nous Portal account link
- **Pre-compiled** — Ships compiled `UnrealEditor-HermesIntegration.dll` for UE 5.8 Win64 (source included for other platforms)

## Installation

### Prerequisites

1. **Hermes Agent** — Install from https://hermes-agent.nousresearch.com/ or via:
   ```powershell
   iex (irm https://hermes-agent.nousresearch.com/install.ps1)
   ```

2. **Initial Setup** — Run once to configure providers and API keys:
   ```bash
   hermes setup
   ```

### Enable the Plugin

1. Install from Fab (installs to `Engine/Plugins/Marketplace/`), or copy the `HermesIntegration/` folder into your project's `Plugins/` directory
2. Enable it under **Edit → Plugins → Editor** (or add to your project's plugin list)
3. Restart the UE5 editor to load the plugin

## Usage

1. **Open the Panel** — Click the **Hermes** button in the toolbar (next to Play button) or run `Hermes.Open`
2. **Interact** — Use the embedded Hermes web UI just like the standalone version
3. **Manage Windows** — The panel opens in a floating window (800x600) or brings existing window to front
4. **Close** — Close the window normally; click Hermes button again to reopen

### Exporting Assets

1. Select one or more assets in the Content Browser
2. Run the export via any of:
   - Menu: *Tools → Hermes → Export Selected Assets…*
   - Console command: `Hermes.ExportSelected`
3. Pick a destination folder in the dialog
4. The plugin writes each package (`.uasset` / `.umap`) plus sidecar files (`.uexp`, `.ubulk`, `.uptnl`) and resolves hard package dependencies via the Asset Registry

The destination folder structure mirrors the project's `Content/` layout, so re-importing is just a folder copy into the receiving project's `Content/`.

## How It Works

- The plugin spawns the Hermes dashboard via `hermes dashboard --port 9119 --host 127.0.0.1 --no-open` if one isn't already running (the dashboard serves the browser UI; `hermes serve` is the headless backend and shows no page)
- The Hermes web UI connects to the server via WebSocket/JSON-RPC
- The embedded browser (CEF) displays the full web interface
- All your existing Hermes configuration (~/.hermes) is reused — the plugin stores no credentials

## Troubleshooting

### "Hermes not found"
- Ensure `hermes` is installed and in your system PATH
- Run `hermes setup` from your shell to configure providers

### "Failed to bind to port 9119"
- Check if another Hermes instance is already running on that port
- Run `hermes dashboard --status` to check running servers

### The panel asks you to sign in
The panel loads your local dashboard, which normally needs no sign-in. A login page means
`dashboard.public_url` is set in your Hermes config (that turns the auth gate on). Clear it:

```bash
hermes config set dashboard.public_url ""
```

then restart the dashboard and press **Reload** in the panel. If you *want* the gate (remote
access), use the **Sign in with Nous Research** button on that page — it handles Google,
Microsoft and GitHub via the Nous Portal — and make sure the callback URL is reachable from
the browser doing the sign-in.

### Plugin fails to load
- Ensure UE5.8+ is being used
- Check the editor logs (Window > Developer Tools > Output Log) for `LogHermes` lines
- Rebuild the plugin: Close editor → right-click `.uproject` → Generate Visual Studio project files → Build

## Architecture

```
HermesIntegration/
├── Source/HermesIntegration/
│   ├── Public/
│   │   ├── HermesIntegration.h       (module interface)
│   │   ├── HermesIntegrationModule.h (plugin module)
│   │   ├── HermesManager.h           (Hermes process management)
│   │   ├── HermesPanel.h             (Slate UI widget)
│   │   ├── HermesAssetExporter.h     (Content Browser → .uasset exporter)
│   │   └── HermesExportLibrary.h     (Blueprint-callable API)
│   └── Private/
│       ├── HermesIntegrationModule.cpp
│       ├── HermesManager.cpp
│       ├── HermesPanel.cpp
│       ├── HermesAssetExporter.cpp
│       └── HermesExportLibrary.cpp
├── Config/
│   └── FilterPlugin.ini
├── Documentation/
│   ├── BUILDING.md                    (BuildPlugin steps, UE 5.4–5.8)
│   ├── CHANGELOG.md
│   ├── CONSOLE_COMMAND.md
│   ├── QUICKSTART.md
│   ├── README.md
│   └── HermesIntegration_Manual.pdf
├── Binaries/
│   └── Win64/
│       ├── UnrealEditor-HermesIntegration.dll
│       └── UnrealEditor.modules
├── Resources/
│   └── Icon128.png
├── HermesIntegration.uplugin
└── README.md
```

## Platform Support

- **Windows (x64)** — prebuilt editor binary for UE 5.8 included; verified loading from `Engine/Plugins/Marketplace/`
- **Linux / macOS** — compile from the included source (CEF-based; the plugin code is platform-agnostic)

## Configuration

Zoom level is stored in the Hermes manager and persists across editor restarts. All other Hermes configuration is read from `~/.hermes/` (standard Hermes config directory).

## License

This plugin integrates Nous Research's Hermes Agent. Refer to Hermes licensing for details.
