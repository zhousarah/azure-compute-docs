---
title: Get Impact Categories - Azure Impact Reporting #Required; 
description: How to get a valid list of impact cetegories. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.topic: overview #Required; leave this attribute/value as-is.
ms.service: azure-impact-reporting #Required; use either service or product per approved list. 
ms.date: 10/30/2022 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
---

# Impact Categories (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in PREVIEW. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Please review our full list of categories in our [REST API reference](https://aka.ms/ImpactRP/APIDocs).

## Category list

|**Category Name**|**Problem Description**|
|----------------------------------|------------------------------------------------------------------------------------------------------------------------|
|ARMOperation.CreateOrUpdate|Use this to report problems related to creating a new azure virtual machines such as provisioning or allocation failures|
|ARMOperation.Delete|Use this to report failures in deleting a resource.|
|ARMOperation.Get|Use this to report failures in querying resource metadata.|
|ARMOperation.Start|Use this to report failures in starting a resource.|
|ARMOperation.Stop|Use this to report failures in stopping a resource.|
|ARMOperation.Other|Use this to report Control Plane operation failures that don’t fall into other ARMOperation categories.|
|Resource.Performance|Use this to report general performance issues. For example, as high usage of CPU, IOPs, disk space, or memory|
|Resource.Performance.Network|Use this to report performance issues which are networking related. For example, degraded network throughput.|
|Resource.Performance.Disk|Use this to report performance issues which are disk related. For example, degraded IOPs|
|Resource.Performance.CPU|Use this to report performance issues which are CPU related.|
|Resource.Performance.Other|Use this to report issues that don’t fall under other Resource.Performance sub-categories.|
|Resource.Connectivity|Use this to report general connectivity issues to or from a resource.|
|Resource.Connectivity.Inbound|Use this to report inbound connectivity issues to a resource.|
|Resource.Connectivity.Outbound|Use this to report outbound connectivity issues from a resource.|
|Resource.Connectivity.Other|Use this to report issues that don’t fall into under other Resource.Connectivity sub-categories|
|Resource.Availability|Use this to report general unavailability issues|
|Resource.Availability.Restart|Use this to report if an unexpected virtual machine restarts|
|Resource.Availability.Boot|Use this to report virtual machines which are in a non-bootable state, not booting at all or is on a reboot loop|
|Resource.Availability.Disk|Use this to report availability issues related to disk|
|Resource.Availability.UnResponsive|Use this to report a resource that is not responsive now or for a period of time in the past|
|Resource.Availability.Storage|Use this to report availability issues related to storage.|
|Resource.Availability.Network|Use this to report network availability issues.|
|Resource.Availability.DNS|Use this to report DNS availability issues.|
|Resource.Availability.Other|Use this to report issues that don’t fall into under other Resource.Availability sub-categories|


## Next steps

> [!div class="nextstepaction"]
> [View previously filed impact reports](ViewImpactInsights.md)