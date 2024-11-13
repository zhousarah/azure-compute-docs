---
title: Enabling retries on virtual machine creates and deletes in Virtual Machine Scale Sets (Preview)
description: Learn how to enable retries on failed virtual machine creates and deletes. 
author: manasisoman
ms.author: manasisoman-work
ms.service: azure-virtual-machine-scale-sets
ms.topic: how-to
ms.date: 10/18/2024
ms.reviewer: ju-shim
---


# Enabling retries on virtual machine creates and deletes in Virtual Machine Scale Sets (Preview)
## Feature overview
Resilient create and delete for Virtual Machine Scale Sets helps to reduce virtual machine create and delete errors by automatically retrying failed operations. Failed virtual machines can accumulate and result in unusable capacity requiring manual effort to detect and clean up. These errors are rare, but this mechanism is built for customers who are deploying or deleting large scales of Scale Sets or virtual machines. 
> [!IMPORTANT]
> Resilient virtual machine create and delete for Virtual Machine Scale Sets is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 
## Prerequisites
Before utilizing Resilient create and delete, complete the feature registration and ensure your API policy is on at least version 2023-07-01.

### Feature Registration 
Register for the ResilientScaleSetVMCreation and ReliableVMDeletion feature flags using the az feature register command: 

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "ResilientVMScaleSetVMCreation" 
az feature register --namespace "Microsoft.Compute" --name "ReliableVMDeletion" 
```

It takes a few moments for the feature to register. Verify the registration status by using the az feature show command:
```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "ResilientVMScaleSetVMCreation"
az feature show --namespace "Microsoft.Compute" --name "ReliableVMDeletion"
```
## How it works
### Resilient create
1.	Resilient create runs on virtual machines scale sets during the initial create or a scale-out. 
2.	Resilient create initiates retries for OS Provisioning Timeout and Virtual Machine Start Timeout errors. Timeouts are hit when a virtual machine isn't provisioned after 20 minutes (Windows) or eight minutes (Linux).
3.	Resilient create attempts the create operation five times per virtual machine or for a maximum of 30 total minutes for all retries. If unsuccessful, the virtual machine remains in a failed state.

:::image type="content" source="./media/resilient-vm-create-delete/resilient-create-workflow.png" alt-text="A screenshot showing how Resilient create performs retries on your virtual machines.":::

### Resilient delete
1.	Resilient delete initiates forced delete retries for any errors that occur during the delete process. For example, InternalExecutionError, TransientFailure, or InternalOperationError.
2.	Resilient delete attempts the forced delete operation five times per virtual machine with an exponential backoff. If unsuccessful, the virtual machine remains in a failed state.
> For example, if you are deleting a scale set of five virtual machines, and each virtual machine enters a “failed” delete state, then the scale set will initiate one delete call on itself to delete those five virtual machines again. If four out of five virtual machines are deleted in the first retry, then the platform will wait a period of 10 minutes before initiating the next delete call for the remaining virtual machine.
4.	To check the status of virtual machines throughout the delete process, refer to the section on [Get create or delete status](#get-status-of-create-and-delete).

:::image type="content" source="./media/resilient-vm-create-delete/resilient-delete-workflow.png" alt-text="A screenshot showing how Resilient delete performs retries on your virtual machines.":::

## Enabling Resilient create and delete

### [Portal](#tab/portal)

To enable on an _existing_ scale set:
1) Log in to your Azure portal using your login with the feature flag enabled
2) Navigate to your Virtual Machine Scale Set
3) Under **Capabilities** select **Health and repair**
4) Enable **Resilient VM create (Preview)** and **Resilient VM delete (Preview)**

:::image type="content" source="media/standby-pools/enable-standby-pool-after-vmss-creation.png" alt-text="A screenshot showing how to enable Resilient create and delete on an existing Virtual Machine Scale Set in the Azure portal.":::

To enable on a _new_ scale set:
1) Navigate to **Monitoring**
2) Select checkboxes **Resilient VM create (Preview)** and **Resilient VM delete (Preview)**

:::image type="content" source="media/standby-pools/enable-standby-pool-after-vmss-creation.png" alt-text="A screenshot showing how to enable Resilient create and delete on a new Virtual Machine Scale Set in the Azure portal.":::

### [CLI](#tab/cli)
Enable Resilient create and delete on your existing scale set using [az standby-vm-pool create](/cli/azure/standby-vm-pool).

```azurecli-interactive
az vmss update \ 
--name <myScaleSet> \ 
--resource-group <myResourceGroup> \ 
--enable-resilient-deletion true 
```

```azurecli-interactive
az vmss update 
--name <myScaleSet> \ 
--resource-group <myResourceGroup>\ 
--set resiliencyPolicy.resilientVMCreationPolicy=true 
```

### [PowerShell](#tab/powershell)
Enable Resilient create and delete on your existing scale set using [New-AzStandbyVMPool](/powershell/module/az.standbypool/new-azstandbyvmpool).

```azurepowershell-interactive
#Create a VM Scale Set profile 
$vmss = new-azvmssconfig -EnableResilientVMCreate -EnableResilientVMDelete 
#Update the VM Scale Set with Resilient virtual machine create and Resilient virtual machine delete 
Update-azvmss -ResourceGroupName $resourceGroupName -VMScaleSetName 
$VMScaleSetName -EnableResilientVMCreate -EnableResilientVMDelete 
```

### [ARM template](#tab/template)
Enable Resilient create and delete on your existing scale set using [az deployment group create](/cli/azure/deployment/group) or [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment).

```json
"properties": {  
  "overprovision": false,  
    " resiliencyPolicy ": {
        "resilientVMCreationPolicy": {  
            "enabled": true  
        },  
        "resilientVMDeletionPolicy": {  
            "enabled": true  
        }  
    }  
```

### [REST](#tab/rest)
Create a standby pool and associate it with an existing scale set using [Create or Update](/rest/api/standbypool/standby-virtual-machine-pools/create-or-update)

```HTTP
PUT https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.StandbyPool/standbyVirtualMachinePools/myStandbyPool?api-version=2023-12-01-preview
{
"type": "Microsoft.StandbyPool/standbyVirtualMachinePools",
"name": "myStandbyPool",
"location": "east us",
"properties": {
	 "elasticityProfile": {
		 "maxReadyCapacity": 20
	 },
	  "virtualMachineState":"Deallocated",
	  "attachedVirtualMachineScaleSetId": "/subscriptions/{subscriptionID}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
	  }
}
```
---

## Get status of create and delete
### Resilient Create
Your virtual machine status is 'Creating' while Resilient create is in progress. 

### Resilient Delete
While the delete attempt is in progress, the state of the resource is listed as 'Deleting.' If a delete retry fails on a particular virtual machine, then the virtual machine falls back to the 'Failed' State. However, the 'Failed' State only indicates that a retry of a deletion failed – and Resilient delete might still perform more retries. Therefore, while Resilient delete is going on, you see the virtual machine alternate states between 'Deleting' and 'Failed.' 

### [Portal](#tab/portal)
Navigate to your Virtual Machine Scale Set. The banner indicates when Resilient Delete is operating:
:::image type="content" source="media/standby-pools/enable-standby-pool-after-vmss-creation.png" alt-text="A screenshot showing the banner during Resilient delete retries in the Azure portal.":::

### [REST API](#tab/rest)
To know the status of your virtual machine during delete, retrieve the return value of the ResilientVMDeletionStatus property. There are two different API endpoints available to get the ResilientVMDeletionStatus:

This endpoint supports only Virtual Machine Scale Sets Uniform.
```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{VMSSName}}/virtualMachines?api-version=2024-07-01 
```
This endpoint supports both Virtual Machine Scale Sets Uniform and Flex.
```http
GET https://management.azure.com/subscriptions/{{subscriptionId}}/resourceGroups/{{ResourceGroupName}}/providers/Microsoft.Compute/virtualMachineScaleSets/{{ResourceName}}/VirtualMachines/{{VMName}}?$expand=resiliencyView&api-version=2024-07-01
```
---


If you use REST API, different return values of ResilientVMDeletionStatus indicate the following:
| ResilientVMDeletionStatus | State of delete                                                                                                                                                                  |
|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Enabled                   | Your scale set has a resiliency policy with the resilientVMDeletion policy set.                                                                                                  |
| Disabled                  | Your scale set either has resilientVMDeletion policy enabled as false, has a resiliency policy but is missing a resilientVMDeletion policy, or doesn't have a resiliency policy. |
| In Progress               | Your scale set has the resilientVMDeletion enabled and the virtual machine is currently being deleted or is marked for deletion.                                                 |
| Failed                    | Your scale set has the resilientVMDeletion enabled and hit the max retry count.                                                                                                  |



## FAQs
### What is the minimum API version to use this policy? 
2023-07-01

### What do I do if my virtual machine is in a 'Failed' state for a long time? 
Resilient delete performs a maximum of five retries on your virtual machine. Therefore, your virtual machine might show up in a 'Failed' state, even when Resilient delete is operating on that virtual machine. See status section.

### Does Resilient create work when I attach a new virtual machine to my scale set? 
No. Resilient create operates during a scale-out of a scale set or when you create a new scale set. 

### Is the provisioning of my virtual machine accelerated with Resilient create?
No, Resilient create improves the odds of provisioning the virtual machine but doesn't improve the speed of the provisioning itself. 
