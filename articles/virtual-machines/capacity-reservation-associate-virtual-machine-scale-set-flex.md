---
title: Associate a virtual machine scale set with flexible orchestration to a capacity reservation group (preview)
description: Learn how to associate a new virtual machine scale set with flexible orchestration mode to a capacity reservation group.
author: bdeforeest
ms.author: bidefore
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 11/22/2022
ms.reviewer: cynthn, jushiman
ms.custom: template-how-to
---

# Associate a virtual machine scale set with flexible orchestration to a capacity reservation group

**Applies to:** :heavy_check_mark: Flexible scale sets

Virtual machine scale sets have two modes:

- **Uniform orchestration**: In this mode, virtual machine scale sets use a virtual machine (VM) profile or a template to scale up to the capacity you want. Although there's some ability to manage or customize individual VM instances, uniform orchestration uses identical VM instances. These instances are exposed through the virtual machine scale set's VM APIs and aren't compatible with the standard Azure infrastructure as a service (IaaS) VM API commands. Because the scale set performs all the actual VM operations, reservations are associated with the virtual machine scale set directly. After the scale set is associated with the reservation, all the subsequent VM allocations are done against the reservation.
- **Flexible orchestration**: In this mode, you get more flexibility to manage the individual virtual machine scale set VM instances. They can use the standard Azure IaaS VM APIs instead of using the scale set interface. To use reservations with flexible orchestration mode, define both the virtual machine scale set property and the capacity reservation property on each VM.

To learn more about these modes, see [Virtual machine scale set orchestration modes](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).

This content applies to the flexible orchestration mode. For more information about uniform orchestration mode, see [Associate a virtual machine scale set with uniform orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).

## Associate a new virtual machine scale set to a capacity reservation group

You have two options:

- **Option 1: Add to the VM profile:** If the scale set with flexible orchestration includes a VM profile, add the capacity reservation group property to the profile during scale set creation. Follow the same process used for a scale set by using uniform orchestration. For sample code, see [Associate a virtual machine scale set with uniform orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).
- **Option 2: Add to the first VM deployed:** If the scale set omits a VM profile, you must add the capacity reservation group to the first VM deployed by using the virtual machine scale set. Follow the same process used to associate a VM. For sample code, see [Associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md).

## Associate an existing virtual machine scale set to a capacity reservation group

There are two steps:

- **Add to the virtual machine scale set:** For sample code, see [Associate a virtual machine scale set with uniform orchestration to a capacity reservation group](capacity-reservation-associate-virtual-machine-scale-set.md).
- **Add to the VMs deployed:** You must add the capacity reservation group to the VMs deployed by using the scale set depending on the upgrade mode. Follow the same process used to associate a VM. For sample code, see [Associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md).

## Next steps

> [!div class="nextstepaction"]
> [Learn how to remove a scale set association from a capacity reservation](capacity-reservation-remove-virtual-machine-scale-set.md)
