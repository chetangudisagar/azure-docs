---
title: Azure Storage Scalability and Performance Targets | Microsoft Docs
description: Learn about the scalability and performance targets for Azure Storage, including capacity, request rate, and inbound and outbound bandwidth for both standard and premium storage accounts. Understand performance targets for partitions within each of the Azure Storage services.
services: storage
documentationcenter: na
author: tamram
manager: timlt
editor: tysonn

ms.assetid: be721bd3-159f-40a1-88c1-96418537fe75
ms.service: storage
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage
ms.date: 07/12/2017
ms.author: tamram

---
# Azure Storage Scalability and Performance Targets
## Overview
This topic describes the scalability and performance topics for Microsoft Azure Storage. For a summary of other Azure limits, see [Azure Subscription and Service Limits, Quotas, and Constraints](../../azure-subscription-service-limits.md).

> [!NOTE]
> All storage accounts run on the new flat network topology and support the scalability and performance targets outlined below, regardless of when they were created. For more information on the Azure Storage flat network architecture and on scalability, see [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx).
> 
> [!IMPORTANT]
> The scalability and performance targets listed here are high-end targets, but are achievable. In all cases, the request rate and bandwidth achieved by your storage account depends upon the size of objects stored, the access patterns utilized, and the type of workload your application performs. Be sure to test your service to determine whether its performance meets your requirements. If possible, avoid sudden spikes in the rate of traffic and ensure that traffic is well-distributed across partitions.
> 
> When your application reaches the limit of what a partition can handle for your workload, Azure Storage will begin to return error code 503 (Server Busy) or error code 500 (Operation Timeout) responses. When this occurs, the application should use an exponential backoff policy for retries. The exponential backoff allows the load on the partition to decrease, and to ease out spikes in traffic to that partition.
> 
> 

If the needs of your application exceed the scalability targets of a single storage account, you can build your application to use multiple storage accounts, and partition your data objects across those storage accounts. See [Azure Storage Pricing](https://azure.microsoft.com/pricing/details/storage/) for information on volume pricing.

## Scalability targets for a storage account
[!INCLUDE [azure-storage-limits](../../../includes/azure-storage-limits.md)]

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

## Azure Blob storage scale targets
[!INCLUDE [storage-blob-scale-targets](../../../includes/storage-blob-scale-targets.md)]

## Azure Files scale targets
For more information on the scale and performance targets for Azure Files and Azure File Sync, please see [Azure Files scalability and performance targets](../files/storage-files-scale-targets.md).

[!INCLUDE [storage-files-scale-targets](../../../includes/storage-files-scale-targets.md)]

### Azure File Sync scale targets
[!INCLUDE [storage-sync-files-scale-targets](../../../includes/storage-sync-files-scale-targets.md)]

## Azure Queue storage scale targets
[!INCLUDE [storage-queues-scale-targets](../../../includes/storage-queues-scale-targets.md)]

## Azure Table storage scale targets
[!INCLUDE [storage-table-scale-targets](../../../includes/storage-tables-scale-targets.md)]

<!-- conceptual info about disk limits -- applies to unmanaged and managed -->
## Scalability targets for virtual machine disks
[!INCLUDE [azure-storage-limits-vm-disks](../../../includes/azure-storage-limits-vm-disks.md)]

See [Windows VM sizes](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) or [Linux VM sizes](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) for additional details.

## Managed virtual machine disks

[!INCLUDE [azure-storage-limits-vm-disks-managed](../../../includes/azure-storage-limits-vm-disks-managed.md)]

## Unmanaged virtual machine disks
[!INCLUDE [azure-storage-limits-vm-disks-standard](../../../includes/azure-storage-limits-vm-disks-standard.md)]

[!INCLUDE [azure-storage-limits-vm-disks-premium](../../../includes/azure-storage-limits-vm-disks-premium.md)]

## See Also
* [Storage Pricing Details](https://azure.microsoft.com/pricing/details/storage/)
* [Azure Subscription and Service Limits, Quotas, and Constraints](../../azure-subscription-service-limits.md)
* [Premium Storage: High-Performance Storage for Azure Virtual Machine Workloads](../storage-premium-storage.md)
* [Azure Storage Replication](../storage-redundancy.md)
* [Microsoft Azure Storage Performance and Scalability Checklist](../storage-performance-checklist.md)
* [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

