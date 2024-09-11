---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 09/06/2024
 ms.author: rogarana
 ms.custom: include file, references_regions
---    
- Only supported for data disks.
- If a Standard HDD, Standard SSD, or Premium SSD disk is 4 TiB or less, deallocate your VM and detach the disk before expanding it beyond 4 TiB. If one of those disk types is already greater than 4 TiB, you can expand it without deallocating the VM and detaching the disk. This doesn't apply to Premium SSD v2 or Ultra Disks.
- Not supported for shared disks.
- Install and use either:
    - The [latest Azure CLI](/cli/azure/install-azure-cli)
    - The [latest Azure PowerShell module](/powershell/azure/install-azure-powershell)
    - The [Azure portal](https://portal.azure.com/)
    - Or an Azure Resource Manager template with an API version that's `2021-04-01` or newer.
- Not available on some classic VMs. Use [this script](#expanding-without-downtime-classic-vm-sku-support) to get a list of classic VM SKUs that support expanding without downtime.

### Preview for Ultra Disk and Premium SSD v2

 > [!IMPORTANT]
> Expanding without downtime for Ultra Disks and Premium SSD v2 disks is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](
https://azure.microsoft.com/support/legal/preview-supplemental-terms/)
for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Currently, you can sign up for a public preview for expanding Ultra Disks and Premium SSD v2 disks without downtimes [here.](https://aka.ms/DDLiveResizeCustomerSurvey)

The preview has the following additional limitations:
- You can't expand a disk while a [background copy](../scripts/create-managed-disk-from-snapshot.md#performance-impact---background-copy-process) of data is also occurring on that disk, like when a disk is being hydrated from snapshots.
- You can't expand a VM that's using [NVMe controllers](../nvme-overview.md) for Ultra Disks or Premium SSD v2 disks without downtime.

#### Regional availability

Resizing Ultra Disks and Premium SSD v2 disks is currently available as a preview in the following regions:

- Australia East
- Brazil South
- Canada Central
- Central India
- Central US
- East US 2
- France Central
- Japan East
- Korea Central
- North Europe
- Southeast Asia