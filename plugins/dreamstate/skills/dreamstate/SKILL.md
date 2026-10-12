---
name: dreamstate
description: "Use Dreamstate for sales and marketing work: finding companies or people, enrichment and scoring, LinkedIn and email outreach sequences, inbox replies, LinkedIn and X posts, CRM records, workflows and AI search visibility."
---

# Dreamstate

Dreamstate is reached through its MCP server, which exposes thirteen tools.

1. `whoami` names the user and the active workspace. If it lists several workspaces, confirm the right one before writing anything.
2. `packs_route` maps a goal to a domain; `packs_get` returns that domain's capabilities and skills.
3. Prefer the narrower `dreamstate:*` skill for the job when one fits. `skills_get` returns the workspace's current copy of any skill.
4. `tools_search` with the goal, `tools_get` for the exact schema, then `tools_run` with `capability_id` and `params`. Read the schema every time.
5. Follow long work with `runs_get`, `runs_events` and `runs_list`; `runs_cancel` stops a run.

Lists, finds, enrichments and signals happen only inside a workbook worksheet. Anything that writes, spends credits or reaches a real person runs `dry_run` first, then live with `max_credits` and an `idempotency_key`. Report only what tools returned.
