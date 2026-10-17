# Open-source contributions

This repository is a public engineering record of five active upstream pull requests: four to [vLLM](https://github.com/vllm-project/vllm) and one to [SGLang](https://github.com/sgl-project/sglang). It documents the bugs, root causes, fixes, validation, limitations, and reusable lessons behind each contribution.

Status was checked on **2026-10-17**. “Open” means submitted upstream and still under review; it does not mean merged or accepted.

## Contribution index

| Project | Pull request | Problem solved | Validation recorded in the PR | Status |
|---|---|---|---|---|
| vLLM | [#59028](https://github.com/vllm-project/vllm/pull/59028) | Mooncake state was deleted when a preempted request was re-admitted in the same scheduling step | 47 Mooncake tests + 25 multi-connector tests; Ruff and diff checks passed | Open, mergeable |
| vLLM | [#59144](https://github.com/vllm-project/vllm/pull/59144) | All-`-inf` logits silently selected token 0 when constraints eliminated every token | 42 sampler + 75 rejection-sampler + 15 GPU-sampler tests; manual Qwen3-0.6B end-to-end check | Open; GitHub reported non-mergeable at snapshot |
| vLLM | [#59225](https://github.com/vllm-project/vllm/pull/59225) | A malformed Harmony recipient containing `<|channel|>` became a fake tool call | 15 normalization cases plus full parse/streaming cases; 64 Harmony parser tests passed | Open, mergeable |
| vLLM | [#59233](https://github.com/vllm-project/vllm/pull/59233) | The CUDA WNA16 MoE kernel violated batch-invariant execution through order-dependent atomics | Machine-independent dispatch regression test; normal-mode behavior preserved | Open, mergeable |
| SGLang | [#41472](https://github.com/sgl-project/sglang/pull/41472) | `request.is_disconnected()` could leak `CancelledError` through the ASGI boundary | Focused disconnect and cancellation regression coverage | Open, mergeable |

The superseded vLLM PR [#59199](https://github.com/vllm-project/vllm/pull/59199) is part of the process history, but is not counted as a sixth contribution because #59233 replaced it.

## Detailed records

- [vLLM contribution overview](vllm/README.md)
  - [Mooncake re-admission cleanup](vllm/59028-mooncake-readmission.md)
  - [No-valid-token sampling failure](vllm/59144-no-valid-token.md)
  - [Harmony recipient normalization](vllm/59225-harmony-recipient.md)
  - [Batch-invariant WNA16 dispatch](vllm/59233-batch-invariant-wna16.md)
- [SGLang contribution overview](sglang/README.md)
  - [Client-disconnect cancellation handling](sglang/41472-client-disconnect.md)
- [Cross-project learnings](LEARNINGS.md)

## What the work covers

These are correctness fixes at system boundaries rather than isolated style changes:

- scheduler state across preemption and immediate re-admission;
- constrained sampling when the legal token set becomes empty;
- streaming parser recovery for malformed model control tokens;
- deterministic kernel dispatch under an explicit invariance contract; and
- cancellation semantics at an HTTP client-disconnect boundary.

Across the five contributions, the common method was to reproduce the narrow failure, identify the violated invariant, make the smallest scoped change, add regression coverage for the failure and its neighboring cases, and state what the patch does **not** solve.

## Evidence policy

- Upstream PRs are the source of truth for current status, review, and CI.
- Test counts here are the results reported in each PR, not a claim that every upstream job is currently green.
- Open PRs are not described as accepted contributions.
- Known unrelated or environment-specific failures are preserved in the detailed notes instead of being hidden.
- No upstream source tree is copied into this repository; this is documentation and learning material with links to the actual changes.

