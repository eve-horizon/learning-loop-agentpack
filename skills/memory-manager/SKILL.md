---
name: memory-manager
description: Procedures for agent memory CRUD — reading, writing, searching, and managing memory entries via Eve CLI and API
---

# Memory Manager

Procedures for managing agent memory using Eve's existing CLI and API surface.

## Reading Memory

### List all memory for an agent
```bash
eve memory list --org $EVE_ORG_ID --agent {slug} --json
```

### List by category
```bash
eve memory list --org $EVE_ORG_ID --agent {slug} --category learnings --json
```

### List by tag
```bash
eve memory list --org $EVE_ORG_ID --agent {slug} --tags networking,k8s --json
```

### Get a specific entry
```bash
eve memory get --org $EVE_ORG_ID --agent {slug} --category learnings --key {key} --json
```

### Search memory
```bash
eve memory search --org $EVE_ORG_ID --agent {slug} --query "docker networking" --limit 10 --json
```

### Unified search (across memory, docs, threads)
```bash
eve search --org $EVE_ORG_ID --query "deployment pattern" --json
```

## Writing Memory

### Set a new entry
```bash
eve memory set --org $EVE_ORG_ID --agent {slug} --category learnings --key k8s-networking \
  --content "Pods on this cluster often fail first on DNS before CNI." \
  --tags k8s,networking --confidence 0.8 --review-in 14d
```

### Set a shared convention (visible to all agents)
```bash
eve memory set --org $EVE_ORG_ID --shared --category conventions --key api-style \
  --content "All API responses use { data: T, meta: { page, total } } envelope." \
  --confidence 0.9
```

### Update an existing entry (supersede)
```bash
eve memory set --org $EVE_ORG_ID --agent {slug} --category learnings --key k8s-networking \
  --content "Updated: DNS resolution is now reliable after CoreDNS upgrade." \
  --supersedes k8s-networking --confidence 0.9
```

### Set a user preference
```bash
eve memory set --org $EVE_ORG_ID --agent {slug} --category user --key communication-style \
  --content "User prefers terse output. No color codes. No emojis. Direct answers only." \
  --confidence 0.95
```

## Managing Skills

### Write a new skill doc
```bash
eve docs write --org $EVE_ORG_ID \
  --path /agents/{slug}/skills/debug-k8s-networking.md \
  --content "$(cat <<'SKILL'
# Debug K8s Networking

1. Check pod DNS resolution: \`kubectl exec -it {pod} -- nslookup kubernetes.default\`
2. Check CNI status: \`kubectl get pods -n kube-system -l k8s-app=calico-node\`
3. Check network policies: \`kubectl get networkpolicies -A\`
4. Test pod-to-pod connectivity: \`kubectl exec -it {pod} -- curl -s http://{target-svc}:8080/health\`
5. Check service endpoints: \`kubectl get endpoints {svc}\`
SKILL
)" \
  --metadata '{"kind":"skill","confidence":0.6,"used_count":0}'
```

### Patch an existing skill
```bash
eve docs patch --org $EVE_ORG_ID \
  --path /agents/{slug}/skills/debug-k8s-networking.md \
  --op replace --search "Check CNI status" \
  --replace "Check CNI status (Calico or Cilium)"
```

### Search skills
```bash
eve docs search --org $EVE_ORG_ID --prefix /agents/{slug}/skills/ --query "networking" --json
```

## KV Store (Ephemeral State)

### Set a value with TTL
```bash
eve kv set --org $EVE_ORG_ID --agent {slug} --namespace session --key last-task \
  --value '{"type":"debug","target":"api-gateway"}' --ttl 3600
```

### Get a value
```bash
eve kv get --org $EVE_ORG_ID --agent {slug} --namespace session --key last-task --json
```

## Lifecycle Management

### List entries due for review
```bash
eve docs stale --org $EVE_ORG_ID --prefix /agents/{slug}/memory/ --json
```

### Mark reviewed and schedule next review
```bash
eve docs review --org $EVE_ORG_ID --path /agents/{slug}/memory/learnings/k8s-networking.md \
  --next-review 14d
```

### Delete stale entry
```bash
eve memory delete --org $EVE_ORG_ID --agent {slug} --category learnings --key outdated-fact
```

## Capacity Guidelines

- **Memory entries**: Keep total under ~2,200 characters across all categories
- **Individual entries**: 1-3 sentences. One fact per entry.
- **Skills**: 500-5,000 tokens each. Longer is fine for complex procedures.
- **Review cadence**: Set `--review-in 14d` for learnings, `30d` for conventions, `7d` for context
- **Confidence thresholds**: 0.5 = uncertain, 0.7 = confident, 0.9 = established fact
