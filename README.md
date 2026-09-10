# Longhaul Binge

An iPad app for binge-watching TV from your own Plex Media Server **offline**,
built for flights.

Named for the flight you actually need this on.

<img src="Longhaul/Resources/Assets.xcassets/AppIcon.appiconset/AppIcon-1024.png" width="120" alt="Longhaul app icon">

## Why this exists

I kept losing downloaded episodes on flights. Plex's own offline mode has been
unreliable for me across a number of app updates, and the failure always landed
at the same moment: on a plane, with no way to fix it.

So this app treats offline as the normal case rather than a degraded one. That
turns out to be a design stance more than a feature:

- An unreachable server never blocks the app. The Downloads tab stays fully
  usable and the Library tab explains itself.
- Launching with no network restores your session from a cached identity instead
  of dropping you at a sign-in screen you cannot complete.
- Artwork is cached to disk and prefetched when you download, so the offline
  library has posters instead of grey boxes.
- Intro and credits markers are stored alongside the video, so skip-intro and
  auto-advance work with no connection at all.
- Playback position is written continuously, so a force-quit at 30,000 feet
  costs you five seconds.

Every one of those is a place where "handle it when the network comes back" is
the easy choice and the wrong one. None of it is technically hard. It just has
to be decided on deliberately, in about a dozen separate places.

The whole thing is one person's spare-time project, so it is not a fair
comparison to a product with real constraints and a much larger surface. But it
does demonstrate that a reliable offline binge client for Plex is achievable,
which is the part I actually care about.

## Status

**Feature-complete for v1 and ready to sideload.** `.opencode/context-log.md` is
the state of record — every phase entry carries its commit SHA, decisions,
tests, and accepted caveats.

| Phase | Status | What it gives you |
| ----- | ------ | ----------------- |
| 0     | Done | Empty iPad app builds and tests run |
| 1     | Done | Sign in with Plex (PIN flow), token persisted in Keychain |
| 2     | Done | Browse your TV library: shows → seasons → episodes (with intro/credits markers) |
| 3     | Done | Background downloads, per-episode and per-season, with pause / resume / cancel |
| 4     | Done | VLCKit offline playback, skip-intro, Up Next overlay, auto-play countdown |
| 5     | Done | Binge mode: resume where you left off, watched tracking, cross-season roll-over |
| 6     | Done | Ship polish: storage management, app icon, audio session, idle-timer hold |
| 7     | Done | Settings screen, countdown options, Plex-style sleep timer |
| 8     | Done | Browse by collections |
| 9     | Done | Design tokens and a semantic icon vocabulary |
| 10    | Done | Offline fallback, downloads filter + transfer status menu, unified empty/error states |
| 11–12 | Done | A real offline library, with cached artwork and collection membership |
| 22    | Done | Multi-select delete in Downloads — episode by episode, or a whole season at once |

Verified on a physical iPad against a real Plex server: downloads, offline
playback, auto-advance across a full unattended season, and the offline library
in airplane mode.

## What it does

**Download.** Tap an episode, or "Download Season" for the whole thing.
Transfers run on a background `URLSession`, so they keep going when the app is
suspended. Pause, resume, cancel, and retry are swipe actions; the Downloads tab
also has a transfer status menu with Pause All / Resume All / Retry Failed, and
an All / Downloaded / In Progress filter so a season being fetched doesn't bury
the episodes you can already watch.

**Play offline.** VLCKit handles MKV/HEVC/DTS/PGS natively — no transcode, no
server round-trip. Files live in Application Support, excluded from iCloud
backup. Hardware keyboard shortcuts: space to play/pause, `x` to close, arrows
to skip ten seconds.

**Binge.** Plex's intro and credits markers are cached with the download, so
they work with no network. A Skip Intro button appears inside any intro marker.
When the credits start — or, for libraries Plex hasn't generated markers for,
near the end of the runtime — an Up Next card counts down and rolls into the
next episode, including across a season boundary.

**Pick up where you left off.** The playhead is persisted while you watch, so
closing the app, backgrounding it, or force-quitting costs you at most five
seconds. Part-played episodes surface on a Continue Watching shelf.

**Stop when you've fallen asleep.** An optional sleep timer (1–4 hours) stops
episodes advancing on their own after a stretch with no interaction. It never
interrupts playback: the episode you're watching always finishes, and the Up
Next card simply waits for a tap instead of counting down.

**Work with no connection at all.** This is the whole point, so it degrades in
layers rather than failing:

- An unreachable server never blocks the app. The Downloads tab stays fully
  usable and only the Library tab explains itself.
- The Library tab offline shows the shows you have episodes of and the
  collections those shows belong to, built entirely from local data.
- Artwork is cached to disk as you browse, and prefetched when you download, so
  the offline library has posters rather than grey boxes.
- A launch with no network restores your session from a cached identity instead
  of dropping you at the sign-in screen.

**Manage the disk.** The Downloads tab reports what the library occupies and
what's free, with bulk deletes for watched episodes or everything. **Select**
puts the list into multi-select: tick episodes one at a time, or tap a season
header to take the whole season, and the bottom bar shows how much space the
selection will give back before you confirm. The list is grouped by show *and*
season for exactly this reason — a season is the unit you finish and clear.
Anything still downloading in the selection is cancelled properly rather than
having its row yanked out from under a live transfer.

## Requirements

- iPadOS 17 or later
- A Plex Media Server you own (or have download permission on), with Plex Pass
  for marker support
- Xcode 26 or later to build (developed against Xcode 27 / Swift 6)
- [xcodegen](https://github.com/yonaskolb/XcodeGen) to generate the Xcode project

## Repository

```
git clone ssh://git@gitlab.goltech.io:2424/sgolub/longhaul.git
```

Private project on self-hosted GitLab. Note the non-default SSH port (2424); the
web UI is on 8929.

## Build

```bash
brew install xcodegen        # one-time
xcodegen generate            # regenerate Longhaul.xcodeproj from project.yml
open Longhaul.xcodeproj
```

The `.xcodeproj` is git-ignored on purpose — `project.yml` is the source of
truth. Never hand-edit the pbxproj; edit the YAML and regenerate.

## Run on a simulator

1. `xcodegen generate` (if you haven't yet, or if `project.yml` changed)
2. `open Longhaul.xcodeproj`
3. In Xcode's toolbar, pick a destination — e.g. **iPad Pro 13-inch (M5)** under
   **iOS Simulators**.
4. Press `⌘R`.
5. The app boots into a sign-in screen. Tap **Sign in with Plex**. An in-app
   browser opens to `app.plex.tv/auth` with a 4-letter code.
6. Sign in to plex.tv (existing cookies usually carry over from Safari). Tap
   **Allow** when Plex asks to authorize the app.
7. Tap **Done** in the in-app browser. The app resolves your server and lands on
   the library grid.

## Run on a physical iPad

This is the intended way to use it — downloads and playback are the point, and
both want real storage and a real screen.

1. In Xcode, select your iPad as the destination.
2. Open the **Longhaul** target → **Signing & Capabilities** → set **Team** to
   your Apple Developer team.
3. `⌘R`. First run requires unlocking the iPad and trusting the certificate in
   Settings → General → VPN & Device Management.

`project.yml` ships `DEVELOPMENT_TEAM: ""` deliberately, so the team ID stays
out of git. Xcode fills it into the generated project locally; regenerating
resets it.

## Known limitations

- **Sideload only.** iOS verifies a developer certificate over the network on
  first install, so a build cannot be *installed* while offline. Install with a
  connection, then go offline; it runs offline indefinitely after that.
- **Resuming a paused download trusts the stored URL.** If Plex rotates the
  token mid-pause, the resumed task 401s and you re-queue from the Library tab.
- **Offline artwork is limited to what's been seen or downloaded.** Posters are
  cached as you browse and prefetched on download, so a show you've never opened
  and never downloaded has no art offline.
- **Watched state is local.** Nothing is written back to your Plex server.

## Test

```bash
xcodebuild test \
  -project Longhaul.xcodeproj \
  -scheme Longhaul \
  -destination 'platform=iOS Simulator,name=iPad Pro 13-inch (M5)'
```

Every phase adds tests and records the running total in the context log. The
protocol seams (`DownloadSession`, `MediaPlayer`, the injected URLSession
executor) exist so downloads, playback, and HTTP can all be faked — no test
touches the network, VLC, or a real Plex server.

## Architecture notes

- **SwiftUI throughout**, with UIKit interop only for the VLC drawable
  (`VLCPlayerView`) and the `ASWebAuthenticationSession` bridge.
- **SwiftData** persists `DownloadJob` rows with episode metadata denormalised
  onto them, so the offline library renders with no server in reach. Markers
  ride along as an opaque JSON blob.
- **VLCKit** comes from the [`virtualox/vlckit-spm`](https://github.com/virtualox/vlckit-spm)
  community wrapper, pinned to an exact version — VideoLAN publishes no official
  SPM, and VLCKit's `4.0.0a19` upstream tag isn't valid semver, so range
  resolution doesn't work.
- **Downloads** run through a background `URLSession` with a fixed identifier so
  iOS can hand completed transfers back after a relaunch.

- **Two storage locations, deliberately.** Downloaded video lives in
  Application Support and is excluded from backup: it is not reproducible
  offline. Cached artwork lives in Caches, because the server can always
  regenerate it and iOS is welcome to purge it under disk pressure.
- **File paths are re-derived, never trusted.** iOS relocates an app's data
  container across reinstalls, so an absolute path stored at download time goes
  stale. Filenames are job UUIDs, which makes the real location recomputable.

## Project layout

```
project.yml                     # xcodegen source of truth (xcodeproj is gitignored)
Tools/GenerateAppIcon.swift     # regenerates the 1024px app icon
Longhaul/
  App/                          # @main, dependency container, UIKit app delegate
  DesignSystem/                 # tokens, icon vocabulary, shared empty/error/loading views
  Features/
    Auth/                       # PIN sign-in flow
    Library/                    # browse: shows / seasons / episodes, collections, offline library
    Downloads/                  # download triggers, offline list, filter, storage tools
    Player/                     # VLC player, markers, Up Next, auto-play, sleep timer
    Settings/                   # playback preferences
    Root/                       # top-level switcher and tab bar
  Models/Plex/                  # Codable structs for Plex API responses
  Persistence/                  # SwiftData models, download manager, artwork prefetch
  Services/
    Persistence/                # Keychain wrapper
    Plex/                       # HTTP client, endpoints, auth, discovery, library, artwork cache
  Resources/                    # Asset catalogs, app icon
LonghaulTests/
  Features/                     # view-model, design-system, filter tests
  Persistence/                  # SwiftData, downloads, storage, collections, artwork
  Player/                       # player view model, auto-play, sleep timer, key commands
  Plex/                         # service + model decoding tests
  Fixtures/                     # golden JSON from sanitized Plex responses
  TestSupport/                  # fakes (HTTP, download session, media player)
```

## Distribution

TestFlight and the App Store, built by CI from a `release/*` tag. See
[DEPLOYMENT.md](DEPLOYMENT.md) for the release process and
[CHANGELOG.md](CHANGELOG.md) for what shipped when.

## License

Proprietary. All rights reserved — see [LICENSE](LICENSE). Source is closed;
distribution is via TestFlight and the App Store only.

This is an independent app with no affiliation to or endorsement from Plex Inc.
It talks to a Plex Media Server using the same documented HTTP API any client
uses, with the user's own account token, against a server they control.
