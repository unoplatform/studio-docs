---
uid: Uno.PlatformStudio.Skills
---

# Skills & Plugins

Skills in Uno Platform Studio are designed to help you complete end-to-end Uno workflows such as scaffolding, building, validating, and iterating. This page explains the user model: skills are delivered through a plugin, and agents can orchestrate those skills for guided flows.

## What Are Skills?

A skill is a reusable capability the AI can execute, for example:

- generating Uno project code and XAML patterns,
- running checks and validation workflows, or
- automating common developer tasks.

Skills are designed to be composable and versioned so they can evolve without changing your day-to-day prompts.

## Skills vs Plugins

A plugin is the installable unit. The single **`uno-platform`** plugin bundles everything in one install:

- the full Uno skill catalog,
- the `ui-test` sub-agent for automated UI testing, and
- the `uno-platform-docs` MCP server for documentation search.

The key user mental model is:

> You do not install skills one by one. You install the one Uno plugin that bundles them all.

Because Agent Skills is an open standard, the same `uno-platform` plugin installs in both Claude Code and GitHub Copilot CLI.

## Install the Uno Plugin

The plugin is published from the public [`unoplatform/studio`](https://github.com/unoplatform/studio) repository.

> [!NOTE]
> The marketplace handle (`uno-agent-skills`) is being finalized ahead of the public release. Confirm the current handle on the [`unoplatform/studio`](https://github.com/unoplatform/studio) repository before running the commands below.

### Claude Code

```text
/plugin marketplace add unoplatform/studio
/plugin install uno-platform@uno-agent-skills
/skills
/agents
```

To update an installed plugin on demand:

```text
/plugin update uno-platform@uno-agent-skills
```

### GitHub Copilot CLI

The same plugin folder works here — Copilot CLI reads the same Agent Skills format:

```text
copilot plugin marketplace add unoplatform/studio
```

### GitHub Copilot in VS Code

No plugin install is required — Copilot in VS Code reads `SKILL.md` files from disk. You can either install the plugin through Copilot CLI, or copy skill folders into `.github/skills/` (project) or `~/.copilot/skills/` (user).

> [!TIP]
> Image placeholder: Marketplace or repository-based plugin installation flow for Uno.

## Where Skills Live

The skills are published in the public [`unoplatform/studio`](https://github.com/unoplatform/studio) repository in two shapes:

- **Inside the plugin** — when you install `uno-platform`, every skill is bundled and becomes available to your agent automatically. Plugin-managed skills are stored in your agent's managed skills location (for example `~/.claude/skills/`).
- **As standalone skills** — the same skills are also published individually under the repository's top-level `skills/` folder, one folder per skill, for agents that do not support a plugin format.

For agents without plugin support (Cursor, Codex CLI, Gemini CLI, Windsurf, and others), copy the skill folders you want from the repository's `skills/` directory into your agent's skills directory:

```text
git clone https://github.com/unoplatform/studio.git
cp -r studio/skills/uno-mvux-feed-basics ~/.cursor/skills/
```

Each skill is named with the grammar `uno-<category>-<topic>` (for example `uno-mvux-feed-basics` or `uno-material-installation`), so the same slug identifies a skill no matter how it was installed.

## Use Skills Automatically

After plugin installation, skills are generally available implicitly:

1. Start a normal chat or prompt.
2. Describe the Uno task you want to complete.
3. The AI selects and executes applicable skills.

No special command is required for basic usage in most flows.

## Use the Uno Agent for Guided Workflows

For more predictable multi-step execution, activate a custom agent:

```text
/agent <agent-name>
```

With an agent enabled, you typically get:

- guided scaffold-build-test-iterate loops,
- better handling of common failure patterns, and
- clearer automation boundaries for complex tasks.

This mode is recommended for larger features, migrations, and workflow-heavy requests.

The plugin currently ships one sub-agent:

- `ui-test` is a specialized agent for automated Uno UI testing and reporting.

## Skill Categories

All skills ship inside the single `uno-platform` plugin, grouped into these categories:

| Category | Focus area |
| --- | --- |
| `mvux` | MVUX reactive patterns: feeds, state, list state, selection, pagination, messaging |
| `material` | Material Design 3 theming: colors, typography, controls, lightweight styling, migration |
| `navigation` | Navigation Extensions: routes, regions, XAML/code navigation, dialogs, tab shells, troubleshooting |
| `toolkit` | Toolkit controls and helpers: AutoLayout, Card, Drawer, NavigationBar, TabBar, SafeArea, responsiveness |
| `themes` | Semantic design language: semantic palette, generated brushes, state/opacity system |
| `testing` | Automated UI testing and assertions, plus a dedicated UI test agent |
| `xaml` | XAML authoring and error-prevention patterns, including WPF-to-WinUI/Uno conversion |
| `figma` | Translating Figma designs into Uno Platform XAML |

The catalog is sourced from the [`unoplatform/studio`](https://github.com/unoplatform/studio) repository and can evolve over time.
