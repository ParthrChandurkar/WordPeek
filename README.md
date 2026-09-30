# WordPeek

WordPeek is a Windows desktop companion for people learning English from local video. It follows VLC playback, renders synchronized words from an adjacent SRT file in a transparent Electron overlay, and displays a concise English definition when the user hovers over a word.

WordPeek does not play video itself. VLC remains responsible for media playback.

## Features

- Secure Electron main, preload, and renderer separation
- VLC 3 integration through its password-protected local HTTP interface
- Windows VLC window tracking and a non-focusable, always-on-top overlay
- SRT discovery, parsing, synchronization, and tokenization
- Mouse pass-through outside interactive subtitle words and definition cards
- English definitions from the Free Dictionary API
- In-memory caching of successful and not-found dictionary results
- Unit, integration, and Electron smoke tests

## Architecture

The main Electron process communicates with VLC, reads subtitle files, performs dictionary requests, and controls the overlay window. A limited preload bridge exposes normalized data to the renderer. The renderer displays the active cue and temporarily enables interaction only around subtitle words and the definition card.

See [`docs/architecture.md`](docs/architecture.md) and [`docs/integration.md`](docs/integration.md) for the design and VLC integration notes.

## Getting Started

Prerequisites:

- Windows
- Node.js 22.12 or newer and npm
- VLC 3.x with its web interface enabled
- A local video and UTF-8 `.srt` file with the same basename

Install dependencies:

```powershell
npm install
```

In VLC, enable the **Web** main interface, configure a Lua HTTP password, and restart VLC. Set the same password for WordPeek, then start the application:

```powershell
$env:WORDPEEK_VLC_PASSWORD = 'your-local-vlc-password'
npm start
```

Keep the VLC HTTP interface bound to the local machine. WordPeek accepts only a loopback VLC URL.

## Configuration

| Variable | Purpose | Default |
| --- | --- | --- |
| `WORDPEEK_VLC_PASSWORD` | VLC HTTP password | Required |
| `WORDPEEK_VLC_URL` | VLC HTTP endpoint | `http://127.0.0.1:8080` |
| `WORDPEEK_POLL_INTERVAL_MS` | Playback polling interval | `250` |
| `WORDPEEK_WINDOW_POLL_INTERVAL_MS` | VLC window tracking interval | `250` |
| `WORDPEEK_DICTIONARY_TIMEOUT_MS` | Dictionary request timeout | `4000` |
| `WORDPEEK_HOVER_DEBOUNCE_MS` | Delay before lookup | `150` |
| `WORDPEEK_POPUP_GRACE_MS` | Pointer grace period for the popup | `120` |
| `WORDPEEK_DEBUG` | Shows diagnostic overlay information when set to `1` | Disabled |

## Subtitle Processing

WordPeek resolves only an adjacent SRT file with the same basename as the active video. It supports UTF-8 SRT files with LF or CRLF line endings, an optional byte-order mark, multiline cues, and basic emphasis tags. Malformed cue blocks are skipped. Embedded subtitles, differently named subtitles, and non-SRT formats are outside the current scope.

## Testing

```powershell
npm test
```

The test command runs the core test suite and an Electron smoke test. Core tests do not require VLC.

For the complete Windows/VLC workflow, use the checked-in
[`sample`](sample) media pair and follow the
[`VLC MVP acceptance checklist`](docs/vlc-mvp-acceptance.md).

## Limitations

- VLC reports playback time at whole-second precision through the selected interface.
- Window tracking follows the top-level VLC window rather than detecting the exact video area.
- Exclusive fullscreen and mixed-DPI cross-monitor behavior are experimental.
- Dictionary results depend on an external API and are not persisted between launches.
- Only the first useful English definition is displayed; translation and vocabulary storage are not implemented.

