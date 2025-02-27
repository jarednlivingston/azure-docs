---
title: Resource Manager template samples for data collection rules
description: Sample Azure Resource Manager templates to create associations between data collection rules and virtual machines in Azure Monitor.
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 02/07/2022

---

# Resource Manager template samples for data collection rules in Azure Monitor
This article includes sample [Azure Resource Manager templates](../../azure-resource-manager/templates/syntax.md) to create an association between a [data collection rule](../essentials/data-collection-rule-overview.md) and the [Azure Monitor agent](./azure-monitor-agent-overview.md). Each sample includes a template file and a parameters file with sample values to provide to the template.

[!INCLUDE [azure-monitor-samples](../../../includes/azure-monitor-resource-manager-samples.md)]

## Permissions required

| Built-in Role | Scope(s) | Reason |  
|:---|:---|:---|  
| [Monitoring Contributor](../../role-based-access-control/built-in-roles.md#monitoring-contributor) | <ul><li>Subscription and/or</li><li>Resource group and/or </li><li>An existing data collection rule</li></ul> | To create or edit data collection rules |
| <ul><li>[Virtual Machine Contributor](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)</li><li>[Azure Connected Machine Resource Administrator](../../role-based-access-control/built-in-roles.md#azure-connected-machine-resource-administrator)</li></ul> | <ul><li>Virtual machines, virtual machine scale sets</li><li>Arc-enabled servers</li></ul> | To deploy associations (i.e. to assign rules to the machine) |
| Any role that includes the action *Microsoft.Resources/deployments/** | <ul><li>Subscription and/or</li><li>Resource group and/or </li><li>An existing data collection rule</li></ul> | To deploy ARM templates |


## Create rule (sample)

View [template format](/azure/templates/microsoft.insights/datacollectionrules)

## Create association with Azure VM

The following sample creates an association between an Azure virtual machine and a data collection rule.

### Template file

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "metadata": {
                "description": "Name of the virtual machine."
            }
        },
        "associationName": {
            "type": "string",
            "metadata": {
                "description": "Name of the association."
            }
        },
        "dataCollectionRuleId": {
            "type": "string",
            "metadata": {
                "description": "Resource ID of the data collection rule."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Compute/virtualMachines/providers/dataCollectionRuleAssociations",
            "name": "[concat(parameters('vmName'),'/microsoft.insights/', parameters('associationName'))]",
            "apiVersion": "2019-11-01-preview",
            "properties": {
                "description": "Association of data collection rule. Deleting this association will break the data collection for this virtual machine.",
                "dataCollectionRuleId": "[parameters('dataCollectionRuleId')]"
            }
        }
    ]
}
```

### Parameter file

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
        "value": "my-windows-vm"
      },
      "associationName": {
        "value": "my-windows-vm-my-dcr"
      },
      "dataCollectionRuleId": {
        "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.insights/datacollectionrules/my-dcr"
      }
   }
}
```

## Create association with Azure Arc

The following sample creates an association between an Azure Arc-enabled server and a data collection rule.

### Template file

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "metadata": {
                "description": "Name of the virtual machine."
            }
        },
        "associationName": {
            "type": "string",
            "metadata": {
                "description": "Name of the association."
            }
        },
        "dataCollectionRuleId": {
            "type": "string",
            "metadata": {
                "description": "Resource ID of the data collection rule."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.HybridCompute/machines/providers/dataCollectionRuleAssociations",
            "name": "[concat(parameters('vmName'),'/microsoft.insights/', parameters('associationName'))]",
            "apiVersion": "2019-11-01-preview",
            "properties": {
                "description": "Association of data collection rule. Deleting this association will break the data collection for this Arc server.",
                "dataCollectionRuleId": "[parameters('dataCollectionRuleId')]"
            }
        }
    ]
}
```

### Parameter file

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "vmName": {
        "value": "my-windows-vm"
      },
      "associationName": {
        "value": "my-windows-vm-my-dcr"
      },
      "dataCollectionRuleId": {
        "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.insights/datacollectionrules/my-dcr"
      }
   }
}
```


## Next steps

* [Learn more about Data Collection rules and associations](./data-collection-rule-azure-monitor-agent.md)
* [Learn more about Azure Monitor agent](./azure-monitor-agent-overview.md)
* [Get other sample templates for Azure Monitor](../resource-manager-samples.md).
