# IT Incident SLA Compliance Analysis

## Overview

This repository contains comprehensive exploratory data analysis on IT incident SLA compliance patterns to understand factors that drive service delivery performance. The analysis investigates the complex interplay between incident classification systems, organizational factors, process adherence variables, and temporal patterns affecting SLA outcomes.

## Repository Structure

- `data/`
  - `incident_event_log_dataset.csv` — raw incident event log (multi-row per incident)
- `task/`
  - `incident_sla_compliance_analysis.ipynb` — Final SLA compliance analysis
- `requirements.txt`
- `README.md`

## Dataset Overview

Source: ServiceNow-like incident event log exported as CSV (141,712 rows, 37 columns).

- **Multi-row per incident** (status changes over time). Use latest closed record per incident for final SLA analysis.
- **Final Analysis Dataset**: 6,729 unique closed incidents after filtering and deduplication.
- Date/time columns are strings and converted to datetime.

Key columns used in SLA analysis:

- **Identifiers and timestamps**: `number`, `opened_at`, `closed_at`, `sys_created_at`, `sys_updated_at`
- **State and ownership**: `incident_state`, `assignment_group`, `assigned_to`
- **Classification**: `priority` (e.g., "1 - Critical", "2 - High", ...), `impact`, `urgency`, `category`, `subcategory`
- **SLA outcomes**: `made_sla` (boolean), `closed_time_hours` (calculated)
- **Process variables**: `knowledge`, `u_priority_confirmation`, `notify`, `reassignment_count`

Missing values:

- Several columns show high missingness (>95%) and are excluded from analysis
- Categorical variables use 'Unknown' imputation strategy

## Environment

```bash
pip install -r requirements.txt
```

Python libs: pandas, numpy, matplotlib, seaborn, scipy, scikit-learn.

## How to Run

Open the main SLA analysis notebook:

- `data/incident_event_log_dataset.csv`

## Research Question

**"What factors have the biggest impact on IT incident SLA performance?"**

## Methods (SLA Compliance Analysis)

1. **Data Processing**:

   - Filter for closed incidents with valid timestamps
   - Deduplicate by latest closed record per incident number
   - Remove negative closed times and convert date formats

2. **Missing Data Strategy**:

   - Drop columns with >95% missing values
   - Use 'Unknown' category for categorical variables
   - Retain outliers as they represent real business scenarios

3. **Feature Engineering**:

   - **Time-based features**: Hour, day, month, quarter patterns
   - **Process indicators**: Knowledge requirements, priority confirmation
   - **Complexity measures**: Severity scores, priority mismatches
   - **Interaction features**: Weekend-priority combinations
   - **Categorical encoding**: Frequency and label encoding

4. **Statistical Analysis**:
   - Bonferroni correction for multiple testing (p < 0.00147)
   - Effect size analysis with minimum threshold of 0.02
   - Correlation analysis for multicollinearity detection
   - Final selection: 25 statistically significant features

## Key Findings

### SLA Performance Overview

- **Overall compliance**: 86.5%
- **Total incidents analyzed**: 6,729 unique closed incidents
- **Performance drivers identified**: Multi-dimensional factors

### Top Performance Drivers

1. **Assignment Group** (effect: 0.294) - Who handles the incident
2. **Subcategory** (effect: 0.279) - What type of issue it is
3. **Location** (effect: 0.199) - Geographic factors
4. **Category** (effect: 0.183) - Service domain classification

### Critical Insights

**Priority Performance Paradox**:

- Critical incidents: 53.3% SLA compliance
- Low priority incidents: 95.5% SLA compliance
- Performance gap: 42.2 percentage points

**Organizational Variance**:

- Assignment group performance range: 44.1%–100.0%
- Performance gap: 55.9 percentage points
- High-volume groups maintain ~88.1% average compliance

**Temporal Patterns**:

- Weekend performance boost: 6.0% improvement
- Monthly volatility: 100.0 percentage point range (June 100% vs November 0%)
- Seasonal decline: Q2 (90.8%) vs Q4 (60.0%)

**Process Impact**:

- Knowledge consultation: +4.5% performance improvement
- Priority confirmation: -14.7% performance (complexity indicator)

### Statistical Rigor

- **25 of 34 features** survived Bonferroni correction
- **Feature composition**: 4 original + 21 engineered features
- **Multicollinearity detected**: 98% correlation between severity_score and priority_numeric

## Data Characteristics & Limitations

- **Extended closed times**: Average 125 days suggests change management rather than typical incidents
- **Generalization limits**: Results may not apply to standard helpdesk operations
- **Priority system concerns**: Performance inversion suggests unrealistic SLA targets for critical issues
- **Dataset nature**: May represent project/change-like work items rather than operational incidents

## Reproducibility Notes

- **Time parsing**: Uses `errors='coerce'`; invalid timestamps drop rows
- **Multi-row handling**: Keeps latest record per incident number for final KPIs
- **Outlier policy**: Retains extremes to reflect actual SLA breach scenarios
- **Missing data**: 'Unknown' imputation preserves categorical structure

## Business Recommendations

1. **Resource Allocation**: Focus on assignment group training and capacity optimization
2. **Service Classification**: Improve subcategory definitions and routing accuracy
3. **Priority System Review**: Address critical incident SLA target realism
4. **Seasonal Planning**: Prepare for Q4 performance challenges
5. **Geographic Strategy**: Leverage high-performing location best practices

## Next Steps (Modeling Suggestions)

- **SLA prediction**: Logistic regression/XGBoost with selected 25 features
