---
title: 使用 Azure 流量管理員設定地理流量路由方法 | Microsoft Docs
description: 本文說明如何使用 Azure 流量管理員設定地理流量路由方法
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: timlt
editor: ''
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/22/2017
ms.author: kumud
ms.openlocfilehash: 7b49e2a4eef5a966f1ef2aa283a3089bb5b73734
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29397798"
---
# <a name="configure-the-geographic-traffic-routing-method-using-traffic-manager"></a>使用流量管理員設定地理流量路由方法

地理流量路由方法可讓您根據要求的來源地理位置，將流量導向特定的端點位置。 本教學課程示範如何使用這個路由方法來建立流量管理員設定檔，並設定端點以接收來自特定地理位置的流量。

## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔

1. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。 如果您沒有帳戶，您可以註冊[免費試用一個月](https://azure.microsoft.com/free/)。
2. 按一下 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。
4. 在 [建立流量管理員設定檔] 中：
    1. 提供設定檔的名稱。 此名稱必須是 trafficmanager.net 區域內的唯一名稱。 若要存取流量管理員設定檔，您可以使用 DNS 名稱 <profilename>.trafficmanager.net。
    2. 選取 [地理] 路由方法。
    3. 選取您要用來建立此設定檔的訂用帳戶。
    4. 使用現有的資源群組，或建立新的資源群組來放置此設定檔。 如果您選擇建立新的資源群組，請使用 [資源群組位置] 下拉式清單來指定資源群組的位置。 這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。
    5. 按一下 [建立] 之後，就會建立流量管理員設定檔並部署到全球。

![建立流量管理員設定檔](./media/traffic-manager-geographic-routing-method/create-traffic-manager-profile.png)

## <a name="add-endpoints"></a>新增端點

1. 在入口網站的搜尋列中，搜尋您建立的流量管理員設定檔名稱，然後在結果出現時按一下。
2. 在 [流量管理員] 中，瀏覽至 [設定] -> **[端點]**。
3. 按一下 [新增] 以顯示 [新增端點]。
3. 按一下 [新增]，然後在顯示的 [新增端點] 中，如下所示操作︰
4. 根據您要新增的端點類型而定，選取 [類型]。 針對生產環境中使用的地理路由設定檔，我們強烈建議使用巢狀端點類型，而且包含的子設定檔中有多個端點。 如需詳細資訊，請參閱[地理流量路由方法的常見問題集](traffic-manager-FAQs.md)。
5. 提供您要用來識別這個端點的 [名稱]。
6. 此頁面上的某些欄位，取決於您要新增的端點類型︰
    1. 如果您要新增 Azure 端點，請根據您想要將流量導向的資源，選取 [目標資源類型] 和 [目標]
    2. 如果您要新增**外部** 端點，請提供 [完整網域名稱 (FQDN)] 端點。
    3. 如果您要新增**巢狀端點**，請選取對應至您想要使用之子設定檔的 [目標資源]，並指定 [子端點計數下限]。
7. 在 [地區對應] 區段中，使用下拉式清單新增區域，表示您想要將這些區域的流量傳送至此端點。 您必須新增至少一個區域，而且可以對應多個區域。
8. 針對您想要在此設定檔下新增的所有端點，重複此步驟

![新增流量管理員端點](./media/traffic-manager-geographic-routing-method/add-traffic-manager-endpoint.png)

## <a name="use-the-traffic-manager-profile"></a>使用流量管理員設定檔
1.  在入口網站的搜尋列中，搜尋您在上一節建立的**流量管理員設定檔**名稱，然後在顯示的結果中按一下流量管理員設定檔。
2. 按一下 [概觀] 。
3. [流量管理員設定檔] 會顯示新建立之流量管理員設定檔的 DNS 名稱。 這可由任何用戶端使用 (例如使用網頁瀏覽器進行瀏覽)，以路由傳送至由路由類型所決定的正確端點。  如果是地理路由，流量管理員會查看連入要求的來源 IP ，判斷其來源區域。 如果該區域對應至端點，流量會路由傳送至該處。 如果此區域未對應至端點，流量管理員會傳回 NODATA 查詢回應。

## <a name="next-steps"></a>後續步驟

- 深入了解[地理流量路由方法](traffic-manager-routing-methods.md#geographic)。
- 深入了解如何[測試流量管理員設定](traffic-manager-testing-settings.md)。
