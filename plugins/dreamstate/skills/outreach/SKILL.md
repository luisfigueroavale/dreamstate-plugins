---
name: outreach
description: "Build a multi-channel sequence and enroll a workbook into it: preview one real message first, fixed scaffold with small personalized slots, channel steps with the wait inside each step, column filters decide who enrolls, paced sends."
---

Run this through the Dreamstate MCP tools: `packs_get` for the pack's capabilities, `tools_get` for the exact input schema, then `tools_run` (`dry_run` first for anything that writes, spends credits or reaches a person). If the server asks you to sign in, approve it in the browser.

# Sequence building

## Boundary

Owns: new conversations, multi-step outreach sequences, and enrollment.
- Existing-thread replies: [`inbox-replies`].
- Building and qualifying who to contact: [`list-building`].
- Warm-source rows and send caps: [`linkedin-warm-signals`].
- Automations that are not outreach sequences: [`workflows`].
- Explaining why a sequence has not sent: [`diagnostics`].
The `sequence` pack lists the tools and their exact inputs.

How to build a multi-channel sequence and enroll a workbook into it. The sequence pack lists the tools and their exact inputs; finding and qualifying the people belongs to the workbook building skill.

## Delete and archive

When user says delete, delete permanently. Show impact first unless user already asked to delete. Archive only when user says archive. Never say hard deletion is unavailable.

## Before writing

- Build a new sequence for each request. "Enroll in a sequence" means a new one. Use or edit an existing sequence only when the user names it or asks to change it; never ask the user to pick an existing sequence.
- Read the ideal customer profile, the product and offer pages, and the message writing notes before asking anything. The value proposition comes from those pages; ask for it only when none of them states it. Ask one question only when a required fact is missing, then build.
- Before writing any sequence copy, load the knowledge skill and read saved Company Context pages that govern it: Product Information and Offer for claims, Ideal Customer for audience fit, and Tone of Voice for register. Name pages used. Pass returned document ids as `context_document_ids` and returned revision ids as `context_revision_ids` on the create-from-worksheet call, so the draft receipt proves exact context lineage. Never reuse another workspace's product copy. If Company Context has no applicable page, say so and ask for the missing fact before writing prose.
- The kind of sequence is a required fact. Unless the request already says it, read selected saved view's structural channel reachability, then ask once, in one question, which reachable channel and goal (a reply, a meeting, a demo). Offer only channels with reachable rows and recommend channel most rows support.
- Use only channels the rows can be reached on. A channel step needs the field it sends to; when that field is missing, find it first with an enrichment.

## Copy

- Every message carries the signal that found this lead, read from the worksheet column that holds it. Bring a signal that lives on a linked row onto the enrolled worksheet with a lookup column. When no column holds a usable signal, add one that writes it first.
- The signal is about them, never about how you found them.
- Fixed scaffold, small slots: greeting, pitch, call to action and sign-off are the same for everyone; only the slots change. Write the scaffold so every value reads correctly, and never invent a fact to fill a slot.
- Greet by first name. The enrolled worksheet's columns are the variables, so add a first name column when it is missing.
- Every sending step that uses row variables carries its own fallback copy, written first, that reads well with every variable empty. Never publish a step without it.
- Follow-ups give a new reason to reply, never a "just following up". Keep them linear: a reply ends the lead's run by itself, so never add steps that check for one.
- Preview at least three real rows, one with thin evidence. Read each like the recipient and fix the scaffold, not one lead. Grade and tighten with message-copy.md.

## Steps

When asked what sequences you can build or which templates exist, list the sequence blueprints and the available step types and answer from those; never answer from memory.

Use a sequence blueprint when one does exactly what you need (its channels and steps match the request); otherwise build your own sequence. A sequence does not need a worksheet: create it on its own and set its enrollment source or trigger later.

When the user asks to enroll a list:

0. Read the workbook's worksheets and enroll the people worksheet (its Qualified view), even when another worksheet is open. Only when no worksheet holds people, fan out the person records first.

1. Read selected saved view's channel reachability, then ask which kind of sequence (see Before writing), offering only channels with reachable rows: LinkedIn (connect, then message) when LinkedIn rows exist, email when verified work-email rows exist, both only when both counts are nonzero. Recommend channel most rows support.
2. List the sequence blueprints before creating anything, and install the one that matches that channel mix, if one does, giving it the worksheet column that holds the signal (add the lookup first when the signal lives on a linked worksheet), with the list's Qualified view as its enrollment source; otherwise build the steps yourself. It lands as a draft that never sends, with the steps, waits and AI writing already set.
3. Write each step's copy for this request with the row's columns as variables (first name, the signal column, a short AI variable column when the raw value is long), following message-copy.md. A message whose only variable is the first name has no signal: when the signal lives on another worksheet (such as the company's open role), add the lookup column first. Keep the blueprint's follow-up steps unless the user asked for one message. Preview real rows and fix the scaffold.
4. Validate, enroll from the Qualified view and report. Never start it unless the user says start. When the Qualified view is empty, still build and validate the sequence bound to that view, preview its copy on a few worksheet rows, and say that rows enroll as they qualify and what would qualify them; never enroll a wider view in its place unless the user asks for it.

- Read each step's schema for what it already does, such as connecting or waiting before it sends, and never add a separate step for what it handles.
- Validate after every change. Never call a sequence ready until validation is clean and the saved sequence shows every requested step.

## Audience

- The workbook is the audience. Filter enrollment on qualified rows (every Required criterion column passes) and on the columns the copy needs being set. Match role families, never one exact title. Without criterion columns, add them from the ideal customer profile, run them, read them, then filter.
- A sequence can have unlimited ordered workbook audiences. Add first workbook, worksheet/tab, and saved view, then set that audience's filter, email and LinkedIn URL identity columns, daily enrollment cap, and priority sort. Add another workbook only when it needs its own audience settings. Earlier audience wins when same person appears in more than one audience.
- Keep filters, identity mapping, cap, and priority inside each audience. Never apply one global filter or sort across several workbooks. Use a workflow source only when entry needs logic a workbook audience cannot express.
- When several people from one company are enrolled, turn on company pause on reply.

## State

- Building never waits on connections or rows. Build, preview, validate and bind anyway, then say what to connect or what will make rows land.
- Publishing and enrolling never send. After enrolling, leave the sequence published and never started unless the user explicitly says start; that is paused. Staged contacts on an unstarted sequence are success.
- Drafted, validated, published, enrolled, started, delivered and replied are separate facts; report only what the last result proves. After an unknown outcome, read state before retrying, and never enroll or send twice.

## Done

Read back everything you built; an empty or wrong result means an input or setting is wrong, so fix it and check again. Report the sequence link, its steps and channels, the enrollment filter, enrolled and held back counts, and that nothing was sent.
