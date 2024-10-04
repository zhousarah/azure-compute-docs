---
title: Get standby pool and instance details
description: Learn how to get details about your standby pool for Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 10/4/2024
ms.reviewer: ju-shim
---

# Get standby pool and instance details


## Standby pool details
Using the standby pool runtime view apis you can get the current status of your standby pool including how many instances are available, what state those instances are in and what zones are currently being used. 


### CLI (#tab/cli)

```azurecli
az standby-vm-pool status --resource-group myResourceGroup --name myScaleS2890-StandbyPool
{
  "id": "/subscriptions/401ef76a-dea9-45da-b19a-db3efced675b/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myScaleS2890-StandbyPool/runtimeViews/latest",
  "instanceCountSummary": [
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 0,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deallocating"
        },
        {
          "count": 3,
          "state": "Deallocated"
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ],
      "zone": 1
    },
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 0,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deallocating"
        },
        {
          "count": 2,
          "state": "Deallocated"
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ],
      "zone": 2
    },
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 0,
          "state": "Starting"
        },
        {
          "count": 0,
          "state": "Running"
        },
        {
          "count": 0,
          "state": "Deallocating"
        },
        {
          "count": 3,
          "state": "Deallocated"
        },
        {
          "count": 0,
          "state": "Deleting"
        }
      ],
      "zone": 3
    }
  ],
  "name": "latest",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews"
}

```


### PowerShell (#tab/powershell)

```azurepowershell
Get-AzStandbyVMPoolStatus `
-SubscriptionId f8da6e30-a9d8-48ab-b05c-3f7fe482e13b `
-ResourceGroupName test-standbypool `
-Name testPool

output
Id                           : /subscriptions/f8da6e30-a9d8-48ab-b05c-3f7fe482e13b/resourceGroups/test-standbypool/providers/Microsoft.Standb
                               yPool/standbyVirtualMachinePools/testPool/runtimeViews/latest
InstanceCountSummary         : {{
                                 "instanceCountsByState": [
                                   {
                                     "state": "Creating",
                                     "count": 0
                                   },
                                   {
                                     "state": "Starting",
                                     "count": 0
                                   },
                                   {
                                     "state": "Running",
                                     "count": 1
                                   },
                                   {
                                     "state": "Deallocating",
                                     "count": 0
                                   },
                                   {
                                     "state": "Deallocated",
                                     "count": 0
                                   },
                                   {
                                     "state": "Deleting",
                                     "count": 0
                                   }
                                 ]
                               }}
Name                         : latest
ProvisioningState            : Succeeded
ResourceGroupName            : test-standbypool
Type                         : Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews
```


### REST (#tab/rest)

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/{standbyVirtualMachinePoolName}/runtimeViews/{runtimeView}?api-version=2024-03-01

Sample output
{
  "properties": {
    "instanceCountSummary": [
      {
        "zone": 1,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 100
          },
          {
            "state": "running",
            "count": 20
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 100
          },
          {
            "state": "starting",
            "count": 0
          },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      },
      {
        "zone": 2,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 100
          },
          {
            "state": "running",
            "count": 20
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 100
          },
          {
            "state": "starting",
            "count": 0
          },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      },
      {
        "zone": 3,
        "instanceCountsByState": [
          {
            "state": "creating",
            "count": 100
          },
          {
            "state": "running",
            "count": 20
          },
          {
            "state": "deallocating",
            "count": 10
          },
          {
            "state": "deallocated",
            "count": 100
          },
          {
            "state": "starting",
            "count": 0
          },
          {
            "state": "deleting",
            "count": 0
          }
        ]
      }
    ],
    "provisioningState": "Succeeded"
  },
  "id": "/subscriptions/00000000-0000-0000-0000-000000000009/resourceGroups/rgstandbypool/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/pool/runtimeViews/latest",
  "name": "pool",
  "type": "Microsoft.StandbyPool/standbyVirtualMachinePools/runtimeViews",
  "systemData": {
    "createdBy": "pooluser@microsoft.com",
    "createdByType": "User",
    "createdAt": "2024-02-14T23:31:59.679Z",
    "lastModifiedBy": "pooluser@microsoft.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2024-02-14T23:31:59.679Z"
  }
}

```




## Instance details




## Standby pool instances
When a virtual machine is in a standby pool, the `isVmInStandbyPool` parameter is set to true. When the virtual machine is moved from the pool instance the scale set, the parameter is automatically updated to false. This can be useful in determining when a virtual machine is ready to recieve traffic or not. 

### [CLI](#tab/cli)

```azurecli
az vm get-instance-view --resource-group myResourceGroup --name myInstance

    "extensions": null,
    "hyperVGeneration": "V2",
    "isVmInStandbyPool": true,
    "maintenanceRedeployStatus": null,
    "statuses": [
      {
        "code": "ProvisioningState/succeeded",
        "displayStatus": "Provisioning succeeded",
        "level": "Info",
        "message": null,
        "time": "2024-08-02T17:22:46.295536+00:00"
      },
      {
        "code": "PowerState/deallocated",
        "displayStatus": "VM deallocated",
        "level": "Info",
        "message": null,
        "time": null
      }
    ],
```

### [REST](#tab/rest)

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myInstance/instanceView?api-version=2024-03-01

{
  "bootDiagnostics": {},
  "isVMInStandbyPool": true,
  "hyperVGeneration": "V2",
  "statuses": [
    {
      "code": "ProvisioningState/succeeded",
      "level": "Info",
      "displayStatus": "Provisioning succeeded",
      "time": "2024-08-02T17:22:46.2955369+00:00"
    },
    {
      "code": "PowerState/deallocated",
      "level": "Info",
      "displayStatus": "VM deallocated"
    }
  ]
}
```

---