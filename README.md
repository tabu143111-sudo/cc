📡 LAN Remote
Use one Android device as a remote control for another over your local Wi-Fi: touchpad, mouse, keyboard, media controls, volume and an ad-skip button. No cloud, no account. Everything stays on your network and is encrypted with a pairing code.
Built with Kotlin + Jetpack Compose. Min SDK 28 (Android 9), target SDK 34.
✨ Features
🖱️ Touchpad
Laptop-style multi-touch pad with smooth, eased cursor movement
⌨️ Keyboard
Type on the controller, text appears on the host (with backspace / enter)
🎵 Media control
Previous, −10 s, play/pause, +10 s, next, seek bar, volume bar, now-playing title and cover art
⏭️ Skip ad
A "Skip ad" button appears on the controller while an ad runs on the host (YouTube and others)
🖼️ Full-screen media view
Edge-to-edge blurred cover art with floating controls
📱 Responsive layout
Portrait, landscape, tablets and foldables; always immersive (no status/navigation bars)
🔍 Auto discovery
Hosts are found on the network automatically (mDNS), or scan a QR code, or type the IP
🔒 Encrypted
All traffic is encrypted with a key derived from your pairing code
🌗 Light / dark theme
Toggle on the home screen
🧩 How it works
 Controller device                              Host device (the one being controlled)
 ┌──────────────────────┐   encrypted TCP :8765  ┌─────────────────────────────────────┐
 │ Touchpad, buttons,   │ ─────────────────────► │ HostService                         │
 │ media card, keyboard │ ◄───────────────────── │  ├─ MediaSession  → play/seek/volume│
 └──────────────────────┘    now-playing state   │  ├─ RemoteAccess  → cursor, gestures│
                                                 │  │   (Accessibility service)        │
                                                 │  └─ InputServer   → real mouse/kbd  │
                                                 │      (optional, started over ADB)   │
                                                 └─────────────────────────────────────┘
