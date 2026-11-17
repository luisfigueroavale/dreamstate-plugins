---
name: setup
description: "Set up a workspace end to end, agent first: read the knowledge index, write Company Context from what the agent knows plus the website and a Known, Unknown and Assumed page, draft the Ideal Customer, write and apply the score rubric, build a first workbook of about ten warm-first leads, draft a first sequence without sending, hand over account connect links, and recommend three next plays. Use it on a new or thin workspace or when the user says get me started or set me up."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run`. Anything that writes, spends credits or reaches a person: dry_run first, then live with the quote_hash the dry_run returned, a max_credits ceiling and an idempotency_key. max_credits 0 is valid only when that quote is 0. A larger paid run, or a live external send above the server's recipient threshold, also needs approved: true with the same quote_hash, after reviewing the preview. Free formulas need no paid-run approval; row execution caps still apply. `max_rows` is a param of `tables.columns.run` and `tables.table_sources.run`, not of `tools_run`. `max_live_sends` goes inside `params` only when the fetched capability schema declares it (for example, a one-contact workflow run). Sequence start, resume, launch and enrollment do not accept it; they have no lifetime send ceiling. The paid-run consent rule is one sentence in the MCP server instructions, rendered from consentRule.json and PAID_RUN_APPROVAL_THRESHOLDS: rows and credits, with no provider prices. If the server asks you to sign in, approve it in the browser.

# Setup

## Boundary

- Owns: first-run setup of a workspace, from context to a launch-ready draft.
- Older installs that ask for first win: [`first-win`].
- The `knowledge` pack lists the tools and their exact inputs.

After setup, Company Context pages belong to [`knowledge`], larger lists and new sources to [`list-building`], sequences and launch to [`outreach`], record and score work to [`crm`], and picking the next play to [`plays`].

## The invariant

The agent does the setup. Never tell the user to fill a page, set criteria or click through settings themselves. Ask only what changes the next action, offer concrete choices, and keep going. Spend nothing that is not quoted and approved under the paid-run consent rule, and send nothing: setup ends with a draft sequence and a launch-ready plan, not a launch.

Every answer ends with the next step and what it costs or needs approved.

## When to use it, and when not

| The request or state | Do this |
|---|---|
| "Set me up", "get me started", a fresh workspace, or whoami's first next step is "set up the workspace" | Run this skill from the first unfinished rung. |
| A set-up workspace and one named task (a list, a sequence, a post) | Load the skill that owns the task; do not rerun setup. |
| "Why is pipeline flat?" or "plan our GTM" | [`gtm-consultant`] diagnoses first. |
| A named proven play | [`plays`]. |
| Change the score rubric after setup | [`crm`], only when the user asks. |

## Where you are

Setup is a ladder. Read where the workspace stands before doing anything, then start at the first unfinished rung. Running setup again on a finished workspace is safe: every step reads first and updates instead of duplicating.

1. `whoami`: identity, the active workspace and its readiness. With more than one workspace, confirm which one before writing anything.
2. `workspace.activation.checklist_get`: six items, in setup order: company knowledge pages, the Ideal Customer and score rubric, a first workbook, a first sequence, a connected sending account, a launched sequence. Each item has done (true, false, or null when its probe failed; null is unknown, never missing) and a link.
3. `knowledge.context.resolve` with purpose setup: the knowledge index first (every page with its path, status and last update, and which standard pages are present, thin or missing), then the profile and Ideal Customer summary, the score rubric state, the activation state and next steps. When it is unavailable, page `knowledge.brain.knowledge.index` instead. Read the index before searching or creating any page.
4. `knowledge.first_win.execute` (limit 10, a 30 day window): offer, Ideal Customer and voice excerpts with document ids, ranked warm evidence, blueprint recommendations and the exact next calls, in one read. Read single sources only to inspect a returned row.

Run independent reads together. State in one sentence what you are checking, then check; never make the user wait on a dossier.

## 1. Company Context from what you already know

Start from what the agent already has before any web research:

- What this conversation, your memory and the user's own files already say: in a coding agent, read the starting folder (docs, pricing notes, customer lists, CRM or LinkedIn exports). Never upload a local file into Dreamstate without asking.
- The pages the index already holds (`knowledge.brain.context.get` for each relevant page).
- The company website, read now with `visibility.research.urls_fetch`, and direct alternatives found with `visibility.research.web_search`.

Write or update four pages in one composition with `knowledge.brain.context.compose_pages`: Product Information, Ideal Customer, Competitor Analysis and Tone of Voice. An existing page is updated from the revision you just read, never created twice. Every researched fact keeps its source and date; every inference is marked as one. A claim needs a number or a named customer behind it, otherwise call it unproven.

Then write a fifth page, Known, Unknown and Assumed, in the same folder:

- Known: confirmed facts and where each came from (the user, a file, the site).
- Unknown: what setup needed and could not find, each with the question that would settle it.
- Assumed: hypotheses in use until confirmed, such as the Ideal Customer draft, with what would change if each is wrong.

Keep that page current for the rest of setup: when the user confirms or corrects something, move it from Assumed to Known in the same turn. Read each page back and report its path and revision from the write receipt.

## 2. Ideal Customer as a hypothesis

In the first answer, say in two or three lines what the company sells, to whom and why it wins, and state the Ideal Customer in one line as a hypothesis. Invite corrections without waiting for them. Ask one question only when the Ideal Customer has two readings that would land on different people. Save what the user confirms with `knowledge.profile.update` and the Ideal Customer page.

## 3. Score rubric (once, here)

The score turns the Ideal Customer into grades the whole workspace uses: today's leads, worksheet admission and CRM sorting all read it. Setup is where it gets written; after setup it changes only when the user asks.

1. Read the current rubric with `crm.score_rubric.get`.
2. Draft criteria from the Ideal Customer: each one is a record attribute, an operator, a value and a weight, marked Required (a failed Required criterion grades F) or Nice to have (grades A to D come from the share met). Keep Required for true disqualifiers; most criteria are Nice to have. An unknown value is left out of the share and never counts as a miss.
3. Keep the default signal weights unless the user's motion clearly differs, for example a content-led company weighting post engagement higher.
4. Preview with `crm.score_rubric.set` and apply false. Show the grade distribution over current records and what drives it. A workspace with no records yet shows no grades; say so and apply anyway.
5. With the user's yes, save with apply true. Saving queues a free rescore of every person and company; `crm.scores.recompute` reruns it on demand.
6. Show one real record's grade with `crm.records.score_explain` so the user sees the rubric working.

Scoring is deterministic and costs zero credits.

## 4. First workbook, warm first

Land about ten people the user could contact this week, warm before cold:

1. Warm rungs first, in this order, from the first-win bundle: an active relationship or open deal, an interested reply nobody answered, recent CRM evidence, LinkedIn engagement, an existing qualified worksheet row. Never claim a warm rung without returned evidence. Reuse rows that already exist where they are: a filtered view on their worksheet, or `tables.rows.send` to copy chosen rows. Never retype them into a new sheet.
2. Install the list blueprint the bundle recommends (`workspace.blueprints.describe`, then `workspace.blueprints.apply`, then `workspace.blueprints.configure` for its source inputs), or reuse a fitting worksheet.
3. Only when warm rows cannot fill the set, run the free preview `tables.sourcing.preview` on the scarcest Ideal Customer criterion. Inspect filter coverage: only applied Required filters qualify a row; unverified or unsupported ones mean possible fit. Then run at most about ten rows from that preview with `tables.table_sources.run` under the paid-run consent rule.
4. Add the Required and Nice to have criterion columns, the evidence column and a first-message draft column with `tables.columns.add`, price them, then run them on the sample with `tables.columns.run`. Criteria and evidence that need the web are `column.research` or `column.web_agent` columns; never type a verdict or evidence into a cell.
5. Read the rows back. Every row shows its warmth rung, one evidence reason, one fit reason and a drafted first message. If fewer than half of cold rows look right, change the source filter once and preview again; never add columns to hide a poor source.

No contact-data lookups yet: they come after the user agrees these are the right people.

## 5. First sequence, drafted not sent

From the first workbook, create the first sequence for the channel the user prefers. Without a stated preference, pick from evidence: LinkedIn when the warm rows are LinkedIn engagement or the buyers are senior and reachable there, email when the rows carry verified work emails and a mailbox is connected, and LinkedIn when neither is clear: `sequence.sequences.create_from_worksheet` stages the selected rows, then `sequence.sequences.validate`, then `sequence.outreach.message_preview` for one real lead. Ground every step in the offer and in why these people fit. A new LinkedIn account starts with a few invites and messages a day. Load the `outreach` skill for step design. Nothing starts in setup.

## 6. Accounts

A missing account never blocks building. Check what is connected with `senders.social_accounts.list`, then hand the user the link for what the sequence needs: `senders.connect.start` returns a hosted LinkedIn sign-in link, and `integrations.connect_start` returns the settings link for a mailbox, CRM or other provider. Say what each account is for in one line.

## 7. Close

Read `workspace.activation.checklist_get` again and report what is done. Recommend three next plays from the first-win bundle's blueprint recommendations (with why now, unmet prerequisites and credit posture), recommend one, and load `plays` to run it when the user picks.

## Worked example

User: "Set me up."

1. `whoami` shows one workspace, no score rubric; `workspace.activation.checklist_get` shows nothing done; `knowledge.context.resolve` with purpose setup shows every standard page missing.
2. The agent reads the user's repo README and pricing page plus the website, then `knowledge.brain.context.compose_pages` writes the four pages and Known, Unknown and Assumed (Unknown: average deal size; Assumed: buyers are heads of sales at 20 to 200 person B2B SaaS companies). It replies with the three-line summary and the one-line Ideal Customer hypothesis.
3. `crm.score_rubric.get` returns revision null and the default grade cutoffs and timing. `crm.score_rubric.set` with `apply` false and `rubric` built from those defaults plus criteria such as:
   ```json
   { "criteria": [
       { "id": "software", "label": "B2B software", "subject": "company", "attr_key": "industry", "operator": "in", "value": ["Software", "SaaS"], "required": false, "weight": 3 },
       { "id": "size", "label": "11 to 200 people", "subject": "company", "attr_key": "employee_range", "operator": "in", "value": ["11-50", "51-200"], "required": false, "weight": 2 },
       { "id": "market", "label": "Target market", "subject": "company", "attr_key": "hq_country", "operator": "in", "value": ["US", "CA"], "required": true },
       { "id": "role", "label": "Sales leader", "subject": "person", "attr_key": "title", "operator": "contains", "value": "sales", "required": false, "weight": 3 } ],
     "grade_cutoffs": "<from get>", "timing": "<from get>" }
   ```
   The attribute keys come from the workspace's own record attributes: read them with `crm.record_objects.attributes_list` before writing a criterion, never guess one. The preview shows no records yet; the user says yes; the same call with `apply` true and `expected_revision` from the get saves revision 1.
4. The bundle finds three engaged LinkedIn people and no CRM deals; `tables.sourcing.preview` fills the rest; ten rows land with fit reasons and drafts at a quoted, approved sample cost.
5. `sequence.sequences.create_from_worksheet`, `sequence.sequences.validate` and one `sequence.outreach.message_preview` produce a draft LinkedIn sequence. `senders.connect.start` returns the LinkedIn sign-in link.
6. Close: checklist shows leads and a first sequence done; three plays offered, one recommended.

## If it goes wrong

- A website that will not load or research that returns nothing: say so in one line, write what the user and their files already say, list the gap under Unknown, and continue.
- A page create refused because it exists: update that page from its current revision instead.
- A rubric preview with every record at No data: the criteria name attributes the records do not have yet. Keep the criteria, apply, and note that grades appear as enrichment fills those attributes.
- A source preview that does not look like the buyers: tighten once (role, an exclusion, a narrower size) and preview again; after a second miss, ask one question about the buyer.
- A refusal that names a plan, credits or approval: report it in one line with the exact next action from the refusal.
- A connect link that expires or a sign-in that does not finish: call `senders.connect.start` again for a fresh link (with `reconnect` true and the account id for an existing account), then confirm with `senders.social_accounts.list`. Keep building meanwhile; only launch waits for the account.
- A checklist item that reads null: its probe failed. Say it is unknown, read that part directly (for example `senders.social_accounts.list` for the account), and never report it as missing.
- No warm evidence and an Ideal Customer the user has not confirmed: preview two narrow sources that test the two readings of the hypothesis, show both samples side by side, and ask which group is right before running either.

## What to return

- Workspace confirmed.
- Company Context pages and Known, Unknown and Assumed created or updated, each with path and revision.
- The Ideal Customer hypothesis and what the user confirmed.
- The saved rubric revision, the grade distribution and one explained record.
- Workbook link with about ten ranked people, warmth rung, evidence and fit reasons, drafts, and credits charged.
- The draft sequence link and its preview, and the account links handed over.
- The activation checklist state and three plays, one recommended.

## Check it works

Done means checked, not built. Read back everything you made or changed, and run, test or preview it where that is offered; publish or send only when the user asked. An empty, failed or wrong result means an input or setting is wrong: find which from the error or the output, correct it and check again, until it is right or a real blocker stops it. Report what you checked and what it showed.

Paid and outward runs follow the paid-run consent rule.
