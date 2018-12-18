---
title: 使用 Azure 入口網站來建立具有區域公用 IP 位址前端的公用 Load Balancer Standard | Microsoft Docs
description: 了解如何使用 Azure 入口網站來建立具有區域公用 IP 位址前端的公用 Load Balancer Standard
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
ms.date: 03/26/2018
ms.author: kumud
ms.openlocfilehash: 533c48b3a004f85dfbd2970d73dcf7de21811dca
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30323611"
---
#  <a name="create-a-public-load-balancer-standard-with-zonal-public-ip-address-frontend-using-azure-portal"></a>使用 Azure 入口網站來建立具有區域公用 IP 位址前端的公用 Load Balancer Standard

本文會逐步說明如何建立具有區域前端的公用 [Load Balancer Standard](https://aka.ms/azureloadbalancerstandard)。 若要了解可用性區域如何與標準 Load Balancer 搭配運作，請參閱[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!NOTE]
> 針對精選的 Azure 資源和區域及 VM 大小系列有提供「可用性區域」支援。 如需如何開始使用，以及有哪些 Azure 資源、區域和 VM 大小系列可供用來試用可用性區域，請參閱[可用性區域概觀](https://docs.microsoft.com/azure/availability-zones/az-overview)。 如需支援，您可以透過 [StackOverflow](https://stackoverflow.com/questions/tagged/azure-availability-zones) 與我們聯繫或[開啟 Azure 支援票證](../azure-supportability/how-to-create-azure-support-request.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。  

## <a name="log-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-a-load-balancer-with-zonal-frontend-ip-address"></a>建立具有區域前端 IP 位址的負載平衡器

1. 透過瀏覽器瀏覽至 Azure 入口網站：[http://portal.azure.com](http://portal.azure.com)，並使用您的 Azure 帳戶登入。
2. 在畫面的左上方，選取 [建立資源] > [網路] > [負載平衡器]。
3. 在 [建立負載平衡器] 頁面的 [名稱] 底下，輸入 **myLoadBalancer**。
4. 在 [類型] 底下選取 [公用]。
5. 在 SKU 底下，選取 [標準]。
6. 依序按一下 [選擇公用 IP 位址]、[新建]，然後在 [建立公用 IP 位址] 頁面中的 [名稱] 底下輸入 **myPublicIPZonal**，針對 [SKU] 選取 [標準]，針對 [可用性區域] 選取 [1]。
    
>[!NOTE] 
> 在此步驟中建立的公用 IP 預設屬於 [標準] SKU。

7. 針對 [資源群組]，按一下 [新建]，然後輸入 **myResourceGroupZLB** 作為資源群組的名稱。
8. 針對 [位置]，選取 [西歐]，然後按一下 [確定]。 負載平衡器接著會開始部署，並且需要幾分鐘的時間才能順利完成部署。

    ![使用 Azure 入口網站建立區域備援 Load Balancer Standard](./media/load-balancer-get-started-internet-availability-zones-zonal-portal/load-balancer-zonal-frontend.png)


## <a name="next-steps"></a>後續步驟
- 深入了解[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。



