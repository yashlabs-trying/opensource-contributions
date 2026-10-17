# SGLang #41472 — client-disconnect cancellation handling

## Problem and root cause

`request.is_disconnected()` may raise `asyncio.CancelledError` when the ASGI receive path is torn down after a client disconnect. The exception could escape `_stream_one_response` and reach the HTTP/ASGI request boundary.

Simply swallowing every `CancelledError` would be incorrect because cancellation also carries real control flow: server shutdown or an explicit cancellation of the current task must still propagate.

## Change

The patch adds a narrow `TokenizerManager._is_disconnected` helper around the probe. If the current task has a pending cancellation request, it re-raises. Otherwise it treats the receive-probe cancellation as a disconnected client. The two direct probes in timeout polling and non-streaming output now use the helper.

## Validation and scope

Focused regression tests cover disconnect-probe cancellation and preservation of genuine task cancellation. Accuracy and performance benchmarks are marked not applicable because the patch does not modify a model, scheduler policy, or kernel. The PR’s recorded CI snapshot still showed failing upstream jobs, so this repository does not claim a fully green upstream run.

## Learning

Cancellation must be handled at the narrowest boundary where its source can be interpreted. The correct distinction is not “exception versus no exception,” but “local transport teardown versus cancellation of the operation that owns the task.”

