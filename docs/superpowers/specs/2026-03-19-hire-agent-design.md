# `/hire-agent` Design Spec

Bring agents from other projects into the current project without repeating the onboarding interview.

## Problem

The agent-factory creates personality-driven agents through a detailed interview process. When you want the same kind of agent in a different project, you repeat the entire interview — even though the agent's personality, communication style, and expertise are already defined somewhere else. The `/upgrade-agent` promotion path solves multi-project *access* for a single agent instance, but doesn't solve the case where you want an independent copy tailored to a new codebase.

## Solution

A new skill, `/hire-agent`, that discovers agents across projects and global scope, lets the user pick one (by intent or browsing), copies it with light adaptation, and drops it into the current project ready to work. The copy is independent — its own memories, its own references — but tracks where it came from.

## Registry System

### File: `~/.claude/agents/registry.json`

Central index of all projects with agent teams and all globally promoted agents. Auto-populated by existing skills.

```json
{
  "projects": [
    {
      "path": "/Users/nervous/Dev/my-saas",
      "name": "My SaaS",
      "registered": "2026-03-19",
      "agents": [
        {
          "slug": "the-skeptic",
          "role": "Code reviewer who questions every abstraction",
          "archetype": "demanding but fair critic",
          "keywords": ["code review", "quality", "refactoring", "architecture"]
        }
      ]
    }
  ],
  "global_agents": [
    {
      "slug": "lector",
      "path": "~/.claude/agents/lector",
      "role": "Documentation reviewer with an eye for clarity",
      "archetype": "meticulous editor",
      "keywords": ["docs", "writing", "documentation", "readme"]
    }
  ]
}
```

### How it stays current

- `/create-agents` writes/updates the project entry after scaffolding
- `/add-agent` appends to the project's agents array
- `/upgrade-agent` (promotion path) moves an agent from a project entry to `global_agents`
- `/hire-agent` validates paths before presenting — silently drops stale entries and rewrites

### Keywords

Each agent gets a `keywords` array derived from its role and character sheet during registration. These power intent-matching during discovery.

## `/hire-agent` Flow

### Phase 1: Discovery

Two entry points, tried in order:

1. **Intent-based** — Ask: "What kind of help are you looking for in this project?" Free-text answer gets matched against agent keywords and roles in the registry. Present ranked matches across all projects and global agents.

2. **Browse** — If no good matches or user prefers to browse: list all registered projects and their agents, grouped by project. User picks.

Global agents (`~/.claude/agents/`) always surface first since they've been promoted as broadly useful.

### Phase 2: Selection & Preview

Once the user picks an agent:

- Show the agent's character sheet (personality, communication style, quirks)
- Show the agent's role and contract summary
- Show where it came from (project name, path)
- Ask: "Want to hire this agent?" (yes / pick a different one)

### Phase 3: Light Adaptation (2-3 questions)

1. "What files or directories should this agent focus on in this project?" — scan project, offer suggestions based on the agent's original scope
2. "Where should output go?" — default: `agents/shared/[slug]/`, confirm or change
3. If agent's original references don't exist in the new project, ask which local docs to substitute

### Phase 4: Scaffold

- Copy agent files (character sheet, prompt, dispatch, contract) into `agents/[slug]/`
- Create `origin.json` with lineage metadata
- Swap references for new project's docs
- Create clean `memory/MEMORY.md`
- Update `agents/team.json` (add agent entry with `"source": "hired"`)
- Regenerate CLAUDE.md discovery block
- Update registry with new project's agent list (if project wasn't registered yet)

## Lineage Tracking

### File: `agents/[slug]/origin.json`

Each hired agent records where it came from:

```json
{
  "hired_from": {
    "project": "/Users/nervous/Dev/my-saas",
    "project_name": "My SaaS",
    "agent_slug": "the-skeptic",
    "date": "2026-03-19"
  },
  "original_global": false
}
```

For agents hired from global scope:

```json
{
  "hired_from": {
    "path": "~/.claude/agents/lector",
    "agent_slug": "lector",
    "date": "2026-03-19"
  },
  "original_global": true
}
```

### What lineage enables

- See where an agent came from at a glance
- Future potential: sync personality updates from the original
- Distinguishes hired from created agents via `team.json` source field

### What lineage does NOT do

- No automatic syncing — the copy is independent from the moment of hire
- No dependency on the source project existing

## `team.json` Changes

Hired agents use a distinct entry format:

```json
{
  "name": "the-skeptic",
  "role": "Code reviewer who questions every abstraction",
  "scope": "project-local",
  "source": "hired",
  "hired_from": {
    "project": "/Users/nervous/Dev/my-saas",
    "agent_slug": "the-skeptic"
  },
  "pipeline_position": null,
  "dependencies": []
}
```

Created agents have `"source": "created"` or no `source` field. Existing `team.json` files without `source` fields continue to work — absent means `"created"`.

## Changes to Existing Skills

### `/create-agents` (SKILL.md)

After Phase 4 (scaffold), add: write/update project entry in `~/.claude/agents/registry.json`. Generate keywords for each agent from role and character sheet.

### `/add-agent` (add-agent.md)

After scaffolding, append new agent to the project's registry entry. Generate keywords.

### `/upgrade-agent` (upgrade-agent.md)

On promotion: move agent from project's `agents` array to `global_agents` in registry. Non-promotion upgrades (MCP, cron) require no registry changes.

### Registry write logic (shared)

- Read `~/.claude/agents/registry.json` (create if missing)
- Find or create project entry by path
- Update agent list
- Write back
- Tolerant of missing/malformed file — recreate from scratch if corrupted

## Design Decisions

| Decision | Choice | Reasoning |
|----------|--------|-----------|
| Copy vs. reference | Copy with lineage | Isolation (project-specific memories/references) with origin trail for future sync potential |
| Discovery | Registry + intent matching + browse | Auto-populating registry grows over time; intent matching makes it smart; browse is the fallback |
| Registry location | `~/.claude/agents/registry.json` | Natural home alongside global agents |
| Adaptation depth | Light (2-3 questions) | Skip the full interview — that's the whole point — but scope the agent to the new project |
| Memory on hire | Clean slate | Memories are project-contextual; carrying them over adds noise |
| Skill name | `/hire-agent` | Clear intent, parallel to `/add-agent` |
