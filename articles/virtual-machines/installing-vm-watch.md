---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title:       # Add a title for the browser tab
description: # Add a meaningful description for search results
author:      iamwilliew # GitHub alias
ms.author:   iamwilliew # Microsoft alias
ms.service:  # Add the ms.service or ms.prod value
# ms.prod:   # To use ms.prod, uncomment it and delete ms.service
ms.topic:    # Add the ms.topic value
ms.date:     09/23/2024
---

# Installing VM Watch

Users can enable VM watch with ease via ARM template, PowerShell, or AZ CLI (links). The following steps detail how VM watch can be installed on a single VM in a Linux environment, using AZ CLI (link) and Azure Portal (link)

1. Once on the Azure Portal, navigate to the Application + Extension Page to verify that Application Health Extension (link) does not exist

   ![A screenshot of a computer  Description automatically generated](media/image1.png)

1. Prepare the installation code below and input the key parameters. These are given in the table below

   Linux Command: <The section below should be in Tabs - REST API, Azure PowerShell and Azure CLI 2.0 like in the screenshot below>

   ![A close-up of a logo  Description automatically generated](media/image2.png)

**Azure cli:**

**Single VM:**

az vm extension set --resource-group <your resource group> --vm-name < vm name> --name ApplicationHealthLinux --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable auto upgrade true

**VM** **Scaleset:**<br>az vmss extension set --resource-group <your resource group> --vmss-name < vmss name> --name ApplicationHealthLinux --publisher Microsoft.ManagedServices --version 2.0 --settings '{"vmWatchSettings": {"enabled": true}}' --enable auto upgrade true

- 

**Azure** **Powershell**

**Single VM:**

Set-AzVMExtension -ResourceGroupName "<your resource group>" -Location "<your location>" -VMName "<vm name>" -Name "<your extension name>" -Publisher "Microsoft.ManagedServices" -ExtensionType "ApplicationHealthLinux" -TypeHandlerVersion "2.0" -Settings @{"vmWatchSettings" = @{"enabled" = $True}} -EnableAutomaticUpgrade $True

**VM** **ScaleSet**

# Define the scale set variables

$vmScaleSetName = "<your scaleset name>"

$vmScaleSetResourceGroup = "<your resource group>"

# Define the setting to enable vmwatch

$publicConfig = @{"vmWatchSettings" = @{"enabled" = $true}}

$extensionName = "<your extension name>"

$extensionType = "ApplicationHealthLinux"

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

  -EnableAutomaticUpgrade $True

# Update the scale set

Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `

  -Name $vmScaleSetName `

  -VirtualMachineScaleSet $vmScaleSet

# Upgrade instances to install the extension

Update-AzVmssInstance -ResourceGroupName $vmScaleSetResourceGroup `

  -VMScaleSetName $vmScaleSetName `

  -InstanceId '\*'

**Azure Resource Manager templates - REST API**

**Single VM**

```

        "type": "Microsoft.Compute/virtualMachines/extensions",
        "apiVersion": "2019-07-01",
        "name": "[concat(parameters('vmName'), '/', 'HealthExtension')]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
        ],
        "properties": {
            "enableAutomaticUpgrade": true,
            "publisher": "Microsoft.ManagedServices",
            "typeHandlerVersion": "2.0",
            "type": "ApplicationHealthLinux",
            "settings": {
                "vmWatchSettings": {
                    "enabled": true                   }
                }
            }
        }
    }
```

**VM** **Scaleset**

{ 

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

**Windows section:** 

Same screenshots as above but using ApplicationHealthWindows

| **Parameters** | **Description** |
|---|---|
| **VM / VMSS Name** | This is the unique identifier assigned to your virtual machine |
| **Resource Group Name** | This is the unique identifier assigned to your resource group within an Azure subscription |

1. Run the prepared code using Azure CLI (link)

   

1. On Successful installation, navigate back to Azure Portal to confirm that Application Health has been installed.

   ![A screenshot of a computer  Description automatically generated](media/image3.png)

1. To confirm that VM watch has been enabled on this VM, navigate back to the Overview Page and click on the JSON view for the VM 

   ![A screenshot of a computer](media/image4.png)

   

Next Steps: