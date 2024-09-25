---
title: Installing VM Watch
description: Getting started guide to installing VM watchon virtual machines
author: iamwilliew 
ms.author: wwilliams
ms.service: azure-virtual-machines
ms.topic: get-started
ms.date:     09/23/2024
ms.subservice: monitoring
---

# Installing VM Watch

Users can enable VM watch with ease via ARM template, PowerShell, or AZ CLI (links). The following steps detail how VM watch can be installed on a single VM in a Linux environment, using AZ CLI (link) and Azure Portal (link)

1. Once on the Azure Portal, navigate to the Application + Extension Page to verify that Application Health Extension (link) does not exist

:::image type="content" source="media/vm-watch-application-extension.png" alt-text="Screenshot of the Application and extension example" :::

1. Prepare the installation code below and input the key parameters. These are given in the table below

   Linux Command: <The section below should be in Tabs - REST API, Azure PowerShell and Azure CLI 2.0 like in the screenshot below>


## Single VM

### [Azure CLI](#tab/Azure CLL)
```
az vm extension set --resource-group <resourcegroup> --vm-name < vmname> --name ApplicationHealthLinux --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable auto upgrade true 
```
#[Azure Powershell](#tab/Azure Powershell)
```
Set-AzVMExtension -ResourceGroupName "<resourcegroup>" -Location "<location>" -VMName "<vmname>" -Name "<extensionname>" -Publisher "Microsoft.ManagedServices" -ExtensionType "ApplicationHealthLinux" -TypeHandlerVersion "2.0" -Settings @{"vmWatchSettings" = @{"enabled" = $True}} -EnableAutomaticUpgrade $True 
```
#[ARM templates - REST API](#tab/ARM templates - REST API)

```
        "type": "Microsoft.Compute/virtualMachines/extensions", 

        "apiVersion": "2019-07-01", 

        "name": "[concat(parameters('vmName'), '/', 'HealthExtension')]", 

        "location": "[parameters('location')]", 

        "dependsOn": [ 

            "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]" 

        ], 

        "properties": { 

            "enableAutomaticUpgrade": true, 

            "publisher": "Microsoft.ManagedServices", 

            "typeHandlerVersion": "2.0", 

            "type": "ApplicationHealthLinux", 

            "settings": { 

                "vmWatchSettings": { 

                    "enabled": true                   } 

                } 

            } 

        } 

    } 

```
---

## VM Scaleset

### [Azure CLI](#tab/Azure CLI)
```
az vmss extension set --resource-group <resourcegroup> --vmss-name < vmssname> --name ApplicationHealthLinux --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable auto upgrade true
```
### [Azure Powershell](#tab/Azure Powershell)
```
### Define the scale set variables 

$vmScaleSetName = "<scalesetname>" 

$vmScaleSetResourceGroup = "<resourcegroup>" 

 

### Define the setting to enable vmwatch 

$publicConfig = @{"vmWatchSettings" = @{"enabled" = $true}} 

$extensionName = "<extensionname>" 

$extensionType = "ApplicationHealthLinux" 

$publisher = "Microsoft.ManagedServices" 

 

### Get the scale set object 

$vmScaleSet = Get-AzVmss ` 

  -ResourceGroupName $vmScaleSetResourceGroup ` 

  -VMScaleSetName $vmScaleSetName 

 

### Add the Application Health extension to the scale set model 

Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet ` 

  -Name $extensionName ` 

  -Publisher $publisher ` 

  -Setting $publicConfig ` 

  -Type $extensionType ` 

  -TypeHandlerVersion "2.0" ` 

  -EnableAutomaticUpgrade $True 

 

### Update the scale set 

Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup ` 

  -Name $vmScaleSetName ` 

  -VirtualMachineScaleSet $vmScaleSet 

 

### Upgrade instances to install the extension 

Update-AzVmssInstance -ResourceGroupName $vmScaleSetResourceGroup ` 

  -VMScaleSetName $vmScaleSetName ` 

  -InstanceId '*' 
```
### [ARM templates - REST API](#tab/arm templates - rest api)

``` 
    	"type": "Microsoft.Compute/virtualMachineScaleSets",  

    "apiVersion": "[parameters('vmss_api_version')]",  

    "name": "[variables('vmss_name')]",  

    "location": "[resourceGroup().location]",  

    "properties": {  

        "virtualMachineProfile": {  

            "extensionProfile": {  

                "extensions": [  

                    {  

                        "name": "[concat(variables('vmss_name'), '/', 'HealthExtension')]",  

                        "properties": {  

                            "publisher": "Microsoft.ManagedServices",  

                            "type": "ApplicationHealthLinux",  

                            "typeHandlerVersion": "2.0",  

                            "autoUpgradeMinorVersion": true,  

                            "settings": {  

                                "vmWatchSettings": {  

                                    "enabled": true  

                                }  

                            }  

                        }  

                    }  

                ]  

            }  

        }  

    }  

}   

```
---


## Windows

| **Parameters** | **Description** |
|---|---|
| **VM / VMSS Name** | This is the unique identifier assigned to your virtual machine |
| **Resource Group Name** | This is the unique identifier assigned to your resource group within an Azure subscription |

1. Run the prepared code using Azure CLI (link)

   

2. On Successful installation, navigate back to Azure Portal to confirm that Application Health has been installed.

:::image type="content" source="media/vm-watch-application-extension.png" alt-text="Screenshot of the Application and extension example" :::

3. To confirm that VM watch has been enabled on this VM, navigate back to the Overview Page and click on the JSON view for the VM 

:::image type="content" source="media/vm-watch-json-view.png" alt-text="Screenshot of theJSOn view example" :::


## Next Steps:

- [Application Health Extension](https://learn.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension?tabs=azure-cli)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Azure Portal](https://portal.azure.com/) 
