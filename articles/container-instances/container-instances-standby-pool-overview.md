---
title: Standby pools for Azure Container Instances
description: Learn how to utilize standby pools to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: conceptual
ms.date: 09/26/2024
ms.reviewer: tomcassidy
---

# Standby pools for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances are currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

Standby Pools for Azure Container Instances enable customers to create a pool of pre-provisioned container groups that can be used to scale out in response to incoming traffic. The container groups in the pool are fully provisioned, initialized, and ready to receive work at any given notice.

## Scaling

When you require a new container group you can immediately pull one from the standby pool that is provisioned and running. 

When a container group is consumed from the pool, the standby pool will automatically begin to refill ensuring that your standby pool maintains the desired capacity configured. 

Standby pools only give out container groups from the pool that are fully provisioned and ready to receive work. For example, the instances in your pool are still being initialized, they aren't in the expected provisioning state and won't be given out when a container is requested. If no instances in the pool are available, Azure Container Instances will default back to net new container group creation. 

## Standby pool size
The number of container groups in a standby pool is determined by setting the `maxReadyCapacity` parameter. 

| Setting | Description | 
|---|---|
| maxReadyCapacity | The maximum number of container groups you want deployed in the pool.|

## Config maps

Azure Container Instances supports multiple ways to apply container configurations such as environment variables and secret volumes. When applying these settings, restarting the pod is required for the changes to take affect. By using config maps, the configurations can be applied without restarting the container. This enables out of band updates so containers can read the new values without restarting. 

Azure Container Instances can be created with or without config maps and can be updated at any point in time post creation using config maps. Updating config maps in an existing running container group can be accomplished quickly and without causing the container to reboot. 

```json
{
    "properties": {
        "containers": [
            {
                "name": "con1",
                "properties": {
                    "image": "mcr.microsoft.com/azuredocs/aci-helloworld",
                    "ports": [
                        {
                            "port": 80,
                            "protocol": "TCP"
                        }
                    ],
                    "resources": {
                        "requests": {
                            "memoryInGB": 0.5,
                            "cpu": 0.5
                        }
                    },
                    "configMap": {
                        "keyValuePairs": {
                            "key1": "value1",
                            "key2": "value2"
                        }
                    }
                }
            }
        ],
        "osType": "Linux",
        "ipAddress": {
            "type": "Public",
            "ports": [
                {
                    "protocol": "tcp",
                    "port": 80
                }
            ]
        }
    },
    "location": "westcentralus"
}

```


## Container Group Profile

The container group profile is what tells the standby pool how to configure the containers in the pool. Each standby pool is associated with a single container group profile. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.


```json
{
    "location":"west central us",
    "properties":{
        "containers": [
        {
            "name":"mycontainerprofile",
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

## Confidential containers
Standby pools for Azure container instances support confidential containers. To utilize [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` in the container group profile.

```json
{
    "location":"west central us",
    "properties":{
        "containers": [
        {
            "name":"mycontainerprofile",
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
            "sku":"Confidential"
            }
        }


```
## Managed Identity
Standby pools for Azure Container Instances supports integration with Managed Identity. Applying a managed identity is performed when requesting a container from the standby pool. 

```json
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
          },
          "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
              "/subscriptions/[subscriptionId]/resourceGroups/[resourceGroup]/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identity}": {}
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

## Next steps

Learn more about [Azure Container Instances](container-instances-overview.md)
