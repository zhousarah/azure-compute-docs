---
title: Frequently asked questions about standby pools for Azure Conatiner Instances
description: Get answers to frequently asked questions for standby pools for Azure Container Instances.
author: mimckitt
ms.author: mimckitt
ms.service: azure-container-instances
ms.topic: how-to
ms.date: 11/1/2024
ms.reviewer: tomvcassidy
---

# Frequently asked questions about standby pools for Azure Container Instances (Preview)

> [!IMPORTANT]
> Standby pools for Azure Container Instances is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA).

Get answers to frequently asked questions about standby pools for Azure Container Instances. 

### How long can my standby pool name be? 
A standby pool can be anywhere between 3 and 24 characters. For more information, see [Resource naming restrictions for Azure resources](/azure/azure-resource-manager/management/resource-name-rules)

### How many container groups can my standby pool for Azure Container Instances have?  
The maximum number of container groups in a standby pool is 2,000.  

### Can I change the size of my standby pool without needing to recreate it? 
Yes. To change the size of your standby pool update the max ready capacity setting.  

### Can a standby pool resource be moved?
No. Standby Pools doesn't currently support that capability. If you need to move a standby pool, you can consider deleting it recreating it in another location.

### I created a standby pool and I noticed that some containers are coming up in a failed state. 
Ensure you have enough quota to complete the standby pool creation. Insufficient quota results in the platform attempting to create the containers in the standby pool but unable to successfully complete the create operation. Check for multiple types of quotas such as Cores, Network Interfaces, IP Addresses, etc.

### I requsted a container from my pool but it created a new container instead. 
Ensure that the container groups in your standby pool are in the running state prior to issuing a request. For example, if using a standby If containers are in any other states such as creating or deleting, the container request will default to creating a new container group from scratch. 


## Next steps

Learn more about [standby pools on Azure Container Instances](container-instances-standby-pool-overview.md).
