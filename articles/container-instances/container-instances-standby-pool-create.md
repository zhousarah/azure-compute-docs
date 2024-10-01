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

## Create a container group profile
The container group profile is what tells the standby pool how to configure the containers in the pool. Each standby pool is associated with a single container group profile. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.

### [CLI](#tab/cli)
Create a container group profile using [az container-profile create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az container-profile create \
  --resource-group myResourceGroup \
  --name myContainerGroupProfile \
  --image "mcr.microsoft.com/azuredocs/aci-helloworld:latest" \
  --cpu 1 \
  --memory 1.5 \
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
    -Image $"mcr.microsoft.com/azuredocs/aci-helloworld:latest" `
    -Cpu 1 `
    -MemoryInGB 1.5 `
    -Ports 8000 `
    -Protocol TCP `
    -IpAddressType Public

```

### [ARM template](#tab/template)
Create a container group profile and deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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
            "name": "mycontainerprofile",
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


### [Bicep](#tab/bicep)
Create a container group profile and deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param subscriptionId string
param resourceGroupName string
param profileName string
param location string = 'West Central US'
param containerImage string = 'mcr.microsoft.com/azuredocs/aci-helloworld:latest'
param containerPort int = 8000
param cpuCores int = 1
param memoryInGb float = 1.5

resource containerGroup 'Microsoft.ContainerInstance/containerGroups@2023-05-15-preview' = {
  name: profileName
  location: location
  properties: {
    containers: [
      {
        name: 'myContainerGroupProfile'
        properties: {
          image: containerImage
          ports: [
            {
              port: containerPort
            }
          ]
          resources: {
            requests: {
              cpu: cpuCores
              memoryInGb: memoryInGb
            }
          }
          command: []
          environmentVariables: []
        }
      }
    ]
    osType: 'Linux'
    imageRegistryCredentials: []
    ipAddress: {
      type: 'Public'
      ports: [
        {
          protocol: 'TCP'
          port: containerPort
        }
      ]
    }
    sku: 'Standard'
  }
}


```

### [REST](#tab/rest)
Create a container group profile using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
PUT
PUT https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile?api-version=2023-05-15-preview   

Request Body
{
    "location":"West Central US",
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


### [Bicep](#tab/bicep)
Create a standby pool and associate it with a container group profile. Deploy the template using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```bicep
param location string = resourceGroup().location
param standbyPoolName string = 'myStandbyPool'
param maxReadyCapacity int = 20
param containerGroupProfile string = '/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile'

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
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool?api-version=2023-12-01-preview 
 
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

Learn more about [standby pools for Azure Container Instances](container-instances-standby-pool-overview.md)
