---
title: 在 Azure 流量管理員中設定多值流量路由方法 | Microsoft Docs
description: 本文說明如何設定流量管理員，以路由傳送流量至 A/AAAA 端點。
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: jeconnoc
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: kumud
ms.openlocfilehash: 1fae1147c941ed28d23a478a0e809b2dffba7a35
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46131118"
---
# <a name="configure-multivalue-routing-method-in-traffic-manager"></a>在流量管理員中設定多值路由方法

本文說明如何設定多值流量路由方法。 多值流量路由方法可讓您傳回多個狀況良好的端點，並協助提升應用程式的可靠性，因為用戶端有多個重試選項，無需另外進行 DNS 查閱。 多值路由只會為設定檔啟用，該設定檔具有使用 IPv4 或 IPv6 位址指定的所有端點。 當系統收到此設定檔的查詢時，便會根據指定之可設定的最大傳回計數傳回所有狀況良好的端點。 

>[!NOTE]
> 此時，使用 IPv4 或 IPv6 位址新增端點僅支援**外部**類型的端點，因此多值路由亦僅支援此類端點。

## <a name="sign-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 登入 Azure 入口網站。
## <a name="create-a-resource-group"></a>建立資源群組
建立流量管理原設定檔的資源群組。
1. 在 Azure 入口網站的左側窗格上，選取 [資源群組]。
2. 在 [資源群組] 的頁面頂端上，選取 [新增]。
3. 在 [資源群組名稱] 中，輸入名稱 *myResourceGroupTM1*。 針對 [資源群組位置]，選取 [美國東部]，然後選取 [確定]。

## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔
建立流量管理員設定檔，其導向使用者流量的方式是將使用者流量傳送至具有最低延遲的端點。

1. 在畫面的左上方，選取 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。
2. 在 [建立流量管理員設定檔] 中，輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：
    | 設定                 | 值                                              |
    | ---                     | ---                                                |
    | 名稱                   | 此名稱在 trafficmanager.net 區域內必須是唯一的，而且會產生 DNS 名稱 trafficmanager.net，用以存取您的流量管理員設定檔。                                   |
    | 路由方法          | 選取 [多值] 路由方法。                                       |
    | 訂用帳戶            | 選取您的訂用帳戶。                          |
    | 資源群組          | 選取 [myResourceGroupTM1]。 |
    | 位置                | 這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。                              |
    |
  
    ![建立流量管理員設定檔](./media/traffic-manager-multivalue-routing-method/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>新增流量管理員端點

將兩個作為外部端點的 IP 位址新增至您在前一個步驟建立的多值流量管理員設定檔。

1. 在入口網站的搜尋列中，搜尋您在上一節建立的流量管理員設定檔名稱，然後在顯示的結果中選取設定檔。
2. 在 [流量管理員設定檔] 的 [設定] 區段中，按一下 [端點]，然後按一下 [新增]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]：

    | 設定                 | 值                                              |
    | ---                     | ---                                                |
    | 類型                    | 外部端點                                   |
    | 名稱           | myEndpoint1                                        |
    | 完網域名稱 (FQDN) 或 IP           | 輸入您希望新增至此流量管理員設定檔之端點的公用 IP 位址                         |
    |        |           |

4. 重複步驟 2 和 3 以新增名為 *myEndpoint2* 的其他端點；至於**完整網域名稱 (FQDN) 或 IP**，則輸入第二個端點的公用 IP 位址。
5.  這兩個端點新增完畢後，它們會顯示在 [流量管理員設定檔] 中，而且監視狀態是 [線上]。

    ![新增流量管理員端點](./media/traffic-manager-multivalue-routing-method/add-endpoint.png)
 
## <a name="next-steps"></a>後續步驟

- 深入了解[加權流量路由方法](traffic-manager-configure-weighted-routing-method.md)。
- 深入了解[優先順序路由方法](traffic-manager-configure-priority-routing-method.md)。
- 深入了解[效能路由方法](traffic-manager-configure-performance-routing-method.md)。
- 深入了解[地理路由方法](traffic-manager-configure-geographic-routing-method.md)。


