---
title: Use multiple Virtual Machine sizes with Instance Mix (Preview)
description: Use multiple Virtual Machine sizes in a scale set using Instance Mix. Optimize deployments using allocation strategies. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 11/18/2024
ms.reviewer: jushiman
---

# Use multiple Virtual Machine sizes with Instance Mix (Preview)
> [!IMPORTANT]
> Instance Mix for Virtual Machine Scale Sets with Flexible Orchestration Mode is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

Instance Mix enables you to specify multiple different Virtual Machine (VM) sizes in your Virtual Machine Scale Set with Flexible Orchestration Mode, and an allocation strategy to further optimize your deployments. 

Instance Mix is best suited for workloads that are flexible in compute requirements and can be run on various different sized VMs. Using Instance Mix you can:
- Deploy a heterogeneous mix of VM sizes in a single scale set. You can view max scale set instance counts in the [documentation](./virtual-machine-scale-sets-orchestration-modes.md#what-has-changed-with-flexible-orchestration-mode).
- Optimize your deployments for cost or capacity through allocation strategies.
- Continue to make use of scale set features, like [Spot Priority Mix](./spot-priority-mix.md), [Autoscale](./virtual-machine-scale-sets-autoscale-overview.md), or [Upgrade Policies](./virtual-machine-scale-sets-set-upgrade-policy.md).
- Spread a heterogeneous mix of VMs across Availability Zones and Fault Domains for high availability and reliability.

## Changes to existing scale set properties
### sku.name
The `sku.name` property should be set to `"Mix"`. VM sizes will be defined in the `skuProfile`.
### sku.tier
The `sku.tier` property is currently an optional scale set property and should be set to `null` for Instance Mix scenarios.

### sku.capacity
The `sku.capacity` property continues to represent the overall size of the scale set in terms of the total number of VMs.

### scaleInPolicy
The optional scale-in property isn't needed for scale set deployments using Instance Mix. During scaling in events, the scale set utilizes the allocation strategy to inform the decision on which VMs should be scaled in. For example, when you use `LowestPrice`, the scale set scales in by removing the more expensive VMs first.

## New scale set properties
### skuProfile
The `skuProfile` property represents the umbrella property for all properties related to Instance Mix, including VM sizes and allocation strategy.

### vmSizes
The `vmSizes` property is where you specify the specific VM sizes that you're using as part of your scale set deployment with Instance Mix.

### allocationStrategy
Instance Mix introduces the ability to set allocation strategies for your scale set. The `allocationStrategy` property is where you specify which allocation strategy you'd like to use for your Instance Mix scale set deployments. There are two options for allocation strategies, `lowestPrice` and `capacityOptimized`. Allocation strategies apply to both Spot and Standard VMs.

#### lowestPrice (default)
This allocation strategy is focused on workloads where cost and cost-optimization are most important. When evaluating what VM split to use, Azure looks at the lowest priced VMs of the VM sizes specified. Azure also considers capacity as part of this allocation strategy. The scale set deploys as many of the lowest priced VMs as it can, depending on available capacity, before moving on to the next lowest priced VM size specified. `lowestPrice` is the default allocation strategy.

#### capacityOptimized
This allocation strategy is focused on workloads where attaining capacity is the primary concern. When evaluating what VM size split to deploy in the scale set, Azure looks only at the underlying capacity available. It doesn't take price into account when determining what VMs to deploy. Using `capacityOptimized` can result in the scale set deploying the most expensive, but most readily available VMs. 

#### Prioritized
This allocation strategy allows you to specify a priority ranking to the VM sizes specified. Note: ranking is optional, but if provided, it must be within the range of the `vmSizes` list size. Ranks can be duplicated across sizes, meaning the sizes have the same priority. Ranks don't need to be in sequential order.

## Cost
Following the scale set cost model, usage of Instance Mix is free. You continue to only pay for the underlying resources, like the VM, disk, and networking.

## Limitations
- Instance Mix is currently available in the following regions: West US, West US2, East US, and East US2. 
- Instance Mix is only available for scale sets using Flexible Orchestration Mode.
- You must have quota for the VM sizes you're requesting with Instance Mix.
- You can specify **up to** five VM sizes with Instance Mix at this time.
- The `Prioritized` allocation strategy is currently only supported via REST and Azure portal deployments. 
- Existing scale sets can't be updated to use Instance Mix. 
- VM sizes can't be changed once the scale set is deployed.
- For REST API deployments, you must have an existing virtual network inside of the resource group that you're deploying your scale set with Instance Mix in.

## Next steps
Learn how to [create a scale set using Instance Mix](instance-mix-create.md).