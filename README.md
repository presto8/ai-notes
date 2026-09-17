# ai-notes

Notes on my personal AI workflow — tools, configurations, and lessons learned,
shared in case they're useful to others.

## What's here

This repo is a running set of notes on how I use AI day to day: agent setups,
CLI tools, configuration tricks, prompts that work well, and things I learned
the hard way. It's informal and grows organically rather than following a
fixed structure.

## Why

Most of what I learn about working with AI tools doesn't fit neatly into a
blog post or a single polished writeup. This is a place to capture it as it
happens.

## Contents

- [Herdr as an AI agent multiplexer](#herdr-as-an-ai-agent-multiplexer)
- [Agents I use](#agents-i-use)
  - [Hermes](#hermes)
  - [OpenCode](#opencode)
  - [Claude Code](#claude-code)
  - [Codex](#codex)
- [skills.sh and npx skills](#skillssh-and-npx-skills)

## Herdr as an AI agent multiplexer

[Herdr](https://herdr.dev) is a terminal multiplexer built specifically around
running multiple AI coding agents at once, rather than being a general-purpose
tmux replacement with agents bolted on. I use it as my daily driver for
running several agent sessions (Claude Code, Codex, OpenCode, Hermes) side by
side across local and remote (SSH) machines.

What makes it useful for this specific job:

- **Agent-state awareness.** Herdr reads the contents of each pane and knows
  whether the agent inside it is working, blocked (waiting on you), or idle.
  You don't have to alt-tab through panes hunting for whichever one is stuck
  waiting for an answer — the sidebar shows you directly.
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

### Config

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

## Agents I use

Short notes on how each agent CLI fits into the workflow above — all of them
run inside Herdr panes day to day.

### Hermes

TODO: notes on Hermes Agent setup, skills, and how I use it day to day.

### OpenCode

TODO: notes on OpenCode setup and workflow.

### Claude Code

TODO: notes on Claude Code setup, including the Foundry-backed BYOK
configuration used at work instead of a standard claude.ai account.

### Codex

TODO: notes on Codex CLI setup and workflow.

## skills.sh and npx skills

[skills.sh](https://skills.sh) is an open ecosystem/registry for "Agent
Skills" — portable `SKILL.md`-based procedural knowledge packages that plug
into most agent CLIs (Claude Code, Codex, OpenCode, Cursor, Hermes, and dozens
more). The `skills` CLI (run via `npx skills`, no install needed) is the tool
for discovering, installing, and syncing them.

Each supported agent has its own project-level and global directory
convention that the CLI writes to automatically, e.g.:

| Agent | Project path | Global path |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.agents/skills/` | (Codex-specific path) |
| Hermes Agent | `.hermes/skills/` | `~/.hermes/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |

### Useful commands

```sh
# Install a skill package from a GitHub repo shorthand
npx skills add vercel-labs/agent-skills

# Install to a specific agent only, and a specific skill within a package
npx skills add vercel-labs/agent-skills --agent claude-code --skill frontend-design

# Install to every supported agent at once
npx skills add vercel-labs/agent-skills --agent '*'

# Try a skill without installing it — pipes a generated prompt straight into an agent
npx skills use vercel-labs/agent-skills@web-design-guidelines | claude

# List what's installed (project + global)
npx skills list
npx skills ls -g            # global only
npx skills ls -a claude-code -a cursor   # filter by agent

# Search/discover interactively
npx skills find
npx skills find typescript

# Keep skills current
npx skills check            # see what has updates available
npx skills update           # update everything
npx skills update my-skill  # update one skill

# Scaffold a new skill
npx skills init my-skill
```

Sources besides a bare `owner/repo` shorthand also work: full GitHub/GitLab
URLs, a direct path to a skill subdirectory within a repo, any git URL
(including private repos, using whatever auth is already configured for that
remote), or a local path.

## License

Copyright (C) 2026 Preston Hunt

Licensed under the GNU General Public License v3.0 — see [LICENSE](LICENSE).
Anyone who distributes this work or a modified version of it must make the
corresponding source available under the same license.
