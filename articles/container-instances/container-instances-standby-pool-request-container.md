---
title: Request a container from a Standby Pool for Azure Container Instances(Preview)
description: Learn how to scale out from a standby pool with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 09/30/2024
ms.reviewer: tomvcassidy
---


# Request a container from a standby pool for Azure Container Instances (Preview)
This article steps through requesting a container group from a standby pool for Azure Container Instances. After requesting a container, the standby pool will automatically begin to refill to maintain the `maxReadyCapacity` parameter. 

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

## Prerequisites

Before utilizing standby pools, complete the feature registration and configure role based access controls mentioned in [Create a standby pool](container-instances-standby-pool-create.md). 


## Limitations
Standby Pools for Azure Container Instances is not available in the Azure portal. 

## Create a container group profile
The container group profile is what tells the standby pool how to configure the containers in the pool. Each standby pool is associated with a single container group profile. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.

### [CLI](#tab/cli)
Request a container group from a standby pool using [az container update](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az container update \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --location "West Central US" \
  --standby-pool "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool" \
  --container-group-profile "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" \
  --revision 1 \
  --container-name myContainerProfile \
  --config-map $newKey=$newValue


```
### [PowerShell](#tab/powershell)
Request a container group from a standby pool using [Update-AzContainer](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
Update-AzContainer `
    -ResourceGroupName myResourceGroup `
    -Location "West Central US" `
    -ContainerGroupName myContainerGroup `
    -StandbyPoolId "/subscriptions/{subscriptionId]/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool" `
    -ContainerGroupProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" `
    -ContainerGroupProfileRevision 1 `
    -ContainerName myContainerProfile `
    -ConfigMap @{ $newKey = $newValue }

```

### [ARM template](#tab/template)
Request a container group from a standby pool using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "subscriptionId": {
      "type": "string",
      "metadata": {
        "description": "The subscription ID."
      }
    },
    "resourceGroup": {
      "type": "string",
      "metadata": {
        "description": "The name of the resource group."
      }
    },
    "location": {
      "type": "string",
      "metadata": {
        "description": "Location for the resource."
      },
      "defaultValue": "West Central US"
    },
    "containerGroupName": {
      "type": "string",
      "metadata": {
        "description": "The name of the container group."
      }
    },
    "standbyPoolName": {
      "type": "string",
      "metadata": {
        "description": "The name of the standby pool."
      }
    },
    "containerGroupProfileName": {
      "type": "string",
      "metadata": {
        "description": "The name of the container group profile."
      }
    },
    "myContainerProfile": {
      "type": "string",
      "metadata": {
        "description": "The name of the container profile."
      }
    },
    "newKey": {
      "type": "string",
      "metadata": {
        "description": "The new key for the config map."
      }
    },
    "newValue": {
      "type": "string",
      "metadata": {
        "description": "The new value for the config map."
      }
    },
    "revisionNumber": {
      "type": "int",
      "metadata": {
        "description": "The revision number for the container group profile."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2023-05-01",
      "name": "[parameters('containerGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
        "standByPoolProfile": {
          "id": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.StandbyPool/standbyContainerGroupPools/', parameters('standbyPoolName'))]"
        },
        "containerGroupProfile": {
          "id": "[concat('/subscriptions/', parameters('subscriptionId'), '/resourceGroups/', parameters('resourceGroup'), '/providers/Microsoft.ContainerInstance/containerGroupProfiles/', parameters('containerGroupProfileName'))]",
          "revision": "[parameters('revisionNumber')]"
        },
        "containers": [
          {
            "name": "[parameters('myContainerProfile')]",
            "properties": {
              "configMap": {
                "keyValuePairs": {
                  "[parameters('newKey')]": "[parameters('newValue')]"
                }
              }
            }
          }
        ]
      }
    }
  ],
  "outputs": {
    "containerGroupId": {
      "type": "string",
      "value": "[resourceId('Microsoft.ContainerInstance/containerGroups', parameters('containerGroupName'))]"
    }
  }
}

```


### [Bicep](#tab/bicep)
Request a container group from a standby pool using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param subscriptionId string = {subscriptionId}
param resourceGroup string = "myResourceGroup"
param location string = "West Central US"
param containerGroupName string = "myContainerGroup"
param standbyPoolName string = "myStandbyPool" 
param containerGroupProfileName string = "myContainerGroupProfile"
param myContainerProfile string = "myContainerGroupProfile"
param newKey string = {newKey}
param newValue string = {newValue}
param revisionNumber int = 1

resource containerGroup 'Microsoft.ContainerInstance/containerGroups@2023-05-01' = {
  name: containerGroupName
  location: location
  properties: {
    standByPoolProfile: {
      id: '/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/${standbyPoolName}'
    }
    containerGroupProfile: {
      id: '/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.ContainerInstance/containerGroupProfiles/${containerGroupProfileName}'
      revision: revisionNumber
    }
    containers: [
      {
        name: myContainerProfile
        properties: {
          configMap: {
            keyValuePairs: {
              '${newKey}': '${newValue}'
            }
          }
        }
      }
    ]
  }
}

```

### [REST](#tab/rest)
Request a container group from a standby pool using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2023-05-01 

Request Body
{
   "location": "{location}",
   "properties": {
       "standByPoolProfile":{
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool"
           },
           "containerGroupProfile": {
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile",
               "revision": {revisionNumber}
           },
           "containers": [
               {
                   "name": "{mycontainerprofile}",
                   "properties": {
                       "configMap": {
                           "keyValuePairs": {
                               "{newKey}": "{newValue}"
                           }
                       }
                   }
               }
           ]
   }
}
```

---



## Next steps

Learn more about [standby pools for Azure Container Instances](container-instances-standby-pool-overview.md)
