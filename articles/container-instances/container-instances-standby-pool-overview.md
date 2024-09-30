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

## Container Group Profile

The container group profile is what tells the standby pool how to configure the containers in the pool. Each standby pool is associated with a single container group profile. If you make changes to the container group profile, you also need to update your standby pool to ensure the updates are applied to the instances in the pool.

```rest
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


## Standby pool instances
You can get a list of instances in your standby pool and their current state. 

### [CLI](#tab/cli)

```cli
az standby-container-group-pool status --resource-group myResourceGroup --name myContainerPool
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myContainerPool/runtimeViews/latest",
  "instanceCountSummary": [
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 5,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ]
    }
  ],
  "name": "latest",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews"
}
```

### [REST](#tab/rest)

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myContainerPool/runtimeViews/latest?api-version=2024-03-01

{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myContainerPool/runtimeViews/latest",
  "name": "latest",
  "type": "Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews",
  "properties": {
    "instanceCountSummary": [
      {
        "instanceCountsByState": [
          {
            "state": "Creating",
            "count": 0
          },
          {
            "state": "Running",
            "count": 5
          },
          {
            "state": "Deleting",
            "count": 0
          }
        ]
      }
    ],
    "provisioningState": "Succeeded"
  }
}
```

---


## Next steps

Learn more about [Azure Container Instances](container-instances-overview.md)
