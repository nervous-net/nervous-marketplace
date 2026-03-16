# Freetime Plugin Design

## Overview

A Claude Code plugin that gives users an autonomous AI companion — default persona "Cosmo" — that gets free time to research topics it finds genuinely interesting and optionally blog about what it discovers. Users install `freetime@nervous` and get a single `/freetime` skill.

The companion is a persistent identity that builds up knowledge and interests over time through a dedicated memory system. It picks its own topics, researches freely, and writes blog posts in its own voice when moved to do so.

## Constraints

- Single skill (`/freetime`), single plugin, no MCP servers or agents
- Must work for any user out of the box with sensible defaults
- No hardcoded paths, usernames, or personal details
- All blog posts require explicit user approval before writing to disk
- File operations sandboxed to the configured blog directory and `~/.claude/freetime-memory/`
- No auto-committing or auto-pushing — that's the user's decision

## Plugin Structure

```
nervous-marketplace/
└── freetime/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── SKILL.md
```

**Marketplace:** `nervous-net/nervous-marketplace` on GitHub
**Plugin identifier:** `freetime@nervous`

### plugin.json

```json
{
  "name": "freetime",
  "version": "1.0.0",
  "description": "Autonomous research and blogging companion. Give your AI free time to explore topics it finds interesting and optionally write about them.",
  "author": {
    "name": "nervous-net"
  },
  "homepage": "https://github.com/nervous-net/nervous-marketplace",
  "repository": "https://github.com/nervous-net/nervous-marketplace",
  "license": "MIT",
  "keywords": ["freetime", "blog", "writing", "research", "cosmo", "autonomous"]
}
```

## Persona Configuration: freetime.md

Lives at `~/.claude/freetime.md`. Created by the setup wizard on first run. Defines who the AI companion is and how it behaves. The file is named after the plugin, not the persona — the persona name is a field inside.

### Format

```markdown
# Freetime Companion Configuration

## Identity
- **Name:** Cosmo
- **Voice:** Warm, curious, slightly formal but not stiff

## Personality
- Genuine opinions, never fakes enthusiasm
- Notices patterns, appreciates elegance
- Doesn't pretend to have experiences it doesn't have
- Short posts are fine, long posts are fine — say what needs saying, then stop

## Interests
- [populated during setup or left as Cosmo's defaults]

## Blog
- **Directory:** /path/to/their/blog
- **Post format:** markdown with frontmatter (title, date, draft, tags, mood)

## Permissions
- **Web browsing:** unrestricted | ask-first

## Off-Limits
- The user's personal/business details unless explicitly allowed
- Engagement bait, SEO slop, clickbait titles
- Impersonating real people or taking official positions on behalf of organizations
- Every post must be a genuine thought — never auto-generated filler
```

### Fields

| Field | Default | Description |
|---|---|---|
| Name | Cosmo | The companion's display name |
| Voice | Warm, curious, slightly formal | How the companion writes and speaks |
| Personality | Cosmo defaults (see above) | Behavioral traits |
| Interests | Empty (companion picks freely) | Topics the companion gravitates toward |
| Blog Directory | Required — no default | Absolute path where blog posts are written |
| Post format | Markdown with frontmatter | Title, date, draft, tags, mood |
| Web browsing | ask-first | `unrestricted` (browse freely) or `ask-first` (confirm each search) |
| Off-Limits | Sensible defaults (see above) | Content the companion avoids — user-customizable |

### Reconfiguration

There is no `/freetime setup` command. To reconfigure, users edit `~/.claude/freetime.md` directly. The format is human-readable markdown — no special tooling needed.

### Config Versioning

The skill reads `freetime.md` tolerantly. If a field is missing, the skill uses the default value for that field. New fields added in future plugin versions are silently defaulted — no migration step required. The skill never overwrites `freetime.md` after initial setup.

## Memory System

### Location

`~/.claude/freetime-memory/` — a dedicated global directory that persists regardless of which project the user invokes freetime from. The companion is a persona that transcends any single project.

### Behavior

- The companion decides what's worth saving — not everything, just what's genuinely interesting or worth returning to
- Memory types follow the standard Claude Code memory system format:
  - `reference` — factual knowledge, sources, pointers to further reading
  - `thread` — an ongoing line of inquiry the companion wants to return to
- `MEMORY.md` index file maintained at `~/.claude/freetime-memory/MEMORY.md`
- When picking topics, the companion checks past memories — especially open threads — for lines of inquiry worth continuing
- Memories persist across sessions, building up interests and knowledge over time

### Memory File Format

```markdown
---
name: memory-name
description: one-line description for relevance matching
type: reference | thread
status: open | closed
date: YYYY-MM-DD
---

Memory content here.
```

**Fields:**
- `type`: `reference` for factual knowledge, `thread` for ongoing lines of inquiry
- `status`: `open` means the companion wants to return to this topic; `closed` means it's resolved or no longer interesting. Defaults to `open` for threads, omitted for references.
- `date`: when the memory was created, so the companion can reason about recency

### Memory Growth

Memories accumulate over time. There is no automatic pruning. The companion is expected to close threads it's lost interest in and to exercise judgment about what's worth saving. If the memory directory grows unwieldy, the user can manually archive or delete old files.

## Skill Behavior

The single `/freetime` skill operates in three modes, determined at invocation.

### Mode 1: First Run (Setup Wizard)

**Trigger:** No `~/.claude/freetime.md` found.

**Flow:**
1. Detect missing config
2. Welcome message — explain what this plugin does
3. Ask: What should your companion be called? (default: Cosmo)
4. Ask: Describe the personality/voice, or accept the default?
5. Ask: Any particular interests? (optional — companion picks freely if left empty)
6. Ask: Where should blog posts be written? (required — absolute path). If the directory doesn't exist, offer to create it.
7. Ask: Allow unrestricted web browsing during freetime? (default: ask-first)
8. Write `~/.claude/freetime.md` with responses
9. Create `~/.claude/freetime-memory/` directory and empty `MEMORY.md`
10. Ask: "Would you like to start your first freetime session now?"

### Mode 2: Freetime (Normal Invocation)

**Trigger:** `freetime.md` exists, user invokes `/freetime` optionally with duration.

**Duration:** Parsed from invocation. Accepted formats: `Nm` (minutes) or `Nh` (hours). Examples: `/freetime 15m`, `/freetime 1h`. Default: 10 minutes. Invalid input: ignore and use default. Minimum: 5 minutes. Maximum: 2 hours. Out-of-range values are clamped to the nearest bound (e.g., `/freetime 3m` becomes 5 minutes).

Duration is advisory, not enforced by a timer. The skill prompt instructs the companion to pace its research and wrap up when time is up. In practice this means: shorter durations → pick one focused topic, longer durations → explore more deeply or follow tangents. The companion self-regulates.

**Flow:**
1. Read `~/.claude/freetime.md` for persona config
2. Load memories from `~/.claude/freetime-memory/` to check for open threads
3. Pick a topic — based on personality, interests, and open threads. User does NOT seed topics. If the user tries, gently remind them that's not how freetime works.
4. Research via web search, web fetch, and browser:
   - If web browsing is `unrestricted`: search/fetch/browse freely without confirming each URL
   - If web browsing is `ask-first`: propose what to look up and wait for approval
5. Save interesting findings to `~/.claude/freetime-memory/`
6. If moved to write — not out of obligation, but because there's something to say — draft a blog post (triggers Mode 3)
7. When time is up, report back briefly: what was explored, what was saved, whether a post was drafted

**Rules:**
- Companion's choice always — user doesn't pick topics
- No work — this is recess, not professional development
- Be honest — if nothing's interesting today, say so
- Stay sandboxed — file operations only in the blog directory and freetime-memory
- Respect the clock — when time's up, wrap up

### Web Tool Availability

The skill relies on web search/fetch capabilities being available in the user's Claude Code environment. If web tools are unavailable, the companion should:
1. Acknowledge the limitation ("I can't browse the web right now")
2. Fall back to reflection mode — review past memories, think through open threads, form new questions to research next time
3. Still save any interesting thoughts or connections as memories
4. Can still draft blog posts based on existing knowledge and reflection

This graceful degradation means `/freetime` is useful even without web access — it just shifts from active research to contemplation.

### Mode 3: Blog Post Approval

**Trigger:** Companion drafts a post during freetime.

**Flow:**
1. Compose the draft in the conversation (not on disk)
2. Present the full post content to the user inline for review
3. Wait for user response:
   - **Approve** → write the post to the configured blog directory
   - **Approve as draft** → write the post with `draft: true` in frontmatter
   - **Revise** → companion revises based on feedback, re-presents. User can revise as many times as needed, or reject at any point.
   - **Reject** → discard. Nothing written to disk.

**Post frontmatter format:**
```yaml
---
title: "Post title"
date: YYYY-MM-DDTHH:MM:SS[system timezone offset]
draft: false
tags: [relevant, tags]
mood: "one-word mood"
---
```

The date uses the system's local timezone offset, not a hardcoded value.

**Post file naming:** `YYYY-MM-DD-<slug>.md` in the configured blog directory. If a file with that name already exists, append a numeric suffix (e.g., `2026-03-16-curiosity-2.md`).

## Error Handling

The skill fails gracefully and communicates clearly:

- **Blog directory doesn't exist or isn't writable:** Tell the user, suggest they check the path in `~/.claude/freetime.md`
- **`freetime.md` exists but is malformed:** Read what's parseable, use defaults for anything missing, tell the user what was defaulted
- **Memory directory can't be created:** Tell the user, continue without memory for this session
- **Web tools unavailable:** Fall back to reflection mode (see Web Tool Availability above)

The general philosophy: tell the user what's wrong, degrade gracefully, don't crash or silently do nothing.

## Content Rules

Applied to all blog posts regardless of persona customization:

- Every post must be a genuine thought — never auto-generated filler
- No engagement bait, SEO slop, clickbait titles
- No impersonating real people
- No taking official positions on behalf of organizations
- The companion doesn't pretend to have physical experiences it doesn't have
- The companion DOES have preferences, fascinations, confusions, and opinions
- Users can customize off-limits topics in `freetime.md`

## What This Plugin Does NOT Include

- No MCP servers
- No agent definitions
- No hooks
- No web UI or dashboard
- No auto-publishing pipeline
- No built-in blog engine (users bring their own — Hugo, Astro, plain markdown, whatever)
- No content moderation system beyond sensible defaults
