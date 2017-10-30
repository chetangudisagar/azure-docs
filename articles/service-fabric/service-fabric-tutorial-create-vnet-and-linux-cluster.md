---
title: Create a Linux Service Fabric cluster in Azure | Microsoft Docs
description: Learn how to deploy a Linux Service Fabric cluster into an existing Azure virtual network using Azure CLI.
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''

ms.assetid:
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/26/2017
ms.author: ryanwi

---

# Deploy a Service Fabric Linux cluster into an Azure virtual network
This tutorial is part one of a series. You will learn how to deploy a Linux Service Fabric cluster into an existing Azure virtual network (VNET) and sub-net using Azure CLI. When you're finished, you have a cluster running in the cloud that you can deploy applications to. To create a Windows cluster using PowerShell, see [Create a secure Windows cluster on Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Create a VNET in Azure using Azure CLI
> * Create a secure Service Fabric cluster in Azure using Azure CLI
> * Secure the cluster with an X.509 certificate
> * Connect to the cluster using Service Fabric CLI
> * Remove a cluster

In this tutorial series you learn how to:
> [!div class="checklist"]
> * Create a secure cluster on Azure
> * [Deploy API Management with Service Fabric](service-fabric-tutorial-deploy-api-management.md)

## Prerequisites
Before you begin this tutorial:
- If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Install the [Service Fabric CLI](service-fabric-cli.md)
- Install the [Azure CLI 2.0](/cli/azure/install-azure-cli)

The following procedures create a five-node Service Fabric cluster. To calculate cost incurred by running a Service Fabric cluster in Azure use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/).

## Sign-in to Azure and select your subscription
This guide uses Azure CLI. When you start a new session, sign in to your Azure account and select your subscription before you execute Azure commands.
 
Run the following script to sign in to your Azure account select your subscription:

```azurecli
az login
az account set --subscription <guid>
```

## Create a resource group
Create a new resource group for your deployment and give it a name and a location.

```azurecli
ResourceGroupName="sflinuxclustergroup"
Location="southcentralus"
az group create --name $ResourceGroupName --location $Location
```

## Deploy the network topology
Next, set up the network topology to which API Management and the Service Fabric cluster will be deployed. The [network.json][network-arm] Resource Manager template is configured to create a virtual network (VNET) and also a subnet and network security group (NSG) for Service Fabric and a subnet and NSG for API Management. Learn more about VNETs, subnets, and NSGs [here](../virtual-network/virtual-networks-overview.md).

The [network.parameters.json][network-parameters-arm] parameters file contains the names of the subnets and NSGs that Service Fabric and API Management deploy to.  API Management is deployed in the [following tutorial](service-fabric-tutorial-deploy-api-management.md). For this guide, the parameter values do not need to be changed. The Service Fabric Resource Manager templates use these values.  If the values are modified here, you must modify them in the other Resource Manager templates used in this tutorial and the [Deploy API Management tutorial](service-fabric-tutorial-deploy-api-management.md). 

Download the following Resource Manager template and parameters file:
- [network.json][network-arm]
- [network.parameters.json][network-parameters-arm]

Use the following script to deploy the Resource Manager template and parameter files for the network setup:

```azurecli
az group deployment create \
    --name VnetDeployment \
    --resource-group $ResourceGroupName \
    --template-file network.json \
    --parameters @network.parameters.json
```
<a id="createvaultandcert" name="createvaultandcert_anchor"></a>
## Deploy the Service Fabric cluster
Once the network resources have finished deploying, the next step is to deploy a Service Fabric cluster to the VNET in the subnet and NSG designated for the Service Fabric cluster. Deploying a cluster to an existing VNET and subnet (deployed previously in this article) requires a Resource Manager template.  For more information, see [Create a cluster by using Azure Resource Manager](service-fabric-cluster-creation-via-arm.md). For this tutorial series, the template is pre-configured to use the names of the VNET, subnet, and NSG that you set up in a previous step.  Download the following Resource Manager template and parameters file:
- [linuxcluster.json][cluster-arm]
- [linuxcluster.parameters.json][cluster-parameters-arm]

Fill in the empty **clusterName**, **adminUserName**, and **adminPassword** parameters in the *linuxcluster.parameters.json* file for your deployment.  Leave the **certificateThumbprint**, **certificateUrlValue**, and **sourceVaultValue** parameters blank if you want to create a self-signed certificate.  If you have an existing certificate previously uploaded to a key vault, fill in those parameter values.

Use the following script to deploy the cluster using the Resource Manager template and parameter files.  A self-signed certificate is created in the specified key vault and is used to secure the cluster.  The certificate is also downloaded locally.

```azurecli
Password="q6D7nN%6ck@6"
Subject="aztestcluster.southcentralus.cloudapp.azure.com"
VaultName="linuxclusterkeyvault"
az group create --name $ResourceGroupName --location $Location

az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName  \
   --template-file linuxcluster.json --parameter-file linuxcluster.parameters.json

```

## Connect to the secure cluster
Connect to the cluster using the Service Fabric CLI `sfctl cluster select` command using your key.  Note, only use the **--no-verify** option for a self-signed certificate.

```azurecli
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

Check that you are connected and the cluster is healthy using the `sfctl cluster health` command.

```azurecli
sfctl cluster health
```

## Clean up resources
The other articles in this tutorial series use the cluster you just created. If you're not immediately moving on to the next article, you might want to delete the cluster to avoid incurring charges. The simplest way to delete the cluster and all the resources it consumes is to delete the resource group.

Log in to Azure and select the subscription ID with which you want to remove the cluster.  You can find your subscription ID by logging in to the [Azure portal](http://portal.azure.com). Delete the resource group and all the cluster resources using the [az group delete](/cli/azure/group?view=azure-cli-latest#az_group_delete) command.

```azurecli
az group delete --name $ResourceGroupName
```

## Next steps
In this tutorial, you learned how to:

> [!div class="checklist"]
> * Create a VNET in Azure using Azure CLI
> * Create a secure Service Fabric cluster in Azure using Azure CLI
> * Secure the cluster with an X.509 certificate
> * Connect to the cluster using Service Fabric CLI
> * Remove a cluster

Next, advance to the following tutorial to learn how to deploy API Management with Service Fabric.
> [!div class="nextstepaction"]
> [Deploy API Managment](service-fabric-tutorial-deploy-api-management.md)


[network-arm]:https://github.com/Azure-Samples/service-fabric-api-management/blob/master/network.json
[network-parameters-arm]:https://github.com/Azure-Samples/service-fabric-api-management/blob/master/network.parameters.json

[cluster-arm]:https://github.com/Azure-Samples/service-fabric-api-management/blob/master/linuxcluster.json
[cluster-parameters-arm]:https://github.com/Azure-Samples/service-fabric-api-management/blob/master/linuxcluster.parameters.json
