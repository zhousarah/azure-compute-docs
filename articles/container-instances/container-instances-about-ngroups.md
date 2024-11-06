<<<<<<< HEAD
---
title: Introduction to NGroups
description: Introduction to NGroups and its features describing how customers can manage N instances of containers with NGroup.
ms.author: shivg
author: shivg
ms.service: azure-container-instances
ms.custom: devx-track-azurecli
services: container-instances
ms.topic: how-to
ms.date: 11/05/2024
---

# About NGroups

Containers have become the standard for packaging, deploying, and managing cloud applications, and effectively managing these containers is just as crucial as running the apps themselves. Azure Container Instances (ACI) is a flexible and scalable serverless computing service that allows you to run containerized applications without managing infrastructure. NGroups is a powerful feature within ACI designed to make managing multiple Azure container groups easier, faster, and hassle-free. These new APIs, an extension of the Azure Container Instance (ACI) API set provides a way to manage multiple related container groups (CGs) together. 

NGroups APIs allow customers to manage a huge scale of container groups with advanced features such as maintaining multiple instances, rolling upgrades, high availability through Availability Zones (AZs), confidentiality, load balancing, zone rebalancing (Zone Any), and auto-scaling. The NGroups feature builds upon ACI ensuring container groups are secure, highly available, and supports the feature set of ACI.  
=======
## **Overview** 

When deploying containerized applications to the cloud, customers face a decision of managing their own infrastructure and orchestrator or choosing to deploy to a prescriptive serverless platform. Running large-scale applications on the cloud with containerized workloads is difficult due to a customer requiring expertise in infrastructure management and container grouping and management platforms. 

NGroups is an easy to use and manage at scale container offering that reduces customer’s infrastructure management responsibilities while still providing the flexibility to configure scaling rules, networking, and security policies. NGroups provides customers with an environment that is configured to run containerized applications at scale with promises such as Maintaining N instances, Rolling Upgrades, High Availability with Availability Zones (AZs), auto-scaling (coming soon), Confidentiality, Load Balancing setup, AZ Rebalancing (Zone Any) etc. Customers need not deploy VMs or orchestrate complex infrastructure to get their application up and running. 

To provide customers with the flexibility to configure NGroups to meet their organizational needs, NGroups integrates well with many Azure resources such as virtual networks, load balancers, Azure Managed Identity, storage accounts etc. This empowers customers to focus on their application and business logic, while Azure takes care of the rest of infrastructure management. 

For 1P customers using ACI, NGroups is the Azure Core Containers standards platform that offers capabilities to keep containers secure, highly available, and compliant with Azure standards. Onboarding to NGroups helps eliminate recurring bookkeeping of compliance checklists for these customers, with the platform natively applying all defined standards on NGroups resources. 

Examples of great workloads to run on top of NGroups include batch applications, CI/CD pipelines, ETL pipelines, and custom orchestrators that use Azure Container Instances (ACI) Container Groups as a unit of compute. 

 

ACI provides a single instance of a Hyper-V isolated version container group on demand. ACI does not provide concepts such as scaling, load balancing, rolling upgrades, spreading instances across availability zones out of the box. This is the best option for customers that simply want to deploy a container quickly, and do not need built-in support for these more advanced concepts. 

 

NGroups API are extension to ACI, which allows customers to quickly deploy a group of CGs and provides support for more complex scenarios. Customers can choose to use ACI if they have a single task, they want to run quickly without needing to consider scale. As their application needs grow, they can easily move their instance to a NGroups. ACI and NGroups will coexist analogous to the needs of customers using VMs with VMSS. 
>>>>>>> be37c7d2ae611c6ca81ec684500b5ac38635df00

## **Concepts** 

### Container Group Profile (CG Profile) 

Managing multiple container groups is a crucial part of running large cloud applications. As of today, in order to run multiple CGs i.e. container groups, customers need to provide relevant properties such as container images, the restart policy and other properties each time. This may result in duplicated effort and management overhead.  

To alleviate this concern, NGroups has introduced - Container Group Profile. The container group profile aka CGProfile serves as a “template” for creating container groups with same set of properties.  


Here are some of the common properties that can be specified in a container group profile: 

- osType. E.g., Linux, Windows 

- containers. Image name, memory, CPU etc. 

- restartPolicy 

- ipAddress protocol and internal port 

- shutdownGracePeriod 

- timeToLive 

And below is a sample CG profile:

```
{ 

    "location": "{{location}}", 

    "properties": { 

        "sku": "Standard", 

        "containers": [ 

            { 

                "name": "container1", 

                "properties": { 

                    "image": "nginx", 

                    "ports": [ 

                        { 

                            "protocol": "TCP", 

                            "port": 80 

                        } 

                    ], 

                    "resources": { 

                        "requests": { 

                            "memoryInGB": 2.0, 

                            "cpu": 1.0 

                        } 

                    } 

                } 

            } 

        ], 

        "restartPolicy": "Always", 

        "shutdownGracePeriod": "PT1H", 

        "ipAddress": { 

            "ports": [ 

                { 

                    "protocol": "TCP", 

                    "port": 80 

                } 

            ], 

            "type": "Public", 

            "autoGeneratedDomainNameLabelScope": "Unsecure" 

        }, 

        "timeToLive": "PT1H", 

        "osType": "Linux" 

    }     

}
```

### NGroups

NGroups resource provides a way to create and manage ‘n’ container groups with a rich set of operations. A NGroups resource will simply reference a container group profile resource and use that to create N instances of similar looking CGs. Within NGroups resource, customer can also specify other properties including but not limited to - number of CGs, update preferences (manual or rolling update), load balancers, subnets and other relevant properties which they want to associate with CGs under a NGroups resource. 

> [!NOTE]
> A CG profile needs to be created **before** creating a NGroups resource.Since the CG profile is an ARM resource, it has its own ARM APIs. A CG profile needs to be created **before** creating a NGroups resource.

**Benefits of referencing container group profile**


1. Container group profile is a separate resource from NGroups. Customers can create multiple NGroups that can refer to the same container group profile. This will also guarantee consistency across all the NGroups that refer to a single container group profile and avoids duplication. 

2. A single ACI CG could also be created from a CG profile. This allows us to rapidly move from prototype to production mode. 

    E.g., a customer creates a single CG using the CG profile. They work on this CG to get their container images working. Then they simply create a NGroups using the same CG profile to massively scale out. 

Here is a sample of a NGroup with managed identity and zones that refers to a container group profile and will create 3 container groups:

 
```
{ 

    "location": "{{location}}", 

    "properties": { 

        "elasticProfile": { 

            "desiredCount": 100 // specifies how many CGs to stamp out 

        }, 

        "containerGroupProfiles": [ 

            { 

                "resource": { 

                    "id": "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ContainerInstance/containerGroupProfiles/{{cgProfile1}}" 

 

                } 

            } 

        ] 

    } 

}
```

### NGroups API
This references a CG profile and adds additional orchestration related properties and capabilities. E.g.,  

- The desired count of CGs to scale out by 

- The VNET to connect all the CGs to while scaling out 

- The Load Balancer or Application Gateway to connect the CGs

This in turn invokes the ACI ARM APIs to create (and subsequently manage) each CG. Since it uses the same ARM APIs, there is no difference between the CGs created by NGroups and the CGs created directly by the customer. They have the exact same API experience.

### Updating a NGroup
As requirements change, we would need to keep updating our NGroups and its CGs. There are two update modes with which we can update a NGroups – **Manual** (default option) and **Rolling**. 

Consider a simple example of updating a CG profile reference from _cgprofile1_ to _cgprofile2_:

- In Manual mode, we update the reference to cgprofile2 and issue an UPDATE NGroups command:

NGroups stores this new CG profile reference. But it won’t update existing CGs with this reference. Existing CGs are currently running and there is no impact on them. However, when NGroups is scaled out, then new CGs will be created with cgprofile2. 

- How do we update existing CGs with cgprofile2?  

For this, we issue a ‘manual update’ command with an explicit list of CGs we want to update. This command only updates the CGs specified in its list. Updating the CG involves calling the ACI’s PUT CG API. The CGs not specified in this list will continue to run with cgprofile1. 

This mode gives us flexibility to update CGs selectively and provides full control over impact on production workloads.  

In **Rolling** mode, when we update the reference to cgprofile2 and issue an UPDATE NGroups command, existing CGs are updated with cgprofile2. The update to existing CGs happen in small batches (and not all at once). This ensures that there is a minimal impact on our workload since only a small percentage of CGs may be unavailable during the update.  

We can configure the batch size and other related rolling update mode settings with the NGroups API.


## **Try NGroups:**

### Prerequisites for working on NGroups:

 Currently supported API version is **2024-09-01-preview**.

1. Send request to NGroups Dev team to register your subscriptions under given AFECs: (corecontainersorg@microsoft.com or raise it under Partner channel) 
-    Microsoft.ContainerInstace/NGroupsPreview 
- Microsoft.ContainerInstance/NGroupsPreviewCanary 
2. Once the afec flags are applied to the subscription, run the following command to register the resourceProvider on your environment:
`az provider register -n Microsoft.ContainerInstance` 


|**IMP**: Please use the api-version - 2024-09-01-preview for canary preview|
|--|


[Swagger Spec](https://github.com/saprakas/azure-rest-api-specs/blob/main/specification/containerinstance/resource-manager/Microsoft.ContainerInstance/preview/2024-09-01-preview/containerInstance.json)


If these prerequisites are not met: requests will fail, and the nGroups resource type will not be recognized. 

### ARM template samples:

Create CG Profile: [ContainerGroupProfile-Sample.json](https://microsoft.sharepoint-df.com/:u:/t/CoreContainersOrg/ESNVt_BUgyhImgal7jUb6AwB6oTKDmjMVxFFs3xZ0JvPjg?e=LIeFnO)

Create Zonal NGroups with CGProfile: [NGroups-Zonal-Sample.json](https://microsoft.sharepoint-df.com/:u:/t/CoreContainersOrg/EbLBbNYGg1FEt0KQjr4Hw-0B8OVm5LRCAMuwFBOELOJEHw?e=noHok4)

NGroups are not visible on the portal as of today, but you can check CGs being created on the portal under the resourceGroup. Customer can also check NGroups resourceId under CG JSON view in orchestrationProfile property: 
![portalngroups.png](/.attachments/portalngroups-1fd4008a-b203-45e3-8e91-8f1a0ef86f21.png)

## **How-to-guide:**

### **How to create a regional (zonal/non-zonal) NGroups?**
First create a CG profile. A sample CG profile is shown below. Currently supported API version is 2024-09-01-preview

```
{ 

    "properties": { 

        "sku": "Standard", 

        "containers": [ 

            { 

                "name": "container1", 

                "properties": { 

                    "image": "nginx", 

                    "ports": [ 

                    { 

                        "protocol": "TCP", 

                        "port": 80 

                    }], 

                    "resources": { 

                        "requests": { 

                            "memoryInGB": 2.0, 

                            "cpu": 1.0 

                        } 

                    } 

                } 

            } 

        ], 

        "restartPolicy": "Always", 

        "shutdownGracePeriod": "PT1H", 

        "ipAddress": { 

            "ports": [ 

            { 

                "protocol": "TCP", 

                "port": 80 

            }], 

            "type": "Public", 

            "autoGeneratedDomainNameLabelScope": "Unsecure" 

        }, 

        "timeToLive": "PT1H", 

        "osType": "Linux" 

    }
```
 Next, you can create a Zonal/Non-Zonal NGroup by either adding zones outside the properties or can leave the zones array empty.

```
{ 

    "properties": { 

        "elasticProfile": { 

            "desiredCount": 5 

        }, 

        "containerGroupProfiles": [ 

            { 

                "resource": { 

                    "id": "[resourceId('Microsoft.ContainerInstance/containerGroupProfiles', parameters('cgProfileName'))]" 

                } 

            } 

        ] 

    }, 

    "zones": [ "1", "2", "3" ] 

}
```
 When the NGroups is scaled out by setting its desiredCount property, the CGs are distributed evenly across all the specified zones. If one zone goes down, the application will still be available since the remaining CGs of the NGroups are running in other zones.  

**Technical capabilities and constraints:** 

1. Once a NGroup is created with a set of zones (e.g., { “1”, “2” }), the zones cannot be removed. However, a new zone can be added to the list. E.g., { “1”, “2”, “3” } 

2. If a specified zone is down, then the overall NGroups operation to create the CGs fails. Retry the request once the zone is back up. Another option is to simply delete the failed CGs. 

3. During scale down, NGroups randomly deletes instances which might not always maintain AZ spread. However, subsequent scale-out operations always try to rebalance the AZ spread.  

4. AZ spread isn’t supported with Spot Container. If you have such a requirement, please reach out to the ACI team. NGroups can enable this scenario if ACI allows it. There is currently no technical limitation to support this, just COGs/business reasons in ACI. 

5. Please also see: impact of availability due to infrastructure/platform updates

### **Create NGroup CGs with a prefix:**

Customers can create NGroup CGs with a prefix instead of just GUID names:

```
"properties": { 

    "elasticProfile": { 

        "desiredCount": 2,             

        "containerGroupNamingPolicy": { 

            "guidNamingPolicy": { 

                "prefix": "cg-" 

            } 

        } 

    },
```
 This is useful when you have multiple NGroups in a single RG and want to differentiate CGs belonging to each NGroup (e.g., in the Azure portal view). You can also change this for each scale out operation to identify CGs that were scaled out together in one operation.

### **Create NGroup with both SystemAssigned and UserAssigned managed identity:**

```
“location”: “{{location}}” 

"identity": { 

    "type": "SystemAssigned, UserAssigned", 

    "userAssignedIdentities": { 

        "/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity1}}": {},  

"/subscriptions/{{subId}}/resourceGroups/{{rgName}}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{{userAssignedIdentity2}}": {} 

    }
```
### **If I delete some CGs from the NGroup, can the NGroup rebuild itself with new CGs to maintain its desired count?**

Yes, you can set the properties.elasticProfile.maintainDesiredCount bool property to true.  

This creates a new CG for every CG that is being deleted/detached from the NGroup. It will try and maintain the desiredCount property of the NGroup to its set value. 

This is useful when you want to use the NGroup as a ‘pool’ which automatically gets replenished when you take away CGs from the pool for your workload scenarios.  

This is a nullable bool property. If you omit it for subsequent NGroup PUT/update calls, it will not be reset to false. To reset, you must explicitly set it to false. When this is null/false, and when a CG is deleted/detached from the NGroup, the NGroup’s desiredCount property reduces accordingly. 
 
### **How do I get the CG name, NGroup Id and other metadata propagated down into the container?**

Currently, we expose only the CG name and orchestrator Id (the ARM resource Id). In future, additional properties could be considered. These 2 properties show up as container environment variables. 

To get these environment variables on the container, specify these tags _at the NGroup level_ 

```
“tags: { 

    “metadata.container.environmentVariable.containerGroupName”: true, 

    “metadata.container.environmentVariable.orchestratorId”: true, 

    : 

    : // other NGroup tags you may have 

    : 

}
```
NGroup understands these tags as ‘special’ and propagates the required environment variables down to each container as shown below.
![tagngroup.png](/.attachments/tagngroup-31a8d9f9-796b-4b4e-9f03-0f2346a502aa.png)
 
