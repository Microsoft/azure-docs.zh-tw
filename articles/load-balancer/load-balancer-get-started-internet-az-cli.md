---
title: 使用 Azure CLI 建立具有區域備援公用 IP 位址前端的公用 Load Balancer Standard | Microsoft Docs
description: 了解如何使用 Azure CLI 建立具有區域備援公用 IP 位址前端的公用 Load Balancer Standard
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2018
ms.author: kumud
ms.openlocfilehash: 23c9d1b56c9a7c49b2cabe434693797fc2d23dfa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967671"
---
#  <a name="create-a-public-load-balancer-standard-with-zone-redundant-frontend-using-azure-cli"></a>使用 Azure CLI 建立具有區域備援前端的公用 Load Balancer Standard

本文會逐步說明如何使用 Public IP Standard 位址來建立具有區域備援前端的公用 [Load Balancer Standard](https://aka.ms/azureloadbalancerstandard)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝並使用 CLI，請確定您已安裝最新的 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)，並使用 [az login](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az_login) 來登入 Azure 帳戶。

> [!NOTE]
 針對精選的 Azure 資源和區域及 VM 大小系列有提供「可用性區域」支援。 如需如何開始使用，以及有哪些 Azure 資源、區域和 VM 大小系列可供用來試用可用性區域，請參閱[可用性區域概觀](https://docs.microsoft.com/azure/availability-zones/az-overview)。 如需支援，您可以透過 [StackOverflow](https://stackoverflow.com/questions/tagged/azure-availability-zones) 與我們聯繫或[開啟 Azure 支援票證](../azure-supportability/how-to-create-azure-support-request.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 


## <a name="create-a-resource-group"></a>建立資源群組

使用下列命令建立資源群組：

```azurecli-interactive
az group create --name myResourceGroupSLB --location westeurope
```

## <a name="create-a-public-ip-standard"></a>建立 Public IP Standard

使用下列命令建立 Public IP Standard：

```azurecli-interactive
az network public-ip create --resource-group myResourceGroupSLB --name myPublicIP --sku Standard
```

## <a name="create-a-load-balancer"></a>建立負載平衡器

使用下列命令，以您在上一個步驟中建立的標準公用 IP 來建立公用 Load Balancer Standard：

```azurecli-interactive
az network lb create --resource-group myResourceGroupSLB --name myLoadBalancer --public-ip-address myPublicIP --frontend-ip-name myFrontEnd --backend-pool-name myBackEndPool --sku Standard
```

## <a name="create-an-lb-probe-on-port-80"></a>在連接埠 80 上建立 LB 探查

使用下列命令建立負載平衡器健康情況探查：

```azurecli-interactive
az network lb probe create --resource-group myResourceGroupSLB --lb-name myLoadBalancer \
  --name myHealthProbe --protocol tcp --port 80
```

## <a name="create-an-lb-rule-for-port-80"></a>為連接埠 80 建立 LB 規則

使用下列命令建立負載平衡器規則：

```azurecli-interactive
az network lb rule create --resource-group myResourceGroup --lb-name myLoadBalancer --name myLoadBalancerRuleWeb \
  --protocol tcp --frontend-port 80 --backend-port 80 --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool --probe-name myHealthProbe
```

## <a name="next-steps"></a>後續步驟
- 深入了解[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。



