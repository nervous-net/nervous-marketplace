# `/hire-agent` Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `/hire-agent` skill that discovers agents across projects via a central registry, lets users pick by intent or browsing, and copies agents into the current project with light adaptation.

**Architecture:** New skill file `hire-agent.md` alongside existing skills. Registry at `~/.claude/agents/registry.json` auto-populated by existing skills. Hired agents get copied files with lineage tracking via `origin.json`, clean memory, and adapted references.

**Tech Stack:** Markdown skill files (no compiled code). JSON for registry and config.

---

### Task 1: Create the Registry Schema and Write Helper Instructions

**Files:**
- Create: `agent-factory/skills/registry-helpers.md`

This task creates a shared reference document that all four skills (create, add, upgrade, hire) will use when reading/writing the registry. This avoids duplicating registry logic across skill files.

- [ ] **Step 1: Write the registry helpers document**

Create `agent-factory/skills/registry-helpers.md` with the registry JSON schema, read/write instructions, keyword generation guidance, and stale entry cleanup logic.

```markdown
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
```

- [ ] **Step 2: Commit**

```bash
git add agent-factory/skills/registry-helpers.md
git commit -m "docs: add registry helpers reference for agent-factory skills"
```

---

### Task 2: Update `/create-agents` to Populate the Registry

**Files:**
- Modify: `agent-factory/skills/SKILL.md` (add registry write step after Phase 4)

- [ ] **Step 1: Read the current SKILL.md**

Read `agent-factory/skills/SKILL.md` to confirm the exact location to insert the registry step.

- [ ] **Step 2: Add registry write step to Phase 4**

Insert a new `### Registry Update` subsection between the `### Summary` subsection at the end of Phase 4 and `## Phase 5: Test Drive (Optional)`:

```markdown
### Registry Update

After scaffolding, update the central agent registry so other projects can discover these agents via `/hire-agent`.

1. Read the registry helpers at `<skill-base-dir>/registry-helpers.md` for schema and write instructions.
2. Read the registry at `~/.claude/agents/registry.json` (create with empty structure if missing).
3. For each agent in the team, generate keywords from their role and character sheet expertise section (4-8 lowercase terms).
4. Find or create a project entry for the current project path.
5. Set the project entry's `name` from the project name detected in Phase 1, `registered` to today's date, and `agents` to the full list of agents with slug, role, archetype, and keywords.
6. Write the registry back to `~/.claude/agents/registry.json`.
```

- [ ] **Step 3: Verify the edit is correct**

Read `agent-factory/skills/SKILL.md` and confirm the new section sits between the Phase 4 Summary and Phase 5: Test Drive.

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/SKILL.md
git commit -m "feat: populate agent registry from /create-agents"
```

---

### Task 3: Update `/add-agent` to Populate the Registry

**Files:**
- Modify: `agent-factory/skills/add-agent.md` (add registry step after scaffold)

- [ ] **Step 1: Read the current add-agent.md**

Read `agent-factory/skills/add-agent.md` to confirm exact insertion point.

- [ ] **Step 2: Add registry update step after scaffold section**

After the scaffold section (step 6: "Regenerate the CLAUDE.md discovery block") and before "Present summary of what was created", add:

```markdown
7. **Update registry** — Read `<skill-base-dir>/registry-helpers.md` for instructions. Read the registry at `~/.claude/agents/registry.json`. Generate keywords for the new agent (4-8 lowercase terms from role and expertise). Find the project entry by path (or create one). Append the new agent to the project's `agents` array. Write the registry back.
```

- [ ] **Step 3: Verify the edit is correct**

Read `agent-factory/skills/add-agent.md` and confirm the new step is numbered correctly and sits in the right place.

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/add-agent.md
git commit -m "feat: populate agent registry from /add-agent"
```

---

### Task 4: Update `/upgrade-agent` to Update the Registry on Promotion

**Files:**
- Modify: `agent-factory/skills/upgrade-agent.md` (add registry step to Multi-Project Promotion)

- [ ] **Step 1: Read the current upgrade-agent.md**

Read `agent-factory/skills/upgrade-agent.md` to confirm exact insertion point in the Multi-Project Promotion section.

- [ ] **Step 2: Add registry update step to promotion flow**

After step 8 ("Regenerate the CLAUDE.md discovery block") in the Multi-Project Promotion section, add:

```markdown
9. **Update registry** — Read `<skill-base-dir>/registry-helpers.md` for instructions. Read the registry at `~/.claude/agents/registry.json`. Generate keywords for the agent (4-8 lowercase terms from role and expertise). Move the agent from the source project's `agents` array to the `global_agents` array with its global path. Write the registry back.
```

Renumber the existing step 9 ("Update the agent's `dispatch.md`...") to step 10, and the existing step 10 ("Tell the user...") to step 11.

- [ ] **Step 3: Verify the edit is correct**

Read `agent-factory/skills/upgrade-agent.md` and confirm step numbering is correct and the registry step sits after CLAUDE.md regeneration.

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/upgrade-agent.md
git commit -m "feat: update agent registry on promotion via /upgrade-agent"
```

---

### Task 5: Create the `/hire-agent` Skill

**Files:**
- Create: `agent-factory/skills/hire-agent.md`

This is the main deliverable. The skill file defines the complete hire flow: discovery, selection, adaptation, and scaffolding.

- [ ] **Step 1: Write the skill file**

Create `agent-factory/skills/hire-agent.md`:

```markdown
---
name: hire-agent
description: Hire an agent from another project or global scope into the current project. Discovers agents via a central registry, matches by intent or browsing, and copies with light adaptation — no repeat interview needed.
---

# Hire Agent

Bring an existing agent from another project into this one. Skip the onboarding interview — the agent's personality, expertise, and communication style carry over. You adapt their scope and references to fit the new project.

## Prerequisites

1. Look for `agents/team.json` in the current project (use Glob tool).
2. If **NOT found** → "No agent team found in this project. Run `/create-agents` first to set up a team, or I can create a minimal team manifest to hold the hired agent." If the user wants to proceed without a full team, create a minimal `agents/team.json`:
   ```json
   {
     "name": "Hired Agents",
     "created": "[today's date]",
     "interaction_model": "parallel",
     "agents": [],
     "pipeline_order": []
   }
   ```
3. If **found** → read it (tolerantly). Also read `agents/team.json` to understand existing team composition.

## Phase 1: Discovery

Read the registry helpers at `<skill-base-dir>/registry-helpers.md` for schema details.

### Step 1: Load and Validate Registry

1. Read `~/.claude/agents/registry.json`. If it doesn't exist or is empty:
   "No agents registered yet. You can:
   - **(a)** Point me at a project directory and I'll scan it for agents
   - **(b)** Run `/create-agents` in another project first to populate the registry"
   If the user provides a path, scan `[path]/agents/team.json` and each agent's `character-sheet.md` to build a temporary candidate list. Proceed to Phase 2.

2. Validate entries — check that project paths and global agent paths still exist. Silently remove stale entries and rewrite the registry.

### Step 2: Intent Matching

Ask: "What kind of help are you looking for in this project?"

*Wait for response.*

Match the user's answer against agent keywords and role descriptions in the registry. Use semantic understanding — "someone to catch bugs" should match agents with keywords like "testing", "review", "quality". Present the top matches (up to 5), grouped by source:

"Here's who I found:

**Global agents:**
- **[name]** — [role] *(available everywhere)*

**From [Project Name] (`[path]`):**
- **[name]** — [role]
- **[name]** — [role]

**From [Other Project] (`[path]`):**
- **[name]** — [role]

Want to hire one of these, or would you prefer to browse all available agents?"

*Wait for response.*

If the user wants to browse, list all registered projects and their agents grouped by project. Let them pick.

If no matches are found: "I didn't find any agents matching that description. Want to browse all available agents, or point me at a specific project directory?"

## Phase 2: Selection & Preview

Once the user indicates interest in an agent:

1. Read the agent's `character-sheet.md` from its source location (project path + `agents/[slug]/character-sheet.md`, or global path + `character-sheet.md`).
2. Read the agent's `contract.md` from the same location.
3. Present:

"Here's **[agent name]**:

**Personality:** [summary from character sheet — identity, voice, key quirks]
**Expertise:** [from character sheet]
**What they do:** [from contract — inputs, outputs, scope]
**Where they're from:** [project name] (`[path]`)

Want to hire them? Or pick a different agent?"

*Wait for response.*

## Phase 3: Light Adaptation

Three quick questions to scope the agent to this project. **One at a time.**

### Question 1: Focus Areas

Scan the current project for directories and key files. Present them alongside what the agent originally focused on:

"In their previous project, **[agent name]** focused on: [original input paths from contract].

This project has:
- `src/` — [brief description if detectable]
- `tests/` — [brief description]
- `docs/` — [brief description]
- [other notable directories]

What should they focus on here?"

*Wait for response.*

### Question 2: Output Location

"Output goes to `agents/shared/[agent-slug]/` by default. Want to change that?"

*Wait for response.*

### Question 3: References

Scan the project for documentation files (same as `/create-agents` Phase 3, Step 4). Present what you found:

"I found these docs in your project:
- [ ] `README.md` — [brief description]
- [ ] `CLAUDE.md` — [brief description]
- [ ] [other docs]

Which should **[agent name]** have access to? I'll copy them into the agent's references directory."

*Wait for response.*

## Phase 4: Scaffold

Generate all files in a single batch.

### Agent Directory

Create:
```
agents/[agent-slug]/
agents/[agent-slug]/references/
agents/[agent-slug]/memory/
agents/shared/[agent-slug]/
```

### Copy and Adapt Agent Files

1. **Character sheet** → `agents/[agent-slug]/character-sheet.md`
   - Copy from source. If the agent is joining an existing team, update the "Relationship to Team" section to reference the current project's agents by name.

2. **Prompt file** → `agents/[agent-slug]/prompt.md`
   - Copy from source.
   - Update all file paths to reflect the new project location.
   - Update the "You read from" section with the new focus areas from Phase 3.
   - Update the "You produce" section with the new output path.
   - Update the memory path to `agents/[agent-slug]/memory/`.
   - Update the knowledge section to reference the new project's docs.

3. **Dispatch doc** → `agents/[agent-slug]/dispatch.md`
   - Copy from source.
   - Update all paths (prompt path, output path, example paths) to the new project location.

4. **Contract** → `agents/[agent-slug]/contract.md`
   - Copy from source.
   - Update input paths, output path, and required context for the new project.

5. **Memory** → `agents/[agent-slug]/memory/MEMORY.md`
   - Create fresh: `# [Agent Name] Memory\n\n(No memories yet.)`
   - Do NOT copy memories from source.

6. **References** → Copy selected project docs into `agents/[agent-slug]/references/`

7. **Lineage** → `agents/[agent-slug]/origin.json`
   - Create with source tracking:
   ```json
   {
     "hired_from": {
       "project": "[source project absolute path]",
       "project_name": "[source project name]",
       "agent_slug": "[original slug]",
       "date": "[today's date YYYY-MM-DD]"
     },
     "original_global": false
   }
   ```
   If hired from a global agent, use this format instead:
   ```json
   {
     "hired_from": {
       "path": "~/.claude/agents/[agent-slug]",
       "agent_slug": "[slug]",
       "date": "[today's date YYYY-MM-DD]"
     },
     "original_global": true
   }
   ```

### Update Team Manifest

Read `agents/team.json`, parse it, add the new agent entry:

```json
{
  "name": "[agent-slug]",
  "role": "[role from source]",
  "scope": "project-local",
  "source": "hired",
  "hired_from": {
    "project": "[source project path]",
    "agent_slug": "[original slug]"
  },
  "pipeline_position": null,
  "dependencies": []
}
```

Write the file back.

### Update CLAUDE.md Discovery Block

Regenerate the block between `<!-- agent-factory:start -->` and `<!-- agent-factory:end -->` markers with all agents listed (same logic as `/add-agent`).

### Update Registry

If this project doesn't have a registry entry yet, add one. Append the hired agent to the project's `agents` array in the registry with generated keywords.

### Summary

"**[agent name]** has been hired!

`agents/[slug]/`
  - `character-sheet.md` — personality (carried over)
  - `prompt.md` — system prompt (adapted for this project)
  - `dispatch.md` — how to invoke
  - `contract.md` — input/output contract (adapted)
  - `origin.json` — hired from [source project name]
  - `references/` — [N] reference docs
  - `memory/` — clean slate

`agents/team.json` — updated
`.claude/CLAUDE.md` — updated with agent discovery block"

## Test Drive (Optional)

"Would you like to test drive **[agent name]** in this project?"

If yes: pick a small, real task appropriate to the agent's role using files from the new project. Dispatch via the Agent tool using the agent's `prompt.md`. Write output to `agents/shared/[agent-slug]/test-drive.md`. Present the output, ask for feedback, iterate if needed.

If no: "They're ready whenever you are."

## Error Handling

- **No registry and no path provided:** Suggest running `/create-agents` in other projects first, or provide a project path to scan directly.
- **Agent slug collision:** "There's already an agent called [name] in this project. Want to rename the incoming agent, or replace the existing one?" *Wait for response.*
- **Source agent files missing:** "I can see [agent name] in the registry but their files at [path] are missing. They may have been deleted. Want to pick a different agent?" Remove the stale entry from the registry.
- **No team.json in current project:** Offer to create a minimal manifest (see Prerequisites).
- **Source project has no agents/ directory:** Remove from registry, inform user, continue with remaining options.

## Rules

- **One question at a time.** Never ask multiple questions in a single message.
- **Wait for responses.** Never skip ahead.
- **Never auto-commit.** Let the user decide when to commit.
- **Never overwrite without asking.** If a file exists, ask before replacing.
- **Read config tolerantly.** Missing fields use sensible defaults.
- **Personality is sacred.** The character sheet and core personality carry over exactly. Only paths, references, and scope get adapted.
- **Clean memory.** Never copy memories from the source agent. Fresh start.
- **Lineage is mandatory.** Every hired agent gets an `origin.json`. No exceptions.
```

- [ ] **Step 2: Verify the skill file reads correctly**

Read `agent-factory/skills/hire-agent.md` end to end to check for formatting issues, broken markdown, or placeholder text.

- [ ] **Step 3: Commit**

```bash
git add agent-factory/skills/hire-agent.md
git commit -m "feat: implement /hire-agent skill for cross-project agent hiring"
```

---

### Task 6: Update Plugin Metadata

**Files:**
- Modify: `agent-factory/.claude-plugin/plugin.json`

- [ ] **Step 1: Read the current plugin.json**

Read `agent-factory/.claude-plugin/plugin.json`.

- [ ] **Step 2: Add hire-agent to the description and keywords**

Update the `description` field to mention hiring agents from other projects. Add `"hire"` and `"cross-project"` to the `keywords` array.

Updated description: `"Create teams of persistent, personality-driven sub-agents for any project. Describe what you need help with and the factory designs, scaffolds, and deploys your agent team. Hire agents from other projects to skip the onboarding interview."`

Updated keywords: `["agents", "subagents", "factory", "team", "mcp", "automation", "workflow", "hire", "cross-project"]`

- [ ] **Step 3: Commit**

```bash
git add agent-factory/.claude-plugin/plugin.json
git commit -m "docs: update plugin metadata for hire-agent feature"
```

---

### Task 7: Update Design Spec with Existing Agents Compatibility

**Files:**
- Modify: `agent-factory/skills/SKILL.md` (add `source` field to team.json schema)
- Modify: `agent-factory/skills/add-agent.md` (add `source` field to new agent entries)

The `team.json` schema in the existing skills doesn't include the `source` field yet. Add it for forward compatibility so created agents get `"source": "created"`.

- [ ] **Step 1: Update team.json schema in SKILL.md**

In the Team Manifest section of `agent-factory/skills/SKILL.md`, add `"source": "created"` to the agent entry example:

```json
{
  "name": "[agent-slug]",
  "role": "[one-sentence role description]",
  "scope": "project-local",
  "source": "created",
  "pipeline_position": null,
  "dependencies": []
}
```

- [ ] **Step 2: Update add-agent.md scaffold step**

In the Scaffold section of `agent-factory/skills/add-agent.md`, step 5 says to add the new agent entry to the `agents` array. Update the instruction to include `"source": "created"` in the entry that gets appended to `team.json`.

- [ ] **Step 3: Verify the edits**

Read both `agent-factory/skills/SKILL.md` and `agent-factory/skills/add-agent.md` to confirm the `source` field is present in both schema examples.

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/SKILL.md agent-factory/skills/add-agent.md
git commit -m "feat: add source field to team.json agent entries"
```

---

### Task 8: End-to-End Verification

This task verifies the complete implementation by reading all modified and created files.

- [ ] **Step 1: Verify all files exist**

Use Glob to confirm these files exist:
- `agent-factory/skills/hire-agent.md`
- `agent-factory/skills/registry-helpers.md`
- `agent-factory/skills/SKILL.md`
- `agent-factory/skills/add-agent.md`
- `agent-factory/skills/upgrade-agent.md`
- `agent-factory/.claude-plugin/plugin.json`

- [ ] **Step 2: Verify registry helpers are referenced correctly**

Grep all skill files for `registry-helpers.md` to confirm each skill that writes to the registry references the helpers document:
- `SKILL.md` should reference it
- `add-agent.md` should reference it
- `upgrade-agent.md` should reference it
- `hire-agent.md` should reference it

- [ ] **Step 3: Verify no placeholder text remains**

Grep all modified/created files for `{{` to ensure no template placeholders leaked into the skill files (as opposed to template files where they belong).

- [ ] **Step 4: Verify git status is clean**

Run `git status` to confirm all changes are committed.

- [ ] **Step 5: Review commit history**

Run `git log --oneline -10` to verify the commit sequence makes sense.
