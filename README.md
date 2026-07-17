# Preload & Cache: A Context Reuse Pattern for Long-Form Knowledge Injection

> **Chinese title:** 静态长文本注入 · 上下文缓存复用方案
> **Formerly known as:** PoorGuy's Excuse
> **Test platform:** OpenClaw Gateway · **Test date:** 2026-07-06
> **Last updated:** 2026-07-07

---

## What This Is

A specialized pattern for injecting pre-prepared research reports, knowledge documents, or analysis materials into a language model's context window, then reusing that context across multiple follow-up conversations via **context caching**. The goal is to eliminate redundant token consumption when repeatedly querying the same body of static knowledge.

> **In one sentence:** This is a solution for "read a book once, then answer questions about it without re-reading."
> 
> It is **not** a solution for "search the internet and summarize."

---

## Scope & Boundaries

### Suitable For

| Scenario | Example |
| --- | --- |
| 📊 Research analysis | Inject 50KB+ reports, then ask follow-up questions across multiple dimensions |
| 📚 Knowledge-base QA | Inject product documentation / compliance manuals, then repeatedly query |
| 📝 Report review | Inject a complete proposal, then discuss section by section |
| 🧠 Deep research | Inject industry data, then analyze from multiple perspectives |

### Not Suitable For

| Scenario | Reason |
| --- | --- |
| 🌐 Real-time data collection | Requires dynamic retrieval; cached context cannot reflect new content |
| 💬 Casual chat | Injection cost far outweighs single-turn dialogue benefit |
| 🔄 Frequent document switching | Each switch requires re-injection, wasting cache |
| 🔍 Web search + summarization | This is a different technical stack \(e.g., MCP + real-time compression\) |

---

## ⚠️ Prerequisites and Baseline Considerations

### Baseline Cache Hit Rate in Typical Usage Scenarios

In production environments where the system prompt comprises dynamically updated artifacts—including memory files, skill configurations, heartbeat routines, and sub-agent spawn overhead—the baseline cache hit rate for ordinary conversational turns typically falls within the range of **20–30%**. This figure reflects the aggregate effect of continuous context drift caused by routine operational modifications to the prompt surface.

### Expected Improvement Under the Preload & Cache Pattern

When a substantial static document (≥50 KB) is injected and maintained without replacement across multiple follow-up turns, the cache hit rate is observed to increase to **80–97%**. The magnitude of improvement is attributable to the stabilization of the context hash following the initial injection, which allows subsequent dialogue turns to be served predominantly from the cached prefix.

### Conditions for Optimal Performance

For the preload-and-cache pattern to yield the observed improvements, the following conditions should be satisfied:

1. **Document immutability during the conversation.** The injected long-form content must not be replaced or overwritten mid-session. Each document switch necessitates a full re-injection and invalidates the existing cache.
2. **Model consistency within a session.** Frequent model switching within a single session may partially flush the context cache, reducing the hit rate for subsequent turns.
3. **Stability of auxiliary prompt components.** Background processes such as heartbeat routines that automatically modify core files (e.g., `MEMORY.md`) will alter the system prompt hash and consequently invalidate the cache. If such processes are present, they should either be suppressed during the injection window or configured to operate on a silent-threshold basis (i.e., modifying files only upon explicit user authorization).

### Diagnostic Guidance

If the observed cache hit rate falls below 50% after injecting a static document, the following diagnostic steps are recommended:

- **Verify prompt stability.** Confirm that no background processes are modifying files referenced in the system prompt during the test window.
- **Check model consistency.** Ensure that the same model instance is used throughout the injection and follow-up dialogue rounds.
- **Assess document size.** Documents smaller than 50 KB may not produce a sufficiently large cached prefix to yield meaningful hit-rate improvements.

> **Note:** The cache hit rates reported herein are derived from empirical observation within a single OpenClaw session. They represent reference data rather than statistically rigorous measurements. A formal A/B test requires independent sessions with identical injection content and identical dialogue protocols.

---

## ⚠️ Data Reliability Notice

This test was conducted within a **single OpenClaw session** with model switching between rounds. Context residuals from the first model may have influenced the second model's cache statistics. **This does not constitute a rigorous A/B test.**

The data below reflects the observed cache behavior in this specific session and should be treated as **reference only**, not as a definitive cross-model comparison. A strict A/B test requires two independent sessions, each injected with identical data and executed independently.

---

## 📊 Observed Results \(Reference Only\)

| Model | Cache Hit Rate | Context Usage | Rounds | New Writes | Session Isolation |
| --- | --- | --- | --- | --- | --- |
| **DeepSeek V4 Flash** | **97%** | 56k / 1.0M \(5%\) | 5 | 0 | ❌ Same session |
| **Agnes 2.0 Flash** | **89%** \(⚠️ residual\) | 65k / 1.0M \(6%\) | 5 | 0 | ❌ Same session |

**Observations:**

- Both models showed stable context caching with zero compaction across all rounds.
- New token writes were 0 in both tests, confirming the reuse pattern works.
- DeepSeek V4 Flash showed a higher hit rate \(97% vs 89%\), though cross-session validation is needed.

---

## 🔬 Test Configuration

| Dimension | Details |
| --- | --- |
| Injected data | xx pet supplies market research \(3 reports combined, ~56-65k tokens\) |
| Dialogue rounds | 5 rounds per model \(10 total\) |
| Framework | Reference comparison \(same session, non-rigorous\) |
| Tool | OpenClaw Gateway |
| Injection method | Direct `.md` file read into session context |

---

## 🗂️ Project Structure

```
preload-and-cache/
├── README.md                          \# Project overview + core findings
├── data/
│   ├── injection-content-summary.md   \# Summary of injected content
│   ├── round1-deepseek-v4-flash.md    \# DeepSeek V4 Flash test log
│   ├── round2-agnes-flash.md          \# Agnes 2.0 Flash test log
│   └── ab-test-results.md             \# Final comparison + analysis
├── docs/
│   ├── methodology.md                 \# Testing methodology
│   └── faq.md                         \# Frequently asked questions
├── assets/
│   └── \(screenshots / charts\)
├── LICENSE
└── .gitignore
```

---

## 🛠️ Reproduction Steps

1. Prepare a long-form research document \(50KB+, `.md` format\)
2. Read and inject the document into an OpenClaw session context
3. Execute 3-5 rounds of short follow-up dialogue
4. Check `session_status` for cache hit rate
5. Switch model and repeat steps 2-4

> **For a rigorous A/B test:** Run steps 2-4 in two separate, isolated sessions with identical injection content and identical dialogue prompts.

---

## 🔗 Related Projects

### Dynamic Token Compression — Real-Time Data Pipeline

Preload & Cache and Dynamic Token Compression complement each other as two halves of a single context-management strategy:

| Aspect | Preload & Cache | Dynamic Token Compression |
|--------|----------------|--------------------------|
| **Data type** | Static, long-form documents | Live, dynamic search results |
| **Use case** | Repeated queries on the same report | One-off queries on fresh data |
| **Core technique** | Context caching + injection | Structured compression + ratio optimization |
| **Best for** | 5+ queries on one document | Single query on scraped data |
| **Reference** | This document | [../dynamic-token-compression/](../dynamic-token-compression/) |

**Together they form a complete pipeline:**

```
Live Data → Dynamic Token Compression → Compressed Result
                                              ↓
                                    If reused repeatedly → Preload & Cache
```

See the https://github.com/LouieKeung-13/dynamic-token-compression/blob/main/README.md for compression ratios, strategy benchmarks, and decision rules.

---

## 📄 License

MIT

---


*Compiled by LouieKeung · 2026*

For more detailed information, please send an email to: markdlouis1995@gmail.com
