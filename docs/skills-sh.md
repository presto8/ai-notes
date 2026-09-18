# skills.sh and npx skills

[skills.sh](https://skills.sh) is an open registry for Agent Skills. An Agent
Skill is a portable package of procedural knowledge, defined in a file named
`SKILL.md`. Many agents (OpenAI Codex, OpenCode, Cursor, Hermes, and others)
read a common skills format from a shared `.agents/skills/` directory. Other
agents, such as Claude Code, keep their own flavor and their own directory.
The `skills` CLI hides this difference. It runs with `npx skills`, needs no
separate install, and writes each skill to the correct place for each agent
you use.

## Where each agent keeps its skills

| Agent | Project path | Global path |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.agents/skills/` | (Codex-specific path) |
| Hermes Agent | `.hermes/skills/` | `~/.hermes/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |

## Commands

```sh
# Install a skill package from a GitHub repo shorthand
npx skills add vercel-labs/agent-skills

# Install to a specific agent only, and a specific skill within a package
npx skills add vercel-labs/agent-skills --agent claude-code --skill frontend-design

# Install to every supported agent at once
npx skills add vercel-labs/agent-skills --agent '*'

# Try a skill without installing it: pipes a generated prompt straight into an agent
npx skills use vercel-labs/agent-skills@web-design-guidelines | claude

# List what is installed (project + global)
npx skills list
npx skills ls -g            # global only
npx skills ls -a claude-code -a cursor   # filter by agent

# Search interactively or by keyword
npx skills find
npx skills find typescript

# Keep skills current
npx skills check            # see what has updates available
npx skills update           # update everything
npx skills update my-skill  # update one skill

# Scaffold a new skill
npx skills init my-skill
```

## Other install sources

You can also install from a full GitHub or GitLab URL, a direct path to a
skill subdirectory in a repository, any git URL, or a path on your local disk.
A private repository works too. The CLI uses the authentication that your git
client already has for that remote.
