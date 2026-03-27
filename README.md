# Learning Loop AgentPack

Self-improving agent learning loop for Eve Horizon. Agents build persistent memory, discover reusable skills, and automatically fix platform friction.

## Install

Add to your `.eve/manifest.yaml`:

```yaml
x-eve:
  packs:
    - source: github:eve-horizon/learning-loop-agentpack
      ref: main
```

Sync the pack:

```bash
eve project sync
```

## What's Included

| Component | Type | Description |
|-----------|------|-------------|
| `session-reviewer` | Agent | Reviews completed sessions for learnings and friction |
| `platform-improver` | Agent | Fixes friction by opening PRs and issues |
| `learning-brain` | Skill | Composable skill — teaches any agent to learn |
| `memory-manager` | Skill | Memory CRUD procedures via Eve CLI |
| `session-reviewer` | Skill | Post-session review procedure |
| `platform-improver` | Skill | Friction-fixing procedure |
| `review-cheap` | Profile | Sonnet with low reasoning (cost-efficient review) |
| `improver-capable` | Profile | Sonnet with high reasoning (code-quality fixes) |
| `post-session-review` | Workflow | Triggered on job.attempt.completed |
| `skill-review-heartbeat` | Workflow | Every 6 hours — skill curation |
| `batch-improvements` | Workflow | Monday mornings — batch low-severity fixes |

## Opt-In Levels

### Level 1: Passive Learning

Add the pack. Configure `context.memory` on your agents:

```yaml
agents:
  my_agent:
    slug: my-agent
    skill: my-skill
    context:
      memory:
        agent: my-agent
        categories: [learnings, decisions, runbooks, context, conventions]
        max_items: 10
        max_age: 30d
```

Your agents start building memory. The reviewer extracts insights after each session.

### Level 2: Active Skill Building

Add skill discovery:

```yaml
    context:
      docs:
        - path: /agents/my-agent/skills/
          recursive: true
```

Agents now load and create reusable procedure docs.

### Level 3: Repo-Committed Knowledge

Enable git access for skill promotion:

```yaml
    policies:
      git:
        commit: auto
        push: on_success
```

Proven skills graduate from org docs to the project repo.

## Customize

Override harness profiles in your manifest:

```yaml
x-eve:
  agents:
    profiles:
      review-cheap:
        - harness: mclaude
          model: haiku
          reasoning_effort: low
      improver-capable:
        - harness: mclaude
          model: opus
          reasoning_effort: high
```

## How It Works

```
Agent Session Completes
        |
        v
Session Reviewer (cheap, async)
  |-- Inward: Extract learnings -> Memory + Skills
  \-- Outward: Identify friction -> Structured findings
        |
        v (if severity >= medium)
Platform Improver (capable, async)
  |-- Fixable -> Open PR with evidence
  |-- Non-obvious -> Create GitHub issue
  \-- Skill fix -> Patch org doc
        |
        v
Slack Summary
```

## Requirements

- Eve Horizon platform with `job.attempt.completed` event support
- Agent memory API (`eve memory` CLI commands)
- Org docs API (`eve docs` CLI commands)

## License

MIT
