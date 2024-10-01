---
title: N-Phase rolling upgrades on Virtual Machine Scale Sets (Preview)
description: Learn about how to configure N-Phase rolling upgrades on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 9/25/2024
ms.reviewer: ju-shim
ms.custom: upgradepolicy, N-Phase
---
# N-Phase rolling upgrades on Virtual Machine Scale Sets (Preview)

N-Phase rolling upgrades enables you to select which virtual machines are placed into each batch when performing a rolling upgrade. Additionally, N-Phase upgrades enables you to skip upgrades on specific virtual machines during the rolling upgrade process. 

> [!NOTE]
>**N-Phase rolling upgrades Virtual Machine Scale Sets is currently in preview.** Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of these features may change prior to general availability (GA).


## Requirements

When using N-Phase rolling upgrades on Virtual Machine Scale Sets, the scale set must also use the [Application Health Extension](virtual-machine-scale-sets-health-extension.md) to monitor application health and report phase ordering information. 

## Concepts

A phase is a high-level grouping construct for VMs which is determined by customers setting metadata emitted to the app health extension. It is a way to communicate from runtime to control plane on which phase a VM will belong. Rolling Upgrade will decide on batches of VMs based on Update Domains (UD), Fault Domains (FD) and/or Zones to ensure that each batch does not cross a phase boundary. Batching will be performed within each of the phases. The phased upgrades are performed in numerical sequence order . Meaning that until all the batches in the first phase are upgraded, none of the VMs in second phase will be touched.

:::image type="content" source="./media/upgrade-policy/n-phase-regional-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a regional scale set.":::


:::image type="content" source="./media/upgrade-policy/n-phase-zonal-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a zonal scale set.":::


To determine the phase number for the VM, the customer needs to leverage The Application Health Extension. Each VM will respond to probes with response body contents containing metadata key-value pairs. This metadata will tell the platform how each VM should interact with Rolling Upgrades. 
An example response body contents is shown below. To specify phase number, use “phaseOrderingNumber” which is a platform understood key. 

```
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": false, \"PhaseOrderingNumber\": 0 } }"
}
```

For skipping an upgrade on a VM, customer can emit "rollingUpgrade.SkipUpgrade” : true and RollingUpgrader will not upgrade the VM. 

```
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": true, \"PhaseOrderingNumber\": 1 } }"
}
```

Once the customer has successfully configure the application health extension and custom metrics on each VM, when a rolling upgrade is initiated, the VMs will be placed into their designated phases and each phase will inherit the rolling upgrade policy associated with the scale set. This rolling upgrade policy will determine batch size, max healthy %, MaxSurge, etc.  


## Next steps
Learn how to [perform manual upgrades](virtual-machine-scale-sets-perform-manual-upgrades.md) on Virtual Machine Scale Sets. 
