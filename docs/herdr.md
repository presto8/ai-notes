# Herdr as an AI agent multiplexer

[Herdr](https://herdr.dev) is a terminal multiplexer built specifically around
running multiple AI coding agents at once, rather than being a general-purpose
tmux replacement with agents bolted on. I use it as my daily driver for
running several agent sessions (Claude Code, Codex, OpenCode, Hermes) side by
side across local and remote (SSH) machines.

## What makes it useful for this specific job

- **Agent-state awareness.** Herdr reads the contents of each pane and knows
  whether the agent inside it is working, blocked (waiting on you), or idle.
  You don't have to alt-tab through panes hunting for whichever one is stuck
  waiting for an answer — the sidebar shows you directly, with an icon next
  to each agent indicating its state, so a glance at the sidebar (or the Dock
  icon, if Ghostty's bell-based attention feature is wired up) tells you
  which agent needs input without opening any of them.
- **Persistence.** It runs as a background server; the terminals live inside
  it, not inside whatever client is attached. Closing the laptop lid or losing
  the network doesn't kill in-flight agent work. Reboot the machine and it
  restores the layout and resumes sessions.
- **Named sessions + SSH remoting.** `herdr session list/attach/stop/delete`
  gives you persistent named workspaces. `herdr --remote <host>` attaches over
  SSH using your local keybindings, so a remote dev box feels the same as
  local.
- **It doesn't replace your agent CLIs.** Herdr owns the terminal, not the
  agent process. Claude Code, Codex, OpenCode, etc. run exactly as they
  normally would inside a Herdr-managed pane.

## Config

Config lives at `~/.config/herdr/config.toml`. A few things worth calling out
from my own config:

```toml
[ui.sound]
enabled = false
```

Fully suppresses Herdr's own notification sounds (agent-finished /
agent-needs-input chimes). You can also mute per-agent instead of globally:

```toml
[ui.sound.agents]
claude = "off"
```

or set a one-off override without touching the config file at all via
`HERDR_DISABLE_SOUND=1`.

Custom keybindings for cross-workspace navigation that work identically over
SSH (without relying on the terminal forwarding the Command/Super modifier):

```toml
[keys]
previous_workspace = "prefix+["
next_workspace = "prefix+]"
```

Command keybindings can run arbitrary shell commands or open a temporary pane
for a command, scoped to the active workspace/tab/pane via environment
variables Herdr injects (`HERDR_ACTIVE_WORKSPACE_ID`, `HERDR_ACTIVE_PANE_CWD`,
etc.):

```toml
[[keys.command]]
key = "g"
type = "pane"
command = "lazygit"
```

Useful commands:

```sh
herdr --default-config          # print the full default config for reference
herdr server reload-config      # apply config.toml changes to the running server without restarting
herdr status server             # confirm the server is up and check version
herdr session list --json       # see all named sessions
```
