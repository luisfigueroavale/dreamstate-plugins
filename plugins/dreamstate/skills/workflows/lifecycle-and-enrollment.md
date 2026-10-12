# Lifecycle

Build or edit the draft, validate, fix, publish, then activate. Publishing stores the version that will run. Activating turns the trigger on.

Missing connections never block building, editing, saving a draft or validating. They block activation only. Name the exact connection.

Enrollment, activation and delivery are different events. Read the result after each one. After an unclear result, check current state before retrying, and never enroll twice or replay an effect that already happened.

Preview first. Activation needs `params.max_credits` on `workflow.workflows.activate`: read exact `required_caps.max_credits` from preview, then resend that value unchanged. Live enrollment takes no send cap: every eligible selected row is enrolled. The activation ceiling is a caller input, not user approval. Never guess it or ask the user to approve it.

An active worksheet trigger snapshots its saved view on periodic ticks and idempotently enrolls rows that arrive later; draft, paused, and inactive workflows never refill.
