---
title: How to deploy Azure File Sync (preview) | Microsoft Docs
description: Learn how to deploy Azure File Sync from start to finish.
services: storage
documentationcenter: ''
author: wmgries
manager: klaasl
editor: jgerend

ms.assetid: 297f3a14-6b3a-48b0-9da4-db5907827fb5
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/08/2017
ms.author: wgries
---

# How to deploy Azure File Sync (preview)
Azure File Sync (preview) allows you to centralize your organization's file shares in Azure Files without giving up the flexibility, performance, and compatibility of an on-premises file server. It does this by transforming your Windows Servers into a quick cache of your Azure File share. You can use any protocol available on Windows Server to access your data locally (including SMB, NFS, and FTPS) and you can have as many caches as you need across the world.

We strongly recommend reading [Planning for an Azure Files deployment](storage-files-planning.md) and [Planning for an Azure File Sync deployment](storage-sync-files-planning.md) before following the steps in this guide.

## Prerequisites
* A Storage Account and an Azure File share in the same region that you want to deploy Azure File Sync. For more information, see:
    - [Region availability](storage-sync-files-planning.md#region-availability) of Azure File Sync,
    - [Create a Storage Account](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) for step-by-step directions on how to create a Storage Account, and
    - [Create a file share](storage-how-to-create-file-share.md) for step-by-step directions on how to create a file share.
* At least one supported Windows Server or Windows Server cluster to sync with Azure File Sync. See [Interoperability with Windows Server](storage-sync-files-planning.md#azure-file-sync-interoperability) for more information on supported versions of Windows Server.

## Deploy the Storage Sync Service 
The Storage Sync Service is the top-level Azure resource representing Azure File Sync. To deploy a Storage Sync Service, navigate to the [Azure portal](https://portal.azure.com/), and search for Azure File Sync. After selecting "Azure File Sync (preview)" from the search results, select "Create" to pop open the "Deploy Storage Sync" tab.

The resulting blade asks for the following information:

- **Name**: A unique name (per subscription) for the Storage Sync Service.
- **Subscription**: The subscription in which to create the Storage Sync Service. Depending on your organization's configuration strategy, you may have access to one or more subscriptions. An Azure Subscription is the most basic container for billing for each cloud service (such as Azure Files).
- **Resource group**: A resource group is a logical group of Azure resources, such as a Storage Account or a Storage Sync Service. You may create a new resource group or use an existing resource group for Azure File Sync (we recommend using resource groups as containers used to isolate resources logically for your organization, such as grouping HR resources or resources for a particular project).
- **Location**: The region in which you would like to deploy Azure File Sync. Only supported regions are available in this list.

Once the "Deploy Storage Sync" form has been completed, click "Create" to deploy the Storage Sync Service.

## Prepare Windows Servers for use with Azure File Sync
For ever server with which you intend to use Azure File Sync, including server nodes in a Failover Cluster, complete the following steps:

For every server, including server nodes in a Failover Cluster, you intend to use with Azure File Sync, complete the following steps:

1. Disable Internet Explorer Enhanced Security Configuration. This is only required for initial server registration, and can be reenabled after the server has been registered.
    1. Open Server Manager.
    2. Click **Local Server**:  
        !["Local Server" on the left-hand side of the Server Manager UI](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
    3. Select the link for **IE Enhanced Security Configuration** on the Properties sub-pane:  
        ![The "IE Enhanced Security Configuration" in the Server Manager UI](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
    4. Select **Off** for both Administrators and Users in the Internet Explorer Enhanced Security Configuration pop-up window:  
        ![The Internet Explorer Enhanced Security Configuration pop-window with "Off" selected](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

2. Ensure that you are running at least PowerShell 5.1.\* (PowerShell 5.1 is the default on Windows Server 2016). You can verify you are running PowerShell 5.1.\* by looking at the value of the PSVersion property of the $PSVersionTable object:

    ```PowerShell
    $PSVersionTable.PSVersion
    ```

    - If your PSVersion is less than 5.1.\*, as will be the case with most installations of Windows Server 2012 R2, you can easily upgrade by downloading and installing [Windows Management Framework (WMF) 5.1](https://www.microsoft.com/download/details.aspx?id=54616). The appropriate package to download and install for Windows Server 2012 R2 is **Win8.1AndW2K12R2-KB\*\*\*\*\*\*\*-x64.msu**.

3. [Install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps). We always recommend using the latest version of the Azure PowerShell modules.

## Install the Azure File Sync agent
The Azure File Sync agent is a downloadable package which enables a Windows Server to be synchronized with an Azure File share. The agent can be downloaded from the [Microsoft Download Center](https://go.microsoft.com/fwlink/?linkid=858257). Once downloaded, double-click on the MSI package to start the Azure File Sync agent installation.

> [!Important]  
> If you intend to use Azure File Sync with a Failover Cluster, the Azure File Sync agent will need to be installed on every node in the cluster.

The Azure File Sync agent installation package should install relatively quickly without too many additional prompts. We recommend the following:
- Leave default installation path `C:\Program Files\Azure\StorageSyncAgent`) to simplify troubleshooting and server maintenance.
- Enabling Microsoft Update to keep Azure File Sync up to date. All updates, including feature updates and hotfixes, to the Azure File Sync agent will occur from Microsoft Update. We always recommend taking the latest update to Azure File Sync. Please see [Azure File Sync update policy](storage-sync-files-planning.md#azure-file-sync-agent-update-policy) for more information.

At the conclusion of the Azure File Sync agent installation, the Server Registration UI will auto-start. Please see the next section for how to register this server with Azure File Sync.

## Register Windows Server with Storage Sync Service
Registering your Windows Server with a Storage Sync Service establishes a trust-relationship between your server (or cluster) and the Storage Sync Service. The Server Registration UI should auto-start after the installation of the Azure File Sync agent, but if it doesn't, you can open it manually from its location: `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe`. Once the Server Registration UI is open, click **Sign-in** to begin.

After sign-in, you are prompted for the following information:

![A screenshot of the Server Registration UI](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Azure subscription**: The subscription containing the Storage Sync Service (see [Deploy the Storage Sync Service](#deploy-the-storage-sync-service) above). 
- **Resource group**: The resource group containing the Storage Sync Service.
- **Storage Sync Service**: The name of the Storage Sync Service with which you want to register.

Once you have selected the appropriate information from the drop downs, click **Register** to complete the server registration. As part of the registration process, you are prompted for an additional sign-in.

## Create a Sync Group
A Sync Group defines the sync topology for a set of files. Endpoints within a Sync Group will be kept in sync with each other. A Sync Group must contain at least one Cloud Endpoint, which represents an Azure File share, and one Server Endpoint, which represents a path on a Windows Server. To create a Sync Group, navigate to your Storage Sync Service in the [Azure portal](https://portal.azure.com/), and click **+ Sync group**:

![Create new Sync Group in the Azure portal](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

The resulting pane asks for the following information to create a Sync Group with a Cloud Endpoint:

- **Sync Group Name**: The name of the Sync Group to be created. This name must be unique within the Storage Sync Service, but can be any name that is logical for you.
- **Subscription**: The subscription you deployed the Storage Sync Service in [Deploy the Storage Sync Service](#deploy-the-storage-sync-service) above.
- **Storage account**: Clicking "Select storage account" will pop open an additional pane allowing you select the storage account containing the Azure File share with which you would like to sync.
- **Azure File share**: The name of the Azure File share with which you would like to sync.

To add a Server Endpoint, navigate to the newly created Sync Group and click "Add server endpoint".

![Add a new Server Endpoint in the Sync Group pane](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

The resulting "Add server endpoint" pane requires the following information to create a Server Endpoint:

- **Registered Server**: The name of the server or cluster on which to create the Server Endpoint.
- **Path**: The path on the Windows Server to be synchronized as part of the Sync Group.
- **Cloud Tiering**: A switch to enable or disable cloud tiering, which enables infrequently used or accessed files to be tiered to Azure Files.
- **Volume Free Space**: the amount of free space to reserve on the volume on which the Server Endpoint resides. For example, if the Volume Free Space is set to 50% on a volume with a single Server Endpoint, roughly half the amount of data will be tiered to Azure Files. Note that regardless of whether cloud tiering is enabled, your Azure File share always has a complete copy of the data in the Sync Group.

Click "Create" to add the Server Endpoint. Your files will now be kept in sync across your Azure File share and your Windows Server. 

> [!Important]  
> You can make changes to any Cloud or Server Endpoint in the Sync Group and have your files synchronized to the other endpoints in the Sync Group. If you make a change to the Cloud Endpoint (Azure File share) directly, please note that changes first need to be discovered by an Azure File Sync change detection job, which is only initatiated for a Cloud Endpoint once every 24 hours. See the [Azure Files FAQ](storage-files-faq.md#afs-change-detection) for more information.

## Next steps
- [Add/Remove an Azure File Sync Server Endpoint](storage-sync-files-server-endpoint.md)
- [Register/deregister a server with Azure File Sync](storage-sync-files-server-registration.md)