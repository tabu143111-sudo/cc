# 📡 LAN Remote

Use one Android device as a **remote control for another** over your local Wi-Fi: touchpad, mouse, keyboard, media controls, volume and an ad-skip button. No cloud, no account. Everything stays on your network and is encrypted with a pairing code.

> Built with Kotlin + Jetpack Compose. Min SDK 28 (Android 9), target SDK 34.

---

## ✨ Features

| | |
|---|---|
| 🖱️ **Touchpad** | Laptop-style multi-touch pad with smooth, eased cursor movement |
| ⌨️ **Keyboard** | Type on the controller, text appears on the host (with backspace / enter) |
| 🎵 **Media control** | Previous, −10 s, play/pause, +10 s, next, seek bar, volume bar, now-playing title and cover art |
| ⏭️ **Skip ad** | A "Skip ad" button appears on the controller while an ad runs on the host (YouTube and others) |
| 🖼️ **Full-screen media view** | Edge-to-edge blurred cover art with floating controls |
| 📱 **Responsive layout** | Portrait, landscape, tablets and foldables; always immersive (no status/navigation bars) |
| 🔍 **Auto discovery** | Hosts are found on the network automatically (mDNS), or scan a QR code, or type the IP |
| 🔒 **Encrypted** | All traffic is encrypted with a key derived from your pairing code |
| 🌗 **Light / dark theme** | Toggle on the home screen |

---

## 🧩 How it works

```
 Controller device                              Host device (the one being controlled)
 ┌──────────────────────┐   encrypted TCP :8765  ┌─────────────────────────────────────┐
 │ Touchpad, buttons,   │ ─────────────────────► │ HostService                         │
 │ media card, keyboard │ ◄───────────────────── │  ├─ MediaSession  → play/seek/volume│
 └──────────────────────┘    now-playing state   │  ├─ RemoteAccess  → cursor, gestures│
                                                 │  │   (Accessibility service)        │
                                                 │  └─ InputServer   → real mouse/kbd  │
                                                 │      (optional, started over ADB)   │
                                                 └─────────────────────────────────────┘
```

- **Host** runs a foreground service that accepts one encrypted connection from the controller.
- **Media + volume** work through the Android media session (needs *notification access*).
- **Cursor, taps, scrolling, typing, ad skipping** work through an *Accessibility service*.
- **Native mode (optional):** an ADB-started `InputServer` injects genuine mouse and keyboard events, so the host behaves as if a physical mouse and keyboard were attached.

---

## 🚀 Getting started

### 1. Build

1. Open this folder in **Android Studio** (Giraffe or newer, JDK 17).
2. Let Gradle sync, then **Run** or **Build ▸ Build APK(s)**.
3. Install the APK on **both** devices.

### 2. Set up the host

1. Open the app ▸ **Host**.
2. Choose a pairing code (6–16 letters or digits) and tap **Start host**.
3. Enable **Accessibility** (mouse, keyboard, gestures, skip ad) and **Notification access** (now-playing info, seek, cover art) when asked.

> The app has a built-in **Setup guide** (home screen) that walks through all of this, with copy buttons for every command.

### 3. Connect the controller

1. Open the app ▸ **Controller**.
2. Pick the host from **Nearby hosts**, or scan the host's QR code, or enter its IP.
3. Enter the pairing code and connect. Both devices must be on the same network.

---

## 🔑 Permanent permissions & native input (optional, via ADB)

Without this you must re-enable the services manually after a reboot.

**Windows:** `adb_setup.bat`  **Mac / Linux:** `./adb_setup.sh`

Run it on the host (USB debugging on, connected to a PC). It:

- grants `WRITE_SECURE_SETTINGS` so the app keeps Accessibility and notification access switched on by itself, even after a reboot
- exempts the app from battery optimisation
- starts the native `InputServer` (real mouse + keyboard injection)

Manual alternative for permissions only:

```bash
adb shell pm grant com.example.lanremote android.permission.WRITE_SECURE_SETTINGS
```

**No PC?** Use Wireless debugging + Termux (see the in-app Setup guide). The Host screen also has **Start native server now** and **Start automatically**.

If the native server is not running, the app falls back to the Accessibility-based pointer and keyboard.

> 🔄 After updating the app, re-run `adb_setup` (or tap *Start native server now*). The native server keeps running old code until restarted.

---

## 👆 Touchpad gestures

| Fingers | Gesture | Action |
|---|---|---|
| 1 | Move | Move cursor |
| 1 | Tap / double-tap | Click / double click |
| 1 | Tap, then touch and drag | Drag / select |
| 1 | Hold | Right click |
| 2 | Drag | Scroll (vertical + horizontal, with momentum) |
| 2 | Pinch | Zoom |
| 2 | Tap | Right click |
| 3 | Swipe up / down | Recents / Home |
| 3 | Swipe left / right | Back / App drawer |
| 4 | Swipe down / up | Notifications / Quick settings |
| 4 | Swipe left / right | Back / Recents |

Tap **Natural / Classic** under the pad to flip scroll direction. Use the sensitivity slider to tune pointer speed. Tap **?** for this list in the app.

> Some phones (Xiaomi/HyperOS, OPPO, Samsung) have their own 3-finger system gestures. Turn them off in system settings if they cancel your touches.

---

## ⏭️ Skip ads

While an ad plays on the host, a pill appears in the media card:

- **Ad playing…** (greyed out): an ad is running but cannot be skipped yet
- **Skip ad**: the host's skip button is visible; tapping it presses that button

Detection uses the Accessibility service, so it depends on the labels the app exposes. It targets YouTube first. If it is not detected, switch the Accessibility service off and on once.

---

## 📁 Project structure

```
LanRemote/
├── app/src/main/java/com/example/lanremote/
│   ├── MainActivity.kt    UI: home, host, connect, remote, media card, full-screen view
│   ├── Touchpad.kt        Multi-touch gesture engine
│   ├── HostService.kt     Host server, media session state, volume, discovery
│   ├── RemoteAccess.kt    Accessibility service: cursor overlay, gestures, typing, skip ad
│   ├── InputServer.kt     ADB-privileged native mouse/keyboard injection
│   ├── NativeInput.kt     Bridge from the app to InputServer
│   ├── NativeStarter.kt   Starts InputServer through Termux
│   ├── MediaListener.kt   Notification listener (cover art, media access)
│   ├── AutoGrant.kt       Keeps Accessibility / notification access enabled
│   ├── Crypto.kt          Key derivation and message encryption
│   └── Guide.kt           In-app setup guide
├── adb_setup.bat / adb_setup.sh / lanremote_start.sh
└── README.md
```

---

## 🔐 Security notes

- Traffic is encrypted with a key derived from the pairing code and a per-connection salt. Use a long code on shared networks.
- The QR code contains the pairing code. Only show it to people you trust.
- The native input server listens on `127.0.0.1` only and requires a random token readable only by this app.
- Accessibility and `WRITE_SECURE_SETTINGS` are powerful permissions. Only install builds you trust.

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| Host not listed | Same Wi-Fi? Host started? Try the IP manually (shown on the Host screen) |
| "Couldn't connect" | Check the pairing code and that the host is running |
| No title / cover art / seek | Enable notification access on the host |
| Cursor or gestures do nothing | Enable the Accessibility service on the host |
| Services turn off after reboot | Run `adb_setup` once to grant permanent access |
| Cursor not smooth / not native | Re-run `adb_setup` or *Start native server now* after updating |
| Skip ad button never appears | Toggle the Accessibility service off and on; make sure the ad is in the foreground app |

---

## 📄 License

Add your license here (for example MIT) before publishing.
