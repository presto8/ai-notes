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
- [gptel.el in Emacs](#gptelel-in-emacs)
- [OpenRouter](#openrouter)
- [Neovim with Copilot](#neovim-with-copilot)
- [Models I use](#models-i-use)

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

## gptel.el in Emacs

[gptel](https://github.com/karthink/gptel) is the Emacs package I use for
in-editor LLM chat and completion — a lightweight universal client rather than
a heavyweight IDE-agent integration. It supports many backends (OpenAI,
Anthropic, Gemini, Ollama, Bedrock, and more) through a common interface, so
switching models is a config-level concern, not a workflow change.

The useful trick is `gptel-make-anthropic`, which lets you point gptel at any
Anthropic-Messages-API-compatible endpoint instead of just `api.anthropic.com`
— handy for a company-hosted Foundry/gateway endpoint:

```elisp
(defun use-anthropic ()
  (gptel-make-anthropic "Claude-Foundry"
    :host     "my-foundry-host.example.com"  ; no scheme, no path
    :endpoint "/v1/messages"                 ; base-url path + /v1/messages
    :stream   t
    :key      (lambda () (getenv "ANTHROPIC_API_KEY"))  ; never hardcode the key
    :header   (lambda ()
                (let ((k (getenv "ANTHROPIC_API_KEY")))
                  `(("x-api-key" . ,k)
                    ("api-key"   . ,k)
                    ("anthropic-version" . "2023-06-01"))))
    :models   '(claude-sonnet-5 claude-opus-5)))

(use-package gptel
  :config
  (use-anthropic)
  :ensure t)
```

Notes:

- Pull the API key from an environment variable (or a proper secrets store)
  rather than hardcoding it in `init.el` — especially if `init.el` ever ends
  up in a dotfiles repo. `:key` and the values inside `:header` both accept a
  function, so a lambda reading `getenv` works cleanly.
- `:models` is a list of model IDs the endpoint actually serves; gptel uses
  this to populate its model-switching UI (`gptel-menu` / `M-x gptel-send`
  transient), so it needs to match what your gateway calls them, not
  necessarily Anthropic's public model names.
- Package installed via `elpa`/`use-package` as normal; no special setup
  beyond `:ensure t` and the backend config above.

## OpenRouter

[OpenRouter](https://openrouter.ai) is a unified API/router that sits in
front of dozens of model providers (Anthropic, OpenAI, Google, Meta, DeepSeek,
Mistral, Groq-hosted models, and many open-weight models) behind a single
OpenAI-compatible endpoint and API key. I use it as a way to reach models that
aren't part of my primary Anthropic/Foundry setup without juggling a separate
account and key per provider.

Practical reasons to reach for it:

- **One key, many models.** Any tool that already speaks the OpenAI chat
  completions API can point at OpenRouter (`https://openrouter.ai/api/v1`)
  and get access to the full model catalog just by changing the model string.
- **Good for trying a model once.** When I want to compare a specific
  response against an open-weight or non-Anthropic model without setting up
  a dedicated provider account, OpenRouter is the fastest path.
- **Fallback routing.** OpenRouter can automatically fall back to an
  alternate provider/model if the primary one is down or rate-limited,
  configurable per request.

Most of my agent CLIs and editor integrations that support a custom
OpenAI-compatible base URL can be pointed at OpenRouter the same way they'd
be pointed at any other compatible endpoint — set the base URL to
`https://openrouter.ai/api/v1`, set the API key, and pick a model from
OpenRouter's catalog (provider-prefixed, e.g. `deepseek/deepseek-v4.1` or
`groq/compound`).

## Neovim with Copilot

For editor-integrated pair programming, ghost-text style inline suggestions,
and autocomplete, I use [github/copilot.vim](https://github.com/github/copilot.vim)
— the official Vim/Neovim plugin — installed via
[lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
require("lazy").setup({
  -- ...
  "github/copilot.vim",
  -- ...
})
```

I run it with the plugin's defaults rather than customizing keybindings —
after `:Copilot setup` (or `:Copilot auth`) to authenticate once, it just
works:

- Suggestions appear as ghost text (dimmed inline text) as you type in insert
  mode.
- `Tab` accepts the full suggestion (the plugin remaps `Tab` for this by
  default).
- `Alt-]` / `Alt-[` cycle to the next/previous suggestion when more than one
  is available.
- `Ctrl-]` dismisses the current suggestion without accepting it.

If `Tab` is already bound to something else (e.g. a snippet engine or
completion plugin), the plugin's docs cover remapping accept to a different
key via `g:copilot_no_tab_map` plus a manual `<Plug>(copilot-accept)` mapping
— I haven't needed to because I don't run a conflicting `Tab` mapping in
insert mode.

A couple of practical notes from using it day to day:

- It's suggestion-only (no chat panel) — for actual conversational
  pair-programming inside Neovim I reach for a terminal-based agent (Claude
  Code, Codex, etc.) running in a split or a Herdr pane alongside the editor,
  rather than a chat UI baked into Neovim itself.
- `:Copilot status` is the fastest way to confirm it's authenticated and
  enabled for the current buffer/filetype when suggestions unexpectedly stop
  appearing.
- `:Copilot disable` / `:Copilot enable` toggles it per-session, useful when
  working on something sensitive or when the suggestions are more noise than
  signal for a particular file.

## Models I use

Day to day, across all of the agents and tools above, I mostly reach for one
model and reserve the others for specific situations:

- **Claude Sonnet 5** — default for almost everything: day-to-day coding,
  chat, agent work. Good balance of speed and capability for the bulk of
  tasks.
- **Claude Opus** — occasionally, specifically for planning and design work
  where I want deeper reasoning before committing to an approach. Not used
  for routine execution — too slow/expensive for that.
- **Claude Fable** — rarely, for specific cases where its behavior fits
  better than Sonnet's.
- **DeepSeek 4.1** — via OpenRouter, for specific tasks where I want to
  compare against or use an open-weight model instead of Claude.
- **GLM 5.3** — via OpenRouter, same category as DeepSeek above.
- **Groq/Compound** — via OpenRouter (Groq-hosted), when I want very fast
  inference and the task doesn't need Claude-level reasoning depth.

The pattern in general: pick the cheapest/fastest model that's still reliable
for the task, and only reach for a heavier model when the task is genuinely
hard (planning, architecture decisions, ambiguous problems) rather than just
long.

## License

Copyright (C) 2026 Preston Hunt

Licensed under the GNU General Public License v3.0 — see [LICENSE](LICENSE).
Anyone who distributes this work or a modified version of it must make the
corresponding source available under the same license.
