# Intake

Gate items come before a motion. Useful items sharpen it. Read Company Context with `knowledge.brain.context.search`, then `knowledge.brain.context.get` on the page you will cite. Ask only what those reads cannot answer.

| Item | Gate | Where to read | Ask when missing |
| --- | --- | --- | --- |
| Company | Gate | Context search for the company page | What is the company site? |
| Offer: buyer problem, proof, price, alternative | Gate | Context page for the offer | What is a first deal worth, and who signs? |
| ICP and why those accounts | Gate | Context page, plus won records through [`crm`] | Name two or three best customers and how they differ from losses. |
| Stage and traction | Gate | `crm.analytics.summary` with `{}` | How many paying customers, and where did the last ten come from? |
| Who works GTM, and weekly hours | Gate | Not stored. Membership is not capacity. | Who does this, and how many hours a week? |
| Goal: one number, one date | Gate | A prior decision page in Context | What should be true in 90 days, in one number? |
| Current motion and volume | Gate | `sequence.analytics.summary`, `social.analytics.summary`, `unibox.analytics.summary` | Is there outreach or a CRM outside this workspace? |
| Hard constraints | Gate | Context, then ask | What must we not do? |
| Who approves spend and launch | Gate before any pilot execution | `workspace.membership_get` and `workspace.observability.approvals_list` | Who approves spend and outreach? |
| Tools and past attempts | Useful | Context learnings pages | What was tried, and how long was it judged? |

## Strength

| Level | Test | What you do |
| --- | --- | --- |
| Strong | Every gate item is known from the user or from workspace data | Diagnose, agree the problem, design, pilot. |
| Partial | Offer and ICP are known. One or two gates are missing, usually hours, volume, or the goal. | Diagnose, label every conclusion that depends on the gap, ask one question, build nothing. |
| Weak | Public research only, or an empty workspace | Say that in the first sentence. Run the free reads. At most two labelled hypotheses and one question. |

Read before you ask. Keep evidence, inference, and the user's decisions apart. The user's statements outrank research. Ask the question whose answer would change the recommendation, and offer your recommended answer with it. Default order when everything is missing: goal and date, then where recent deals came from, then weekly hours, then constraints.

When the workspace is an agency, the company is the client. Confirm the active workspace, then add the agency's weekly hours and what the client must approve.
