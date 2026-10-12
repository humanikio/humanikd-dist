# Notifications

The Humanik OS app shows system notifications on macOS and Windows. It needs to be
running: a closed or minimised window is fine, a quit app shows nothing.

## What you are notified about

| Notification | When |
| --- | --- |
| A message from an employee | An AI employee writes in your one-to-one chat with it and you have not read it yet. This covers every workspace you are in, not only the one the app is showing |
| An employee has an error | An employee's run, or the machine its office runs on, goes into an error |
| Sign in required | humanikd cannot run agent jobs because Claude Code on this machine is signed out. At most once every 30 minutes |
| Update available | A newer humanikd release exists. At most once every 12 hours for the same release |

Group chats do not notify in the app. They live in the browser.

## One notification per chat

An employee often sends several messages in one reply. The app waits a few
seconds after the newest one, then shows **one** notification for the chat: the
newest message, and "3 new messages" when there were more. A later message in the
same chat replaces that notification instead of adding another.

When you read the chat, in the app or in the browser, its notification is
removed. In Notification Center, notifications are grouped by workspace.

On macOS the Dock icon shows how many chats have unread messages.

Clicking a notification opens the app on that chat. Clicking a sign-in or update
notification opens the Machine page.

## When the app stays quiet

- **While the app window is in front.** You can see the chats then, and the blue
  dots and the count on the icon tell you the rest.
- **For messages that were already unread when the app started.** Only what
  arrives while it runs.
- **During Do Not Disturb or a Focus mode.** The system holds the notification.
  The app cannot see these settings.

## Turning notifications on

The first time the app has something to show, macOS asks whether to allow
notifications from Humanik OS. Windows allows them unless you have turned them
off.

**Profile > Notifications** shows whether notifications are on:

| What you see | What to do |
| --- | --- |
| On | Nothing. **Send a test** shows a sample notification |
| Off | **Turn on**. If you said no before, macOS does not ask again, so this opens System Settings instead |
| Cannot be read on this system | Use **Open System Settings** (or **Open Windows Settings**) and check there |

Where the setting lives:

- **macOS:** System Settings > Notifications > Humanik OS > Allow notifications.
  "Banners" or "Alerts" both work.
- **Windows:** Settings > System > Notifications > Humanik OS. Also check that
  notifications are on at the top of that page.

## Keeping them on after a restart

Notifications come from the app, so they stop when it is not running. **Open at
login**, on by default, starts the app hidden when you log in. See
[Open at login](desktop-app.md#open-at-login).

## Sign-in and update warnings without the app

humanikd shows its own sign-in and update warnings when the app is not running,
has notifications turned off, or is not responding. They look different:

- **macOS:** they come from Script Editor, and follow Script Editor's notification
  setting. Clicking one does nothing.
- **Windows:** they come from Windows PowerShell.

While the app is running and allowed to notify, humanikd hands these warnings to
the app instead, so they show as Humanik OS. This needs humanikd v0.1.21 or later.

humanikd running as a system service (the local-model setup) has no desktop to
show anything on. It writes these warnings to its log only.

## If notifications do not appear

| What you see | What to check |
| --- | --- |
| Nothing, and Profile says Off | Turn them on (above) |
| Nothing, and Profile says On | **Send a test**. If the test does not appear either, check Do Not Disturb and Focus |
| The test appears, but messages do not | Is the app window in front? It stays quiet then. Otherwise, is it a group chat? Those do not notify in the app |
| Nothing after restarting the computer | The app is not running. Turn on **Open at login**, or open the app |
| Sign-in warnings show as Script Editor | The app is not running, its notifications are off, or humanikd is older than v0.1.21 |
