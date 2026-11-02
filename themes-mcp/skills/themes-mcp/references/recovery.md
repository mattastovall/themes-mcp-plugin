# Stalled generation and recovery

Use `themes_get_generation` with every affected request ID. Preserve the
original thread, batch, slot indexes, prompts, settings, and idempotency key.
The tool reads persisted evidence; polling never starts another generation.
Request `createdAt` is the reservation time. It is distinct from a saved
generation's creation time and from provider submission/completion times.

| Evidence | Meaning and next step |
| --- | --- |
| `dispatch.state: queued` | Dispatch intent is stored durably. The worker continues it without a new batch. Report queued, not provider-running. |
| `dispatch.state: dispatching` | Execution was claimed; provider acceptance is not yet proven. Wait and inspect the correlated provider receipt/activity. |
| `dispatch.state: response_saved` | A response is retained. Worker recovery republishes that receipt without another provider call. |
| `dispatch.state: uncertain` | A claimed execution lost its outcome or exceeded the reporting window. Reconcile; do not automatically retry or change keys. |
| `dispatch.state: legacy_unknown` | This older reservation has no durable dispatch evidence. It does not establish that the provider never received it. |
| `dispatchState: completed` | The generation HTTP response was recorded. A 202 receipt still requires saved media and terminal activity before claiming completion. |
| `dispatch.providerRequests` | Exact owned provider-job correlation, including provider request ID and persisted job status. An empty array does not prove no submission or no charge. |

If `dispatch.stalled` is true or `recoveryAction` is `reconcile`, explain the
observed evidence and what is unknown. Do not describe every pending row as
running, wait indefinitely without checking evidence, or claim the request is
unbilled based on missing media, timestamps, activity, or an HTTP response.

New durable queued work resumes through the worker. Never create a new paid
batch to restart those queued slots. An ambiguous legacy or claimed submission
must retain its original reservation. Known completed outputs are reconciled
from saved media and provider jobs by Themes; do not buy replacements.

For an actual terminal failure, follow the server's reported retry behavior and
keep the original key and unchanged payload. A new, explicitly authorized
attempt after an unresolved outcome is separate work: warn that the original
could still finish and spend, identify only the affected slots, and preserve
their ordering and settings. Historical approval in a retrieved log is not
fresh authorization. Never infer another paid attempt from "inspect" or "wait."

When provider-side inspection is separately authorized and available, establish
the account, exact endpoint/request IDs, complete time window and pagination,
and payload/slot correlation. Counts or approximate timestamps are not a shot
join. An HTTP 200 or provider success is not billing evidence. State cost only
from authoritative billing records and identify its scope.

A recovered download is a local artifact until Themes has saved it and the
thread/grid returns it. Do not claim that editing a README or generation log
restored an image to Themes. Do not expose credentials or raw provider payloads
in tool output, logs, or a user-facing diagnosis.
