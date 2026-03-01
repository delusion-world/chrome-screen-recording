# Screencast for Claude Code

Record any application's screen as MP4 using OBS Studio, controlled directly from Claude Code. Supports application capture, window capture, and full display capture with auto-adapting resolution.

## Prerequisites

- **macOS** (uses ScreenCaptureKit for screen capture)
- **OBS Studio** - `brew install --cask obs`
- **Node.js** >= 18 - `brew install node`
- **Claude Code** CLI installed

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/delusion-world/screencast.git ~/.claude/plugins/screencast
```

Or clone anywhere you prefer — the plugin auto-discovers its script location.

### 2. Install dependencies

```bash
cd ~/.claude/plugins/screencast
bash scripts/setup.sh
```

### 3. Add to Claude Code

Copy the slash command and skill into your project or global Claude Code config:

**Option A: Project-level** (per-project)

```bash
# From your project root:
mkdir -p .claude/commands .claude/skills

# Copy slash command
cp ~/.claude/plugins/screencast/commands/screencast.md.example .claude/commands/screencast.md

# Copy skill
cp -r ~/.claude/plugins/screencast/skills/screencast .claude/skills/
```

**Option B: Global** (all projects)

```bash
mkdir -p ~/.claude/commands ~/.claude/skills
cp ~/.claude/plugins/screencast/commands/screencast.md.example ~/.claude/commands/screencast.md
cp -r ~/.claude/plugins/screencast/skills/screencast ~/.claude/skills/
```

### 4. Configure OBS

1. Open **OBS Studio**
2. Go to **Tools > WebSocket Server Settings**
   - Check **Enable WebSocket server**
   - Set port to **4455** (default)
   - Uncheck **Enable Authentication** (or set `OBS_WS_PASSWORD` env var)
3. Grant **Screen Recording** permission to OBS in **System Settings > Privacy & Security**

### 5. First-time setup

In Claude Code, run:

```
/screencast setup
```

This creates a "Screencast" scene in OBS with a screen capture source targeting Chrome by default.

## Usage

```
/screencast start                        # Start recording
/screencast stop                         # Stop recording, get MP4 path
/screencast status                       # Check connection, recording state, target app
/screencast setup                        # Create OBS scene (default: Chrome)
/screencast setup com.apple.Safari       # Create OBS scene for Safari
/screencast app com.apple.Safari         # Switch target to Safari
/screencast app                          # Show current target app
/screencast mode display                 # Switch to full screen capture
/screencast mode application             # Switch to app capture (default)
/screencast window                       # List all available windows
/screencast window "Chrome] mech"        # Capture a specific window by search
/screencast dir /path/to/output          # Set output directory
```

Or just say "record Safari" / "record my screen" and the skill triggers automatically.

## Capture Modes

### Application mode (default)

Records all windows of a specific app, even when behind other windows.

```
/screencast app com.google.Chrome
/screencast start
```

### Display mode

Records the entire screen.

```
/screencast mode display
/screencast start
```

### Window mode

Records a single specific window. Use the `window` command to list and select:

```
/screencast window                       # List all windows
/screencast window "Chrome] mech"        # Select window by search
/screencast start
```

The canvas resolution auto-adapts to match the captured source for pixel-perfect recording.

## Supported Applications

Switch recording target to any macOS application by bundle ID:

| App | Bundle ID |
|-----|-----------|
| Chrome | `com.google.Chrome` |
| Safari | `com.apple.Safari` |
| Firefox | `org.mozilla.firefox` |
| VS Code | `com.microsoft.VSCode` |
| Cursor | `com.todesktop.230313mzl4w4u92` |
| Figma | `com.figma.Desktop` |
| Terminal | `com.apple.Terminal` |
| iTerm2 | `com.googlecode.iterm2` |
| Slack | `com.tinyspeck.slackmacgap` |
| Discord | `com.hnc.Discord` |
| Zoom | `us.zoom.xos` |
| Finder | `com.apple.finder` |

To find any app's bundle ID:

```bash
osascript -e 'id of app "AppName"'
```

## How It Works

This plugin uses OBS Studio's WebSocket API (v5) to control recording:

1. **Setup** creates a "Screencast" scene in OBS with a macOS ScreenCaptureKit capture source
2. **App/Window/Mode** switches the capture target and mode
3. **Start** auto-adapts canvas resolution to match the source, then begins MP4 recording
4. **Stop** ends recording and returns the file path

The controller script (`scripts/obs-controller.mjs`) communicates with OBS via `obs-websocket-js` and outputs JSON for Claude to parse.

## Recordings

By default, recordings are saved to `/tmp/obs-recordings/`. Change with:

```
/screencast dir /path/to/your/preferred/directory
```

## Configuration

### OBS WebSocket Password

If you have authentication enabled on OBS WebSocket Server, set the password:

```bash
export OBS_WS_PASSWORD="your-password-here"
```

Add to your `~/.zshrc` or `~/.bashrc` to persist.

### Custom Install Path

If you cloned the repo to a non-standard location, set:

```bash
export SCREENCAST_DIR="/path/to/screencast"
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "ECONNREFUSED" | Open OBS and enable WebSocket Server |
| Auth failure | Disable authentication in OBS WebSocket settings, or set `OBS_WS_PASSWORD` |
| Black screen | Grant Screen Recording permission in System Settings > Privacy & Security |
| Empty file (20B) | App has no visible window — make sure it's open and not minimized |
| Source creation failed | Manually add a Screen Capture source in OBS |
| Unknown app | Run `osascript -e 'id of app "Name"'` to find bundle ID |

## License

MIT
