# ARPMAN — Ableton Extensions Project

## Who is the user?

Captain (Joerg). Alias **ARPMAN** for developer identity, **WireDrifter** for YouTube music channel. Both appear in the extensions — ARPMAN in context menus and manifests, WireDrifter in the footer link.

## Project overview

Eight Ableton Live Extensions built with the **Extensions SDK 1.0.0-beta.0** (Node.js / TypeScript). All published on the Ableton Discord `#extensions-community-gallery` channel.

All UI text is in **English** (switched from German for international distribution).

---

## Folder structure

```
/Users/jd/Cowork/AbletonExtensions/
├── set-info/           Extension: live set overview (tempo, scale, tracks)
├── scale-filter/       Extension: remove or snap MIDI notes to scale
├── clip-info/          Extension: MIDI clip statistics popup
├── clip-colors/        Extension: colorize clips on a track
├── midi-device-lookup/ Extension: device-specific CC/NRPN lookup (pencilresearch/midi)
├── midi-invaders/      Extension: Space Invaders game using scale notes
├── note-forge/         Extension: MIDI note generator written to clip
├── bbc-sound-effects/  Extension: search & download BBC sound effects (display name: BBC SFX Browser)
├── screenshots/        Screenshots for GitHub release pages (PNG, named by extension)
├── deploy/             Built .ablx files + documentation
│   ├── GITHUB_RELEASE.md   How to create and update GitHub Releases
│   └── DISCORD_POSTS.md    Ready-to-paste Discord post texts for all extensions (single source of truth)
├── deploy.sh           Builds all extensions and copies .ablx to deploy/
├── versions.json       Central version registry for all extensions
├── README.md           GitHub landing page with all extension listings
├── sdk/                SDK source and API docs (read-only reference)
└── CLAUDE.md           This file
```

---

## Extensions

### set-info
- **manifest name:** `set-info` | **version:** 1.0.6
- **Context menu:** right-click MidiTrack or AudioTrack → "Set Info (ARPMAN)"
- **Command:** `setinfo.show`
- **Dialog size:** 300 × 310
- Shows: tempo, scale, track count, return tracks, scenes
- Footer: `© 2026 ARPMAN · WireDrifter · v1.0.0`

### scale-filter
- **manifest name:** `scalefilter` | **version:** 1.0.10
- **Context menu:** right-click MidiClip → "Remove (ARPMAN)" / "Snap to Scale (ARPMAN)"
- **Commands:** `scalefilter.remove`, `scalefilter.snap`
- Remove: deletes notes outside the active Live scale
- Snap: moves out-of-scale notes to nearest scale note (tie: snaps UP)
- Uses `song.rootNote`, `song.scaleIntervals`, `song.scaleName`

### clip-info
- **manifest name:** `clip-info` | **version:** 1.0.7
- **Context menu:** right-click MidiClip → "Info (ARPMAN)"
- **Command:** `clipinfo.show`
- **Dialog size:** 320 × 490
- Shows: note count (with muted badge), velocity min/avg/max bar, pitch range, top 5 pitch classes
- `clip.notes` returns float velocities — always use `Math.round()` on velocity values
- Duration formatted as "X Bars + Y Beats" (assumes 4/4: 4 beats per bar)

### clip-colors
- **manifest name:** `clipcolors` | **version:** 1.0.6
- **Context menu:** right-click MidiTrack/AudioTrack → "Color Clips (ARPMAN)" / "Color Settings (ARPMAN)"
- **Commands:** `clipcolors.applyOnMidi`, `clipcolors.applyOnAudio`, `clipcolors.showSettings`
- **Dialog size:** 380 × 720
- Settings persisted to `~/.ableton-clip-colors.json`
- Color families: Red, Orange, Yellow, Green, Blue, Purple, Teal (5 shades each)
- Modes: Fixed (first shade), Random, Cycle (steps through shades)
- Separate color family for Audio and MIDI clips
- Footer CSS class: `.copyright` (different from `.footer` used in other extensions)

### midi-device-lookup
- **manifest name:** `midi-device-lookup` | **version:** 1.0.17
- **Context menu:** right-click MidiTrack or AudioTrack → "MIDI Device Lookup (ARPMAN)"
- **Command:** `mididevice.show`
- **Dialog size:** 980 × 660
- **Permissions:** `network` (in manifest)
- **Offline-first architecture:**
  - `scripts/fetch-data.mjs` fetches all device data from api.midi.guide in a single request, saves as `src/deviceData.json` (~2.2 MB, compact array format)
  - `build.ts` embeds `src/deviceData.json` as `__DEVICE_DATA__` constant via esbuild define
  - `extension.ts` writes HTML (with embedded `const DATA = {...}`) to `storageDirectory/dialog.html`, opens via `file://` URL
  - If `storageDirectory/deviceData.json` exists (after Update Database), it takes priority over bundled data
- **UI:** Full device list on open (399 devices grouped by manufacturer) — search filters in real time
  - Search chips bar shows last 8 searches (localStorage `mdl-history`) — click to re-run
  - Click device → parameter table with filter input
  - Click any param row → copies CC/NRPN number to clipboard
  - "Copy TSV" button → copies full (or filtered) param table as tab-separated text
  - Back button returns to search with previous query active
  - Section dividers in param table (dark rows, no section column)
  - Update Database button: fetches data from api.midi.guide in one request, sends JSON via `close_and_send("SAVE:...")`, extension saves and reopens
  - After update: compares new data against bundled `__DEVICE_DATA__`, injects `NEW_KEYS` set → green "✦ New (N)" chip + "NEW" badge on new devices in list
  - NRPN filter chip: shows only devices with NRPN parameters
  - Section chips in device view: one chip per section, click to filter table to that section
  - Params count shown as green badge; NRPN shown as purple badge (right-aligned)
  - "Device not listed?" contribute link shown when search returns no results
- **Rebuild data:** `npm run fetch-data && npm run build:dev`
- Param array format: `[section, name, desc, ccMsb, ccLsb, ccMin, ccMax, nrpnMsb, nrpnLsb, nrpnMin, nrpnMax, usage, notes]` — trailing empty fields trimmed
- **CRITICAL:** `\n`/`\r` in TypeScript template literals become literal newline chars in HTML output → JS syntax error → nothing works. Always use `\\n`/`\\r` inside template literals that contain embedded JS strings.

### midi-invaders
- **manifest name:** `midi-invaders` | **version:** 1.0.14
- **Context menu:** right-click MidiClip → "MIDI Invaders (ARPMAN)"
- **Command:** `midiinvaders.play`
- **Dialog size:** 680 × 530
- Space Invaders-style game using the clip's active scale notes as the note targets
- Builds scale pitches from `song.rootNote` + `song.scaleIntervals` (8 notes starting at C3 + rootNote)
- Canvas-based game rendered inside the modal dialog

### note-forge
- **manifest name:** `note-forge` | **version:** 1.0.11
- **Context menu:** right-click MidiClip → "Note Forge (ARPMAN)"
- **Command:** `noteforge.open`
- **Dialog size:** 760 × 500
- MIDI note generator: user configures pattern, rhythm, velocity in dialog
- Writes generated notes back to the clip (replace or append mode)
- Renames clip to `NF-<Root>-<Scale>` after writing
- Falls back to C Major intervals if scale mode is off
- Uses `song.rootNote`, `song.scaleIntervals`, `song.scaleName`, `song.scaleMode`

### bbc-sound-effects
- **manifest name:** `bbc-sound-effects` | **display name:** BBC SFX Browser | **version:** 0.2.27
- **Context menu:**
  - right-click AudioTrack or AudioTrack.ArrangementSelection → "BBC SFX Browser (ARPMAN)" (drops as clip)
  - right-click Simpler → "BBC SFX Browser (ARPMAN)" (loads into Simpler sample slot)
- **Commands:** `bbc.open-clip`, `bbc.open-simpler`
- **Dialog size:** 600 × 460 (main), 480 × 200 (error dialog)
- **Permissions:** `filesystem`, `network` (declared in manifest)
- Searches the BBC Sound Effects library via network, downloads WAV/MP3, places in track or Simpler
- Uses a separate `src/dialog.html` file (not an inline HTML string)
- External HTML file imported via `import dialogHtml from "./dialog.html"` — requires build config support
- Uses CSS variables for theming (see Unified Color Scheme section)
- **Attribution:** Based on original work by ancientplaces (MIT). Display name changed from "BBC Sound Effects" to "BBC SFX Browser" at ancientplaces' request. GitHub tag stays `bbc-sound-effects` to preserve download URLs.

---

## SDK key facts

- Entry point: `export function activate(activation: ActivationContext)`
- Init: `const context = initialize(activation, "1.0.0")`
- Context menu scopes: `"MidiClip"`, `"MidiTrack"`, `"AudioTrack"`, `"AudioTrack.ArrangementSelection"`, `"Simpler"`
- Register menu: `context.ui.registerContextMenuAction(scope, label, commandName)`
- Register command: `context.commands.registerCommand(name, handler)`
- Get object: `context.getObjectFromHandle(arg as Handle, MidiClip)`
- Song data: `context.application.song` → `.tempo`, `.rootNote`, `.scaleIntervals`, `.scaleName`, `.scaleMode`, `.tracks`, `.scenes`, `.returnTracks`
- Modal dialog: `context.ui.showModalDialog(url, width, height)` — returns Promise<string>
- Dialog HTML passed as `data:text/html,${encodeURIComponent(html)}`
- Close dialog from HTML: `window.webkit.messageHandlers.live.postMessage({ method: "close_and_send", params: ["ok"] })`
- MIDI pitch: 0 = C-2, 60 = C3 (Middle C). Octave = `Math.floor(pitch / 12) - 2`
- `declare const __APP_VERSION__: string;` — injected at build time, used in newer extensions

---

## Unified Color Scheme

All extensions use the same dark blue-tinted color palette. For extensions with inline HTML (extension.ts), use these hardcoded values. For bbc-sound-effects (dialog.html), use the CSS variables.

| Role | Value |
|---|---|
| Main background | `hsl(240,15%,8%)` / `#0d0d15` |
| Surface / raised | `hsl(240,12%,13%)` / `#1a1a22` |
| Sunken | `hsl(240,18%,5%)` / `#08080d` |
| Border subtle | `hsl(240,20%,6%)` |
| Border mid | `hsl(240,15%,18%)` / `#262633` |
| Text primary | `hsl(0,0%,85%)` / `#d9d9d9` |
| Text dim | `hsl(240,10%,50%)` / `#737385` |
| Accent (ARPMAN magenta) | `hsl(300,100%,50%)` |
| Accent dim | `hsl(300,100%,38%)` |
| Secondary blue | `hsl(210,80%,65%)` |

**Do not use neutral grays** (`#1e1e1e`, `#2a2a2a`, `#252525`) — they make dialogs look flat and inconsistent. Always use the blue-tinted equivalents above.

---

## Build & deploy

### Dev mode (live reload in Ableton)
```bash
cd /Users/jd/Cowork/AbletonExtensions/<extension-name>
npm start
```
Requires Developer Mode enabled in Ableton: Preferences → Extensions → Developer Mode.

### Build single package
```bash
cd /Users/jd/Cowork/AbletonExtensions/<extension-name>
npm run package
```
Creates `<name>-<version>.ablx` in the extension folder and bumps the patch version.

### Deploy all extensions
```bash
bash /Users/jd/Cowork/AbletonExtensions/deploy.sh
```
Builds all extensions, copies `.ablx` to `deploy/`, deploys to Ableton, uploads to GitHub Releases, pushes `versions.json`.

### Deploy single extension manually
```bash
cd /Users/jd/Cowork/AbletonExtensions/<name>
npm run package
cp dist/extension.js "/Users/jd/Library/Application Support/Ableton/Extensions/arpman.<name>/dist/extension.js"
cp <name>-*.ablx /Users/jd/Cowork/AbletonExtensions/deploy/<name>.ablx
gh release upload <name> deploy/<name>.ablx --clobber --repo jdaehler/abletonliveextensions
```
Then sync versions.json (see Versioning section).

### Install in Ableton
Drag the `.ablx` file into Ableton Preferences → Extensions.

---

## Versioning

Each extension has its own version in `manifest.json` and `package.json`. Central registry: `versions.json` at project root — also hosted on GitHub, used by all extensions for the update check.

`npm run package` bumps the patch version automatically. `npm run build` does NOT bump.

**After any build that changes the running extension, sync versions.json:**
```bash
node -e "
const fs = require('fs'), base = '/Users/jd/Cowork/AbletonExtensions';
const names = ['set-info','scale-filter','clip-info','clip-colors','bbc-sound-effects','midi-invaders','note-forge','midi-device-lookup'];
const v = {};
for (const n of names) v[n] = JSON.parse(fs.readFileSync(base+'/'+n+'/package.json','utf8')).version;
fs.writeFileSync(base+'/versions.json', JSON.stringify(v,null,2)+'\n');
console.log(v);
"
SHA=$(gh api repos/jdaehler/abletonliveextensions/contents/versions.json --jq '.sha')
CONTENT=$(base64 -i /Users/jd/Cowork/AbletonExtensions/versions.json)
gh api repos/jdaehler/abletonliveextensions/contents/versions.json \
  --method PUT --field message="Sync versions.json" \
  --field content="$CONTENT" --field sha="$SHA" --jq '.commit.sha'
```

If versions.json is out of sync, all extensions show "Update available" on open — even when up to date.

---

## GitHub Releases & Screenshots

### Release structure
Each extension has a GitHub Release with tag = manifest name (e.g. `midi-device-lookup`).
- Download URL (permanent): `https://github.com/jdaehler/abletonliveextensions/releases/download/<tag>/<name>.ablx`
- Release notes contain: description, feature list, install instructions, download link, WireDrifter credit, screenshots, "All extensions" link
- Landing page (README): `https://github.com/jdaehler/abletonliveextensions`

### Updating screenshots
Screenshots live in `screenshots/` — named by extension (e.g. `midi-device-lookup.png`, `midi-device-lookup2.png`).

To upload new screenshots and rebuild release notes:
```bash
gh release upload <tag> screenshots/<name>.png --clobber --repo jdaehler/abletonliveextensions
```
When updating release notes programmatically, always preserve the screenshots and the "All extensions" link at the end — the `sed '/^---$/,$d'` trick strips everything after the first `---`, so re-append images and the link manually after.

### Adding a new release (first time)
```bash
gh release create <tag> --title "<Display Name>" --notes "<description>" --repo jdaehler/abletonliveextensions
gh release upload <tag> deploy/<name>.ablx --repo jdaehler/abletonliveextensions
```

---

## Branding conventions

- Context menu labels: `"Action Name (ARPMAN)"` — always include `(ARPMAN)` suffix
- Footer in every dialog: `© 2026 ARPMAN · <a href="https://youtube.com/@WireDrifter">WireDrifter</a> · v1.0.x`
- CSS class for footer: `.footer` (most extensions) or `.copyright` (clip-colors)
- Title color: `hsl(300,100%,50%)` (ARPMAN magenta)
- All strings English — no German in UI or context menus
- All dialogs dark — colors are hardcoded, no light mode support

---

## Publishing

- Platform: Ableton Discord → `#extensions-community-gallery`
- Tags used: `Utility` (all), `Workflow` (midi-device-lookup, bbc-sfx-browser), `Destructive` (scale-filter only)
- Post format: emoji + name, description, bullet list of features, install + download link, WireDrifter credit
- After posting: update existing post (edit) rather than deleting — replies stay attached

---

## Known SDK quirks

- `clip.notes` velocity values are floats (e.g. 27.420454) — always `Math.round()`
- `npm start` must be restarted after writing a new `extension.ts` (compiles stale file otherwise)
- The npx project wizard creates files IN the current directory, not a subfolder — always `cd` into the extension folder first
- Live shows extension name as prefix in context menu: `"scalefilter: Snap to Scale (ARPMAN)"`
- No official marketplace yet; extforlive.com is unofficial
- For extensions needing `network` or `filesystem` access, declare `"permissions"` array in `manifest.json`
- External HTML files (like bbc-sound-effects) require `import dialogHtml from "./dialog.html"` and appropriate build config
- `\n`/`\r` in TypeScript template literals → literal newlines in HTML output → JS parse error → nothing works. Use `\\n`/`\\r` in embedded JS strings inside template literals.
- All extensions show dialogs in dark mode only — colors hardcoded, unaffected by Ableton or system theme
- GitHub raw content (`raw.githubusercontent.com`) can be cached for a few minutes — version check may show stale data briefly after pushing versions.json
