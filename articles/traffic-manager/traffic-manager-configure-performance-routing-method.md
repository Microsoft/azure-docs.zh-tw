---
title: 使用 Azure 流量管理員設定效能流量路由方法 | Microsoft Docs
description: 此文章說明如何設定流量管理員，以最低的延遲將流量路由傳送到端點
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
ms.openlocfilehash: 014aa646459cd64fca7c697419324caa3edaeeea
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110656"
---
# <a name="configure-the-performance-traffic-routing-method"></a>設定效能流量路由方法

效能流量路由方法可讓您將流量從用戶端的網路導向具有最低延遲的端點。 通常，具有最低延遲的資料中心即為最短地理距離的資料中心。 此流量路由方法無法考慮到網路組態或負載的即時變更。

##  <a name="to-configure-performance-routing-method"></a>設定效能路由方法

1. 從瀏覽器登入 [Azure 入口網站](http://portal.azure.com)。 如果您沒有帳戶，您可以註冊[免費試用一個月](https://azure.microsoft.com/free/)。 
2. 在入口網站的搜尋列中，搜尋「流量管理員設定檔」，然後按一下您想要設定路由方法的設定檔名稱。
3. 在 [流量管理員設定檔] 刀鋒視窗中，確認您想要納入組態的雲端服務和網站皆存在。
4. 在 [設定] 區段中，按一下 [組態]，並在 [組態] 刀鋒視窗中，完成下列設定：
    1. 在 [流量路由方法設定] 中，針對 [路由方法] 選取 [效能]。
    2. 將此設定檔中的所有端點都設定為如下所示的相同「端點監視設定」：
        1. 選取適當的 [通訊協定]，並指定 [連接埠] 號碼。 
        2. 針對 [路徑]，輸入正斜線 */*。 若要監視端點，您必須指定路徑和檔案名稱。 正斜線 "/" 是相對路徑的有效項目，表示檔案位於根目錄 (預設值)。
        3. 按一下頁面頂端的 [儲存]。
5.  測試組態中的變更，如下所示：
    1.  在入口網站的搜尋列中，搜尋流量管理員設定檔名稱，並按一下於結果中顯示的流量管理員設定檔。
    2.  在 [流量管理員設定檔] 刀鋒視窗中，按一下 [概觀]。
    3.  [流量管理員設定檔] 刀鋒視窗會顯示新建立流量管理員設定檔的 DNS 名稱。 這可由任何用戶端使用 (例如使用網頁瀏覽器進行瀏覽)，以路由傳送至由路由類型所決定的正確端點。 在此情況下，所有要求都會路由傳送到用戶端網路具有最低延遲的端點。
6. 在流量管理員設定檔運作後，您可以編輯授權 DNS 伺服器上的 DNS 記錄，以將您的公司網域名稱指向流量管理員網域名稱。

![使用流量管理員設定效能流量路由方法][1]

## <a name="next-steps"></a>後續步驟

- 深入了解[加權流量路由方法](traffic-manager-configure-weighted-routing-method.md)。
- 深入了解[優先順序路由方法](traffic-manager-configure-priority-routing-method.md)。
- 深入了解[地理路由方法](traffic-manager-configure-geographic-routing-method.md)。
- 深入了解如何[測試流量管理員設定](traffic-manager-testing-settings.md)。

<!--Image references-->
[1]: ./media/traffic-manager-performance-routing-method/traffic-manager-performance-routing-method.png