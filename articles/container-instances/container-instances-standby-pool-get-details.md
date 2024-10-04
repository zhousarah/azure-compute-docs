---
title: Get standby pool and instance details
description: Learn how to get details about your standby pool for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 10/4/2024
ms.reviewer: ju-shim
---

# Get standby pool and instance details
This article discusses how to retrieve various information about your standby pool and the container groups within it. 

## Standby pool details
Use the standby pool runtime view APIs to get the current status of your standby pool including how many container groups are available and their current provisioning state.


### [CLI](#tab/cli)

```azurecli
az standby-container-group-pool status --resource-group myResourceGroup --name myStandbyPool
```

**Sample output**
```azurecli
{
  "id": "/subscriptions/401ef76a-dea9-45da-b19a-db3efced675b/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
  "instanceCountSummary": [
    {
      "instanceCountsByState": [
        {
          "count": 0,
          "state": "Creating"
        },
        {
          "count": 20,
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


### [PowerShell](#tab/powershell)

```azurepowershell
Get-AzStandbyContainerGroupPoolStatus -ResourceGroupName myResourceGroup -Name myStandbyPool
```

**Sample output**
```azurepowershell
Id: /subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest
InstanceCountSummary         : {{
        "instanceCountsByState": [
        {
            "state": "Creating",
            "count": 0
        },
        {
            "state": "Running",
            "count": 0
        },
        {
            "state": "Deleting",
            "count": 0
        }
        ],
    }
  }
Name                         : latest
ProvisioningState            : Succeeded
ResourceGroupName            : myResourceGroup
Type                         : Microsoft.StandbyPool/standbyContainerGroupPools/runtimeViews

```


### [REST](#tab/rest)

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest?api-version=2024-03-01

```

**Sample output**
```rest
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyContainerGroupPools/myStandbyPool/runtimeViews/latest",
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
            "count": 20
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


## Container group details

When a virtual machine is in a standby pool, the `isVmInStandbyPool` parameter is set to true. When the virtual machine is moved from the pool instance the scale set, the parameter is automatically updated to false. This can be useful in determining when a virtual machine is ready to receive traffic or not. This information is also available to query via [Azure Instance Metadata Service](../virtual-machines/instance-metadata-service.md)

### [CLI](#tab/cli)

```azurecli
 az container show --resource-group myResourceGroup --name mystandbypool_0682ad54
{
  "confidentialComputeProperties": null,
  "containers": [
    {
      "command": [],
      "environmentVariables": [],
      "image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
      "instanceView": {
        "currentState": {
          "detailStatus": "",
          "exitCode": null,
          "finishTime": null,
          "startTime": "2024-10-04T22:22:36.416000+00:00",
          "state": "Running"
        },
        "events": [
          {
            "count": 1,
            "firstTimestamp": "2024-10-04T22:22:20+00:00",
            "lastTimestamp": "2024-10-04T22:22:20+00:00",
            "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-helloworld@sha256:565dba8ce20ca1a311c2d9485089d7ddc935dd50140510050345a1b0ea4ffa6e\"",
            "name": "Pulling",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2024-10-04T22:22:26+00:00",
            "lastTimestamp": "2024-10-04T22:22:26+00:00",
            "message": "Successfully pulled image \"mcr.microsoft.com/azuredocs/aci-helloworld@sha256:565dba8ce20ca1a311c2d9485089d7ddc935dd50140510050345a1b0ea4ffa6e\"",
            "name": "Pulled",
            "type": "Normal"
          },
          {
            "count": 1,
            "firstTimestamp": "2024-10-04T22:22:36+00:00",
            "lastTimestamp": "2024-10-04T22:22:36+00:00",
            "message": "Started container",
            "name": "Started",
            "type": "Normal"
          }
        ],
        "previousState": null,
        "restartCount": 0
      },
      "livenessProbe": null,
      "name": "mycontainerprofile",
      "ports": [
        {
          "port": 8000,
          "protocol": null
        }
      ],
      "readinessProbe": null,
      "resources": {
        "limits": null,
        "requests": {
          "cpu": 1.0,
          "gpu": null,
          "memoryInGb": 1.5
        }
      },
      "securityContext": null,
      "volumeMounts": null
    }
  ],
  "diagnostics": null,
  "dnsConfig": null,
  "encryptionProperties": null,
  "extensions": null,
  "id": "/subscriptions/401ef76a-dea9-45da-b19a-db3efced675b/resourceGroups/myresourcegroup/providers/Microsoft.ContainerInstance/containerGroups/mystandbypool_0682ad54",
  "identity": null,
  "imageRegistryCredentials": [],
  "initContainers": [],
  "instanceView": {
    "events": [],
    "state": "Running"
  },
  "ipAddress": {
    "autoGeneratedDomainNameLabelScope": null,
    "dnsNameLabel": null,
    "fqdn": null,
    "ip": "68.219.45.164",
    "ports": [
      {
        "port": 8000,
        "protocol": "TCP"
      }
    ],
    "type": "Public"
  },
  "location": "northeurope",
  "name": "mystandbypool_0682ad54",
  "osType": "Linux",
  "priority": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "restartPolicy": null,
  "sku": "Standard",
  "subnetIds": null,
  "tags": null,
  "type": "Microsoft.ContainerInstance/containerGroups",
  "volumes": null,
  "zones": null
}
```

### [PowerShell](#tab/powershell)

```azurepowershell
get-azVM -resourceGroupName myResourceGroup -name myVM -status


ResourceGroupName : myResourceGroup
Name              : myVM
HyperVGeneration  : V2
IsVMInStandbyPool : True
BootDiagnostics   : 
Disks[0]          : 
  Name            : myVM_1a995f8c_OsDisk_1_c250259c33b942eb910504cf6512332d
  Statuses[0]     : 
    Code          : ProvisioningState/succeeded
    Level         : Info
    DisplayStatus : Provisioning succeeded
    Time          : 10/4/2024 7:45:11 PM
Statuses[0]       : 
  Code            : ProvisioningState/succeeded
  Level           : Info
  DisplayStatus   : Provisioning succeeded
  Time            : 10/4/2024 7:45:11 PM
Statuses[1]       : 
  Code            : PowerState/deallocated
  Level           : Info
  DisplayStatus   : VM deallocated
```


### [REST](#tab/rest)

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM/instanceView?api-version=2024-03-01

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

## Next steps
Review the most [frequently asked questions](standby-pools-faq.md) about standby pools for Virtual Machine Scale Sets.
