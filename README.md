# ai-notes

Notes on my AI workflow. I share them here in case they help others.

## What is here

This repository holds notes on how I use AI tools each day: agent setups,
CLI tools, configuration tricks, prompts that work well, and lessons I
learned the hard way. The notes are informal. The repository grows over
time and does not follow a fixed structure.

## Why this repository exists

Most lessons about AI tools do not fit into a blog post or a single
polished article. This repository is a place to record each lesson as it
happens.

## Contents

- [Why I prefer CLI/TUI over desktop apps](#why-i-prefer-clitui-over-desktop-apps)
- [Herdr as an AI agent multiplexer](#herdr-as-an-ai-agent-multiplexer)
- [Agents I use](#agents-i-use)
  - [Hermes](#hermes)
  - [OpenCode](#opencode)
  - [Claude Code](#claude-code)
  - [Codex](#codex)
- [skills.sh and npx skills](#skillssh-and-npx-skills)
- [Skills I use](#skills-i-use)
- [gptel.el in Emacs](#gptelel-in-emacs)
- [OpenRouter](#openrouter)
- [Neovim with Copilot](#neovim-with-copilot)
- [Models I use](#models-i-use)

## Why I prefer CLI/TUI over desktop apps

I use terminal and TUI (text user interface, a program you control fully
from the keyboard) tools instead of desktop apps whenever a good option
exists. Terminal tools start faster, use fewer resources, work well over
SSH, and let me control them from the keyboard instead of the mouse. Read
[docs/cli-vs-gui.md](docs/cli-vs-gui.md) for the full explanation,
including the keyboard-versus-mouse question.

## Herdr as an AI agent multiplexer

[Herdr](https://herdr.dev) is a terminal multiplexer (a tool that runs many
terminal sessions inside one window) built for running many AI coding
agents at the same time. It is not a general tmux replacement with agents
added later. I use it every day to run several agent sessions (Claude Code,
Codex, OpenCode, Hermes) side by side, on my local machine and on remote
machines over SSH.

Herdr helps in these ways:

- Agent-state awareness. Herdr reads the contents of each pane (a terminal
  window section) and knows if the agent inside it is working, blocked
  (stopped and waiting for you), or idle. You do not have to check each
  pane by hand to find the one that needs you. The sidebar shows an icon
  next to each agent for its state. If Ghostty's bell-based attention
  feature is set up, the Dock icon also shows you when an agent needs
  input.
- Persistence. Herdr runs as a background server. The terminal sessions
  live inside that server, not inside whatever client is attached to it.
  If you close your laptop lid or lose network, agent work in progress does
  not stop. If you restart the machine, Herdr restores the layout and
  resumes the sessions.
- Named sessions and SSH access. The commands `herdr session list`,
  `attach`, `stop`, and `delete` give you named workspaces that persist
  across restarts. `herdr --remote <host>` connects to a remote machine
  over SSH and uses your local keybindings, so a remote machine feels the
  same as your local one.
- Herdr does not replace your agent CLIs. Herdr owns the terminal, not the
  agent process. Claude Code, Codex, OpenCode, and other agents run exactly
  as they would without Herdr, inside a Herdr-managed pane.

### Config

The config file is at `~/.config/herdr/config.toml`. A few settings from my
own config, worth calling out:

```toml
[ui.sound]
enabled = false
```

This setting turns off all of Herdr's own notification sounds (the chimes
for agent-finished and agent-needs-input events). You can also turn off
sound for one agent instead of all agents:

```toml
[ui.sound.agents]
claude = "off"
```

You can also turn off sound once, without changing the config file, with
the environment variable `HERDR_DISABLE_SOUND=1`.

These keybindings let you switch workspaces. They work the same way over
SSH, because they do not depend on the terminal forwarding the
Command/Super key:

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

## Agents I use

Short notes on how each agent CLI fits into the workflow above. I run all
of them inside Herdr panes each day.

### Hermes

TODO: notes on Hermes Agent setup, skills, and daily use.

### OpenCode

TODO: notes on OpenCode setup and workflow.

### Claude Code

I use the Claude Code CLI. TODO: notes on setup, including the
Foundry-backed BYOK (bring your own key) configuration I use at work
instead of a standard claude.ai account.

### Codex

I use the Codex CLI. TODO: notes on setup and workflow.

## skills.sh and npx skills

[skills.sh](https://skills.sh) is an open registry for Agent Skills:
portable packages of procedural knowledge, each defined in a file named
`SKILL.md`. Most agent CLIs can read these packages, including Claude Code,
Codex, OpenCode, Cursor, and Hermes. The `skills` CLI, run with `npx
skills`, needs no separate install. Use it to find, install, and update
skill packages.

Each supported agent has its own project-level and global directory for
skills. The CLI writes to the correct directory for you. For example:

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

You can also install from other sources: a full GitHub or GitLab URL, a
direct path to a skill subdirectory in a repository, any git URL (including
a private repository, using whatever authentication your git client
already has for that remote), or a path on your local disk.

## Skills I use

Two skills I installed with `npx skills add` and use often:

- [SimpleEnglish](https://github.com/AminBlg/SimpleEnglish). This skill
  makes an agent write in ASD-STE100 Simplified Technical English, a
  controlled language that aerospace has used since 1983 to make
  instructions hard to misread. The rules include short sentences, active
  voice, one instruction per sentence, and no hedging words like "should"
  or "might". I use it when I want an agent's writing, in documents,
  replies, or commit messages, to read as plain and clear instead of
  typical AI-generated prose. Install:

  ```sh
  npx skills add AminBlg/SimpleEnglish
  ```

- [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill).
  This skill turns an agent into a security auditor that runs in six
  phases: it maps the codebase, hunts for vulnerabilities with several
  parallel agents, tries to disprove each finding with a separate
  validation agent, writes a human-readable report, writes the same
  findings as structured JSON, and verifies every claim in that JSON
  against the real source code with a fresh agent. Cloudflare used this
  skill to start its own internal vulnerability-discovery system. Install:

  ```sh
  npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
  ```

  After you install it, ask the agent to "security audit this codebase" or
  "find security vulnerabilities in ./src". The skill activates on its
  own.

## gptel.el in Emacs

[gptel](https://github.com/karthink/gptel) is the Emacs package I use for
LLM chat and text completion inside the editor. It is a light client, not a
full IDE-agent integration. It supports many backends (OpenAI, Anthropic,
Gemini, Ollama, Bedrock, and more) through one interface, so switching
models is a config change, not a change in how you work.

The function `gptel-make-anthropic` lets you point gptel at any endpoint
that speaks the Anthropic Messages API, not only `api.anthropic.com`. This
is useful for a company-hosted gateway endpoint:

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

- Read the API key from an environment variable, or from a secrets store,
  instead of writing it directly in `init.el`. This matters if `init.el`
  ever ends up in a dotfiles repository. Both `:key` and the values inside
  `:header` can hold a function, so a function that reads `getenv` works
  well here.
- `:models` lists the model IDs that the endpoint serves. gptel uses this
  list to build its model-switching menu (`gptel-menu`, or the transient
  menu from `M-x gptel-send`). The list must match the names your gateway
  uses, which are not always Anthropic's public model names.
- Install the package through `elpa` and `use-package` as normal. It needs
  no setup beyond `:ensure t` and the backend config shown above.

## OpenRouter

[OpenRouter](https://openrouter.ai) is a router that sits in front of many
model providers (Anthropic, OpenAI, Google, Meta, DeepSeek, Mistral,
Groq-hosted models, and many open-weight models) behind one
OpenAI-compatible endpoint and API key. I use it to reach models outside my
main Anthropic and Foundry setup, without a separate account and key for
each provider.

Reasons I reach for it:

- One key, many models. Any tool that already speaks the OpenAI chat
  completions API can point at OpenRouter
  (`https://openrouter.ai/api/v1`) and reach the full model catalog. You
  only need to change the model name.
- Good for trying a model once. When I want to compare a response against
  an open-weight or non-Anthropic model, without setting up a separate
  account for that provider, OpenRouter is the fastest path.
- Fallback routing. If a request's primary provider or model is down or
  rate-limited, OpenRouter can route the request to a different provider
  or model instead. You can configure this per request.

Most of my agent CLIs and editor tools that support a custom
OpenAI-compatible base URL can point at OpenRouter the same way they point
at any other compatible endpoint: set the base URL to
`https://openrouter.ai/api/v1`, set the API key, and pick a model from
OpenRouter's catalog. Each model name carries a provider prefix, for
example `deepseek/deepseek-v4.1` or `groq/compound`.

### Fees when you add funds

OpenRouter charges a processing fee, close to 5 percent, on every deposit,
and it applies a minimum charge. A small deposit pays a much higher
effective rate because of that minimum. Add at least $15 at a time to keep
the fee from taking an outsized share of the deposit. I pay with a rewards
credit card that returns 2 percent cash back, which brings my net fee down
to close to 3 percent.

## Neovim with Copilot

For pair programming inside the editor, ghost-text style inline
suggestions (dimmed text shown ahead of your cursor as a suggestion), and
autocomplete, I use
[github/copilot.vim](https://github.com/github/copilot.vim). This is the
official Vim and Neovim plugin. I install it with
[lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
require("lazy").setup({
  -- ...
  "github/copilot.vim",
  -- ...
})
```

I keep the plugin's default keybindings. Run `:Copilot setup` (or
`:Copilot auth`) once to sign in, and the plugin works from there:

- Suggestions show as ghost text while you type in insert mode.
- Press `Tab` to accept the full suggestion. The plugin remaps `Tab` for
  this by default.
- Press `Alt-]` or `Alt-[` to move to the next or previous suggestion, when
  more than one is available.
- Press `Ctrl-]` to dismiss the current suggestion without accepting it.

If `Tab` is already bound to something else, for example a snippet engine
or a completion plugin, the plugin's own documentation shows how to remap
accept to a different key. This uses the setting `g:copilot_no_tab_map`
with a manual mapping to `<Plug>(copilot-accept)`. I have not needed to do
this, because I do not run a competing `Tab` mapping in insert mode.

A few notes from daily use:

- The plugin only suggests code. It has no chat panel. For a conversation
  with an agent inside Neovim, I run a terminal-based agent (Claude Code,
  Codex, and so on) in a split window or a Herdr pane next to the editor,
  instead of a chat panel built into Neovim.
- Run `:Copilot status` to confirm that the plugin is signed in and active
  for the current file type. Use this command first if suggestions stop
  appearing.
- `:Copilot disable` and `:Copilot enable` turn the plugin off and on for
  the current session. This is useful when you work on something sensitive
  or when the suggestions add more noise than value for a given file.

## Models I use

Across all the agents and tools listed above, I use one model for most
work and reserve the others for specific cases:

- Claude Sonnet 5. My default for almost everything: daily coding, chat,
  and agent work. It gives a good balance of speed and capability for most
  tasks.
- Claude Opus. I use this only for planning and design work, when I want
  deeper reasoning before I commit to an approach. I do not use it for
  routine execution, because it costs more and runs slower than the task
  needs.
- Claude Fable. I use this rarely, for specific cases where its output
  fits the task better than Sonnet's.
- DeepSeek 4.1. I use this through OpenRouter, for specific tasks where I
  want to compare against, or use, an open-weight model instead of Claude.
- GLM 5.3. I use this through OpenRouter, for the same reason as DeepSeek
  above.
- Groq/Compound. I use this through OpenRouter (hosted by Groq) when I
  want very fast inference and the task does not need Claude-level
  reasoning depth.

The general pattern: pick the cheapest and fastest model that still gives
a reliable result for the task. Reach for a larger model only when the
task itself is hard, for example planning, architecture decisions, or an
ambiguous problem, not simply because the task is long.

## License

Copyright (C) 2026 Preston Hunt

This work is under the GNU General Public License, version 3.0. Read the
full text in [LICENSE](LICENSE). If you distribute this work, or a
modified version of it, you must make the corresponding source available
under the same license.
