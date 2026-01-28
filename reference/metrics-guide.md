# Metrics Guide Reference

Definitions, thresholds, and interpretation guidance for Cortex AI evaluation metrics.

## When to Load

From `cortex-ai-evaluation` when user needs:
- Understanding of what each metric means
- Recommended threshold values
- Guidance on interpreting results
- Help choosing success criteria

---

## Core Metrics Overview

| Metric | Range | Best For | Quick Interpretation |
|--------|-------|----------|---------------------|
| Accuracy | 0-100% | Classification | % of correct predictions |
| Precision | 0-100% | Classification | % of predicted positives that are correct |
| Recall | 0-100% | Classification | % of actual positives that were found |
| F1 Score | 0-100% | Classification | Balance of precision and recall |
| Completeness | 0-100% | Extraction | % of expected fields present |
| LLM Judge Score | 1-10 | All | AI quality assessment |
| Similarity Score | 0-100 | Text comparison | How similar two texts are |

---

## Metric Definitions

### Accuracy

**What it measures:** The percentage of predictions that match the expected values.

**Formula:**
```
Accuracy = (Correct Predictions) / (Total Predictions) × 100
```

**SQL:**
```sql
SELECT 
  ROUND(
    SUM(CASE WHEN predicted = expected THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
    2
  ) AS accuracy_pct
FROM evaluation_results;
```

**When to use:**
- Classification tasks with balanced classes
- Extraction tasks where all fields are equally important
- Quick overall quality assessment

**Limitations:**
- Can be misleading with imbalanced data (e.g., 95% of samples are class A)
- Treats all errors equally

---

### Precision

**What it measures:** Of all items predicted as a certain class, how many were actually that class.

**Formula:**
```
Precision = True Positives / (True Positives + False Positives) × 100
```

**SQL:**
```sql
SELECT 
  category,
  ROUND(
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) * 100.0 /
    NULLIF(SUM(CASE WHEN predicted = category THEN 1 ELSE 0 END), 0),
    2
  ) AS precision_pct
FROM evaluation_results
GROUP BY category;
```

**When to use:**
- When false positives are costly (spam detection, fraud alerts)
- When you want to minimize incorrect positive predictions

**Interpretation:**
- High precision = When we predict this class, we're usually right
- Low precision = Many false alarms

---

### Recall (Sensitivity)

**What it measures:** Of all actual instances of a class, how many did we correctly identify.

**Formula:**
```
Recall = True Positives / (True Positives + False Negatives) × 100
```

**SQL:**
```sql
SELECT 
  category,
  ROUND(
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) * 100.0 /
    NULLIF(SUM(CASE WHEN actual = category THEN 1 ELSE 0 END), 0),
    2
  ) AS recall_pct
FROM evaluation_results
GROUP BY category;
```

**When to use:**
- When false negatives are costly (disease detection, security threats)
- When you want to find all instances of a class

**Interpretation:**
- High recall = We find most actual instances
- Low recall = We miss many actual instances

---

### F1 Score

**What it measures:** Harmonic mean of precision and recall, balancing both metrics.

**Formula:**
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**SQL:**
```sql
WITH metrics AS (
  SELECT 
    category,
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) AS tp,
    SUM(CASE WHEN predicted = category AND actual != category THEN 1 ELSE 0 END) AS fp,
    SUM(CASE WHEN predicted != category AND actual = category THEN 1 ELSE 0 END) AS fn
  FROM evaluation_results
  GROUP BY category
)
SELECT 
  category,
  ROUND(
    2 * (tp * 1.0 / NULLIF(tp + fp, 0)) * (tp * 1.0 / NULLIF(tp + fn, 0)) /
    NULLIF((tp * 1.0 / NULLIF(tp + fp, 0)) + (tp * 1.0 / NULLIF(tp + fn, 0)), 0) * 100,
    2
  ) AS f1_score_pct
FROM metrics;
```

**When to use:**
- When you need a single metric that balances precision and recall
- When both false positives and false negatives matter
- For comparing different models or configurations

---

### Completeness

**What it measures:** Percentage of expected fields/values that are present and non-empty.

**Formula:**
```
Completeness = (Non-empty Fields) / (Expected Fields) × 100
```

**SQL:**
```sql
SELECT 
  record_id,
  ROUND(
    (IFF(field1 IS NOT NULL AND field1 != '', 1, 0) +
     IFF(field2 IS NOT NULL AND field2 != '', 1, 0) +
     IFF(field3 IS NOT NULL AND field3 != '', 1, 0)) * 100.0 / 3,
    1
  ) AS completeness_pct
FROM extraction_results;
```

**When to use:**
- AI_EXTRACT evaluation
- Document parsing assessment
- Data quality checks

---

### LLM Judge Score

**What it measures:** AI-assessed quality on a 1-10 scale based on defined criteria.

**Scale interpretation:**
| Score | Quality Level | Action |
|-------|--------------|--------|
| 9-10 | Excellent | Production ready |
| 7-8 | Good | Acceptable for most use cases |
| 5-6 | Acceptable | May need refinement |
| 3-4 | Poor | Requires improvement |
| 1-2 | Unacceptable | Major issues |

**SQL:**
```sql
SELECT 
  AVG(evaluation:overall_score::FLOAT) AS avg_llm_score,
  STDDEV(evaluation:overall_score::FLOAT) AS score_stddev,
  MIN(evaluation:overall_score::INT) AS min_score,
  MAX(evaluation:overall_score::INT) AS max_score
FROM llm_evaluations;
```

**When to use:**
- Subjective quality assessment
- When ground truth is not available
- Evaluating AI_COMPLETE outputs
- Translation and summarization quality

---

### Similarity Score (Jaro-Winkler)

**What it measures:** How similar two text strings are, with emphasis on matching prefixes.

**Scale:** 0-100 (100 = identical)

**SQL:**
```sql
SELECT 
  JAROWINKLER_SIMILARITY(extracted_value, expected_value) AS similarity
FROM comparison_results;
```

**Interpretation:**
| Score | Meaning |
|-------|---------|
| 100 | Exact match |
| 95-99 | Minor differences (typos, formatting) |
| 85-94 | Close match (word order, abbreviations) |
| 70-84 | Partial match (some overlap) |
| <70 | Significant differences |

---

## Recommended Thresholds

### By Use Case

#### Mission-Critical Applications (Healthcare, Finance, Legal)

| Metric | Minimum Threshold | Target |
|--------|------------------|--------|
| Accuracy | 95% | 98%+ |
| F1 Score | 90% | 95%+ |
| Completeness | 98% | 100% |
| LLM Judge Score | 9/10 | 9.5/10 |

#### Business Applications (Analytics, Reporting)

| Metric | Minimum Threshold | Target |
|--------|------------------|--------|
| Accuracy | 90% | 95%+ |
| F1 Score | 85% | 90%+ |
| Completeness | 95% | 98%+ |
| LLM Judge Score | 8/10 | 9/10 |

#### Exploratory / Internal Use

| Metric | Minimum Threshold | Target |
|--------|------------------|--------|
| Accuracy | 80% | 90%+ |
| F1 Score | 75% | 85%+ |
| Completeness | 90% | 95%+ |
| LLM Judge Score | 7/10 | 8/10 |

### By Cortex AI Function

#### AI_EXTRACT

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| Field Accuracy | ≥95% | 85-94% | <85% |
| Completeness | ≥98% | 90-97% | <90% |
| LLM Judge Score | ≥8.5/10 | 7-8.4/10 | <7/10 |

#### AI_CLASSIFY

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| Accuracy | ≥90% | 80-89% | <80% |
| F1 Score (macro) | ≥85% | 75-84% | <75% |
| Per-class F1 | ≥80% each | ≥70% each | <70% any |

#### AI_COMPLETE

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| LLM Judge Overall | ≥8/10 | 6-7.9/10 | <6/10 |
| Relevance Score | ≥8/10 | 7-7.9/10 | <7/10 |
| Helpfulness Score | ≥8/10 | 7-7.9/10 | <7/10 |

#### AI_SENTIMENT

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| Classification Accuracy | ≥85% | 75-84% | <75% |
| Mean Absolute Error | ≤0.15 | 0.15-0.25 | >0.25 |
| Direction Accuracy | ≥90% | 80-89% | <80% |

#### AI_TRANSLATE

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| Fluency Score | ≥8/10 | 6-7.9/10 | <6/10 |
| Adequacy Score | ≥8/10 | 6-7.9/10 | <6/10 |
| Meaning Preserved | ≥95% | 85-94% | <85% |

---

## Interpreting Results

### Decision Framework

```
┌──────────────────────────────────────────────────────────────┐
│                    EVALUATION RESULTS                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   All metrics ≥ Production threshold                         │
│        │                                                      │
│        ▼                                                      │
│   ╔═══════════════════════════════════════════════════════╗  │
│   ║  ✓ DEPLOY TO PRODUCTION                                ║  │
│   ╚═══════════════════════════════════════════════════════╝  │
│                                                               │
│   Most metrics ≥ Acceptable threshold                        │
│        │                                                      │
│        ▼                                                      │
│   ╔═══════════════════════════════════════════════════════╗  │
│   ║  ⚠ REVIEW & ITERATE                                    ║  │
│   ║  - Identify failing metrics                            ║  │
│   ║  - Refine prompts/schema for those areas               ║  │
│   ║  - Re-evaluate                                         ║  │
│   ╚═══════════════════════════════════════════════════════╝  │
│                                                               │
│   Any metric < Acceptable threshold                          │
│        │                                                      │
│        ▼                                                      │
│   ╔═══════════════════════════════════════════════════════╗  │
│   ║  ✗ SIGNIFICANT REWORK NEEDED                           ║  │
│   ║  - Analyze failure patterns                            ║  │
│   ║  - Consider alternative approaches                     ║  │
│   ║  - May need schema redesign or different function      ║  │
│   ╚═══════════════════════════════════════════════════════╝  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Common Patterns and Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| High accuracy, low recall for one class | Class imbalance | Add more examples of that class, adjust thresholds |
| Low completeness, high accuracy | Field sometimes missing | Make prompts more specific, check document quality |
| LLM judge scores inconsistent | Vague evaluation criteria | Add specific rubrics to judge prompt |
| High accuracy on test, low in production | Test data not representative | Sample production data for evaluation |
| Metrics degrading over time | Data drift | Set up monitoring, retrain periodically |

---

## Sample Size Guidelines

### Minimum Sample Sizes for Reliable Metrics

| Confidence Level | Accuracy Precision | Minimum Samples |
|-----------------|-------------------|-----------------|
| Quick check | ±10% | 30 |
| Reasonable estimate | ±5% | 100 |
| High confidence | ±3% | 300 |
| Statistical rigor | ±1% | 1,000+ |

### SQL for Confidence Intervals

```sql
-- Calculate 95% confidence interval for accuracy
WITH stats AS (
  SELECT 
    COUNT(*) AS n,
    AVG(CASE WHEN is_correct THEN 1.0 ELSE 0.0 END) AS accuracy
  FROM evaluation_results
)
SELECT 
  ROUND(accuracy * 100, 2) AS accuracy_pct,
  ROUND((accuracy - 1.96 * SQRT(accuracy * (1 - accuracy) / n)) * 100, 2) AS lower_95ci,
  ROUND((accuracy + 1.96 * SQRT(accuracy * (1 - accuracy) / n)) * 100, 2) AS upper_95ci
FROM stats;
```

---

## Setting Custom Thresholds

### Questions to Ask

1. **What is the cost of errors?**
   - High cost → Higher thresholds
   - Low cost → Lower thresholds acceptable

2. **What is the current baseline?**
   - Set targets relative to existing performance
   - Aim for incremental improvement

3. **What do downstream systems require?**
   - Consider integration requirements
   - Match SLAs of dependent systems

4. **How much human review is available?**
   - More review capacity → Lower thresholds OK
   - No review → Higher thresholds required

### Threshold Setting Template

```sql
-- Create threshold configuration table
CREATE TABLE evaluation_thresholds (
  pipeline_name STRING,
  metric_name STRING,
  production_threshold FLOAT,
  acceptable_threshold FLOAT,
  alert_threshold FLOAT,
  created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  created_by STRING DEFAULT CURRENT_USER(),
  notes STRING
);

-- Insert thresholds for a pipeline
INSERT INTO evaluation_thresholds 
  (pipeline_name, metric_name, production_threshold, acceptable_threshold, alert_threshold, notes)
VALUES
  ('invoice_extraction', 'accuracy', 95.0, 90.0, 85.0, 'Financial data - high accuracy required'),
  ('invoice_extraction', 'completeness', 98.0, 95.0, 90.0, 'All fields needed for processing'),
  ('invoice_extraction', 'llm_judge_score', 8.5, 7.5, 6.5, 'Quality assessment threshold');
```

---

## Metrics Aggregation

### Weighted Average Across Fields

```sql
-- Weight fields by importance
SELECT 
  record_id,
  ROUND(
    (field1_accuracy * 0.4 +   -- 40% weight - most important
     field2_accuracy * 0.35 +  -- 35% weight
     field3_accuracy * 0.25)   -- 25% weight - least important
    , 2
  ) AS weighted_accuracy
FROM field_accuracies;
```

### Macro vs Micro Averaging

```sql
-- Macro average (average of per-class metrics)
SELECT 
  ROUND(AVG(class_f1_score), 2) AS macro_f1
FROM per_class_metrics;

-- Micro average (global TP, FP, FN)
SELECT 
  ROUND(
    2 * SUM(tp) * 1.0 / (2 * SUM(tp) + SUM(fp) + SUM(fn)) * 100,
    2
  ) AS micro_f1
FROM per_class_metrics;
```

---

## Quick Reference Card

### Metric Selection Guide

| Question | Use This Metric |
|----------|----------------|
| "How often are we right overall?" | Accuracy |
| "When we predict X, how often is it actually X?" | Precision |
| "Of all actual X, how many did we find?" | Recall |
| "Balance of precision and recall?" | F1 Score |
| "Are all expected values present?" | Completeness |
| "How good is the overall quality?" | LLM Judge Score |
| "How similar are extracted vs expected text?" | Similarity Score |

### Threshold Quick Reference

| Risk Level | Accuracy | F1 | LLM Score |
|-----------|----------|-----|-----------|
| High (finance, healthcare) | ≥95% | ≥90% | ≥9/10 |
| Medium (business apps) | ≥90% | ≥85% | ≥8/10 |
| Low (internal/exploratory) | ≥80% | ≥75% | ≥7/10 |
