---
title: Planning for an Azure File Sync (preview) deployment | Microsoft Docs
description: Learn what to consider when planning for an Azure Files deployment.
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

# Planning for an Azure File Sync (preview) deployment
Azure File Sync (preview) allows you to centralize your organization's file shares in Azure Files without giving up the flexibility, performance, and compatibility of an on-premises file server. It does this by transforming your Windows Servers into a quick cache of your Azure File share. You can use any protocol available on Windows Server to access your data locally (including SMB, NFS, and FTPS) and you can have as many caches as you need across the world.

This guide describes what to consider when deploying Azure File Sync. It is recommended that you read [Planning for an Azure Files deployment](storage-files-planning.md) guide testing. 

## Understanding Azure File Sync terminology
Before going into the Azure File Sync details, it's important to understand the terminology.

### Storage Sync Service
The Storage Sync Service is the top-level Azure resource representing Azure File Sync. The Storage Sync Service resource is a peer of the Storage Account resource, and can similarily be deployed into Azure Resouce Groups. A distinct top-level resource from the Storage Account resource is required because the Storage Sync Service can create sync relationships with multiple storage accounts via multiple Sync Groups. A subscription can have multiple Storage Sync Service resources deployed.

### Sync Group
A Sync Group defines the sync topology for a set of files. Endpoints within a Sync Group will be kept in sync with each other. If, for example, you have two distinct sets of files that you want to manage with AFS, you would create two Sync Groups and add different endpoints to each. A Storage Sync Service can host as many Sync Groups as you need.  

### Registered Server
The Registered Server object represents a trust-relationship between your server (or cluster) and the Storage Sync Service. You can register as many servers to a Storage Sync Service instance as you like. However, a server (or cluster) can only be registered with one Storage Sync Service at any given time.

### Azure File Sync agent
The Azure File Sync agent is a downloadable package which enables a Windows Server to be synchronized with an Azure File share. The Azure File Sync agent consists of three main components: 
- **FileSyncSvc.exe**: the background Windows service responsible for monitoring changes on Server Endpoints and initiating sync sessions to Azure.
- **StorageSync.sys**: the Azure File Sync file system filter, responsible tiering files to Azure Files (when cloud tiering is enabled).
- **PowerShell management cmdlets**: PowerShell cmdlets for interacting with the Microsoft.StorageSync Azure Resource Provider. These can be found at the following locations (by default):
    - C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll
    - C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll

### Server Endpoint
A Server Endpoint represents a specific location on a Registered Server, such as a folder on a server volume or the root of the volume. Multiple Server Endpoints can exist on the same volume if their namespaces are not overlapping (e.g. F:\sync1 and F:\sync2). You can configure cloud tiering policies individually for each Server Endpoint. If you add a server location with an existing set of files as a Server Endpoint to a Sync Group, those files will be merged with any other files already on other endpoints in the Sync Group.

### Cloud Endpoint
A Cloud Endpoint is an Azure File share that is part of a Sync Group. The entire Azure File share syncs and an Azure File share can only be a member of one Cloud Endpoint, and therefore, one Sync Group. If you add an Azure File share with an existing set of files as a Cloud Endpoint to a Sync Group, those files will be merged with any other files already on other endpoints in the Sync Group.

> [!Important]  
> Azure File Sync supports making changes to the Azure File share directly, however note that any changes made on the Azure File share first need to be discovered by an Azure File Sync change detection job, which is only initatiated for a Cloud Endpoint once every 24 hours. See the [Azure Files FAQ](storage-files-faq.md#afs-change-detection) for more information.

### Cloud tiering 
Cloud tiering is an optional feature of Azure File Sync, which enables infrequently used or access files to be tiered to Azure Files. When a file is tiered, the Azure File Sync file system filter (StorageSync.sys) replaces the file locally with a pointer, or reparse point, representing a URL to the file in Azure Files. A tiered file has the "offline" attribute set in NTFS, so third party applications can identify tiered files. When a user opens a tiered file, the Azure File Sync seamlessly recalls the file data from Azure Files without the user needing to know the file is not stored locally on the system. This functionality is also known as Hierarchical Storage Management (HSM).

## Azure File Sync Interoperability 
This section covers Azure File Sync interoperability with Windows Server features and roles and third party solutions.

### Supported versions of Windows Server
At present, the supported versions of Windows Server by Azure File Sync are:

| Version | Supported SKUs | Supported Deployment Options |
|---------|----------------|------------------------------|
| Windows Server 2016 | Datacenter and Standard | Full (Server with a UI) |
| Windows Server 2012 R2 | Datacenter and Standard | Full (Server with a UI) |

Future versions of Windows Server will be added as they are released, and older versions of Windows may be added based on user feedback.

> [!Important]  
> We recommend keeping all Windows Servers used with Azure File Sync up-to-date with the latest updates from Windows Update. 

### File system features
| Feature | Support Status | Notes |
|---------|----------------|-------|
| Access Control Lists (ACLs) | Fully supported | Windows ACLs are preserved by Azure File Sync, and are enforced by Windows Server on Server Endpoints. Windows ACLs are not (yet) supported by Azure Files if Files are accessed directly in the cloud. |
| Hard Links | Skipped | |
| Symbolic Links | Skipped | |
| Mount points | Partially supported | Mount points may be the root of a Server Endpoint, but will be skipped if contained in Server Endpoint's namespace. |
| Junctions | Skipped | |
| Reparse points | Skipped | |
| NTFS Compression | Fully supported | |
| Sparse files | Fully supported | Sparse files will sync (not blocked), but will sync to the cloud as a full file. If the file contents change in the cloud (or on another server), the file will no longer be sparse, when the change is downloaded |
| Alternate Data Streams (ADS) | Preserved, but not synced | |

> [!Note]  
> Only NTFS volumes are supported.

### Failover Clustering
Windows Server Failover Clustering is supported by Azure File Sync for the "File Server for general use" deployment option. Failover Clustering is not supported on "Scale-Out File Server for application data" (SOFS) or on Clustered Shared Volumes (CSV).

> [!Note]  
> The Azure File Sync agent must be installed on every node in a Failover Cluster for sync to work properly.

### Windows Server Data Deduplication
For volumes without cloud tiering enabled, Azure File Sync supports Data Deduplication being enabled on the volume. We do not support interop between Azure File Sync with cloud tiering enabled and Data Deduplication at this time.

### Anti-virus solutions
Because anti-virus works by scanning files for known malicious code, anti-virus may cause the recall of tiered files. Since tiered files have the "offline" attribute set, we recommend consulting with your software vendor regarding how to configure their solution to skip reading offline files. 

The following solutions are known to support skipping offline files:

- [Symantec Endpoint Protection](https://support.symantec.com/en_US/article.tech173752.html)
- [McAfee EndPoint Security](https://kc.mcafee.com/resources/sites/MCAFEE/content/live/PRODUCT_DOCUMENTATION/26000/PD26799/en_US/ens_1050_help_0-00_en-us.pdf) (see "Scan only what you need to" section on page 90)
- [Kaspersky Anti-Virus](https://support.kaspersky.com/4684)
- [Sophos Endpoint Protection](https://community.sophos.com/kb/en-us/40102)
- [TrendMicro OfficeScan](https://success.trendmicro.com/solution/1114377-preventing-performance-or-backup-and-restore-issues-when-using-commvault-software-with-osce-11-0#collapseTwo) 

### Backup solutions
Like anti-virus solutions, backup solutions may cause the recall of tiered files. We recommend using a cloud backup solution to backup the Azure File share rather than using an on-premises backup product.

### Encryption solutions
Support for encryption solutions depends on how they are implemented. Azure File Sync is known to work with:

- BitLocker encryption
- Azure RMS (and legacy Active Directory RMS)

Azure File Sync is known not to work with:

- NTFS Encrypted File System (EFS)

In general, Azure File Sync should be able to interop with encryption solutions that sit below the file system, such as BitLocker, and solutions implemented in the file format, such as BitLocker, but no special interop has been made to interop with solutions that sit above the file system (like NTFS Encrypted File System).

### Other Hierarchical Storage Management (HSM) solutions
No other Hierarchical Storage Management solution should be used with Azure File Sync.

## Region availability
Azure File Sync is only available in the following regions in Preview:

| Region | Datacenter location |
|--------|---------------------|
| West US | California, USA |
| West Europe | Netherlands |
| South East Asia | Singapore |
| Australia East | New South Wales, Australia |

In Preview, we only support sync with an Azure File share in the same region as the Storage Sync Service.

## Azure File Sync agent update policy
[!INCLUDE [storage-sync-files-agent-update-policy](../../../includes/storage-sync-files-agent-update-policy.md)]

## Next steps
* [Planning for an Azure Files deployment](storage-files-planning.md)
* [Deploying Azure Files](storage-files-deployment-guide.md)
* [Deploying Azure File Sync](storage-sync-files-deployment-guide.md)
