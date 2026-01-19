# Firefox MIUI Tab KeepAlive

> **Status:** This project is still in an early development stage.
> **Current workaround:** At the moment, the KeepAlive lock can only be activated reliably using the following trick: start a YouTube video briefly and pause it again **before** enabling the lock.


A userscript that prevents Firefox Mobile tabs from being aggressively discarded and reloaded by MIUI’s background memory management.

This script is primarily intended for **Xiaomi / MIUI devices**, where Firefox tabs are frequently reloaded after short periods of inactivity, causing:
- loss of form data
- restarted videos
- broken multi-step workflows
- destroyed long-lived research sessions

The script works entirely inside Firefox using standard web APIs.  
No APKs, no root, no system modifications.

---

## What problem does this solve?

MIUI applies extremely aggressive background process and memory management.  
Firefox Mobile is **not whitelisted** by MIUI and is therefore treated as expendable.

As a result:
- background tabs are discarded
- paused videos restart
- filled forms are lost
- pages reload after switching apps or turning the screen off

This userscript prevents that by making Firefox appear **continuously active** to the Android system.

---

## How it works (high-level)

### 1. Phantom MediaSession KeepAlive (global)
The script creates a **silent, invisible phantom media session**.

From Android’s point of view:
- Firefox is actively playing media
- the app must not be suspended or killed

From the user’s point of view:
- no sound
- no visible video
- no UI interference

This mechanism applies **globally** to all tabs, including purely text-based pages and forms.

---

### 2. Background playback support (whitelisted domains only)
Some video platforms pause playback when a tab is backgrounded.

For selected domains (e.g. YouTube, Vimeo), the script:
- fakes the Page Visibility API
- allows background playback
- applies this only to **one tab at a time**

Only the **last actively playing tab** becomes the background-play “owner”.  
All other tabs remain completely idle.

This keeps CPU and battery usage low even with many open tabs.

---

### 3. Owner-based architecture (efficient by design)
- Only one tab is allowed to:
  - fake visibility
  - emit minimal synthetic user activity
- Ownership automatically switches to the last tab that actually started playback
- If no video exists (e.g. form pages), ownership is assigned when KeepAlive is enabled

This ensures:
- minimal wakeups
- predictable behavior
- no unnecessary background work

---

## User Interface

- A small **transparent lock icon** appears in the bottom-right corner
- Tap once to enable or disable KeepAlive globally
- The icon is injected **only once** (iframe-safe)

---

## Installation

### Option 1: Greasy Fork (recommended)

1. Install a userscript manager for Firefox (Android):
   - [Tampermonkey (Firefox Add-ons)](https://addons.mozilla.org/firefox/addon/tampermonkey/)
   - [Violentmonkey (Firefox Add-ons)](https://addons.mozilla.org/firefox/addon/violentmonkey/)

2. Open the script on Greasy Fork:
   - [Firefox MIUI Tab KeepAlive](https://greasyfork.org/en/scripts/563165-firefox-miui-tab-keepalive)

3. Click **Install**.

4. Reload Firefox tabs if needed.

This method provides automatic updates.

---

### Option 2: Direct install from GitHub
1. Install a userscript manager (Tampermonkey / Violentmonkey)
2. Open the raw script file:
   - `firefox-miui-tab-keepalive.user.js`
3. Confirm installation in your userscript manager

---

## Configuration (optional)

The script works out of the box.  
Advanced users may customize behavior by editing the configuration section at the top of the script.

### 1. Background-play domain whitelist

```js
const BGPLAY_HOST_SUFFIXES = [
  'youtube.com',
  'youtube-nocookie.com',
  'vimeo.com',
  // 'your-domain.tld',
];
````

* Suffix matching is used
  (`youtube.com` also matches `m.youtube.com`, `www.youtube.com`, etc.)
* Add domains where background playback should be allowed
* The global KeepAlive mechanism is **not affected** by this list

---

### 2. Timing parameters (advanced)

```js
const USER_ACTIVITY_TICK_MS = 60000;
const OWNER_HEARTBEAT_MS = 8000;
const OWNER_TTL_MS = 25000;
const MEDIASESSION_REFRESH_MS = 60000;
```

**What they affect:**

* Higher values → fewer wakeups, better battery life
* Lower values → more aggressive keepalive behavior

The defaults are intentionally conservative and battery-friendly.

---

## What this script does NOT do

* No audible audio playback
* No visible video
* No hijacking of real media focus
* No system-level modifications
* No tracking or external network calls

---

## Intended use cases

* Long form filling without data loss
* Prevent paused videos from restarting
* Keep research tabs alive for days or weeks
* Switch apps without Firefox tabs being destroyed

---

## License

GNU General Public License v3 (GPL-3.0)

If you distribute a modified version of this script, it must remain GPL-licensed.

---

## Source & Development

* Source code and issue tracking: **GitHub repository**
* Distribution and updates: **Greasy Fork**
* https://github.com/DJ-Flitzefinger/firefox-miui-tab-keepalive/releases/latest

Contributions, bug reports, and platform-specific findings are welcome.