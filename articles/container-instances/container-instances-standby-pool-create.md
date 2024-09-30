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


# Create a standby pool for Azure Container Instances (Preview)
This article steps through creating a standby pool for Azure Container Instances. 

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

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
To allow standby pools to create container groups, you need to assign the appropriate permissions to the standby pool resource provider. 
 
1) In the Azure portal, navigate to your subscriptions.
2) Select the subscription you want to adjust permissions.
3) Select **Access Control (IAM)**.
4) Select **Add** and **Add Custom Role**.
5) Name your role **ContainersContributor**.
6) Move to the Permissions Tab.
7) Select **Add Permissions**.
8) Search for `Microsoft.Container` and select **Microsoft Container Instance**.
9) Select the permissions box to select all the permissions available.
10) Select **Add**.
11) Select **Review + create**.
12) Select **Create**.
13) Select **Add** and **Add role assignment**.
14) Under the roles tab, search for the custom role you created called **ContainersContributor** and select it.
15) Move to the Members tab.
16) Select **+ Select Members**.
17) Search for Standby Pool Resource Provider.
18) Select the **Standby Pool Resource Provider** and select **Review + Assign**.

For more information on assigning roles, see [assign Azure roles using the Azure portal](/azure/role-based-access-control/quickstart-assign-role-user-portal).

## Limitations
Standby Pools for Azure Container Instances is not available in the Azure portal. 

## Create a standby pool

### [CLI](#tab/cli)
Create a standby pool and associate it with a container group profile using [az standby-container-group-pool create](/cli/azure/standby-container-group-pool).

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
Create a standby pool and associate it with a container group profile using [New-AzStandbyContainerGroupPool](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
New-AzStandbyContainerGroupPool `
   -ResourceGroup myResourceGroup `
   -Location eastus `
   -Name myStandbyPool `
   -MaxReadyCapacity 20 `
   -RefillPolicy always `
   -ContainerProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile"
```

### [ARM template](#tab/template)
Create a standby pool and associate it with a container group profile. Create a template and deploy it using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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
        "refillPolicy" : {
           "type": "string",
           "defaultValue": "always"
        },
        "containerGroupProfile" : {
           "type": "string",
           "defaultValue": "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{ContainerProfile}"
        }
    },
    "resources": [ 
        {
            "type": "Microsoft.StandbyPool/standbyContainerGroupPools",
            "apiVersion": "2024-03-01",
            "name": "[parameters('name')]",
            "location": "[parameters('location')]",
            "properties": {
               "elasticityProfile": {
                   "maxReadyCapacity": "[parameters('maxReadyCapacity')]",
                   "refillPolicy": "[parameters('refillPolicy')]"
               },
               "containerGroupProfile": "[parameters('containerGroupProfile')]"
            }
        }
    ]
   }

```


### [Bicep](#tab/bicep)
Create a standby pool and associate it with a container group profile. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param location string = resourceGroup().location
param standbyPoolName string = 'myStandbyPool'
param maxReadyCapacity int = 20
param containerGroupProfile string = '/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{ContainerProfile}'

resource standbyPool 'Microsoft.standbypool/standbyContainerGroupsPools@2024-03-01' = {
  name: standbyPoolName
  location: location
  properties: {
     elasticityProfile: {
      maxReadyCapacity: maxReadyCapacity
      refillPolicy: always
    }
    containerGroupProfile: containerGroupProfile
  }
}
```

### [REST](#tab/rest)
Create a standby pool and associate it with a container group profile using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{StandbyPoolName}?api-version=2023-12-01-preview 
 
Request Body
{
    "properties": {
        "elasticityProfile": {
            "maxReadyCapacity": 20,
            "refillPolicy": "always"
        },
        "containerGroupProperties": {
            "containerGroupProfile": {
                "id": "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{ContainerProfile}",
                "revision": 1
            }
        }
    },
    "location": "west central us"
}
```

---

## Next steps

Learn more about [standby pools for Azure Container Instances](container-instances-standby-pool-overview.md)
