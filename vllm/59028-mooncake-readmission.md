# vLLM #59028 — Mooncake state cleanup for re-admitted requests

## Problem and root cause

`reset_prefix_cache(reset_running_requests=True)` can preempt a request outside the normal `schedule()` path. The next scheduling step may immediately re-admit the same request, so its ID can appear both among newly scheduled/resumed requests and in `preempted_req_ids`.

Mooncake rebuilt fresh state for the scheduled request and then processed the stale preemption entry, deleting the state it had just created. That could cause V2 assertions, V1 `NoneType` failures, loss of a fresh asynchronous `load_spec`, and worker-side deletion of recreated KV-store state.

## Change

The connector collects request IDs re-admitted in the current scheduler output from `scheduled_new_reqs` and resumed cached requests. It excludes those IDs from Mooncake preemption cleanup for that step. Requests that remain genuinely preempted are still cleaned normally, and global `SchedulerOutput.preempted_req_ids` semantics are not changed.

## Validation

Regression cases cover scheduled-new overlap, resumed overlap, a mixed re-admitted/genuinely-preempted step, preservation and consumption of a rebuilt `load_spec`, unchanged normal cleanup, and worker cleanup receiving only real preemptions. The PR reports 47 Mooncake tests and 25 relevant multi-connector tests passing, plus Ruff and `git diff --check`. One unrelated existing no-GPU device failure was recorded rather than attributed to this patch.

## Learning

A scheduler output is not necessarily a set of disjoint lifecycle events. Cleanup should use the final state transition for the current step, not blindly replay every stale event list in sequence.

