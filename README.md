# IT Incident SLA Compliance Analysis

This project analyzes IT incident management data to identify key factors affecting SLA (Service Level Agreement) compliance and develops machine learning models to predict SLA breaches. The work is designed for COMP647's practical machine learning requirements and aligns with my industrial internship project involving Accelo CRM integration with LLM-based chatbots.

## Project Overview

For my Machine Learning project, I chose the Incident Log Dataset from ServiceNow audit logs available on Kaggle. This dataset, with over 141,000 events and ~24,918 unique IT incidents, is ideal as it perfectly aligns with both COMP647's educational objectives and my industry internship project (COMP693), enabling rigorous academic analysis and practical real-world application.

**Dataset Source**: [Incident Event Log Dataset on Kaggle](https://www.kaggle.com/datasets/mithilkotawadekar/incident-event-log-dataset)

## Repository Structure

```
.
├── data/
│   ├── incident_event_log_dataset.csv  # Raw event log data (141,712 records)
│   └── incidents_cleaned.csv          # Preprocessed data (6,729 records)
├── task/
│   ├── assignment02_revised.ipynb     # EDA and data preprocessing analysis
│   └── assignment03.ipynb            # Feature Engineering and modeling
├── requirements.txt
└── README.md
```

## Dataset Overview

### Raw Data

- **Source**: ServiceNow-style incident event logs from Kaggle
- **Scale**: 141,712 records across 37 features
- **Structure**: Multi-event log format with multiple records per incident (average 5.7 events per incident)
- **Coverage**: 24,918 unique incidents from 2010-2016 (6 years of historical data)

### Analysis Dataset

- **Final Sample Size**: 6,729 unique completed incidents
- **Preprocessing Steps**:
  - Filtered for closed incidents (Resolved/Closed states)
  - Deduplicated to keep latest record per incident
  - Removed invalid timestamps (closed_at < opened_at)
  - Result: 141,712 → 6,729 records (85.5% filtered out)

### Target Variable

- **Target**: `made_sla` (SLA compliance boolean)
  - SLA Met: 85.9% (5,781 incidents)
  - SLA Breach: 14.1% (948 incidents)
  - Class Imbalance Ratio: 6.1:1

### Key Variables

- **Identifiers & Timestamps**: `number`, `opened_at`, `closed_at`, `sys_created_at`, `sys_updated_at`
- **State & Ownership**: `incident_state`, `assignment_group`, `assigned_to`
- **Classification**: `priority` (Critical, High, Moderate, Low), `impact`, `urgency`, `category`, `subcategory`
- **Operational Metrics**: `reassignment_count`, `sys_mod_count`, `reopen_count`
- **Process Variables**: `knowledge`, `u_priority_confirmation`, `notify`

## Research Question

**"What operational and organizational factors most strongly predict SLA compliance outcomes in IT incident management?"**

## Analysis Workflow

### Assignment 2: Exploratory Data Analysis (EDA)

#### 1. Data Preprocessing

- Filtered for closed incidents with valid timestamps
- Deduplicated to keep latest closed record per incident number
- Removed logically impossible records (closed_at < opened_at)
- Result: 141,712 → 6,729 unique incidents

#### 2. Missing Data Strategy

Three-tier approach:

- **Tier 1 (>95% missing)**: Removed 5 features
  - `caused_by` (100%), `vendor` (99.9%), `cmdb_ci` (99.8%), `change` (99.6%), `problem_id` (98.0%)
- **Tier 2 (5-95% missing)**: Kept 5 features (may contain useful information)
- **Tier 3 (<5% missing)**: Kept 8 features
- **Final**: 32 features retained

#### 3. Outlier Analysis

- Method: IQR (Interquartile Range) detection
- Findings: Outliers show significant SLA performance differences (22-40%p gaps)
- Decision: Retained outliers as they represent genuinely complex cases
- Impact: High-reassignment incidents (66.0% SLA) vs normal (88.3% SLA) = 22.3%p gap

#### 4. Key Findings

**Operational Complexity Dominates SLA Outcomes**

- System modification count (`sys_mod_count`): r = -0.368 (strongest predictor)
- Reassignment count (`reassignment_count`): r = -0.248 (second strongest)
- 0 reassignments: 92.6% SLA → 1 reassignment: 84.7% (7.9%p drop)
- Threshold effect: >3 modifications shows performance cliff (<95% SLA)

**Priority Classification Paradox**

- Critical priority: 53.3% SLA compliance
- Low priority: 95.5% SLA compliance
- Performance gap: 42.2 percentage points

**Organizational Variance**

- Assignment Group performance range: 52.2% ~ 100.0% (47.8%p gap)
- Category performance range: 67.9% ~ 94.3% (26.4%p gap)
- Group 64: Perfect 100% SLA across 212 incidents

**Process Variable Impact**

- Knowledge base consultation: +3.2%p improvement
- Priority confirmation: -14.7%p (serves as complexity indicator)

### Assignment 3: Feature Engineering & Modeling

#### 1. Feature Engineering

**Operational Complexity Features (4 features)**

- `activity_score`: sys_mod_count + (reassignment_count × 2)
- `has_reassignment`: Binary flag for any reassignment
- `high_modification`: Binary flag for >3 modifications
- `is_complex`: Composite flag (>4 mods OR >2 reassignments)

**Severity Features (4 features)**

- `is_high_priority`, `is_high_impact`, `is_high_urgency`
- `is_high_severity`: Composite OR logic (any dimension high)

**Interaction Features (3 features)**

- `severity_complexity`: High severity × complexity
- `confirmed_complex`: Priority confirmation × complexity
- `priority_reassign`: High priority × reassignment

**Total**: 11 new features created (32 → 43 features)

#### 2. Feature Selection

- **ANOVA F-test**: Statistical significance-based selection (filter method)
- **Random Forest Importance**: Model-based importance (embedded method)
- **Final Selection**: 13 features (union of both methods)
- **Dimensionality Reduction**: 35% (20 → 13 features)

#### 3. Model Training & Evaluation

**Models Trained**

1. Logistic Regression (linear baseline)
2. Random Forest (non-linear ensemble)
3. Gradient Boosting (sequential boosting)

**Final Selected Model: Random Forest**

- Test F1-Score: 0.5644 (Class 0 - SLA Breach)
- Test Recall: 84.2% (SLA breach detection rate)
- Test Precision: 42.44%
- Train-Test Gap: 8.4% (good generalization)
- Cross-Validation F1: 0.7244 (±0.0130)

**Performance Metrics**

- ROC-AUC: 0.8986 (~0.90) (excellent discrimination benchmark)
- Average Precision: 0.52 (271.1% improvement over 14.1% baseline)
- Class Imbalance Handling: (class_weight='balanced'; ~3.55× loss weight for breach vs ~0.58× for met, ≈6.1× relative)

#### 4. Explainable AI (XAI)

**SHAP Analysis Results**

- SHAP waterfall plots provide individual prediction explanations for high-risk incidents
- Feature importance from SHAP analysis reveals which features drive individual predictions

**Top 5 Feature Importance**

1. sys_mod_count (20.75%)
2. confirmed_complex (15.63%)
3. is_complex (15.61%)
4. activity_score (11.71%)
5. high_modification (9.74%)

## Key Insights

1. **Operational efficiency matters more than initial severity**: System modification count and reassignment count are stronger predictors than priority classification.

2. **Critical importance of first-touch resolution**: Zero-reassignment incidents achieve 92.6% SLA, but performance drops sharply after the first reassignment.

3. **Significant organizational factor impact**: Identical operational metrics show up to 47.8%p performance gaps across different assignment groups.

4. **Effectiveness of complexity flags**: `is_complex` flag distinguishes simple incidents (98.3% SLA) from complex ones (68.9% SLA) with a 29.4%p gap.

## Business Recommendations

1. **Improve initial routing accuracy**: Getting the first assignment correct is most critical for SLA performance improvement.

2. **Early identification of complex incidents**: Provide additional support when incidents exceed 4 system modifications or 2 reassignments.

3. **Support for low-performing groups**: Specialized training and process improvement for Assignment Groups 31, 10, and 9.

4. **Category 46 improvement priority**: Focus on Category 46 with high volume (579 incidents) and low SLA (76.7%).

5. **Deploy predictive model**: Integrate Random Forest model into production to identify high-risk incidents early and prioritize resource allocation.

## Setup & Usage

### Environment Setup

```bash
pip install -r requirements.txt
```

### Required Libraries

- **Data Processing**: pandas>=1.0.0, numpy>=1.19.0
- **Statistical Analysis**: scipy>=1.5.0
- **Visualization**: matplotlib>=3.2.0, seaborn>=0.11.0
- **Machine Learning**: scikit-learn>=0.24.0, imbalanced-learn>=0.8.0
- **Explainable AI**: shap>=0.40.0
- **Notebook Execution**: ipykernel
