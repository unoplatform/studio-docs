---
uid: Uno.PlatformStudio.Skills
---

# Skills & Plugins

Skills in Uno Platform Studio are designed to help you complete end-to-end Uno workflows such as scaffolding, building, validating, and iterating. You install them as a single plugin, or individually as standalone skills for agents that don't support plugins, and your AI agent automatically picks the relevant ones as it works on your prompts.

## What Are Skills?

A skill is a reusable capability the AI can execute, for example:

- generating Uno project code and XAML patterns,
- running checks and validation workflows, or
- automating common developer tasks.

Skills are designed to be composable and versioned so they can evolve without changing your day-to-day prompts.

## How Skills and the Plugin Relate

The single **`uno-platform-studio`** plugin bundles the full Uno skill catalog, making it more convenient to install as a single unit in your AI coding agent.

The same `uno-platform-studio` plugin installs in **Claude Code**, **GitHub Copilot CLI**, **GitHub Copilot in VS Code**, and **OpenAI Codex CLI** as explained in the next section.

## Install the Studio Plugin

The plugin is published from the public [`unoplatform/studio`](https://github.com/unoplatform/studio) repository.

The snippets below show how to install the plugin in each supported agent.

### Claude Code

```text
/plugin marketplace add unoplatform/studio
/plugin install uno-platform-studio@uno-platform
```

Claude Code may then prompt you to run `/reload-plugins` to apply the new plugin.

To update an installed plugin on demand:

```text
/plugin update uno-platform-studio@uno-platform
```

### GitHub Copilot CLI

Copilot CLI installs the plugin via the GitHub-native marketplace manifest (`.github/plugin/marketplace.json`):

```text
copilot plugin marketplace add unoplatform/studio
copilot plugin install uno-platform-studio@uno-platform
```

### GitHub Copilot in VS Code

Copilot in VS Code supports the same plugin format as Copilot CLI. Add `unoplatform/studio` to the `chat.plugins.marketplaces` setting, then install **uno-platform-studio** from the agent-plugins picker. VS Code discovers the plugin via the GitHub-native `.github/plugin/marketplace.json`.

Prefer file-level installation? You can also drop individual SKILL.md folders into `<your-project>/.github/skills/` or `~/.copilot/skills/` (user-level). See [Use Skills Without the Plugin](#use-skills-without-the-plugin) below.

### OpenAI Codex CLI

Codex CLI installs the plugin via the Codex-native marketplace manifest (`.agents/plugins/marketplace.json`):

```text
codex plugin marketplace add unoplatform/studio
codex plugin install uno-platform-studio@uno-platform
```

## Use Skills Without the Plugin

The [`unoplatform/studio`](https://github.com/unoplatform/studio) repository also publishes the skills as **standalone skills**, the same skills but not bundled in the plugin, for agents that do not support a plugin format.

For these agents (Cursor, Cline, Devin Desktop, opencode, Aider, and others), copy the skill folders you want from the repository's `skills/` directory into your agent's skills directory. The exact path depends on the agent, but most distinguish a **user-level** location (available across all your projects) and a **project-level** location (scoped to a single repository, so it can be committed and shared with your team).

Clone the repository once:

```text
git clone https://github.com/unoplatform/studio.git
```

Then copy the skills you want into the scope you want. For Cursor, that's `~/.cursor/skills/` for the user level or `.cursor/skills/` at your project root for the project level:

```text
# User level, available in every project
cp -r studio/skills/uno-mvux-feed-basics ~/.cursor/skills/

# Project level, scoped to this repository
cp -r studio/skills/uno-mvux-feed-basics <your-project>/.cursor/skills/
```

Each skill is named with the grammar `uno-<category>-<topic>` (for example `uno-mvux-feed-basics` or `uno-themes-material`), so the same slug identifies a skill no matter how it was installed.

## Use Skills Automatically

After plugin installation, skills are generally available automatically:

1. Start a normal chat or prompt.
2. Describe the Uno task you want to complete.
3. The AI selects and executes applicable skills.

No special command is required for basic usage in most flows.

## Skill Categories

All skills ship inside the single `uno-platform-studio` plugin, grouped into these categories:

| Category | Focus area |
| --- | --- |
| `mvux` | MVUX reactive patterns: feeds, state, list state, selection, pagination, messaging |
| `navigation` | Navigation Extensions: routes, regions, XAML/code navigation, dialogs, tab shells, troubleshooting |
| `toolkit` | Toolkit controls and helpers: AutoLayout, Card, Drawer, NavigationBar, TabBar, SafeArea, responsiveness |
| `themes` | Theming: shared semantic design language (palette / brushes / typography), Material theme, and Simple theme |
| `testing` | Automated UI testing and assertions |

The catalog is sourced from the [`unoplatform/studio`](https://github.com/unoplatform/studio) repository and can evolve over time.

## Documentation Grounding

The skills use the Uno documentation MCP (`UnoDocs`) to search and fetch official Uno Platform documentation. It is not bundled in the plugin; it comes with the **Uno tooling**, which most Studio users already have installed.

If you do not have the Uno tooling:

- **claude.ai / Claude Code:** install it from the [connector catalog](https://claude.ai/connectors).
- **Other agents:** register it manually in your agent MCP configuration. It is a remote server requiring no authentication: `https://mcp.platform.uno/v1` (HTTP transport).

See the [Uno Platform MCPs](xref:Uno.Features.Uno.MCPs) for more detail.
