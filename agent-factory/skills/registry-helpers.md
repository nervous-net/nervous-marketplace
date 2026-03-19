<!-- ABOUTME: Shared instructions for reading and writing the central agent registry -->
<!-- ABOUTME: Used by /create-agents, /add-agent, /upgrade-agent, and /hire-agent skills -->

# Registry Helpers

Shared instructions for reading and writing `~/.claude/agents/registry.json`.

## Schema

```json
{
  "projects": [
    {
      "path": "/absolute/path/to/project",
      "name": "Human-Readable Project Name",
      "registered": "YYYY-MM-DD",
      "agents": [
        {
          "slug": "agent-slug",
          "role": "One-sentence role description",
          "archetype": "One-line archetype",
          "keywords": ["keyword1", "keyword2"]
        }
      ]
    }
  ],
  "global_agents": [
    {
      "slug": "agent-slug",
      "path": "~/.claude/agents/agent-slug",
      "role": "One-sentence role description",
      "archetype": "One-line archetype",
      "keywords": ["keyword1", "keyword2"]
    }
  ]
}
```

## Reading the Registry

1. Check if `~/.claude/agents/registry.json` exists (use Glob).
2. If it exists, read and parse it. If parsing fails (malformed JSON), start with an empty registry: `{"projects": [], "global_agents": []}`.
3. If it doesn't exist, start with an empty registry.

## Writing a Project Entry

1. Read the registry (or start empty).
2. Find the project entry by matching `path` against the current project's absolute path.
3. If found, update the `agents` array. If not found, create a new project entry.
4. Write the file back to `~/.claude/agents/registry.json`. Create the `~/.claude/agents/` directory if it doesn't exist.

## Writing a Global Agent Entry

1. Read the registry (or start empty).
2. Check if the agent slug already exists in `global_agents`. If so, update it. If not, append.
3. If the agent was previously in a project entry, remove it from that project's `agents` array.
4. Write the file back.

## Generating Keywords

Extract 4-8 single-word terms from:
- The agent's role description (nouns and verbs that describe what they do)
- The expertise section of the character sheet (domain terms)

Keywords should be lowercase, singular, and specific enough to distinguish this agent from others. Examples: "security", "review", "api", "testing", "docs", "ux", "performance", "accessibility".

## Validating Entries (Stale Cleanup)

When presenting registry contents to a user (only in `/hire-agent`):
1. For each project entry, check if the path exists (use Bash: `ls [path]/agents/team.json`).
2. Remove entries where the path no longer exists.
3. For global agents, check if the path exists.
4. Remove stale global entries.
5. Write the cleaned registry back.
