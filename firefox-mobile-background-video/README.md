# Firefox Mobile Background Video

A lightweight Userscript inspired by Mozilla’s `video-bg-play` add-on, designed to keep **video/audio playing in the background** on **mobile Firefox** (and other browsers using a Userscript manager).

It works by spoofing the **Page Visibility API** so that websites continue to believe the tab is still visible.

---

## ✨ Features

- ✅ Keeps videos/audio playing when the tab is in the background (supported sites only)
- ✅ Works with **Firefox Mobile**
- ✅ Runs **only on explicitly whitelisted domains**
- ✅ Optional support for **iframes/embedded players**
- ✅ Optional “synthetic activity” (off by default)
- ✅ No hard-blocking of `visibilitychange` events (more compatible)

---

## 🔧 How it works (short)

Many video platforms pause playback when the tab is not visible (background / switched tab / screen locked).
This script spoofs these values:

- `document.visibilityState` → `"visible"`
- `document.hidden` → `false`

(Some sites also use WebKit aliases such as `document.webkitHidden`, which are also spoofed.)

---

## ✅ Supported websites

The script only activates on domains listed in **two places**:

1) **Userscript header match rules** (`// @match ...`)
2) **Runtime whitelist** (`CFG.HOST_SUFFIXES`)

To add a new website, you **must add it to both lists**.

Examples included by default:

- YouTube
- Netflix
- Twitch
- Spotify
- Vimeo
- TikTok
- SoundCloud
- Prime Video
- and more…

---

## 📦 Installation

### 1) Install a Userscript Manager
You need a Userscript manager extension/add-on first.

Recommended:
- Tampermonkey (depending on platform availability)
- **Violentmonkey** (works well on Firefox Mobile)

### 2) Install the script
Install it from Greasy Fork:

➡️ **Greasy Fork link:**

   **[Install Firefox Mobile Background Video](https://greasyfork.org/en/scripts/563406-firefox-mobile-background-video)**


### 3) Done ✅
Open one of the supported sites and start playing a video/audio stream.
Playback should continue even if you switch apps/tabs.

---

## ⚙️ Configuration

Open the script in your Userscript manager and edit the `CFG` section at the top.

### Core options

```js
const CFG = {
  ENABLED: true,

  // If true, also run inside iframes/embeds
  ALLOW_IFRAMES: true,

  // Domain whitelist (suffix match)
  HOST_SUFFIXES: [
    'youtube.com',
    'netflix.com',
    'spotify.com',
  ],

  // Spoof Page Visibility API
  SPOOF_VISIBILITY: true,

  // Optional: emit user activity events (default OFF)
  SYNTHETIC_ACTIVITY: false,
};
