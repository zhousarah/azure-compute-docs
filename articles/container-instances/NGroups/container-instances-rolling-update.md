---
title: NGroups Rolling Update - Customer Documentation
description: Introduction to NGroups Rolling Update
ms.author: shivg
author: shivg
ms.service: azure-container-instances
ms.custom: devx-track-azurecli
services: container-instances
ms.topic: how-to
ms.date: 11/07/2024
---

# NGroups Rolling Update

## Introduction

As requirements change, we would need to keep updating our NGroups and its CGs. 

There are two update modes with which we can update a NGroups – **Manual** (default option) and **Rolling**.  

Within **Rolling Update (RU)** too, there are 2 options – ‘*in-place*’ update and ‘*replace*’ update. Replace RU is the default option. 

This document describes RU in detail. Manual update is covered in the NGroups documentation link below.

## Feature Description

Consider a simple example of updating a CG profile reference from *cgprofile1* to *cgprofile2*.


### In-place rolling update
With in-place RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, "*existing CGs*" are updated with *cgprofile2*. The update to existing CGs happen in small batches (and not all at once). This ensures that there is a minimal impact on our workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because in-place RU updates existing CGs, there are certain limitations to the CG properties that ACI enforces.  

Please see ACI’s [limitations](../container-instances-update.md) w.r.t updating CGs. Also, please see here for [properties](../container-instances-update.md) on CGs that require a delete (versus an update).

### Replace rolling update
With replace RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, "*new CGs*" are created with *cgprofile2*. Existing CGs with *cgprofile1* are deleted. This creation and deletion happen in small batches (and not all at once). This ensures that there is a minimal impact on our workload since only a small percentage of CGs are impacted during the update.  

Like in-place RU, we can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because replace RU creates new CGs, there are fewer limitations enforced by ACI. Hence this is the more powerful option. Consequently, this is the default option when a customer selects RU. 

## Usage

### Triggering a rolling update

Rolling update is triggered when a NGroups PUT call is made and the CG profile in the PUT call is different from the CG profile currently referenced in the NGroups.  

NGroups will then automatically group instances into batches and update one batch at a time. The size of the batch is determined by *maxBatchPercent*.  

**Updating a batch:**

 - With **in-place** update involves - Invoking a CG PUT call to ‘update’ each CG of the batch 

- With **replace** update involves – Invoking a CG PUT call to create new CGs and delete existing CGs of the batch. There exists a 1:1 correspondence with the CGs being created and the CGs being deleted. The CG names are however different. 

If “enough number of CGs” from the batch gives healthy signals after *pauseTimeBetweenBatches* period, then NGroups will automatically start the next batch for update, else it will stop the rollout. This “enough number of CGs” can be specified by *maxUnhealthyPercent* and *maxUnhealthyUpdatedPercent*. 

To issue a RU, specify the following in the request and issue an UPDATE NGroups 

```
{ 
    "location": "{{location}}", 
    "properties": { 
        "updateProfile": { 
            "updateMode": "Rolling", 

            "rollingUpdateProfile": { 
                // Maximum percentage of total CGs which can be updated  
                // simultaneously by rolling update in one batch. 
                “maxBatchPercent”: “10”, // optional, defaults to 20% 

                // Maximum percentage of the total CGs across the whole NGroup  
                // that can be unhealthy at a time either by rolling update or health 
                // checks by liveness probes. If unhealthy CGs are more than this number,  
                // current rolling update is marked as failed. 
                // This check will be prerequisite to start any batch. 
                “maxUnhealthyPercent”: “10”, // optional, defaults to 20% 

                // Maximum percentage of the updated CGs which can be in unhealthy state  
                // after each batch is updated. If unhealthy CGs are more than this number,  
                // current rolling update is marked as failed. 
                “maxUnhealthyUpdatedPercent”: 10, // optional, defaults to 20% 

                // The wait time between batches after completing the one batch of the  
                // rolling update and starting the next batch. The time duration should  
                // be specified in ISO 8601 format for duration. 
                "pauseTimeBetweenBatches": "PT2M", // optional, defaults to PT1M 

                // A nullable boolean property. Default is null 
                // Sets the mode to either in-place RU or replace (default) RU. 
                “inPlaceUpdate”: null/false/true 
            } 
        }, 
        "containerGroupProfiles": [
            { 
                "resource": { 
                    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/cgp1" 
                } 
            } 
        ] 
    } 
} 
```

If image version is set to “latest” tag for container images within the CG profile, then NGroups will automatically pick up the latest image version during the RU. To prevent application from misbehaving, it is recommended to not use the “latest” tag for images. Instead, use specific versions.  


> [!NOTE]
> Since replace rolling update was introduced recently hence to apply this flighting change customer need to provide a tag in order to use replace rolling update. In order to use replace RU, set the tag "rollingupdate.replace.enabled: true" for the NGroups resource. 
> This tag is however temporary and will be removed after a few months.
```dotnetcli
“tags”: { 
    “rollingupdate.replace.enabled”: true 
} 
```

### Getting status of running RU

For getting latest status of your rolling update, you can use this REST API:
```dotnetcli
GET 

/subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/latestRollingUpdate 
```

This returns a [RollingUpdate structure](#api-model) containing relevant information about the RU 

### Cancelling a RU 

To cancel a RU, use the following API. Once cancelled, the RU cannot be resumed/restarted. A new RU needs to be triggered. NGroups prefers this model to keep the design simple. 
```azurepowershell
POST 

/subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/cancelRollingUpdate 
{ 
    // There is no resource body 
} 
```

### Boundry of a batch in RU
The CGs of a specific batch in a RU will not cross a fault model boundary. Here, a fault model represents a zone/fault-domain combination. E.g., zone 1 / FD 0 is a fault model boundary. Similarly, zone 1 / FD 1 is another fault model boundary. Similarly, zone 2 / FD 0. 

E.g., if a customer has a multi-zonal NGroups with say, 3 zones, then a batch will be confined to CGs belonging to just 1 zone at max. A batch will never have CGs spread across multiple zones. 

Similarly, if a customer has a multi-zonal, multi-FD NGroups with say, 3 zones and say, 5 FDs in each zone, then too a batch will be confined to CGs belonging to just 1 FD of a single zone at max.  

NGroups maintains this boundary of a fault model in a batch even though the no. of CGs it selects in a batch is much less than the maxBatchPercent setting. i.e., NGroups prefers safe updates over faster (and hence riskier) updates. 

The only time a fault model boundary is crossed is when RU selects unhealthy CGs as part of the 1st batch. In this 1st batch, RU will try to update all unhealthy CGs (to improve overall availability of the NGroups). Hence, when updating unhealthy CGs, RU can exceed the maxBatchPercent setting. 


### API Model

```dotnetcli
/// <summary> 
/// The object representing a rolling update. 
/// This will be returned to the customer when they request for the current  
/// rolling update status info. 
/// The base class ‘Resource’ is just an ARM resource. 
/// </summary> 
public class RollingUpdate : Resource 
{ 
    public Properties properties; 

    public class Properties 
    { 
        public RollingUpdateProfile profile { get; set; } 
        public RollingUpdateProgress progress { get; set; } 

        /// <summary> 
        /// Last action which was taken on this rolling update.  
        /// </summary> 
        public RollingUpdateAction lastAction { get; set; } 

        /// <summary> 
        /// Last action (start/cancel) timestamp. 
        /// </summary> 
        public DateTimeOffset lastActionTime { get; set; } 
    } 
} 

public enum RollingUpdateAction 
{ 
    /// <summary> 
    /// Ideally this action should never be set. 
    /// </summary> 
        Unknown = 0, 

    /// <summary> 
    /// Rolling update is started when Update mode is set to rolling and customer  
    /// try to update the Container Groups by Put/Patch call. 
    /// </summary> 
    Start, 

    /// <summary> 
    /// Permanently cancel the ongoing rolling update. 
    /// </summary> 
    Cancel 
} 

/// <summary> 
/// This entity contains the information about the current state of the  
/// overall rolling update operation. 
/// </summary> 
public class RollingUpdateProgress 
{ 
    public ContainerGroupsUpdateSummary containerGroupsUpdateSummary { get; set; } 

    /// <summary> 
    /// Current status of the ongoing rolling update.  
    /// </summary> 
    public RollingUpdateState state { get; set; } 

    /// <summary> 
    /// Rolling update start timestamp 
    /// </summary> 
    public DateTimeOffset startTime { get; set; } 

    /// <summary> 
    /// Timestamp when this update reached terminal state. 
    /// </summary> 
    public DateTimeOffset? finishTime { get; set; } 

    /// <summary> 
    /// Error details in the latest rolling update if any. 
    /// </summary> 
    public ApiError error { get; set; } 

    /// <summary> 
    /// This contains the information about the Container Groups in each  
    /// state succeeded/failed/pending. 
    /// </summary> 
    public class ContainerGroupsUpdateSummary 
    { 
        public int succeededCount { get; set; } 
        public int failedCount { get; set; }             
        public int pendingCount { get; set; } 
    } 
} 
```
