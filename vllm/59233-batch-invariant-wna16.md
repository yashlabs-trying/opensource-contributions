# vLLM #59233 — keep WNA16 out of batch-invariant mode

## Problem and root cause

`VLLM_BATCH_INVARIANT` promises invariant execution, but W4A16 MoE dispatch could still choose the CUDA `moe_wna16_gemm` path. Its split-K partial sums use FP16/BF16 `atomicAdd`; CUDA block completion order is not fixed, so otherwise identical executions can accumulate in a different order and produce different output bits.

## Change

When batch-invariant mode is enabled, `should_moe_wna16_use_cuda()` no longer selects the CUDA WNA16 kernel. Dispatch falls through to the existing Triton implementation. With the flag disabled, normal kernel eligibility and performance behavior are unchanged.

## Validation

The regression test verifies that CUDA WNA16 remains eligible in normal mode and is disabled in invariant mode. CUDA platform detection is mocked, so the dispatch contract can be tested without depending on a particular machine.

## Process note

This PR supersedes closed PR #59199, which combined the fix with unrelated Harmony changes. Splitting the work produced a focused one-commit, two-file change that is easier to review.

## Learning

Determinism can be enforced at dispatch when an existing deterministic implementation is available. A feature contract should constrain backend selection explicitly instead of relying on users to know which fast kernels violate it.

