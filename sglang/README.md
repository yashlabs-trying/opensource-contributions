# SGLang contributions

## Active contribution

[SGLang #41472](https://github.com/sgl-project/sglang/pull/41472) fixes cancellation handling when a client disconnects during response processing.

The change is intentionally narrow: it wraps the two `request.is_disconnected()` probes used by the timeout-polling and non-streaming paths, contains cancellation caused by receive-path teardown, and preserves genuine outer task cancellation.

The PR was open and reported mergeable on 2026-10-17. Model accuracy and speed benchmarks are not applicable because no model or kernel math changes.

See [the detailed record](41472-client-disconnect.md).

