# OpenCode

[OpenCode](https://opencode.ai) is an open source (MIT) coding agent. It runs
as a terminal UI, a desktop app, and a web app. It reads its provider and model
catalog from [models.dev](https://models.dev) and accepts custom providers, so
the harness does not depend on one model vendor. That is why it is my default:
Claude Code, Codex, and Command Code are tied to the companies that publish
them, while OpenCode lets me change providers and models without leaving the
agent.

## Install

```sh
curl -fsSL https://opencode.ai/v2/install | bash
```

Or use a package manager:

```sh
brew install anomalyco/tap/opencode-v2
npm install -g @opencode/cli
```

## Connect a provider

Run `/connect`, choose a provider, and enter its credentials. Then run
`/models` to pick a model. The catalog is from models.dev. To add a provider
that is not in the catalog, define it in `opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "acme": {
      "name": "Acme",
      "env": ["ACME_API_KEY"],
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://llm.acme.example/v1"
      },
      "models": {
        "qwen3-coder": { "name": "Qwen 3 Coder" }
      }
    }
  }
}
```

This is the part that matters to me: the provider is configuration, not a
different agent. I can move between an Anthropic model, an OpenRouter model, a
local Ollama model, and an OpenCode Go model with `/models`.

## Agents and workflow

OpenCode ships a `build` agent and a read-only `plan` agent, switched with
`Tab`. I use `plan` to read an unfamiliar codebase and agree on an approach,
then `build` to make the changes. `@general` is a subagent for searches and
multi-step side tasks. I run OpenCode in a cmux pane next to the editor, like
the other agents in this repository.

## Skills

Skills are the reason OpenCode is more than a generic harness for me. A skill
is a directory with a `SKILL.md` and, optionally, scripts, references, and
templates. OpenCode advertises each skill to the model by name and description.
The model loads the body with the `skill` tool only when the skill is relevant,
so the full instructions do not sit in every prompt.

### Where it looks

OpenCode discovers skills automatically in these locations:

| Scope | Sources |
| --- | --- |
| Global | `~/.config/opencode/skills` |
| Global compatibility | `~/.claude/skills`, `~/.agents/skills` |
| Project | `.opencode/skills` |
| Project compatibility | `.claude/skills`, `.agents/skills` |

For project sources, it searches from the current directory up to the project
root. A skill is either a root-level `*.md` file or a directory containing
`SKILL.md`; the directory form is recommended because it gives supporting files
a private base directory.

The compatibility rows are the useful part. Skills I installed for Claude Code
or for the shared `.agents` format show up in OpenCode without a second install.
When the source changes, OpenCode reloads the skill.

### Extra sources and remote catalogs

Add more directories or HTTP catalogs with the `skills` array in
`opencode.jsonc`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": [
    "./team-skills",
    "~/shared/opencode-skills",
    "https://example.com/opencode/skills/"
  ]
}
```

An HTTP catalog is a base URL with an `index.json`:

```json
{
  "skills": [
    {
      "name": "git-release",
      "version": "3",
      "files": ["git-release.md", "references/release-policy.md"]
    }
  ]
}
```

OpenCode downloads each file from `<base-url>/git-release/<file>` and refreshes
its cache when `version` changes. That is the closest thing to an agent
updating its own skill set: point it at a catalog, publish a new version, and
the next run uses the new skill. It still loads skills on demand and does not
rewrite its own instructions.

### Permissions

Skills are selected by ID. Use the `skill` permission action to allow, ask, or
deny them, either globally or per agent:

```jsonc
{
  "permissions": [
    { "action": "skill", "resource": "*", "effect": "allow" },
    { "action": "skill", "resource": "experimental-*", "effect": "ask" }
  ]
}
```

## Notes

- The skill ID comes from the path, not the frontmatter name. A skill at
  `.opencode/skills/release/SKILL.md` has the ID `release`.
- `metadata.opencode/autoinvoke: false` hides a skill from the model's list but
  keeps it loadable by ID.
- A skill without a `description` is not advertised. Put the trigger conditions
  in the description, not only in the body.
