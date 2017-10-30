---
title: Design your network infrastructure for disaster recovery with Azure Site Recovery| Microsoft Docs
description: This article discusses network design considerations for Azure Site Recovery
services: site-recovery
documentationcenter: ''
author: prateek9us
manager: jwhit
editor: ''

ms.assetid: ce787731-d930-4f00-a309-e2fbc2e1f53b
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 10/19/2017
ms.author: pratshar

---
# Designing your network for disaster recovery

This article discusses network considerations when using [Azure Site Recovery](site-recovery-overview.md) for disaster recovery from on-premises to Azure, or to a secondary on-premises site. It focuses on defining IP address ranges and subnets after failover to a secondary location.

## Overview

Site Recovery is a Microsoft Azure service that orchestrates the protection and recovery of virtualized applications for the purpose of business continuity disaster recovery (BCDR).

In a world of 24/7 connectivity, it's important to keep your business infrastructure and applications up and running. The purpose of BCDR is to restore failed components, so that your organization can quickly resume normal operations. Developing disaster recovery strategies to deal with unlikely events is very challenging. This is due to the inherent difficulty of predicting the future, particularly for improbable events. And, because of the high cost of maintaining adequate measures of protection against far-reaching catastrophes.

Crucial for BCDR planning, a Recovery Time Objective (RTO) and Recovery Point Objective (RPO) must be defined in your BCDR plan. When a disaster strikes a data center, you can quickly (low RTO) bring online their replicated virtual machines located in either the secondary data center or Microsoft Azure with minimum data loss (low RPO).

Failover is made possible by Site Recovery, which initially copies designated virtual machines from the primary data center to the secondary data center or to Azure (depending on the scenario), and then periodically refreshes the replicas. During infrastructure planning, network design should be considered as potential bottleneck that can prevent you from meeting company RTO and RPO objectives.  

When administrators are planning to deploy a disaster recovery solution, one of the key questions in their minds is how the virtual machine would be reachable after the failover is completed. Site Recovery allows the administrator to choose the network to which a virtual machine would be connected to after failover. If the primary site is Azure, or it is an on-premises site managed by a VMM server, then this is achieved by using network mapping. Learn more about [Network Mapping in Azure to Azure DR](site-recovery-network-mapping-azure-to-azure.md) and [Network Mapping from VMM](site-recovery-network-mapping.md)


While designing the network for the recovery site, the administrator has two choices:

* Use a different IP address range for the network at recovery site. In this scenario the virtual machine after failover will get a new IP address and the administrator would have to do a DNS update. 
* Use same IP address range for the network at the recovery site. In certain scenarios administrators prefer to retain the IP addresses that they have on the primary site even after the failover. In a normal scenario an administrator would have to update the routes to indicate the new location of the IP addresses. But in the scenario where a stretched subnet is deployed between the primary and the recovery sites, retaining the IP addresses for the virtual machines becomes an attractive option. Stretching a subnet from an on-premises network to an Azure network or between two Azure networks is not possible.  

When administrators are planning to deploy a disaster recovery solution, one of the key questions in their mind is how the applications will be reachable after the failover is completed. Modern applications are almost always dependent on networking to some degree, so physically moving a service from one site to another represents a networking challenge. There are two main ways that this problem is dealt with in disaster recovery solutions. The first approach is to maintain fixed IP addresses. Despite the services moving and the hosting servers being in different physical locations, applications take the IP address configuration with them to the new location. The second approach involves completely changing the IP address during the transition into the recovered site. Each approach has several implementation variations which are summarized below.

While designing the network for the recovery site, the administrator has two choices:

## Option 1: Retain IP addresses
From a disaster recovery process perspective, using fixed IP addresses appears to be the easiest method to implement, but it has a number of potential challenges, which in practice makes it the least popular approach. Azure Site Recovery provides the capability to retain the IP addresses in all scenarios. Before one decides to retain IP, appropriate thought should be given to the constraints it imposes on the failover capabilities. Let us look at the factors that can help you to make a decision to retain IP addresses, or not. This can be achieved in two ways, by using a stretched subnet or by doing a full subnet failover.

### Stretched subnet
Here the subnet is made available simultaneously in both primary and DR locations. In simple terms this means you can move a server and its IP (Layer 3) configuration to the second site and the network will route the traffic to the new location automatically. This is trivial to deal with from a server perspective but it has a number of challenges:

* From a Layer 2 (data link layer) perspective, it requires networking equipment that can manage a stretched VLAN, but this has become less of a problem as it is now widely available. The second and more difficult problem is that by stretching the VLAN the potential fault domain is extended to both sites, essentially becoming a single point of failure. While this is an unlikely occurrence, it has happened that a broadcast storm started but could not be isolated. We have found mixed opinions about this last issue and have seen many successful implementations as well as “we won't implement this technology here.”
* Stretched subnet is not possible if you are using Microsoft Azure as the DR site.

### Subnet failover
It is possible to implement subnet failover to obtain the benefits of the stretched subnet solution described above without stretching the subnet across multiple sites. Here any given subnet would be present at Site 1 or Site 2, but never at both sites simultaneously. In order to maintain the IP address space in the event of a failover, it is possible to programmatically arrange for the router infrastructure to move the subnets from one site to another. In a failover scenario the subnets would move with the associated protected VMs. The main drawback to this approach is in the event of a failure you have to move the whole subnet. This might be OK, but it could affect failover granularity.

Let’s examine how a fictional enterprise named Contoso is able to replicate its VMs to a recovery location while failing over the entire subnet. Let's first look at how Contoso is able to manage their subnets while replicating VMs between two on-premises locations, and then we'll discuss how subnet failover works when [Azure is used as the disaster recovery site](#failover-to-azure).

#### Fail over from on-premises to Azure 
Azure Site Recovery allows Azure to be used as a disaster recovery site for your virtual machines.  

Let’s examine a scenario where a fictional company named Woodgrove Bank has on-premises infrastructure hosting their line-of-business applications, and they are hosting their mobile applications on Azure. Connectivity between Woodgrove Bank VMs in Azure and on-premises servers is provided by a site-to-site (S2S) Virtual Private Network (VPN) or ExpressRoute. Site-to-site VPN allows Woodgrove Bank’s virtual network in Azure to be seen as an extension of Woodgrove Bank’s on-premises network. This communication is enabled by site-to-site VPN between Woodgrove Bank edge and Azure virtual network. Now Woodgrove wants to use Site Recovery to replicate its workloads running primary Azure region to another Azure region. This option meets the needs of Woodgrove, which wants an economical DR option and is able to store data in public cloud environments. Woodgrove has to deal with applications and configuration, which depend on hard-coded IP addresses, and so have a requirement to retain IP addresses for their applications after failing over to another region in Azure.

Woodgrove has decided to assign IP addresses from IP address range (172.16.1.0/24, 172.16.2.0/24) to its resources running in Azure.

For Woodgrove to be able to replicate its virtual machines to Azure while retaining the IP addresses, an Azure Virtual Network needs to be created. It should be an extension of the on-premises network so that applications can fail over from the on-premises site to Azure seamlessly. Azure allows you to add site-to-site as well as point-to-site VPN connectivity to the virtual networks created in Azure. When setting up your site-to-site connection, Azure network allows you to route traffic to the on-premises location (Azure calls it local-network) only if the IP address range is different from the on-premises IP address range, because Azure doesn’t support stretching subnets.  This means that if you have a subnet 192.168.1.0/24 on-premises, you can’t add a local-network 192.168.1.0/24 in the Azure network. This is expected because Azure doesn’t know that there are no active VMs in the subnet and that the subnet is being created only for DR purposes. To be able to correctly route network traffic out of an Azure network the subnets in the network and the local-network must not conflict.

![Before Subnet Failover](./media/site-recovery-network-design/network-design7.png)

Before failover

To help Woodgrove fulfill their business requirements, we need to implement the following workflows:

* Create an additional network, let us call it Recovery Network, where the failed-over virtual machines would be created.
* To ensure that the IP for a VM is retained after a failover, go to the Configure tab under VM properties, specify the same IP that the VM has on-premises, and then click Save. When the VM is failed over, Azure Site Recovery will assign the provided IP to the virtual machine.

![Network properties](./media/site-recovery-network-design/network-design8.png)

Once the failover is triggered and the virtual machines are created in the Recovery Network with the desired IP, connectivity to this network can be established using a [Vnet to Vnet Connection](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md). If required this action can be scripted.  As we discussed in the previous section about subnet failover, even in the case of failover to Azure, routes would have to be appropriately modified to reflect that 192.168.1.0/24 has now moved to Azure.

![After Subnet Failover](./media/site-recovery-network-design/network-design9.png)

After failover

If you don't have a 'Azure Network' as shown in the picture above. You can create a site-to-site VPN connection between your 'Primary Site' and 'Recovery Network' after the failover.  


#### Fail over to a secondary on-premises site
Let us look at a scenario where we want to retain the IP of each of the VMs and fail-over the complete subnet together. The primary site has applications running in subnet 192.168.1.0/24. When the failover happens, all the virtual machines that are part of this subnet will be failed over to the recovery site and retain their IP addresses. Routes will have to be appropriately modified to reflect the fact that all the virtual machines belonging to subnet 192.168.1.0/24 have now moved to the recovery site.

In the following illustration the routes between primary site and recovery site, third site and primary site, and third site and recovery site will have to be appropriately modified.

The following pictures show the subnets before the failover. Subnet 192.168.0.1/24 is active on the Primary Site before the failover and becomes active of the Recovery Site after the failover

![Before Failover](./media/site-recovery-network-design/network-design2.png)

Before failover

The picture below shows networks and subnets after failover.

![After Failover](./media/site-recovery-network-design/network-design3.png)

After failover

If your secondary site is on-premises, and you are using a VMM server to manage it, then when enabling protection for a specific virtual machine, Site Recovery will allocate networking resources according to the following workflow:

* Site Recovery allocates an IP address for each network interface on the virtual machine from the static IP address pool defined on the relevant network for each System Center VMM instance.
* If the administrator defines the same IP address pool for the network on the recovery site as that of the IP address pool of the network on the primary site, while allocating the IP address to the replica virtual machine, Site Recovery would allocate the same IP address as that of the primary virtual machine.  The IP address is reserved in VMM, but not set as the failover IP address on the Hyper-v host. The failover IP address on a Hyper-v host is set just before the failover.


If the same IP address is not available, Site Recovery allocates some other available IP address from the defined IP address pool.

After the VM is enabled for protection you can use following sample script to verify the IP that has been allocated to the virtual machine. The same IP would be set as Failover IP and assigned to the VM at the time of failover:

        $vm = Get-SCVirtualMachine -Name <VM_NAME>
        $na = $vm[0].VirtualNetworkAdapters>
        $ip = Get-SCIPAddress -GrantToObjectID $na[0].id
        $ip.address  

> [!NOTE]
> In the scenario where virtual machines use DHCP, the management of IP addresses is completely outside the control of Site Recovery. An administrator has to ensure that the DHCP server serving the IP addresses on the recovery site can serve from the same range as that of the primary site.
>
>



## Option 2: Changing IP addresses
This approach seems to be the most prevalent based on what we have seen. It takes the form of changing the IP address of every VM that is involved in the failover. A drawback of this approach requires the incoming network to ‘learn’ that the application that was at IPx is now at IPy. Even if IPx and IPy are logical names, DNS entries typically have to be changed or flushed throughout the network, and cached entries in network tables have to be updated or flushed, therefore a downtime could be seen depending upon how the DNS infrastructure has been set up. These issues can be mitigated by using low TTL values in the case of intranet applications and using [Azure Traffic Manager with Site Recovery](http://azure.microsoft.com/blog/2015/03/03/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/), for internet-based applications

### Changing the IP addresses - Illustration
Let us look at the scenario where you are planning to use different IPs across the primary and the recovery sites. In the following example we also have a third site from where the applications hosted on primary or recovery site can be accessed.

![Different IP - Before Failover](./media/site-recovery-network-design/network-design10.png)


In the picture above there are some applications hosted in subnet 192.168.1.0/24 subnet on the primary site, and they have been configured to come up on the recovery site in subnet 172.16.1.0/24 after a failover. VPN connections/network routes have been configured appropriately so that all three sites can access each other.

As the picture below shows, after failing over one or more applications, they will be restored in the recovery subnet. In this case we are not constrained to fail over the entire subnet at the same time. No changes are required to reconfigure VPN or network routes. A failover and some DNS updates will make sure that applications remain accessible. If the DNS is configured to allow dynamic updates then the virtual machines would register themselves using the new IP once they start after a failover.

![Different IP - After Failover](./media/site-recovery-network-design/network-design11.png)


After failing-over the replica virtual machine might have an IP address that isn’t the same as the IP address of the primary virtual machine. Virtual machines will update the DNS server that they are using after they start. DNS entries typically have to be changed or flushed throughout the network, and cached entries in network tables have to be updated or flushed, so it is not uncommon to be faced with downtime while these state changes take place. This issue can be mitigated by:

* Using low TTL values for intranet applications.
* Using Azure Traffic Manger with Site Recovery for internet-based applications.
* Using the following script within your recovery plan to update the DNS Server to ensure a timely update (The script is not required if the Dynamic DNS registration is configured)

        param(
		string]$Zone,
        [string]$name,
        [string]$IP
        )
        $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
        $newrecord = $record.clone()
        $newrecord.RecordData[0].IPv4Address  =  $IP
        Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord

### Changing the IP addresses – disaster recovery to Azure
The [Networking Infrastructure Setup for Microsoft Azure as a Disaster Recovery Site](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/) blog post explains how to set up the required Azure networking infrastructure when retaining IP addresses isn’t a requirement. It starts with describing the application and then look at how to set up networking on-premises and on Azure and then concluding with how to do a test failover and a planned failover.

## Next steps
Learn about [network mapping](site-recovery-network-mapping.md).
