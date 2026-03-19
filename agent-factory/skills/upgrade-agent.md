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
   ```
   "mcp__[agent-slug]__*"
   ```

7. Tell the user:
   "MCP server scaffolded at `agents/[agent-slug]/mcp-server/`.

   Next steps:
   1. `cd agents/[agent-slug]/mcp-server && npm install`
   2. Implement the TODO tools in `src/index.ts`
   3. `npm run build`
   4. The server will be available next time you start Claude Code in this project."

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
      "name": "[agent-slug]",
      "role": "[role]",
      "scope": "global",
      "path": "~/.claude/agents/[agent-slug]"
    }
    ```"

## Error Handling

- **Agent not found in manifest:** "I don't see an agent called [name] in your team. Available agents: [list]." Let the user pick again.
- **MCP server directory already exists:** "This agent already has an MCP server at `agents/[slug]/mcp-server/`. Want to overwrite it?" *Wait for response.*
- **Global agent directory already exists:** "There's already a global agent at `~/.claude/agents/[slug]/`. This might be from a different project. Want to see what's there before proceeding?" *Wait for response.*
- **Cron tool unavailable:** "The cron scheduling tool isn't available in your environment. You can set up a system cron manually — here's the command to run: [show the claude command]."

## Rules

- **One question at a time.** Never ask multiple questions in a single message.
- **Wait for responses.** Never skip ahead.
- **Never auto-commit.** Let the user decide when to commit.
- **Never overwrite without asking.** If a file exists, ask before replacing.
- **Read config tolerantly.** Missing fields use sensible defaults.
- **Templates are starting points.** Fill `{{placeholders}}` with specific content. Never leave template variables in generated files.
