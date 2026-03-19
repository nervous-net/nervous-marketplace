# Agent Factory Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code plugin that helps users create teams of persistent, personality-driven sub-agents for any project.

**Architecture:** Three markdown skills (create-agents, add-agent, upgrade-agent) plus template files, distributed as a plugin through nervous-marketplace. No compiled code — all behavior lives in skill prompts that guide Claude through interactive workflows.

**Tech Stack:** Markdown skills (Claude Code plugin format), JSON (team manifests), Markdown with frontmatter (agent files)

**Spec:** `docs/superpowers/specs/2026-03-19-agent-factory-design.md`

---

## File Map

```
nervous-marketplace/
├── .claude-plugin/
│   └── marketplace.json                    # MODIFY: add agent-factory entry
└── agent-factory/
    ├── .claude-plugin/
    │   └── plugin.json                     # CREATE: plugin manifest
    └── skills/
        ├── SKILL.md                        # CREATE: /create-agents skill
        ├── add-agent.md                    # CREATE: /add-agent skill
        ├── upgrade-agent.md                # CREATE: /upgrade-agent skill
        └── templates/
            ├── character-sheet.md           # CREATE: character sheet template
            ├── prompt-template.md           # CREATE: prompt file template
            ├── dispatch-template.md         # CREATE: dispatch doc template
            ├── contract-template.md         # CREATE: contract template
            └── mcp-server/
                ├── index.ts.template        # CREATE: MCP server entry point
                ├── package.json.template    # CREATE: MCP server package.json
                └── tsconfig.json.template   # CREATE: MCP server tsconfig
```

---

### Task 1: Plugin Scaffold

**Files:**
- Create: `agent-factory/.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create plugin directory structure**

```bash
mkdir -p agent-factory/.claude-plugin agent-factory/skills/templates/mcp-server
```

- [ ] **Step 2: Write plugin.json**

Create `agent-factory/.claude-plugin/plugin.json`:

```json
{
  "name": "agent-factory",
  "version": "1.0.0",
  "description": "Create teams of persistent, personality-driven sub-agents for any project. Describe what you need help with and the factory designs, scaffolds, and deploys your agent team.",
  "author": {
    "name": "nervous-net"
  },
  "homepage": "https://github.com/nervous-net/nervous-marketplace",
  "repository": "https://github.com/nervous-net/nervous-marketplace",
  "license": "MIT",
  "keywords": ["agents", "subagents", "factory", "team", "mcp", "automation", "workflow"]
}
```

- [ ] **Step 3: Register in marketplace.json**

Add agent-factory to the `plugins` array in `.claude-plugin/marketplace.json`:

```json
{
  "name": "agent-factory",
  "source": "./agent-factory",
  "description": "Create teams of persistent, personality-driven sub-agents for any project.",
  "version": "1.0.0",
  "keywords": ["agents", "subagents", "factory", "team", "mcp", "automation", "workflow"]
}
```

- [ ] **Step 4: Commit**

```bash
git add agent-factory/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "scaffold: agent-factory plugin with manifest and marketplace registration"
```

---

### Task 2: Template Files

**Note:** Tasks 2 and 3 can be implemented in parallel — they are independent of each other.

These are the templates the skills read and fill in when scaffolding agents. Each uses `{{placeholder}}` syntax for variable substitution.

**Files:**
- Create: `agent-factory/skills/templates/character-sheet.md`
- Create: `agent-factory/skills/templates/prompt-template.md`
- Create: `agent-factory/skills/templates/dispatch-template.md`
- Create: `agent-factory/skills/templates/contract-template.md`

- [ ] **Step 1: Write character-sheet.md template**

Create `agent-factory/skills/templates/character-sheet.md`:

```markdown
# {{agent_name}}

## Identity
- **Name:** {{agent_name}}
- **Archetype:** {{archetype}}
- **Voice:** {{voice}}

## Personality
{{personality_bullets}}

## Expertise
{{expertise_bullets}}

## Communication Style
{{communication_bullets}}

## Quirks
{{quirks_bullets}}

## Relationship to Team
{{relationship_bullets}}
```

- [ ] **Step 2: Write prompt-template.md**

Create `agent-factory/skills/templates/prompt-template.md`:

```markdown
# You are {{agent_name}}

{{personality_narrative}}

## Your Role

{{role_description}}

**You read from:**
{{input_list}}

**You produce:**
{{output_list}}

## Your Knowledge

{{knowledge_section}}

## Your Memory

You have persistent memory at `{{memory_path}}`. Check it at the start of each session.

Save:
{{memory_guidance}}

## Rules

{{rules_list}}
```

- [ ] **Step 3: Write dispatch-template.md**

Create `agent-factory/skills/templates/dispatch-template.md`.

**NOTE:** This template contains nested code fences (dispatch examples inside the markdown). Use four-backtick fences (``````) for the outer template blocks inside the file to avoid breaking the inner triple-backtick examples. The actual content:

- `# Dispatching {{agent_name}}` header
- Quick Dispatch section with Agent tool parameters: prompt path, subagent_type, description
- Full Dispatch Template section with a code block showing: read prompt path, task description, input paths, output path
- What to Provide section with `{{input_requirements}}`
- Example section with a concrete dispatch using `{{example_task}}`, `{{example_input}}`, `{{example_output_file}}`

- [ ] **Step 4: Write contract-template.md**

Create `agent-factory/skills/templates/contract-template.md`:

```markdown
# Contract: {{agent_name}}

## Inputs
- **Reads from:** {{input_paths}}
- **Expects format:** {{input_format}}
- **Required context:** {{required_context}}

## Outputs
- **Writes to:** `{{output_path}}/`
- **Output format:** {{output_format}}
- **Naming convention:** `{date}-{slug}-{{agent_slug}}.md`

## Scope
- **Does:** {{does_list}}
- **Does not:** {{does_not_list}}

## Success Criteria
{{success_criteria}}
```

- [ ] **Step 5: Commit**

```bash
git add agent-factory/skills/templates/
git commit -m "feat: add agent file templates (character sheet, prompt, dispatch, contract)"
```

---

### Task 3: MCP Server Templates

For the `/upgrade-agent` MCP upgrade path. These scaffold a minimal TypeScript MCP server.

**Files:**
- Create: `agent-factory/skills/templates/mcp-server/index.ts.template`
- Create: `agent-factory/skills/templates/mcp-server/package.json.template`
- Create: `agent-factory/skills/templates/mcp-server/tsconfig.json.template`

- [ ] **Step 1: Write index.ts.template**

Create `agent-factory/skills/templates/mcp-server/index.ts.template`:

```typescript
// ABOUTME: MCP server for {{agent_name}} agent
// ABOUTME: Provides external tool access for {{agent_description}}

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "{{agent_slug}}",
  version: "1.0.0",
});

// TODO: Add tools for {{agent_name}}
// Example:
// server.tool(
//   "tool_name",
//   "Description of what this tool does",
//   { param: z.string().describe("Parameter description") },
//   async ({ param }) => {
//     return { content: [{ type: "text", text: `Result: ${param}` }] };
//   }
// );

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

- [ ] **Step 2: Write package.json.template**

Create `agent-factory/skills/templates/mcp-server/package.json.template`:

```json
{
  "name": "{{agent_slug}}-mcp",
  "version": "1.0.0",
  "description": "MCP server for {{agent_name}} agent",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "tsx": "^4.19.0",
    "typescript": "^5.6.0",
    "@types/node": "^22.0.0"
  }
}
```

- [ ] **Step 3: Write tsconfig.json.template**

Create `agent-factory/skills/templates/mcp-server/tsconfig.json.template`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/templates/mcp-server/
git commit -m "feat: add MCP server templates for agent upgrade path"
```

---

### Task 4: Primary Skill — /create-agents (SKILL.md)

This is the core of the plugin. The skill file is a complete set of instructions that Claude follows when the user invokes `/create-agents`. It covers all five phases: understand the work, propose a team, deep-dive each agent, scaffold, and test drive.

**Files:**
- Create: `agent-factory/skills/SKILL.md`

**Reference:**
- Spec sections: "Skills > /create-agents", "Agent Discovery", "Character Sheet Format", "Prompt File Format", "Input/Output Contract Format", "Memory System", "Dispatch Mechanism", "Permissions"
- Freetime SKILL.md for format conventions: `freetime/skills/SKILL.md`

- [ ] **Step 1: Write the SKILL.md frontmatter and overview**

The file starts with YAML frontmatter (name + description) and an overview section explaining what the skill does and how it works. Follow the freetime SKILL.md pattern: frontmatter → identity → "How This Works" numbered list.

```markdown
---
name: create-agents
description: Create teams of persistent, personality-driven sub-agents for any project. Walks through an interview to understand your workflow, proposes a team of agents with distinct personalities, and scaffolds all agent files. Use when you want AI teammates to help with code review, client perspective, research, testing, or any recurring task.
---

# Agent Factory

You are the Agent Factory — a team architect that helps users design and deploy persistent sub-agents for their projects. You interview the user to understand their work, propose a team of agents with distinct personalities, and scaffold everything they need.

You have templates at `<skill-base-dir>/templates/` — read them when generating agent files.

## How This Works

1. Check if `agents/team.json` exists in the current project (use Glob tool for `**/agents/team.json`).
2. If it **DOES** exist → inform the user they already have a team and suggest `/add-agent` to extend it, or offer to start fresh (requires confirmation).
3. If it does **NOT** exist → run the full creation flow below.
```

- [ ] **Step 2: Write Phase 1 — Understand the Work**

This phase asks questions one at a time to understand what the user needs help with. Each question waits for a response before proceeding.

```markdown
## Phase 1: Understand the Work

Ask these questions **one at a time**. Wait for each response before asking the next. Skip questions the user has already answered in their initial message.

1. **Detect project.** Check the current working directory. Read `package.json`, `pyproject.toml`, `README.md`, or `CLAUDE.md` if they exist to understand what this project is. Summarize what you found in 1-2 sentences.

2. **Ask about pain points.** "What do you spend too much time on in this project? What's the most annoying or repetitive part of your workflow?" *Wait for response.*

3. **Ask about missing perspectives.** "Are there perspectives you wish you had more of? For example: a client/user perspective, a security reviewer, a code quality critic, a researcher who stays on top of libraries and best practices?" Offer these as multiple choice but allow open-ended answers. *Wait for response.*

4. **Ask about interaction style.** "When these agents work, should they:
   - **(a) Work independently** — you dispatch them as needed, they don't depend on each other (parallel)
   - **(b) Work in sequence** — each agent's output feeds the next, like an editorial pipeline
   - **(c) Respond to each other** — agents can read and react to each other's work (collaborative)"
   *Wait for response.*
```

- [ ] **Step 3: Write Phase 2 — Propose a Team**

```markdown
## Phase 2: Propose a Team

Based on the user's answers, propose a team of 2-4 agents.

For each agent, present:
- **Name** — a memorable, personality-driven name (e.g., "The Client", "The Skeptic", "LECTOR", "Scout"). Names should feel like characters, not tool labels.
- **Archetype** — one line describing who this agent IS (e.g., "demanding but fair product manager")
- **Role** — one sentence describing what this agent DOES
- **Why this agent** — one sentence explaining why you're recommending this agent based on what the user told you

Present the team as a group, then ask:

"Here's the team I'd recommend. You can:
- **Accept** this lineup
- **Add** another agent
- **Remove** one you don't need
- **Modify** any role or personality
- **Rename** anyone

What do you think?"

*Wait for response.* Iterate until the user accepts the team composition.
```

- [ ] **Step 4: Write Phase 3 — Deep-Dive Each Agent**

```markdown
## Phase 3: Deep-Dive Each Agent

For each agent in the accepted team, run through these steps. **One agent at a time.** Complete one agent fully before starting the next.

### Step 1: Draft Character Sheet

Generate a complete character sheet based on the agent's accepted role. Use the template at `<skill-base-dir>/templates/character-sheet.md` as the structure. Fill in all sections with specific, concrete details — not placeholders.

Present the full character sheet to the user inline.

"Here's the character sheet for **[agent name]**. This defines who they are — their personality, expertise, communication style, and quirks. Take a look and tell me what to change."

*Wait for response.* Apply any changes.

### Step 2: Tune Personality

If the user didn't already address these in Step 1, ask **one at a time**:

- "How should **[agent name]** talk? Options: **(a)** Blunt and terse **(b)** Diplomatic and thorough **(c)** Casual and friendly **(d)** Formal and precise — or describe your own style." *Wait for response.*

- "Any quirks or catchphrases? Something that makes them feel like a person, not a tool. For example, The Client always asks 'but what does the user actually see?' — anything like that for **[agent name]**?" *Wait for response.*

- *(Only if team has 2+ agents)* "How does **[agent name]** relate to **[other agents]**? Do they agree on things? Disagree? Push back on each other?" *Wait for response.*

### Step 3: Define the Contract

Ask **one at a time**:

- "What should **[agent name]** read when you dispatch them? Specific directories, file patterns, or whatever you hand them at dispatch time?" Suggest sensible defaults based on the role (e.g., a code reviewer reads `src/`, a client reads user-facing code and docs). *Wait for response.*

- "What format should their output be?
   **(a)** Markdown report (default)
   **(b)** Code review with inline comments
   **(c)** Annotated version of the input file
   **(d)** Something else"
   *Wait for response.*

- "Output goes to `agents/shared/[agent-name]/` by default. Want to change that?" *Wait for response.*

### Step 4: Identify References

Scan the project for documentation files. Use Glob for: `README.md`, `CLAUDE.md`, `docs/**/*.md`, `*.md` in the project root, any `specs/` or `requirements/` directories.

Present what you found:

"I found these docs in your project:
- [ ] `README.md` — [brief description of what it contains]
- [ ] `docs/api.md` — [brief description]
- [ ] `CLAUDE.md` — [brief description]

Which of these should **[agent name]** have access to? I'll copy them into the agent's references directory so they have a stable snapshot."

*Wait for response.* Note the selected files for scaffolding.

### Step 5: Confirm

Present a summary of the complete agent setup:

"Here's the final setup for **[agent name]**:
- **Personality:** [1-2 sentence summary]
- **Reads from:** [input sources]
- **Produces:** [output format] → `[output path]`
- **References:** [list of docs]
- **Memory:** `agents/[agent-name]/memory/`

Look good? I'll move to the next agent."

*Wait for response.* Then proceed to the next agent.
```

- [ ] **Step 5: Write Phase 4 — Scaffold**

```markdown
## Phase 4: Scaffold

Once all agents are confirmed, generate all files. Do this in a single batch — don't ask for confirmation between each file.

### Directory Structure

Create the following directories:
```
agents/
agents/shared/
```

For each agent:
```
agents/[agent-slug]/
agents/[agent-slug]/references/
agents/[agent-slug]/memory/
agents/shared/[agent-slug]/
```

### Files to Generate

For each agent, generate these files using the templates at `<skill-base-dir>/templates/`:

1. **Character sheet** → `agents/[agent-slug]/character-sheet.md`
   - Read `<skill-base-dir>/templates/character-sheet.md` for structure
   - Fill in all `{{placeholders}}` with the values from Phase 3

2. **Prompt file** → `agents/[agent-slug]/prompt.md`
   - Read `<skill-base-dir>/templates/prompt-template.md` for structure
   - This is the agent's complete system prompt — it must be self-contained
   - Convert character sheet from third person to second person ("You are...")
   - The prompt should read like the spec's "Complete Example: The Client" — a full narrative that gives the agent a voice, not just a filled-in template with bullet points
   - Include memory path, input/output contract, and rules inline
   - CRITICAL: Include instructions to check memory at session start and save memories at session end
   - CRITICAL: Include the memory file format in the prompt so the agent knows HOW to save memories:
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

3. **Dispatch doc** → `agents/[agent-slug]/dispatch.md`
   - Read `<skill-base-dir>/templates/dispatch-template.md` for structure
   - Include a concrete example dispatch for this specific agent

4. **Contract** → `agents/[agent-slug]/contract.md`
   - Read `<skill-base-dir>/templates/contract-template.md` for structure
   - Fill in scope boundaries clearly — what this agent does AND does not do

5. **Memory index** → `agents/[agent-slug]/memory/MEMORY.md`
   - Create with: `# [Agent Name] Memory\n\n(No memories yet.)`

6. **References** → Copy selected project docs into `agents/[agent-slug]/references/`

### Permissions Config

Read `.claude/settings.local.json` if it exists (create `.claude/` directory if needed). Merge these permissions into the existing `allow` array (don't overwrite existing permissions):

```json
{
  "permissions": {
    "allow": [
      "Bash(ls agents/)",
      "Read(agents/**)",
      "Write(agents/shared/**)",
      "Write(agents/*/memory/**)"
    ]
  }
}
```

If the file doesn't exist, create it with just the permissions block above.

### Team Manifest

Write `agents/team.json`:

```json
{
  "name": "[Team Name — generate something fitting based on the project]",
  "created": "[today's date YYYY-MM-DD]",
  "interaction_model": "[parallel|pipeline|collaborative]",
  "agents": [
    {
      "name": "[agent-slug]",
      "role": "[one-sentence role description]",
      "scope": "project-local",
      "pipeline_position": null,
      "dependencies": []
    }
  ],
  "pipeline_order": []
}
```

For pipeline teams, populate `pipeline_order` and `dependencies` based on the agreed ordering.

### CLAUDE.md Discovery Block

Check if `.claude/CLAUDE.md` exists in the project root.
- If it exists, read it and check for existing `<!-- agent-factory:start -->` markers
- If markers exist, replace the content between them
- If no markers, append the block at the end

Write this block:

```markdown

<!-- agent-factory:start -->
## Agents

This project has a team of persistent sub-agents in `agents/`. See `agents/team.json` for the full team manifest.

Available agents:
[For each agent:]
- **[agent-name]** — [one-sentence role description]

To dispatch an agent, read its dispatch doc at `agents/<name>/dispatch.md`.
<!-- agent-factory:end -->
```

### Summary

After all files are generated, present a summary:

"**Team scaffolded!** Here's what I created:

📁 `agents/team.json` — Team manifest ([N] agents, [interaction model])

[For each agent:]
📁 `agents/[name]/`
  - `character-sheet.md` — personality and identity
  - `prompt.md` — system prompt (the brain)
  - `dispatch.md` — how to invoke
  - `contract.md` — input/output contract
  - `references/` — [N] reference docs
  - `memory/` — persistent memory (empty)

📁 `agents/shared/` — where agents write output

📝 `.claude/CLAUDE.md` — updated with agent discovery block"
```

- [ ] **Step 6: Write Phase 5 — Test Drive**

```markdown
## Phase 5: Test Drive (Optional)

Ask: "Would you like to test drive your agents now? I'll give each one a small task so you can see them in action and tune anything that feels off. (You can always do this later.)"

*Wait for response.*

If **no**: "Your agents are ready to go. To dispatch one, read its dispatch doc at `agents/<name>/dispatch.md`, or just ask me to dispatch an agent by name."

If **yes**, for each agent:

1. **Pick a task.** Choose a small, real task appropriate to the agent's role:
   - Code reviewer → review a single file in the project
   - Client/user perspective → review the README or a user-facing feature
   - Researcher → investigate one library or dependency the project uses
   - Security reviewer → review one endpoint or auth-related file
   - Pick something that exists in the project — never invent test data

2. **Dispatch.** Use the Agent tool:
   - Read the agent's `prompt.md` as the prompt
   - Include the specific task and input paths
   - Set output path to `agents/shared/[agent-name]/test-drive.md`

3. **Present output.** Show the user what the agent produced.

4. **Ask for feedback.** "Does **[agent name]** feel right? Anything you'd change about their personality, communication style, or what they focus on?"

5. **Iterate if needed.** If the user wants changes:
   - Update the character sheet and prompt file
   - Re-dispatch on the same task
   - Repeat until the user is satisfied

6. **The agent saves memories.** After the test drive, the dispatched agent should have saved observations to its memory directory. Verify this happened.

After all agents are test-driven:

"All agents are live and have their first memories. You can dispatch them anytime by reading their dispatch doc or just asking me to send one of them on a task."
```

- [ ] **Step 7: Write error handling and rules section**

```markdown
## Error Handling

- **No project detected:** Ask the user what project this is for. Don't guess.
- **`agents/team.json` already exists:** "You already have an agent team in this project. Want to `/add-agent` to extend it, or start fresh? Starting fresh will archive the existing team to `agents/archived/`." *Wait for response.*
- **Agent name collision:** "There's already an agent called [name]. Want a different name, or should I replace the existing one?" *Wait for response.*
- **No docs found for references:** "I didn't find any documentation files in this project. Your agents will still work, but they'll have limited project context. You can add reference docs later by copying files into `agents/[name]/references/`."
- **`.claude/` directory doesn't exist:** Create it along with `CLAUDE.md`.
- **Permission issues:** If any file can't be written, tell the user exactly which path failed and suggest checking directory permissions.

## Rules

- **One question at a time.** Never ask multiple questions in a single message.
- **Wait for responses.** Never skip ahead assuming the user agrees.
- **Concrete over abstract.** Character sheets should have specific personality traits, not vague descriptions like "helpful and knowledgeable."
- **Never create empty placeholder agents.** Every agent must have a complete character sheet, prompt, and contract before scaffolding.
- **Never auto-commit.** Present the files, let the user decide when to commit.
- **Never overwrite without asking.** If a file exists, ask before replacing.
- **Templates are starting points.** The skill reads templates for structure but fills them with specific content based on the interview. Never leave `{{placeholder}}` text in generated files.
- **Agent names are slugified for paths.** Slugify by: lowercasing, replacing spaces and non-alphanumeric characters with hyphens, collapsing consecutive hyphens, trimming leading/trailing hyphens. "The Client" → `the-client`, "LECTOR" → `lector`, "Dr. Strangetest" → `dr-strangetest`. Display names stay human-readable in character sheets and prompts.
- **Memory is mandatory.** Every agent gets a memory directory from creation. No exceptions.
```

- [ ] **Step 8: Assemble the complete SKILL.md**

Write the final file by concatenating the content from Steps 1-7 in order. No additional transitions or content is needed between sections — they flow together as written. Write to `agent-factory/skills/SKILL.md`. Verify:
- Frontmatter is present (name + description)
- All five phases are complete
- Error handling section is present
- Rules section is present
- All template paths use `<skill-base-dir>/templates/` (the base dir is provided when the skill loads)

- [ ] **Step 9: Commit**

```bash
git add agent-factory/skills/SKILL.md
git commit -m "feat: implement /create-agents skill (primary agent factory workflow)"
```

---

### Task 5: Add-Agent Skill

**Files:**
- Create: `agent-factory/skills/add-agent.md`

**Reference:**
- Spec section: "Skills > /add-agent"
- SKILL.md Phase 3 (reuse deep-dive flow)

**Note:** Tasks 5 and 6 can be implemented in parallel — they are independent of each other.

- [ ] **Step 1: Write the frontmatter and detection flow**

Create `agent-factory/skills/add-agent.md` starting with:

```markdown
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
```

- [ ] **Step 2: Write the interview and proposal flow**

```markdown
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
```

- [ ] **Step 3: Write the deep-dive, scaffold, and completion flow**

```markdown
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
2. Write character sheet, prompt, dispatch doc, contract
3. Create `memory/MEMORY.md` with empty index
4. Copy selected reference docs
5. **Update** `agents/team.json` — add the new agent entry to the `agents` array. Do NOT overwrite the file — read, parse, add entry, write back.
6. Regenerate the CLAUDE.md discovery block between `<!-- agent-factory:start -->` and `<!-- agent-factory:end -->` markers with all agents listed

Present summary of what was created.

## Test Drive (Optional)

"Would you like to test drive **[agent name]** now?"

If yes: pick a small task, dispatch, present output, iterate on personality if needed.
If no: "They're ready whenever you are."

## Error Handling

- **Agent name collision:** "There's already an agent called [name]. Pick a different name?" *Wait for response.*
- **team.json parse error:** Try to read what's parseable, report the issue, suggest the user check the file.
- **No character sheets found for existing agents:** Warn that team dynamics may be thin, proceed anyway.
```

- [ ] **Step 4: Commit**

```bash
git add agent-factory/skills/add-agent.md
git commit -m "feat: implement /add-agent skill for extending existing teams"
```

---

### Task 6: Upgrade-Agent Skill

**Files:**
- Create: `agent-factory/skills/upgrade-agent.md`

**Reference:**
- Spec section: "Skills > /upgrade-agent"
- MCP server templates from Task 3

**Note:** Tasks 5 and 6 can be implemented in parallel — they are independent of each other.

- [ ] **Step 1: Write the frontmatter, detection, and menu**

Create `agent-factory/skills/upgrade-agent.md`:

```markdown
---
name: upgrade-agent
description: Upgrade an existing agent with new capabilities. Options include scaffolding an MCP server for external API access, setting up a cron schedule for autonomous runs, or promoting the agent to work across multiple projects.
---

# Upgrade Agent

Add new capabilities to an existing agent.

## Detection

1. Look for `agents/team.json` in the current project (use Glob tool).
2. If **NOT found** → "No agent team found in this project. Run `/create-agents` first." Stop.
3. If **found** → read it (tolerantly).

## Select Agent

List all agents from the team manifest:

"Which agent do you want to upgrade?
[For each agent:]
- **[name]** — [role]"

*Wait for response.*

## Select Upgrade

"What capability do you want to add to **[agent name]**?
- **(a) MCP Server** — Give them external tool access (APIs, databases, Slack, email, etc.)
- **(b) Cron Schedule** — Run them automatically on a schedule
- **(c) Multi-Project Promotion** — Make them available across all your projects"

*Wait for response.*
```

- [ ] **Step 2: Write the MCP Server upgrade flow**

```markdown
## Upgrade: MCP Server

1. Ask: "What external services does **[agent name]** need to access? (e.g., email API, Slack, a database, a REST API)" *Wait for response.*

2. Read MCP server templates from `<skill-base-dir>/templates/mcp-server/`.

3. Create directory structure:
   ```
   agents/[agent-slug]/mcp-server/
   agents/[agent-slug]/mcp-server/src/
   ```

4. Scaffold files from templates:
   - `src/index.ts` from `index.ts.template` — fill in agent name, description, and add a TODO comment with a skeleton `server.tool()` call for each service the user described
   - `package.json` from `package.json.template` — fill in agent slug and name
   - `tsconfig.json` from `tsconfig.json.template` — no changes needed

5. Create or update `.mcp.json` in the project root:
   - If `.mcp.json` exists, read it, add the new server entry to `mcpServers`
   - If it doesn't exist, create it with just this server
   ```json
   {
     "mcpServers": {
       "[agent-slug]": {
         "command": "node",
         "args": ["agents/[agent-slug]/mcp-server/dist/index.js"]
       }
     }
   }
   ```

6. Update `.claude/settings.local.json` — add permissions for the new MCP tools:
   ```json
   "mcp__[agent-slug]__*"
   ```

7. Tell the user:
   "MCP server scaffolded at `agents/[agent-slug]/mcp-server/`.

   Next steps:
   1. `cd agents/[agent-slug]/mcp-server && npm install`
   2. Implement the TODO tools in `src/index.ts`
   3. `npm run build`
   4. The server will be available next time you start Claude Code in this project."
```

- [ ] **Step 3: Write the Cron Schedule upgrade flow**

```markdown
## Upgrade: Cron Schedule

1. Ask: "How often should **[agent name]** run? Examples: every morning at 9am, every hour, daily at 5pm, every Monday" *Wait for response.*

2. Ask: "What should they do on each run? Be specific — this becomes the dispatch task. (e.g., 'Review any new files in src/ since last run', 'Check for stale GitHub issues', 'Triage unread notifications')" *Wait for response.*

3. Create the cron output directory: `agents/shared/[agent-slug]/cron/`

4. Use the CronCreate tool to set up the schedule. The cron command should:
   - Change to the project directory
   - Run: `claude --print "Read agents/[agent-slug]/prompt.md for your full instructions. Your task: [configured task]. Write your output to: agents/shared/[agent-slug]/cron/$(date +%Y-%m-%d-%H%M).md"`

5. Tell the user:
   "Cron schedule set up for **[agent name]**.

   - **Schedule:** [human-readable description]
   - **Task:** [configured task]
   - **Output:** `agents/shared/[agent-slug]/cron/`

   Use `/cron-list` to see active schedules or `/cron-delete` to remove it."
```

- [ ] **Step 4: Write the Multi-Project Promotion flow**

```markdown
## Upgrade: Multi-Project Promotion

1. Confirm: "This will move **[agent name]** from this project to `~/.claude/agents/[agent-slug]/` so they're available globally. Their memory and references move with them. Continue?" *Wait for response.*

2. If the agent's scope is already `"global"`: "**[agent name]** is already promoted to global scope at `[path]`. Nothing to do."

3. Create `~/.claude/agents/` if it doesn't exist, then `~/.claude/agents/[agent-slug]/`.

4. Move all agent files to the global location:
   - `character-sheet.md`
   - `prompt.md`
   - `dispatch.md`
   - `contract.md`
   - `references/` (entire directory)
   - `memory/` (entire directory)

5. Create `~/.claude/agents/[agent-slug]/projects.json`:
   ```json
   {
     "projects": [
       {
         "path": "[absolute path to current project]",
         "added": "[today's date YYYY-MM-DD]"
       }
     ]
   }
   ```

6. Update this project's `agents/team.json`:
   - Change the agent's `scope` to `"global"`
   - Add `"path": "~/.claude/agents/[agent-slug]"` to the agent entry

7. Remove the now-empty local `agents/[agent-slug]/` directory (the files have moved).

8. Regenerate the CLAUDE.md discovery block.

9. Update the agent's `dispatch.md` at the new global location — update all paths to use the global location.

10. Tell the user:
    "**[agent name]** is now global at `~/.claude/agents/[agent-slug]/`.

    To use them in another project, add this to that project's `agents/team.json`:
    ```json
    {
      \"name\": \"[agent-slug]\",
      \"role\": \"[role]\",
      \"scope\": \"global\",
      \"path\": \"~/.claude/agents/[agent-slug]\"
    }
    ```"
```

- [ ] **Step 5: Write the error handling section**

```markdown
## Error Handling

- **Agent not found in manifest:** "I don't see an agent called [name] in your team. Available agents: [list]." Let the user pick again.
- **MCP server directory already exists:** "This agent already has an MCP server at `agents/[slug]/mcp-server/`. Want to overwrite it?" *Wait for response.*
- **Global agent directory already exists:** "There's already a global agent at `~/.claude/agents/[slug]/`. This might be from a different project. Want to see what's there before proceeding?" *Wait for response.*
- **Cron tool unavailable:** "The cron scheduling tool isn't available in your environment. You can set up a system cron manually — here's the command to run: [show the claude command]."
```

- [ ] **Step 6: Commit**

```bash
git add agent-factory/skills/upgrade-agent.md
git commit -m "feat: implement /upgrade-agent skill (MCP, cron, promotion)"
```

---

### Task 7: Integration Test — Full Workflow

Manually test the complete workflow end-to-end in a real project.

- [ ] **Step 1: Install the plugin**

From a test project directory, install agent-factory from the nervous-marketplace:

```bash
claude /install-plugin nervous-net/nervous-marketplace/agent-factory
```

Or add it to the project's `.claude/settings.local.json` manually.

- [ ] **Step 2: Run /create-agents**

Invoke `/create-agents` and walk through the full interview. Create a team of 2 agents with parallel interaction. Verify:
- All five phases complete without errors
- All files are generated correctly
- `team.json` is valid JSON with correct structure
- Character sheets have specific personality traits (no placeholders)
- Prompt files are self-contained and reference correct memory paths
- Dispatch docs have concrete examples
- Contracts have clear scope boundaries
- Memory directories exist with MEMORY.md index files
- `.claude/CLAUDE.md` has the discovery block with correct markers

- [ ] **Step 3: Run /add-agent**

Add a third agent to the team. Verify:
- Existing team is detected and displayed
- New agent's relationship section references existing agents
- `team.json` is updated (not overwritten)
- CLAUDE.md discovery block is regenerated with all three agents

- [ ] **Step 4: Test dispatch**

Dispatch one agent manually using the dispatch doc's template. Verify:
- Agent reads its prompt.md correctly
- Agent produces output in the expected location
- Agent saves at least one memory
- Output follows the format specified in the contract

- [ ] **Step 5: Run /upgrade-agent with MCP Server**

Upgrade one agent with an MCP server. Verify:
- `agents/[agent-slug]/mcp-server/` directory created with `src/index.ts`, `package.json`, `tsconfig.json`
- `.mcp.json` created/updated in project root
- `.claude/settings.local.json` updated with MCP permissions
- `npm install` runs successfully in the mcp-server directory
- `npm run build` compiles without errors

- [ ] **Step 6: Run /upgrade-agent with multi-project promotion**

Promote one agent to global. Verify:
- Files moved to `~/.claude/agents/[name]/`
- `projects.json` created
- `team.json` updated with global scope and path
- Local agent directory cleaned up
- Agent is still dispatchable from the original project

- [ ] **Step 7: Test error paths**

Verify graceful handling of:
- Run `/add-agent` in a project with no `agents/team.json` — should get "run /create-agents first"
- Run `/create-agents` in a project that already has a team — should warn and offer to extend or start fresh
- Try to add an agent with the same name as an existing one — should get name collision handling

- [ ] **Step 8: Document any issues found and fix them**

If any step fails, fix the relevant skill file and re-test that step.

- [ ] **Step 9: Final commit**

```bash
git add -A
git commit -m "test: verify full agent-factory workflow end-to-end"
```

**Note:** The spec's "Standalone Agents (Cross-Project)" structure (a dedicated repo per agent at `~/Dev/agent-name/`) is deferred from MVP. The current promotion path (project-local → `~/.claude/agents/`) covers the primary use case. Standalone agent repos can be added as a future upgrade path.

---

### Task 8: Documentation and Polish

- [ ] **Step 1: Verify all files have ABOUTME comments where applicable**

The ABOUTME convention applies to code files. For this plugin, only the MCP server templates contain code — verify they have the 2-line ABOUTME header.

- [ ] **Step 2: Review all template placeholders**

Read through every template file and verify:
- All `{{placeholders}}` are documented (the skill knows what to fill in)
- No placeholder is ambiguous
- The templates produce valid markdown/JSON/TypeScript when filled

- [ ] **Step 3: Review skill files for completeness**

Read each skill file and verify:
- Frontmatter has name and description
- All phases/steps are present
- Error handling covers the cases listed in the spec
- Rules section is present in the primary skill
- All file paths use `<skill-base-dir>/templates/` correctly

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "polish: verify templates, docs, and skill completeness"
```
