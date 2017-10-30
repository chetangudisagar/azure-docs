---
title: Delegating offers in Azure Stack | Microsoft Docs
description: Learn how to put other people in charge of creating offers and signing up users for you.
services: azure-stack
documentationcenter: ''
author: AlfredoPizzirani
manager: byronr
editor: ''

ms.assetid: 157f0207-bddc-42e5-8351-197ec23f9d46
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/25/2017
ms.author: alfredop

---
# Delegating offers in Azure Stack

*Applies to: Azure Stack integrated systems and Azure Stack Development Kit*

As the Azure Stack operator, you often want to put other people in charge
of creating offers and signing up users for you. For example, if you are a service provider and you want resellers to sign up
customers and manage them on your behalf. It can also happen in an
enterprise if you are part of a central IT group and want divisions or
subsidiaries to sign up users without your intervention.

Delegation helps you with these tasks, helping you to reach and manage
more users than you would be able to do directly. The following
illustration shows one level of delegation, but Azure Stack supports
multiple levels. Delegated providers can in turn delegate to other
providers, up to five levels.

![](media/azure-stack-delegated-provider/image1.png)

Azure Stack operators can delegate the creation of offers and users
to other users by using the delegation functionality.

## Roles and steps in delegation
To understand delegation, keep in mind that there are three roles
involved:

* The Azure Stack operator manages the Azure Stack
  infrastructure, creates an offer template, and delegates others to
  offer it to their users.
* The delegated Azure Stack operators are called **delegated providers**. They can
  belong to other organizations (such as other Azure Active
  Directory users).
* **Users** sign up for the offers and use them for managing their
  workloads, creating VMs, storing data, etc.

As shown in the following graphic, there are two steps in setting up
delegation.

1. **Identify the delegated providers** by subscribing them to an
   offer based on a plan that contains only the subscriptions service.
   Users who subscribe to this offer acquire some of the Azure Stack
   operator’s capabilities, including the ability to extend offers
   and sign users up for them.
2. **Delegate an offer to the delegated provider**, this offer functions as
   a template for what the delegated provider can offer. The delegated
   provider is now able to take the offer, choose a name for it (but
   not change its services and quotas), and offer it to customers.

![](media/azure-stack-delegated-provider/image2.png)

To act as delegated providers, users need to establish a relationship
with the main provider; in other words, they need to create a
subscription. In this scenario, this subscription identifies the
delegated providers as having the right to present offers on behalf of
the main provider.

Once this relationship is established, the Azure Stack operator can
delegate an offer to the delegated provider. The delegated provider is
now able to take the offer, rename it (but not change its substance),
and offer it to its customers.

The following sections describe how to establish a delegated provider, delegate an offer, and verify that
users can sign up for it.

## Set up roles

To see a delegated provider at work, you need additional Azure
Active Directory accounts in addition to your Azure Stack operator
account. If you do not have them, create the two accounts. The accounts
can belong to any AAD user. We refer to them as the delegated
provider (DP) and the user.

| **Role** | **Organizational rights** |
| --- | --- |
| Delegated Provider |User |
| User |User |

## Identify the delegated providers
1. Sign in as Azure Stack operator.
2. Create the offer that enables users to become
   delegated providers. This requires that you create a plan and an
   offer based on it:
   
   a.  [Create a plan](azure-stack-create-plan.md).
       This plan should include only the subscriptions service. In this
       article, we use a plan called PlanForDelegation.
   
   b.  [Create an offer](azure-stack-create-offer.md)
       based on this plan. In this article, we use an offer
       called OfferToDP.
   
   c.  Once the creation of the offer is complete, add the delegated provider as a subscriber to this offer by clicking
       **Subscriptions** &gt; **Add** &gt; **New Tenant Subscription**.
   
   ![](media/azure-stack-delegated-provider/image3.png)

> [!NOTE]
> As with all Azure Stack offers, you have the option of making
> the offer public and letting users sign up for it, or keeping it
> private and have the Azure Stack operator manage the sign-up. Delegated
> providers are usually a small group and you want to control who is
> admitted to it, so keeping this offer private makes sense in most
> cases.
> 
> 

## Azure Stack operator creates the delegated offer

You have now established your delegated provider. The next step is to
create the plan and offer that you are going to delegate, and which your
customers will use. You should define this offer exactly as you want the
customers to see it, because the delegated provider will not be able to
change the plans and quotas it includes.

1. As Azure Stack operator, [create a
   plan](azure-stack-create-plan.md)
   and [an
   offer](azure-stack-create-offer.md)
   based on it. For this article, we use an offer
   called DelegatedOffer.
   
   > [!NOTE]
   > This offer does not have to be public. It can be made
   > public if you choose, but, in most cases, you only want delegated
   > providers to have access to it. Once you delegate a private offer as
   > described in the following steps, the delegated provider has
   > access to it.
   > 
   > 
1. Delegate the offer. Go to DelegatedOffer, and in the Settings pane,
   click **Delegated Providers** &gt; **Add**.
2. Select the delegated provider’s subscription from the drop-down list
   box and click **Delegate**.

> ![](media/azure-stack-delegated-provider/image4.png)
> 
> 

## Delegated provider customizes the offer

Sign in to the user portal as the delegated provider and create a new offer using the delegated offer as a template.

1. Click **New** &gt; **Tenant Offers + Plans** &gt; **Offer**.

    ![](media/azure-stack-delegated-provider/image5.png)


1. Assign a name to the offer. Here we choose ResellerOffer. Select the delegated offer to base it on and then click **Create**.
   
   ![](media/azure-stack-delegated-provider/image6.png)

    >[!NOTE] 
    > Note the difference compared to offer creation as experienced by the Azure Stack operator. The delegated provider does not construct the offer from base plans and add-on plans; they can only choose from offers that have been delegated to them, and can't make changes to those offers.

1. Make the offer public by clicking **Browse** &gt; **Offers**, selecting the offer, and clicking **Change State**.
2. The delegated provider exposes these offers through their own portal URL. These offers are visible only through the delegated portal. To find and change this URL:
   
    a.  Click **Browse**&gt; **More services**&gt; **Subscriptions**&gt; Select the delegated provider subscription, in our case its *DPSubscription*&gt; **Properties**.
   
    b.  Copy the portal URL to a separate location, such as Notepad.
   
    ![](media/azure-stack-delegated-provider/dpportaluri.png)  
   
   You have now completed the creation of a delegated offer as a delegated provider. Sign out as the delegated provider. Close the browser tab you have been using.

## Sign up for the offer
1. In a new browser window, go to the delegated portal URL you saved in
   the previous step. Sign in to the portal as user. Note: Use
   the delegated portal for this step. The delegated offer are not
   visible otherwise.
2. In the dashboard, click **Get a subscription**. You will see that
   only the delegated offers created by the delegated provider are
   presented to the user:

> ![](media/azure-stack-delegated-provider/image8.png)
> 
> 

This concludes the process of offer delegation. The user can now sign up
for this offer by getting a subscription for it.

## Multiple-tier delegation

Multiple-tier delegation allows the delegated provider to delegate the
offer to other entities. This allows, for example, the creation of
deeper reseller channels, in which the provider managing Azure Stack
delegates an offer to a distributor, who in turn delegates to reseller.
Azure Stack supports up to five levels of delegation.

To create multiple tiers of offer delegation, the delegated provider in
turn delegates the offer to the next provider. The process is the same
for the delegated provider as it was for the Azure Stack operator (see
[Azure Stack operator creates the delegated offer](#cloud-operator-creates-the-delegated-offer)).

## Next steps
[Provision a VM](azure-stack-provision-vm.md)

