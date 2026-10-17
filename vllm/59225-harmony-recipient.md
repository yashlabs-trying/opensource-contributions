# vLLM #59225 — normalize malformed Harmony recipients

## Problem and root cause

GPT-OSS can emit a malformed Harmony header whose recipient contains a channel marker, for example `assistant<|channel|>analysis`. The parser passed that string to function-recipient classification. Because the decorated name was not recognized as `assistant`, ordinary reasoning could be surfaced as a fake tool call.

## Change

The existing recipient normalizer already handled embedded `<|constrain|>`. The patch extends the same boundary to strip an embedded `<|channel|>` before classification. `assistant<|channel|>analysis` normalizes to `assistant`, while a real recipient such as `functions.get_weather<|channel|>commentary` still normalizes to the function name and remains a tool call.

## Validation

Fifteen parametrized normalization cases cover each marker and their combination. End-to-end tests exercise both full `parse()` and incremental `parse_delta()` with the reported malformed value. The PR reports 64 Harmony parser tests passing. It also records 26 pre-existing device errors in an unrelated test class rather than presenting them as regressions from this patch.

## Learning

Parser recovery should remove only known transport/control syntax, then apply the existing semantic classifier. Testing both batch and streaming paths prevents a fix from working only after the whole message is buffered.

