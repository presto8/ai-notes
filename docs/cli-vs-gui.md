# Why I prefer CLI/TUI over desktop apps

I use terminal and TUI (text user interface) tools instead of desktop apps
whenever a good terminal option exists. For example, I use `lazygit` instead
of GitHub Desktop, `k9s` instead of a Kubernetes dashboard app, and agent CLIs
instead of chat-app wrappers. The reasons below come from my own experience
and from what other people have written.

- Startup and response time. A desktop app, especially one built on
  Electron, must start a rendering engine, load a framework, draw its
  widgets, and manage its window before you can use it. A CLI or TUI tool
  starts, does its job, and gets out of your way. This difference in speed is
  not a small annoyance. Each time an app makes you wait, it breaks your
  concentration.
- Resource use. TUI tools do not need Electron or a browser engine to show
  text. This matters on a remote machine, inside a container, or anywhere
  resources are limited. `htop` and `btop` show this well when you compare
  them with a full graphical resource monitor.
- Works over SSH. Connect to a machine over SSH, and a TUI tool works right
  away. A desktop app either does not run at all, or needs X11 forwarding,
  which gives a worse experience. I do most of my work over SSH (read the
  Herdr notes in the main README), so this alone rules out most desktop
  apps.
- Tools combine well. Small tools that each do one job, and pass data to
  each other through pipes, let you build the exact query you need on the
  spot. No desktop app gives you the same freedom to combine tools in the
  moment.
- AI agents already live in the terminal. Claude Code, Codex, and most other
  coding agents are terminal tools by nature. They work with files, git, and
  the shell. A terminal-based workflow is the natural fit for agent-driven
  development today. It is not a nostalgic choice.

## Keyboard versus mouse

Beyond the CLI and TUI question, staying on the keyboard instead of reaching
for the mouse gives a real efficiency gain on its own, in my experience. Each
time your hand moves from the keyboard to the mouse and back, you lose time
and break your typing flow.

This point is disputed, not settled fact. Old and often-cited claims say the
mouse is faster for most tasks. Read [Dan Luu's look into where those claims
came from](https://danluu.com/keyboard-v-mouse/) for the background. The
honest answer depends on the task. Editing text, and working in a tool you
already know well, tends to favor the keyboard. Some pointing and selection
tasks favor the mouse. For the kind of work I do (editing, moving between
agent panes, git operations, running commands), staying on the keyboard wins
in practice. This is why I favor tools built keyboard-first over tools built
mouse-first.
