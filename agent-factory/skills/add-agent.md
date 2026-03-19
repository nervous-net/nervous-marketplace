---
name: add-agent
description: Add a new agent to an existing team. Detects the current team from agents/team.json, shows the roster, interviews you about what gap to fill, and scaffolds the new agent with full personality and memory.
---

# Add Agent

Add a new member to an existing agent team.

## Detection

1. Look for `agents/team.json` in the current project (use Glob tool).
2. If **NOT found** → "No agent team found in this project. Run `/create-agents` first to set up a team." Stop.
3. If **found** → read it (tolerantly — missing fields use defaults). Also read each agent's `character-sheet.md` to understand existing team dynamics.

## Show Current Team

Present the existing team:

"Your current team (**[team name]**, [interaction model]):

[For each agent:]
- **[name]** — [role]

[N] agents total."

## Interview

Ask: "What gap are you trying to fill? What's missing from this team, or what do you spend time on that these agents don't cover?" *Wait for response.*

## Propose Candidates

Based on the user's answer AND the existing team composition, propose 1-2 new agents:

For each candidate:
- **Name** and **archetype**
- **Role** — one sentence
- **Why** — how this agent complements the existing team
- **Team dynamics** — how they'd interact with existing agents (agree/disagree/complement)

"Which one fits? Or describe what you're looking for and I'll design a custom agent."

*Wait for response.*

## Deep-Dive

Run the same deep-dive process as `/create-agents` Phase 3 (Steps 1-5):
1. Present draft character sheet — MUST include "Relationship to Team" section referencing existing agents by name
2. Tune personality (one question at a time, wait for responses)
3. Define contract
4. Identify references (scan project, present checklist)
5. Confirm complete setup

## Scaffold

Generate files for the new agent only:
1. Create `agents/[agent-slug]/` directory structure (including `references/`, `memory/`, and `agents/shared/[agent-slug]/`)
2. Write character sheet, prompt, dispatch doc, contract (using templates at `<skill-base-dir>/templates/`)
3. Create `memory/MEMORY.md` with empty index
4. Copy selected reference docs
5. **Update** `agents/team.json` — add the new agent entry to the `agents` array. Do NOT overwrite the file — read, parse, add entry, write back.
6. Regenerate the CLAUDE.md discovery block between `<!-- agent-factory:start -->` and `<!-- agent-factory:end -->` markers with all agents listed

**Slugification:** Lowercase, replace spaces and non-alphanumeric characters with hyphens, collapse consecutive hyphens, trim leading/trailing hyphens.

**Prompt file:** Must be a full narrative giving the agent a voice (not just filled-in template). Include the memory file format so the agent knows how to save memories:
```
When saving memories, use this format:
---
name: short-name
description: one-line description
type: user | feedback | project | reference
date: YYYY-MM-DD
---
Memory content here.
```
Then add the memory file to `memory/MEMORY.md` index.

Present summary of what was created.

## Test Drive (Optional)

"Would you like to test drive **[agent name]** now?"

If yes: pick a small, real task appropriate to the agent's role. Dispatch via the Agent tool using the agent's `prompt.md`. Write output to `agents/shared/[agent-slug]/test-drive.md`. Present the output, ask for feedback, iterate if needed.

If no: "They're ready whenever you are."

## Error Handling

- **Agent name collision:** "There's already an agent called [name]. Pick a different name?" *Wait for response.*
- **team.json parse error:** Try to read what's parseable, report the issue, suggest the user check the file.
- **No character sheets found for existing agents:** Warn that team dynamics may be thin, proceed anyway.

## Rules

- **One question at a time.** Never ask multiple questions in a single message.
- **Wait for responses.** Never skip ahead.
- **Templates are starting points.** Fill them with specific content. Never leave `{{placeholder}}` text in generated files.
- **Never auto-commit.** Let the user decide when to commit.
- **Never overwrite without asking.** If a file exists, ask before replacing.
- **Memory is mandatory.** Every agent gets a memory directory from creation.
- **Read config tolerantly.** Missing fields use sensible defaults.
