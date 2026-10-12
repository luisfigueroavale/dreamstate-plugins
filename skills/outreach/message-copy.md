# Message copy

Preview first. Draft one full message for one real qualified prospect, subject line included for email, written in chat from that lead's own worksheet values, and show it with the personalized parts called out. A preview is text in the conversation: it needs no contact record, no enrollment and no send. When the user asked you to preview and then build, show the preview and keep going in the same turn; otherwise let them react, revise, and only then write every step.

Keep the scaffold fixed: greeting, the pitch paragraphs, the call to action, the sign-off and every paragraph break are the same for every lead. The personalized parts are short slots, such as first name, company, niche, target persona or one signal you can point to in the prospect's data.

Ground every personalized claim in the prospect's row or the workspace knowledge. If the data is empty, use the fallback wording, never a guess. Use only variables the workspace offers. An unresolved variable in a preview means the copy is not ready.

For each sending step, decide row variables first. Write fallback message before personalized message, for any empty row value. Put it in that step's `fallback_template` field; it is not its own step and not a branch. Fallback must not use row variables that can be empty, sender variables always present are allowed. Never publish a sending step with row variables and no fallback. Before publishing, render every step for at least three real rows, including one row with an empty value.

Merge fields are the enrolled worksheet's columns. Before copy greets someone by first name, check that the worksheet has a first name column. If it does not, add a visible `first_name` formula column with `FIRST_NAME({{name}})` through `tables.columns.add` (it runs no paid work), then write `{{first_name}}`. Derive any other short slot the copy needs the same way, as a visible formula column, before drafting.

Write in the workspace voice from its message writing notes. Short beats long: one idea per message, one clear ask. A follow-up refers back to the earlier touch instead of restarting the pitch. Each message still makes sense if it is the only one the person reads.

When the user asks to change one step, change only that step and leave the others untouched. Validate after every copy change.

## Personalize from the signal that found them

Every message should only make sense for this one lead. Name and company alone fit anyone, so each sending step carries the signal the list was built from:

- Hiring lists: say what they are hiring for. "Saw {{company}} is hiring an {{matched_job}} in San Francisco" beats "saw you're hiring". Use the job title column the source returned, and connect it to why your offer matters for that hire.
- People who posted about a need: refer to what they actually wrote, in a few words. "Your post about replacing your sales tool" beats "your recent post".
- Role or company lists (VPs of Sales at YC startups): use what is specific to them, such as their batch, what the company sells, its stage or team size, or something they shipped.

Make these slots AI variables when the raw value is long or messy. Add an AI column per slot that writes one short phrase from that row's evidence only, for example `hiring_angle` (the role and why it matters), `post_topic` (what the post asked for, in five to eight words) or `opener` (one sentence a human would write). Give it a fallback answer for rows without evidence. Run it on one pilot row, read the output, then run it on the qualified rows and reference the column in the steps as `{{hiring_angle}}`, `{{post_topic}}` or `{{opener}}`. Never paste a raw headline or a whole post into a sentence, and greet with `{{first_name}}`, never the full name.

Use conditions to keep the wrong people out and the right copy in. Filter enrollment on the signal and the evidence the copy needs: every Required criterion column passes, a title family matched with contains or in (never one exact title string), the signal column is set (the job title, the post topic, the opener). When different groups need different angles (founders versus sales leaders, hiring versus posting), filter each group into its own sequence or saved view rather than one message that tries to fit both.

Each follow-up brings a new reason to reply (a relevant result, a short resource, an easy question), not "following up" on the first message. Read every step side by side before publishing: if two steps say the same thing, rewrite the later one. If a step would read the same for any company, it is not done.

## Check the copy like a reviewer before publishing

Before publishing or enrolling, preview at least three different real rows, including one with the thinnest evidence, and judge each rendered message as pass, tighten or block:
- Block: an unresolved or empty variable, a claim the row does not support, a fact about the lead that is wrong, or anything that reveals how they were found.
- Tighten: the message would read the same for another company, the opener is generic, a follow-up repeats an earlier step, the ask is unclear, or it runs long.
- Pass: specific to this person, grounded in their row, one clear ask.
If the rendered messages across rows read nearly identical apart from the name, the signal slot is too thin: strengthen the evidence column or the AI step prompt, not one row's copy. Fix every block and tighten, preview again, and say in the report what you checked.

## A/B testing

When a step writes with AI, it can test variants: vary either the angle (the overall approach) or the opener (only the first line), not both, and keep each variant a genuinely different idea rather than a paraphrase. Turn on automatic winner promotion when the user wants the sequence to keep the better variant on its own. Read the results per variant before changing copy, and change the losing variant, not the whole sequence.
