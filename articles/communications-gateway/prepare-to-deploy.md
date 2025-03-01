---
title: Prepare to deploy Azure Communications Gateway 
description: Learn how to complete the prerequisite tasks required to deploy Azure Communications Gateway in Azure.
author: rcdun
ms.author: rdunstan
ms.service: communications-gateway
ms.topic: how-to 
ms.date: 03/13/2023
---

# Prepare to deploy Azure Communications Gateway

This article guides you through each of the tasks you need to complete before you can start to deploy Azure Communications Gateway. In order to be successfully deployed, the Azure Communications Gateway has dependencies on the state of your Operator Connect or Teams Phone Mobile environments.
The following sections describe the information you need to collect and the decisions you need to make prior to deploying Azure Communications Gateway.

## Prerequisites

You must have signed an Operator Connect agreement with Microsoft. For more information, see [Operator Connect](https://cloudpartners.transform.microsoft.com/practices/microsoft-365-for-operators/connect).

You need an onboarding partner for integrating with Microsoft Phone System. If you're not eligible for onboarding to Microsoft Teams through Azure Communications Gateway's [Basic Integration Included Benefit](onboarding.md) or you haven't arranged alternative onboarding with Microsoft through a separate arrangement, you need to arrange an onboarding partner yourself.

You must own globally routable numbers that you can use for testing, as follows.

|Type of testing|Numbers required |
|---------|---------|
|Automated validation testing by Microsoft Teams test suites|Minimum: 3. Recommended: 6 (to run tests simultaneously).|
|Manual test calls made by you and/or Microsoft staff during integration testing |Minimum: 1|

We strongly recommend that you have a support plan that includes technical support, such as [Microsoft Unified Support](https://www.microsoft.com/en-us/unifiedsupport/overview) or [Premier Support](https://www.microsoft.com/en-us/unifiedsupport/premier).

## 1. Add the Project Synergy application to your Azure tenancy

> [!NOTE]
>This step and the next step ([2. Assign an Admin user to the Project Synergy application](#2-assign-an-admin-user-to-the-project-synergy-application)) set you up as an Operator in the Teams Phone Mobile (TPM) and Operator Connect (OC) environments. If you've already gone through onboarding, go to [3. Create a network design](#3-create-a-network-design).

The Operator Connect and Teams Phone Mobile programs require your Azure Active Directory tenant to contain a Microsoft application called Project Synergy. Operator Connect and Teams Phone Mobile inherit permissions and identities from your Azure Active Directory tenant through the Project Synergy application. The Project Synergy application also allows configuration of Operator Connect or Teams Phone Mobile and assigning users and groups to specific roles.

We recommend that you use an existing Azure Active Directory tenant for Azure Communications Gateway, because using an existing tenant uses your existing identities for fully integrated authentication. However, if you need to manage identities for Operator Connect separately from the rest of your organization, create a new dedicated tenant first.

To add the Project Synergy application:

1. Sign in to the [Azure portal](https://ms.portal.azure.com/) as an Azure Active Directory Global Admin.
1. Select **Azure Active Directory**.
1. Select **Properties**.
1. Scroll down to the Tenant ID field. Your tenant ID is in the box. Make a note of your tenant ID.
1. Open PowerShell.
1. If you don't have the Azure Active Directory module installed, install it:
    ```azurepowershell
    Install-Module AzureAD
    ```
1. Run the following cmdlet, replacing *`<AADTenantID>`* with the tenant ID you noted down in step 4.
    ```azurepowershell
    Connect-AzureAD -TenantId "<AADTenantID>"
    New-AzureADServicePrincipal -AppId eb63d611-525e-4a31-abd7-0cb33f679599 -DisplayName "Operator Connect"
    ```

## 2. Assign an Admin user to the Project Synergy application

The user who sets up Azure Communications Gateway needs to have the Admin user role in the Project Synergy application.

1. In your Azure portal, navigate to **Enterprise applications** using the left-hand side menu. Alternatively, you can search for it in the search bar; it's under the **Services** subheading.
1. Set the **Application type** filter to **All applications** using the drop-down menu.
1. Select **Apply**.
1. Search for **Project Synergy** using the search bar. The application should appear.
1. Select your **Project Synergy** application.
1. Select **Users and groups** from the left hand side menu.
1. Select **Add user/group**.
1. Specify the user you want to use for setting up Azure Communications Gateway and give them the **Admin** role.

## 3. Create a network design

Ensure your network is set up as shown in the following diagram and has been configured in accordance with the *Network Connectivity Specification* that you've been issued. You must have two Azure Regions with cross-connect functionality. For more information on the reliability design for Azure Communications Gateway, see [Reliability in Azure Communications Gateway](reliability-communications-gateway.md).

To configure MAPS, follow the instructions in [Azure Internet peering for Communications Services walkthrough](../internet-peering/walkthrough-communications-services-partner.md).
    :::image type="content" source="media/azure-communications-gateway-redundancy.png" alt-text="Network diagram of an Azure Communications Gateway that uses MAPS as its peering service between Azure and an operators network.":::

## 4. Collect basic information for deploying an Azure Communications Gateway

 Collect all of the values in the following table for the Azure Communications Gateway resource.

|**Value**|**Field name(s) in Azure portal**|
 |---------|---------|
 |The name of the Azure subscription to use to create an Azure Communications Gateway resource. You must use the same subscription for all resources in your Azure Communications Gateway deployment. |**Project details: Subscription**|
 |The Azure resource group in which to create the Azure Communications Gateway resource. |**Project details: Resource group**|
 |The name for the deployment. This name can contain alphanumeric characters and `-`. It must be 3-24 characters long. |**Instance details: Name**|
 |The management Azure region: the region in which your monitoring and billing data is processed. We recommend that you select a region near or colocated with the two regions for handling call traffic. |**Instance details: Region**
 |The voice codecs to use between Azure Communications Gateway and your network. |**Instance details: Supported Codecs**|
 |The Unified Communications as a Service (UCaaS) platform(s) Azure Communications Gateway should support. These platforms are Teams Phone Mobile and Operator Connect Mobile. |**Instance details: Supported Voice Platforms**|
 |Whether your Azure Communications Gateway resource should handle emergency calls as standard calls or directly route them to the Emergency Services Routing Proxy (US only). |**Instance details: Emergency call handling**|
 |The scope at which Azure Communications Gateway's autogenerated domain name label is unique. Communications Gateway resources get assigned an autogenerated domain name label that depends on the name of the resource. You'll need to register the domain name later when you deploy Azure Communications Gateway. Selecting **Tenant** gives a resource with the same name in the same tenant but a different subscription the same label. Selecting **Subscription** gives a resource with the same name in the same subscription but a different resource group the same label. Selecting **Resource Group** gives a resource with the same name in the same resource group the same label. Selecting **No Re-use** means the label doesn't depend on the name, resource group, subscription or tenant. |**Instance details: Auto-generated Domain Name Scope**|
 |The number used in Teams Phone Mobile to access the Voicemail Interactive Voice Response (IVR) from native dialers.|**Instance details: Teams Voicemail Pilot Number**|
 |A list of dial strings used for emergency calling.|**Instance details: Emergency Dial Strings**|
 |Whether an on-premises Mobile Control Point is in use.|**Instance details: Enable on-premises MCP functionality**|

## 5. Collect Service Regions configuration values

Collect all of the values in the following table for both service regions in which you want to deploy Azure Communications Gateway.

 |**Value**|**Field name(s) in Azure portal**|
 |---------|---------|
 |The Azure regions to use for call traffic. |**Service Region One/Two: Region**|
 |The IPv4 address used by Microsoft Teams to contact your network from this region. |**Service Region One/Two: Operator IP address**|
 |The set of IP addresses/ranges that are permitted as sources for signaling traffic from your network. Provide an IPv4 address range using CIDR notation (for example, 192.0.2.0/24) or an IPv4 address (for example, 192.0.2.0). You can also provide a comma-separated list of IPv4 addresses and/or address ranges.|**Service Region One/Two: Allowed Signaling Source IP Addresses/CIDR Ranges**|
 |The set of IP addresses/ranges that are permitted as sources for media traffic from your network. Provide an IPv4 address range using CIDR notation (for example, 192.0.2.0/24) or an IPv4 address (for example, 192.0.2.0). You can also provide a comma-separated list of IPv4 addresses and/or address ranges.|**Service Region One/Two: Allowed Media Source IP Address/CIDR Ranges**|

## 6. Collect Test Lines configuration values

Collect all of the values in the following table for all the test lines that you want to configure for Azure Communications Gateway.

 |**Value**|**Field name(s) in Azure portal**|
 |---------|---------|
 |The name of the test line. |**Name**|
 |The phone number of the test line, in E.164 format and including the country code. |**Phone Number**|
 |The purpose of the test line: **Manual** (for manual test calls by you and/or Microsoft staff during integration testing) or **Automated** (for automated validation with Microsoft Teams test suites).|**Testing purpose**|

> [!IMPORTANT]
> You must configure at least three automated test lines. We recommend six automated test lines (to allow simultaneous tests).

## 7. Decide if you want tags

Resource naming and tagging is useful for resource management. It enables your organization to locate and keep track of resources associated with specific teams or workloads and also enables you to more accurately track the consumption of cloud resources by business area and team.

If you believe tagging would be useful for your organization, design your naming and tagging conventions following the information in the [Resource naming and tagging decision guide](/azure/cloud-adoption-framework/decision-guides/resource-tagging/).

## 8. Get access to Azure Communications Gateway for your Azure subscription

Access to Azure Communications Gateway is restricted. When you've completed the previous steps in this article, contact your onboarding team and ask them to enable your subscription. If you don't already have an onboarding team, contact azcog-enablement@microsoft.com with your Azure subscription ID and contact details.

Wait for confirmation that Azure Communications Gateway is enabled before moving on to the next step.

## 9. Register the Microsoft Voice Services resource provider

If the **Microsoft.VoiceServices** resource provider isn't already registered in your subscription, register this provider.

1. Sign in to the [Azure portal](https://portal.azure.com/) and go to your Azure subscription.
1. Select **Resource providers** under the **Settings** tab.
1. Search for the **Microsoft.VoiceServices** resource provider.
1. Check if the resource provider is already marked as registered. If it isn't, choose the resource provider and select **Register**.

## 10. Set up application roles for Azure Communications Gateway

Azure Communications Gateway contains services that need to access the Operator Connect API on your behalf. To enable this access, you must grant specific application roles to an AzureCommunicationsGateway service principal under the Project Synergy Enterprise Application. You created the Project Synergy application in [1. Add the Project Synergy application to your Azure tenancy](#1-add-the-project-synergy-application-to-your-azure-tenancy). We created the Azure Communications Gateway service principal for you when you followed [9. Register the Microsoft Voice Services resource provider](#9-register-the-microsoft-voice-services-resource-provider).

> [!IMPORTANT]
> Granting permissions has two parts: configuring the service principal with the appropriate roles (this step) and adding the ID of the service principal to the Operator Connect or Teams Phone Mobile environment. You'll add the service principal to the Operator Connect or Teams Phone Mobile environment later, as part of [deploying Azure Communications Gateway](deploy.md).

Do the following steps in the tenant that contains your Project Synergy application.

1. Sign in to the [Azure portal](https://ms.portal.azure.com/) as an Azure Active Directory Global Admin.
1. Select **Azure Active Directory**.
1. Select **Properties**.
1. Scroll down to the Tenant ID field. Your tenant ID is in the box. Make a note of your tenant ID.
1. Open PowerShell.
1. If you didn't install the Azure Active Directory module as part of [1. Add the Project Synergy application to your Azure tenancy](#1-add-the-project-synergy-application-to-your-azure-tenancy), install it:
    ```azurepowershell
    Install-Module AzureAD
    ```
1. Run the following cmdlet, replacing *`<AADTenantID>`* with the tenant ID you noted down in step 4.
    ```azurepowershell
    Connect-AzureAD -TenantId "<AADTenantID>"
    ```
1. Run the following PowerShell commands. These commands add the following roles for Azure Communications Gateway: `TrunkManagement.Read`, `NumberManagement.Read`, `NumberManagement.Write`, `Data.Read`, `Data.Write`, `TrunkManagement.Write`, `PartnerSettings.Read`.
    ```azurepowershell
    # Get the Service Principal ID for Azure Communications Gateway
    $commGwayApplicationId = "8502a0ec-c76d-412f-836c-398018e2312b"
    $commGwayEnterpriseApplication = Get-AzureADServicePrincipal -Filter "AppId eq '$commGwayApplicationId'"
    $commGwayObjectId = $commGwayEnterpriseApplication.ObjectId
    
    # Get the Service Principal ID for Project Synergy (Operator Connect)
    $projectSynergyApplicationId = "eb63d611-525e-4a31-abd7-0cb33f679599"
    $projectSynergyEnterpriseApplication = Get-AzureADServicePrincipal -Filter "AppId eq '$projectSynergyApplicationId'"
    $projectSynergyObjectId = $projectSynergyEnterpriseApplication.ObjectId
    
    # Required Operator Connect - Project Synergy Roles
    $trunkManagementRead = "72129ccd-8886-42db-a63c-2647b61635c1"
    $trunkManagementWrite = "e907ba07-8ad0-40be-8d72-c18a0b3c156b"
    
    $partnerSettingsRead = "d6b0de4a-aab5-4261-be1b-0e1800746fb2"
    
    $numberManagementRead = "130ecbe2-d1e6-4bbd-9a8d-9a7a909b876e"
    $numberManagementWrite = "752b4e79-4b85-4e33-a6ef-5949f0d7d553"
    
    $dataRead = "eb63d611-525e-4a31-abd7-0cb33f679599"
    $dataWrite = "98d32f93-eaa7-4657-b443-090c23e69f27"
    
    $requiredRoles = $trunkManagementRead, $numberManagementRead, $numberManagementWrite, $dataRead, $dataWrite, $trunkManagementWrite, $partnerSettingsRead
    
    foreach ($role in $requiredRoles) {
        # Assign the relevant Role to the Azure Communications Gateway Service Principal
        New-AzureADServiceAppRoleAssignment -ObjectId $commGwayObjectId -PrincipalId $commGwayObjectId -ResourceId $projectSynergyObjectId -Id $role
    }
    ```

## Next steps

- [Create an Azure Communications Gateway resource](deploy.md)