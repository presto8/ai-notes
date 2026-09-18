# ai-notes

Notes on my AI workflow: tools, configurations, and lessons learned. I share
them here in case they help others. Most of what I learn about AI tools does
not fit into a blog post or a single polished article, so this repository is
where I record each lesson as it happens.

## What is here

This repository is a running set of notes on how I use AI each day: agent
setups, CLI tools, configuration tricks, prompts that work well, and lessons I
learned the hard way. The notes are informal. The repository grows over time
and does not follow a fixed structure.

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
- [gptel in Emacs](#gptel-in-emacs)
- [OpenRouter](#openrouter)
- [Neovim with Copilot](#neovim-with-copilot)
- [Models I use](#models-i-use)
- [Using GitHub to manage an agentic workflow](#using-github-to-manage-an-agentic-workflow)
- [Todo](#todo)

## Why I prefer CLI/TUI over desktop apps

I use terminal and TUI (text user interface, a full-screen program you control
from the keyboard) tools instead of desktop apps whenever a good option
exists. Terminal tools start faster, use fewer resources, work over SSH, and
let me stay on the keyboard instead of the mouse. Read
[docs/cli-vs-gui.md](docs/cli-vs-gui.md) for the full explanation, including
the keyboard-versus-mouse question.

## Herdr as an AI agent multiplexer

[Herdr](https://herdr.dev) is a terminal multiplexer (a program that runs many
terminal sessions inside one window) built for running many AI coding agents
at the same time. It is not a general tmux replacement with agent features
added later. I use it every day to run several agent sessions (Claude Code,
Codex, OpenCode, Hermes) side by side, on my local machine and on remote
machines over SSH. Read [docs/herdr.md](docs/herdr.md) for what makes it
useful for this job and for my configuration notes.

## Agents I use

Short notes on how each agent CLI fits into the workflow above. I run all of
them inside Herdr panes each day.

### Hermes

TODO: notes on Hermes Agent setup, skills, and daily use.

### OpenCode

TODO: notes on OpenCode setup and workflow.

### Claude Code

I use the Claude Code CLI. TODO: notes on setup, including the Foundry-backed
BYOK (bring your own key) configuration I use at work instead of a standard
claude.ai account.

### Codex

I use the Codex CLI. TODO: notes on setup and workflow.

## skills.sh and npx skills

[skills.sh](https://skills.sh) is an open registry for Agent Skills. An Agent
Skill is a portable package of procedural knowledge, defined in a file named
`SKILL.md`. Most agent CLIs can read these packages, including Claude Code,
Codex, OpenCode, Cursor, and Hermes. The `skills` CLI runs with `npx skills`
and needs no separate install. Use it to find, install, and update skill
packages.

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

You can also install from other sources: a full GitHub or GitLab URL, a direct
path to a skill subdirectory in a repository, any git URL, or a path on your
local disk. A private repository works too. The CLI uses the authentication
that your git client already has for that remote.

## Skills I use

Two skills I installed with `npx skills add` and use often:

- [SimpleEnglish](https://github.com/AminBlg/SimpleEnglish). This skill makes
  an agent write in [ASD-STE100](https://www.asd-ste100.org/) Simplified
  Technical English. This is a controlled language that aerospace has used
  since 1983 to make instructions hard to misread. The rules include short
  sentences, active voice, one instruction per sentence, and no hedging words
  such as "should" or "might". I use it when I want an agent's writing
  (documents, replies, commit messages) to read as plain and clear instead of
  typical AI prose. Install:

  ```sh
  npx skills add AminBlg/SimpleEnglish
  ```

- [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill).
  This skill turns an agent into a security auditor that works in six phases.
  It maps the codebase, hunts for vulnerabilities with several parallel
  agents, and tries to disprove each finding with a separate validation agent.
  It then writes a human-readable report, writes the same findings as
  structured JSON, and verifies every claim in that JSON against the source
  code with a fresh agent. Cloudflare used this skill to start its own
  internal vulnerability-discovery system. Install:

  ```sh
  npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
  ```

  After you install it, ask the agent to "security audit this codebase" or
  "find security vulnerabilities in ./src". The skill activates on its own.

## gptel in Emacs

[gptel](https://github.com/karthink/gptel) is the Emacs package I use for LLM
chat and text completion inside the editor. It is a light client, not a full
IDE-agent integration. It supports many backends (OpenAI, Anthropic, Gemini,
Ollama, and more) through one interface. Because of this, a model
switch is a configuration change, not a change in how you work.

The function `gptel-make-anthropic` lets you point gptel at any endpoint that
speaks the Anthropic Messages API, not only `api.anthropic.com`. This is
useful for a company-hosted gateway endpoint:

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
  instead of writing it directly in `init.el`. This matters if `init.el` ever
  ends up in a dotfiles repository. Both `:key` and the values inside
  `:header` accept a function, so a function that reads `getenv` works here.
- `:models` lists the model IDs that the endpoint serves. gptel uses this list
  to build its model-switching menu (`gptel-menu`, or the transient menu from
  `M-x gptel-send`). The list must match the names your gateway uses. These
  are not always the public Anthropic model names.
- Install the package through `elpa` and `use-package` as normal. It needs no
  setup beyond `:ensure t` and the backend configuration shown above.

## OpenRouter

[OpenRouter](https://openrouter.ai) is a router that sits in front of many
model providers (Anthropic, OpenAI, Google, Meta, DeepSeek, Mistral,
Groq-hosted models, and many open-weight models) behind one OpenAI-compatible
endpoint and API key. I use it to reach models outside my main Anthropic and
Foundry setup, without a separate account and key for each provider.

Reasons I use it:

- One key, many models. Any tool that already speaks the OpenAI chat
  completions API can point at OpenRouter (`https://openrouter.ai/api/v1`)
  and reach the full model catalog. You only change the model name.
- Good for trying a model once. When I want to compare a response against an
  open-weight or non-Anthropic model, OpenRouter is the fastest path. I do
  not need a separate account for that provider.
- Fallback routing. If a request's primary provider or model is down or
  rate-limited, OpenRouter can send the request to a different provider or
  model. You can configure this per request.
- Popularity data. OpenRouter publishes which models its users call the
  most. The [rankings page](https://openrouter.ai/rankings) shows token
  volume by model, and you can filter it by category, such as programming
  or roleplay. The [models page](https://openrouter.ai/models?order=top-weekly)
  sorts the full catalog by weekly usage. This is a fast way to see which
  models people use in practice, not only which ones benchmark well.
- An automatic model. The [Auto Router](https://openrouter.ai/openrouter/auto)
  (`openrouter/auto`) reads each prompt and picks a model for it from a
  curated set. Use it when you do not want to choose a model yourself. The
  response says which model it used. Read the
  [Auto Router documentation](https://openrouter.ai/docs/features/model-routing)
  for the details.

Most of my agent CLIs and editor tools accept a custom OpenAI-compatible base
URL. To use OpenRouter with them: set the base URL to
`https://openrouter.ai/api/v1`, set the API key, and pick a model from the
OpenRouter catalog. Each model name carries a provider prefix, for example
`deepseek/deepseek-v4.1`.

### Fees when you add funds

OpenRouter charges a processing fee, close to 5 percent, on every deposit,
and it applies a minimum charge. A small deposit pays a much higher effective
rate because of that minimum. Add at least $15 at a time to keep the fee from
taking an outsized share of the deposit. I pay with a rewards credit card that
returns 2 percent cash back. This brings my net fee down to close to 3
percent.

## Neovim with Copilot

For pair programming inside the editor, ghost-text suggestions (dimmed text
shown ahead of your cursor), and autocomplete, I use
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

I keep the default keybindings. Run `:Copilot setup` (or `:Copilot auth`)
once to sign in. After that, the plugin works without further setup:

- Suggestions show as ghost text while you type in insert mode.
- Press `Tab` to accept the full suggestion. The plugin remaps `Tab` for this
  by default.
- Press `Alt-]` or `Alt-[` to move to the next or previous suggestion, when
  more than one is available.
- Press `Ctrl-]` to dismiss the current suggestion without accepting it.

If `Tab` is already bound to something else, for example a snippet engine or a
completion plugin, the plugin documentation shows how to remap accept to a
different key. Set `g:copilot_no_tab_map` and add a manual mapping to
`<Plug>(copilot-accept)`. I have not needed this, because I do not run a
competing `Tab` mapping in insert mode.

Notes from daily use:

- The plugin only suggests code. It has no chat panel. For a conversation
  with an agent inside Neovim, I run a terminal-based agent (Claude Code,
  Codex, and so on) in a split window or a Herdr pane next to the editor.
- Run `:Copilot status` to make sure that the plugin is signed in and active
  for the current file type. Use this command first if suggestions stop.
- `:Copilot disable` and `:Copilot enable` turn the plugin off and on for the
  current session. This is useful when you work on something sensitive, or
  when the suggestions add more noise than value for a given file.

## Models I use

Across all the agents and tools listed above, I use one model for most work
and reserve the others for specific cases:

- Claude Sonnet 5. My default for almost everything: daily coding, chat, and
  agent work. It gives a good balance of speed and capability for most tasks.
- Claude Opus. I use this only for planning and design work, when I want
  deeper reasoning before I commit to an approach. I do not use it for
  routine execution. It costs more and runs slower than routine tasks need.
- Claude Fable. I use this rarely. It costs close to 4 times what Sonnet 5
  costs. That price is hard to justify outside a narrow set of cases where
  its output fits the task better than Sonnet's.
- DeepSeek 4.1. I use this through OpenRouter, for tasks where I want to
  compare against, or use, an open-weight model instead of Claude.
- GLM 5.3. I use this through OpenRouter, for the same reason as DeepSeek
  above.
- Groq/Compound. I use this through Groq's own API, not through OpenRouter. I
  reach for it when I want very fast inference and the task does not need
  Claude-level reasoning depth. It is free as far as I have seen. I have never
  hit a usage limit, and it is extremely fast.

The general pattern: pick the cheapest and fastest model that still gives a
reliable result for the task. Use a larger model only when the task itself is
hard (planning, architecture decisions, an ambiguous problem), not because the
task is long.

## Using GitHub to manage an agentic workflow

TODO: notes on how I use GitHub itself as a guardrail around agentic coding
work. This includes a protected `main` branch, all changes through a pull
request (including my own solo work), required linear history with rebase
merges, and branch rulesets. The goal is to keep an agent from pushing
straight to `main` or rewriting history it must not touch.

## License

Copyright (C) 2026 Preston Hunt

This work is under the GNU General Public License, version 3.0. Read the full
text in [LICENSE](LICENSE). If you distribute this work, or a modified version
of it, you must make the corresponding source available under the same
license.

## Todo

Tools and ideas I want to look into, but have not tried yet:

- [seshagy](https://github.com/lmilojevicc/seshagy). An agent-aware terminal
  dashboard that shows your project directories, active multiplexer sessions,
  and terminal-based coding agents in one view. It has native support for
  both tmux and Herdr. Under Herdr, it uses the agent-state detection that
  Herdr provides, so it lines up with the Herdr setup described above.
