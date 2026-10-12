---
name: visibility
description: "Get found by search engines and answer engines. Pick the query set, gather crawl, index, mention and citation evidence, fix pages, then measure the change."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run` (`dry_run` first for anything that writes, spends credits or reaches a person). If the server asks you to sign in, approve it in the browser.

# Visibility

## Boundary

- Owns: choosing queries and measuring search and answer-engine visibility.
- Drafting and publishing articles: [`writing`].
- The `visibility` pack lists the tools and their exact inputs.

<!-- architect-operation-contract
{"required_capability_ids":[]}
-->

Improve discoverability and qualified organic traffic in traditional search and in generative answer engines, as one practice. This skill owns search evidence, crawlability, page structure, technical readiness, tracked prompts and citation measurement. It hands evidence-backed page briefs to `writing`, which authors the content.

## Operating contract

```operating-contract
actions: []
```

Read measured evidence, propose prioritized changes, and treat paid or asynchronous measurement as bounded work: show scope and cost, then run it with credits as consent. Before a sensitive change, show the evidence and the exact change. Knowledge owns business facts; writing owns content changes.

## Loop

1. Resolve the exact site, then read brand, audience, conversion goal and competitors.
2. Check the search connection before using its data, and keep property, date range, freshness and completeness with every number. See search-evidence.md.
3. For a named page, inspect that page now; never assume an earlier audit still describes it. See technical.md.
4. Before adding prompts, read the tracked query set and its status. See query-sets.md.
5. Start a new probe only when the user wants new engine observations; a metric refresh recomputes existing ones. See probes.md.
6. Prioritize by business impact, effort, confidence and dependency, and state every assumption.

## Evidence rules

- An observation is what a capability returned for a stated window; everything else is inference. Say which is which.
- Movement claims compare a window with the preceding equal-length window. Missing rows do not prove rank zero.
- Each technical check answers its own question; one passing check is never an overall health claim.
- A scan or refresh may spend credits or run asynchronously. Wait for its terminal state: a queued scan is not a result. Refresh only when existing evidence is too stale for the decision.
- Crawler fetches, AI referral traffic and probe recommendations are separate evidence; never fold them into one claim. Absent crawler evidence is not proof of blocking.
- A generated proposal for a site file is a reviewable draft; generating it neither installs nor proves it was read.

## Exact denominators

Prompt visibility is citations or qualifying mentions divided by completed runs for that exact prompt, engine set, locale and window. Report run count, citation count and percentage together. Zero citations over zero runs is unavailable, not 0%. Weight averages by their denominators and never merge engines, locales or windows silently. Sentiment uses its own total. Citation share, mention rate and referral traffic are distinct and never one score.

The tracked query set is a revisioned instrument: changing a prompt's wording breaks the comparison, so change it deliberately and disclose the break.

## Handoff to writing

A brief names the target query and intent, the evidence window and property, current and previous performance, the destination page or a new page, observed competing results, the conversion goal, internal links, and evidence gaps. When an existing page owns the intent, request a revision instead of a duplicate.

## Done

One report covers search and answer engines: site and property, window, freshness and completeness, measured findings, assumptions, prioritized actions, query set revision, engines covered, completed versus expected runs, denominators, and the weakest prompts by name. Never claim an improvement until a later equal-scope measurement shows it. When blocked, say what is missing, what was already measured, and the one read or decision that unblocks. Read back what you changed before reporting.
