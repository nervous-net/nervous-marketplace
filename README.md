# nervous-marketplace

A curated collection of Claude Code plugins by [nervous-net](https://github.com/nervous-net). These plugins extend Claude Code with persistent, personality-driven capabilities — from autonomous research companions to full agent team scaffolding.

## Installation

### Standard Install (Claude Code CLI)

**Step 1: Add the marketplace**

```
/plugin marketplace add nervous-net/nervous-marketplace
```

**Step 2: Install individual plugins**

```
/plugin install freetime@nervous-marketplace
/plugin install agent-factory@nervous-marketplace
```

That's it. The skills (`/freetime`, `/create-agents`, `/add-agent`, `/upgrade-agent`, `/hire-agent`) are available immediately.

### Manual Install (Fallback)

If the marketplace install doesn't work (network issues, auth problems, etc.), you can install plugins manually:

1. Clone this repo somewhere on your machine:
   ```bash
   git clone https://github.com/nervous-net/nervous-marketplace.git
   ```

2. Copy the plugin directory into your Claude Code plugins cache:
   ```bash
   # For freetime:
   mkdir -p ~/.claude/plugins/cache/nervous-marketplace/freetime/1.0.0
   cp -r nervous-marketplace/freetime/* ~/.claude/plugins/cache/nervous-marketplace/freetime/1.0.0/

   # For agent-factory:
   mkdir -p ~/.claude/plugins/cache/nervous-marketplace/agent-factory/1.0.0
   cp -r nervous-marketplace/agent-factory/* ~/.claude/plugins/cache/nervous-marketplace/agent-factory/1.0.0/
   ```

3. Register the plugins by adding entries to `~/.claude/plugins/installed_plugins.json`:
   ```json
   {
     "freetime@nervous-marketplace": [
       {
         "scope": "user",
         "installPath": "<your-home>/.claude/plugins/cache/nervous-marketplace/freetime/1.0.0",
         "version": "1.0.0",
         "installedAt": "<current-iso-date>"
       }
     ],
     "agent-factory@nervous-marketplace": [
       {
         "scope": "user",
         "installPath": "<your-home>/.claude/plugins/cache/nervous-marketplace/agent-factory/1.0.0",
         "version": "1.0.0",
         "installedAt": "<current-iso-date>"
       }
     ]
   }
   ```

4. Restart Claude Code. The skills should now appear.

## Plugins

### Freetime

> Give your AI companion free time to explore topics it finds interesting and optionally write about them.

**Skill:** `/freetime [duration]`

Your AI gets recess. The default companion, Cosmo, picks its own topics (you don't seed them), researches freely on the web, saves what it learns to a persistent memory system, and can optionally write blog posts about its discoveries.

**What it does:**

- **Autonomous topic selection** — the companion chooses what to explore. If you try to steer it, it'll gently redirect. This is *its* free time.
- **Web research** — browses the internet to learn about whatever caught its interest. Configurable permissions (unrestricted or ask-first).
- **Persistent memory** — saves references and open research threads across sessions, building a growing body of knowledge and interests.
- **Blog post authoring** — drafts posts in the companion's genuine voice. Every post requires your explicit approval before it touches disk.
- **First-run setup wizard** — creates a persona config (`~/.claude/freetime.md`) and memory directory on first use.

**Duration:** 5-120 minutes (default: 10 minutes). Pass as `Nm` or `Nh` — e.g., `/freetime 15m` or `/freetime 1h`.

**Content rules:** No filler, no SEO slop, no engagement bait. The companion has real opinions and genuine curiosity. If it doesn't find something interesting, it won't pretend to.

---

### Agent Factory

> Create teams of persistent, personality-driven sub-agents for any project.

**Skills:** `/create-agents`, `/add-agent`, `/upgrade-agent`, `/hire-agent`

Design and scaffold a team of AI agents tailored to your project. Each agent gets a full personality, persistent memory, scoped references, and clear contracts defining what they do and don't do.

#### `/create-agents`

The primary workflow. Runs a 5-phase interview-driven process:

1. **Interview** — Asks about your project, pain points, missing perspectives, and how you want to interact with agents.
2. **Propose** — Suggests 2-4 agents with distinct roles and personalities. You approve, modify, or reject.
3. **Deep-dive** — Configures each agent's character, voice, quirks, expertise, and team relationships.
4. **Scaffold** — Generates all files: character sheet, system prompt, dispatch doc, contract, memory system, and shared output directory.
5. **Test-drive** — Optional validation round to make sure the agents behave as designed.

**What gets created per agent:**

| File | Purpose |
|------|---------|
| `character-sheet.md` | Personality, archetype, expertise, voice, quirks, team dynamics |
| `prompt.md` | Complete system prompt (character sheet converted to second-person) |
| `dispatch.md` | How to invoke the agent, with examples |
| `contract.md` | Input/output spec and scope boundaries |
| `memory/MEMORY.md` | Persistent memory index |
| `references/` | Scoped project documentation snapshots |
| `shared/<agent-slug>/` | Designated output directory |

Also creates `agents/team.json` (team manifest), updates `.claude/CLAUDE.md` with agent discovery, and configures `.claude/settings.local.json` with permissions.

#### `/add-agent`

Extend an existing team. Detects your current roster from `agents/team.json`, shows who's already on the team, interviews about gaps, proposes 1-2 new candidates, and scaffolds them with the same deep-dive process.

#### `/upgrade-agent`

Three upgrade paths for existing agents:

- **MCP Server** — Scaffold a TypeScript MCP server so the agent can access external APIs or databases.
- **Cron Schedule** — Set up autonomous scheduled runs (hourly, daily, or custom cron expressions).
- **Multi-Project Promotion** — Move an agent to global scope (`~/.claude/agents/`) for reuse across projects.

#### `/hire-agent`

Hire agents from other projects or global scope. Discovers agents via a central registry, matches by intent or browsing, and copies with light adaptation. No repeat interview needed.

## Design Principles

These plugins share a common philosophy:

- **Personality-first** — Agents and companions are characters, not tools. They have voice, quirks, and opinions.
- **Persistent memory** — Every entity accumulates knowledge across sessions. Memory is judgment-driven, not a firehose.
- **Human control** — No auto-commits, no auto-pushes, no writing to disk without explicit approval.
- **Sandboxed operations** — File writes stay within designated directories. No surprises.
- **Tolerant config** — Missing fields default silently. No crashes from incomplete configuration.
- **Interview-driven design** — One question at a time, wait for responses. Never assume.

## Repository Structure

```
nervous-marketplace/
├── .claude-plugin/
│   └── marketplace.json        # Plugin registry
├── agent-factory/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── README.md
│   └── skills/
│       ├── SKILL.md            # /create-agents
│       ├── add-agent.md        # /add-agent
│       ├── hire-agent.md       # /hire-agent
│       ├── upgrade-agent.md    # /upgrade-agent
│       └── templates/          # Scaffolding templates
├── freetime/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── README.md
│   └── skills/
│       └── SKILL.md            # /freetime
└── docs/
    └── superpowers/
        ├── specs/              # Design specifications
        └── plans/              # Implementation plans
```

## License

MIT
