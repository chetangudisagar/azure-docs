---
title: Azure resource policies | Microsoft Docs
description: Describes how to use Azure Resource Manager policies to ensure consistent resource properties are set during deployment. Policies can be applied at the subscription or resource groups.
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn

ms.assetid: abde0f73-c0fe-4e6d-a1ee-32a6fce52a2d
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/09/2017
ms.author: tomfitz

---
# Resource policy overview
Resource policies enable you to establish conventions for resources in your organization. By defining conventions, you can control costs and more easily manage your resources. For example, you can specify that only certain types of virtual machines are allowed. Or, you can require that all resources have a particular tag. Policies are inherited by all child resources. So, if a policy is applied to a resource group, it is applicable to all the resources in that resource group.

There are two concepts to understand about policies:

* policy definition - you describe when the policy is enforced and what action to take
* policy assignment - you apply the policy definition to a scope (subscription or resource group)

This topic focuses on policy definition. For information about policy assignment, see [Use Azure portal to assign and manage resource policies](resource-manager-policy-portal.md) or [Assign and manage policies through script](resource-manager-policy-create-assign.md).

Policies are evaluated when creating and updating resources (PUT and PATCH operations).

## How is it different from RBAC?
There are a few key differences between policy and role-based access control (RBAC). RBAC focuses on **user** actions at different scopes. For example, you are added to the contributor role for a resource group at the desired scope, so you can make changes to that resource group. Policy focuses on **resource** properties during deployment. For example, through policies, you can control the types of resources that can be provisioned. Or, you can restrict the locations in which the resources can be provisioned. Unlike RBAC, policy is a default allow and explicit deny system. 

To use policies, you must be authenticated through RBAC. Specifically, your account needs the:

* `Microsoft.Authorization/policydefinitions/write` permission to define a policy
* `Microsoft.Authorization/policyassignments/write` permission to assign a policy 

These permissions are not included in the **Contributor** role.

## Built-in policies

Azure provides some built-in policy definitions that may reduce the number of policies you have to define. Before proceeding with policy definitions, you should consider whether a built-in policy already provides the definition you need. The built-in policy definitions are:

* Allowed locations
* Allowed resource types
* Allowed storage account SKUs
* Allowed virtual machine SKUs
* Apply tag and default value
* Enforce tag and value
* Not allowed resource types
* Require SQL Server version 12.0
* Require storage account encryption

You can assign any of these policies through the [portal](resource-manager-policy-portal.md), [PowerShell](resource-manager-policy-create-assign.md#powershell), or [Azure CLI](resource-manager-policy-create-assign.md#azure-cli).

## Policy definition structure
You use JSON to create a policy definition. The policy definition contains elements for:

* mode
* parameters
* display name
* description
* policy rule
  * logical evaluation
  * effect

The following example shows a policy that limits where resources are deployed:

```json
{
  "properties": {
    "mode": "all",
    "parameters": {
      "allowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of locations that can be specified when deploying resources",
          "strongType": "location",
          "displayName": "Allowed locations"
        }
      }
    },
    "displayName": "Allowed locations",
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

## Mode

We recommend that you set `mode` to `all`. When you set it to **all**, resource groups and all resource types are evaluated for the policy. The portal uses **all** for all policies. If you use PowerShell or Azure CLI, you need to specify the `mode` parameter and set it to **all**.
 
Previously, policy was evaluated on only resource types that supported tags and location. The `indexed` mode continues this behavior. If you use the **all** mode, policies are also evaluated on resource types that do not support tags and location. [Virtual network subnet](https://github.com/Azure/azure-policy-samples/tree/master/samples/Network/enforce-nsg-on-subnet) is an example of a newly added type. In addition, resource groups are evaluated when mode is set to **all**. For example, you can [enforce tags on a resource group](https://github.com/Azure/azure-policy-samples/tree/master/samples/ResourceGroup/enforce-resourceGroup-tags). 

## Parameters
Using parameters helps simplify your policy management by reducing the number of policy definitions. You define a policy for a resource property (such as limiting the locations where resources can be deployed), and include parameters in the definition. Then, you reuse that policy definition for different scenarios by passing in different values (such as specifying one set of locations for a subscription) when assigning the policy.

You declare parameters when you create policy definitions.

```json
"parameters": {
  "allowedLocations": {
    "type": "array",
    "metadata": {
      "description": "The list of allowed locations for resources.",
      "displayName": "Allowed locations"
    }
  }
}
```

The type of a parameter can be either string or array. The metadata property is used for tools like Azure portal to display user-friendly information. 

In the policy rule, you reference parameters with the following syntax: 

```json
{ 
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

## Display name and description

You use the **displayName** and **description** to identify the policy definition, and provide context for when it is used.

## Policy rule

The policy rule consists of **If** and **Then** blocks. In the **If** block, you define one or more conditions that specify when the policy is enforced. You can apply logical operators to these conditions to precisely define the scenario for a policy. In the **Then** block, you define the effect that happens when the **If** conditions are fulfilled.

```json
{
  "if": {
    <condition> | <logical operator>
  },
  "then": {
    "effect": "deny | audit | append"
  }
}
```

### Logical operators
The supported logical operators are:

* `"not": {condition  or operator}`
* `"allOf": [{condition or operator},{condition or operator}]`
* `"anyOf": [{condition or operator},{condition or operator}]`

The **not** syntax inverts the result of the condition. The **allOf** syntax (similar to the logical **And** operation) requires all conditions to be true. The **anyOf** syntax (similar to the logical **Or** operation) requires one or more conditions to be true.

You can nest logical operators. The following example shows a **not** operation that is nested within an **allOf** operation. 

```json
"if": {
  "allOf": [
    {
      "not": {
        "field": "tags",
        "containsKey": "application"
      }
    },
    {
      "field": "type",
      "equals": "Microsoft.Storage/storageAccounts"
    }
  ]
},
```

### Conditions
The condition evaluates whether a **field** meets certain criteria. The supported conditions are:

* `"equals": "value"`
* `"like": "value"`
* `"match": "value"`
* `"contains": "value"`
* `"in": ["value1","value2"]`
* `"containsKey": "keyName"`
* `"exists": "bool"`

When using the **like** condition, you can provide a wildcard (*) in the value.

When using the **match** condition, provide `#` to represent a digit, `?` for a letter, and any other character to represent that actual character. For examples, see [Apply resource policies for names and text](resource-manager-policy-naming-convention.md).

### Fields
Conditions are formed by using fields. A field represents properties in the resource request payload that is used to describe the state of the resource.  

The following fields are supported:

* `name`
* `kind`
* `type`
* `location`
* `tags`
* `tags.*` 
* property aliases - for a list, see [Aliases](#aliases).

### Effect
Policy supports three types of effect - `deny`, `audit`, `append`, `AuditIfNotExists`, and `DeployIfNotExists`. 

* **Deny** generates an event in the audit log and fails the request
* **Audit** generates a warning event in audit log but does not fail the request
* **Append** adds the defined set of fields to the request 
* **AuditIfNotExists** - enable auditing if a resource does not exist
* **DeployIfNotExists** - deploy a resource if it does not already exist. Currently, this effect is only supported through built-in policies.

For **append**, you must provide the following details:

```json
"effect": "append",
"details": [
  {
    "field": "field name",
    "value": "value of the field"
  }
]
```

The value can be either a string or a JSON format object. 

With **AuditIfNotExists** and **DeployIfNotExists**, you can evaluate the existence of a child resource, and apply a rule when that resource does not exist. For example, you can require that a network watcher is deployed for all virtual networks.

For an example of auditing when a virtual machine extension is not deployed, see [Audit VM extensions](https://github.com/Azure/azure-policy-samples/blob/master/samples/Compute/audit-vm-extension/azurepolicy.json).

## Aliases

You use property aliases to access specific properties for a resource type. Aliases enable you to restrict what values or conditions are permitted for a property on a resource. Each alias maps to paths in different API versions for a given resource type. During policy evaluation, the policy engine gets the property path for that API version.

**Microsoft.Cache/Redis**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Cache/Redis/enableNonSslPort | Set whether the non-ssl Redis server port (6379) is enabled. |
| Microsoft.Cache/Redis/shardCount | Set the number of shards to be created on a Premium Cluster Cache.  |
| Microsoft.Cache/Redis/sku.capacity | Set the size of the Redis cache to deploy.  |
| Microsoft.Cache/Redis/sku.family | Set the SKU family to use. |
| Microsoft.Cache/Redis/sku.name | Set the type of Redis Cache to deploy. |

**Microsoft.Cdn/profiles**

| Alias | Description |
| ----- | ----------- |
| Microsoft.CDN/profiles/sku.name | Set the name of the pricing tier. |

**Microsoft.Compute/disks**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |


**Microsoft.Compute/virtualMachines**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageId | Set the identifier of the image used to create the virtual machine. |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/licenseType | Set that the image or disk is licensed on-premises. This value is only used for images that contain the Windows Server operating system.  |
| Microsoft.Compute/virtualMachines/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/virtualMachines/osDisk.Uri | Set the vhd URI. |
| Microsoft.Compute/virtualMachines/sku.name | Set the size of the virtual machine. |

**Microsoft.Compute/virtualMachines/extensions**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/virtualMachines/extensions/publisher | Set the name of the extension’s publisher. |
| Microsoft.Compute/virtualMachines/extensions/type | Set the type of extension. |
| Microsoft.Compute/virtualMachines/extensions/typeHandlerVersion | Set the version of the extension. |

**Microsoft.Compute/virtualMachineScaleSets**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Compute/imageId | Set the identifier of the image used to create the virtual machine. |
| Microsoft.Compute/imageOffer | Set the offer of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imagePublisher | Set the publisher of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageSku | Set the SKU of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/imageVersion | Set the version of the platform image or marketplace image used to create the virtual machine. |
| Microsoft.Compute/licenseType | Set that the image or disk is licensed on-premises. This value is only used for images that contain the Windows Server operating system. |
| Microsoft.Compute/VirtualMachineScaleSets/computerNamePrefix | Set the computer name prefix for all  the virtual machines in the scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/osdisk.imageUrl | Set the blob URI for user image. |
| Microsoft.Compute/VirtualMachineScaleSets/osdisk.vhdContainers | Set the container URLs that are used to store operating system disks for the scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/sku.name | Set the size of virtual machines in a scale set. |
| Microsoft.Compute/VirtualMachineScaleSets/sku.tier | Set the tier of virtual machines in a scale set. |
  
**Microsoft.Network/applicationGateways**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Network/applicationGateways/sku.name | Set the size of the gateway. |

**Microsoft.Network/virtualNetworkGateways**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Network/virtualNetworkGateways/gatewayType | Set the type of this virtual network gateway. |
| Microsoft.Network/virtualNetworkGateways/sku.name | Set the gateway SKU name. |

**Microsoft.Sql/servers**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Sql/servers/version | Set the version of the server. |

**Microsoft.Sql/databases**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Sql/servers/databases/edition | Set the edition of the database. |
| Microsoft.Sql/servers/databases/elasticPoolName | Set the name of the elastic pool the database is in. |
| Microsoft.Sql/servers/databases/requestedServiceObjectiveId | Set the configured service level objective ID of the database. |
| Microsoft.Sql/servers/databases/requestedServiceObjectiveName | Set the name of the configured service level objective of the database.  |

**Microsoft.Sql/elasticpools**

| Alias | Description |
| ----- | ----------- |
| servers/elasticpools | Microsoft.Sql/servers/elasticPools/dtu | Set the total shared DTU for the database elastic pool. |
| servers/elasticpools | Microsoft.Sql/servers/elasticPools/edition | Set the edition of the elastic pool. |

**Microsoft.Storage/storageAccounts**

| Alias | Description |
| ----- | ----------- |
| Microsoft.Storage/storageAccounts/accessTier | Set the access tier used for billing. |
| Microsoft.Storage/storageAccounts/accountType | Set the SKU name. |
| Microsoft.Storage/storageAccounts/enableBlobEncryption | Set whether the service encrypts the data as it is stored in the blob storage service. |
| Microsoft.Storage/storageAccounts/enableFileEncryption | Set whether the service encrypts the data as it is stored in the file storage service. |
| Microsoft.Storage/storageAccounts/sku.name | Set the SKU name. |
| Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly | Set to allow only https traffic to storage service. |

## Policy sets

Policy sets enable you to group several related policy definitions. The policy set simplifies assignment and management because you work with group as a single item. For example, you can group all related tagging policies in a single policy set. Rather than assigning each policy individually, you apply the policy set.
 
The following example illustrates how to create a policy set for handling two tags (costCenter and productName). It uses two built-in policies for applying the default tag value, and enforcing the tag value. The policy set declares two parameters, costCenterValue and productNameValue for reusability. It references the two built-in policy definitions multiple times with different parameters. For each parameter, you can either provide a fixed value, as shown for tagName, or a parameter from the policy set, as shown for tagValue.

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                }
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                }
            }
        },
        "policyDefinitions": [
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    },
    "id": "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policySetDefinitions/billingTagsPolicy",
    "type": "Microsoft.Authorization/policySetDefinitions",
    "name": "billingTagsPolicy"
}
```

You add a policy set with the **New-AzureRMPolicySetDefinition** PowerShell command.

For REST operations, use the **2017-06-01-preview** API version, as shown in the following example:

```
PUT /subscriptions/<subId>/providers/Microsoft.Authorization/policySetDefinitions/billingTagsPolicySet?api-version=2017-06-01-preview
```

## Next steps
* After defining a policy rule, assign it to a scope. To assign policies through the portal, see [Use Azure portal to assign and manage resource policies](resource-manager-policy-portal.md). To assign policies through REST API, PowerShell or Azure CLI, see [Assign and manage policies through script](resource-manager-policy-create-assign.md).
* For example policies, see [Azure resource policy GitHub repository](https://github.com/Azure/azure-policy-samples).
* For guidance on how enterprises can use Resource Manager to effectively manage subscriptions, see [Azure enterprise scaffold - prescriptive subscription governance](resource-manager-subscription-governance.md).
* The policy schema is published at [http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json](http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json). 

