# vLLM contributions

Four active pull requests address correctness in scheduling, sampling, protocol parsing, and quantized-kernel dispatch.

| PR | Area | Core invariant | Scope |
|---|---|---|---|
| [#59028](https://github.com/vllm-project/vllm/pull/59028) | Mooncake KV connector | State rebuilt for a request re-admitted this step must not be removed by stale preemption metadata | Local filtering inside Mooncake; global scheduler semantics unchanged |
| [#59144](https://github.com/vllm-project/vllm/pull/59144) | Sampling and scheduler | A request with no legal token must fail explicitly, never emit an arbitrary token | CPU, GPU/MRV2, speculative decode, async bookkeeping, TP transport, scheduler termination |
| [#59225](https://github.com/vllm-project/vllm/pull/59225) | GPT-OSS Harmony parser | Control markers embedded in a recipient must not turn assistant output into a tool call | Recipient normalization plus full and incremental parser tests |
| [#59233](https://github.com/vllm-project/vllm/pull/59233) | W4A16 MoE dispatch | Batch-invariant mode may select only invariant implementations | Dispatch-only guard; default-mode performance and eligibility unchanged |

## Development pattern

The work starts from user-visible failures and traces them to the smallest violated invariant. Fixes are deliberately placed at the narrowest ownership boundary: Mooncake-specific cleanup, sampler output propagation, parser normalization, or kernel eligibility. Tests include both the failure and a control case that must remain unchanged.

## Status note

As of 2026-11-23 all four PRs are open and none is merged. #59028 was rebased cleanly onto current `main` on 2026-11-24 (same 207-line change). #59144 still carries `needs-rebase`: upstream deleted `vllm/v1/sample/sampler.py` and `rejection_sampler.py` in a sampler refactor, so the fix has to be ported to the new code path. #59225 is labelled `bug`/`gpt-oss`/`tool-calling` and awaits maintainer review. #59233 was last checked on 2026-10-17, when GitHub reported it mergeable.

