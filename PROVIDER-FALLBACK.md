# Provider Fallback Chain

## The idea

One provider will fail. It might be down, rate-limited, out of quota, or
its key expired. A fallback chain means your work continues on the next
provider instead of stopping.

```
request → provider 1 → success? done.
          └─ fail → provider 2 → success? done.
                     └─ fail → provider 3 → success? done.
                                └─ fail → error to the operator.
```

If all providers fail, the agent errors. Nothing auto-migrates — the
operator (you) does the work or fixes the config.

## How to order providers

Order by what you measured, not by what you assume:

1. **Latency** — time the same query on each provider. The fastest
   reliable one goes first.
2. **Cost** — record price per 1k tokens (input and output separately).
   A cheap provider that is slow can be a good fallback.
3. **Limits** — free tiers usually cap requests per day. A free provider
   is a fallback, not a primary, unless you measured it holding up.
4. **Quality** — run the same task on each provider and compare the
   output. A provider that is fast and cheap but wrong is not a fallback.

Measure all four before choosing an order. Re-measure monthly — pricing
and limits change without notice.

## What counts as a failure

- HTTP 429 / 5xx, timeouts, connection errors.
- Authentication errors (wrong or expired key) — but fix the key; do not
  silently rely on the fallback forever.
- Empty or malformed responses.
- A response that is clearly a different model than the one requested
  (some endpoints serve a different model than the name implies — verify
  by behavior, not by label).

## Rules

- **Do not hide failures.** Log which provider answered each request, so
  a degraded primary is visible instead of silently bypassed.
- **Do not chain endlessly.** Three providers is enough. More than that
  hides the real problem.
- **Free-tier providers are fallbacks.** They cap requests per day and
  can disappear. Never build a workflow that only works while a free
  tier is generous.
- **Judge-only models are not chat models.** Some endpoints expose a
  model for judging only — using it as a chat model will fail or return
  nonsense. Check the provider's docs for the intended use.
- **When the chain is exhausted, stop and report.** Do not retry the
  same request in a loop.

## Cost and latency notes

Keep a small table (in your own notes, not in git) per provider:

| Provider | Model | Latency (same query) | Cost / 1k in | Cost / 1k out | Daily limit |
| --- | --- | --- | --- | --- | --- |
| primary | … | … | … | … | … |
| fallback-1 | … | … | … | … | … |
| fallback-2 | … | … | … | … | … |

If a cell is empty, you have not measured it — do not claim that provider
is "cheaper" or "faster".
