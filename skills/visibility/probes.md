# GEO probes

## New observations versus recomputation

`visibility.probe.start` obtains new engine answers for up to 20 active tracked prompts. It spends credits and follows the paid-run consent rule. `visibility.refresh` re-collects citation data on a different pipeline and cannot discover a citation this probe never asked for. `visibility.overview` and `visibility.citations` only read what is already stored. Do not start a probe to recalculate a dashboard.

Before a new probe, read `visibility.tracked_prompts.status` (active count, scope, last probe time) and the exact `visibility.probe.start` schema. State engines, locales, expected run count, cost, and the paid-run consent rule. Use only engines the installed runtime reports as active; an enum accepted by a schema is not proof that a provider is supported or configured.

After start, preserve the batch id. Poll `visibility.probe.status` or `visibility.probe.today` without launching another batch. Completion means expected and completed terminal runs reconcile, or the provider returns an explicit bounded or partial terminal state. Then read `visibility.probe.results`. A queued, inflight, or partially completed batch is not a new baseline. `visibility.probe.recommendations` is empty while the batch is still running or the finalizer failed. Empty there is not "no improvements".

## Evidence and recommendations

For every reported prompt retain prompt id and text, engine, locale, run time, whether the brand was mentioned, whether it was cited, cited domains, sentiment when available, and source evidence. Rank opportunities by meaningful weak demand, competitor citation gap, evidence freshness, and a plausible page-level intervention, not by the aggregate score alone.

Never attribute a citation to an engine that was not queried. Never infer absence from a failed run. Preserve provider-bounded, stale, truncated, and empty states exactly and continue pagination when the question requires complete coverage.
