# Hermes Integration Plugin

Embed Nous Research's Hermes Agent web UI directly in the UE5 editor. Access your local Hermes AI assistant without leaving Unreal Engine.

## What's New in v4 (plugin v1.0.1)

- **Blank-panel fix** — the plugin now launches the Hermes **dashboard** (the browser UI) instead of `hermes serve` (the headless backend, which serves no web page). This was the cause of the empty panel.
- **Finds Hermes automatically** — resolves the Hermes executable via PATH *or* the standard local install, so auto-launch works even when `hermes` isn't on your PATH.
- **Reliable first load** — waits for the dashboard's one-time web-UI build and reloads when ready; shows an actionable message instead of a blank window on timeout.
- **Blueprint API** — `UHermesExportLibrary` exposes the asset exporter to Editor Utility Blueprints (*Export Assets To Folder*, *Export Selected Assets To Folder*).
- Added a PDF manual and a 5.4–5.8 packaging script.

## What's New in v3

- **Asset Exporter** — Select assets in the Content Browser, then run `Hermes.ExportSelected` (or use the menu: *Tools → Hermes → Export Selected Assets…*) to copy them as `.uasset` files along with their hard package dependencies to any folder. Re-import the bundle into another Unreal project, or use it as a hand-off package.
- This unblocks the "exporting Unreal Engine files" capability on the Fab listing.

## Features

- **Embedded Web UI** — Full Hermes web interface runs inside an editor panel
- **Asset Exporter** — Copy selected Content Browser assets and their hard package deps as `.uasset` files to a folder
- **Auto-Detection** — Detects existing Hermes server on port 9119
- **Auto-Launch** — Starts the Hermes dashboard if not already running (uses `hermes` from PATH, or the default local install)
- **Blueprint API** — Drive the asset exporter from Editor Utility Blueprints (`UHermesExportLibrary`)
- **Zoom Controls** — Ctrl+= and Ctrl+− to adjust zoom level (remembered across restarts)
- **Cross-Platform** — Windows, Linux, and macOS support via Chromium Embedded Framework

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

1. The plugin is already in your project's `Plugins/HermesIntegration/` directory
2. Enable it under **Edit → Plugins → Editor** (it's listed in the project's plugins array)
3. Restart the UE5 editor to load the plugin

## Usage

1. **Open the Panel** — Click the **Hermes** button in the toolbar (next to Play button)
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
- All your existing Hermes configuration (~/.hermes) is reused

## Troubleshooting

### "Hermes not found"
- Ensure `hermes` is installed and in your system PATH
- Run `hermes setup` from your shell to configure providers

### "Failed to bind to port 9119"
- Check if another Hermes instance is already running on that port
- Run `hermes dashboard --status` to check running servers

### Plugin fails to load
- Ensure UE5.8+ is being used
- Check the editor logs (Window > Developer Tools > Output Log)
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
│   │   └── HermesAssetExporter.h     (Content Browser → .uasset exporter)
│   └── Private/
│       ├── HermesIntegrationModule.cpp
│       ├── HermesManager.cpp
│       ├── HermesPanel.cpp
│       └── HermesAssetExporter.cpp
├── HermesIntegration.uplugin
└── Binaries/ (generated after first compile)
```

## Platform Support

- **Windows** — WebView2 or CEF
- **Linux** — CEF
- **macOS** — CEF

## Configuration

Zoom level is stored in the Hermes manager and persists across editor restarts. All other Hermes configuration is read from `~/.hermes/` (standard Hermes config directory).

## License

This plugin integrates Nous Research's Hermes Agent. Refer to Hermes licensing for details.
