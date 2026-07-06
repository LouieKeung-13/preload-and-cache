# FAQ

> Frequently Asked Questions about the Preload & Cache pattern and observed context caching behavior.

---

## Q1: Why conduct an A/B test at all?

Different LLM inference providers implement context caching with varying granularity, expiration policies, and encoding strategies. An A/B test provides empirical evidence of how the preload-and-cache pattern performs across distinct model providers, rather than relying on theoretical assumptions.

## Q2: Why DeepSeek V4 Flash and Agnes 2.0 Flash?

- **DeepSeek V4 Flash** — A lightweight inference model via DeepSeek, with a mature context caching mechanism.
- **Agnes 2.0 Flash** — A lightweight inference model via a different provider, enabling a cross-provider comparison.

These two models represent the class of "fast, cost-efficient" inference endpoints where caching overhead is most relevant.

## Q3: After injecting 50KB+ of data, why do only 2–3k new tokens appear across 5 dialogue rounds?

Because the injected context is cached on the first pass. Subsequent rounds reuse the cached portion, and the only new tokens consumed are the incremental input (the user's follow-up question) and the model's output. With a high hit rate, the majority of the 56k+ token context is served from cache, not recomputed.

## Q4: What accounts for the ~8 percentage-point difference between 97% and 89% hit rates?

Several factors may contribute, though no single cause can be confirmed without cross-session testing:

- **Cache granularity:** Block-level vs. request-level caching implementations differ by provider.
- **Expiration policies:** Some providers evict cached blocks sooner than others.
- **Encoding differences:** How each provider tokenizes and indexes context affects cache alignment.
- **Context residuals:** Because both tests ran in the same session, residual cache state from the first model may have influenced the second model's statistics.

## Q5: How much cost savings does this pattern deliver in practice?

Consider a concrete scenario: inject 56k tokens of research data, then run 10 rounds of dialogue (each round ~200 tokens input, ~700 tokens output).

| Mode | Calculation | Total Tokens |
| --- | --- | --- |
| Without cache | 56k + 10 × (200 + 700) | ~65k |
| With 97% cache | 56k + 10 × (~3 + 700) | ~63k |
| With 89% cache | 56k + 10 × (~6 + 700) | ~63.6k |

The saving is modest in token count but significant in **latency**: cached tokens are served near-instantly versus recomputed. Actual benefit scales with the ratio of injected context to dialogue turns — larger injections + more turns = greater relative savings.

## Q6: Should we stress-test with 100k+ context?

Not strictly necessary. The 56–65k range covers the vast majority of real-world use cases (single report injection followed by multi-round discussion). A 100k+ test becomes worthwhile if:

- The project is intended for external publication or team sharing.
- You want to verify system stability under extreme context pressure.
- You plan to inject multiple large documents simultaneously.

## Q7: How do I reproduce this test?

See the **Reproduction Steps** section in `README.md`. For a rigorous A/B test, run two independent OpenClaw sessions with identical injection content and identical dialogue prompts, then compare results.
