---
title: N-Phase rolling upgrades on Virtual Machine Scale Sets (Preview)
description: Learn about how to configure N-Phase rolling upgrades on Virtual Machine Scale Sets.
author: mimckitt
ms.author: mimckitt
ms.topic: how-to
ms.service: azure-virtual-machine-scale-sets
ms.date: 9/25/2024
ms.reviewer: ju-shim
ms.custom: upgradepolicy, N-Phase
---
# N-Phase rolling upgrades on Virtual Machine Scale Sets (Preview)

> [!NOTE]
>**N-Phase rolling upgrades Virtual Machine Scale Sets is currently in preview.** Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of these features may change prior to general availability (GA).

N-Phase rolling upgrades enables you to select which virtual machines are placed into each batch when performing a rolling upgrade. Additionally, N-Phase upgrades enable you to skip upgrades on specific virtual machines during the rolling upgrade process. 

## Requirements

- When using N-Phase rolling upgrades on Virtual Machine Scale Sets, the scale set must also use the [Application Health Extension with Rich Health States](virtual-machine-scale-sets-health-extension.md) to monitor application health and report phase ordering information. N-Phase upgrades is not supported when using the application health extension with binary states. 
- N-Phase upgrades requires the application health extension to emit a HTTP or HTTPs response. TCP responses are not supported. 


## Concepts

A phase is a high-level grouping construct for virtual machines. Each phase is determined by setting metadata emitted from the [Application Health Extension](virtual-machine-scale-sets-health-extension.md). N-Phase rolling upgrades take the information retrieved from the application health extension and use it to create upgrade batches within each phase. N-Phase rolling upgrades also uses update domains (UD), fault domains (FD) and zone information to ensure that each batch doesn't cross a boundary. This helps to further ensure resiliency when performing upgrades. 

The phased upgrades are performed in numerical sequence order. Until all the batches in the first phase are upgraded, the virtual machines in the following phases remain untouched. 

:::image type="content" source="./media/upgrade-policy/n-phase-regional-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a regional scale set.":::


:::image type="content" source="./media/upgrade-policy/n-phase-zonal-scale-set.png" alt-text="Diagram that shows a high level diagram of what happens when using n-phase upgrades on a zonal scale set.":::


Each virtual machine responds to the application health extension probes with response body contents containing metadata key-value pairs. This metadata is set by the customer and tells the platform how each virtual machine should interact with Rolling Upgrades. If no phase ordering metadata is received, the batches are determined by the scale set. 

To specify phase number the virtual machine should be associated with, use `phaseOrderingNumber` parameter.  

```HTTP
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": false, \"PhaseOrderingNumber\": 0 } }"
}
```

For skipping an upgrade on a virtual machine, use `SkipUpgrade` parameter. This tells the rolling upgrade to skip over this virtual machine when performing the upgrades.  

```HTTP
{
     “applicationHealthState”: “Healthy”,
      “customMetrics”: "{ \"rollingUpgrade\": { \"SkipUpgrade\": true, \"PhaseOrderingNumber\": 1 } }"
}
```

Once you have successfully configured the application health extension and custom metrics on each virtual machine, when a rolling upgrade is initiated, the virtual machines are placed into their designated phases and each phase inherits the rolling upgrade policy associated with the scale set. For more information on the rolling upgrade policy, see [configuring the rolling upgrade policy](virtual-machine-scale-sets-configure-rolling-upgrades.md) for Virtual Machine Scale Sets. 

## Configure the application health extension

### Install the application health extension

The extension requires at a minimum either an "http" or "https" request with an associated port or request path respectively. TCP probes are also supported, but cannot set the `ApplicationHealthState` through the probe response body and do not have access to the *Unknown* state.

| Name | Value / Example | Data Type |
| ---- | --------------- | --------- |
| protocol | `http` or `https` or `tcp` | string |
| port | Optional when protocol is `http` or `https`, mandatory when protocol is `tcp` | int |
| requestPath | Mandatory when protocol is `http` or `https`, not allowed when protocol is `tcp` | string |
| intervalInSeconds | Optional, default is 5 seconds. This setting is the interval between each health probe. For example, if intervalInSeconds == 5, a probe is sent to the local application endpoint once every 5 seconds. | int |
| numberOfProbes | Optional, default is 1. This setting is the number of consecutive probes required for the health status to change. For example, if numberOfProbles == 3, you will need 3 consecutive "Healthy" signals to change the health status from "Unhealthy"/"Unknown" into "Healthy" state. The same requirement applies to change health status into "Unhealthy" or "Unknown" state.  | int |
| gracePeriod | Optional, default = `intervalInSeconds` * `numberOfProbes`; maximum grace period is 7200 seconds | int |

#### [Azure CLI](#tab/azure-cli)

Use [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set) to add the Application Health extension to the scale set model definition.

The following example adds the **Application Health - Rich States** extension to the scale set model of a Linux-based scale set.

You can also use this example to upgrade an existing extension from Binary to Rich Health States.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthLinux \
  --publisher Microsoft.ManagedServices \
  --version 2.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --settings ./extension.json
```
The extension.json file content.

```json
{
  "protocol": "<protocol>",
  "port": <port>,
  "requestPath": "</requestPath>",
  "gracePeriod": <healthExtensionGracePeriod>
}
```
**Upgrade the VMs to install the extension.**

```azurecli-interactive
az vmss update-instances \
  --resource-group <myVMScaleSetResourceGroup> \
  --name <myVMScaleSet> \
  --instance-ids "*"
```

#### [Azure PowerShell](#tab/azure-powershell)

Use the [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) cmdlet to add the Application Health extension to the scale set model definition.

The following example adds the **Application Health - Rich States** extension to the `extensionProfile` in the scale set model of a Windows-based scale set. The example uses the new Az PowerShell module.

```azurepowershell-interactive
# Define the scale set variables
$vmScaleSetName = "myVMScaleSet"
$vmScaleSetResourceGroup = "myVMScaleSetResourceGroup"

# Define the Application Health extension properties
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"; "gracePeriod" = 600};
$extensionName = "myHealthExtension"
$extensionType = "ApplicationHealthWindows"
$publisher = "Microsoft.ManagedServices"

# Get the scale set object
$vmScaleSet = Get-AzVmss `
  -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName

# Add the Application Health extension to the scale set model
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet `
  -Name $extensionName `
  -Publisher $publisher `
  -Setting $publicConfig `
  -Type $extensionType `
  -TypeHandlerVersion "2.0" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
  
# Upgrade instances to install the extension
Update-AzVmssInstance -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName `
  -InstanceId '*'

```


##### [REST API](#tab/rest-api)

The following example adds the **Application Health - Rich States** extension (with name `myHealthExtension`) to the `extensionProfile` in the scale set model of a Windows-based scale set.

You can also use this example to upgrade an existing extension from Binary to Rich Health States by making a PATCH call instead of a PUT.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/myHealthExtension?api-version=2018-10-01`
```

```json
{
  "name": "myHealthExtension",
  "location": "<location>",
  "properties": {
    "publisher": "Microsoft.ManagedServices",
    "type": "ApplicationHealthWindows",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "2.0",
    "settings": {
      "protocol": "<protocol>",
      "port": <port>,
      "requestPath": "</requestPath>",
      "intervalInSeconds": <intervalInSeconds>,
      "numberOfProbes": <numberOfProbes>,
      "gracePeriod": <gracePeriod>
    }
  }
}
```
Use `PATCH` to edit an already deployed extension.

**Upgrade the VMs to install the extension.**

```
POST on `/subscriptions/<subscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/< myScaleSet >/manualupgrade?api-version=2022-08-01`
```

```json
{
  "instanceIds": ["*"]
}
```

---

### Configure the application health extension response
Configuring the application health extension response can be accomplished in many different ways. It can be integrated into existing applications, dynamically updated and be used along side various functions to provide an output based on a specific situation. 

#### Example 1
This sample app will configure a health state based on the assigned Availability Zone of the VM. For best results, please create a VMSS with 2 or more availability zones. This application requires PowerShell and is recommended for Windows VMs.

The VM health states will be set based on the following:

Zone 1 ==> "Healthy"
Zone 2 ==> "Unhealthy"
Zone 3 ==> "Unknown"

You can use Custom Script Extension to run [start.ps1](https://github.com/Azure-Samples/application-health-samples/blob/main/Rich%20Health%20States/powershell-demo/start.ps1) on your VM, it will then download application.ps1 and start emitting HTTP health probe responses to "http://localhost:8000/".

```powershell
New-NetFirewallRule -DisplayName 'HTTP(S) Inbound' -Direction Inbound -Action Allow -Protocol TCP -LocalPort @('8000')
$Hso = New-Object Net.HttpListener
$Hso.Prefixes.Add('http://localhost:8000/')
$Hso.Start()

function GetZoneFromImds()
{
    $imdsObject = Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -Uri "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
    $zone = $imdsObject.compute.zone
    return [int]($zone)
}

function GenerateResponseJson()
{
    $zone = GetZoneFromImds

    # During grace period, Health Signal ==> Initializing
    # After grace period if application still communicates "Invalid", Health Signal ==> "Unknown" ~ "Unhealthy"

    # In this demo: If VM is in zone 1 ==> "Healthy" status, zone 2 ==> "Unhealthy" status, other zones ==> "Unknown" status
    $appHealthState = if (1 -eq $zone) { "Healthy" } elseif (2 -eq $zone) { "Unhealthy" } else { "Invalid" } 

    $hashTable = @{
        'ApplicationHealthState' = $appHealthState
        'CustomMetrics' = @{
                            'RollingUpgrade' = @{
                                'SkipUpgrade' = false
                                'PhaseOrderingNumber' = 1
                            }
                        }
    } 
    $hashTable.CustomMetrics = ($hashTable.CustomMetrics | ConvertTo-Json)
    return ($hashTable | ConvertTo-Json)
}


While($Hso.IsListening)
{
    $context = $Hso.GetContext()
    $response = $context.Response
    $response.StatusCode = 200
    $response.ContentType = 'application/json'
    $responseJson = GenerateResponseJson
    $responseBytes = [System.Text.Encoding]::UTF8.GetBytes($responseJson)
    $response.OutputStream.Write($responseBytes, 0, $responseBytes.Length)
    $response.Close()
}

$Hso.Stop()
```


#### Example 2
This sample will create a listener which can respond to the health extension calls and will skip an upgrade for instance 1.


```powershell
 New-NetFirewallRule -DisplayName 'HTTP(S) Inbound' -Direction Inbound -Action Allow -Protocol TCP -LocalPort @('8000')
                $Hso = New-Object Net.HttpListener
                $Hso.Prefixes.Add('http://localhost:8000/')
                $Hso.Start()
                function GetVmInstanceIdFromImds()
                {
                    $imdsObject = Invoke-RestMethod -Headers @{""Metadata""=""true""} -Method GET -Uri ""http://169.254.169.254/metadata/instance?api-version=2021-02-01""
                    $vmName = $imdsObject.compute.name
                    return [int]($vmName.Split('_')[1])
                }
                function GenerateResponseJson()
                {
                    $numberOfPhases = 3
                    $vmInstanceId = GetVmInstanceIdFromImds
                    $skipUpgrade = 1 -eq $vmInstanceId
                    $appHealthState = ""Healthy""
                    $phaseOrderingNumber = $numberOfPhases - ($vmInstanceId % $numberOfPhases)
                    $hashTable = @{
                        'ApplicationHealthState' = $appHealthState
                        'CustomMetrics' = @{
                            'RollingUpgrade' = @{
                                'SkipUpgrade' = $skipUpgrade
                                'PhaseOrderingNumber' = $phaseOrderingNumber
                            }
                        }
                    } 
                    $hashTable.CustomMetrics = ($hashTable.CustomMetrics | ConvertTo-Json)
                    return ($hashTable | ConvertTo-Json)
                }
                While($Hso.IsListening)
                {
                    $context = $Hso.GetContext()
                    $response = $context.Response
                    $response.StatusCode = 200
                    $response.ContentType = 'application/json'
                    $responseJson = GenerateResponseJson
                    $responseBytes = [System.Text.Encoding]::UTF8.GetBytes($responseJson)
                    $response.OutputStream.Write($responseBytes, 0, $responseBytes.Length)
                    $response.Close()
                }
                $Hso.Stop()
```



For more response configuration examples, see [Application Health Samples](https://github.com/Azure-Samples/application-health-samples)


## Next steps
Learn how to [perform manual upgrades](virtual-machine-scale-sets-perform-manual-upgrades.md) on Virtual Machine Scale Sets. 
