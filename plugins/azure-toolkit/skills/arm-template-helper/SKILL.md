---
name: arm-template-helper
description: Creates and validates Azure Resource Manager (ARM) templates for infrastructure deployment. Use when creating ARM templates, deploying Azure infrastructure as code, or validating Azure templates.
---

# ARM Template Helper

## Quick Start

Create and validate Azure Resource Manager templates for infrastructure as code deployments.

## Instructions

### Step 1: Create ARM template structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {},
  "resources": [],
  "outputs": {}
}
```

### Step 2: Add resources

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2021-04-01",
    "name": "[parameters('storageAccountName')]",
    "location": "[parameters('location')]",
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "StorageV2"
  }
]
```

### Step 3: Validate template

```bash
# Validate ARM template
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file template.json \
  --parameters parameters.json

# Test deployment (what-if)
az deployment group what-if \
  --resource-group myResourceGroup \
  --template-file template.json
```

### Step 4: Deploy template

```bash
# Deploy ARM template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file template.json \
  --parameters parameters.json
```

## Best Practices

1. Use parameters for configurable values
2. Use variables for computed values
3. Add outputs for important resource properties
4. Use dependsOn for resource dependencies
5. Implement proper naming conventions
6. Add tags for resource organization
7. Use linked templates for modularity
8. Validate before deploying
