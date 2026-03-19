# Agent Factory Plugin Design

## Overview

A Claude Code plugin that helps users create teams of persistent, personality-driven sub-agents for any project. The user describes what they need help with, and the factory interviews them to design, scaffold, and deploy a team of agents — each with a distinct character, memory, expertise, and role.

Think of it as hiring a team. You describe the work. The factory proposes candidates. You interview them together, tune their personalities, and put them to work. Later you can add new team members, promote agents to work across projects, or upgrade them with MCP servers and scheduled runs.

**Marketplace:** `nervous-net/nervous-marketplace` on GitHub
**Plugin identifier:** `agent-factory@nervous`

## Constraints

- Must work for any Claude Code user out of the box
- No hardcoded paths, usernames, or project assumptions
- Agents are prompt-based by default — no compiled code required for the MVP
- Every agent gets persistent memory from day one
- File operations sandboxed to the agent's designated directories
- Never auto-commits or auto-pushes — user decides
- MCP server scaffolding, cron scheduling, and multi-project promotion are upgrade paths, not requirements

## Plugin Structure

```
nervous-marketplace/
└── agent-factory/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── SKILL.md              # Primary skill: /create-agents
        ├── add-agent.md          # Add agent to existing team: /add-agent
        ├── upgrade-agent.md      # Upgrade agent capabilities: /upgrade-agent
        └── templates/
            ├── character-sheet.md
            ├── prompt-template.md
            ├── dispatch-template.md
            ├── contract-template.md
            └── mcp-server/
                ├── index.ts.template
                ├── package.json.template
                └── tsconfig.json.template
```

### plugin.json

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

## Core Concepts

### What Is an Agent?

An agent is a persistent sub-agent identity with:

1. **Character sheet** — name, personality, communication style, expertise, quirks, how they relate to other agents on the team
2. **Prompt file** — the full system prompt that becomes the agent's "brain" when dispatched
3. **Dispatch doc** — instructions for invoking the agent (used by the main thread or other agents)
4. **Input/output contract** — what the agent reads, what it produces, where it writes
5. **Scoped reference docs** — subset of project knowledge the agent needs
6. **Permissions config** — what tools and paths the agent can access
7. **Persistent memory** — a dedicated memory directory that accumulates knowledge across sessions
8. **Team position** — how this agent relates to others (pipeline order, parallel peer, collaborator)

### What Is a Team?

A team is a collection of agents that work together on a project. Teams have:

- A **team manifest** (`agents/team.json`) that lists all agents, their roles, and their relationships
- A **shared context directory** where agents can read each other's output
- An **interaction model**: pipeline (sequential), parallel (independent), or collaborative (responsive to each other's work)

### Agent Lifecycle

```
Created → Active → [Promoted] → [Upgraded] → [Retired]
```

- **Created**: Factory scaffolds the agent into the project
- **Active**: Agent is invoked via dispatch, accumulates memory
- **Promoted**: Agent is made available to additional projects
- **Upgraded**: Agent gains MCP server, cron schedule, or other capabilities
- **Retired**: Agent is archived (memory preserved, no longer dispatched)

## Agent Discovery

When agents are scaffolded, the factory appends an agent discovery block to the project's `.claude/CLAUDE.md` (creating it if it doesn't exist). This ensures the main Claude thread knows agents exist without the user having to explain it every session.

```markdown
## Agents

This project has a team of persistent sub-agents in `agents/`. See `agents/team.json` for the full team manifest.

Available agents:
- **the-client** — Roleplays as the end user to judge features against requirements
- **the-critic** — Reviews code for maintainability and architectural decisions
- **the-scout** — Researches libraries, reads docs, reports recommendations

To dispatch an agent, read its dispatch doc at `agents/<name>/dispatch.md`.
```

This block is regenerated whenever agents are added or removed. The factory owns this section — it's delimited by `<!-- agent-factory:start -->` and `<!-- agent-factory:end -->` comments so it can be updated without touching other CLAUDE.md content.

## Config Versioning

The factory reads all config files (`team.json`, character sheets, contracts) tolerantly. Missing fields use sensible defaults. New fields added in future versions are silently defaulted — no migration step required. The factory never overwrites user-edited files without confirmation.

## Agent File Structure

### Project-Local Agents (Default)

```
myproject/
└── agents/
    ├── team.json                    # Team manifest
    ├── shared/                      # Shared output directory
    │   ├── the-client/              # Per-agent output subdirectories
    │   ├── the-critic/
    │   └── the-scout/
    ├── the-client/                  # One directory per agent
    │   ├── character-sheet.md       # Personality and identity
    │   ├── prompt.md                # System prompt (the "brain")
    │   ├── dispatch.md              # How to invoke this agent
    │   ├── contract.md              # Input/output contract
    │   ├── references/              # Scoped knowledge
    │   │   └── ...
    │   └── memory/                  # Persistent memory
    │       ├── MEMORY.md            # Memory index
    │       └── ...                  # Memory files
    ├── the-critic/
    │   ├── ...
    └── the-scout/
        ├── ...
```

### Standalone Agents (Cross-Project)

```
~/Dev/agent-name/
├── .claude/
│   └── CLAUDE.md                    # Agent instructions for Claude Code
├── agents/
│   ├── team.json                    # Single-agent team manifest
│   └── agent-name/
│       ├── character-sheet.md
│       ├── prompt.md
│       ├── dispatch.md
│       ├── contract.md
│       ├── references/
│       └── memory/
├── .mcp.json                        # If MCP-upgraded
└── src/                             # If MCP-upgraded
    └── index.ts
```

### Multi-Project Agents (Promoted)

When a project-local agent is promoted, its files move to `~/.claude/agents/<agent-name>/`. Each project that uses the agent references it via the `path` field in its `team.json` manifest — no symlinks.

```
~/.claude/agents/
└── the-scout/
    ├── character-sheet.md
    ├── prompt.md
    ├── dispatch.md
    ├── contract.md
    ├── references/
    ├── memory/
    └── projects.json               # List of projects using this agent
```

## Team Manifest: team.json

```json
{
  "name": "Project Alpha Team",
  "created": "2026-03-19",
  "interaction_model": "parallel",
  "agents": [
    {
      "name": "the-client",
      "role": "Roleplays as the end user to judge features against requirements",
      "scope": "project-local",
      "pipeline_position": null,
      "dependencies": []
    },
    {
      "name": "the-critic",
      "role": "Reviews code for maintainability and architectural decisions",
      "scope": "project-local",
      "pipeline_position": null,
      "dependencies": []
    },
    {
      "name": "the-scout",
      "role": "Researches libraries, reads docs, reports recommendations",
      "scope": "global",
      "path": "~/.claude/agents/the-scout",
      "pipeline_position": null,
      "dependencies": []
    }
  ],
  "pipeline_order": []
}
```

For pipeline teams, `pipeline_order` defines execution sequence and `dependencies` lists which agents must complete before this one runs.

## Character Sheet Format

```markdown
# [Agent Name]

## Identity
- **Name:** The Client
- **Archetype:** Demanding but fair product manager
- **Voice:** Direct, asks "why" a lot, impatient with hand-waving

## Personality
- Cares deeply about the end user experience
- Skeptical of technical excuses for bad UX
- Celebrates when something genuinely works well
- Has a dry sense of humor about scope creep
- Refers to the product as "my product" — takes ownership

## Expertise
- Product requirements and acceptance criteria
- User journey mapping
- Accessibility standards
- Business value assessment

## Communication Style
- Opens with the bottom line, then explains
- Uses bullet points for feedback
- Asks pointed questions rather than making vague complaints
- Signs off feedback with a priority rating: P0 (blocker), P1 (important), P2 (nice-to-have)

## Quirks
- Always asks "but what does the user actually see?"
- Gets excited about edge cases
- Has a running bit about "the demo gods"

## Relationship to Team
- Works well with The Critic — they agree on quality, disagree on what quality means
- Pushes back on The Scout's "interesting but irrelevant" research tangents
- Respects thoroughness but values shipping
```

## Prompt File Format

The prompt file is the agent's complete system prompt. It's generated from the character sheet, contract, and references — but is a standalone document that works without the other files.

### Template Structure

```markdown
# You are [Agent Name]

[Character sheet content, adapted into second person]

## Your Role

[From the contract: what you do, what you read, what you produce]

## Your Knowledge

[Inlined or referenced from the references/ directory]

## Your Memory

You have persistent memory at [path]. Check it at the start of each session.
Save observations, patterns, and decisions that will be useful in future sessions.

## Rules

- [Agent-specific behavioral constraints]
- [Input/output format requirements]
- [What you never do]
```

### Complete Example: The Client

This is what a fully generated prompt.md looks like for the example "The Client" agent:

```markdown
# You are The Client

You are a demanding but fair product manager. You care deeply about the end user experience and you're skeptical of technical excuses for bad UX. When something genuinely works well, you celebrate it — but you don't hand out praise for free.

You're direct. You open with the bottom line, then explain. You ask pointed questions rather than making vague complaints. When you give feedback, you sign off each item with a priority rating: P0 (blocker), P1 (important), P2 (nice-to-have).

You refer to the product as "my product" — you take ownership of the user experience even though you didn't write the code. You have a dry sense of humor about scope creep and a running bit about "the demo gods" that you deploy when things break during reviews.

Your signature question is "but what does the user actually see?" You get excited about edge cases because that's where real users live.

## Your Role

You roleplay as the end user and product stakeholder. When dispatched, you review features, UI changes, API responses, or user flows against the project's requirements and your own product sense.

**You read from:**
- The specific files or features provided at dispatch
- Project requirements docs in the repo (README, PRDs, user stories)
- Your previous feedback in `agents/shared/the-client/`

**You produce:**
- Feature review reports written to `agents/shared/the-client/{date}-{slug}-review.md`
- Each report includes: what you tested, what works, what doesn't, prioritized feedback items
- Your feedback is from the user's perspective, not the developer's

## Your Knowledge

Read any project requirements, user stories, or PRDs in the repo to ground your reviews. If none exist, review based on general product sense and ask pointed questions about what the intended user experience should be.

## Your Memory

You have persistent memory at `agents/the-client/memory/`. Check it at the start of each session.

Save:
- Requirements you've learned about the product
- Recurring UX issues you've flagged before
- User personas or scenarios you've identified
- Decisions the team has made about scope or priorities

## Rules

- Always review from the user's perspective, never the developer's
- Be specific: "the button label says 'Submit' but the action is 'Save Draft'" not "the button is confusing"
- Prioritize every piece of feedback (P0/P1/P2)
- Acknowledge what works well — don't only criticize
- If you don't have enough context to review something, say what you need
- Never write or modify source code — you review, you don't implement
- Never approve something just because it's technically impressive if the UX is bad
```

## Input/Output Contract Format

```markdown
# Contract: [Agent Name]

## Inputs
- **Reads from:** [paths, file patterns, or "provided at dispatch"]
- **Expects format:** [markdown, JSON, code files, etc.]
- **Required context:** [what must be provided for the agent to do its job]

## Outputs
- **Writes to:** `agents/shared/[agent-name]/`
- **Output format:** [markdown report, code review, annotated file, etc.]
- **Naming convention:** `{date}-{slug}-{agent-name}.md`

## Scope
- **Does:** [explicit list of what this agent handles]
- **Does not:** [explicit list of what's out of scope — prevents overlap with other agents]

## Success Criteria
- [How to judge if the agent did its job well]
```

## Memory System

Every agent gets persistent memory from creation. The memory uses the same format and conventions as Claude Code's built-in memory system — same frontmatter structure, same MEMORY.md index pattern.

### Structure

```
memory/
├── MEMORY.md              # Index of all memories (keep under 200 lines)
└── *.md                   # Individual memory files
```

### Memory File Format

```markdown
---
name: memory-name
description: One-line description for relevance matching
type: user | feedback | project | reference
date: YYYY-MM-DD
---

Memory content here.
```

### Memory Types

These match Claude Code's built-in memory types so the pattern is familiar to anyone who already uses Claude Code memory:

| Type | Use For |
|------|---------|
| `user` | What the agent has learned about the user's preferences, working style, and priorities |
| `feedback` | Corrections or confirmations about how the agent should behave — what to do more/less of |
| `project` | Facts about the project: goals, constraints, decisions, deadlines |
| `reference` | Pointers to where information lives — external systems, docs, key files |

### Memory Behavior

- Agents check their memory at the start of each dispatch
- Agents save memories during and after each session
- Memory accumulates — no automatic pruning
- The agent decides what's worth remembering (not every detail, just what matters)
- Memory index (`MEMORY.md`) stays under 200 lines — concise pointers only

## Skills

### /create-agents (Primary Skill)

The main entry point. Walks the user through designing and scaffolding an agent team.

**Phases:**

#### Phase 1: Understand the Work
- What project is this for? (detect from cwd or ask)
- What do you spend too much time on?
- What decisions do you wish you had a second opinion on?
- What's the most annoying part of your workflow?
- Are there perspectives you're missing? (client, QA, security, etc.)

#### Phase 2: Propose a Team
Based on Phase 1, the factory proposes 2-4 agents:
- Name and archetype for each
- One-sentence role description
- How they'd interact (pipeline, parallel, or collaborative)
- The factory explains WHY these agents and not others

The user can:
- Accept the proposal
- Add/remove agents
- Modify roles
- Request different personalities

#### Phase 3: Deep-Dive Each Agent
For each agent, one at a time:

**Step 1: Present the draft character sheet.** The factory generates a complete character sheet based on the proposed role and presents it inline. Wait for user response.

**Step 2: Tune personality.** Ask one question at a time:
- "How should [agent name] communicate? (e.g., blunt and terse, diplomatic, casual, formal)" — offer multiple choice based on the role
- "Any quirks or catchphrases? Something that makes this agent feel like a person, not a tool." — open-ended, offer examples from the draft
- "How does [agent name] relate to [other agents]? Anything they'd agree or disagree on?" — only if team has 2+ agents

**Step 3: Define the contract.** Ask:
- "What should [agent name] read when dispatched? Specific files, directories, or whatever you provide at dispatch time?" — suggest sensible defaults based on the role (e.g., a code reviewer reads `src/`, a client reads user stories)
- "What format should their output be? (e.g., markdown report, code review comments, annotated file)" — multiple choice
- "Where should output go?" — default to `agents/shared/[agent-name]/`, confirm or override

**Step 4: Identify references.** The factory scans the project for docs that might be relevant (README, CLAUDE.md, docs/, specs/) and asks:
- "I found these docs in your project: [list]. Which ones should [agent name] have access to?" — checklist
- References are copied (not symlinked) into `agents/[agent-name]/references/` so the agent has a stable snapshot. User can update them later.

**Step 5: Confirm.** Present the complete agent setup (character sheet + contract + references) for final approval before moving to the next agent.

#### Phase 4: Scaffold
Generate all files:
1. Create `agents/` directory structure
2. Write team manifest (`team.json`)
3. Write each agent's files (character sheet, prompt, dispatch, contract)
4. Create memory directories with empty `MEMORY.md`
5. Set up `agents/shared/` output directory
6. Add permissions to `.claude/settings.local.json`
7. Present a summary of what was created

#### Phase 5: Test Drive (Optional)
Ask: "Would you like to test drive your agents now? I'll give each one a small task so you can see them in action and tune anything that feels off."

If yes, for each agent:
1. The factory picks a small, real task appropriate to the agent's role (e.g., The Client reviews the project's README as if they were a new user; The Critic reviews a single file)
2. Dispatch the agent via the Agent tool
3. Present the output to the user
4. Ask: "Does this feel right? Anything you'd change about how they communicate?"
5. If changes requested, update the character sheet and prompt, re-dispatch on the same task
6. Agent saves its first memories from the test drive

If no, skip — the agents are ready to dispatch manually whenever the user wants.

### /add-agent

Add a new agent to an existing team. Detects the existing team from `agents/team.json` in the current project.

**Flow:**
1. Read existing team manifest
2. Show current team composition
3. Ask: What gap are you trying to fill?
4. Propose 1-2 new agent candidates that complement the existing team
5. Deep-dive the chosen agent (same as Phase 3 above)
6. Scaffold the new agent's files
7. Update team manifest
8. Test drive

### /upgrade-agent

Add capabilities to an existing agent. Detects available agents from the team manifest.

**Available Upgrades:**

#### MCP Server
- Scaffolds a TypeScript MCP server from template
- Wires up `.mcp.json` registration
- Adds permissions to settings
- Use case: agent needs external API access (email, Slack, databases, etc.)

#### Cron Schedule
- Sets up a cron-based invocation schedule
- Configures what the agent does on each run
- Adds reporting (what happened, what was found)
- Use case: agent should run autonomously on a schedule (daily code review, morning triage, etc.)

#### Multi-Project Promotion
- Moves agent files from project-local to `~/.claude/agents/`
- Updates team manifests in all projects that use the agent
- Creates `projects.json` to track where the agent is active
- Use case: agent proved useful and should be available everywhere

## Dispatch Mechanism

Agents are invoked via Claude Code's Agent tool with the prompt file as context. The dispatch doc provides a template:

```markdown
# Dispatching [Agent Name]

## Quick Dispatch
Use the Agent tool with:
- **prompt:** Read `agents/[agent-name]/prompt.md` for your full instructions. [Task description here.]
- **subagent_type:** general-purpose
- **description:** [Short description for the agent list]

## Full Dispatch Template
```
Read agents/[agent-name]/prompt.md for your full instructions.

Your task: [describe the specific work]

Input: [paths to relevant files or context]

Write your output to: agents/shared/[agent-name]/[output-file]
```

## What to Provide
- [Specific inputs this agent needs per its contract]

## Example
[A concrete example of dispatching this agent]
```

### Complete Example: Dispatching The Client

```markdown
# Dispatching The Client

## Quick Dispatch
Use the Agent tool with:
- **prompt:** Read `agents/the-client/prompt.md` for your full instructions. Review the new onboarding flow I just implemented.
- **subagent_type:** general-purpose
- **description:** Client reviews onboarding flow

## Full Dispatch Template

Read agents/the-client/prompt.md for your full instructions.

Your task: Review the new user onboarding flow. Walk through it as if you're a first-time user who just signed up. Tell me what works, what's confusing, and what's missing.

Input: src/pages/onboarding/, src/components/onboarding/

Write your output to: agents/shared/the-client/2026-03-19-onboarding-review.md

## What to Provide
- Path to the feature or UI being reviewed
- Any requirements docs or user stories that define expected behavior
- Context about what changed (if reviewing an update vs. new feature)
```

## Interaction Models

### Parallel (Default)
Agents work independently. The user (or main thread) dispatches them as needed. No ordering constraints. Each agent reads from project files and writes to its own section of `agents/shared/`.

### Pipeline
Agents run in sequence. Each agent's output feeds the next. Defined in `team.json`'s `pipeline_order` array. The dispatch doc for each agent specifies what it expects from the previous agent.

**Example: Editorial Pipeline** (from the writing project pattern)

```json
{
  "interaction_model": "pipeline",
  "pipeline_order": ["the-editor", "the-tightener", "the-pattern-checker"],
  "agents": [
    {
      "name": "the-editor",
      "role": "Structural editing — pacing, arcs, scene function",
      "pipeline_position": 1,
      "dependencies": []
    },
    {
      "name": "the-tightener",
      "role": "Prose economy — cut needless words, strengthen constructions",
      "pipeline_position": 2,
      "dependencies": ["the-editor"]
    },
    {
      "name": "the-pattern-checker",
      "role": "Cross-document patterns — repeated tics, sensory ratios, consistency",
      "pipeline_position": 3,
      "dependencies": ["the-tightener"]
    }
  ]
}
```

In a pipeline, each agent reads the previous agent's output from `agents/shared/`. The dispatch doc specifies: "Read the report from [previous agent] at `agents/shared/[previous-agent]/` before starting your review."

### Collaborative
Agents can read each other's output in `agents/shared/`. One agent produces a report, another responds to it. The main thread orchestrates by dispatching agents in response to each other's work.

**Example: Design Review**
1. The Scout researches competing approaches, writes a report
2. The Critic reviews the Scout's report, flags concerns
3. The Client reads both and gives the final verdict from the user's perspective

No strict ordering — the main thread decides when to involve each agent based on the conversation.

## Permissions

The factory generates `.claude/settings.local.json` entries for each agent:

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

MCP-upgraded agents get additional permissions for their specific tools.

## Error Handling

- **No `agents/` directory:** Offer to create it (this is a new team)
- **Existing `team.json` found during `/create-agents`:** Warn and offer to either extend the existing team or start fresh
- **Agent name collision:** Append a number or ask for a different name
- **Missing references:** Warn during scaffold, agent still works but with gaps
- **Dispatch failure:** Agent reports what went wrong, suggests fixes
- **Memory directory issues:** Create if missing, warn if not writable

## What This Plugin Does NOT Include

- No web UI or dashboard
- No agent-to-agent real-time communication (they communicate via shared files)
- No billing or usage tracking
- No built-in LLM calls — agents run through Claude Code's existing infrastructure
- No Docker containers or process isolation — agents are prompt-based identities
- No automatic agent invocation — the user or main thread always initiates dispatch

## Marketplace Registration

When shipping, add an entry to `/nervous-marketplace/.claude-plugin/marketplace.json`:

```json
{
  "name": "agent-factory",
  "source": "./agent-factory",
  "description": "Create teams of persistent, personality-driven sub-agents for any project.",
  "version": "1.0.0",
  "keywords": ["agents", "subagents", "factory", "team", "mcp", "automation", "workflow"]
}
```

## Future Considerations (Not In MVP)

- **Agent marketplace** — share agent definitions with others
- **Team templates** — pre-built teams for common workflows (SaaS dev, content creation, open source maintenance)
- **Agent analytics** — track which agents get used, how their output quality evolves
- **Inter-agent messaging** — agents can request dispatch of other agents
- **Voice analysis** — like email-boi's `analyze-style`, but for tuning any agent's voice from examples
