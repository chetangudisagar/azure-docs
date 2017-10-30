---
title: Configure Secure LDAP (LDAPS) in Azure AD Domain Services | Microsoft Docs
description: Configure Secure LDAP (LDAPS) for an Azure AD Domain Services managed domain
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: stevenpo
editor: curtand

ms.assetid: c6da94b6-4328-4230-801a-4b646055d4d7
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2017
ms.author: maheshu

---
# Configure secure LDAP (LDAPS) for an Azure AD Domain Services managed domain

## Before you begin
Ensure you've completed [Task 2 - export the secure LDAP certificate to a .PFX file](active-directory-ds-admin-guide-configure-secure-ldap-export-pfx.md).


## Task 3 - enable secure LDAP for the managed domain using the Azure portal
To enable secure LDAP, perform the following configuration steps:

1. Navigate to the **[Azure portal](https://portal.azure.com)**.

2. Search for 'domain services' in the **Search resources** search box. Select **Azure AD Domain Services** from the search result. The **Azure AD Domain Services** page lists your managed domain.

    ![Find managed domain being provisioned](./media/getting-started/domain-services-provisioning-state-find-resource.png)

2. Click the name of the managed domain (for example, 'contoso100.com') to see more details about the domain.

    ![Domain Services - provisioning state](./media/getting-started/domain-services-provisioning-state.png)

3. Click **Secure LDAP** on the navigation pane.

    ![Domain Services - Secure LDAP page](./media/active-directory-domain-services-admin-guide/secure-ldap-blade.png)

4. By default, secure LDAP access to your managed domain is disabled. Toggle **Secure LDAP** to **Enable**.

    ![Enable secure LDAP](./media/active-directory-domain-services-admin-guide/secure-ldap-blade-configure.png)
5. By default, secure LDAP access to your managed domain over the internet is disabled. Toggle **Allow secure LDAP access over the internet** to **Enable**, if desired. 

    > [!TIP]
    > If you enable secure LDAP access over the internet, we recommend setting up an NSG to lock down access to required source IP address ranges. See the instructions to [lock down LDAPS access to your managed domain over the internet](#task-5---lock-down-ldaps-access-to-your-managed-domain-over-the-internet).
    >

6. Click the folder icon following **.PFX file with secure LDAP certificate**. Specify the path to the PFX file with the certificate for secure LDAP access to the managed domain.

7. Specify the **Password to decrypt .PFX file**. Provide the same password you used when exporting the certificate to the PFX file.

8. When you are done, click the **Save** button.

9. You see a notification that informs you secure LDAP is being configured for the managed domain. Until this operation is complete, you cannot modify other settings for the domain.

    ![Configuring secure LDAP for the managed domain](./media/active-directory-domain-services-admin-guide/secure-ldap-blade-configuring.png)

> [!NOTE]
> It takes about 10 to 15 minutes to enable secure LDAP for your managed domain. If the provided secure LDAP certificate does not match the required criteria, secure LDAP is not enabled for your directory and you see a failure. For example, the domain name is incorrect, the certificate has already expired or expires soon. In this case, retry with a valid certificate.
>
>

<br>

## Task 4 - configure DNS to access the managed domain from the internet
> [!NOTE]
> **Optional task** - If you do not plan to access the managed domain using LDAPS over the internet, skip this configuration task.
>
>

Before you begin this task, ensure you have completed the steps outlined in [Task 3](#task-3---enable-secure-ldap-for-the-managed-domain-using-the-azure-portal-preview).

Once you have enabled secure LDAP access over the internet for your managed domain, you need to update DNS so that client computers can find this managed domain. At the end of task 3, an external IP address is displayed on the **Properties** tab in **EXTERNAL IP ADDRESS FOR LDAPS ACCESS**.

Configure your external DNS provider so that the DNS name of the managed domain (for example, 'ldaps.contoso100.com') points to this external IP address. In our example, we need to create the following DNS entry:

    ldaps.contoso100.com  -> 52.165.38.113

That's it - you are now ready to connect to the managed domain using secure LDAP over the internet.

> [!WARNING]
> Remember that client computers must trust the issuer of the LDAPS certificate to be able to connect successfully to the managed domain using LDAPS. If you are using a publicly trusted certification authority, you do not need to do anything since client computers trust these certificate issuers. If you are using a self-signed certificate, install the public part of the self-signed certificate into the trusted certificate store on the client computer.
>
>


## Task 5 - lock-down LDAPS access to your managed domain over the internet
> [!NOTE]
> **Optional task** - If you have not enabled LDAPS access to the managed domain over the internet, skip this configuration task.
>
>

Before you begin this task, ensure you have completed the steps outlined in [Task 3](#task-3---enable-secure-ldap-for-the-managed-domain-using-the-azure-portal-preview).

Exposing your managed domain for LDAPS access over the internet represents a security threat. The managed domain is reachable from the internet at the port used for secure LDAP (that is, port 636). Therefore, you can choose to restrict access to the managed domain to specific known IP addresses. For improved security, create a network security group (NSG) and associate it with the subnet where you have enabled Azure AD Domain Services.

The following table illustrates a sample NSG you can configure, to lock down secure LDAP access over the internet. The NSG contains a set of rules that allow inbound LDAPS access over TCP port 636 only from a specified set of IP addresses. The default 'DenyAll' rule applies to all other inbound traffic from the internet. The NSG rule to allow LDAPS access over the internet from specified IP addresses has a higher priority than the DenyAll NSG rule.

![Sample NSG to secure LDAPS access over the internet](./media/active-directory-domain-services-admin-guide/secure-ldap-sample-nsg.png)

**More information** - [Network security groups](../virtual-network/virtual-networks-nsg.md).

<br>

## Related content
* [Azure AD Domain Services - Getting Started guide](active-directory-ds-getting-started.md)
* [Administer an Azure AD Domain Services managed domain](active-directory-ds-admin-guide-administer-domain.md)
* [Administer Group Policy on an Azure AD Domain Services managed domain](active-directory-ds-admin-guide-administer-group-policy.md)
* [Network security groups](../virtual-network/virtual-networks-nsg.md)
* [Create a Network Security Group](../virtual-network/virtual-networks-create-nsg-arm-pportal.md)
