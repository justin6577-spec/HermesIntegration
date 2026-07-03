# Hermes Integration Plugin

Embed Nous Research's Hermes Agent web UI directly in the UE5 editor. Access your local Hermes AI assistant without leaving Unreal Engine.

## Features

- **Embedded Web UI** — Full Hermes web interface runs inside an editor panel
- **Auto-Detection** — Detects existing Hermes server on port 9119
- **Auto-Launch** — Starts Hermes if not already running (requires `hermes` in PATH)
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
2. Open `EchoesOfNorrath.uproject` — it's already listed in the plugins array
3. Restart the UE5 editor to load the plugin

## Usage

1. **Open the Panel** — Click the **Hermes** button in the toolbar (next to Play button)
2. **Interact** — Use the embedded Hermes web UI just like the standalone version
3. **Manage Windows** — The panel opens in a floating window (800x600) or brings existing window to front
4. **Close** — Close the window normally; click Hermes button again to reopen

## How It Works

- The plugin spawns a local Hermes server via `hermes serve --port 9119` if one isn't already running
- The Hermes web UI connects to the server via WebSocket/JSON-RPC
- The embedded browser (CEF) displays the full web interface
- All your existing Hermes configuration (~/.hermes) is reused

## Troubleshooting

### "Hermes not found"
- Ensure `hermes` is installed and in your system PATH
- Run `hermes setup` from your shell to configure providers

### "Failed to bind to port 9119"
- Check if another Hermes instance is already running on that port
- Run `hermes serve --status` to check running servers

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
│   │   └── HermesPanel.h             (Slate UI widget)
│   └── Private/
│       ├── HermesIntegrationModule.cpp
│       ├── HermesManager.cpp
│       └── HermesPanel.cpp
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
