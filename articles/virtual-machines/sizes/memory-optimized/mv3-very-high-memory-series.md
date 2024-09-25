---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: Mv3 Very High Memory Series
description: Overview of Mv3 Very High Memory Virtual Machines
author:      phgantik # GitHub alias
ms.author:   phgantik # Microsoft alias
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.date:     09/11/2024
---

# Mv3 Very High Memory Series

The Mv3 Very High Memory (VHM) series, powered by 4<sup>th</sup> generation Intel® Xeon® Scalable processors, are the next generation of memory-optimized VM sizes delivering faster performance, lower total cost of ownership and improved resilience to failures compared to previous generation Mv2 VMs. The Mv3 VHM sizes offer up to 32TB of memory and 8000 MBps throughput to remote storage and provides up to 25% networking performance improvements over previous VHM generations.

## Mv3 Very High Memory series

[Premium Storage](/azure/virtual-machines/premium-storage-performance): Supported<br>[Premium Storage caching](/azure/virtual-machines/premium-storage-performance): Supported<br>[Live Migration](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[Memory Preserving Updates](/azure/virtual-machines/maintenance-and-updates): Not Supported<br>[VM Generation Support](/azure/virtual-machines/generation-2): Generation 2<br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>[Ephemeral OS Disks](/azure/virtual-machines/ephemeral-os-disks): Supported<br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported<br>[Write Accelerator](/azure/virtual-machines/how-to-enable-write-accelerator): Supported

## Mv3 Very High Memory series

| **Size**<sup>1</sup> | **vCPU**<sup>2</sup> | **Memory: GiB** | **Max Data Disks** | **Max temp storage throughput: IOPS/MBps*** | **Max un-cached Premium** **SSD  throughput: IOPS/MBps** | **Max un-cached Ultra Disk and Premium SSD V2 disk throughput: IOPS/MBps** | **NIC's (max)** | **Max network bandwidth (Mbps)** <sup>3</sup> |
|---|---|---|---|---|---|---|---|---|
| **standard_m896ixds_32_v3**    | 896 | 30400 | 64 | 4,096 | 110,000/8,000 | 200000/8,000 | 8 | 185,000 |
| **standard_m1792ixds_32_v3**    | 1792 | 30400 | 64 | 4,096 | 110,000/8,000 | 200,000/8,000 | 8 | 185,000 |

<sup>1</sup>VHM VM Sizes are virtual machine sizes that are Isolated to a specific hardware type and dedicated to a single customer

<sup>2</sup>Full Cores

<sup>3</sup>Note that Bandwidth is shared between Storage and Network


## Size table definitions

Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.

Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.

Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to **ReadOnly** or **ReadWrite**. For uncached data disk operation, the host cache mode is set to **None**.

To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](/azure/virtual-machines/disks-performance).

**Expected network bandwidth** is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput).

Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

## Other sizes and information

[General purpose](/azure/virtual-machines/sizes-general)

[Memory optimized](/azure/virtual-machines/sizes-memory)

[Storage optimized](/azure/virtual-machines/sizes-storage)

[GPU optimized](/azure/virtual-machines/sizes-gpu)

[High performance compute](/azure/virtual-machines/sizes-hpc)

[Previous generations](/azure/virtual-machines/sizes-previous-gen)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

More information on Disks Types: [Disk Types](/azure/virtual-machines/disks-types)
