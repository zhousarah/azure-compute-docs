---
title: Azure Impact Reporting - Overview #Required; page title is displayed in search results. Include the brand.
description: An overview of Azure Impact Reporting - a service that enables you to send reports when you observe a negative change in performance of your Azure workloads. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.service: azure-impact-reporting #Required; use either service or product per approved list. 
ms.topic: overview #Required; leave this attribute/value as-is.
ms.date: 11/18/2022 #Required; mm/dd/yyyy format.
ms.custom: template-overview #Required; leave this attribute/value as-is.
---

# What is Impact Reporting? (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in PREVIEW. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Azure Impact Reporting enables you to report issues with performance, connectivity, and availability impacting your Azure workloads directly to Microsoft. It is an additional tool you can leverage and a quicker way to let us know that something might be wrong. These reports are used by Azure internal systems to aid in quality improvements and regression detection.

## What is an "Impact"?

In this context, an impact is any unexpected behavior or issue negatively affecting your workloads that has been root caused to the Azure platform.

Examples of impacts include:

* Performance Impact: Your application's performance degrades suddenly, you investigate and realize that database writes to your IaaS SQL virtual machine are unusually slow.
* Connectivity Impact: You're not able to successfully write to blob store despite having the right permissions
* Availability: Your Azure virtual machine unexpectedly reboots

## Next steps
<!-- Add a context sentence for the following links -->
* [File an impact report](ReportVMImpact.md)
<!-- - [View previous impact reports](links-how-to.md) -->
