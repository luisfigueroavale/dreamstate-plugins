---
name: skill-authoring
description: "Read, edit and create this workspace's skills, the editable Markdown playbooks agents and people follow, keeping each one short, revision safe and in the workspace's own words. Use it when the user wants to change how a skill works, capture a repeatable practice as a new skill, or fix a skill that reports issues. It does not change what an agent is allowed to do (a skill grants nothing) and does not edit Company Context pages (knowledge does)."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs the user's yes before you send approved: true; never send it on your own. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` is a param of the external-send sequence capabilities (`sequence.sequences.activate_audience`, `sequence.sequences.resume`, and the rest of that family), sent inside `params`, not a gating input of `tools_run`. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows, dollars, credits, and the conversion. If the server asks you to sign in, approve it in the browser.

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
2. Follow its `edit` descriptor: read the named tool's schema with tools_get, then run it with tools_run, passing the document, the revision from that read and the complete new Markdown in the named content field. The save replaces the whole page, so send the full text, frontmatter included.
3. Change only what the user asked for. Keep the frontmatter name equal to the folder name.
4. On a revision conflict, someone else saved first. Re-read, merge your change into their text and save again on the new revision. Never resend the old text over theirs.
5. When `edit` is null an existing starter skill is still a repo default for this workspace. Read skills_list again shortly; the workspace copy appears once seeding finishes. This applies only to starter skills, never to a skill you just created.

## Create

1. Check skills_list query first; if a skill already covers the need, add a section to it instead.
2. Follow the `create` descriptor from skills_list, step by step, with tools_get then tools_run for each step. Send every param the descriptor fills in exactly as given. The first step names `parent_folder_id` explicitly and returns the new folder id that the second step needs.
3. Use a kebab-case id such as `pricing-objections` for the folder, and write SKILL.md with frontmatter `name` (the same id) and `description` (one sentence: what it does, when to use it, what it does not cover). A skill is plain Markdown and grants no tools, so it needs no tool list.
4. Run the descriptor's `verify` read: skills_get with the new id. The skill is created only when that read returns it. Fix every issue it reports.
5. Skills are read live, so there is no indexing delay. If skills_get says not found, the skill does not exist yet: never report it created or verified.

## Shape of a good skill

- One thesis line, the invariant, a short numbered practice, guardrails, and what to return.
- Keep SKILL.md to about 5 KB. Put detail, examples and edge cases in supporting .md pages in the same folder and say when to read each one.
- Teach practice, not tool grammar. Describe the task and let tools_search and tools_get supply exact names and fields.

## Guardrails

- Never delete or rewrite a starter skill unless the user asked for that. Prefer adding a new skill or a new section.
- Remove a skill only on an explicit request, following its `remove` descriptor; say that it can be restored from trash.
- A skill cannot widen what an agent may do. Do not write one that claims to approve spend, sends or permissions.
- Do not copy secrets, credentials or customer personal data into a skill.

## What to return

- Which skill changed or was created, with its path and new revision.
- A short summary of what changed and why, in the user's words.
- A link to the skill on the Skills page, and any issues still reported.

## Check it works

Done means checked, not built. Before you report, read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
