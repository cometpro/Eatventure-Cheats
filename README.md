# ⚡ Eatventure Utility Tool

> A standalone Android cheat utility for **Eatventure** built with [Frida](https://frida.re/) — developed as a proof-of-concept for the game owner.

---

## 📱 Features

| Feature | Description |
|---|---|
| 💎 **Gem Editor** | View, set, or max out your gem balance instantly |
| 📦 **Chest Spawner** | Spawn any of 16 chest types — x1, x10, or x100 at a time |
| 🐾 **Pet Food** | Quickly spawn Pet chests in bulk |
| 🎮 **Floating Overlay** | Non-intrusive 💎 button — tap to open, tap to close |
| 📦 **Fully Standalone** | No ADB, no root, no external files — just install the APK |

---

## 🛠️ How It Works

The utility is built using **Frida Gadget** injected into the game's native library (`libmain.so`). The gadget loads an embedded JavaScript payload at runtime which:

1. Resolves native function addresses from `libil2cpp.so` using RVAs extracted via [Il2CppDumper](https://github.com/Perfare/Il2CppDumper)
2. Hooks `get_Gems()` to capture the live game object instance
3. Hooks `CreateChest()` to capture the live `GameContext`
4. Renders a floating Android `WindowManager` overlay with full UI controls

No game server communication is modified. All changes are client-side only.

---

## 📂 Repository Structure

```
eatventure-utility/
├── repack_eatventure.py   # APK repacker — injects gadget + embeds script
├── cheat.js               # Frida script — native hooks + Android overlay GUI
├── README.md
└── docs/
    └── dump_analysis.md   # Notes on Il2CppDumper findings
```

---

## 🔬 Reverse Engineering Notes

The game uses **Unity IL2CPP** (armeabi-v7a). Key methods identified from `dump.cs`:

```csharp
// RVA: 0x170F3E0  →  double get_Gems()
// RVA: 0x170F458  →  void set_Gems(double value)
// RVA: 0x170F51C  →  void GiveLotsOfGems()        ← leftover dev cheat
// RVA: 0x170F624  →  void ZeroGems()
// RVA: 0x1614AC0  →  GameEntity CreateChest(GameContext, ChestType, bool)
```

Chest types (from `ChestType` enum):

```
Small=0  Big=1  Mine=2  Space=3  SeaPort=4  MiddleAges=5
Pet=6  Club=7  Christmas=8  GreekAdventure=9  PirateAdventure=10
ShinyMine=11  ShinySpace=12  ShinySeaPort=13  ShinyMiddleAges=14  PotionShop=15
```

---

## ⚙️ Building

### Prerequisites

- Python 3.x
- [lief](https://github.com/lief-project/LIEF) — `pip install lief`
- Android SDK build-tools (`zipalign`, `apksigner`)
- Java (for `keytool`)
- [frida-gadget armeabi-v7a](https://github.com/frida/frida/releases) — rename to `libfrida-gadget.so`

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/SnowSilver/eatventure-utility
cd eatventure-utility

# 2. Place the following in C:\frida\:
#    - libfrida-gadget.so  (frida-gadget-17.7.3-android-arm.so)
#    - com.hwqgrhhjfd.idlefastfood.xapk  (original game XAPK)

# 3. Run the repacker
python repack_eatventure.py

# 4. Install on device
#    Copy eatventure_modded.apk to your phone and install
#    Enable "Install unknown apps" in Android settings if prompted
```

---

## 📸 UI Overview

```
┌─────────────────────────────┐
│ ⚡ Eatventure Utility    ✕  │
├─────────────────────────────┤
│ 💎 GEMS                     │
│ Gems: 12500                 │
│ [Refresh]    [Give Lots]    │
│ [Amount input field      ]  │
│ [Set Gems]   [Zero Gems]    │
├─────────────────────────────┤
│ 📦 CHESTS                   │
│ [Chest type dropdown    ▾]  │
│    [×1]    [×10]   [×100]   │
├─────────────────────────────┤
│ 🐾 PET FOOD (via Pet Chest) │
│    [×1]    [×10]    [×50]   │
├─────────────────────────────┤
│      SnowSilver v0.1.0      │
└─────────────────────────────┘
```

The 💎 floating button sits in the top-left corner and toggles the panel.

---

## ⚠️ Disclaimer

This tool was developed as a **proof-of-concept** for the game owner to demonstrate client-side vulnerabilities. It is not intended for use on live accounts or distribution to players. The repacked APK will not pass Play Integrity checks and cannot access Google Play services.

---

## 🔧 Tech Stack

- [Frida](https://frida.re/) 17.7.3
- [Il2CppDumper](https://github.com/Perfare/Il2CppDumper) v6.7.46
- [LIEF](https://lief.re/) — ELF patching
- Android `WindowManager` overlay API
- Python 3 — APK repack pipeline

---

*Built by SnowSilver — proof-of-concept only*
