---
description: Estimate GCP resource costs and provide optimization recommendations
allowed-tools: Bash(gcloud:*), Bash(bq:*), Read
argument-hint: [project-id]
---

# Estimate GCP Cost

## Context

- gcloud version: !`gcloud --version 2>&1 | head -1 || echo "gcloud not installed"`
- Current project: !`gcloud config get-value project 2>&1 || echo "No project set"`
- Billing account: !`gcloud billing accounts list --format="value(name)" 2>&1 | head -1 || echo "No billing account"`

## Your Task

Analyze GCP costs and provide optimization recommendations for the specified project.

### Arguments

- `$1`: Project ID (optional, uses current project if not provided)

### Steps

1. **Analyze current costs**
   - Query billing data from BigQuery export
   - Group by service and resource type
   - Identify top cost drivers

2. **Get recommendations**
   - Query Recommender API for rightsizing
   - Identify committed use discount opportunities
   - Find unused resources

3. **Provide optimization suggestions**
   - Committed use discounts
   - Preemptible VM candidates
   - Rightsizing recommendations
   - Storage class optimization
   - Unused resource cleanup

4. **Calculate potential savings**
   - Estimate savings per recommendation
   - Prioritize by impact
   - Provide implementation commands

### Output Format

```markdown
## GCP Cost Analysis

**Project:** [project-id]
**Period:** Last 30 days
**Total Cost:** $[amount]

### Top Cost Drivers
1. [Service]: $[amount]
2. [Service]: $[amount]

### Optimization Opportunities

**High Priority (Est. Savings: $[amount]/month)**
- [Recommendation with gcloud command]

**Medium Priority (Est. Savings: $[amount]/month)**
- [Recommendation with gcloud command]

### Implementation Steps
1. [Step with command]
2. [Step with command]
```
