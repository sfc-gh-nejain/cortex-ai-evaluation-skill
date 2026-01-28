# Evaluation Methods Reference

Detailed reference for evaluation techniques used with Cortex AI functions.

## When to Load

From `cortex-ai-evaluation` when user needs:
- Detailed LLM-as-judge prompt templates
- Ground truth table schema examples
- Advanced comparison techniques
- Custom evaluation criteria setup

---

## Evaluation Method 1: LLM-as-Judge

Uses a language model to evaluate the quality of AI function outputs. Best for subjective quality assessment where ground truth is not available.

### How It Works

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  AI Function    │────→│  Judge LLM      │────→│  Quality Score  │
│  Output         │     │  (Claude)       │     │  + Feedback     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Prompt Templates by Function Type

#### AI_EXTRACT Judge Prompt

```sql
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  $$You are an expert document extraction evaluator.

TASK: Evaluate the quality of structured data extracted from a document.

EXTRACTED DATA:
$$ || extracted_output::STRING || $$

EXPECTED SCHEMA:
$$ || schema_description || $$

EVALUATION CRITERIA:
1. Completeness (0-10): Are all expected fields present?
2. Accuracy (0-10): Do values appear correct based on field names?
3. Format (0-10): Are values in the expected format?
4. Plausibility (0-10): Do values make logical sense?

For each field, check:
- Is it present and non-empty?
- Does the value match what the field name suggests?
- Is the format appropriate (dates look like dates, numbers like numbers)?

Return JSON:
{
  "overall_score": <1-10 weighted average>,
  "completeness_score": <1-10>,
  "accuracy_score": <1-10>,
  "format_score": <1-10>,
  "plausibility_score": <1-10>,
  "field_evaluations": {
    "<field_name>": {
      "present": <true/false>,
      "score": <1-10>,
      "issue": "<null or issue description>"
    }
  },
  "critical_issues": ["<issue1>", "<issue2>"],
  "suggestions": ["<suggestion1>", "<suggestion2>"]
}$$,
  {'response_format': {'type': 'json'}, 'max_tokens': 2048}
);
```

#### AI_CLASSIFY Judge Prompt

```sql
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  $$You are an expert text classification evaluator.

TASK: Evaluate if the assigned classification is appropriate.

INPUT TEXT:
$$ || input_text || $$

ASSIGNED LABEL: $$ || assigned_label || $$

AVAILABLE CATEGORIES:
$$ || category_list_with_descriptions || $$

EVALUATION CRITERIA:
1. Appropriateness: Is this the best category for this text?
2. Confidence: How certain should we be about this classification?
3. Ambiguity: Could this text reasonably belong to other categories?

Consider:
- Key phrases and topics in the text
- The definitions of each category
- Edge cases and borderline situations

Return JSON:
{
  "is_correct": <true/false>,
  "assigned_label": "<the label that was assigned>",
  "recommended_label": "<what it should be>",
  "confidence": "<high/medium/low>",
  "score": <1-10>,
  "reasoning": "<explanation of the evaluation>",
  "alternative_labels": ["<other possible labels>"],
  "key_indicators": ["<phrases that support the classification>"]
}$$,
  {'response_format': {'type': 'json'}, 'max_tokens': 1024}
);
```

#### AI_COMPLETE Judge Prompt (Response Quality)

```sql
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  $$You are an expert at evaluating AI-generated text quality.

TASK: Evaluate the quality of this AI-generated response.

ORIGINAL PROMPT/QUESTION:
$$ || original_prompt || $$

GENERATED RESPONSE:
$$ || ai_response || $$

CONTEXT (if available):
$$ || COALESCE(context, 'None provided') || $$

EVALUATION DIMENSIONS:
1. Relevance (1-10): Does it answer what was asked?
2. Accuracy (1-10): Is the information factually correct?
3. Completeness (1-10): Does it fully address the question?
4. Clarity (1-10): Is it well-written and easy to understand?
5. Conciseness (1-10): Is it appropriately brief without missing key points?
6. Helpfulness (1-10): Would this be useful to the requester?

Return JSON:
{
  "overall_score": <1-10>,
  "dimensions": {
    "relevance": {"score": <1-10>, "note": "<observation>"},
    "accuracy": {"score": <1-10>, "note": "<observation>"},
    "completeness": {"score": <1-10>, "note": "<observation>"},
    "clarity": {"score": <1-10>, "note": "<observation>"},
    "conciseness": {"score": <1-10>, "note": "<observation>"},
    "helpfulness": {"score": <1-10>, "note": "<observation>"}
  },
  "strengths": ["<strength1>", "<strength2>"],
  "weaknesses": ["<weakness1>", "<weakness2>"],
  "summary": "<one-sentence assessment>",
  "improvement_suggestions": ["<suggestion1>", "<suggestion2>"]
}$$,
  {'response_format': {'type': 'json'}, 'max_tokens': 2048}
);
```

#### AI_SENTIMENT Judge Prompt

```sql
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  $$You are an expert sentiment analysis evaluator.

TASK: Evaluate if the sentiment score is accurate.

INPUT TEXT:
$$ || input_text || $$

ASSIGNED SENTIMENT SCORE: $$ || sentiment_score || $$
(Scale: -1 = very negative, 0 = neutral, 1 = very positive)

EVALUATION:
1. Read the text and determine the actual sentiment
2. Compare your assessment to the assigned score
3. Note any nuances (mixed sentiment, sarcasm, etc.)

Return JSON:
{
  "assigned_score": <the score given>,
  "expected_score": <what you think it should be>,
  "difference": <absolute difference>,
  "is_accurate": <true if difference < 0.3>,
  "actual_sentiment": "<positive/negative/neutral/mixed>",
  "confidence": "<high/medium/low>",
  "nuances": ["<sarcasm>", "<mixed feelings>", etc.],
  "explanation": "<why the scores match or differ>"
}$$,
  {'response_format': {'type': 'json'}, 'max_tokens': 512}
);
```

#### AI_TRANSLATE Judge Prompt

```sql
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  $$You are an expert translation quality evaluator.

TASK: Evaluate the quality of this translation.

SOURCE TEXT ($$ || source_language || $$):
$$ || source_text || $$

TRANSLATION ($$ || target_language || $$):
$$ || translated_text || $$

EVALUATION DIMENSIONS:
1. Fluency (1-10): Does it read naturally in the target language?
2. Adequacy (1-10): Does it convey the same meaning as the source?
3. Terminology (1-10): Are domain-specific terms translated correctly?
4. Grammar (1-10): Is the grammar correct in the target language?

Return JSON:
{
  "overall_score": <1-10>,
  "fluency": {"score": <1-10>, "issues": ["<issue>"]},
  "adequacy": {"score": <1-10>, "issues": ["<issue>"]},
  "terminology": {"score": <1-10>, "issues": ["<issue>"]},
  "grammar": {"score": <1-10>, "issues": ["<issue>"]},
  "meaning_preserved": <true/false>,
  "suggested_improvements": ["<improvement>"],
  "critical_errors": ["<error>"]
}$$,
  {'response_format': {'type': 'json'}, 'max_tokens': 1024}
);
```

### Best Practices for LLM-as-Judge

1. **Be Specific**: Include clear evaluation criteria in the prompt
2. **Use Structured Output**: Always request JSON for consistent parsing
3. **Provide Context**: Include schema definitions, category descriptions
4. **Set Score Ranges**: Define what each score level means
5. **Request Explanations**: Ask for reasoning, not just scores
6. **Run Multiple Times**: Average scores across 2-3 evaluations for stability

---

## Evaluation Method 2: Ground Truth Comparison

Compares AI function outputs against known correct values. Most reliable method when labeled data is available.

### Ground Truth Table Schemas

#### For AI_EXTRACT:

```sql
CREATE TABLE evaluation_ground_truth_extract (
  -- Identifier
  record_id STRING PRIMARY KEY,
  source_file STRING,
  
  -- Expected extraction values (customize per use case)
  expected_invoice_number STRING,
  expected_vendor_name STRING,
  expected_invoice_date DATE,
  expected_total_amount DECIMAL(10,2),
  expected_line_items VARIANT,  -- For array/nested data
  
  -- Metadata
  labeled_by STRING,
  labeled_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  confidence STRING DEFAULT 'high',  -- high/medium/low
  notes STRING
);
```

#### For AI_CLASSIFY:

```sql
CREATE TABLE evaluation_ground_truth_classify (
  -- Identifier
  record_id STRING PRIMARY KEY,
  input_text STRING,
  
  -- Expected classification
  expected_primary_label STRING NOT NULL,
  expected_secondary_labels ARRAY,  -- For multi-label
  
  -- Metadata
  labeled_by STRING,
  labeled_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  ambiguity_level STRING DEFAULT 'clear',  -- clear/ambiguous/edge_case
  notes STRING
);
```

#### For AI_SENTIMENT:

```sql
CREATE TABLE evaluation_ground_truth_sentiment (
  -- Identifier
  record_id STRING PRIMARY KEY,
  input_text STRING,
  
  -- Expected sentiment
  expected_sentiment_score FLOAT,  -- -1 to 1
  expected_sentiment_label STRING,  -- positive/negative/neutral/mixed
  
  -- Metadata
  labeled_by STRING,
  labeled_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  notes STRING
);
```

### Comparison Techniques

#### Exact Match (for categorical/ID fields):

```sql
CASE 
  WHEN LOWER(TRIM(extracted_value)) = LOWER(TRIM(expected_value)) 
  THEN 'EXACT_MATCH'
  ELSE 'MISMATCH'
END AS match_status
```

#### Fuzzy Match (for text fields):

```sql
-- Using Jaro-Winkler similarity (0-100 scale)
JAROWINKLER_SIMILARITY(
  LOWER(extracted_value), 
  LOWER(expected_value)
) AS similarity_score,

CASE 
  WHEN JAROWINKLER_SIMILARITY(LOWER(extracted_value), LOWER(expected_value)) >= 95 
       THEN 'EXACT_MATCH'
  WHEN JAROWINKLER_SIMILARITY(LOWER(extracted_value), LOWER(expected_value)) >= 85 
       THEN 'CLOSE_MATCH'
  WHEN JAROWINKLER_SIMILARITY(LOWER(extracted_value), LOWER(expected_value)) >= 70 
       THEN 'PARTIAL_MATCH'
  ELSE 'MISMATCH'
END AS match_status
```

#### Numeric Match (with tolerance):

```sql
-- Normalize and compare numbers
WITH normalized AS (
  SELECT 
    TRY_TO_DECIMAL(REGEXP_REPLACE(extracted_amount, '[^0-9.]', ''), 10, 2) AS extracted_num,
    TRY_TO_DECIMAL(REGEXP_REPLACE(expected_amount, '[^0-9.]', ''), 10, 2) AS expected_num
)
SELECT 
  CASE 
    WHEN extracted_num = expected_num THEN 'EXACT_MATCH'
    WHEN ABS(extracted_num - expected_num) / NULLIF(expected_num, 0) < 0.01 
         THEN 'WITHIN_1_PERCENT'
    WHEN ABS(extracted_num - expected_num) / NULLIF(expected_num, 0) < 0.05 
         THEN 'WITHIN_5_PERCENT'
    ELSE 'MISMATCH'
  END AS match_status
FROM normalized;
```

#### Date Match (with format flexibility):

```sql
-- Compare dates regardless of format
CASE 
  WHEN TRY_TO_DATE(extracted_date) = TRY_TO_DATE(expected_date) 
       THEN 'EXACT_MATCH'
  WHEN DATEDIFF('day', TRY_TO_DATE(extracted_date), TRY_TO_DATE(expected_date)) BETWEEN -1 AND 1 
       THEN 'OFF_BY_ONE_DAY'
  ELSE 'MISMATCH'
END AS date_match_status
```

#### Semantic Match (using LLM):

```sql
-- For fields where meaning matters more than exact text
SELECT AI_COMPLETE(
  'claude-3-5-sonnet',
  'Are these two values semantically equivalent? ' ||
  'Value 1: "' || extracted_value || '" ' ||
  'Value 2: "' || expected_value || '" ' ||
  'Return JSON: {"equivalent": true/false, "reason": "explanation"}',
  {'response_format': {'type': 'json'}, 'max_tokens': 256}
):equivalent::BOOLEAN AS is_semantic_match;
```

---

## Evaluation Method 3: Statistical Metrics

Calculate standard ML metrics for classification and extraction tasks.

### Classification Metrics

#### Accuracy:
```sql
SELECT 
  ROUND(
    SUM(CASE WHEN predicted = actual THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
    2
  ) AS accuracy_pct
FROM predictions;
```

#### Precision (per class):
```sql
-- Precision = TP / (TP + FP)
SELECT 
  category,
  ROUND(
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) * 100.0 /
    NULLIF(SUM(CASE WHEN predicted = category THEN 1 ELSE 0 END), 0),
    2
  ) AS precision_pct
FROM predictions
CROSS JOIN (SELECT DISTINCT actual AS category FROM predictions) c
GROUP BY category;
```

#### Recall (per class):
```sql
-- Recall = TP / (TP + FN)
SELECT 
  category,
  ROUND(
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) * 100.0 /
    NULLIF(SUM(CASE WHEN actual = category THEN 1 ELSE 0 END), 0),
    2
  ) AS recall_pct
FROM predictions
CROSS JOIN (SELECT DISTINCT actual AS category FROM predictions) c
GROUP BY category;
```

#### F1 Score (per class):
```sql
-- F1 = 2 * (Precision * Recall) / (Precision + Recall)
WITH class_metrics AS (
  SELECT 
    category,
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) AS tp,
    SUM(CASE WHEN predicted = category AND actual != category THEN 1 ELSE 0 END) AS fp,
    SUM(CASE WHEN predicted != category AND actual = category THEN 1 ELSE 0 END) AS fn
  FROM predictions
  CROSS JOIN (SELECT DISTINCT actual AS category FROM predictions) c
  GROUP BY category
)
SELECT 
  category,
  ROUND(tp * 100.0 / NULLIF(tp + fp, 0), 2) AS precision_pct,
  ROUND(tp * 100.0 / NULLIF(tp + fn, 0), 2) AS recall_pct,
  ROUND(
    2 * (tp * 1.0 / NULLIF(tp + fp, 0)) * (tp * 1.0 / NULLIF(tp + fn, 0)) /
    NULLIF((tp * 1.0 / NULLIF(tp + fp, 0)) + (tp * 1.0 / NULLIF(tp + fn, 0)), 0) * 100,
    2
  ) AS f1_score_pct
FROM class_metrics;
```

#### Confusion Matrix:
```sql
-- Generate confusion matrix
SELECT 
  actual AS actual_category,
  predicted AS predicted_category,
  COUNT(*) AS count
FROM predictions
GROUP BY actual, predicted
ORDER BY actual, predicted;
```

### Extraction Metrics

#### Field-Level Accuracy:
```sql
SELECT 
  '<field_name>' AS field,
  COUNT(*) AS total,
  SUM(CASE WHEN match_status = 'EXACT_MATCH' THEN 1 ELSE 0 END) AS exact_matches,
  SUM(CASE WHEN match_status IN ('EXACT_MATCH', 'CLOSE_MATCH') THEN 1 ELSE 0 END) AS acceptable_matches,
  ROUND(SUM(CASE WHEN match_status = 'EXACT_MATCH' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS exact_accuracy_pct,
  ROUND(SUM(CASE WHEN match_status IN ('EXACT_MATCH', 'CLOSE_MATCH') THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS fuzzy_accuracy_pct
FROM comparison_results;
```

#### Mean Absolute Error (for numeric fields):
```sql
SELECT 
  AVG(ABS(extracted_value - expected_value)) AS mae,
  AVG(ABS(extracted_value - expected_value) / NULLIF(expected_value, 0)) AS mape  -- Mean Absolute Percentage Error
FROM numeric_comparisons;
```

---

## Evaluation Method 4: A/B Comparison

Compare two versions of an AI function configuration.

### Setup A/B Test:

```sql
-- Run both versions on same data
CREATE TABLE ab_test_results AS
SELECT 
  record_id,
  input_data,
  
  -- Version A (current)
  AI_EXTRACT(
    file => TO_FILE('@stage', record_id),
    responseFormat => <config_A>
  ):response AS version_a_output,
  
  -- Version B (new)
  AI_EXTRACT(
    file => TO_FILE('@stage', record_id),
    responseFormat => <config_B>
  ):response AS version_b_output

FROM test_dataset;
```

### Compare Versions:

```sql
-- Compare A vs B using LLM judge
SELECT 
  record_id,
  PARSE_JSON(AI_COMPLETE(
    'claude-3-5-sonnet',
    'Compare these two extraction results and determine which is better. ' ||
    'Version A: ' || version_a_output::STRING ||
    ' Version B: ' || version_b_output::STRING ||
    ' Return JSON: {"winner": "A" or "B" or "tie", "reason": "explanation", "a_score": 1-10, "b_score": 1-10}',
    {'response_format': {'type': 'json'}, 'max_tokens': 512}
  )) AS comparison
FROM ab_test_results;

-- Aggregate A/B results
SELECT 
  SUM(CASE WHEN comparison:winner = 'A' THEN 1 ELSE 0 END) AS version_a_wins,
  SUM(CASE WHEN comparison:winner = 'B' THEN 1 ELSE 0 END) AS version_b_wins,
  SUM(CASE WHEN comparison:winner = 'tie' THEN 1 ELSE 0 END) AS ties,
  AVG(comparison:a_score::FLOAT) AS avg_a_score,
  AVG(comparison:b_score::FLOAT) AS avg_b_score
FROM ab_test_results;
```

---

## Choosing the Right Evaluation Method

| Scenario | Recommended Method | Why |
|----------|-------------------|-----|
| No labeled data available | LLM-as-Judge | Provides quality scores without ground truth |
| Have labeled test data | Ground Truth Comparison | Most accurate and reliable |
| Comparing model versions | A/B Comparison | Direct side-by-side comparison |
| Production monitoring | Quick Validation + Sampling | Low cost, catches regressions |
| Initial pipeline testing | LLM-as-Judge + Human review | Balances automation with human insight |
| Classification tasks | Statistical Metrics | Standard ML metrics apply directly |
| Subjective quality (summaries, translations) | LLM-as-Judge | Human-like quality assessment |
