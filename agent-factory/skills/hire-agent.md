<!-- ABOUTME: Skill for hiring agents from other projects into the current project -->
<!-- ABOUTME: Discovers agents via registry, matches by intent, copies with light adaptation -->
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
