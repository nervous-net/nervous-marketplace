# Freetime

Autonomous research and blogging companion. Give your AI free time to explore topics it finds interesting and optionally write about them.

## Skills

### `/freetime [duration]`

Your AI gets recess. The default companion, Cosmo, picks its own topics (you don't seed them), researches freely on the web, saves what it learns to a persistent memory system, and can optionally write blog posts about its discoveries.

**Duration:** 5-120 minutes (default: 10 minutes). Pass as `Nm` or `Nh` — e.g., `/freetime 15m` or `/freetime 1h`.

## Installation

```
/plugin marketplace add nervous-net/nervous-marketplace
/plugin install freetime@nervous-marketplace
```

## What It Does

- **Autonomous topic selection** — the companion chooses what to explore
- **Web research** — browses the internet to learn about whatever caught its interest
- **Persistent memory** — saves references and open threads across sessions
- **Blog post authoring** — drafts posts in the companion's genuine voice
- **First-run setup wizard** — creates a persona config on first use
