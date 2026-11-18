# 🖱️ Mouse-Odometer

**A car-style odometer for your mouse.** Mouse-Odometer runs quietly in the Windows system tray and adds up how far your mouse actually travels, then shows the totals in meters and kilometres for the **last hour**, **today**, the **last 30 days** and your **lifetime**.

```
╭──────────────────────────────────────────╮
│ MOUSE-ODOMETER                 📌  ⚙  ✕ │
│ LIFETIME                                 │
│ [0][0][0][1][2].[3][4][5]  km            │
│ on screen 9.871 km                       │
│                                          │
│ LAST HOUR     TODAY        30 DAYS       │
│ 42.1 m        312.4 m      7.912 km      │
│                                          │
│ ▂▃▅▂▇▆▃▁▂▅▆▇▃▂▅▆▄▃▂▅▇▆▃▂▄▅▆▇▅█         │
│ 30 days ago                       today  │
│ Desk distance · 800 DPI (calibrated)     │
╰──────────────────────────────────────────╯
```

---

## What is it for?

Most of us move a mouse for hours every day without any idea how far that adds up to. Mouse-Odometer answers that question the way a car does: with a running odometer.

- **Curiosity and fun:** find out whether your mouse has run a marathon this year (milestone notifications will tell you).
- **Ergonomics and RSI awareness:** see how much arm and wrist movement a working day really involves, and compare days.
- **Gamers:** compare sessions and see how far low-sensitivity flicking takes you across the mousepad.
- **Desk setup:** check whether a bigger mousepad or a different DPI changes how much you move.

## How it measures

Mouse-Odometer measures **desk distance**, meaning the physical distance the mouse travels on your desk or mousepad.

It reads the mouse sensor's own movement counts through Windows Raw Input and divides them by your mouse's DPI. Like a car's wheel counter, it measures the device itself, not the map. That means it is **not** affected by:

- Windows pointer speed or "Enhance pointer precision"
- a cursor pinned against a screen edge
- games that lock or hide the cursor

As a smaller secondary figure, it also records **screen distance**: how far the cursor travelled across your monitors, converted to real meters using each monitor's physical size (read from its EDID, with a manual override in Settings).

> **Getting accurate meters:** Windows never reports a mouse's DPI. Enter it from your mouse's spec sheet or software, or use the built-in **ruler calibration**: slide the mouse 10 cm along a ruler and Mouse-Odometer works out the DPI for you.

## Features

- 🛞 **Rolling odometer** for lifetime distance, with dark kilometre wheels and amber metre wheels
- ⏱️ **Last hour / today / last 30 days / lifetime**, updated live every second
- 📊 **30-day sparkline** with per-day hover tooltips
- 🖥️ **Screen distance** per monitor, with EDID physical-size detection and manual overrides
- 📏 **DPI calibration wizard** (10 cm, 20 cm, 4 in or 8 in), operated from the keyboard
- 💬 **Live tray tooltip**, e.g. `Today: 312.4 m · Lifetime: 12.345 km`
- 🏁 **Milestone notifications**: 100 m, 1 km, 5 km, 10 km, half marathon, marathon, 100 km, 500 km, 1,000 km, then every 1,000 km
- 📤 **CSV export** (one row per day) and **reset** with confirmation
- 🚀 **Start with Windows** toggle (per user, no admin rights)
- 📦 **Portable single-file exe**: no installer, no .NET install required

## Privacy

Everything stays on your PC. Mouse-Odometer records **only distance totals per minute**. It does not record cursor positions, clicks, keystrokes, window titles or anything about what you are doing, and it makes no network connections. Your data lives in `%LOCALAPPDATA%\Mouse-Odometer\`:

| File | Contents |
|---|---|
| `odometer.db` | SQLite: per-minute totals for 35 days, daily totals after that, lifetime sums |
| `settings.json` | DPI, screen-distance and notification preferences, monitor overrides |
| `error.log` | Errors only, capped at 1 MB |

---

## This repository: TINS (There Is No Source)

This project is distributed as a **[TINS](https://thereisnosource.com) specification**. The file [`Mouse-Odometer-TINS.md`](Mouse-Odometer-TINS.md) is a complete, implementation-ready plan. It contains:

- the full functional spec: UI layouts, user flows, edge cases and validation messages
- an assessment of every Windows platform surface involved (Raw Input, DisplayConfig/EDID, NotifyIcon, HKCU Run key, SQLite)
- architecture, data models and the key algorithms, with worked examples
- **31 build steps containing every source file in full**
- an automated logic-test harness, 22 manual test scenarios and an acceptance checklist

You can build it yourself from the plan below, or hand the spec to an AI coding assistant to generate it.

### Tech stack

| | |
|---|---|
| Language / runtime | C# on **.NET 10 (LTS)**, `net10.0-windows` |
| UI | **WPF** (dashboard, settings, calibration) plus a WinForms `NotifyIcon` for the tray |
| Input | Win32 Raw Input (`WM_INPUT`, `RIDEV_INPUTSINK`) on a dedicated thread |
| Storage | `Microsoft.Data.Sqlite` 10.0.0 (WAL mode) |
| Distribution | Self-contained single-file exe for `win-x64` and `win-arm64` |

### Requirements

- Windows 10 (1809 or later) or Windows 11, x64 or ARM64
- [.NET 10 SDK](https://dotnet.microsoft.com/download) (building only)

### Build from the TINS plan

1. Follow **Steps 0–29** in `Mouse-Odometer-TINS.md` to create the solution and its files.
2. Build, test and run:

   ```powershell
   dotnet build MouseOdometer\MouseOdometer.csproj -c Debug
   dotnet run --project LogicTests        # prints ALL PASSED
   dotnet run --project MouseOdometer     # tray icon appears, dashboard opens
   ```

3. Publish the portable exe:

   ```powershell
   dotnet publish MouseOdometer\MouseOdometer.csproj -c Release -r win-x64   -o publish\win-x64
   dotnet publish MouseOdometer\MouseOdometer.csproj -c Release -r win-arm64 -o publish\win-arm64
   ```

> **Status:** the pure logic and Win32 interop code in the plan has been compiled with the .NET 10 SDK and passes all 59 automated checks. The WPF/XAML files have been reviewed but not yet compiled on Windows, so the first Windows build may need small fixes.

## Using it

1. Run `Mouse-Odometer.exe`. A gauge icon appears in the tray and the dashboard opens.
2. Open **Settings** (⚙) and enter your mouse DPI, or click **Calibrate…**.
3. Left-click the tray icon at any time to show or hide the dashboard. 📌 keeps it open.
4. Right-click the tray icon for **Settings**, **Start with Windows**, **Export CSV** and **Exit**.

Running the exe a second time doesn't start a second tracker; it just opens the existing dashboard.

## FAQ

**Why does my distance look wrong?**
Check the DPI first. The default 800 DPI is only a guess. If your mouse has DPI buttons, calibrate on the stage you use most.

**Why is screen distance different from desk distance?**
Pointer speed and acceleration change how far the cursor moves for the same hand movement. Desk distance ignores them; screen distance includes them.

**Does it work with pen tablets, touchscreens or Remote Desktop?**
These send absolute positions rather than sensor counts, so they don't add desk distance. Cursor movement from them still counts as screen distance.

**I changed my DPI. Is my history recalculated?**
No. History is stored in meters as it was measured at the time. The new DPI applies from then on.

---

## 📚 Citation

### Academic Citation

If you use this codebase in your research or project, please cite:

```bibtex
@software{mouse_odometer_dotnet,
  title = {Mouse-Odometer: a system-tray odometer for Windows that measures how far your mouse travels},
  author = {[Drift Johnson]},
  year = {2026},
  url = {https://github.com/MushroomFleet/mouse-odometer-dotnet},
  version = {1.0.0}
}
```

### Donate:

[![Ko-Fi](https://cdn.ko-fi.com/cdn/kofi3.png?v=3)](https://ko-fi.com/driftjohnson)
