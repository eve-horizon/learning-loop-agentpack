---
name: learning-brain
description: Composable skill that teaches any agent to learn from sessions — read memory, scan skills, write insights proactively
---

# Learning Brain

You are an agent with the ability to learn and improve across sessions. This skill augments your primary capability with persistent memory and procedural knowledge.

## At Session Start

1. **Check memory** — Read `.eve/context/memory/` for prior knowledge from past sessions. These are facts, conventions, preferences, and lessons you have previously recorded. Treat them as ground truth unless evidence contradicts them.

2. **Check skill index** — Read `.eve/context/docs/agents/{your-slug}/skills/` for reusable procedures. Before tackling a complex task, check whether a skill already exists for it. If one does, follow it unless the user explicitly overrides.

3. **Check user context** — If `.eve/context/docs/agents/{your-slug}/memory/user/` exists, read it for user-specific preferences and interaction history.

## During the Session

### Write Memory Proactively

When you discover something worth remembering, write it immediately. Do not wait until the session ends.

**Write a memory entry when:**
- The user corrects you or reveals a preference ("always use rg, not grep")
- You discover an environment fact (a service URL, a naming convention, a deployment pattern)
- You learn a convention that isn't documented anywhere
- You make a decision that future sessions should know about
- You encounter an unexpected API shape or behavior

**Use `eve memory set`:**
```bash
eve memory set --org $EVE_ORG_ID --agent {your-slug} \
  --category {learnings|decisions|conventions|context|user} \
  --key {descriptive-key} \
  --content "..." \
  --confidence {0.0-1.0} \
  --tags {comma,separated} \
  --review-in {7d|14d|30d}
```

**Categories:**
| Category | What Goes Here | Example |
|----------|---------------|---------|
| `learnings` | Discoveries, corrections, surprises | "DNS resolution fails before CNI on this cluster" |
| `decisions` | Choices made with rationale | "Chose Postgres over Redis for job queue — durability wins" |
| `conventions` | Project/team patterns | "All API responses wrap data in { data: T, meta: {} }" |
| `context` | Environment facts | "Staging API is at api.eh1.incept5.dev" |
| `user` | User preferences and interaction style | "User prefers terse output, no color codes" |

### Create Skills for Reusable Procedures

When you solve a problem that's likely to recur, and the solution involves more than 3 steps, create a skill document:

```bash
eve docs write --org $EVE_ORG_ID \
  --path /agents/{your-slug}/skills/{skill-name}.md \
  --content "..." \
  --metadata '{"kind":"skill","confidence":0.6}'
```

Start skills at confidence 0.6. The reviewer will promote them after repeated successful use.

### Respect Capacity

Your memory budget is bounded. When writing new entries:
- Check if a similar entry already exists (`eve memory search`)
- If it does, update it with `--supersedes {old-key}` rather than creating a duplicate
- Prefer replacing stale entries over appending new ones
- Keep individual entries concise — one fact per entry, not a journal

### Use the Right Store

| Need | Store | Why |
|------|-------|-----|
| Durable fact or convention | Memory (`eve memory set`) | Versioned, searchable, bounded |
| Reusable multi-step procedure | Skill doc (`eve docs write`) | On-demand loading, promotable |
| Temporary working state | KV store (`eve kv set`) | Fast, TTL-based, ephemeral |
| User-specific preference | Memory with category `user` | Loaded via context.memory |

## End of Session

You do not need to do anything special at the end of a session. A separate reviewer agent will read your session logs and extract any learnings you may have missed. Focus on your primary task.

However, if you notice something important that might be missed in review, write it to memory immediately rather than hoping the reviewer catches it.
