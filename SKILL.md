---
name: cortex-ai-evaluation
description: |
  **[REQUIRED]** For **ALL** evaluation, testing, and quality assessment of Cortex AI function outputs.
  Use when: evaluating AI_EXTRACT results, measuring classification accuracy, validating AI_COMPLETE outputs,
  testing AI pipelines, setting up quality monitoring, comparing model performance, ground truth testing.
  Triggers:
  - Evaluation: evaluate, eval, evaluation, assess, quality check, validate, verify, test accuracy
  - Metrics: accuracy, precision, recall, F1 score, confusion matrix, quality score, performance metrics
  - Testing: test pipeline, test extraction, test classification, test model, regression test, A/B test
  - Ground truth: ground truth, golden dataset, labeled data, expected values, correct answers
  - Monitoring: quality monitoring, drift detection, alert on quality, track performance
  - Functions: evaluate AI_EXTRACT, evaluate AI_CLASSIFY, evaluate AI_COMPLETE, evaluate AI_SENTIMENT
---

# Cortex AI Evaluation

Entry point for evaluating and testing Cortex AI function pipelines. This skill provides an interactive, guided experience for setting up model/function evaluation on your unstructured data processing pipelines.

## Reference Files

This skill uses reference documentation for detailed guidance:

| Reference | Location | Use For |
|-----------|----------|---------|
| Evaluation Methods | `reference/evaluation-methods.md` | LLM-as-judge, ground truth comparison techniques |
| Metrics Guide | `reference/metrics-guide.md` | Metrics definitions, thresholds, interpretation |

**Load reference context** when user needs specific details:
- For evaluation technique details → Read `reference/evaluation-methods.md`
- For metrics and thresholds → Read `reference/metrics-guide.md`

## When to Use

- User wants to evaluate AI_EXTRACT, AI_CLASSIFY, AI_COMPLETE, or other Cortex AI function outputs
- User mentions accuracy, quality, precision, recall, or F1 score
- User wants to compare AI outputs against ground truth / expected values
- User wants to set up quality monitoring for their AI pipeline
- User wants to test before deploying to production
- User mentions regression testing or A/B testing for AI functions

## Supported Cortex AI Functions

| Function | What It Does | Evaluation Focus |
|----------|--------------|------------------|
| AI_EXTRACT | Extract structured data from documents | Field accuracy, completeness |
| AI_CLASSIFY | Categorize text/images | Classification accuracy, F1 |
| AI_COMPLETE | Generate text completions | Response quality, relevance |
| AI_SENTIMENT | Analyze sentiment | Score accuracy, agreement |
| AI_SUMMARIZE_AGG | Summarize multiple rows | Coverage, coherence |
| AI_TRANSLATE | Translate text | Fluency, adequacy |
| AI_PARSE_DOCUMENT | Parse document structure | Extraction completeness |
| AI_TRANSCRIBE | Transcribe audio/video | Transcription accuracy |

## Workflow

```
Start
  ↓
Step 1: Identify Pipeline to Evaluate
  ↓
  ├─→ Which Cortex AI function?
  ├─→ Where is the output data?
  └─→ What does the output look like?
  ↓
Step 2: Choose Evaluation Type
  ↓
  ├─→ Quick Validation ─────────────→ Step 2a: Completeness & format checks
  │
  ├─→ LLM-as-Judge ─────────────────→ Step 2b: AI scores quality 1-10
  │
  ├─→ Ground Truth Comparison ──────→ Step 2c: Compare against known values
  │
  ├─→ Human-in-the-Loop ────────────→ Step 2d: Interactive review
  │
  └─→ Automated Regression ─────────→ Step 2e: CI/CD pipeline testing
  ↓
Step 3: Configure Evaluation
  ↓
  ├─→ Define success metrics & thresholds
  ├─→ Select sample size / test data
  └─→ Provide ground truth (if applicable)
  ↓
Step 4: Run Evaluation
  ↓
  ├─→ Execute evaluation queries
  ├─→ Calculate metrics
  └─→ Display results with pass/fail
  ↓
Step 5: Act on Results
  ↓
  ├─→ PASS (≥threshold) ────────────→ Deploy to production
  │
  ├─→ FAIL (<threshold) ────────────→ Refine prompts/schema
  │                                    (Loop to document-intelligence
  │                                     or text-classification skill)
  │
  ├─→ Export Report ────────────────→ Generate evaluation report
  │
  └─→ Set Up Monitoring ────────────→ Create scheduled quality checks
```

---

## Step 1: Identify Pipeline to Evaluate

**Ask** user about their pipeline using `ask_user_question`:

**Question 1 - Cortex AI Function:**
```
Which Cortex AI function are you evaluating?
Options:
1. AI_EXTRACT - Document/text extraction
2. AI_CLASSIFY - Text or image classification
3. AI_COMPLETE - LLM text generation
4. AI_SENTIMENT - Sentiment analysis
5. AI_TRANSLATE - Translation
6. AI_PARSE_DOCUMENT - Document parsing
7. AI_TRANSCRIBE - Audio/video transcription
8. Other / Multiple functions
```

**Question 2 - Data Location:**
```
Where is the AI function output data?
Options:
1. Results table - Already stored in a Snowflake table
2. Inline query - I'll run the AI function as part of evaluation
3. Staged files - Results are in files on a stage
4. Dynamic table - Continuous pipeline output
```

**If "Results table":**
```
Please provide:
1. The fully qualified table name (database.schema.table)
2. The column containing AI function output
3. Any identifier column (e.g., file_path, id)
```

**Verify the data:**
```sql
-- Preview the output data
SELECT * FROM <database>.<schema>.<results_table>
ORDER BY <timestamp_column> DESC
LIMIT 5;
```

**If "Inline query":**
```
I'll help you build an evaluation that runs the AI function and evaluates it in one step.
Please provide:
1. The source data location (table or stage)
2. The AI function configuration you're using
```

---

## Step 2: Choose Evaluation Type

**Ask** user using `ask_user_question`:

```
What type of evaluation do you need?
Options:
1. Quick Validation - Basic completeness and format checks (no ground truth needed)
2. LLM-as-Judge - AI evaluates quality on a 1-10 scale
3. Ground Truth Comparison - Compare against known correct values
4. Human-in-the-Loop - Interactive review of samples
5. Automated Regression - Set up CI/CD-style testing
```

### Route based on selection:
- **Quick Validation** → Step 2a
- **LLM-as-Judge** → Step 2b  
- **Ground Truth Comparison** → Step 2c
- **Human-in-the-Loop** → Step 2d
- **Automated Regression** → Step 2e

---

## Step 2a: Quick Validation

Performs basic quality checks without ground truth or LLM calls.

**Ask** user which checks to perform:
```
Which validation checks do you want to run?
Options:
1. All checks (completeness, format, null detection)
2. Completeness only - Check if required fields are present
3. Format validation - Check data types and patterns
4. Anomaly detection - Find unusual values
```

### For AI_EXTRACT Outputs:

**Ask** for the expected fields:
```
What fields should be present in the extraction?
Example: invoice_number, vendor_name, total_amount
```

```sql
-- Quick validation for AI_EXTRACT
WITH extraction_data AS (
  SELECT 
    <id_column> AS record_id,
    <output_column> AS raw_output,
    <output_column>:response AS response
  FROM <results_table>
)
SELECT 
  record_id,
  
  -- Completeness checks
  CASE WHEN response:<field1> IS NOT NULL AND response:<field1>::STRING != '' 
       THEN 'OK' ELSE 'MISSING' END AS <field1>_status,
  CASE WHEN response:<field2> IS NOT NULL AND response:<field2>::STRING != '' 
       THEN 'OK' ELSE 'MISSING' END AS <field2>_status,
  CASE WHEN response:<field3> IS NOT NULL AND response:<field3>::STRING != '' 
       THEN 'OK' ELSE 'MISSING' END AS <field3>_status,
  
  -- Overall completeness score
  ROUND(
    (IFF(response:<field1> IS NOT NULL AND response:<field1>::STRING != '', 1, 0) +
     IFF(response:<field2> IS NOT NULL AND response:<field2>::STRING != '', 1, 0) +
     IFF(response:<field3> IS NOT NULL AND response:<field3>::STRING != '', 1, 0)) 
    / 3.0 * 100, 1
  ) AS completeness_pct

FROM extraction_data;
```

**Summary statistics:**
```sql
-- Aggregated quality metrics
SELECT 
  COUNT(*) AS total_records,
  ROUND(AVG(completeness_pct), 1) AS avg_completeness,
  SUM(CASE WHEN completeness_pct = 100 THEN 1 ELSE 0 END) AS fully_complete,
  SUM(CASE WHEN completeness_pct < 50 THEN 1 ELSE 0 END) AS low_quality,
  MIN(completeness_pct) AS min_completeness,
  MAX(completeness_pct) AS max_completeness
FROM (
  -- validation query above
);
```

### For AI_CLASSIFY Outputs:

```sql
-- Quick validation for AI_CLASSIFY
SELECT 
  <id_column>,
  <output_column>:labels[0]::STRING AS classification,
  
  -- Check if classification is valid
  CASE 
    WHEN <output_column>:labels[0]::STRING IN ('<expected_categories>') THEN 'VALID'
    WHEN <output_column>:labels[0] IS NULL THEN 'MISSING'
    ELSE 'UNEXPECTED_CATEGORY'
  END AS validation_status
  
FROM <results_table>;

-- Category distribution
SELECT 
  <output_column>:labels[0]::STRING AS classification,
  COUNT(*) AS count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) AS percentage
FROM <results_table>
GROUP BY 1
ORDER BY count DESC;
```

### For AI_COMPLETE Outputs:

```sql
-- Quick validation for AI_COMPLETE
SELECT 
  <id_column>,
  <output_column> AS response,
  
  -- Basic checks
  CASE WHEN <output_column> IS NULL OR <output_column> = '' THEN 'EMPTY'
       WHEN LENGTH(<output_column>) < 10 THEN 'TOO_SHORT'
       WHEN LENGTH(<output_column>) > 50000 THEN 'TOO_LONG'
       ELSE 'OK'
  END AS response_status,
  
  LENGTH(<output_column>) AS response_length

FROM <results_table>;
```

**After quick validation, ask:**
```
Quick validation complete! Results:
- Total records: X
- Average completeness: Y%
- Records with issues: Z

What would you like to do next?
Options:
1. View detailed issues
2. Run deeper evaluation (LLM-as-judge)
3. Export validation report
4. Done
```

---

## Step 2b: LLM-as-Judge Evaluation

Uses Claude to evaluate AI function output quality on a 1-10 scale.

**Ask** user about evaluation criteria:
```
What aspects should the LLM judge evaluate?
Options:
1. Overall quality (accuracy, completeness, format)
2. Accuracy only - Is the extracted/classified data correct?
3. Completeness only - Are all expected items present?
4. Custom criteria - I'll specify what to evaluate
```

### For AI_EXTRACT Evaluation:

```sql
-- LLM-as-judge for AI_EXTRACT
SELECT 
  <id_column> AS record_id,
  <output_column>:response AS extracted_data,
  PARSE_JSON(AI_COMPLETE(
    'claude-3-5-sonnet',
    $$You are an expert at evaluating document extraction quality.

TASK: Evaluate the quality of this extraction result.

EXTRACTED DATA:
$$ || <output_column>:response::STRING || $$

EXTRACTION SCHEMA (expected fields):
- <field1>: <description1>
- <field2>: <description2>
- <field3>: <description3>

Evaluate each field on:
1. Is the value present and non-empty?
2. Does the format look correct for this field type?
3. Does the value seem plausible/realistic?

Return a JSON object:
{
  "overall_score": <1-10>,
  "field_scores": {
    "<field1>": {"score": <1-10>, "issue": "<null or description>"},
    "<field2>": {"score": <1-10>, "issue": "<null or description>"},
    "<field3>": {"score": <1-10>, "issue": "<null or description>"}
  },
  "summary": "<brief assessment>",
  "suggestions": ["<improvement 1>", "<improvement 2>"]
}$$,
    {'response_format': {'type': 'json'}, 'max_tokens': 1024}
  )) AS evaluation
FROM <results_table>
LIMIT 10;  -- Start with sample
```

### For AI_CLASSIFY Evaluation:

```sql
-- LLM-as-judge for AI_CLASSIFY
SELECT 
  <id_column> AS record_id,
  <text_column> AS input_text,
  <output_column>:labels[0]::STRING AS classification,
  PARSE_JSON(AI_COMPLETE(
    'claude-3-5-sonnet',
    $$You are an expert at evaluating text classification quality.

TASK: Evaluate if this classification is correct.

INPUT TEXT:
$$ || <text_column> || $$

ASSIGNED CLASSIFICATION: $$ || <output_column>:labels[0]::STRING || $$

AVAILABLE CATEGORIES:
- <category1>: <description1>
- <category2>: <description2>
- <category3>: <description3>

Evaluate:
1. Is the assigned category appropriate for this text?
2. Is there a better category that should have been chosen?
3. Is the classification confident or borderline?

Return a JSON object:
{
  "is_correct": <true/false>,
  "confidence": "<high/medium/low>",
  "correct_category": "<same or different category>",
  "reasoning": "<why the classification is or isn't correct>",
  "score": <1-10>
}$$,
    {'response_format': {'type': 'json'}, 'max_tokens': 512}
  )) AS evaluation
FROM <results_table>
LIMIT 10;
```

### For AI_COMPLETE Evaluation:

```sql
-- LLM-as-judge for AI_COMPLETE
SELECT 
  <id_column> AS record_id,
  <prompt_column> AS input_prompt,
  <output_column> AS response,
  PARSE_JSON(AI_COMPLETE(
    'claude-3-5-sonnet',
    $$You are an expert at evaluating LLM response quality.

TASK: Evaluate the quality of this AI-generated response.

ORIGINAL PROMPT:
$$ || <prompt_column> || $$

GENERATED RESPONSE:
$$ || <output_column> || $$

Evaluate on these dimensions:
1. Relevance: Does the response address the prompt?
2. Accuracy: Is the information factually correct?
3. Completeness: Does it fully answer the question?
4. Coherence: Is it well-structured and clear?
5. Helpfulness: Would this be useful to the user?

Return a JSON object:
{
  "overall_score": <1-10>,
  "dimension_scores": {
    "relevance": <1-10>,
    "accuracy": <1-10>,
    "completeness": <1-10>,
    "coherence": <1-10>,
    "helpfulness": <1-10>
  },
  "strengths": ["<strength1>", "<strength2>"],
  "weaknesses": ["<weakness1>", "<weakness2>"],
  "summary": "<brief assessment>"
}$$,
    {'response_format': {'type': 'json'}, 'max_tokens': 1024}
  )) AS evaluation
FROM <results_table>
LIMIT 10;
```

**Aggregate LLM-judge results:**
```sql
-- Summary of LLM-judge evaluation
SELECT 
  COUNT(*) AS total_evaluated,
  ROUND(AVG(evaluation:overall_score::FLOAT), 2) AS avg_score,
  SUM(CASE WHEN evaluation:overall_score::INT >= 8 THEN 1 ELSE 0 END) AS high_quality,
  SUM(CASE WHEN evaluation:overall_score::INT BETWEEN 5 AND 7 THEN 1 ELSE 0 END) AS medium_quality,
  SUM(CASE WHEN evaluation:overall_score::INT < 5 THEN 1 ELSE 0 END) AS low_quality,
  ROUND(AVG(CASE WHEN evaluation:is_correct::BOOLEAN THEN 1 ELSE 0 END) * 100, 1) AS accuracy_pct
FROM evaluation_results;
```

---

## Step 2c: Ground Truth Comparison

Compare AI function outputs against known correct values.

**Ask** user about ground truth data:
```
How will you provide ground truth data?
Options:
1. Enter values manually - For a few test samples
2. Load from table - Ground truth is in a Snowflake table
3. Upload file - I have a CSV/Excel with correct values
4. Sample & label - Help me create ground truth from samples
```

### If "Enter values manually":

**Ask** for the test file/record and expected values:
```
For each test record, please provide the expected values.

Record 1: <file_path or id>
- <field1>: [expected value]
- <field2>: [expected value]
- <field3>: [expected value]
```

### If "Load from table":

**Ask** for table details:
```
Please provide:
1. Ground truth table name (database.schema.table)
2. Join key column (e.g., file_path, id)
3. Expected value columns
```

### Create/verify ground truth table:

```sql
-- Ground truth table schema
CREATE TABLE IF NOT EXISTS <database>.<schema>.evaluation_ground_truth (
  record_id STRING PRIMARY KEY,
  
  -- For AI_EXTRACT
  expected_field1 STRING,
  expected_field2 STRING,
  expected_field3 STRING,
  
  -- For AI_CLASSIFY
  expected_category STRING,
  
  -- For AI_SENTIMENT
  expected_sentiment FLOAT,
  
  -- Metadata
  created_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  created_by STRING DEFAULT CURRENT_USER()
);

-- Insert ground truth
INSERT INTO <database>.<schema>.evaluation_ground_truth 
  (record_id, expected_field1, expected_field2, expected_field3)
VALUES 
  ('<id1>', '<value1>', '<value2>', '<value3>'),
  ('<id2>', '<value1>', '<value2>', '<value3>');
```

### Comparison Queries:

**For AI_EXTRACT:**
```sql
-- Ground truth comparison for AI_EXTRACT
WITH comparison AS (
  SELECT 
    e.<id_column> AS record_id,
    
    -- Field 1 comparison
    e.<output_column>:response:<field1>::STRING AS extracted_field1,
    g.expected_field1,
    CASE 
      WHEN LOWER(TRIM(e.<output_column>:response:<field1>::STRING)) = LOWER(TRIM(g.expected_field1)) 
           THEN 'EXACT_MATCH'
      WHEN JAROWINKLER_SIMILARITY(
             LOWER(e.<output_column>:response:<field1>::STRING), 
             LOWER(g.expected_field1)
           ) > 90 THEN 'FUZZY_MATCH'
      WHEN e.<output_column>:response:<field1> IS NULL THEN 'MISSING'
      ELSE 'MISMATCH'
    END AS field1_status,
    JAROWINKLER_SIMILARITY(
      LOWER(COALESCE(e.<output_column>:response:<field1>::STRING, '')), 
      LOWER(COALESCE(g.expected_field1, ''))
    ) AS field1_similarity,
    
    -- Field 2 comparison
    e.<output_column>:response:<field2>::STRING AS extracted_field2,
    g.expected_field2,
    CASE 
      WHEN LOWER(TRIM(e.<output_column>:response:<field2>::STRING)) = LOWER(TRIM(g.expected_field2)) 
           THEN 'EXACT_MATCH'
      WHEN JAROWINKLER_SIMILARITY(
             LOWER(e.<output_column>:response:<field2>::STRING), 
             LOWER(g.expected_field2)
           ) > 90 THEN 'FUZZY_MATCH'
      WHEN e.<output_column>:response:<field2> IS NULL THEN 'MISSING'
      ELSE 'MISMATCH'
    END AS field2_status,
    
    -- Field 3 comparison (numeric)
    TRY_TO_DECIMAL(REGEXP_REPLACE(e.<output_column>:response:<field3>::STRING, '[^0-9.]', ''), 10, 2) AS extracted_field3,
    TRY_TO_DECIMAL(REGEXP_REPLACE(g.expected_field3, '[^0-9.]', ''), 10, 2) AS expected_field3_num,
    CASE 
      WHEN TRY_TO_DECIMAL(REGEXP_REPLACE(e.<output_column>:response:<field3>::STRING, '[^0-9.]', ''), 10, 2) =
           TRY_TO_DECIMAL(REGEXP_REPLACE(g.expected_field3, '[^0-9.]', ''), 10, 2)
           THEN 'EXACT_MATCH'
      WHEN e.<output_column>:response:<field3> IS NULL THEN 'MISSING'
      ELSE 'MISMATCH'
    END AS field3_status

  FROM <results_table> e
  JOIN <database>.<schema>.evaluation_ground_truth g 
    ON e.<id_column> = g.record_id
)
SELECT 
  *,
  -- Overall accuracy
  ROUND(
    (IFF(field1_status IN ('EXACT_MATCH', 'FUZZY_MATCH'), 1, 0) +
     IFF(field2_status IN ('EXACT_MATCH', 'FUZZY_MATCH'), 1, 0) +
     IFF(field3_status = 'EXACT_MATCH', 1, 0)) 
    / 3.0 * 100, 1
  ) AS accuracy_pct
FROM comparison;
```

**For AI_CLASSIFY (Confusion Matrix):**
```sql
-- Ground truth comparison for AI_CLASSIFY
WITH comparison AS (
  SELECT 
    e.<id_column> AS record_id,
    e.<output_column>:labels[0]::STRING AS predicted_category,
    g.expected_category AS actual_category,
    CASE 
      WHEN e.<output_column>:labels[0]::STRING = g.expected_category THEN 1 
      ELSE 0 
    END AS is_correct
  FROM <results_table> e
  JOIN <database>.<schema>.evaluation_ground_truth g 
    ON e.<id_column> = g.record_id
)
-- Confusion matrix
SELECT 
  actual_category,
  predicted_category,
  COUNT(*) AS count
FROM comparison
GROUP BY actual_category, predicted_category
ORDER BY actual_category, predicted_category;

-- Accuracy metrics
SELECT 
  COUNT(*) AS total_samples,
  SUM(is_correct) AS correct_predictions,
  ROUND(AVG(is_correct) * 100, 2) AS accuracy_pct,
  
  -- Per-class metrics (for multi-class)
  COUNT(DISTINCT actual_category) AS num_classes
FROM comparison;
```

**Calculate Precision, Recall, F1 per class:**
```sql
-- Per-class precision, recall, F1 for classification
WITH predictions AS (
  SELECT 
    e.<output_column>:labels[0]::STRING AS predicted,
    g.expected_category AS actual
  FROM <results_table> e
  JOIN <database>.<schema>.evaluation_ground_truth g 
    ON e.<id_column> = g.record_id
),
class_metrics AS (
  SELECT 
    category,
    SUM(CASE WHEN predicted = category AND actual = category THEN 1 ELSE 0 END) AS true_positives,
    SUM(CASE WHEN predicted = category AND actual != category THEN 1 ELSE 0 END) AS false_positives,
    SUM(CASE WHEN predicted != category AND actual = category THEN 1 ELSE 0 END) AS false_negatives
  FROM predictions
  CROSS JOIN (SELECT DISTINCT actual AS category FROM predictions) classes
  GROUP BY category
)
SELECT 
  category,
  true_positives,
  false_positives,
  false_negatives,
  ROUND(true_positives / NULLIF(true_positives + false_positives, 0) * 100, 2) AS precision_pct,
  ROUND(true_positives / NULLIF(true_positives + false_negatives, 0) * 100, 2) AS recall_pct,
  ROUND(
    2 * (true_positives / NULLIF(true_positives + false_positives, 0)) * 
        (true_positives / NULLIF(true_positives + false_negatives, 0)) /
    NULLIF(
      (true_positives / NULLIF(true_positives + false_positives, 0)) + 
      (true_positives / NULLIF(true_positives + false_negatives, 0)), 0
    ) * 100, 2
  ) AS f1_score_pct
FROM class_metrics
ORDER BY category;
```

---

## Step 2d: Human-in-the-Loop Evaluation

Interactive review where user labels samples and provides feedback.

**Ask** user about sample selection:
```
How many samples would you like to review?
Options:
1. Quick review - 5 samples
2. Standard review - 10-20 samples
3. Thorough review - 50+ samples
4. Custom - I'll specify the number
```

### Generate review samples:

```sql
-- Select random samples for human review
CREATE OR REPLACE TEMPORARY TABLE human_review_samples AS
SELECT 
  <id_column> AS record_id,
  <input_column> AS input_data,
  <output_column> AS ai_output,
  NULL AS human_label,
  NULL AS human_score,
  NULL AS human_feedback,
  CURRENT_TIMESTAMP() AS sampled_at
FROM <results_table>
ORDER BY RANDOM()
LIMIT <sample_size>;

-- Display for review
SELECT 
  record_id,
  input_data,
  ai_output
FROM human_review_samples;
```

**For each sample, ask user:**
```
Sample X of Y:

INPUT:
<input_data>

AI OUTPUT:
<ai_output>

Please evaluate:
1. Is this output correct? (Yes / Partially / No)
2. Quality score (1-10):
3. Any feedback or notes:
```

**Store human labels:**
```sql
-- Update with human evaluation
UPDATE human_review_samples
SET 
  human_label = '<correct/partially/incorrect>',
  human_score = <1-10>,
  human_feedback = '<user feedback>'
WHERE record_id = '<record_id>';
```

**Calculate human evaluation metrics:**
```sql
-- Human evaluation summary
SELECT 
  COUNT(*) AS samples_reviewed,
  ROUND(AVG(human_score), 2) AS avg_human_score,
  SUM(CASE WHEN human_label = 'correct' THEN 1 ELSE 0 END) AS correct_count,
  SUM(CASE WHEN human_label = 'partially' THEN 1 ELSE 0 END) AS partial_count,
  SUM(CASE WHEN human_label = 'incorrect' THEN 1 ELSE 0 END) AS incorrect_count,
  ROUND(
    SUM(CASE WHEN human_label = 'correct' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1
  ) AS accuracy_pct
FROM human_review_samples
WHERE human_label IS NOT NULL;
```

---

## Step 2e: Automated Regression Testing

Set up repeatable tests for CI/CD pipelines.

**Ask** user about test configuration:
```
What type of regression testing do you need?
Options:
1. Golden dataset test - Compare against fixed expected outputs
2. Quality threshold test - Ensure metrics stay above threshold
3. A/B comparison - Compare two model versions
4. Drift detection - Alert when quality changes significantly
```

### Golden Dataset Test:

```sql
-- Create regression test procedure
CREATE OR REPLACE PROCEDURE <database>.<schema>.run_extraction_regression_test()
RETURNS VARIANT
LANGUAGE SQL
AS
$$
DECLARE
  test_results VARIANT;
  pass_count INT;
  fail_count INT;
  accuracy FLOAT;
BEGIN
  -- Run AI function on golden dataset
  CREATE OR REPLACE TEMPORARY TABLE regression_results AS
  SELECT 
    g.record_id,
    g.expected_field1,
    g.expected_field2,
    AI_EXTRACT(
      file => TO_FILE('<stage>', g.record_id),
      responseFormat => <your_response_format>
    ):response AS actual_output,
    CASE 
      WHEN AI_EXTRACT(...):response:<field1>::STRING = g.expected_field1 
      THEN 'PASS' ELSE 'FAIL' 
    END AS test_status
  FROM <database>.<schema>.golden_dataset g;
  
  -- Calculate metrics
  SELECT COUNT(*) INTO pass_count FROM regression_results WHERE test_status = 'PASS';
  SELECT COUNT(*) INTO fail_count FROM regression_results WHERE test_status = 'FAIL';
  SELECT pass_count * 100.0 / (pass_count + fail_count) INTO accuracy;
  
  -- Return results
  test_results := OBJECT_CONSTRUCT(
    'test_name', 'extraction_regression',
    'run_time', CURRENT_TIMESTAMP(),
    'total_tests', pass_count + fail_count,
    'passed', pass_count,
    'failed', fail_count,
    'accuracy_pct', accuracy,
    'status', IFF(accuracy >= 90, 'PASS', 'FAIL')
  );
  
  -- Log results
  INSERT INTO <database>.<schema>.regression_test_log 
  SELECT test_results;
  
  RETURN test_results;
END;
$$;

-- Run the test
CALL <database>.<schema>.run_extraction_regression_test();
```

### Quality Threshold Test:

```sql
-- Create threshold monitoring task
CREATE OR REPLACE TASK <database>.<schema>.quality_threshold_monitor
  WAREHOUSE = <warehouse>
  SCHEDULE = 'USING CRON 0 8 * * * UTC'  -- Daily at 8 AM UTC
AS
BEGIN
  -- Calculate current quality metrics
  LET current_accuracy FLOAT := (
    SELECT AVG(CASE WHEN validation_status = 'PASS' THEN 1 ELSE 0 END) * 100
    FROM <results_table>
    WHERE created_at >= DATEADD('day', -1, CURRENT_TIMESTAMP())
  );
  
  LET threshold FLOAT := 90.0;
  
  -- Log and alert if below threshold
  IF (current_accuracy < threshold) THEN
    INSERT INTO <database>.<schema>.quality_alerts
    VALUES (
      CURRENT_TIMESTAMP(),
      'QUALITY_BELOW_THRESHOLD',
      current_accuracy,
      threshold,
      'Accuracy dropped to ' || current_accuracy || '%, below threshold of ' || threshold || '%'
    );
  END IF;
END;

ALTER TASK <database>.<schema>.quality_threshold_monitor RESUME;
```

---

## Step 3: Configure Evaluation

**Ask** user about success criteria:
```
What are your success criteria?
Options:
1. Standard thresholds (90% accuracy for production)
2. Custom thresholds - I'll specify my requirements
3. Relative improvement - Compare to baseline
```

### Standard Thresholds by Use Case:

| Metric | Production Ready | Acceptable | Needs Work |
|--------|-----------------|------------|------------|
| Overall Accuracy | ≥90% | 80-89% | <80% |
| Field Completeness | ≥95% | 85-94% | <85% |
| LLM Judge Score | ≥8/10 | 6-7/10 | <6/10 |
| Classification F1 | ≥85% | 75-84% | <75% |

**If custom thresholds:**
```
Please specify your thresholds:
- Minimum accuracy (%): [default: 90]
- Minimum completeness (%): [default: 95]
- Minimum LLM judge score (1-10): [default: 8]
- Sample size for evaluation: [default: 50]
```

---

## Step 4: Run Evaluation

Execute the configured evaluation and display results.

```sql
-- Run comprehensive evaluation
WITH evaluation_results AS (
  -- Your evaluation query from Steps 2a-2e
  SELECT 
    record_id,
    <metrics_columns>,
    <scores_columns>
  FROM <evaluation_query>
)
SELECT 
  '=== EVALUATION RESULTS ===' AS section,
  NULL AS metric,
  NULL AS value,
  NULL AS status
UNION ALL
SELECT 
  'Summary',
  'Total Records Evaluated',
  COUNT(*)::STRING,
  NULL
FROM evaluation_results
UNION ALL
SELECT 
  'Summary',
  'Average Accuracy',
  ROUND(AVG(accuracy_pct), 1)::STRING || '%',
  CASE WHEN AVG(accuracy_pct) >= 90 THEN 'PASS' ELSE 'FAIL' END
FROM evaluation_results
UNION ALL
SELECT 
  'Summary',
  'Average LLM Judge Score',
  ROUND(AVG(llm_score), 2)::STRING || '/10',
  CASE WHEN AVG(llm_score) >= 8 THEN 'PASS' ELSE 'FAIL' END
FROM evaluation_results;
```

**Display results to user:**
```
╔══════════════════════════════════════════════════════════════╗
║                    EVALUATION RESULTS                         ║
╠══════════════════════════════════════════════════════════════╣
║  Total Records Evaluated:  100                                ║
║  ────────────────────────────────────────────────────────────║
║  METRICS                    VALUE           STATUS            ║
║  ────────────────────────────────────────────────────────────║
║  Overall Accuracy:          92.3%           ✓ PASS            ║
║  Average Completeness:      97.1%           ✓ PASS            ║
║  LLM Judge Score:           8.4/10          ✓ PASS            ║
║  Records with Issues:       8               ⚠ REVIEW          ║
║  ────────────────────────────────────────────────────────────║
║  OVERALL STATUS:            PASS - Ready for production       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Step 5: Act on Results

**Ask** user using `ask_user_question`:

```
Evaluation complete! What would you like to do?
Options:
1. View detailed results - See individual record scores
2. Investigate failures - Drill into low-scoring records
3. Export report - Generate evaluation report
4. Set up monitoring - Create scheduled quality checks
5. Refine pipeline - Improve prompts/schema (requires refinement)
6. Done
```

### If "Investigate failures":

```sql
-- Show lowest-scoring records
SELECT 
  record_id,
  ai_output,
  accuracy_pct,
  llm_score,
  issues
FROM evaluation_results
WHERE accuracy_pct < 90 OR llm_score < 8
ORDER BY accuracy_pct ASC, llm_score ASC
LIMIT 20;
```

### If "Export report":

```sql
-- Create evaluation report table
CREATE OR REPLACE TABLE <database>.<schema>.evaluation_report_<timestamp> AS
SELECT 
  'Evaluation Report' AS report_type,
  CURRENT_TIMESTAMP() AS generated_at,
  '<pipeline_name>' AS pipeline_name,
  '<function_type>' AS function_type,
  COUNT(*) AS total_evaluated,
  ROUND(AVG(accuracy_pct), 2) AS avg_accuracy,
  ROUND(AVG(llm_score), 2) AS avg_llm_score,
  SUM(CASE WHEN status = 'PASS' THEN 1 ELSE 0 END) AS passed,
  SUM(CASE WHEN status = 'FAIL' THEN 1 ELSE 0 END) AS failed,
  CASE WHEN AVG(accuracy_pct) >= 90 THEN 'PASS' ELSE 'FAIL' END AS overall_status
FROM evaluation_results;

-- Export to CSV
COPY INTO @<stage>/evaluation_report_<timestamp>.csv
FROM <database>.<schema>.evaluation_report_<timestamp>
FILE_FORMAT = (TYPE = 'CSV' HEADER = TRUE);
```

### If "Set up monitoring":

```sql
-- Create quality monitoring infrastructure
CREATE OR REPLACE TABLE <database>.<schema>.quality_audit_log (
  audit_id STRING DEFAULT UUID_STRING(),
  audit_time TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  pipeline_name STRING,
  records_checked INT,
  avg_accuracy FLOAT,
  avg_completeness FLOAT,
  avg_llm_score FLOAT,
  status STRING,
  alerts ARRAY
);

-- Create monitoring task
CREATE OR REPLACE TASK <database>.<schema>.quality_monitor_<pipeline>
  WAREHOUSE = <warehouse>
  SCHEDULE = 'USING CRON 0 9 * * * UTC'  -- Daily at 9 AM
AS
INSERT INTO <database>.<schema>.quality_audit_log 
  (pipeline_name, records_checked, avg_accuracy, avg_completeness, avg_llm_score, status, alerts)
SELECT 
  '<pipeline_name>',
  COUNT(*),
  AVG(accuracy_pct),
  AVG(completeness_pct),
  AVG(llm_score),
  CASE WHEN AVG(accuracy_pct) >= 90 THEN 'HEALTHY' ELSE 'DEGRADED' END,
  CASE 
    WHEN AVG(accuracy_pct) < 90 
    THEN ARRAY_CONSTRUCT('Accuracy below 90%: ' || ROUND(AVG(accuracy_pct), 1) || '%')
    ELSE ARRAY_CONSTRUCT()
  END
FROM (
  -- Run evaluation on recent data
  SELECT * FROM <evaluation_query>
  WHERE created_at >= DATEADD('day', -1, CURRENT_TIMESTAMP())
);

ALTER TASK <database>.<schema>.quality_monitor_<pipeline> RESUME;
```

### If "Refine pipeline":

Route user to the appropriate skill for refinement:

**For AI_EXTRACT issues:**
```
The evaluation identified extraction quality issues. 
I'll help you refine the extraction prompts/schema.

Loading document-intelligence skill...
```
**Load** the `document-intelligence` skill and go to Step 2.5 (Test Run & Prompt Refinement).

**For AI_CLASSIFY issues:**
```
The evaluation identified classification issues.
I'll help you refine the categories or add examples.

Loading text-classification skill...
```
**Load** the `text-classification` skill and go to Step 2.5 (Prompt Refinement).

---

## Quick Reference: Evaluation SQL Templates

### Template 1: Quick Validation
```sql
SELECT 
  COUNT(*) AS total,
  SUM(CASE WHEN <output>:<field> IS NOT NULL THEN 1 ELSE 0 END) AS has_field,
  ROUND(AVG(CASE WHEN <output>:<field> IS NOT NULL THEN 1 ELSE 0 END) * 100, 1) AS completeness_pct
FROM <table>;
```

### Template 2: LLM-as-Judge
```sql
SELECT 
  <id>,
  PARSE_JSON(AI_COMPLETE('claude-3-5-sonnet', 
    'Evaluate this output 1-10: ' || <output>::STRING || 
    ' Return JSON: {"score": N, "issues": []}'
  )):score::INT AS llm_score
FROM <table>;
```

### Template 3: Ground Truth Comparison
```sql
SELECT 
  e.<id>,
  CASE WHEN e.<extracted> = g.<expected> THEN 'MATCH' ELSE 'MISMATCH' END AS status,
  JAROWINKLER_SIMILARITY(e.<extracted>, g.<expected>) AS similarity
FROM <results> e JOIN <ground_truth> g ON e.<id> = g.<id>;
```

### Template 4: Classification Accuracy
```sql
SELECT 
  ROUND(AVG(CASE WHEN predicted = actual THEN 1 ELSE 0 END) * 100, 2) AS accuracy_pct
FROM (
  SELECT e.<output>:labels[0]::STRING AS predicted, g.expected_category AS actual
  FROM <results> e JOIN <ground_truth> g ON e.<id> = g.<id>
);
```

---

## Best Practices

1. **Start with Quick Validation** - Catch obvious issues before expensive LLM evaluations
2. **Sample Appropriately** - 50-100 samples is usually sufficient for initial evaluation
3. **Use Ground Truth When Possible** - Most reliable evaluation method
4. **Set Up Monitoring Early** - Catch regressions before they impact users
5. **Document Thresholds** - Record why you chose specific acceptance criteria
6. **Iterate Based on Failures** - Use evaluation insights to improve prompts

## Troubleshooting

**"LLM judge scores are inconsistent"**
- Add more specific evaluation criteria to the prompt
- Use structured output format for consistent JSON responses
- Run multiple evaluations and average the scores

**"Ground truth comparison shows many false mismatches"**
- Check for formatting differences (whitespace, case)
- Use fuzzy matching (JAROWINKLER_SIMILARITY) for text fields
- Normalize numeric values before comparison

**"Evaluation is too slow"**
- Start with smaller sample sizes
- Use Quick Validation before LLM-as-Judge
- Run evaluations during off-peak hours
