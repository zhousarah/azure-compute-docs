---
 title: include file
 description: include file
 services: virtual-machines
 author: roygara
 ms.service: azure-virtual-machines
 ms.topic: include
 ms.date: 08/26/2024
 ms.author: rogarana
 ms.custom: include file
---    
- Only supported for data disks.
- If a disk is 4 TiB or less, you should deallocate your VM and detach the disk before expanding it beyond 4 TiB. If a disk is already greater than 4 TiB, you can expand it without deallocating the VM and detaching the disk.
- Not supported for Ultra disks or Premium SSD v2 disks.
- Not supported for shared disks.
- Install and use either:
    - The [latest Azure CLI](/cli/azure/install-azure-cli)
    - The [latest Azure PowerShell module](/powershell/azure/install-azure-powershell)
    - The [Azure portal](https://portal.azure.com/)
    - Or an Azure Resource Manager template with an API version that's `2021-04-01` or newer.
- Not available on some classic VMs. Use [this script](#expanding-without-downtime-classic-vm-sku-support) to get a list of classic VM SKUs that support expanding without downtime.
- Ultra Disks and Premium SSD v2 disks have preview support for expanding without downtime.

### Preview for Ultra Disk and Premium SSD v2

Currently, you can sign up for a public preview for expanding Ultra Disks and Premium SSD v2 disks without downtime.

The preview has the following additional limitations:
- You can't expand a disk while a background copy of data is also occurring on the disk.
- You can't currently expand a VM using NVMe controllers.
- You can't currently expand a shared Ultra Disk or Premium SSD v2.