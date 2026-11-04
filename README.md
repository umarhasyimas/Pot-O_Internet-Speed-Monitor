# Pot-O Internet Speed Monitor

A tiny, native Windows utility that shows your **live download and upload speed** right on the taskbar,
and - new in 2.1 - **which application is using your connection**.

It measures real network traffic. It does **not** run speed tests, contact any server, or send any data anywhere.

- Native C++17 / Win32 - no .NET, Qt, Electron or any other runtime
- Single small executable, no installer
- Freeware

> Created by **Muhammad Umar Hasyim Ashari** - (c) 2026

---

## Features

| | |
|---|---|
| **Taskbar speed widget** | Large, readable download / upload speed that sits next to the system tray. Drag it anywhere, lock it, choose one-line or two-line layout and four sizes. DPI aware. |
| **Tray icon** | Optional live speed in the tray icon, tooltip with current speed and today's usage. Left-click toggles the widget. |
| **Per-application usage** | Download / upload speed and session totals for every program, grouped by executable. Optionally hides Windows system processes so you only see user-installed applications. |
| **Themes** | Follow system, Light or Dark. Follows Windows live when set to *Follow system*. |
| **Units** | Bytes (KB/s, MB/s) or bits (Kbps, Mbps). |
| **Network adapter** | Measure all active adapters, or pick one (Ethernet, Wi-Fi, VPN ...). |
| **Update interval** | 0.5 s, 1 s or 2 s. |
| **Usage statistics** | Live window with current speed, session peaks, session totals and today's totals (kept across restarts). |
| **Start with Windows** | Per-user start-up entry, or an administrator scheduled task (needed for per-application monitoring at sign-in). |
| **Lightweight** | No extra threads for the basic monitor, one timer, counters read from the Windows IP Helper API. |
| **Robust** | Recovers when Explorer restarts, handles adapters appearing/disappearing without false speed spikes, single instance. |

## Requirements

- Windows 10 or Windows 11 (x64)
- To **build**: Visual Studio 2019 / 2022 / 2026 with the *Desktop development with C++* workload
- To use **per-application usage**: administrator rights (see below)

## Build

1. Clone or download this repository.
2. Open a *Developer Command Prompt for VS* (or just double-click `build.bat`).
3. Run:

   ```bat
   build.bat
   ```

The executable is created in `build\` and is named after the version defined in `src\version.h`:

```
build\Pot-O_InternetSpeedMonitor_2.1.0.exe
```

The icon, version information and manifest are embedded in the executable. The C++ runtime is linked statically,
so the exe runs on a clean Windows install without any redistributable.

To change the version, edit **only** `src\version.h`; the exe name, the file properties and the About box follow it.

## Usage

Run the exe. A speed widget appears next to the system tray and an icon appears in the notification area.

- **Drag** the widget to move it (position is remembered). **Right-click** the widget or the tray icon for all options.
- **Left-click** the tray icon to show or hide the widget.

### Right-click menu

| Item | Description |
|---|---|
| Show speed widget | Show / hide the taskbar widget |
| Widget | Lock position, reset position, hide in full-screen apps, layout (one / two lines), size |
| Theme | Follow system / Light / Dark |
| Units | Bytes or bits |
| Update interval | 0.5 / 1 / 2 seconds |
| Network adapter | All active adapters, or a specific one |
| Live speed in tray icon | Draw the speed inside the tray icon |
| Start with Windows | Start the app when you sign in |
| Start with Windows as administrator | Start with admin rights at sign-in via Task Scheduler (no UAC prompt) |
| Per-application usage... | Open the per-application window |
| Usage statistics... | Live statistics window |
| About... | Version and author |
| Exit | Close the application |

## Per-application usage

Open it from the menu: **Per-application usage...**

For each program you see current download / upload speed, the amount downloaded / uploaded since the app started,
and the executable name. Click a column header to sort. By default only user-installed applications are listed;
untick *Show user-installed applications only* to include Windows processes such as `svchost.exe`.

### How it works

Windows does not offer a normal API for per-process network usage. The app starts an **ETW (Event Tracing for
Windows)** real-time session on the `Microsoft-Windows-Kernel-Network` provider, which reports every TCP/UDP send and
receive together with the process ID. Processes are grouped by executable path and named from their file description.
The trace session exists only while the app runs.

### Administrator rights

Windows only allows starting this kind of trace session as administrator. Without it, the app works normally
(widget, tray, statistics) and the per-application window explains that rights are missing.

- Click **Restart as administrator** in the per-application window (a UAC prompt appears), or
- run the exe as administrator, or
- enable **Start with Windows as administrator** (create it once from an elevated instance) so that the app starts
  elevated at every sign-in without a UAC prompt.

### Notes and limitations

- Per-application numbers count TCP/UDP **payload** bytes. They will be slightly lower than the adapter totals shown
  in the widget, which also include protocol overhead and traffic not attributed to a process (for example ICMP).
- Loopback (local `127.0.0.1` / `::1`) traffic is not counted.
- Per-application totals are kept for the current session (they restart when the app restarts).
- A process that exits before it was first seen by the monitor may appear as *Unknown process (PID n)*.
- "User-installed" means "not located in the Windows folder". Programs bundled with Windows that live elsewhere
  (for example Microsoft Edge under *Program Files*) are listed as applications.

## Settings

Settings are stored in `Pot-O_InternetSpeedMonitor.ini` **next to the executable** (the file name does not change
between versions, so updating the exe keeps your settings). Keep the exe in a folder you can write to.
Today's usage is stored there as well. Delete the file to reset everything.

The start-up entry is stored per user in
`HKCU\Software\Microsoft\Windows\CurrentVersion\Run` (value `Pot-O_InternetSpeedMonitor`).
The optional administrator start-up is a scheduled task named `Pot-O_InternetSpeedMonitor`.

To remove the application completely: untick the start-up options in the menu, exit the app and delete the exe
and the `.ini` file.

## Troubleshooting

**The widget is not where I expect it.** Drag it to the place you like, or use *Widget > Reset position*.
The default position is just left of the notification area of the primary taskbar.

**The widget disappears while a blue-light filter / dimmer is running.** Fixed in 2.1.0. If it still hides
for another overlay or full-screen app, turn off *Widget > Hide in full-screen apps*.

**The per-application list is empty.** Check the status line in that window: monitoring needs administrator
rights, and only applications that actually used the network since the app started appear.

**Speeds look different from my browser / Windows Task Manager.** The widget shows traffic of the selected adapters
at the interface level (including protocol overhead). Task Manager and other tools may measure at other layers or
use different time windows.

**Antivirus / SmartScreen warns about the exe.** The exe is unsigned. If you prefer, review the source and build it yourself.

## Technical overview

| Area | Technology |
|---|---|
| Language | C++17 |
| UI | Win32 API + GDI (custom painted widget, tray icon and statistics window), list view for the application table |
| Interface counters | IP Helper API (`GetIfTable2`), per-interface deltas keyed by interface LUID |
| Per-application traffic | ETW `Microsoft-Windows-Kernel-Network` real-time session |
| Tray | `Shell_NotifyIcon` (version 4) |
| Settings | INI file, registry `Run` key, Task Scheduler (optional) |
| Build | MSVC `cl.exe` + `rc.exe`, static CRT |

## Privacy

The application makes **no network connections** and collects no data. Everything is measured locally and stays on
your computer.

## Author

**Muhammad Umar Hasyim Ashari** - (c) 2026

## License

Pot-O Internet Speed Monitor is **freeware**: free to use and share.
