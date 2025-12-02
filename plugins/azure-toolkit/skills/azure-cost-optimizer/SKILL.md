---
name: azure-cost-optimizer
description: Analyzes Azure costs and provides optimization recommendations including reserved instances, rightsizing, and unused resources. Use when optimizing Azure spending or analyzing Azure costs.
---

# Azure Cost Optimizer

## Quick Start

Analyze Azure costs and implement optimization strategies to reduce spending while maintaining performance.

## Instructions

### Step 1: Analyze current costs

```bash
# View cost analysis
az cost-management query \
  --type Usage \
  --dataset-aggregation name="PreTaxCost" function="Sum" \
  --dataset-grouping name="ResourceGroup" type="Dimension"

# View costs by service
az cost-management query \
  --type Usage \
  --dataset-grouping name="ServiceName" type="Dimension"
```

### Step 2: Identify optimization opportunities

**Check for unused resources:**
- Unattached disks
- Idle virtual machines
- Unused public IPs
- Orphaned network interfaces

**Review resource utilization:**
- VM CPU and memory usage
- Database DTU utilization
- Storage account access patterns
- Network bandwidth usage

### Step 3: Implement cost-saving measures

**Reserved Instances:**
- 1-year or 3-year commitments
- Up to 72% savings vs pay-as-you-go
- Recommended for stable workloads

**Azure Hybrid Benefit:**
- Use existing Windows Server licenses
- Use existing SQL Server licenses
- Significant cost savings

**Rightsizing:**
- Downsize overprovisioned VMs
- Adjust database tiers
- Optimize storage tiers

**Spot VMs:**
- Up to 90% savings
- Suitable for interruptible workloads
- Batch processing, dev/test

## Advanced

For detailed information, see:
- [Pricing Models](reference/pricing-models.md) - Azure pricing tiers and models
- [Reserved Instances](reference/reserved-instances.md) - RI recommendations and strategies
- [Rightsizing](reference/rightsizing.md) - VM and resource sizing guidance
