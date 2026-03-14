# SmartSearch AI — Metrics & Evaluation Framework

## Overview

This document defines how we measure the success of SmartSearch AI across three dimensions: **Search Quality**, **User Experience**, and **Business Impact**. It includes metric definitions, measurement methodology, evaluation pipeline design, and a dashboard specification.

---

## 1. Metric Categories

### 1.1 Search Quality Metrics

| Metric | Definition | Formula | Target |
|--------|-----------|---------|--------|
| **Intent Accuracy** | % of queries where AI correctly identifies all intent signals | Correct intents / Total queries | ≥ 85% |
| **Relevance@K** | % of top-K results rated as relevant by human evaluators | Relevant results in top K / K | ≥ 70% at K=4 |
| **NDCG@4** | Normalized Discounted Cumulative Gain for top 4 results | Standard NDCG formula | ≥ 0.75 |
| **Zero-Result Rate** | % of queries returning 0 results | Zero-result queries / Total queries | < 8% |
| **Query Refinement Rate** | % of users who rephrase their initial query | Refined queries / Total sessions | < 15% (lower = better) |

### 1.2 User Experience Metrics

| Metric | Definition | Formula | Target |
|--------|-----------|---------|--------|
| **Time to First Click** | Seconds from search submission to first product click | Median timestamp delta | < 20 sec |
| **Search-to-Click Rate (CTR)** | % of searches resulting in at least one product click | Sessions with click / Total search sessions | ≥ 35% |
| **Results Scroll Depth** | How far users scroll through results | Median position of last viewed result | ≥ 3 of 4 results viewed |
| **Intent Panel Engagement** | % of users who view/interact with the AI intent panel | Panel views / Total searches | ≥ 60% |
| **CSAT Score** | Post-search satisfaction survey (1-5 scale) | Average rating | ≥ 4.2 |

### 1.3 Business Impact Metrics

| Metric | Definition | Formula | Target |
|--------|-----------|---------|--------|
| **Search-to-Cart Rate** | % of searches leading to add-to-cart | Cart additions from search / Total searches | ≥ 10% |
| **Search-to-Purchase Rate** | % of searches leading to completed purchase | Purchases from search / Total searches | ≥ 3% |
| **Revenue per Search** | Average revenue generated per search query | Total search-attributed revenue / Total searches | +25% vs baseline |
| **Return Visit Rate** | % of users who return to use SmartSearch again within 7 days | Returning users / Total unique users | ≥ 40% |

---

## 2. Evaluation Pipeline

### 2.1 Offline Evaluation

**Purpose:** Test search quality before shipping changes to production.

**Process:**
1. Maintain a "golden dataset" of 500+ queries with human-labeled ideal results
2. Run every model/algorithm change against the golden dataset
3. Compute Relevance@K, NDCG@4, and Intent Accuracy automatically
4. Block deployment if any metric drops >5% from the current baseline

**Golden Dataset Structure:**
```json
{
  "query": "lightweight running shoes under 3000",
  "expected_intent": {
    "category": "Footwear",
    "budget": 3000,
    "attrs": ["lightweight"],
    "use_case": "Fitness & Sports"
  },
  "relevant_product_ids": [1, 2],
  "ideal_ranking": [1, 2]
}
```

### 2.2 Online Evaluation (A/B Testing)

**Purpose:** Measure real-world impact on user behavior and business metrics.

**Design:**
- **Control group (50%):** Traditional keyword + filter search
- **Treatment group (50%):** SmartSearch AI natural language search
- **Minimum sample size:** 10,000 searches per group (for 95% confidence, 80% power)
- **Primary metric:** Search-to-Cart Rate
- **Guardrail metrics:** Page load time (must stay < 2s), error rate (must stay < 1%)

**Rollout gates:**
- Week 1-2: 5% traffic → validate no regressions
- Week 3-4: 20% traffic → measure directional lift
- Week 5-8: 50% traffic → achieve statistical significance
- Week 9+: Full rollout if primary metric shows ≥ 15% lift

### 2.3 Continuous Monitoring

**Real-time alerts for:**
- Intent parsing error rate > 5%
- API latency p95 > 2 seconds
- Zero-result rate spikes > 2x baseline
- CSAT drops below 3.5

---

## 3. Dashboard Specification

### 3.1 Search Quality Dashboard

**Refresh:** Daily

**Panels:**
1. Intent Accuracy trend (7-day rolling average)
2. NDCG@4 trend (7-day rolling)
3. Zero-result rate by category
4. Top 20 failing queries (lowest relevance scores)
5. Query volume by category (pie chart)

### 3.2 User Experience Dashboard

**Refresh:** Real-time

**Panels:**
1. Search volume (hourly, daily)
2. CTR funnel: Search → Click → Cart → Purchase
3. Time to first click distribution (histogram)
4. CSAT distribution and trend
5. Most popular query patterns (word cloud)

### 3.3 Business Impact Dashboard

**Refresh:** Daily

**Panels:**
1. Revenue per search (SmartSearch vs control)
2. Search-to-cart rate (A/B comparison)
3. Return visit rate (cohort analysis)
4. Revenue attribution from AI search

---

## 4. Prompt Evaluation Framework

Since SmartSearch relies on LLM-based intent parsing, we need a dedicated evaluation framework for prompt quality.

### 4.1 Eval Dimensions

| Dimension | What We Test | Method |
|-----------|-------------|--------|
| **Accuracy** | Does the model extract the correct intent signals? | Compare against golden labels |
| **Completeness** | Does the model capture ALL relevant signals? | Count missing signals |
| **Robustness** | Does the model handle typos, slang, multilingual input? | Adversarial test set |
| **Latency** | Does intent parsing complete within SLA? | p50, p95, p99 latency |
| **Consistency** | Same query → same intent across multiple runs? | Run 10x, measure variance |

### 4.2 Eval Process

1. Curate 200+ diverse test queries spanning all categories
2. Human-label the expected intent for each query
3. Run the prompt against all test queries
4. Score each dimension automatically
5. Flag any regression > 3% for human review
6. Run this eval on every prompt change, model update, or weekly (whichever comes first)

---

## 5. Cost Monitoring

| Resource | Metric | Alert Threshold |
|----------|--------|----------------|
| LLM API calls | Cost per 1,000 queries | > $2.50 |
| Search infrastructure | Monthly compute cost | > $500/month |
| Embedding generation | Cost per catalog update | > $50/update |
| Total AI cost per purchase | LLM cost / Attributed purchases | > $0.50 |

---

## 6. Reporting Cadence

| Report | Audience | Frequency | Contents |
|--------|---------|-----------|----------|
| **Search Quality Report** | Engineering, Data Science | Weekly | Quality metrics, failing queries, model performance |
| **Product Performance Review** | Product, Leadership | Bi-weekly | Business metrics, A/B results, user feedback |
| **Exec Dashboard** | C-suite | Monthly | Revenue impact, cost efficiency, strategic roadmap |

---

*This framework is designed to be implemented incrementally: start with offline evaluation (Week 1), add online metrics (Week 2-3), build dashboards (Week 4), and establish continuous monitoring (Week 5+).*
