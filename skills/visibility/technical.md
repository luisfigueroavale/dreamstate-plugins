# Technical SEO

## Diagnose by layer

- Discovery: `visibility.workspace_site_get`, then `visibility.sitemap_get`. Check `sitemap.ok`. A false `ok` is `no_website`, `parse_failed`, or `fetch_failed`, not an empty sitemap.
- Crawl permission: `visibility.seo.robots_audit` returns `bots[]` with `allowed`, `blocked`, `not_applicable`, or `error`. It does not edit robots.txt.
- Fetch and render: `visibility.html_analysis_get` is the cached homepage. `analysis` null means no brand site. Refresh with `visibility.site_scan`.
- Indexability: directives and canonicals from that HTML analysis, plus `visibility.seo.agent_readiness_latest` for the last stored scan. `visibility.seo.agent_readiness_scan` hits the live site and follows the paid-run consent rule.
- Experience: `visibility.pagespeed_get`. `is_stale` is true when mobile or desktop is missing or older than 25 hours. `visibility.pagespeed_refresh` calls PageSpeed for one strategy and caches it. `visibility.site_scan` refreshes both strategies plus the HTML fetch and the site files.
- Search presentation: title, description, headings, and structured data on the fetched HTML. `visibility.seo.llms_txt_generate` stores a file and returns `generated_needs_publishing`. No catalog capability hosts that file at `/llms.txt`. Read the stored file with `visibility.seo.llms_txt_get`.

Read the specific layer that could explain the symptom. A clean robots file does not prove indexability; a fast page does not prove discoverability; an llms.txt file does not improve traditional rankings by itself.

For every issue retain URL, check id, observed time, evidence excerpt or metric, severity, affected scope, and confidence. De-duplicate symptoms that share one root cause. Rank fixes by impact and dependency so a sitewide canonical bug precedes individual title polish.

Scans and refreshes are durable operations. Do not claim completion from the start receipt. Read the terminal run and exact findings, and preserve partial, errored, or provider-bounded states rather than filling gaps with generic audit advice.
