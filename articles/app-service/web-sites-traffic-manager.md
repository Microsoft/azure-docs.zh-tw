---
title: 使用 Azure 流量管理員來控制 Azure App Service 流量
description: 本文提供與 Azure App Service 相關之 Azure 流量管理員的摘要資訊。
services: app-service\web
documentationcenter: ''
author: cephalin
writer: cephalin
manager: erikre
editor: mollybos
ms.assetid: dabda633-e72f-4dd4-bf1c-6e945da456fd
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/25/2016
ms.author: cephalin
ms.openlocfilehash: 92ab7bf64445ff772f33a18e7f7946a7e0be333a
ms.sourcegitcommit: 3017211a7d51efd6cd87e8210ee13d57585c7e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/06/2018
ms.locfileid: "34824035"
---
# <a name="controlling-azure-app-service-traffic-with-azure-traffic-manager"></a>使用 Azure 流量管理員來控制 Azure App Service 流量
> [!NOTE]
> 本文提供與 Azure App Service 相關之 Microsoft Azure 流量管理員的摘要資訊。 如需 Azure 流量管理員本身的詳細資訊，請造訪本文結尾的連結。
> 
> 

## <a name="introduction"></a>簡介
您可以使用 Azure 流量管理員，來控制如何將來自 Web 用戶端的要求分散至 Azure App Service 中的應用程式。 將 App Service 端點新增至 Azure 流量管理員設定檔時，Azure 流量管理員就會持續追蹤您 App Service 應用程式的狀態 (執行中、已停止或已刪除)，以判定其中哪些端點應接收流量。

## <a name="routing-methods"></a>路由方法
Azure 流量管理員使用四種不同的路由方法。 下列清單說明 Azure App Service 專屬的這些方法。

* **[優先順序](../traffic-manager/traffic-manager-routing-methods.md#priority)：** 所有流量都使用主要應用程式，並提供備份以免主要或備份應用程式無法使用。
* **[加權](../traffic-manager/traffic-manager-routing-methods.md#weighted)︰** 將流量分配給一組應用程式 (不論是平均分配還是根據您定義的權數)。
* **[效能](../traffic-manager/traffic-manager-routing-methods.md#performance)：** 當您的應用程式在不同地理位置時，使用「最靠近」的應用程式 (因為網路延遲最低)。
* **[地理](../traffic-manager/traffic-manager-routing-methods.md#geographic)：** 依據產生 Web 應用程式 DNS 查詢的地理位置，將使用者導向特定應用程式。 

如需詳細資訊，請參閱[流量管理員路由方法](../traffic-manager/traffic-manager-routing-methods.md)。

## <a name="app-service-and-traffic-manager-profiles"></a>App Service 和流量管理員設定檔
若要設定以控制 App Service 應用程式流量，可在使用上述三種負載平衡方法的其中一種之 Azure 流量管理員中建立設定檔，然後新增要控制其設定檔流量的端點 (在此案例中為 App Service)。 系統會定期與設定檔溝通您的應用程式狀態 (執行中、已停止或已刪除)，讓 Azure 流量管理員可相應地導向流量。

搭配使用 Azure 流量管理員與 Azure 時，請牢記下列重點：

* 若為相同地區內的純應用程式部署，App Service 已經提供與應用程式模式無關的容錯移轉和循環配置資源功能。
* 若是在搭配使用 App Service 與其他 Azure 雲端服務的相同地區中進行部署，您可以結合兩種端點類型來啟用混合案例。
* 在設定檔中，您只能針對每個區域指定一個 App Service 端點。 選取一個應用程式作為某一個地區的端點時，將無法為該設定檔選擇使用該地區中其餘的應用程式。
* 您在 Azure 流量管理員設定檔中指定的 App Service 端點，會出現在設定檔中該應用程式之 [設定] 頁面的 [網域名稱] 區段下方，但您無法在該處進行設定。
* 將應用程式新增至設定檔後，在該應用程式之入口網站頁面的 [儀表板] 上，[網站 URL]  會顯示應用程式的自訂網域 URL (如果已設定一個)。 否則會顯示流量管理員設定檔 URL (例如，`contoso.trafficmanager.net`)。 應用程式的直接網域名稱和流量管理員 URL 都會顯示在應用程式之 [設定] 頁面的 [網域名稱]  區段下方。
* 您的自訂網域名稱會如預期般運作，但除了將之新增至應用程式外，您還必須設定 DNS 對應以指向流量管理員 URL。 如需如何為 App Service 應用程式設定自訂網域的資訊，請參閱[將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)。
* 您只能將標準或進階模式的應用程式新增至 Azure 流量管理員設定檔。

## <a name="next-steps"></a>後續步驟
如需 Azure 流量管理員的概念和技術概觀，請參閱 [Traffic Manager 概觀](../traffic-manager/traffic-manager-overview.md)。

如需將流量管理員與 App Service 搭配使用的詳細資訊，請參閱[將 Azure 流量管理員與 Azure 網站搭配使用](http://blogs.msdn.com/b/waws/archive/2014/03/18/using-windows-azure-traffic-manager-with-waws.aspx)和 [Azure 流量管理員現在可以與 Azure 網站整合部落格文章](https://azure.microsoft.com/blog/2014/03/27/azure-traffic-manager-can-now-integrate-with-azure-web-sites/)。

