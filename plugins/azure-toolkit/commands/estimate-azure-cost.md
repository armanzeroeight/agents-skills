---
description: Estimate Azure resource costs and provide optimization recommendations
allowed-tools: Bash(az:*), Read
argument-hint: [resource-group]
---

# Estimate Azure Cost

## Context

- Azure CLI version: !`az --version 2>&1 | head -1 || echo "Azure CLI not installed"`
- Current subscription: !`az account show --query name -o tsv 2>&1 || echo "Not logged in"`
- Resource groups: !`az group list --query "[].name" -o tsv 2>&1 | head -10 || echo "No resource groups"`

## Your Task

Analyze Azure costs and provide optimization recommendations for the specified resource group or subscription.

### Arguments

- `$1`: Resource group name (optional, analyzes entire subscription if not provided)

### Steps

1. **Analyze current costs**
   - Query cost management data
   - Group by resource type and service
   - Identify top cost drivers

2. **Identify optimization opportunities**
   - Find unused resources
   - Check for oversized VMs
   - Review storage tiers
   - Identify RI candidates

3. **Provide recommendations**
   - Reserved instance opportunities
   - Rightsizing suggestions
   - Azure Hybrid Benefit eligibility
   - Spot VM candidates
   - Storage optimization

4. **Calculate potential savings**
   - Estimate savings per recommendation
   - Prioritize by impact
   - Provide implementation steps

### Output Format

```markdown
## Azure Cost Analysis

**Scope:** [subscription/resource-group]
**Period:** Last 30 days
**Total Cost:** $[amount]

### Top Cost Drivers
1. [Service]: $[amount]
2. [Service]: $[amount]

### Optimization Opportunities

**High Priority (Est. Savings: $[amount]/month)**
- [Recommendation with details]

**Medium Priority (Est. Savings: $[amount]/month)**
- [Recommendation with details]

### Implementation Steps
1. [Step with command]
2. [Step with command]
```
