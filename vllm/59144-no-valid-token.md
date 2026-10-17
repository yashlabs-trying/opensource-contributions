# vLLM #59144 — fail safely when no token is valid

## Problem and root cause

Combinations such as `allowed_token_ids` plus `bad_words` can eliminate every legal token. Once every logit is `-inf`, `argmax` still returns index 0. The engine therefore emitted an arbitrary, disallowed token instead of recognizing an impossible sampling state.

## Change

The patch detects rows where every logit is negative infinity after logit processors run. The mask is produced in CPU sampling, GPU/MRV2 sampling, and speculative rejection sampling; propagated through synchronous and asynchronous output bookkeeping and tensor-parallel batch sharding; converted to request IDs; and consumed by the scheduler. Only affected requests become `FINISHED_ERROR`, are marked non-resumable, and discard the garbage token. Other requests in the batch continue.

Static validation also rejects contradictions that can be proven from single-token bad words at step zero. Context-dependent multi-token bad words remain a runtime case.

## Boundaries

The condition is intentionally exact: all `-inf`. NaN rows, `+inf` rows, and rows containing at least one finite value are not reclassified. In speculative decoding, only the consumed prefix is considered.

## Validation

The PR records mixed-batch, constraint-conflict, NaN, positive-infinity, mixed finite/`-inf`, and all-`-inf` tests. It reports 42 existing sampler tests, 75 rejection-sampler tests, and 15 GPU-sampler tests passing, plus a manual Qwen3-0.6B end-to-end check where the impossible request ended with zero emitted tokens while the engine stayed alive.

## Learning

Low-level numerical functions can return a value for a state that is invalid at the product level. The correct fix requires an explicit error signal that survives every optimized execution and transport path.

