# Herdr as an AI agent multiplexer

[Herdr](https://herdr.dev) is a terminal multiplexer (a program that runs many
terminal sessions inside one window) built for running many AI coding agents
at the same time. It is not a general tmux replacement with agent features
added later. I use it every day to run several agent sessions (Claude Code,
Codex, OpenCode, Hermes) side by side, on my local machine and on remote
machines over SSH.

## What makes it useful for this job

- Agent-state awareness. Herdr reads the contents of each pane (one terminal
  section inside the window) and knows if the agent inside it is working,
  blocked (stopped and waiting for you), or idle. You do not have to look at
  each pane by hand to find the one that needs you. The sidebar shows an icon
  next to each agent for its state. If Ghostty's bell-based attention feature
  is set up, the Dock icon also tells you when an agent needs input.
- Persistence. Herdr runs as a background server. The terminal sessions live
  inside that server, not inside whatever client is attached to it. If you
  close your laptop lid or lose the network, agent work in progress does not
  stop. If you restart the machine, Herdr restores the layout and resumes the
  sessions.
- Named sessions and SSH access. The commands `herdr session list`, `attach`,
  `stop`, and `delete` give you named workspaces that persist across
  restarts. `herdr --remote <host>` connects to a remote machine over SSH and
  uses your local keybindings, so a remote machine feels the same as your
  local one.
- Herdr does not replace your agent CLIs. Herdr owns the terminal, not the
  agent process. Claude Code, Codex, OpenCode, and other agents run exactly as
  they would without Herdr, inside a Herdr-managed pane.

## Configuration

The configuration file is at `~/.config/herdr/config.toml`. A few settings
from my own configuration that are worth calling out:

```toml
[ui.sound]
enabled = false
```

This setting turns off all of Herdr's own notification sounds (the chimes for
agent-finished and agent-needs-input events). You can also turn off sound for
one agent instead of all agents:

```toml
[ui.sound.agents]
claude = "off"
```

You can also turn off sound once, without changing the configuration file,
with the environment variable `HERDR_DISABLE_SOUND=1`.

These keybindings switch between workspaces. They work the same way over SSH,
because they do not depend on the terminal forwarding the Command or Super
key:

```toml
[keys]
previous_workspace = "prefix+["
next_workspace = "prefix+]"
```

Command keybindings run a shell command, or open a temporary pane that runs
the command. Herdr sets environment variables that scope the command to the
active workspace, tab, or pane (for example `HERDR_ACTIVE_WORKSPACE_ID` and
`HERDR_ACTIVE_PANE_CWD`):

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
