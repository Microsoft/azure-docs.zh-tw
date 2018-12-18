---
title: 使用 Azure 流量管理員設定優先順序流量路由方法 | Microsoft Docs
description: 此文章說明如何在流量管理員中設定優先順序流量路由方法
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: timlt
editor: ''
ms.assetid: 6dca6de1-18f7-4962-bd98-6055771fab22
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: kumud
ms.openlocfilehash: 0db83cde6facc89b8b8aa72e6419129ec868235c
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111136"
---
# <a name="configure-priority-traffic-routing-method-in-traffic-manager"></a>在流量管理員中設定優先順序流量路由方法

不論網站模式為何，Azure 網站已為資料中心 (也稱為「區域」) 內的網站提供容錯移轉功能。 流量管理員會在不同資料中心為網站提供容錯移轉。

服務容錯移轉的一個常見模式是將流量傳送到主要服務，並提供一組完全相同的備份服務以供容錯移轉。 下列步驟說明如何使用 Azure 雲端服務和網站，設定此優先順序容錯移轉︰

## <a name="to-configure-the-priority-traffic-routing-method"></a>設定優先順序流量路由方法

1. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。 如果您沒有帳戶，您可以註冊[免費試用一個月](https://azure.microsoft.com/free/)。 
2. 在入口網站的搜尋列中，搜尋「流量管理員設定檔」，然後按一下您想要設定路由方法的設定檔名稱。
3. 在 [流量管理員設定檔] 刀鋒視窗中，確認您想要納入組態的雲端服務和網站皆存在。
4. 在 [設定] 區段中，按一下 [組態]，並在 [組態] 刀鋒視窗中，完成下列設定：
    1. 針對 [流量路由方法設定]，請確認流量路由方法為 [優先順序]。 如果不是，請按一下下拉式清單中的 [優先順序]。
    2. 將此設定檔中的所有端點都設定為如下所示的相同「端點監視設定」：
        1. 選取適當的 [通訊協定]，並指定 [連接埠] 號碼。 
        2. 針對 [路徑]，輸入正斜線 */*。 若要監視端點，您必須指定路徑和檔案名稱。 正斜線 "/" 是相對路徑的有效項目，表示檔案位於根目錄 (預設值)。
        3. 按一下頁面頂端的 [儲存]。
5. 在 [設定] 區段中，按一下 [端點]。
6. 在 [端點] 刀鋒視窗中，檢閱端點的優先順序。 當您選取 [優先順序] 流量路由方法時，所選端點的順序便很重要。 確認端點的優先順序。  主要端點會位於最上方。 請重複檢查顯示的順序。 所有的要求都將路由傳送到第一個端點，且如果流量管理員偵測到端點狀況不良，流量會自動容錯移轉到下一個端點。 
7. 若要變更端點的優先順序，請按一下端點，然後在顯示的 [端點] 刀鋒視窗中，按一下 [編輯] 並依所需變更 [優先順序] 值。 
8. 按一下 [儲存] 以儲存端點設定的變更。
9. 完成組態變更之後，請按一下頁面底部的 [ **儲存** ]。
10. 測試組態中的變更，如下所示：
    1.  在入口網站的搜尋列中，搜尋流量管理員設定檔名稱，並按一下於結果中顯示的流量管理員設定檔。
    2.  在 [流量管理員設定檔] 刀鋒視窗中，按一下 [概觀]。
    3.  [流量管理員設定檔] 刀鋒視窗會顯示新建立流量管理員設定檔的 DNS 名稱。 這可由任何用戶端使用 (例如使用網頁瀏覽器進行瀏覽)，以路由傳送到由路由類型所決定的正確端點。 在此情況下，所有的要求都將路由傳送到第一個端點，且如果流量管理員偵測到端點狀況不良，流量會自動容錯移轉到下一個端點。
11. 在流量管理員設定檔運作後，您可以編輯授權 DNS 伺服器上的 DNS 記錄，以將您的公司網域名稱指向流量管理員網域名稱。

![使用流量管理員設定優先順序流量路由方法][1]

## <a name="next-steps"></a>後續步驟


- 深入了解[加權流量路由方法](traffic-manager-configure-weighted-routing-method.md)。
- 深入了解[效能路由方法](traffic-manager-configure-performance-routing-method.md)。
- 深入了解[地理路由方法](traffic-manager-configure-geographic-routing-method.md)。
- 深入了解如何[測試流量管理員設定](traffic-manager-testing-settings.md)。

<!--Image references-->
[1]: ./media/traffic-manager-priority-routing-method/traffic-manager-priority-routing-method.png