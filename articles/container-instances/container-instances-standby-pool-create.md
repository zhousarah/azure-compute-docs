---
title: Create a standby pool for Azure Container Instances (Preview)
description: Learn how to create a standby pool to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 09/30/2024
ms.reviewer: tomvcassidy
---


# Create a standby pool (Preview)
This article steps through creating a standby pool for Azure Container Instances with Flexible Orchestration.

> [!IMPORTANT]
> Standby pools for Azure Container Instances with Flexible Orchestration is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

## Prerequisites
Before utilizing standby pools, complete the feature registration and configure role based access controls. 

### Feature Registration 
Register the standby pool resource provider and the standby pool preview feature with your subscription using Azure Cloud Shell. Registration can take up to 30 minutes to successfully show as registered. You can rerun the below commands to determine when the feature is successfully registered. 

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNameSpace Microsoft.ContainerInstance

Register-AzResourceProvider -ProviderNamespace Microsoft.StandbyPool

Register-AzProviderFeature -FeatureName StandbyContainerGroupPoolPreview -ProviderNamespace Microsoft.StandbyPool
```

### Role-based Access Control Permissions
To allow standby pools to create virtual machines, you need to assign the appropriate RBAC permissions.
 
1) In the Azure Portal, navigate to your subscriptions.
2) Select the subscription you want to adjust RBAC permissions.
3) Select Access Control (IAM).
4) Select Add -> Add Custom Role.
5) Name your role ContainersContributor.
6) Move to the Permissions Tab.
7) Select Add Permissions.
8) Search for Microsoft.Container and select Microsoft Container Instance.
9) Select the permissions box to select all the permissions available.
10) Select Add.
11) Select Review + create.
12) Select Create.
13) Select Add -> Add role assignment
14) Under the roles tab, search for the custom role you created earlier called ContainersContributor and select it
15) Move to the Members tab
16) Select + Select Members
17) Search for Standby Pool Resource Provider.
18) Select the Standby Pool Resource Provider and select Review + Assign.

If you do not have Contributor/ Owner/ Administrator roles to the subscription you are using ACI Pools for, you will also need to setup StandbyPools RBAC roles (Standby Pool create, reads, etc.) and assign to yourself.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).

## Limitations
Standby Pools for Azure Container instances is not available in the Azure portal. 

## Create a standby pool

### [CLI](#tab/cli)
Create a standby pool and associate it with an existing scale set using [az standby-container-group-pool create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az standby-container-group-pool create \
   --resource-group myResourceGroup 
   --location eastus \
   --name myStandbyPool \
   --max-ready-capacity 20 \
   --refill-policy always \
   --container-profile-id "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile"
```
### [PowerShell](#tab/powershell)
Create a standby pool and associate it with an existing scale set using [New-AzStandbyVMPool](/powershell/module/az.standbypool/new-azstandbyvmpool).

```azurepowershell-interactive
New-AzStandbyVMPool `
   -ResourceGroup myResourceGroup `
   -Location eastus `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -VMState "Deallocated" `
   -VMSSId "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
```

### [ARM template](#tab/template)
Create a standby pool and associate it with an existing scale set. Create a template and deploy it using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
           "type": "string",
           "defaultValue": "east us"    
        },
        "name": {
           "type": "string",
           "defaultValue": "myStandbyPool"
        },
        "maxReadyCapacity" : {
           "type": "int",
           "defaultValue": 10
        },
        "virtualMachineState" : {
           "type": "string",
           "defaultValue": "Deallocated"
        },
        "attachedVirtualMachineScaleSetId" : {
           "type": "string",
           "defaultValue": "/subscriptions/{subscriptionID}/resourceGroups/StandbyPools/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
        }
    },
    "resources": [ 
        {
            "type": "Microsoft.StandbyPool/standbyVirtualMachinePools",
            "apiVersion": "2023-12-01-preview",
            "name": "[parameters('name')]",
            "location": "[parameters('location')]",
            "properties": {
               "elasticityProfile": {
                   "maxReadyCapacity": "[parameters('maxReadyCapacity')]" 
               },
               "virtualMachineState": "[parameters('virtualMachineState')]",
               "attachedVirtualMachineScaleSetId": "[parameters('attachedVirtualMachineScaleSetId')]"
            }
        }
    ]
   }

```


### [Bicep](#tab/bicep)
Create a standby pool and associate it with an existing scale set. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param location string = resourceGroup().location
param standbyPoolName string = 'myStandbyPool'
param maxReadyCapacity int = 20
@allowed([
  'Running'
  'Deallocated'
])
param vmState string = 'Deallocated'
param virtualMachineScaleSetId string = '/subscriptions/{subscriptionID}/resourceGroups/StandbyPools/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet}'

resource standbyPool 'Microsoft.standbypool/standbyvirtualmachinepools@2023-12-01-preview' = {
  name: standbyPoolName
  location: location
  properties: {
     elasticityProfile: {
      maxReadyCapacity: maxReadyCapacity
    }
    virtualMachineState: vmState
    attachedVirtualMachineScaleSetId: virtualMachineScaleSetId
  }
}
```

### [REST](#tab/rest)
Create a standby pool and associate it with an existing scale set using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2023-12-01-preview
{
"type": "Microsoft.StandbyPool/standbyVirtualMachinePools",
"name": "myStandbyPool",
"location": "east us",
"properties": {
     "elasticityProfile": {
         "maxReadyCapacity": 20
     },
      "virtualMachineState":"Deallocated",
      "attachedVirtualMachineScaleSetId": "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
      }
}
```

---

## Next steps

Learn how to [update and delete a standby pool](standby-pools-update-delete.md).
