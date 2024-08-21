---
title: include file
description: include file
author: roygara
ms.service: azure-disk-storage
ms.topic: include
ms.date: 08/15/2024
ms.author: rogarana
ms.custom: include file
---

The following list contains Ultra Disk's limitations:
- Ultra Disks can't be used as an OS disk.
- Ultra Disks can't be used with Azure Compute Gallery.
- Currently, Ultra Disks only support Single VM and Availability zone infrastructure options.
- Ultra Disks don't support availability sets.
- The size of an Ultra Disk can't be expanded without either deallocating the VM or detaching the Ultra Disk.
- Existing disks currently can't change their type to an Ultra Disk. They must be [migrated](../disks-convert-types.md#migrate-to-premium-ssd-v2-or-ultra-disk-using-snapshots).
- Currently, Azure Government and Azure China don't support [customer-managed keys](../disk-encryption.md#customer-managed-keys) for Ultra disks.
- Azure Disk Encryption isn't supported for VMs with Ultra Disks. Instead, you should use encryption at rest with platform-managed or customer-managed keys.
- Azure Site Recovery isn't supported for VMs with Ultra Disks.
- Ultra Disks don't support disk caching.
- Snapshots are supported with [other limitations](../disks-incremental-snapshots.md#incremental-snapshots-of-premium-ssd-v2-and-ultra-disks).
- Azure Backup support for VMs with Ultra Disks is [generally available](/azure/backup/backup-support-matrix-iaas#vm-storage-support). Azure Backup has limitations when using Ultra Disks, see [VM storage support](/azure/backup/backup-support-matrix-iaas#vm-storage-support) for details.

Ultra Disks support a 4k physical sector size by default but also supports a 512E sector size. Most applications are compatible with 4k sector sizes, but some require 512-byte sector sizes. Oracle Database, for example, requires release 12.2 or later in order to support 4k native disks. For older versions of Oracle DB, 512-byte sector size is required.

The following table outlines the regions Ultra Disks are available in, and their corresponding availability options.

> [!NOTE]
> If a region in the following list lacks availability zones that support Ultra disks, then a VM in that region must be deployed without infrastructure redundancy to attach an Ultra Disk.

| Redundancy options | Regions |
|--------------------|---------|
| **Single VMs** | Australia Central<br/>Brazil South<br/>Brazil Southeast<br/>Canada East<br/>Central India<br/>East Asia<br/>Germany West Central<br/>Korea Central<br/>Korea South<br/>UK West <br/>North Central US, South Central US, West US<br/>US Gov Arizona, US Gov Texas, US Gov Virginia |
| **One availability zone** | UAE North |
| **Two availability zones** |  France Central <br/> Qatar Central <br/> South Africa North <br/> Switzerland North |
| **Three availability zones** | Australia East<br/>Canada Central<br/>China North 3 <br/>North Europe, West Europe<br/>Italy North<br/>Japan East<br/>Poland Central <br/>Southeast Asia<br/>Sweden Central<br/>UK South<br/>Central US, East US, East US 2, West US 2, West US 3 |

Not every VM size is available in every supported region with Ultra Disks. The following table lists VM series that are compatible with Ultra Disks.

|VM Type     |Sizes    |Description  |
|------------|---------|-------------|
| General purpose|[DSv3-series](../dv3-dsv3-series.md#dsv3-series), [Ddsv4-series](../ddv4-ddsv4-series.md#ddsv4-series), [Dsv4-series](../dv4-dsv4-series.md#dsv4-series), [Dasv4-series](../dav4-dasv4-series.md#dasv4-series), [Dsv5-series](../dv5-dsv5-series.md#dsv5-series), [Ddsv5-series](../ddv5-ddsv5-series.md#ddsv5-series), [Dasv5-series](../dasv5-dadsv5-series.md#dasv5-series)| Balanced CPU-to-memory ratio. Ideal for testing and development, small to medium databases, and low to medium traffic web servers.|
| Compute optimized|[FSv2-series](../fsv2-series.md)| High CPU-to-memory ratio. Good for medium traffic web servers, network appliances, batch processes, and application servers.|
| Memory optimized|[ESv3-series](../ev3-esv3-series.md#esv3-series), [Easv4-series](../eav4-easv4-series.md#easv4-series), [Edsv4-series](../edv4-edsv4-series.md#edsv4-series), [Esv4-series](../ev4-esv4-series.md#esv4-series), [Esv5-series](../ev5-esv5-series.md#esv5-series), [Edsv5-series](../edv5-edsv5-series.md#edsv5-series), [Easv5-series](../easv5-eadsv5-series.md#easv5-series), [Ebsv5 series](../ebdsv5-ebsv5-series.md#ebsv5-series), [Ebdsv5 series](../ebdsv5-ebsv5-series.md#ebdsv5-series), [M-series](../m-series.md), [Mv2-series](../mv2-series.md), [Msv2/Mdsv2-series](../msv2-mdsv2-series.md)|High memory-to-CPU ratio. Great for relational database servers, medium to large caches, and in-memory analytics.
| Storage optimized|[LSv2-series](../lsv2-series.md), [Lsv3-series](../lsv3-series.md), [Lasv3-series](../lasv3-series.md)|High disk throughput and IO ideal for Big Data, SQL, NoSQL databases, data warehousing, and large transactional databases.|
| GPU optimized|[NCv2-series](../ncv2-series.md), [NCv3-series](../ncv3-series.md), [NCasT4_v3-series](../nct4-v3-series.md), [ND-series](../nd-series.md), [NDv2-series](../ndv2-series.md), [NVv3-series](../nvv3-series.md), [NVv4-series](../nvv4-series.md), [NVadsA10 v5-series](../nva10v5-series.md)| Specialized virtual machines targeted for heavy graphic rendering and video editing, as well as model training and inferencing (ND) with deep learning. Available with single or multiple GPUs.|
| <nobr>Performance optimized</nobr> |[HB-series](../hb-series.md), [HC-series](../hc-series.md), [HBv2-series](../hbv2-series.md)|The fastest and most powerful CPU virtual machines with optional high-throughput network interfaces (RDMA).|
