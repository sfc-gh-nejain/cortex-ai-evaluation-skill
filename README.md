# Cortex AI Evaluation Skill

An interactive skill for [Cortex Code](https://docs.snowflake.com/user-guide/snowflake-cortex/cortex-agents) that helps you evaluate and test Snowflake Cortex AI function pipelines.

## Installation

Install this skill in Cortex Code using the following command:

```bash
cortex skill add https://github.com/sfc-gh-nejain/cortex-ai-evaluation-skill
```

After installation, the skill will be available in your Cortex Code environment and will automatically trigger on evaluation-related tasks.

To verify installation:
```bash
cortex skill list
```

## Overview

When processing unstructured data with Cortex AI functions like `AI_EXTRACT`, `AI_CLASSIFY`, or `AI_COMPLETE`, you need to validate that your pipeline produces accurate, high-quality results. This skill provides a guided, interactive experience for setting up comprehensive evaluations.

## Supported Cortex AI Functions

| Function | Description | Evaluation Focus |
|----------|-------------|------------------|
| `AI_EXTRACT` | Extract structured data from documents | Field accuracy, completeness |
| `AI_CLASSIFY` | Categorize text or images | Classification accuracy, F1 score |
| `AI_COMPLETE` | Generate text completions | Response quality, relevance |
| `AI_SENTIMENT` | Analyze sentiment | Score accuracy |
| `AI_SUMMARIZE_AGG` | Summarize multiple rows | Coverage, coherence |
| `AI_TRANSLATE` | Translate text | Fluency, adequacy |
| `AI_PARSE_DOCUMENT` | Parse document structure | Extraction completeness |
| `AI_TRANSCRIBE` | Transcribe audio/video | Transcription accuracy |

## Evaluation Methods

### 1. Quick Validation
Basic completeness and format checks without requiring ground truth data.
- Field presence checks
- Format validation
- Anomaly detection

### 2. LLM-as-Judge
Uses Claude to evaluate output quality on a 1-10 scale.
- Configurable evaluation criteria
- Detailed feedback and suggestions
- Works without labeled data

### 3. Ground Truth Comparison
Compare AI outputs against known correct values.
- Exact and fuzzy matching
- Precision, recall, F1 metrics
- Confusion matrix for classification

### 4. Human-in-the-Loop
Interactive review where you label samples and provide feedback.
- Random sampling
- Structured labeling interface
- Aggregated human evaluation metrics

### 5. Automated Regression Testing
Set up repeatable tests for CI/CD pipelines.
- Golden dataset tests
- Quality threshold monitoring
- Drift detection alerts

## Workflow

```
Start
  ↓
Step 1: Identify Pipeline to Evaluate
  ├─→ Which Cortex AI function?
  ├─→ Where is the output data?
  └─→ What does the output look like?
  ↓
Step 2: Choose Evaluation Type
  ├─→ Quick Validation
  ├─→ LLM-as-Judge
  ├─→ Ground Truth Comparison
  ├─→ Human-in-the-Loop
  └─→ Automated Regression
  ↓
Step 3: Configure Evaluation
  ├─→ Define success metrics & thresholds
  └─→ Select sample size / test data
  ↓
Step 4: Run Evaluation
  ├─→ Execute evaluation queries
  └─→ Display results with pass/fail
  ↓
Step 5: Act on Results
  ├─→ PASS → Deploy to production
  ├─→ FAIL → Refine prompts/schema
  ├─→ Export evaluation report
  └─→ Set up quality monitoring
```

## File Structure

```
cortex-ai-evaluation/
├── README.md                          # This file
├── SKILL.md                           # Main skill definition
└── reference/
    ├── evaluation-methods.md          # Detailed evaluation techniques
    └── metrics-guide.md               # Metrics definitions & thresholds
```

## Usage

### In Cortex Code

The skill will automatically trigger when you mention evaluation-related tasks:

```
> I want to evaluate my AI_EXTRACT pipeline
> Check the accuracy of my classification results
> Set up quality monitoring for my document extraction
```

Or invoke directly:
```
> /cortex-ai-evaluation
```

### Trigger Keywords

- **Evaluation:** evaluate, eval, assess, quality check, validate, verify
- **Metrics:** accuracy, precision, recall, F1 score, confusion matrix
- **Testing:** test pipeline, test extraction, regression test, A/B test
- **Ground truth:** ground truth, golden dataset, labeled data, expected values
- **Monitoring:** quality monitoring, drift detection, track performance

## Recommended Thresholds

### By Risk Level

| Risk Level | Accuracy | F1 Score | LLM Judge Score |
|-----------|----------|----------|-----------------|
| High (finance, healthcare) | ≥95% | ≥90% | ≥9/10 |
| Medium (business apps) | ≥90% | ≥85% | ≥8/10 |
| Low (internal/exploratory) | ≥80% | ≥75% | ≥7/10 |

## Example: Evaluating AI_EXTRACT

```sql
-- Quick validation for extraction completeness
SELECT 
  file_path,
  CASE WHEN response:invoice_number IS NOT NULL THEN 'OK' ELSE 'MISSING' END AS invoice_status,
  CASE WHEN response:vendor_name IS NOT NULL THEN 'OK' ELSE 'MISSING' END AS vendor_status,
  CASE WHEN response:total_amount IS NOT NULL THEN 'OK' ELSE 'MISSING' END AS amount_status
FROM extraction_results;

-- LLM-as-judge evaluation
SELECT 
  file_path,
  PARSE_JSON(AI_COMPLETE(
    'claude-3-5-sonnet',
    'Evaluate this extraction result 1-10: ' || response::STRING ||
    ' Return JSON: {"score": N, "issues": []}'
  )):score::INT AS quality_score
FROM extraction_results;
```

## Integration

This skill integrates with other Cortex Code skills:
- **document-intelligence** - Route back for extraction refinement when evaluation fails
- **text-classification** - Route back for classification tuning when accuracy is low

## License

Internal Snowflake use.

## Contributing

To improve this skill, edit the files and submit a PR to this repository.
