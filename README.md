# AppleMusicRichPresence

A Vencord plugin that adds Discord Rich Presence support for Apple Music on macOS.

![Apple Music Rich Presence Demo](https://github.com/Vendicated/Vencord/assets/70191398/1f811090-ab5f-4060-a9ee-d0ac44a1d3c0)

## Features

- **Real-time track updates** — Shows currently playing track, artist, and album
- **Customizable activity** — Configure activity type (Playing/Listening), custom format strings for name, details, and state
- **Rich artwork** — Display album or artist artwork as large/small images with hover text
- **Interactive buttons** — "Listen on Apple Music" and "View on SongLink" buttons
- **Timestamps** — Shows progress bar with elapsed/remaining time
- **Smart link handling** — Clickable links to album, artist, or track on Apple Music
- **Radio support** — Detects and handles radio stations (no duration/progress)
- **Performance optimized** — Caches API responses to minimize network requests

## Requirements

- **macOS** (uses AppleScript to communicate with the Music app)
- **Apple Music app** running and playing music
- **Vencord** installed on Discord (Desktop/Client)

## Installation

### Via Vencord Plugin Manager (Recommended)

1. Open Vencord settings → Plugins
2. Click "Add Plugin" → "From GitHub"
3. Enter: `rafaelreverberi/AppleMusicRichPresence`
4. Enable the plugin and restart Discord

### Manual Installation

1. Download the latest release from [GitHub Releases](https://github.com/rafaelreverberi/AppleMusicRichPresence/releases)
2. Place the plugin folder in your Vencord plugins directory:
   - **macOS**: `~/Library/Application Support/Vencord/plugins/`
   - **Linux**: `~/.config/Vencord/plugins/`
   - **Windows**: `%APPDATA%\Vencord\plugins\`
3. Enable the plugin in Vencord settings and restart Discord

## Configuration

Access settings via Vencord Settings → Plugins → AppleMusicRichPresence.

### Activity Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Activity Type** | "Playing" or "Listening" | Playing |
| **Status Display** | What shows in member list: Off / Artist / Track | Off |
| **Refresh Interval** | Seconds between presence updates (1-15) | 5 |
| **Enable Timestamps** | Show progress bar with start/end times | Enabled |
| **Enable Buttons** | Show "Listen on Apple Music" / "SongLink" buttons | Enabled |

### Format Strings

Use these placeholders in format strings:

| Placeholder | Replaced With |
|-------------|---------------|
| `{name}` | Track name |
| `{artist}` | Artist name(s) |
| `{album}` | Album name |

**Default formats:**
- **Name**: `Apple Music`
- **Details**: `{name}`
- **State**: `{artist} · {album}`
- **Large Image Text**: `{album}`
- **Small Image Text**: `{artist}`

### Image & Link Settings

| Setting | Options | Default |
|---------|---------|---------|
| **Large Image** | Album Artwork / Artist Artwork / Disabled | Album Artwork |
| **Large Image Link** | Album / Artist / Disabled | Album |
| **Small Image** | Album Artwork / Artist Artwork / Disabled | Artist Artwork |
| **Small Image Link** | Album / Artist / Disabled | Artist |
| **Details Link** | Album / Artist / Disabled | Album |
| **State Link** | Album / Artist / Disabled | Artist |

## How It Works

### Architecture

```
┌─────────────────┐     AppleScript      ┌──────────────────┐
│  Vencord Plugin │ ◄──────────────────► │  Music.app       │
│  (TypeScript)   │  Track metadata      │  (macOS)         │
└────────┬────────┘                      └──────────────────┘
         │
         │ HTTPS (iTunes Search API)
         ▼
┌──────────────────┐
│  Apple/iTunes    │
│  Artwork & Links │
└──────────────────┘
```

### Data Flow

1. **Plugin starts** → Sets up interval based on `refreshInterval` setting
2. **On each interval** → Calls native module via `VencordNative.pluginHelpers.AppleMusicRichPresence.fetchTrackData()`
3. **Native module (native.ts)**:
   - Checks if `Music` app is running (`pgrep`)
   - Queries player state via AppleScript (`osascript`)
   - If playing, fetches track metadata (ID, name, album, artist, duration, position)
   - Calls iTunes Search API to get artwork URLs, Apple Music links, SongLink URL
   - Caches results by track ID (5 failure max before giving up)
4. **Plugin receives `TrackData`** → Builds Discord Activity object
5. **Dispatches `LOCAL_ACTIVITY_UPDATE`** → Discord shows rich presence

### Native Module (`native.ts`)

- Uses `osascript` to execute AppleScript commands against the Music app
- Fetches supplementary data from `https://itunes.apple.com/search`
- Scrapes artist artwork from Apple Music web pages (Open Graph meta tags)
- Implements caching with failure counting to avoid repeated failed requests

### Plugin Module (`index.tsx`)

- Defines all user settings via `definePluginSettings`
- Handles custom format string interpolation (`{name}`, `{artist}`, `{album}`)
- Builds `Activity` object with assets, buttons, timestamps, and links
- Dispatches updates via `FluxDispatcher`

## Development

### Building

This plugin is written in TypeScript and uses Vencord's plugin API. No build step required — Vencord loads TypeScript directly.

### Project Structure

```
appleMusic.desktop/
├── index.tsx      # Main plugin entry point, settings, activity building
├── native.ts      # Native module: AppleScript + iTunes API integration
└── README.md      # This file
```

### Type Definitions

```typescript
interface TrackData {
    name: string;
    album?: string;
    artist?: string;
    appleMusicLink?: string;
    appleMusicArtistLink?: string;
    songLink?: string;
    albumArtwork?: string;
    artistArtwork?: string;
    playerPosition?: number;
    duration?: number;
}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Plugin hidden" / not showing | Only works on macOS. Check `IS_MAC` constant. |
| No activity showing | Ensure Music app is open and playing (not paused) |
| Artwork not loading | Check internet connection; iTunes API may be rate-limited |
| Buttons not working | Enable "Enable Buttons" in settings; ensure track has Apple Music link |
| High CPU usage | Increase refresh interval (default 5s) |

## Privacy

- No data is collected or sent to third parties except:
  - **iTunes Search API** (Apple) — for track metadata and artwork
  - **Apple Music web pages** — for artist artwork (Open Graph scraping)
- All requests use Vencord's user agent
- No analytics, tracking, or telemetry

## License

GPL-3.0-or-later — See [LICENSE](LICENSE) for details.

## Credits

- **Author**: [RyanCaoDev](https://github.com/RyanCaoDev)
- **Vencord Team** — For the amazing plugin platform
- **Apple** — For the Music app and iTunes API

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

*This plugin is part of the [Vencord](https://vencord.dev) ecosystem.*