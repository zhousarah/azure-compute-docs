---
title: Private Preview - Azure Impact Reporting #Required; page title is displayed in search results. Include the brand.
description: How to register for Azure Impact Reporting private preview to enable capabilities of sharing impact reports with Microsoft. #Required; article description that is displayed in search results. 
author: rolandnyamo #Required; your GitHub user alias, with correct capitalization.
ms.author: ronyamo #Required; microsoft alias of author; optional team alias.
ms.topic: how-to #Required; leave this attribute/value as-is.
ms.service: azure-impact-reporting #Required; use either service or product per approved list. 
ms.custom: template-overview #Required; leave this attribute/value as-is.
---

# Register your Subscription for Impact Reporting Feature (Preview)
> [!IMPORTANT]
> Azure Impact Reporting is currently in PREVIEW. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

Follow the steps below to register your subscription for Impact Reporting.

1. Navigate to your subscription
1. Under the `Settings` tab section, go to `Preview Features`
1. Under this tab, filter for `Microsoft.Impact` in the `type` section
![image](images/preview.png)
1. Click on `Allow Impact Reporting` feature and register
1. After approval, please go to `Resource providers`, search for `Microsoft.Impact` and register

Once your request is approved, you will have the ability to report Impact to your Azure workloads.

### Register your Subscription for Impact Reporting Feature - Script

To onboard multiple subscriptions, please use the collowing script.

<!-- [ NOTE, TIP, IMPORTANT, CAUTION, and WARNING are supported. Case-sensitive.]  -->

> [!WARNING]
> Please note that the following script is offered with no guaranty from Microsoft.

```bash
#!/bin/bash

# List of your subscription IDs
SUBSCRIPTIONS=("sub_Id", "sub_Id2")

# Resource provider namespace to register, e.g., 'Microsoft.Compute'
PROVIDER_NAMESPACE="Microsoft.Impact"

# Feature name
FEATURE_NAME="AllowImpactReporting"

# AppId/MI id that needs "Impact Reporter" role
APP_ID="app_Id"

# role name that's used to grant access to appId/MI to be able to report impacts
ROLE_NAME="Impact Reporter"

# Loop through each subscription
for SUBSCRIPTION_ID in "${SUBSCRIPTIONS[@]}"
do
    # Select the subscription
    az account set --subscription "$SUBSCRIPTION_ID"

    # register resource provider
    az provider register --namespace "$PROVIDER_NAMESPACE" --wait

    # register preview feature
    az feature register --namespace "$PROVIDER_NAMESPACE" --name "$FEATURE_NAME"
   
    # Grant the role to the app ID or managed identity
    az role assignment create --assignee "$APP_ID" --role "$ROLE_NAME"	

    echo "Registered $PROVIDER_NAMESPACE in $SUBSCRIPTION_ID"
done
```

### [HPC] Register your Subscription for Impact Reporting Feature - Script

> [!IMPORTANT]
> The following script is intended to be used for HPC Guest Health Reporting scenario customers.

> [!WARNING]
> Please note that the following script is offered with no guaranty from Microsoft.

```bash
#!/bin/bash

# List of your subscription IDs
SUBSCRIPTIONS=("sub_Id1")

# Resource provider namespace to register, e.g., 'Microsoft.Compute'
PROVIDER_NAMESPACE="Microsoft.Impact"

# Feature name
FEATURE_NAME="AllowImpactReporting"

# HPC Feature name
HPC_FEATURE_NAME="AllowHPCImpactReporting"

# AppId/MI id that needs impact reporter role
APP_ID="app_Id"

# role name that's used to grant access to appId/MI to be able to report impacts
ROLE_NAME="Impact Reporter"

# Loop through each subscription
for SUBSCRIPTION_ID in "${SUBSCRIPTIONS[@]}"
do
    # Select the subscription
    az account set --subscription "$SUBSCRIPTION_ID"

    # register resource provider
    az provider register --namespace "$PROVIDER_NAMESPACE" --wait

    # register preview feature
    az feature register --namespace "$PROVIDER_NAMESPACE" --name "$FEATURE_NAME"

    # register preview feature
    az feature register --namespace "$PROVIDER_NAMESPACE" --name "$HPC_FEATURE_NAME"
   
    # Grant the role to the app ID or managed identity
    az role assignment create --assignee "$APP_ID" --role "$ROLE_NAME"	

    echo "Registered $PROVIDER_NAMESPACE in $SUBSCRIPTION_ID"
done
```

## Next steps
<!-- Add a context sentence for the following links -->
- [File an impact report](ReportVMImpact.md)
