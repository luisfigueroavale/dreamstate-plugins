# GEO query sets

## Construct from real demand, not a generic prompt list

Build candidate prompts from distinct evidence streams: product/category language, ICP jobs and objections, Search Console queries already gathered elsewhere in this skill, competitor comparison questions, and high-intent buyer decisions. Record the source for each prompt. Balance prompt types across discovery, problem understanding, category education, comparison, evaluation, and purchase intent; a set made only of branded prompts measures recall, not market visibility.

Every prompt must be natural enough that a real buyer could ask it, specific enough to have a stable intent, and scoped to language and country. Avoid near-duplicates whose only difference is punctuation or a synonym. Include unbranded prompts that give every credible competitor a fair chance to appear.

## Prepare, review, save

`visibility.tracked_prompts.list` and `visibility.tracked_prompts.status` are the current set. `visibility.keywords_get` is Search Console. `state` `not_connected` or `auth_expired` means not measured, not zero traffic. `visibility.geo_queries_list` is a different table from tracked prompts.

`visibility.tracked_prompts.generate` returns candidate drafts and saves nothing. It spends credits and follows the paid-run consent rule. Review candidates for duplicates, leading brand bias, unsupported locales, and coverage gaps. `visibility.tracked_prompts.create` adds one prompt and leaves the rest. `visibility.tracked_prompts.update` revises one. `visibility.tracked_prompts.save` replaces the set: prompts you omit are deactivated, and inactive prompts that were never probed are deleted. Use `save` only when that replacement is the request. Read back with `visibility.tracked_prompts.list` and report the stable ids and active count. Tags that start with `cat:` are dropped on create.

Do not generate a new set because metrics are weak. Weak results may be the finding. Revise the measurement instrument only when audience, market, language, product, or question taxonomy changed, and record the reason and effective date.

## Version and comparison rules

Treat additions, removals, activation changes, wording changes, locale changes, and engine coverage changes as query-set revisions. Compare trends only across the unchanged intersection or label the result as a new baseline. Never splice old results for retired wording into a new prompt's history.
