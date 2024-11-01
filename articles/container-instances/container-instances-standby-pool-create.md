---
title: Create a standby pool for Azure Container Instances (Preview)
description: Learn how to create a standby pool to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 11/1/2024
ms.reviewer: tomvcassidy
---


# Create a standby pool for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 


This article steps through creating a container group profile and using that profile to configure a standby pool for Azure Container Instances. 

## Prerequisites

Before utilizing standby pools, complete the feature registration and configure role based access controls listed in the [Standby Pools for Azure Container Instances](container-instances-standby-pool-overview.md#prerequisites) overview page. 

## Create a container group profile
The container group profile tells the standby pool how to configure the containers in the pool. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.

> [!NOTE]
> To use [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` when creating your container group profile.

### [CLI](#tab/cli)
Create a container group profile using [az container-profile create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az container-group-profile create \
  --resource-group myResourceGroup \
  --name myContainerGroupProfile \
  --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
  --cpu 1 \
  --memory 1.5 \
  --sku standard \
  --ports 8000 \
  --protocol TCP \
  --ip-address Public \
  --os-type Linux \
  --location "West Central US"

```
### [PowerShell](#tab/powershell)
Create a container group profile using [New-AzContainerGroupProfile](/powershell/module/az.standbypool/new-AzStandbyContainerGroupPool).

```azurepowershell-interactive
New-AzContainerGroupProfile `
    -ResourceGroupName myResourceGroup `
    -Name myContainerGroupProfile `
    -Location "West Central US"  `
    -OsType Linux `
    -Image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" `
    -Cpu 1 `
    -MemoryInGB 1.5 `
    -Sku Standard `
    -Ports 8000 `
    -Protocol TCP `
    -IpAddressType Public

```

### [ARM template](#tab/template)
Create a container group profile and save the template file. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2023-05-15-preview",
      "name": "[parameters('profileName')]",
      "location": "[parameters('location')]",
      "properties": {
        "containers": [
          {
            "name": "myContainerGroupProfile",
            "properties": {
              "image": "[parameters('containerImage')]",
              "ports": [
                {
                  "port": 8000
                }
              ],
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGB": 1.5
                }
              },
              "command": [],
              "environmentVariables": []
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "TCP",
              "port": 8000
            }
          ]
        },
        "imageRegistryCredentials": [],
        "sku": "Standard"
      }
    }
  ],
  "parameters": {
    "profileName": {
      "type": "string",
      "defaultValue": "myContainerGroupProfile",
      "metadata": {
        "description": "Name of the container profile"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "West Central US",
      "metadata": {
        "description": "Location for the resource"
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
      "metadata": {
        "description": "The container image to use"
      }
    }
  }
}


```


### [REST](#tab/rest)
Create a container group profile using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile?api-version=2023-05-15-preview   

Request Body
{
    "location": "West Central US",
    "properties":{
        "containers": [
        {
            "name":"myContainerGroupProfile",
            "properties": {
                "command":[],
                "environmentVariables":[],
                "image":"mcr.microsoft.com/azuredocs/aci-helloworld:latest",
                "ports":[
                    {
                        "port":8000
                    }
                ],
                "resources": {
                    "requests": {
                        "cpu":1,
                        "memoryInGB":1.5
                                }
                            }
                        }
                    }
                ],
                "imageRegistryCredentials":[],
                "ipAddress":{
                "ports":[
                    {
                        "protocol":"TCP",
                        "port":8000
                    }
            ],
            "type":"Public"
            },
            "osType":"Linux",
            "sku":"Standard"
            }
        }
```

---

## Create a standby pool

### [CLI](#tab/cli)
Create a standby pool and associate it with a container group profile using [az standby-container-group-pool create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az standby-container-group-pool create \
   --resource-group myResourceGroup 
   --location WestCentralUS \
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
   -Location WestCentralUS `
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
           "defaultValue": "West Central US"    
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
           "defaultValue": "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile"
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


### [REST](#tab/rest)
Create a standby pool and associate it with a container group profile using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool?api-version=2023-03-01 
 
Request Body
{
    "properties": {
        "elasticityProfile": {
            "maxReadyCapacity": 20,
            "refillPolicy": "always"
        },
        "containerGroupProperties": {
            "containerGroupProfile": {
                "id": "/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile",
                "revision": 1
            }
        }
    },
    "location": "West Central US"
}
```

---

## Next steps

[Request a container from the standby pool](container-instances-standby-pool-request-container.md)
