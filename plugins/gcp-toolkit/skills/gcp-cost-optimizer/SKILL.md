---
name: gcp-cost-optimizer
description: Analyzes GCP costs and provides optimization recommendations including committed use discounts, rightsizing, and unused resources. Use when optimizing GCP spending or analyzing GCP costs.
---

# GCP Cost Optimizer

## Quick Start

Analyze GCP costs and implement optimization strategies to reduce spending.

## Instructions

### Step 1: Analyze current costs

```bash
# View billing data
gcloud billing accounts list

# Export billing data to BigQuery
gcloud billing accounts projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID

# Query costs
bq query --use_legacy_sql=false \
  'SELECT service.description, SUM(cost) as total_cost
   FROM `project.dataset.gcp_billing_export`
   WHERE DATE(_PARTITIONTIME) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
   GROUP BY service.description
   ORDER BY total_cost DESC'
```

### Step 2: Identify optimization opportunities

**Committed Use Discounts:**
- 1-year or 3-year commitments
- Up to 57% savings for Compute Engine
- Up to 70% savings for Cloud SQL

**Sustained Use Discounts:**
- Automatic discounts for running instances
- Up to 30% for instances running >25% of month

**Preemptible VMs:**
- Up to 80% savings
- Suitable for fault-tolerant workloads

**Rightsizing:**
- Use Recommender API for suggestions
- Downsize overprovisioned instances
- Adjust machine types

### Step 3: Implement cost-saving measures

```bash
# Get rightsizing recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.instance.MachineTypeRecommender

# Apply recommendation
gcloud recommender recommendations mark-claimed \
  RECOMMENDATION_ID \
  --project=PROJECT_ID \
  --location=us-central1 \
  --recommender=google.compute.instance.MachineTypeRecommender
```

## Best Practices

1. Enable billing export to BigQuery
2. Set up budget alerts
3. Use labels for cost allocation
4. Review Recommender suggestions monthly
5. Implement committed use discounts for stable workloads
6. Use preemptible VMs for batch processing
7. Clean up unused resources regularly
8. Optimize storage classes
