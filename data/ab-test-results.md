# Comparative Results: Context Cache Hit Rate Across Inference Models

> **Test period:** 2026-07-06 to 2026-07-07
> **Test framework:** Single-session model switching (non-rigorous)

---

## ⚠️ Methodological Note

Both test rounds were executed within a **single OpenClaw session** with model switching between rounds. Context carryover from the first model may have influenced the second model's cache statistics. **This does not constitute a rigorous A/B test.** The results below should be treated as **observational reference only** and not as a definitive cross-model comparison.

A conclusive comparison requires two isolated sessions, each receiving identical injection content and executing independent dialogue rounds.

---

## Observed Data (Reference Only)

| Metric | Test A (DeepSeek V4 Flash) | Test B (Agnes 2.0 Flash) |
| --- | --- | --- |
| **Cache hit rate** | **97%** | **89%** |
| Cache hit volume | 53k tokens | 504k tokens |
| New token writes | 0 | 0 |
| Context utilization | 56k / 1.0M (5%) | 65k / 1.0M (6%) |
| Compaction count | 0 | 0 |
| Input tokens | 1.7k | 64k (incl. 58k initial injection) |
| Output tokens | 748 | 858 |
| Dialogue rounds | 5 | 5 |
| Session isolation | ❌ Same session ⚠️ | ❌ Same session ⚠️ |

---

## Observations

### Common Findings

- **Zero compaction** across both rounds, indicating stable context management.
- **Zero new token writes** in both tests, confirming that subsequent dialogue rounds were fully served from cache after the initial injection.
- **Low context utilization** (5–6% of 1M window), leaving ample headroom for additional content.

### Differential Findings

- DeepSeek V4 Flash achieved a higher cache hit rate (97% vs. 89%), a difference of approximately 8 percentage points.
- However, context carryover from Test A may have contributed to the elevated cache hit volume observed in Test B (504k vs. 53k), suggesting the second round's statistics may be partially inflated.
- Both hit rates are substantially above the 80% threshold defined in the evaluation criteria, supporting the viability of the preload-and-cache pattern.
- **Cross-session validation is required** to draw reliable conclusions about inter-model differences.

---

## Recommended Follow-Up

| Step | Action | Priority |
| --- | --- | --- |
| 1 | Launch two independent sessions, inject identical data | 🔴 Required |
| 2 | Execute 5 rounds of identical dialogue in each session | 🔴 Required |
| 3 | Record `session_status` cache metrics independently | 🔴 Required |
| 4 | Compare results across isolated sessions | 🟡 Recommended |
