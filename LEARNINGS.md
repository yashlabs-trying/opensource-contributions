# Cross-project learnings

## 1. Preserve state-machine invariants across lifecycle boundaries

The Mooncake and SGLang fixes look different, but both occur where one lifecycle overlaps another. A request can be preempted and re-admitted in one scheduler output; an ASGI receive probe can be cancelled even though the outer task should continue. In both cases, the fix must distinguish a stale/local event from a real terminal event.

Practical rule: write down which component owns state, when it is created, when it may be destroyed, and which identifiers can appear in multiple event sets during one step.

## 2. Invalid numerical states need explicit control-flow semantics

`argmax` is defined mechanically even when every logit is `-inf`, but token 0 is not a semantically valid answer. Numerical kernels and sampling code need explicit checks for states that are mathematically representable yet invalid for the API contract.

Practical rule: detect the exact invalid condition, carry it through asynchronous and sharded paths, terminate only affected requests, and test mixed batches so healthy requests continue.

## 3. Normalize protocol noise before classification

The Harmony parser bug came from classifying a recipient before removing a control marker that could be embedded by malformed model output. Normalization belongs at the parser boundary, before deciding whether text names a function.

Practical rule: normalize only known protocol markers, preserve valid function recipients, and test both full parsing and incremental streaming.

## 4. Determinism is a dispatch contract, not only a kernel property

The WNA16 issue did not require rewriting the CUDA kernel. In batch-invariant mode, the safest fix was to prevent dispatch to an implementation whose split-K atomic reduction is order-dependent, while leaving default performance behavior unchanged.

Practical rule: a feature flag promising invariance must constrain every eligible backend. A slower deterministic fallback is preferable to silently violating the contract.

## 5. Cancellation is data and control flow at the same time

`asyncio.CancelledError` can signal either a local receive-path teardown or a genuine request to cancel the current task. Catching all cancellation would make shutdown and cancellation unreliable; catching none leaks a routine disconnect into the server boundary.

Practical rule: catch cancellation at the narrowest operation, inspect the current task’s cancellation state, and re-raise real outer-task cancellation.

## 6. Regression tests should protect neighbors, not only the reported case

The strongest tests in these changes cover paired cases:

- re-admitted requests are preserved while genuinely preempted requests are still cleaned up;
- all-`-inf` rows fail while NaN, `+inf`, and mixed finite rows retain their existing semantics;
- malformed assistant recipients stop becoming tools while real function recipients remain tools;
- invariant mode disables the nondeterministic backend while normal mode remains eligible; and
- disconnect-probe cancellation is contained while real task cancellation propagates.

## 7. Scope statements build trust

Each PR records limits: no CUDA-kernel rewrite, no accuracy benchmark where model math is unchanged, no claim that unrelated device failures were fixed, and no claim of upstream acceptance before merge. Precise non-claims make the actual result easier to review.

