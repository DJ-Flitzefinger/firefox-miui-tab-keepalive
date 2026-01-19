# Changelog

All notable changes to **Firefox MIUI Tab KeepAlive** will be documented in this file.

The format is based on **Keep a Changelog** and this project adheres to **Semantic Versioning**.

## [Unreleased]
### Added
### Changed
### Fixed

## [1.2.0] - 2026-01-19
### Added
- Web Locks API support to signal the browser that the tab holds an important resource.
- IndexedDB heartbeat mechanism with periodic writes to keep database connection active.

### Changed
- `MEDIASESSION_REFRESH_MS` reduced from 30000 to 10000 for more aggressive MediaSession persistence.
- `OWNER_HEARTBEAT_MS` reduced from 8000 to 3000 for faster owner synchronization.
- `OWNER_TTL_MS` reduced from 25000 to 12000 to match new heartbeat timing.

## [1.1.0] - 2026-01-19
### Added
- Compare-and-swap verification for owner claiming (`OWNER_CLAIM_VERIFY_DELAY_MS`) to reduce cross-tab race conditions.
- Configurable canvas fallback framerate (`CANVAS_STREAM_FPS`) for lower CPU usage.
- Visibility hook uninstall logic to restore original Page Visibility API and remove event blockers when disabling the script.
- Synthetic activity emitter using `MouseEvent`/`PointerEvent` with plausible coordinates for improved compatibility on some sites.

### Changed
- MediaSession refresh interval reduced for more reliable "Now Playing" persistence on Android/MIUI.

### Fixed
- Prevented leftover visibility hooks / event listeners from persisting after disabling KeepAlive.
- Improved stability when multiple tabs attempt to claim background-play ownership.

## [1.0.0] - 2026-01-18
### Added
- Initial release.
- Global phantom MediaSession keepalive to prevent MIUI from discarding/reloading Firefox Mobile tabs.
- Background playback support on whitelisted sites via Page Visibility API spoofing.
- Owner-based architecture to ensure only one tab performs background-play related activity.
- Floating lock UI to enable/disable KeepAlive globally.
- Exponential backoff watchdog to keep phantom playback running.
- Cross-tab synchronization via GM storage keys.
