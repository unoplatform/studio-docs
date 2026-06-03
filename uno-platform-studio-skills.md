---
uid: Uno.PlatformStudio.Skills
---

# Skills & Plugins

Skills in Uno Platform Studio are designed to help you complete end-to-end Uno workflows such as scaffolding, building, validating, and iterating. This page explains the user model: skills are delivered through a plugin, and your AI agent picks the relevant ones automatically as it works on your prompts.

## What Are Skills?

A skill is a reusable capability the AI can execute, for example:

- generating Uno project code and XAML patterns,
- running checks and validation workflows, or
- automating common developer tasks.

Skills are designed to be composable and versioned so they can evolve without changing your day-to-day prompts.

## Skills vs Plugins

A plugin is the installable unit. The single **`uno-platform-studio`** plugin bundles the full Uno skill catalog. The Uno documentation MCP that skills use to search and fetch official docs is provided by the **Uno tooling** (`uno.devserver`) — it is not bundled in the plugin.

The key user mental model is:

> You do not install skills one by one. You install the one Uno plugin that bundles them all.

The same `uno-platform-studio` plugin installs in **Claude Code**, **GitHub Copilot CLI**, **GitHub Copilot in VS Code**, and **OpenAI Codex CLI**. Internally, the plugin folder ships Anthropic-native, GitHub-native, and Codex-native manifests side-by-side, so every plugin agent sees a first-class native install without relying on cross-vendor compatibility shims.

### The Uno documentation MCP

Skills use the Uno documentation MCP (a remote, no-auth server at `https://mcp.platform.uno/v1`) to search and fetch official docs, via two tools:

- `uno_platform_docs_search` — search official documentation.
- `uno_platform_docs_fetch` — fetch a full Uno docs page in markdown.

This MCP is **registered by the Uno tooling** (`uno.devserver`), not bundled in the plugin — most Studio users already have the tooling installed, and bundling it would double-register the same server. claude.ai users have access to the same MCP through the [connector catalog](https://claude.ai/connectors).

## Install the Studio Plugin

The plugin is published from the public [`unoplatform/studio`](https://github.com/unoplatform/studio) repository.

> [!NOTE]
> Slugs are finalized for the public release: marketplace handle `uno-platform`, plugin name `uno-platform-studio`. Confirm against the [`unoplatform/studio`](https://github.com/unoplatform/studio) repository before running the commands below.

### Claude Code

```text
/plugin marketplace add unoplatform/studio
/plugin install uno-platform-studio@uno-platform
/skills
```

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

Copilot in VS Code supports the same plugin format as Copilot CLI. Add `unoplatform/studio` to the `chat.plugins.marketplaces` setting (the defaults `copilot-plugins` and `awesome-copilot` stay registered), then install **uno-platform-studio** from the agent-plugins picker. VS Code discovers the plugin via the GitHub-native `.github/plugin/marketplace.json`.

Prefer file-level installation? You can also drop individual SKILL.md folders into `.github/skills/` (project) or `~/.copilot/skills/` (user) — see [Where Skills Live](#where-skills-live) below.

### OpenAI Codex CLI

Codex CLI installs the plugin via the Codex-native marketplace manifest (`.agents/plugins/marketplace.json`):

```text
codex plugin marketplace add unoplatform/studio
codex plugin install uno-platform-studio@uno-platform
```

> [!TIP]
> Image placeholder: Marketplace or repository-based plugin installation flow for Uno.

## Curated Marketplace Listings

The direct `marketplace add` commands above work the moment the `unoplatform/studio` repository is public. For broader discovery, the plugin will also be listed on the following curated catalogs soon after release:

- **Anthropic Official marketplace** at [claude.com/plugins](https://claude.com/plugins) — Claude Code users browse and install from the official catalog.
- **[`github/awesome-copilot`](https://github.com/github/awesome-copilot)** — community catalog pre-registered as a default marketplace in Copilot CLI and Copilot in VS Code.
- **Codex CLI** — uses a bring-your-own catalog model; no centralized submission-and-approval marketplace exists yet. Users add `unoplatform/studio` directly via `codex plugin marketplace add`.

Curated listings are an additive discovery layer; install commands and behavior are identical regardless of how you discover the plugin.

## Where Skills Live

The skills are published in the public [`unoplatform/studio`](https://github.com/unoplatform/studio) repository in two shapes:

- **Inside the plugin** — when you install `uno-platform-studio`, every skill is bundled and becomes available to your agent automatically. Plugin-managed skills are stored in your agent's managed skills location (for example `~/.claude/skills/`).
- **As standalone skills** — the same skills are also published individually under the repository's top-level `skills/` folder, one folder per skill, for agents that do not support a plugin format.

For agents without plugin support (Cursor, Gemini CLI, Windsurf, Cline, Roo Code, opencode, Aider, and others), copy the skill folders you want from the repository's `skills/` directory into your agent's skills directory:

```text
git clone https://github.com/unoplatform/studio.git
cp -r studio/skills/uno-mvux-feed-basics ~/.cursor/skills/
```

Each skill is named with the grammar `uno-<category>-<topic>` (for example `uno-mvux-feed-basics` or `uno-themes-material`), so the same slug identifies a skill no matter how it was installed.

## Use Skills Automatically

After plugin installation, skills are generally available implicitly:

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
