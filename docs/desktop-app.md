# The Humanik OS app

Humanik OS is the desktop app for macOS and Windows. It runs next to humanikd and
gives you one window for:

- **Chats** with your AI employees in a workspace, split into the ones that run on
  this machine and the ones that run in the cloud.
- **Machine**: whether humanikd is running and connected, its settings, its log,
  and updates.
- **Profile**: your account, the app's notifications, and opening at login.

You sign in with your Humanik account. Anything the app does not have (group
chats, the console, billing, members) opens in your browser. The app always asks
first: the first click shows an "Open in browser" button, and only that button
opens the browser.

The app and humanikd are separate programs. Quitting the app does not stop
humanikd, and humanikd does not need the app.

## Install

The install command installs the app with humanikd:

**macOS**

```bash
curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh
```

**Windows** (PowerShell)

```powershell
irm https://github.com/humanikio/humanikd-dist/releases/latest/download/install.ps1 | iex
```

| | macOS | Windows |
| --- | --- | --- |
| Where it goes | `/Applications/Humanik OS.app`. If you cannot write to `/Applications`, it goes to `Applications` in your home folder | `%LOCALAPPDATA%\Programs\Humanik OS\Humanik OS.exe`, with a Start menu shortcut |
| Admin password | Not needed | Not needed |
| After installing | The app opens (not when you install over SSH) | The app opens |
| Needs | macOS 12 or later, Intel or Apple silicon | Windows 10 or 11 (64-bit) and Microsoft Edge WebView2, which Windows 11 includes. The installer tells you if it is missing |

Linux gets humanikd only. There is no Linux app.

**To install humanikd without the app:**

```bash
curl -fsSL https://github.com/humanikio/humanikd-dist/releases/latest/download/install.sh | sh -s -- --no-app
```

```powershell
$env:HUMANIKD_NO_APP = '1'; irm https://github.com/humanikio/humanikd-dist/releases/latest/download/install.ps1 | iex
```

### Why there is no download button

The app is not yet signed by Apple or Microsoft. A file you download in a browser
is marked as coming from the internet, and macOS and Windows block or warn about
unsigned apps with that mark. On macOS the message says the app "cannot be opened"
or "is damaged", which is not true but is all it says.

The install command downloads without that mark, so the app opens normally. It
also checks every file against the checksum published with the release and
refuses to install one that does not match. See
[Verify a download](../README.md#verify-a-download-recommended) to check by hand.

A download button will come once the app is signed.

## Open at login

The app opens when you log in, hidden in the menu bar (macOS) or the system tray
(Windows), with no window. This keeps notifications coming after a restart. It is
on from the first time the app starts.

Turn it off in **Profile > Notifications > Open at login**. It is also listed in:

- **macOS:** System Settings > General > Login Items, under "Allow in the
  Background".
- **Windows:** Settings > Apps > Startup.

If you move the app to another folder, the login entry follows it the next time
the app starts.

## Closing and quitting

Closing the window hides it. The app keeps running in the menu bar or tray, and
keeps showing notifications. To stop it, choose **Quit** from the menu bar or tray
icon. Click the icon, or open the app again, to bring the window back.

## Updating

Run the install command again. It replaces the app with the newest release and
opens it. Your sign-in, workspace choice and settings are kept.

The app's **Machine > Updates** tab updates humanikd, not the app.

## Uninstall

Turn off **Open at login** first (Profile > Notifications), then quit the app.

**macOS:** delete `/Applications/Humanik OS.app`. To remove its settings too,
delete `~/Library/Application Support/Humanik OS`. If you skipped turning off Open
at login, also delete `~/Library/LaunchAgents/io.humanik.desktop.login.plist`.

**Windows:** delete the `%LOCALAPPDATA%\Programs\Humanik OS` folder and the
Humanik OS shortcut in the Start menu. If you skipped turning off Open at login,
switch it off in Settings > Apps > Startup.

Uninstalling the app does not touch humanikd or this machine's enrollment.

## Notifications

See [Notifications](notifications.md).
