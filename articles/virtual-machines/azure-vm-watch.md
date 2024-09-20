---
title: Azure VM Watch
description: Learn how to use Azure VM Watch service offering
author:      iamwilliew 
ms.author:   wwilliams
ms.service: azure-virtual-machines
ms.topic: conceptual
ms.date:     09/20/2024
---

# Azure VM watch - Enhancing VM Health Monitoring

Azure VM watch is a standardized, lightweight, and adaptable in-VM service offering a uniform data model for collecting and transmitting health signals from virtual machines and scale sets. It is integrated into Application Health (link) to provide ease of use and manageability for customers. In addition, VM watch is offered at no additional cost for customers. Some key benefits of VM watch include: 

- Improve overall customer experience on the platform
- Improve outage detection and recovery times 

## VM watch Monitoring Specifics

- **Seamlessly Integrated**: VM watch is made available through the Application Health Extension (link) 
- **Compatibility:** VM watch operates seamlessly on both Linux and Windows environments. Also, VM watch is suitable for individual VMs and VMSS VMs alike.
- **Flexible Deployment:** Users can enable VM Watch with ease via ARM template, PowerShell, or AZ CLI
- **Resource Governance:** VM Watch provides efficient monitoring without impacting system performance.
- **Ready Out-of-the-Box**: VM Watch comes equipped with a suite of default tests which are easily configurable to enable scenario specific tests. Detailed information regarding the Tests (Checks, Metrics, and Event Logs) are given below.

  

### Network:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Outbound connectivity** | Check | Verify the network outbound connectivity from the Azure VM. |
| **DNS Resolution** | Check | Verify if the dns name(s) can be resolved. |
| **SegmentsRetransimitted** | Metric | The number of TCP segments transmitted containing one or more previously transmitted octets. |
| **NormalizedSegmentsRetransmitted** | Metric | SegmentsRetransimitted / (SegmentsSent + SegmentsReceived) |
| **ConnectionResets** | Metric | Number of times TCP connections have made a direct transition to the CLOSED state from either the ESTABLISHED state or the CLOSE_WAIT state |
| **NormalizedConnectionResets** | Metric | ConnectionResets / CurrentConnections |
| **FailedConnectionAttempts** | Metric | Number of times TCP connections have made a direct transition to the CLOSED state from either the SYN_SENT state or the SYN_RCVD state. |
| **NormalizedFailedConnectionAttempts** | Metric | FailedConnectionAttempts / (ActiveConnectionOpenings + PassiveConnectionOpenings) |
| **ActiveConnectionOpenings** | Metric | Number of times TCP connections have made a direct transition to the SYN_SENT state from the CLOSED state |
| **PassiveConnectionOpenings** | Metric | Number of times TCP connections have made a direct transition to the SYN_RCVD state from the LISTEN state |
| **CurrentConnections** | Metric | Number of connections established |
| **SegmentsReceived** | Metric | Number of segments received, including those received in error |
| **SegmentsSent** | Metric | Number of segments sent, including those on current connections but excluding those containing only retransmitted octets |

 

### Disk

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Azure Disk I/O** | Check | Verify file creation, write, read, delete operations on each drive mounted to the VM |
| **FreeSpaceInBytes** | Metric | The free disk space of the target mount point |
| **UsedSpaceInBytes** | Metric | The used disk space of the target mount point |
| **CapacityInBytes**  | Metric | The disk spce capacity of the target mount point |
| **UsedPercent**      | Metric | The used disk space percentageof the target mount point |
| **WriteOps**         | Metric | The write operations per seconds of the target disk/partition |
| **ReadOps**          | Metric | The read operations per seconds of the target disk/partition | 


### CPU:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **ProcessCoreUsage** | Metric | An instantaneous measurement of the percentage of a single CPU core that the target process is using (100 = 100%, a whole core) |
| **ProcessMachineUsage** | Metric | The percentage of the machine's total CPU that this process is using |
| **MachineTotalCpuUsage** | Metric | The VM's total instantaneous CPU utilization |

 

### Process:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Process Creation** | Check | Starts a lightweight process to validate that process creation is possible |
| **Running Process(es)** | Check | Verify if the target process(es) are running |
| **UpTime** | Metric | How long the target process has been up and running since last process startup |

 

### IMDS:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **IMDS** | Check | Verify user can reach of IMDS endpoint from within the VM and VM information is returned from the IMDS endpoint query |

 

### Clock:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Clock Skew** | Check | Verify the clock skew between remote NTP server and the Azure VM. For Windows VM, fallback to check if Windows Time Service is synced with w32tm if remote NTP server is inaccessible |

 

### AzBlob:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Azure Storage blob connectivity** | Check | Verify the connectivity to the Azure Storage Blob and download the Blob with MSI or SAS token. |

### Hardware:

| **Signal Name** | **Type** | **Description** |
|:---:|:---:|:---:|
| **Hardware Health Monitor** | EventLog | Collect hardware health info from Windows event log, currently only disk related critical events are collected, including events with id 7, 500, 504, 505, 512 and 549 |

### Next Steps:

To Learn more about VM watch, proceed to Get Started Article

Get Started installing VM watch to a VM


