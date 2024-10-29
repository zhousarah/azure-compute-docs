---
title: Cobalt Virtual Machines
description: An overview of Cobalt virtual machines. 
author: archat
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 10/29/2024
ms.author: archatC
---

# Cobalt Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Cobalt 100-based Virtual Machines (VMs) are powered by Azure's first-generation Cobalt 100 processor, delivering outstanding performance for both general-purpose and memory-optimized workloads. These VMs utilize Microsoft's first 64-bit Arm-based Azure Cobalt 100 CPU, which has been fully designed in-house. The Cobalt 100 processor, based on the Arm Neoverse N2 design, aims to enhance performance and power efficiency for various workloads. Operating at 3.4 GHz, the Azure Cobalt 100 processor provides an entire physical core for each virtual machine vCPU. 

For more information, check out the [Azure Kubernetes Service (AKS) demo on Cobalt 100 VMs](https://aka.ms/C100-VM-deploy-demo), which showcases how to leverage these VMs for cloud-native applications. 

## List of VM size families by type
The following section lists Cobalt 100-based VM series supported by Cobalt 100 processors. To find detailed information on each size in that series, including summeries and recommended workloads, check out the links in the *Series* column. 

### Cobalt 100-based VMs
| Family | Series | Regions |
|----|---|---|
| General Purpose  | [Dpsv6-series](./general-purpose/dpsv6-series.md), [Dpdsv6-series](./general-purpose/dpdsv6-series.md), [Dplsv6-series](./general-purpose/dplsv6-series.md), [Dpldsv6-series](./general-purpose/dpldsv6-series.md) | Canada Central, Central US, East US 2, East US, Germany West Central, Japan East, Mexico Central, North Europe, Southeast Asia, Sweden Central, Switzerland North, UAE North, West Europe, and West US 2. Australia East, Brazil South, France Central, India Central, South Central US, UK South, West US 3 |
| Memory Optimized | [Epsv6-series](./memory-optimized/epsv6-series.md), [Epdsv6-series](./memory-optimized/epdsv6-series.md) | Canada Central, Central US, East US 2, East US, Germany West Central, Japan East, Mexico Central, North Europe, Southeast Asia, Sweden Central, Switzerland North, UAE North, West Europe, and West US 2. Australia East, Brazil South, France Central, India Central, South Central US, UK South, West US 3 |


[!INCLUDE [sizes-footer](./includes/sizes-footer.md)]
