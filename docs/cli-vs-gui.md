# Why I prefer CLI/TUI over desktop apps

I default to terminal and TUI tools over traditional GUI desktop apps
wherever a good terminal option exists — terminal editors, `lazygit` over
GitHub Desktop, `k9s` over a Kubernetes dashboard, agent CLIs over chat-app
wrappers, and so on. A few reasons, some from my own experience and some
backed by what's out there:

- **Startup and response time.** A GUI app — especially anything built on
  Electron — has to spin up a rendering engine, load a framework, draw
  widgets, and manage window chrome before you can do anything. A CLI/TUI
  tool starts, does its job, and gets out of your way. That difference in
  latency isn't just "a bit annoying" — it breaks concentration and flow
  every time you have to wait on it.
- **Resource footprint.** TUI tools don't drag in Electron/Chromium just to
  render some text, which matters a lot on a remote box, a container, or
  anywhere resources are constrained — `htop`/`btop` over a full GUI resource
  monitor is a good example.
- **Remote-first by default.** SSH into a box and a TUI just works; a GUI
  either isn't an option at all or means X11 forwarding and a noticeably
  worse experience. Since a lot of my work happens over SSH (see the Herdr
  notes in the main README), this alone rules out most GUI equivalents.
- **Composability.** Small tools that do one thing well and pipe into each
  other (`fd`, `rg`, `fzf`, etc.) let you build exactly the query you need on
  the fly. There's no GUI equivalent that's as flexible to assemble in the
  moment.
- **AI agents live in the terminal anyway.** Claude Code, Codex, and most
  other coding agents are inherently CLI/terminal-bound tools — they operate
  on files, git, and shells. A terminal-centric workflow is simply the
  native environment for agent-driven development right now, not a
  nostalgia choice.

## Keyboard vs. mouse

Beyond the CLI/TUI question generally, staying on the keyboard instead of
reaching for the mouse is, in my experience, a meaningful efficiency gain on
its own — every hand movement from keyboard to mouse and back costs time and
breaks typing flow. This one is genuinely debated rather than settled
science: there are old, widely cited claims that the mouse is actually faster
for most tasks (see [Dan Luu's dig into where those studies actually came
from](https://danluu.com/keyboard-v-mouse/)), and the honest conclusion is
that it depends heavily on the task — raw text manipulation and navigating a
tool you already know well tends to favor the keyboard, while some pointing
and selection tasks genuinely favor the mouse. For the kind of work I do
(editing, navigating agent panes, git operations, running commands), staying
keyboard-driven wins in practice, which is exactly why I gravitate toward
tools that are designed keyboard-first rather than mouse-first.
