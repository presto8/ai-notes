# cmux as my AI agent terminal

[cmux](https://cmux.com) ([source](https://github.com/manaflow-ai/cmux)) is a
free, GPL-licensed macOS terminal for running many AI coding agents at the
same time. It is native Swift and AppKit (no Electron), and it uses libghostty
for rendering. It is not a Ghostty fork: Ghostty is a standalone terminal, and
cmux is a different app built on the same engine. I run Claude Code, Codex,
OpenCode, and Hermes in cmux panes every day. It replaced both Ghostty and
[Herdr](herdr.md) for me on the Mac.

## Why I switched from Herdr

- Agent attention without polling. A pane gets a blue ring, and its workspace
  lights up in the sidebar, when the agent inside needs input. A notification
  panel lists every pending item and jumps to the newest one. This works
  through standard terminal escape sequences (OSC 9/99/777) and through agent
  hooks (`cmux hooks setup`), so any agent with hooks or OSC support works.
- A sidebar that shows context. Each workspace shows its git branch, linked
  PR status, working directory, listening ports, and latest notification
  text.
- One tool instead of two. With Herdr I ran two programs: Ghostty as the
  terminal, then Herdr as a TUI inside it, each with its own configuration,
  keybindings, and notification setup. cmux combines the terminal and the
  multiplexer in one app, so there is one thing to start and configure. It
  also brings native Cmd-key shortcuts, desktop notifications, and Ghostty
  rendering.
- Programmable. Every action is in the `cmux` CLI and the socket API: create
  workspaces, split panes, send keystrokes, drive the built-in browser. Agents
  can arrange their own panes.
- Remote work. `cmux ssh <host>` opens a workspace for a remote machine, and
  cmux can attach to remote tmux sessions. My Linux dev host runs tmux with
  auto-attach, so its agents keep running when the laptop sleeps, and I drive
  them from cmux.

## Configuration

cmux settings live in `~/.config/cmux/cmux.json` (JSON with comments). Terminal
rendering (font, theme, cursor, scrollback) stays in
`~/.config/ghostty/config`. `cmux reload-config` reloads both without a
restart.

My configuration makes Cmd+N open a new workspace that starts Hermes, and
keeps links in my normal browser instead of the cmux in-app browser:

```jsonc
{
  "actions": {
    "hermes-workspace": {
      "type": "workspace",
      "title": "Hermes",
      "restart": "new",
      "shortcut": "cmd+n",
      "workspace": {
        "name": "Hermes",
        "layout": { "pane": { "surfaces": [
          { "type": "terminal", "name": "Hermes", "command": "hermes", "focus": true }
        ] } }
      }
    }
  },
  "shortcuts": { "bindings": { "newTab": null } },
  "ui": {
    "newWorkspace": {
      "action": "hermes-workspace",
      "contextMenu": [
        { "action": "hermes-workspace", "title": "Hermes" },
        { "type": "separator" },
        { "action": "cmux.newWorkspace", "title": "Blank Workspace" },
        { "action": "cmux.newBrowser", "title": "New Browser" }
      ]
    }
  },
  "browser": {
    "openTerminalLinksInCmuxBrowser": false,
    "interceptTerminalOpenCommandInCmuxBrowser": false
  },
  "sidebar": {
    "openPortLinksInCmuxBrowser": false,
    "openPullRequestLinksInCmuxBrowser": false
  }
}
```

Useful commands:

```sh
cmux identify --json           # which window/workspace/pane/surface am I in
cmux list-workspaces           # all workspaces in the current window
cmux new-split right --command "codex"   # new pane that starts an agent
cmux ssh circe                 # workspace for a remote host
cmux hooks setup               # install notification hooks for supported agents
cmux reload-config             # apply cmux.json and Ghostty config changes
```

Note that `app.sendAnonymousTelemetry` defaults to `true`. Set it to `false`
in `cmux.json` if you do not want that.

## Linux

cmux is macOS-only. The project has a waitlist for other platforms, but no
Linux build. I am looking for something that gives the same agent-attention
workflow on a Linux desktop. Candidates, none chosen yet:

- [Herdr](herdr.md). What I used before cmux. It is a TUI, so it runs on
  Linux, and it has agent-state detection and persistent sessions. This is
  the default if nothing better turns up.
- [Zellij](https://zellij.dev). A general terminal multiplexer with layouts
  and plugins, but no built-in agent awareness.
- tmux plus agent hooks. The most portable option; attention signals would
  have to come from agent hooks or a status-line script.
