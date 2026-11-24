# herdr-resurrect-autostart

A tiny plugin for the **Herdr CLI/TUI** (not the desktop app) that makes
[herdr-resurrect](https://github.com/ntindle/herdr-resurrect) restore your session
**as soon as the server starts**.

(中文：讓 herdr-resurrect 在 Herdr server 重啟後立刻自動還原，不必等第一個事件。)

## Why

herdr-resurrect can auto-restore (`autoRestore`), but it only fires on the *first plugin event*
after a server restart, and Herdr has no "server ready" event. If you restart and touch nothing,
agents that Herdr does not resume natively (for example Codex) stay as bare shells until you switch
a tab or open a pane.

This plugin uses Herdr's `[[startup]]` hook, which runs once after the session is restored and the API
socket is ready. The whole plugin is one command:

```sh
sleep 2; "$HERDR_BIN_PATH" plugin action invoke restore --plugin ntindle.herdr-resurrect
```

The 2 second sleep lets the restored shells finish loading their rc files, so herdr-resurrect does not
mistake a starting shell for a busy one.

It does not copy or modify any herdr-resurrect code; it only calls its public `restore` action.

## Requirements

- Herdr CLI/TUI 0.9.0 or newer
- [herdr-resurrect](https://github.com/ntindle/herdr-resurrect) installed and enabled
  (`herdr plugin install ntindle/herdr-resurrect`)
- Read both plugins' code before installing: plugins are not sandboxed.

## Install

```sh
herdr plugin install jim19950427/herdr-resurrect-autostart
```

## Recommended herdr-resurrect settings (yours to change, not bundled)

In `~/.config/herdr/plugins/config/ntindle.herdr-resurrect/`:

1. `settings.json`: keep `"autoRestore": true`. It protects the good snapshot from being overwritten
   by an autosave that happens before the restore reads it. Both triggers can fire in one boot; `restore`
   skips panes that are already running something.
2. `allowlist.txt`: add the agents you use, one per line (`codex`, `claude`). A snapshot taken while
   an agent is still starting (before Herdr detects it) records it as a plain command, and plain commands
   are only relaunched if on the allowlist. The flag is decided when the snapshot is saved, so it
   helps from the next snapshot on.

## Limits

- Only tested on macOS with Herdr 0.9.5 and herdr-resurrect `dab5bed`.
- Not tested: installing straight from GitHub with the command above (developed with `herdr plugin link`).
- If herdr-resurrect renames or removes its `restore` action, this plugin stops doing anything.
- Codex session ids come from herdr-resurrect scanning Codex's local files, so several Codex conversations in
  one directory can resume the wrong one.

## License

MIT
