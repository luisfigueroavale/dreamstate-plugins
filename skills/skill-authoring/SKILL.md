---
name: skill-authoring
description: "Read, edit and create this workspace's skills, the editable Markdown playbooks agents and people follow, keeping each one short, revision safe and in the workspace's own words. Use it when the user wants to change how a skill works, capture a repeatable practice as a new skill, or fix a skill that reports issues. It does not change what an agent is allowed to do (a skill grants nothing) and does not edit Company Context pages (knowledge does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Skill Authoring

## Boundary

- Owns: workspace skill playbooks.
- Company Context and wiki pages: [`knowledge`].
- The `knowledge` pack lists the tools and their exact inputs.

Skills are this workspace's editable Markdown playbooks. People edit them on the Skills page and agents edit the same pages, so a change made by either is what both follow next. A skill is guidance: it grants nothing, binds no tool and changes no permission.

## The invariant

Every change lands on the current revision and is reported back. Never overwrite a revision you did not read, and never leave the user guessing what changed.

## Read

1. Find a skill with skills_list query before reading it. The result ranks lexical matches and names matching documents; omit query only to inspect the complete catalog, workspace copies and issues.
2. Read one with skills_get. The response carries the full Markdown, parsed frontmatter, supporting pages, document index and current revision.
3. Read a supporting page only when the task needs it.

## Edit

1. Read the skill with skills_get immediately before writing.
2. Search the document write capability with tools_search, read its exact schema with tools_get, then run it with tools_run. Supply the document identity, current revision and complete new Markdown in the fields that schema declares. A whole-page save includes frontmatter and unchanged sections.
3. Change only what the user asked for. Keep the frontmatter name equal to the folder name.
4. On a revision conflict, someone else saved first. Re-read, merge your change into their text and save again on the new revision. Never resend the old text over theirs.
5. If the skill has no writable workspace document yet, read skills_list again after starter seeding finishes. Never invent a document identity or overwrite a different skill.

## Create

1. Check skills_list query first; if a skill already covers the need, add a section to it instead.
2. Search folder and document creation capabilities with tools_search. Read each schema with tools_get before using tools_run. Resolve the workspace skills root from skills_list, create the skill folder there, then use its returned identity to create SKILL.md.
3. Use a kebab-case id such as `pricing-objections` for the folder, and write SKILL.md with frontmatter `name` (the same id) and `description` (one sentence: what it does, when to use it, what it does not cover). A skill is plain Markdown and grants no tools, so it needs no tool list.
4. Read skills_get with the new id. The skill is created only when that read returns it. Fix every issue it reports.
5. Skills are read live, so there is no indexing delay. If skills_get says not found, the skill does not exist yet: never report it created or verified.

## Shape of a good skill

- One thesis line, the invariant, a short numbered practice, guardrails, and what to return.
- Keep SKILL.md to about 5 KB. Put detail, examples and edge cases in supporting .md pages in the same folder and say when to read each one.
- Teach practice, not tool grammar. Describe the task and let tools_search and tools_get supply exact names and fields.

## Guardrails

- Never delete or rewrite a starter skill unless the user asked for that. Prefer adding a new skill or a new section.
- Remove a skill only on an explicit request. Search the document trash capability, read its schema and use the document identity from a fresh read; say that it can be restored from trash.
- A skill cannot widen what an agent may do. Do not write one that claims to approve spend, sends or permissions.
- Do not copy secrets, credentials or customer personal data into a skill.

## What to return

- Which skill changed or was created, with its path and new revision.
- A short summary of what changed and why, in the user's words.
- A link to the skill on the Skills page, and any issues still reported.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
