# Lifecycle

Build or edit the draft, validate, fix, publish, then activate. Publishing stores the version that will run. Activating turns the trigger on.

Missing connections never block building, editing, saving a draft or validating. They block activation only. Name the exact connection.

Enrollment, activation and delivery are different events. Read the result after each one. After an unclear result, check current state before retrying, and never enroll twice or replay an effect that already happened.

Preview first. Activation stores `params.max_credits` on `workflow.workflows.activate`: read exact `required_caps.max_credits` from preview, then resend that value unchanged. This persisted per-delivery ceiling is a declared domain input, not run `max_credits`; put run gating fields at door. Live enrollment takes no send cap: every eligible selected row is enrolled. Never guess it or ask user to approve it.

An active worksheet trigger snapshots its saved view on periodic ticks and idempotently enrolls rows that arrive later; draft, paused, and inactive workflows never refill.

"Enroll in a workflow" means the worksheet the user is looking at, or just built, and it is not a question to send back. Reuse a workflow only when its table trigger is that worksheet or the user named it. Otherwise compose a new draft for this audience: a worksheet rows trigger on that worksheet plus steps that fit what the list was built for (a follow-up task per person is a safe default), validate, then enroll the worksheet into it. Never enroll into another worksheet's workflow, and never stop to ask which workflow when none was named.
