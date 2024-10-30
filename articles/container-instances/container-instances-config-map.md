---
title: Config maps for Azure Container Instances (Preview)
description: Learn how to use config maps with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 10/30/2024
ms.reviewer: tomvcassidy
---
# Config maps for Azure Container Instances

A config map is a property associated with a container group profile that can be used to apply container configurations similar to environment variables and secret volumes. When applying these settings, restarting the pod is required for the changes to take effect. By using config maps, the configurations can be applied without restarting the container. This enables out of band updates so containers can read the new values without restarting. 

Azure Container Instances can be created with or without config maps and can be updated at any point in time post creation using config maps. Updating config maps in an existing running container group can be accomplished quickly and without causing the container to reboot. 


## How it works

A config map can be included in the container properties or in a container group profile. Creating a container group profile with the config map settings makes applying those settings simple and easy to automate. 

### Create a container group profile with config map settings 

> [!NOTE]
> To use [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` when creating your container group profile.

### [CLI](#tab/cli)
Create a container group profile using [az container-profile create](/cli/azure/standby-container-group-pool).

```azurecli-interactive
az container-profile create \
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
  --config-map $newKey=$newValue

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
    -ConfigMap @{ $newKey = $newValue }

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
              "configMap": {
                  "keyValuePairs": {
                        "key1": "value1",
                        "key2": "value2"
                                   }                   
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
                "configMap": {
                  "keyValuePairs": {
                        "key1": "value1",
                        "key2": "value2"
                                   }
                            },     
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

### Apply config map settings using a container group profile

### [CLI](#tab/cli)
Apply the config map settings stored in the container group profile using [az container update](/cli/azure/container).

```azurecli-interactive
az container update \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --location "West Central US" \
  --container-group-profile "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" \
  --revision 1 \
  --container-name myContainerProfile 


```
### [PowerShell](#tab/powershell)
Apply the config map settings stored in the container group profile using [Update-AzContainer](/powershell/module/az.containerinstance/update-azcontainergroup).

```azurepowershell-interactive
Update-AzContainer `
    -ResourceGroupName myResourceGroup `
    -Location "West Central US" `
    -ContainerGroupName myContainerGroup `
    -ContainerGroupProfileId "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile" `
    -ContainerGroupProfileRevision 1 `
    -ContainerName myContainerProfile 

```

### [ARM template](#tab/template)
Apply the config map settings stored in the container group profile using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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


### [REST](#tab/rest)
Apply the config map settings stored in the container group profile using [Create or Update](/rest/api/container-instances/container-groups/create-or-update)

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2023-05-01 

Request Body
{
   "location": "{location}",
   "properties": {
           "containerGroupProfile": {
               "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroupProfiles/myContainerGroupProfile",
               "revision": {revisionNumber}
           },
           "containers": [
               {
                   "name": "{myContainerGroupProfile}",
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


Once the update has  been applied to an existing container and you will see the values mounted in the container without requiring a restart.

```
/mnt/configmap/<containername>/key1 with value as “value1”  

/mnt/configmap/<containername>/key2 with value as “value2”  
```

### Apply config map settings without container group profile

Config map settings can also be applied directly to the instance by specifying the details in the update commands. 


### [CLI](#tab/cli)
Apply the config map settings using [az container update](/cli/azure/container).

```azurecli-interactive
az container update \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --location "West Central US" \
  --container-name myContainerProfile \
  --config-map $newKey=$newValue

```
### [PowerShell](#tab/powershell)
Apply the config map settings using [Update-AzContainer](/powershell/module/az.containerinstance/update-azcontainergroup).

```azurepowershell-interactive
Update-AzContainer `
    -ResourceGroupName myResourceGroup `
    -Location "West Central US" `
    -ContainerGroupName myContainerGroup `
    -ContainerName myContainerProfile `
    -ConfigMap @{ $newKey = $newValue }
```

### [ARM template](#tab/template)
Apply the config map settings using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).


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
  },
  "resources": [
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2023-05-01",
      "name": "[parameters('containerGroupName')]",
      "location": "[parameters('location')]",
      "properties": {
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


### [REST](#tab/rest)
Apply the config map settings using [Create or Update](/rest/api/container-instances/container-groups/create-or-update)

```HTTP
PUT
https://management.azure.com/subscriptions/{SubscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/myContainerGroup?api-version=2023-05-01 

Request Body
{
   "location": "{location}",
   "properties": {
           "containers": [
               {
                   "name": "{myContainerGroupProfile}",
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


Once the update has  been applied to an existing container and you will see the values mounted in the container without requiring a restart.

```
/mnt/configmap/<containername>/key1 with value as “value1”  

/mnt/configmap/<containername>/key2 with value as “value2”  
```

## Next steps
Learn how to use config maps with [standby pools to increase scale and availability](container-instances-standby-pool-get-details.md)