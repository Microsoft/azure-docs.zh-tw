---
title: Azure 流量管理員的效能考量 | Microsoft Docs
description: 了解流量管理員的效能，以及如何在使用流量管理員時測試您的網站效能
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: timlt
editor: ''
ms.assetid: 3ba5dfa1-2922-43f1-9a23-d06969c4a516
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: kumud
ms.openlocfilehash: f686685138625a53971f1fc5fc754fd22c9d67b2
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110786"
---
# <a name="performance-considerations-for-traffic-manager"></a>流量管理員的效能考量

此頁面說明使用流量管理員的效能考量。 請考慮下列狀況：

您在美國東部和東亞區域有網站的執行個體。 其中一個執行個體在流量管理員探查中健全狀況檢查失敗。 應用程式流量會被導向健全狀況良好的區域。 此容錯移轉是預期中之事，但效能可能是個問題，因為延遲的流量現在要流動到遙遠的區域。

## <a name="performance-considerations-for-traffic-manager"></a>流量管理員的效能考量

流量管理員對您的網站造成的唯一效能影響就是初始 DNS 查閱。 對流量管理員設定檔名稱的 DNS 要求是由裝載 trafficmanager.net 區域的 Microsoft DNS 根伺服器處理。 流量管理員會根據流量管理員的原則和探查結果，填入並定期更新 Microsoft 的 DNS 根伺服器。 因此即使在初始 DNS 查閱期間，也不會傳送任何 DNS 查詢給流量管理員。

流量管理員元件由幾個元件組成︰DNS 名稱伺服器、API 服務、儲存層、端點監視服務。 如果流量管理員服務元件失敗，對您的流量管理員設定檔相關聯的 DNS 名稱不會有影響。 Microsoft DNS 伺服器中的記錄保持不變。 不過，端點監視和 DNS 更新不會發生。 因此，當主要網站關閉時，流量管理員無法將 DNS 更新為指向您的容錯移轉網站。

系統會快速解析 DNS 名稱並快取結果。 初始 DNS 查閱的速度取決於用戶端用於名稱解析的 DNS 伺服器。 一般而言，用戶端可以在大約 50 毫秒內完成 DNS 查閱。 系統會快取查閱的結果用於 DNS 存留時間 (TTL) 的持續時間。 流量管理員的預設 TTL 為 300 秒。

流量不會透過流量管理員流動。 一旦完成 DNS 查閱，用戶端會有您的網站的執行個體 IP 位址。 用戶端直接連線到該位址，不會通過流量管理員。 您選擇的流量管理員原則並不會影響到 DNS 效能。 不過，效能路由方法可能對應用程式體驗產生負面影響。 例如，如果您的原則將流量從北美重新導向裝載於亞洲的執行個體，這些工作階段的網路延遲可能是個效能問題。

## <a name="measuring-traffic-manager-performance"></a>測試流量管理員效能

有幾個網站可幫助您了解效能和流量管理員設定檔的行為。 這些網站其中許多都是免費，但可能有限制。 某些網站提供付費的監視與報告增強功能。

這些網站上的工具會測量世界各地用戶端位置的 DNS 延遲，並顯示解析的 IP 位址。 這些工具大多數不會快取 DNS 結果。 因此，每次執行測試後工具會顯示完整的 DNS 查閱。 當您從自己的用戶端進行測試時，在 TTL 期間只會發生一次完整的 DNS 查閱效能。

## <a name="sample-tools-to-measure-dns-performance"></a>測量 DNS 效能的範例工具

* [SolveDNS](http://www.solvedns.com/dns-comparison/)

    SolveDNS 提供許多效能工具。 其 DNS 比較工具可以顯示 DNS 名稱解析所需的時間長度，以及與其他 DNS 服務提供者之間的比較。

* [WebSitePulse](http://www.websitepulse.com/help/tools.php)

    其中一個最簡單的工具是 WebSitePulse。 輸入 URL，您就會看到 DNS 解析時間、第一個位元組、最後一個位元組，以及其他效能統計資料。 您可以從三個不同的測試位置中選擇。 在本範例中，您會看到第一次執行顯示 DNS 查閱用了 0.204 秒。

    ![pulse1](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse.png)

    因為結果已快取過了，對相同流量管理員端點進行第二次測試時，DNS 查閱用了 0.002 秒。

    ![pulse2](./media/traffic-manager-performance-considerations/traffic-manager-web-site-pulse2.png)

* [CA App Synthetic Monitor](https://asm.ca.com/en/checkit.php)

    以前稱為 Watchmouse Check Website 工具，這個網站會顯示同時從多個地理區域進行 DNS 解析的時間。 輸入 URL 可以看到 DNS 解析時間、連接時間，以及數個地理位置的速度。 使用這項測試，查看在世界各地不同位置哪個裝載的服務會傳回。

    ![pulse1](./media/traffic-manager-performance-considerations/traffic-manager-web-site-watchmouse.png)

* [Pingdom](http://tools.pingdom.com/)

    這項工具提供網頁每個項目的效能統計資料。 [頁面分析] 索引標籤顯示執行 DNS 查閱所花費的時間百分比。

* [What's My DNS?](http://www.whatsmydns.net/)

    這個網站將會從 20 個不同位置執行 DNS 查閱，並將結果顯示在地圖上。

* [Dig Web Interface](http://www.digwebinterface.com)

    這個網站會顯示更詳細的 DNS 資訊，包括 CNAME 和 A 記錄。 請務必選取選項下的 [Colorize output] 和 [Stats]，並選取 [Nameservers] 下的 [All]。

## <a name="next-steps"></a>後續步驟

[關於流量管理員流量路由方法](traffic-manager-routing-methods.md)

[測試流量管理員設定](traffic-manager-testing-settings.md)

[流量管理員的相關作業 (REST API 參考)](http://go.microsoft.com/fwlink/?LinkId=313584)

[Azure 流量管理員 Cmdlet](http://go.microsoft.com/fwlink/p/?LinkId=400769)

