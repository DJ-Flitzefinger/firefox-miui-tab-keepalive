# Firefox MIUI Tab KeepAlive

**Prevents MIUI from killing Firefox Mobile tabs in the background** by maintaining a silent phantom MediaSession.

> ⚠️ **Early Development**: This project is still in an early development phase. Expect bugs and breaking changes. Feedback and contributions are welcome!

---

## The Problem

Xiaomi's MIUI and similar aggressive Android skins have extremely aggressive memory management that kills background apps and browser tabs. This results in:

- Switching away from Firefox for a few minutes → tabs get killed
- Turning off the screen → Firefox gets terminated
- Watching a YouTube video, switching to another app briefly → video tab is gone
- Loss of form data

Firefox loses all tab state, and MIUI ignores Android's standard app lifecycle.

## The Solution

This userscript tricks MIUI into thinking Firefox is actively playing media by:

1. Creating an **invisible, silent video** using a canvas-based stream
2. Registering a **MediaSession** that appears in Android's notification area
3. Maintaining **activity signals** (Web Locks, IndexedDB heartbeat) to appear busy

MIUI respects apps with active media playback, so Firefox stays alive.

## Features

- **Master Tab System**: Only one YouTube tab runs the keepalive at a time to save battery
- **Visual Indicator**: A subtle circle badge shows on YouTube pages; a lock icon indicates the active master tab
- **One-Tap Toggle**: Click the badge to activate/deactivate
- **Multi-Tab Aware**: Switching the master tab to another YouTube tab is seamless
- **Battery Optimized**: Uses minimal resources (0.5 FPS canvas, efficient timers)
- **Non-Intrusive**: Doesn't interfere with actual video playback

## How It Works

```
┌────────────────────────────────────────────────────────┐
│                    YouTube Tab                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Canvas (2x2 px) → captureStream() → <video>    │   │
│  │         ↓                                       │   │
│  │  MediaSession API → Android Notification        │   │
│  │         ↓                                       │   │
│  │  "Phantom KeepAlive" appears in media controls  │   │
│  └─────────────────────────────────────────────────┘   │
│                         +                              │
│  Web Lock (exclusive) + IndexedDB heartbeat (15s)      │
└────────────────────────────────────────────────────────┘
                          ↓
        MIUI sees active media → doesn't kill Firefox
```

## Installation

### Prerequisites

- **Firefox Mobile** (tested on Firefox for Android)
- **Tampermonkey** extension ([Install from Firefox Add-ons](https://addons.mozilla.org/firefox/addon/tampermonkey/))
- A **Xiaomi/MIUI device** (or other aggressive Android skin like ColorOS, FunTouchOS, etc.)

### Steps

1. Install Tampermonkey in Firefox Mobile
2. Visit the Greasy Fork page and click **Install**:
   
   **[Install Firefox MIUI Tab KeepAlive](https://greasyfork.org/en/scripts/563165-firefox-miui-tab-keepalive)**

3. Tampermonkey will ask for confirmation → click **Install**
4. Open any YouTube page → you should see a small circle in the bottom-right corner

5. (optional): If videos are not played on your phone then they are in backround or screen is switched off additionally install this script (README.md in folder "firefox-mobile-background-video"):

   **[Install Firefox Mobile Background Video](https://greasyfork.org/en/scripts/563406-firefox-mobile-background-video)**

## Usage

### Activating the KeepAlive

1. Open YouTube in Firefox Mobile
2. Start a random Video and then pause it (otherwise the need Player in the notifications will not appear)
3. Look for the **circle badge** (○) in the bottom-right corner
4. **Tap the circle** → a lock icon appears inside
5. Check your Android notifications → "Phantom KeepAlive" should appear

### Deactivating

- **Tap the badge again** (when showing the lock) → keepalive stops
- Or: Simply close the master tab

### Switching Master Tab (not needed to be done, but good to know)

If you have multiple YouTube tabs open:
- Tap the badge on any other YouTube tab → that tab becomes the new master
- The previous master automatically releases

### Visual States

| Badge State | Meaning |
|-------------|---------|
| ○ (empty circle) | KeepAlive available but not active on this tab |
| 🔒 (circle with lock) | This tab is the master, keepalive is running |

## Configuration

Advanced users can modify the `SETTINGS` object at the top of the script:

```javascript
const SETTINGS = {
  mediaSession: {
    refreshIntervalMs: 10000,      // How often to refresh MediaSession (ms)
    skipIfRealMediaOnPage: true,   // Don't refresh while real video plays
  },
  phantomVideo: {
    retryDelayMinMs: 15000,        // Min delay for play() retry
    retryDelayMaxMs: 120000,       // Max delay (exponential backoff)
    canvasStreamFps: 0.5,          // Canvas frame rate (lower = less battery)
  },
  extras: {
    webLock: true,                 // Use Web Locks API
    indexedDBHeartbeat: true,      // Periodic IndexedDB write
  },
  indexedDB: {
    heartbeatIntervalMs: 15000,    // IndexedDB write interval
  },
};
```

### Tuning for Battery vs. Stability

| Setting | Lower Value | Higher Value |
|---------|-------------|--------------|
| `refreshIntervalMs` | More stable, more battery | Less battery, may drop |
| `canvasStreamFps` | Less battery | More "realistic" video |
| `heartbeatIntervalMs` | More activity signals | Less battery |

## Limitations

- **YouTube only**: The script currently only runs on `youtube.com` and `m.youtube.com`. This is intentional—YouTube is the most common use case, and limiting scope reduces battery impact.

- **Requires one YouTube tab**: You need at least one YouTube tab open (it doesn't need to play anything).

- **Not 100% guaranteed**: Extremely aggressive battery saver modes or very low memory situations may still kill Firefox. This script significantly improves survival rate but can't override all system-level kills.

- **Media notification visible**: A "Phantom KeepAlive" entry appears in your Android media controls. This is by design—it's what keeps Firefox alive.

- **Music played in other apps can break the workaround**: Apps that play sound and overwrite the player in the Android notifications will overwrite/stop the KeepAlive session. This could work for a few seconds, but if stopped for to long MIUI will most likely start killing tabs.

## Troubleshooting

### YouTube tabs getting reloaded, lock icons disappears on master Tab
- This is most likely caused by RAM is to full ---> try to close as many apps as possible

### Badge doesn't appear
- Make sure Tampermonkey is enabled
- Check that the script is enabled in Tampermonkey's dashboard
- Refresh the YouTube page

### KeepAlive activates but Firefox still gets killed
- Try lowering `refreshIntervalMs` to `5000`
- Make sure MIUI's battery saver isn't set to "Ultra" mode
- Add Firefox to MIUI's "Battery saver exclusions"

### Interferes with music apps
- The script is designed to not steal audio focus
- If you experience issues, try disabling `skipIfRealMediaOnPage`

### "Phantom KeepAlive" doesn't appear in notifications
- Some MIUI versions hide media notifications by default
- Check notification settings for Firefox

### Optimize XIAOMIs aggressive Memorymanagement
- It is a good additional idea to apply the tweaks mentioned on this site:
https://dontkillmyapp.com/xiaomi

## Tested On

- Xiaomi MI 11 Ultra with HyperOS 1.0.11.0.UKAEUVF
- Firefox Mobile (latest)
- Tampermonkey

*If you've tested on other devices/versions, please open an issue or PR to update this list!*

### Development Notes

- The script uses a **Master Tab** pattern to ensure only one tab runs the keepalive
- State is shared via Tampermonkey's `GM_getValue`/`GM_setValue` storage
- The canvas-based approach was chosen over MP4 data URLs for stability

## License

GNU General Public License v3 (GPL-3.0)

If you distribute a modified version of this script, it must remain GPL-licensed.

## Acknowledgments

- Inspired by the frustration of losing Firefox tabs on MIUI
- Thanks to the Tampermonkey team for making userscripts possible on mobile

---

## ⚠️ Temporary Workaround (v2.x.x Issue)

Version 2.x.x currently has a critical bug. Until a fix is released, please use **version 1.2.1** with the following workaround:

1. Open a random YouTube video
2. Press **Play** and then **Pause** (you should now see the video player in the notifications displaying the original video title)
3. Activate the script by pressing on the circle so it shows the lock icon. If it was already activated, deactivate and reactivate it.
4. The title in the notification player should now show **"Phantom KeepAlive"** instead of the original YouTube title

This workaround ensures the MediaSession is properly initialized. A proper fix is being worked on.
