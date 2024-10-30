---
title: Standby pools for Azure Container Instances
description: Learn how to utilize standby pools to reduce scale-out latency with Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: conceptual
ms.date: 10/29/2024
ms.reviewer: tomcassidy
---

# Standby pools for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances are currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

Standby Pools for Azure Container Instances enable customers to create a pool of pre-provisioned container groups that can be used to scale out in response to incoming traffic. The container groups in the pool are fully provisioned, initialized, and ready to receive work at any given notice.

## Limitations
Standby pools for Azure Container Instances is not available in the Azure portal. 

## Scaling

When you require a new container group, you can immediately pull one from the standby pool that is provisioned and running. 

When a container group is consumed from the pool, the standby pool automatically begins to refill ensuring that your standby pool maintains the desired capacity configured. 

Standby pools only give out container groups from the pool that are fully provisioned and ready to receive work. For example, the instances in your pool are still being initialized. In this case, the instances aren't in the expected provisioning state and are't given out when a container is requested. If no instances in the pool are available, Azure Container Instances will default back to net new container group creation. 

## Standby pool size
The number of container groups in a standby pool is determined by setting the `maxReadyCapacity` parameter. 

| Setting | Description | 
|---|---|
| maxReadyCapacity | The maximum number of container groups you want deployed in the pool.|

## Container group profile

The container group profile is what tells the standby pool how to configure the containers in the pool. Each standby pool is associated with a single container group profile. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.


```json
{
    "location":"{location}",
    "properties":{
        "containers": [
        {
            "name":"[containerProfileName]",
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

## Config maps

A config map is a property associated with a container group profile that can be used to apply container configurations similar to environment variables and secret volumes. When applying these settings, restarting the pod is required for the changes to take effect. By using config maps, the configurations can be applied without restarting the container. This enables out of band updates so containers can read the new values without restarting. 

Azure Container Instances can be created with or without config maps and can be updated at any point in time post creation using config maps. Updating config maps in an existing running container group can be accomplished quickly and without causing the container to reboot. 

```json
{
    "properties": {
        "containers": [
            {
                "name": "{containerProfileName}",
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
    "location": "{location}"
}

```




## Confidential containers
Standby pools for Azure container instances support confidential containers. To utilize [confidential containers](container-instances-confidential-overview.md) update the `sku` type to `Confidential` in the container group profile.

> [!IMPORTANT] 
> Values passed using config maps are not included in the security policy or validated by the runtime before the file mount is made available to the container. Any values that could have an impact to data or application security cannot be trusted by the application during execution and instead should be made available to the container using environment variables.


```json
{
    "location":"{location}",
    "properties":{
        "containers": [
        {
            "name":"{containerProfileName}",
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
Standby pools for Azure Container Instances support integration with Managed Identity. Applying a managed identity is performed when requesting a container from the standby pool and including the `identity` parameters and settings. 

```json
{
   "location": "{location}",
   "properties": {
       "standByPoolProfile":{
               "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.StandbyPool/standbyContainerGroupPools/{standbyPoolName}"
           },
           "containerGroupProfile": {
               "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{containerProfileName}",
               "revision": {revisionNumber}
           },
          },
          "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
              "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identity}": {}
            },
           "containers": [
               {
                   "name": "{containerProfileName}",
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

[Create a standby pool for Azure Container Instances](container-instances-standby-pool-create.md)
