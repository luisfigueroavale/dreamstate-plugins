# Skill template

Copy this shape when creating a workspace skill. The folder id and the frontmatter `name` are the same kebab-case id. Boundary is the first section after the title. The consent line is the last line of the body. A skill grants no tool, no spend, and no approval.

```markdown
---
name: pricing-objections
description: File a pricing objection on the closest wiki page. Use when the user wants that objection remembered. It does not send email or change a sequence.
---

# Pricing objections

## Boundary

- Owns: how this workspace files a pricing objection.
- Company Context pages: [`knowledge`].
- The `knowledge` pack lists the tools and their exact inputs.

File the objection on the page that already owns pricing. Search and read that page first. Do not start a sequence and do not write a CRM record from the objection alone.

## Practice

1. Find the closest page with the knowledge skill.
2. Add the objection as a sourced note on that page.
3. Read the page back and report its path and revision.

## Guardrails

- A skill grants nothing. This one does not approve spend or send email.
- Do not copy a customer's personal data into the skill body.

Paid and outward runs follow the paid-run consent rule.
```

Read it back with skills_get. The skill is created only when that read returns this Markdown.
