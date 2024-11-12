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

As requirements change, we would need to keep updating our NGroups and its container groups (CGs). 

There are two update modes with which we can update an NGroups – **Manual** (default option) and **Rolling**.  

Within **Rolling Update (RU)** too, there are 2 options – [*in-place*](#in-place-rolling-update) update and [*replace*](#replace-rolling-update) update. Replace RU is the default option. 

This document describes RU in detail. Manual update is covered in the NGroups documentation link [here](container-instances-about-ngroups.md).

## Feature Description

Consider a basic example of updating a CG profile reference from *cgprofile1* to *cgprofile2*.


### In-place Rolling Update
With in-place RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, "*existing CGs*" are updated with *cgprofile2*. The update to existing CGs happens in small batches (and not all at once). It ensures that there's a minimal impact on our workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because in-place RU updates existing CGs, there are certain limitations to the CG properties that Azure Container Instances (ACI) enforces.  

See ACI’s [limitations](../container-instances-update.md) w.r.t updating CGs. Also, see here for [properties](../container-instances-update.md) on CGs that require a delete (versus an update).

### Replace Rolling Update
With replace RU, when we update the reference to *cgprofile2* and issue an UPDATE NGroups command, "*new CGs*" are created with *cgprofile2*. Existing CGs with *cgprofile1* are deleted. This creation and deletion happen in small batches (and not all at once). It ensures that there's a minimal impact on our workload since only a small percentage of CGs is impacted during the update.  

Like in-place RU, we can configure the batch size and other related rolling update mode settings with the NGroups API.  

Because replace RU creates new CGs, there are fewer limitations enforced by ACI. Hence replace RU is a more powerful option and it's the default option when a customer selects RU. 

## Usage

### Triggering a Rolling Update

Rolling update is triggered when an NGroups PUT call is made and the CG profile in the PUT call is different from the CG profile currently referenced in the NGroups.  

NGroups then automatically group instances into batches and update one batch at a time. *maxBatchPercent* parameter determines the size of the batch. 

##### Updating a Batch

 - With **in-place** update involves - Invoking a CG PUT call to update each CG of the batch 

- With **replace** update involves – Invoking a CG PUT call to create new CGs and delete existing CGs of the batch. There exists a 1:1 correspondence with the CGs being created and the CGs being deleted. The CG names are however different. 

If a sufficient number of CGs in the batch provide healthy signals after the pauseTimeBetweenBatches period, NGroups automatically starts the next batch for the update. Otherwise, it stops the rollout. The *maxUnhealthyPercent* parameter specifies the total number of unhealthy CGs, while the *maxUnhealthyUpdatedPercent* parameter specifies the total number of unhealthy CGs after the update.

Here's an example to issue a rolling update request to NGroups: 

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

If image version is set to **latest** tag for container images within the CG profile, then NGroups automatically picks up the latest image version during the RU. To prevent application from misbehaving, it's recommended to not use the *latest* tag for images. Instead, use specific versions.  


> [!NOTE]
> Since replace rolling update was introduced recently hence to apply this flighting change customer need to provide a tag in order to use replace rolling update. In order to use replace RU, set the tag "rollingupdate.replace.enabled: true" for the NGroups resource. 
> This tag is however temporary and will be removed after a few months.
```dotnetcli
“tags”: { 
    “rollingupdate.replace.enabled”: true 
} 
```

### Getting Status of a Running Rolling Update

For getting latest status of your rolling update, you can use this REST API:
```dotnetcli
GET 

/subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/latestRollingUpdate 
```

This returns a [RollingUpdate structure](#api-model) containing relevant information about the RU 

### Cancelling a Rolling Update 

To cancel an RU, use the following API. Once canceled, the RU can't be resumed/restarted. A new RU needs to be triggered. NGroups prefers this model to keep the design simple. 
```azurepowershell
POST 

/subscriptions/{subscriptionId}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/NGroups/{{ngroupsName}}/cancelRollingUpdate 
{ 
    // There is no resource body 
} 
```

### Boundary of a Batch in a Rolling Update

The CGs of a specific batch in an RU don't cross a fault model boundary. A fault model represents a zone/fault-domain (FD) combination. For example, zone 1 / FD 0 is a fault model boundary, zone 1 / FD 1 is another fault model boundary, and zone 2 / FD 0 is yet another fault model boundary.

If a customer has a multi-zonal NGroups set up with three zones, a batch is confined to CGs belonging to only one zone at most. A batch never consists of CGs spread across multiple zones.

If a customer has a multi-zonal and multi-FD NGroups setup, a batch still consists of CGs belonging to only one FD in a single zone at most.

NGroups maintains this fault model boundary in a batch, even when the number of CGs selected for a batch is much less than the maxBatchPercent setting. It reflects that NGroups prefers safe updates over faster (and thus riskier) updates.

The only time a fault model boundary is crossed when the RU selects unhealthy CGs for the first batch. In this batch, the RU attempts to update all unhealthy CGs to improve the overall availability of NGroups. As a result, when updating unhealthy CGs, the RU may exceed the maxBatchPercent setting.


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
