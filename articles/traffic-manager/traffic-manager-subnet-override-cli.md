---
title: 使用 Azure CLI Azure 流量管理員子網覆寫 |Microsoft Docs
description: 本文將協助您瞭解流量管理員子網覆寫如何用來覆寫流量管理員設定檔的路由方法，以透過預先定義的 IP 位址，將流量導向至端點對應的終端使用者 IP 位址。
services: traffic-manager
documentationcenter: ''
author: duongau
ms.topic: how-to
ms.service: traffic-manager
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 7a448afb85a35674921ce74a25eaf2a97430dc61
ms.sourcegitcommit: 0aec60c088f1dcb0f89eaad5faf5f2c815e53bf8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98201411"
---
# <a name="traffic-manager-subnet-override-using-azure-cli"></a>使用 Azure CLI 的流量管理員子網覆寫

流量管理員子網覆寫可讓您變更設定檔的路由方法。  新增覆寫會根據終端使用者的 IP 位址將流量導向至端點對應的預先定義 IP 範圍。 

## <a name="how-subnet-override-works"></a>子網覆寫的運作方式

將子網覆寫新增至流量管理員設定檔時，流量管理員會先檢查終端使用者的 IP 位址是否有子網覆寫。 如果找到一個，就會將使用者的 DNS 查詢導向至對應的端點。  如果找不到對應，流量管理員會切換回設定檔的原始路由方法。 

您可以將 IP 位址範圍指定為 CIDR 範圍 (例如，1.2.3.0/24) 或位址範圍 (例如 1.2.3.4-5.6.7.8) 。 與每個端點相關聯的 IP 範圍對該端點而言必須是唯一的。 不同端點之間的任何 IP 範圍重迭，都會導致流量管理員拒絕設定檔。

有兩種類型的路由設定檔可支援子網覆寫：

* **地理** -如果流量管理員找到 DNS 查詢之 IP 位址的子網覆寫，它就會將查詢路由傳送至端點的任何健康情況。
* **效能** -如果流量管理員找到 DNS 查詢之 IP 位址的子網覆寫，它只會將流量路由至端點（如果其狀況良好）。  如果子網覆寫端點的狀況不良，流量管理員會切換回效能路由啟發學習法。

## <a name="create-a-traffic-manager-subnet-override"></a>建立流量管理員子網覆寫

若要建立流量管理員子網覆寫，您可以使用 Azure CLI 將覆寫的子網新增至流量管理員端點。

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- 本文需要 2.0.28 版或更新版本的 Azure CLI。 如果您是使用 Azure Cloud Shell，就已安裝最新版本。

## <a name="update-the-traffic-manager-endpoint-with-subnet-override"></a>使用子網覆寫來更新流量管理員端點。
使用 Azure CLI 以 [az network 流量管理員端點更新](/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update)來更新您的端點。

```azurecli-interactive
### Add a range of IPs ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 1.2.3.4-5.6.7.8 \
    --type AzureEndpoints

### Add a subnet ###
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --subnets 9.10.11.0:24 \
    --type AzureEndpoints
```

您可以使用 **--remove** 選項執行 [az network 流量管理員端點更新](/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-update)來移除 IP 位址範圍。

```azurecli-interactive
az network traffic-manager endpoint update \
    --name MyEndpoint \
    --profile-name MyTmProfile \
    --resource-group MyResourceGroup \
    --remove subnets \
    --type AzureEndpoints
```

## <a name="next-steps"></a>後續步驟

深入了解「流量管理員」的 [流量路由方法](traffic-manager-routing-methods.md)。

深入瞭解 [子網流量路由方法](./traffic-manager-routing-methods.md#subnet-traffic-routing-method)