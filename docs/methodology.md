# Experimental Methodology

> This document describes the experimental design, execution protocol, and data collection framework for evaluating the Preload & Cache pattern.

---

## 1. Objective

Evaluate the context cache hit rate of different LLM inference models under a **long-form knowledge injection + multi-turn short dialogue** workflow.

**Working hypotheses:**

1. After a one-time injection of 50KB+ of research data, subsequent dialogue rounds should achieve high cache hit rates.
2. Different inference providers exhibit measurable differences in cache performance.
3. The preload-and-cache pattern yields significant token cost reduction compared to naive re-injection.

---

## 2. Experimental Design

### 2.1 Comparison Mode

**Actual execution (non-rigorous):** Both tests were conducted within a single OpenClaw session with model switching between rounds. This introduces the risk of context carryover.

**Recommended execution (rigorous A/B test):** Two independent OpenClaw sessions should be launched, each assigned a different model, each receiving identical injection content, with dialogue and cache metrics recorded independently.

### 2.2 Controlled Variables

| Variable | Model A | Model B |
| --- | --- | --- |
| Injected data | Identical (3 reports) | Identical (3 reports) |
| Injection size | ~56k tokens | ~58k tokens |
| Dialogue rounds | 5 | 5 |
| Test environment | Same OpenClaw Gateway | Same OpenClaw Gateway |
| Injection method | Read `.md` into context | Read `.md` into context |
| **Session isolation** | ❌ Same session ⚠️ | ❌ Same session ⚠️ |

### 2.3 Variables Under Test

- **Independent variable (planned):** Inference model / provider
- **Confounding factor (actual):** Context cache may persist across model switches within the same session
- **Dependent variables:** Cache hit rate (%), new token writes (tokens), compaction count

---

## 3. Actual Execution Protocol

The following sequence was followed in this test run:

```
Step 1: Record initial session_status (Model A)
Step 2: Inject research data (3 .md files)
Step 3: Execute 5 rounds of dialogue
Step 4: Record final session_status
Step 5: Switch model (context carryover risk) ⚠️
Step 6: Repeat Steps 1–4
Step 7: Aggregate results (reference only)
```

---

## 4. Recommended Protocol (Strict A/B Test)

For future validation, the following protocol is recommended:

```
Session A (Model A):
  Step 1: Launch new session, record initial session_status
  Step 2: Inject research data
  Step 3: Execute 5 rounds of dialogue
  Step 4: Record final session_status

Session B (Model B):
  Step 1: Launch a separate new session, record initial session_status
  Step 2: Inject identical research data
  Step 3: Execute identical 5 rounds of dialogue
  Step 4: Record final session_status

Step 5: Compare results across independent sessions
```

---

## 5. Data Collection

### 5.1 Collection Instrument

OpenClaw Gateway's built-in `session_status` command.

### 5.2 Metrics

| Metric | Definition | Target |
| --- | --- | --- |
| **Cache hit rate** | Percentage of tokens served from cache | > 80% |
| **Cache hit volume** | Absolute number of tokens served from cache | Higher is better |
| **New writes** | Tokens not found in cache, requiring recomputation | 0 |
| **Context utilization** | Current context size relative to context window | Well below 100% |
| **Compaction count** | Number of times context was truncated or compressed | 0 |

### 5.3 Known Limitations of This Test

- Both rounds were executed within a **single session**, meaning context carryover from the first model may have influenced the second model's cache statistics.
- This is **not a rigorous A/B test**; results should be treated as observational reference, not as a definitive cross-model comparison.
- A follow-up test in two isolated sessions is recommended for conclusive findings.

### 5.4 Caveats

- Cache statistics reported by `session_status` may use different counting conventions across providers.
- Model switching within a session may partially flush context, affecting the second round's baseline hit rate.
- For cleaner comparisons, consider using two models from the same provider to isolate model-level effects from provider-level effects.

---

## 6. Interpretation Criteria

| Result | Assessment |
| --- | --- |
| Cache hit rate > 90% | Excellent ❇️ |
| Cache hit rate 80–90% | Good ✅ |
| Cache hit rate < 80% | Acceptable ⚠️ |
| New writes > 0 | Partial cache coverage |
| Compaction triggered | Context stability concern ❌ |
