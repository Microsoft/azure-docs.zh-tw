---
title: Azure 流量管理員 - 常見問題集 | Microsoft Docs
description: 本文提供 Azure 流量管理員常見問題集的解答。
services: traffic-manager
documentationcenter: ''
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: 75d5ff9a-f4b9-4b05-af32-700e7bdfea5a
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2018
ms.author: kumud
ms.openlocfilehash: 6c196d16258e4bf000f998899086c7a6d0197fba
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038006"
---
# <a name="traffic-manager-frequently-asked-questions-faq"></a>流量管理員常見問題集 (FAQ)

## <a name="traffic-manager-basics"></a>流量管理員基本概念

### <a name="what-ip-address-does-traffic-manager-use"></a>「流量管理員」使用什麼 IP 位址？

如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)所述，流量管理員是在 DNS 層級運作。 它會傳送 DNS 回應，將用戶端導向適當的服務端點。 用戶端會接著直接連線到服務端點，而不會透過「流量管理員」。

因此，「流量管理員」並不提供端點或 IP 位址來供用戶端連線。 如果您的服務需要靜態 IP 位址，就必須在服務上設定，而不是在「流量管理員」中設定。

### <a name="what-types-of-traffic-can-be-routed-using-traffic-manager"></a>哪種類型的流量可以使用流量管理員路由傳送？
如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)中所述，流量管理員端點可以是 Azure 內部或外部裝載的任何網際網路對向服務。 因此，流量管理員可以將源自公用網際網路的流量路由傳送至一組也是網際網路面向的端點。 如果您的端點是在私人網路內部 (例如，內部版本的 [Azure Load Balancer](../load-balancer/load-balancer-overview.md#internalloadbalancer))，或是讓使用者從這類內部網路提出 DNS 要求，則無法將流量管理員用於這些流量。


### <a name="does-traffic-manager-support-sticky-sessions"></a>流量管理員是否支援「黏性」工作階段？

如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)所述，流量管理員是在 DNS 層級運作。 它會使用 DNS 回應將用戶端導向到適當的服務端點。 用戶端會直接連接至服務端點，而不會透過流量管理員。 因此，流量管理員看不到用戶端與伺服器之間的 HTTP 流量。

此外，流量管理員收到之 DNS 查詢的來源 IP 位址屬於遞迴 DNS 服務，而不是用戶端。 因此，流量管理員無法追蹤個別的用戶端，也就無法實作「黏性」工作階段。 不是只有流量管理員才受此限制，所有 DNS 型流量管理系統都是如此。

### <a name="why-am-i-seeing-an-http-error-when-using-traffic-manager"></a>我在使用流量管理員時為何看到 HTTP 錯誤？

如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)所述，流量管理員是在 DNS 層級運作。 它會使用 DNS 回應將用戶端導向到適當的服務端點。 用戶端會接著直接連線到服務端點，而不會透過「流量管理員」。 流量管理員看不到用戶端與伺服器之間的 HTTP 流量。 因此，您看到的任何 HTTP 錯誤必定來自您的應用程式。 為了讓用戶端連接至應用程式，所有 DNS 解析步驟都已完成。 這包括流量管理員在應用程式流量流程上的任何互動。

因此，進一步的調查應該將焦點放在應用程式上。

從用戶端的瀏覽器傳送的 HTTP 主機標頭是最常見的問題來源。 請確定應用程式已設定為接受您使用之網域名稱的正確主機標頭。 關於使用 Azure App Service 的端點，請參閱[在使用流量管理員的 Azure App Service 中設定 Web 應用程式的自訂網域名稱](../app-service/web-sites-traffic-manager-custom-domain-name.md)。

### <a name="what-is-the-performance-impact-of-using-traffic-manager"></a>使用「流量管理員」對效能有什麼影響？

如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)所述，流量管理員是在 DNS 層級運作。 由於用戶端會直接連線到服務端點，因此在連線建立之後，使用「流量管理員」時並不會造成任何效能影響。

因為流量管理員會在 DNS 層級與應用程式整合，所以它確實需要在 DNS 解析鏈結中插入額外的 DNS 查閱。 「流量管理員」對 DNS 解析時間的影響極小。 流量管理員使用全球網路上的名稱伺服器，並使用[任一傳播](https://en.wikipedia.org/wiki/Anycast)網路功能，以確保一律將 DNS 查詢路由傳送至最靠近的可用名稱伺服器。 此外，快取 DNS 回應意謂著因使用「流量管理員」而造成的額外 DNS 延遲僅適用於一小部分工作階段。

「效能」方法會將流量路由傳送至最靠近的可用端點。 最後結果就是這個方法引起的整體效能影響會降到最低。 增加的任何 DNS 延遲都被端點更短的網路延遲所抵銷。

### <a name="what-application-protocols-can-i-use-with-traffic-manager"></a>我可以搭配「流量管理員」使用哪些應用程式通訊協定？

如[流量管理員的運作方式](../traffic-manager/traffic-manager-how-it-works.md)所述，流量管理員是在 DNS 層級運作。 完成 DNS 查閱之後，用戶端就會直接連線到應用程式端點，而不會透過「流量管理員」。 因此，連線可以使用任何應用程式通訊協定。 如果您選取 TCP 作為監視通訊協定，流量管理員的端點健康情況監視可以在不使用任何應用程式通訊協定的情況下完成。 如果您選擇使用應用程式通訊協定來驗證健康情況，端點必須能夠回應 HTTP 或 HTTPS GET 要求。

### <a name="can-i-use-traffic-manager-with-a-naked-domain-name"></a>我是否可以在流量管理員中使用「裸」網域名稱？

否。 DNS 標準不允許 CNAME 與同名的其他 DNS 記錄並存。 DNS 區域的頂點 (或根) 永遠包含兩筆預先存在的 DNS 記錄：SOA 和授權 NS 記錄。 這表示在區域頂點建立 CNAME 記錄就違反 DNS 標準。

流量管理員需要 DNS CNAME 記錄來對應虛名 DNS 名稱。 例如，您將 `www.contoso.com` 對應至流量管理員設定檔 DNS 名稱 `contoso.trafficmanager.net`。 此外，流量管理員設定檔會傳回第二筆 DNS CNAME，以指出用戶端應該連接至哪一個端點。

為了解決此問題，我們建議使用 HTTP 重新導向，將流量從裸網域名稱導向不同的 URL，這樣就可以使用流量管理員。 例如，裸網域 'contoso.com' 可以將使用者重新導向 CNAME 'www.contoso.com'，後者指向流量管理員 DNS 名稱。

我們的功能待處理項目中追蹤了對「流量管理員」中裸網域的完整支援。 您可以[在我們的社群意見反應站投票](https://feedback.azure.com/forums/217313-networking/suggestions/5485350-support-apex-naked-domains-more-seamlessly)，以表達您支持這項功能要求。

### <a name="does-traffic-manager-consider-the-client-subnet-address-when-handling-dns-queries"></a>處理 DNS 查詢時，流量管理員會考量用戶端子網路位址嗎？ 
是，除了所收到 DNS 查詢的來源 IP 位址 (通常是 DNS 解析程式的 IP 位址) 之外，在執行「地理」和「效能」路由方法的查閱時，如果代表使用者發出要求的解析程式在查詢中包含了用戶端子網路位址，則流量管理員也會考量該位址。  
具體而言即 [RFC 7871 – DNS 查詢中的用戶端子網路](https://tools.ietf.org/html/rfc7871) (英文)，此子網路提供 [DNS 的延伸機制 (EDNS0)](https://tools.ietf.org/html/rfc2671) (英文)，可將用戶端子網路位址從支援它的解析程式傳遞出去。

### <a name="what-is-dns-ttl-and-how-does-it-impact-my-users"></a>什麼是 DNS TTL，以及它如何影響我的使用者？

DNS 查詢進入流量管理員時，它會在稱為存留時間 (TTL) 的回應中設定值。 此值 (單位為秒) 會向 DNS 解析程式下游表示快取此回應要多久時間。 雖然 DNS 解析程式不保證會快取此結果，但是快取它可讓它們回應對於快取的任何後續查詢，而不是移至流量管理員 DNS 伺服器。 這會影響回應，如下所示：
- 較高的 TTL 會減少進入流量管理員 DNS 伺服器的查詢數目，因為提供服務的查詢數目是可計費的使用量，所以可減少客戶的成本。
- 較高的 TTL 可能會減少執行 DNS 查閱所花費的時間。
- 較高的 TTL 也表示您的資料不會反映流量管理員透過其探查代理程式取得的最新健康情況資訊。

### <a name="how-high-or-low-can-i-set-the-ttl-for-traffic-manager-responses"></a>我可以將流量管理員回應的 TTL 設為多高或多低？

您可以在每個設定檔層級將 DNS TTL 設定為最低 0 秒、最高 2,147,483,647 秒 (符合 [RFC-1035](https://www.ietf.org/rfc/rfc1035.txt ) 的最大範圍)。 當 TTL 為 0 時，表示下游 DNS 解析程式不會快取查詢回應，且所有查詢都預期觸達流量管理員 DNS 伺服器來進行解析。

### <a name="how-can-i-understand-the-volume-of-queries-coming-to-my-profile"></a>如何了解傳給我的設定檔的查詢量？ 
流量管理員提供的其中一個計量是某設定檔回應的查詢數量。 您可以在設定檔層級的彙總取得此資訊，或者也可以進一步分割此資料，以查看所傳回特定端點的查詢量。 此外，您可以設定警示，以在查詢回應量跨越設定的條件時通知您。 如需詳細資訊，請參閱[流量管理員的計量與警示](traffic-manager-metrics-alerts.md)。

## <a name="traffic-manager-geographic-traffic-routing-method"></a>流量管理員地理流量路由方法

### <a name="what-are-some-use-cases-where-geographic-routing-is-useful"></a>地理路由派上用場的使用案例有哪些？ 
每當 Azure 客戶需要依據地理區域來區別他們的使用者位置時，就可以使用地理路由類型。 例如，使用地理流量路由方法，您可以為特定區域的使用者提供不同於其他區域的使用者體驗。 另一個例子是符合本機資料主權規定，要求特定區域的使用者只能由該區域中的端點提供服務。

### <a name="how-do-i-decide-if-i-should-use-performance-routing-method-or-geographic-routing-method"></a>如何決定應該要使用效能路由方法或使用地理路由方法？ 
這兩個常用路由方法的主要差異，在於效能路由的主要目標是將流量傳送至能提供呼叫者最低延遲的端點，而地理路由的主要目標則是對您的呼叫者強制使用地理柵欄，這樣就能特意將他們路由到特定端點。 因為地理位置相近度和低延遲有相互關聯，所以可能會發生重疊，但也並非總是如此。 可能有在其他地理位置的端點可以提供呼叫者更佳的延遲體驗，在此情況下，效能路由會將使用者傳送到該端點，但地理位置路由會一律將他們傳送到您為其地理位置所對應的端點。 為了進一步釐清，請考慮以下範例：您可以使用地理路由製造不尋常的對應，例如將來自亞洲的所有流量都傳送到在美國的端點，或將所有來自美國的流量傳送到在亞洲的端點。 在此情況下，地理路由會完全按照您的設定執行，而不會將效能最佳化列入考量。 
>[!NOTE]
>在某些情況下，您可能需要效能路由和地理路由功能兩者，對於這些情況，巢狀設定檔可能是最佳選擇。 例如，您可以設定使用地理路由的父設定檔，將所有來自北美洲的流量，傳送到所含端點在美國的巢狀設定檔，並使用效能路由將那些流量傳送到該集合中的最佳端點。 

### <a name="what-are-the-regions-that-are-supported-by-traffic-manager-for-geographic-routing"></a>流量管理員的地理路由支援哪些區域？ 
[這裡](traffic-manager-geographic-regions.md)可以找到流量管理員所使用的國家/區域階層。 雖然此頁面一有變動就會隨時更新，但您也可以透過程式設計方式，利用 [Azure 流量管理員 REST API](https://docs.microsoft.com/rest/api/trafficmanager/) 擷取相同的資訊。 

### <a name="how-does-traffic-manager-determine-where-a-user-is-querying-from"></a>流量管理員如何判斷使用者從何處執行查詢？ 
流量管理員會查看查詢的來源 IP (這很可能是代表使用者執行查詢的本機 DNS 解析程式)，並使用內部 IP 至區域的對應來判斷位置。 此對應會融合網際網路上的變動而不斷更新。 

### <a name="is-it-guaranteed-that-traffic-manager-can-correctly-determine-the-exact-geographic-location-of-the-user-in-every-case"></a>它保證流量管理員可以在每個案例中正確判斷使用者確切的地理位置嗎？
否，基於下列因素，流量管理員不能保證我們從 DNS 查詢的來源 IP 位址推斷的地理區域將會一律對應到使用者的位置： 

- 首先，如先前的常見問題集所述，我們看到的來源 IP 位址是代表使用者執行查閱的 DNS 解析程式的 IP 位址。 儘管 DNS 解析程式的地理位置是使用者地理位置的良好 Proxy，但是，根據 DNS 解析程式服務的使用量和客戶選擇使用的特定 DNS 解析程式服務而定，它也可以不一樣。 例如，位於馬來西亞的客戶可以在他們裝置的設定中指定使用 DNS 解析程式服務，此服務可能會挑選其位於新加坡的 DNS 伺服器來處理該使用者/裝置的查詢解析。 在此情況下，流量管理員只會看到解析程式對應到新加坡位置的 IP 位址。 此外，請參閱此頁面上先前有關用戶端子網路位址支援的常見問題集。

- 其次，流量管理員會使用內部對應來進行 IP 位址到地理區域的轉譯。 雖然此對應已通過驗證並持續更新以提供其精確度並負責網際網路的進化本質，但仍會產生我們的資料無法確切代表所有 IP 位址之地理位址的可能性。


###  <a name="does-an-endpoint-need-to-be-physically-located-in-the-same-region-as-the-one-it-is-configured-with-for-geographic-routing"></a>端點必須實際位於地理路由設定的相同區域嗎？ 
否，端點的位置不限制哪些區域可以對應它。 例如，美國中部 Azure 區域的端點可以接納所有來自印度的使用者。

### <a name="can-i-assign-geographic-regions-to-endpoints-in-a-profile-that-is-not-configured-to-do-geographic-routing"></a>您可以在未設定為執行地理路由的設定檔中指派地理區域給端點嗎？ 

是，如果設定檔的路由方法不是地理，您可以使用 [Azure 流量管理員 REST API](https://docs.microsoft.com/rest/api/trafficmanager/)，將地理區域指派給該設定檔中的端點。 在非地理路由類型的設定檔中，將忽略此設定。 如果您稍後將此設定檔變更為地理路由類型，流量管理員可以使用這些對應。


### <a name="why-am-i-getting-an-error-when-i-try-to-change-the-routing-method-of-an-existing-profile-to-geographic"></a>當我嘗試將現有設定檔的路由方法變更為地理時為何會發生錯誤？

設為地理路由的設定檔下的所有端點，必須至少有一個區域對應到它。 若要將現有的設定檔轉換成地理路由類型，您需要先使用 [Azure 流量管理員 REST API](https://docs.microsoft.com/rest/api/trafficmanager/)，將地理區域和其所有端點相關聯，才能將路由類型變更為地理。 如果使用入口網站，先刪除端點，將設定檔的路由方法變更為地理，然後新增端點及其地理區域對應。 


###  <a name="why-is-it-strongly-recommended-that-customers-create-nested-profiles-instead-of-endpoints-under-a-profile-with-geographic-routing-enabled"></a>為什麼強烈建議客戶建立巢狀設定檔，而不是在啟用地理路由的設定檔下新增端點？ 

在使用地理路由類型的設定檔內，一個區域只能指派給一個端點。 如果該端點不是已連結子設定檔的巢狀類型，萬一該端點變成狀況不良，流量管理員會繼續將流量傳送給它，因為選擇不傳送任何流量也沒有比較好。 即使指派的區域位於指派給狀況不良端點的區域「上層」，「流量管理員」也不會容錯移轉至另一個端點 (例如，如果端點的西班牙區域狀況不良，我們將不會容錯移轉至另一個已被指派歐洲地區的端點)。 這是為了確保流量管理員會顧及客戶在其設定檔中已設定的地理界限。 若要在端點變成狀況不良時能夠容錯移轉至另一個端點，建議將地理區域指派給內有多個端點的巢狀設定檔，而不是指派給個別的端點。 如此一來，如果巢狀子設定檔中的端點失敗，流量就可以容錯移轉至相同巢狀子設定檔內的另一個端點。

### <a name="are-there-any-restrictions-on-the-api-version-that-supports-this-routing-type"></a>支援此路由類型的 API 版本有任何限制嗎？

是，只有 2017 年 3 月 1 日的 API 版本和更新版本支援地理路由類型。 任何舊版 API 都無法用來建立地理路由類型的設定檔，或將地理區域指派給端點。 如果使用舊版 API 從 Azure 訂用帳戶擷取設定檔，將不會傳回地理路由類型的任何設定檔。 此外，使用舊版 API 時，傳回的任何設定檔如果有已指派地理區域的端點，則不會顯示其地理區域指派。

## <a name="real-user-measurements"></a>實際使用者度量

### <a name="what-are-the-benefits-of-using-real-user-measurements"></a>使用「實際使用者度量」有哪些優點？
當您使用效能路由方法時，「流量管理員」會檢查來源 IP 和「EDNS 用戶端子網路」(如果已傳入)，並根據服務所維護的網路延遲智慧功能檢查這些資訊，為您的使用者挑選最佳的 Azure 區域來進行連線。 「實際使用者度量」可藉由除了確保延遲表格將使用者網路從使用者連線位置充分延伸到 Azure 之外，也將使用者體驗加到此延遲表格中，為您的使用者加強這項功能。 這可提升使用者路由的精確度。

### <a name="can-i-use-real-user-measurements-with-non-azure-regions"></a>我是否可以將「實際使用者度量」與非 Azure 區域搭配使用？
「實際使用者度量」只會測量連線到 Azure 區域的延遲並進行回報。 如果您將效能型路由與非 Azure 區域中所裝載的端點搭配使用，您仍然可以從這項功能受益，因為您會獲得更多有關所選取要與此端點建立關聯之代表性 Azure 區域的延遲資訊。

### <a name="which-routing-method-benefits-from-real-user-measurements"></a>哪個路由方法可從「實際使用者度量」受益？
透過「實際使用者度量」取得的額外資訊僅適用於使用效能路由方法的設定檔。 請注意，透過 Azure 入口網站檢視「實際使用者度量」時，從所有設定檔都可以使用 [實際使用者度量] 連結。

### <a name="do-i-need-to-enable-real-user-measurements-each-profile-separately"></a>我是否需要為每個設定檔個別啟用「實際使用者度量」？
否，每一訂用帳戶只需啟用一次，所測量到和回報的所有延遲資訊便可供所有設定檔使用。

### <a name="how-do-i-turn-off-real-user-measurements-for-my-subscription"></a>如何關閉我訂用帳戶的「實際使用者度量」？
當您從用戶端應用程式停止收集及傳回延遲度量時，即可停止產生與「實際使用者度量」相關的費用。 例如，當度量 JavaScript 內嵌在網頁中時，您可以移除該 JavaScript，或是在轉譯頁面時關閉其引動過程，來停止使用這項功能。

您也可以透過刪除您的金鑰來關閉「實際使用者度量」。 一旦您刪除金鑰，所有傳送給「流量管理員」且具有該金鑰的度量就都會被捨棄。

### <a name="can-i-use-real-user-measurements-with-client-applications-other-than-web-pages"></a>我是否可以將「實際使用者度量」與網頁以外的用戶端應用程式搭配使用？
是，「實際使用者度量」是設計成可以內嵌透過各種不同類型的使用者用戶端收集的資料。 當支援新類型的用戶端應用程式時，將會更新這個 FAQ。

### <a name="how-many-measurements-are-made-each-time-my-real-user-measurements-enabled-web-page-is-rendered"></a>每次轉譯已啟用「實際使用者度量」功能的網頁時，會執行多少個度量？
將「實際使用者度量」與所提供的度量 JavaScript 搭配使用時，每個頁面轉譯都會進行 6 個度量。 這些度量會接著回報給「流量管理員」服務。 請注意，將會根據回報給「流量管理員」的度量數目向您收取費用。 例如，如果使用者在進行度量時但在回報度量前離開您的網頁，這些度量就不列入計費。

### <a name="is-there-a-delay-before-real-user-measurements-script-runs-in-my-webpage"></a>「實際使用者度量」在我的網頁中執行前是否會有延遲？
否，在叫用此指令碼前沒有任何以程式設計方式設定的延遲。

### <a name="can-i-use-configure-real-user-measurements-with-only-the-azure-regions-i-want-to-measure"></a>我是否可以將「實際使用者度量」只搭配我想要測量的 Azure 區域使用？
否，每次叫用「實際使用者度量」指令碼時，它都會測量服務所決定的 6 個 Azure 區域。 這個集合會在不同的引動過程之間變更，當發生大量這樣的引動過程時，度量涵蓋範圍就會跨不同的 Azure 區域。

### <a name="can-i-limit-the-number-of-measurements-made-to-a-specific-number"></a>我是否可以將度量數目限制為特定數目？
度量 JavaScript 內嵌在您網頁內，因此您可以完全控制要何時開始或停止使用它。 只要「流量管理員」服務收到索取要測量之 Azure 區域清單的要求，就會傳回一組區域。

### <a name="can-i-see-the-measurements-taken-by-my-client-application-as-part-of-real-user-measurements"></a>我是否可以查看用戶端應用程式在「實際使用者度量」過程中所進行的度量？
由於度量邏輯是從您用戶端應用程式執行的，因此您可以完全控制所進行的作業，包括查看延遲度量。 「流量管理員」不會回報在連結至您訂用帳戶之金鑰底下收到的度量彙總檢視。

### <a name="can-i-modify-the-measurement-script-provided-by-traffic-manager"></a>我是否可以修改「流量管理員」所提供的度量指令碼？
雖然您可以控制網頁上內嵌的指令碼，但強烈建議您不要對度量指令碼進行任何變更，以確保它可正確測量並回報延遲。

### <a name="will-it-be-possible-for-others-to-see-the-key-i-use-with-real-user-measurements"></a>其他人是否可能看到我搭配「實際使用者度量」使用的金鑰？
當您將度量指令碼內嵌到網頁中時，其他人可能看到該指令碼及您的「實際使用者度量」(RUM) 金鑰。 但值得注意的是，這個金鑰與您的訂用帳戶識別碼不同，是由「流量管理員」所產生且僅供此用途使用。 得知您的 RUM 金鑰並不會危害您 Azure 帳戶的安全性。

### <a name="can-others-abuse-my-rum-key"></a>其他人是否可能濫用我的 RUM 金鑰？
雖然其他人可能使用您的金鑰來傳送錯誤資訊給 Azure，但請注意，幾個錯誤的度量並不會改變路由，因為我們會將這些度量與所收到的所有其他度量一併納入考量。 如果您需要變更金鑰，您可以在舊金鑰被捨棄時重新產生金鑰。

###  <a name="do-i-need-to-put-the-measurement-javascript-in-all-my-web-pages"></a>我是否需要將度量 JavaScript 放在所有網頁中？
「實際使用者度量」會隨著度量數目增加而變得更有價值。 儘管如此，是要將它放在所有網頁中，還是放在幾個選取的網頁中，仍是取決於您。 建議您一開始將它放在瀏覽次數最多且使用者應該至少會停留 5 秒的頁面中。

### <a name="can-information-about-my-end-users-be-identified-by-traffic-manager-if-i-use-real-user-measurements"></a>如果我使用「實際使用者度量」，「流量管理員」是否可以識別出使用者的相關資訊？
使用所提供的度量 JavaScript 時，「流量管理員」將可以檢視使用者的用戶端 IP 位址，以及他們所用本機 DNS 解析程式的來源 IP 位址。 「流量管理員」會在將用戶端 IP 位址截斷成無法識別出傳送度量的特定使用者之後，才使用該 IP 位址。 

### <a name="does-the-webpage-measuring-real-user-measurements-need-to-be-using-traffic-manager-for-routing"></a>測量「實際使用者度量」的網頁是否必須使用「流量管理員」來進行路由？
否，它不需要使用「流量管理員」。 「流量管理員」的路由端與「實際使用者度量」部分是分開運作的，雖然最好是將它們都放在相同的 Web 屬性中，但不一定要這麼做。

### <a name="do-i-need-to-host-any-service-on-azure-regions-to-use-with-real-user-measurements"></a>我是否必須在 Azure 區域裝載任何服務，才能搭配使用「實際使用者度量」？
否，您不需要在 Azure 裝載任何伺服器端元件，「實際使用者度量」便能運作。 度量 JavaScript 所下載的單一像素影像，以及在不同 Azure 區域中執行它的服務，是由 Azure 所裝載並管理的。 

### <a name="will-my-azure-bandwidth-usage-increase-when-i-use-real-user-measurements"></a>使用「實際使用者度量」時，我的 Azure 頻寬使用量是否會增加？
如先前的回答中所述，「實際使用者度量」的伺服器端元件是由 Azure 所擁有和管理的。 這意謂著您的 Azure 頻寬使用量將不會因為使用「實際使用者度量」而增加。 請注意，這並不包括 Azure 收費項目以外的任何頻寬使用量。 我們藉由只下載一個單一像素影像來測量連線到 Azure 區域的延遲，將使用的頻寬降到最低。 

## <a name="traffic-view"></a>流量檢視

### <a name="what-does-traffic-view-do"></a>「流量檢視」有哪些功能？
「流量檢視」是「流量管理員」的一項功能，可協助您進一步了解使用者及他們的體驗情況。 它會使用「流量管理員」所收到的查詢和此服務所維護的網路延遲智慧表格，來提供您下列資訊：
- 使用者從哪些區域連線到您在 Azure 中的端點。
- 有多少使用者從這些區域連線。
- 使用者被路由傳送到哪些 Azure 區域。
- 使用者連線到這些 Azure 區域時的延遲體驗。

此資訊除了以未經處理資料的形式供您下載之外，也可供您透過入口網站中的地理對應重疊和表格式檢視取用。

### <a name="how-can-i-benefit-from-using-traffic-view"></a>我如何從使用「流量檢視」受益？

「流量檢視」可提供您「流量管理員」所接收流量的整體檢視。 尤其是可用來了解您的使用者群從何處連線，以及同樣重要的是他們的平均延遲體驗。 您可以接著使用此資訊來找出所需專注的領域，例如，將您的 Azure 電腦設備擺設區域擴充到可以為這些使用者提供較低延遲的區域。 另一個您可以透過「流量檢視」獲得的深入解析是查看不同區域的流量模式，這可進而協助您決定是要在這些區域增加還是減少投資。

### <a name="how-is-traffic-view-different-from-the-traffic-manager-metrics-available-through-azure-monitor"></a>「流量檢視」與透過 Azure 監視器提供的「流量管理員」計量有何不同？

「Azure 監視器」可用來了解設定檔及其端點所收到流量的彙總層級。 它也可以顯示端點的健康情況檢查結果，來讓您追蹤端點的健康狀態。 當您不僅要了解這些，還必須了解使用者連線到 Azure 的區域層級體驗時，便可以使用「流量檢視」來達到該目的。

### <a name="does-traffic-view-use-edns-client-subnet-information"></a>「流量檢視」是否使用「EDNS 用戶端子網路」資訊？

由 Azure 流量管理員提供的 DNS 查詢確實會考量 ECS 資訊，以增加路由的精確度。 但是，當建立其中顯示使用者連線來源的資料集時，「流量檢視」僅使用 DNS 解析程式的 IP 位址。

### <a name="how-many-days-of-data-does-traffic-view-use"></a>「流量檢視」會使用多少天的資料？

「流量檢視」會處理從您檢視資料的前一天算起往前推 7 天的資料來建立輸出。 這是一個移動時段，每次您瀏覽時都會使用最新的資料。

### <a name="how-does-traffic-view-handle-external-endpoints"></a>「流量檢視」如何處理外部端點？

當您在「流量管理員」設定檔中使用裝載在 Azure 區域外的外部端點時，可以選擇讓它對應到作為其延遲特性之 Proxy 的 Azure 區域 (事實上，如果您使用效能路由方法，就必須這麼做)。 如果它擁有這個 Azure 區域對應，當建立「流量檢視」輸出時，就會使用該 Azure 區域的延遲計量。 如果未指定任何 Azure 區域，這些外部端點的資料中就不會有任何延遲資訊。

### <a name="do-i-need-to-enable-traffic-view-for-each-profile-in-my-subscription"></a>我是否必須為訂用帳戶中的每個設定檔啟用「流量檢視」？

在預覽期間，「流量檢視」會在訂用帳戶層級啟用。 作為我們在公開推出之前所做的改進的一部分，您現在可以在設定檔層級啟用「流量檢視」，以便能更細微地啟用此功能。 根據預設，「流量檢視」將針對設定檔停用。

>[!NOTE]
>如果您在預覽期間於訂用帳戶層級啟用「流量檢視」，現在就必須針對該訂用帳戶下的每個設定檔，將它重新啟用。
 
### <a name="how-can-i-turn-off-traffic-view"></a>我是否可以關閉「流量檢視」？ 
您可以使用入口網站或 REST API，針對任何設定檔關閉「流量檢視」。 

### <a name="how-does-traffic-view-billing-work"></a>「流量檢視」如何計費？

「流量檢視」價格取決於用來建立輸出的資料點數目。 目前，唯一支援的資料類型是您設定檔所接收的查詢。 此外，您只需針對在已啟用「流量檢視」之期間所執行的處理支付費用。 這意謂著如果您在一個月中的某個期間啟用「流量檢視」，而在其他期間關閉此功能，則只有在啟用此功能之期間處理的資料點會列入您的帳單中。

## <a name="traffic-manager-endpoints"></a>流量管理員端點

### <a name="can-i-use-traffic-manager-with-endpoints-from-multiple-subscriptions"></a>可以將流量管理員用於來自多個訂用帳戶的端點嗎？

來自多個訂用帳戶的端點不能與 Azure Web 應用程式搭配使用。 Azure Web Apps 規定 Web Apps 使用的任何自訂網域名稱只能在單一訂用帳戶內使用。 無法使用具有相同網域名稱的多個訂用帳戶中的 Web Apps。

針對其他端點類型，則可以將「流量管理員」與來自多個訂用帳戶的端點搭配使用。 在 Resource Manager 中，來自任何訂用帳戶的端點都可以新增至流量管理員，只要設定流量管理員設定檔的人員具有端點的讀取權限即可。 使用 [Azure Resource Manager 角色型存取控制 (RBAC)](../role-based-access-control/role-assignments-portal.md)可以授與這些權限。


### <a name="can-i-use-traffic-manager-with-cloud-service-staging-slots"></a>我可以使用流量管理員來設定雲端服務「預備」位置嗎？

是。 雲端服務「預備」位置可在流量管理員中設定為外部端點。 健康情況檢查仍然以 Azure 端點費率計費。 因為外部端點類型正在使用中，所以不會自動挑選基礎服務的變更。 使用外部端點時，流量管理員無法偵測雲端服務何時停止或刪除。 因此，流量管理員會持續計費健康情況檢查，直到停用或刪除端點為止。

### <a name="does-traffic-manager-support-ipv6-endpoints"></a>流量管理員是否支援 IPv6 端點？

流量管理員目前不提供 IPv6 定址的名稱伺服器。 不過，連接到 IPv6 端點的 IPv6 用戶端仍然可以使用流量管理員。 用戶端不會直接向流量管理員提出 DNS 要求。 相反地，用戶端會使用遞迴 DNS 服務。 僅支援 IPv6 的用戶端會透過 IPv6 將要求傳送至遞迴 DNS 服務。 然後，遞迴服務應該就能夠使用 IPv4 來連絡流量管理員名稱伺服器。

流量管理員會以端點的 DNS 名稱來回應。 為了支援 IPv6 端點，必須有一筆 DNS AAAA 記錄將端點 DNS 名稱指向 IPv6 位址。 流量管理員健康情況檢查僅支援 IPv4 位址。 此服務必須在相同 DNS 名稱上公開 IPv4 端點。

### <a name="can-i-use-traffic-manager-with-more-than-one-web-app-in-the-same-region"></a>可以使用流量管理員搭配相同區域中的多個 Web 應用程式嗎？

通常，流量管理員用於將流量導向不同區域中部署的應用程式。 不過，也可用於應用程式在相同區域中有多個部署時。 流量管理員 Azure 端點不允許將相同 Azure 區域中的多個 Web 應用程式端點新增至相同的流量管理員設定檔。

### <a name="how-do-i-move-my-traffic-manager-profiles-azure-endpoints-to-a-different-resource-group"></a>如何將流量管理員設定檔的 Azure 端點移至不同的資源群組？

與流量管理員設定檔相關聯的 Azure 端點，是使用其資源識別碼來追蹤的。 當作端點使用的 Azure 資源 (例如，公用 IP、傳統雲端服務、WebApp，或以巢狀方式使用的另一個流量管理員設定檔) 移至不同的資源群組時，其資源識別碼也會隨之變更。 在此情況下，目前您必須先刪除端點，然後再將端點加回設定檔中，以更新流量管理員設定檔。 

##  <a name="traffic-manager-endpoint-monitoring"></a>流量管理員端點監視

### <a name="is-traffic-manager-resilient-to-azure-region-failures"></a>流量管理員有能力從 Azure 區域失敗中恢復嗎？

流量管理員在 Azure 中是提供高可用性應用程式的一個重要元件。
若要提供高可用性，流量管理員必須擁有極高的可用性，並且可從區域失敗中恢復。

根據設計，流量管理員元件可從任何 Azure 區域的全面失敗中恢復。 此恢復功能適用於所有流量管理員元件︰DNS 名稱伺服器、API、儲存層及端點監視服務。

萬一整個 Azure 區域停機，也能預期流量管理員會持續正常運作。 在多個 Azure 區域中部署的應用程式可依賴流量管理員，將流量導向其應用程式的可用執行個體。

### <a name="how-does-the-choice-of-resource-group-location-affect-traffic-manager"></a>資源群組位置的選擇如何影響流量管理員？

流量管理員是單一全域服務。 不是區域性。 如何選擇資源群組位置，對於部署在該資源群組中的流量管理員設定檔而言，並沒有差別。

Azure Resource Manager 需要所有資源群組指定位置，這會決定部署在該資源群組中資源的預設位置。 當您建立流量管理員設定檔時，它會建立在資源群組中。 所有流量管理員設定檔都使用**全域**當作位置，覆寫資源群組預設值。

### <a name="how-do-i-determine-the-current-health-of-each-endpoint"></a>如何判斷每個端點目前的健全狀況？

每個端點的目前監視狀態以及整體設定檔都會顯示於 Azure 入口網站中。 您也可以透過流量監視 [REST API](https://msdn.microsoft.com/library/azure/mt163667.aspx)、[PowerShell cmdlets](https://docs.microsoft.com/powershell/module/azurerm.trafficmanager) 和[跨平台 Azure CLI](../cli-install-nodejs.md) 取得此資訊。

您也可以使用 Azure 監視器來追蹤端點的健康情況，並查看它們的視覺效果表示法。 如需如何使用 Azure 監視器的相關資訊，請參閱 [Azure 監視文件](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics)。

### <a name="can-i-monitor-https-endpoints"></a>我可以監視 HTTPS 端點嗎？

是。 流量管理員支援透過 HTTPS 探查。 在監視組態中將 **HTTPS** 設定為通訊協定。

流量管理員無法提供任何憑證驗證，包括：

* 不會驗證伺服器端憑證
* 不支援 SNI 伺服器端憑證
* 不支援用戶端憑證

### <a name="i-stopped-an-azure-cloud-service--web-application-endpoint-in-my-traffic-manager-profile-but-i-am-not-receiving-any-traffic-even-after-i-restarted-it-how-can-i-fix-this"></a>我已停止「流量管理員」設定檔中的 Azure 雲端服務 / Web 應用程式端點，但即使在重新啟動它之後都未收到任何流量。 我該怎麼辦？

當 Azure 雲端服務 / Web 應用程式端點停止時，「流量管理員」會停止檢查其健康情況，並僅在它偵測到端點已重新啟動之後，才會重新啟動健康情況檢查。 若要避免這種延遲，在重新啟動該端點之後，請在「流量管理員」設定檔中停用然後重新啟用該端點。   

### <a name="can-i-use-traffic-manager-even-if-my-application-does-not-have-support-for-http-or-https"></a>即使我的應用程式不支援 HTTP 或 HTTPS 也可以使用流量管理員嗎？

是。 您可以指定 TCP 作為監視通訊協定，流量管理員可以起始 TCP 連線，並且等待端點的回應。 如果端點在逾時期間內以建立連線的回應來回覆連線要求，則該端點會標示為狀況良好。

### <a name="what-specific-responses-are-required-from-the-endpoint-when-using-tcp-monitoring"></a>使用 TCP 監視時需要端點的哪些特定回應？

使用 TCP 監視時，流量管理員會啟動三向 TCP 交握，方法是將 SYN 要求傳送至端點的指定連接埠。 接著會等待一段時間 (在逾時設定中指定) 讓端點回應。 如果端點在監視設定指定的逾時期間內以 SYN-ACK 回應 SYN 要求，則該端點視為狀況良好。 如果收到 SYN-ACK 回應，流量管理員會藉由再回應 RST 以重設連線。

### <a name="how-fast-does-traffic-manager-move-my-users-away-from-an-unhealthy-endpoint"></a>流量管理員將我的使用者從狀況不良的端點移開時間有多快？

流量管理員提供多項設定，可協助您控制流量管理員設定檔的容錯移轉行為，如下所示：
- 您可以將探查間隔設定為 10 秒，指定流量管理員更頻繁地探查端點。 這可確保能夠立即偵測到狀況不良的任何端點。 
- 您可以指定在健康情況檢查要求逾時之前要等候多久時間 (最小逾時值是 5 秒)。
- 您可以指定在端點標示為狀況不良之前，可以發生多少次失敗。 這個值最低可以為 0，只要端點的第一次健康情況檢查失敗，就會標示為狀況不良。 不過，對於容許失敗次數使用最小值 0，可能會導致因為在探查時可能發生的任何暫時性問題，讓端點離開輪替。
- 您可以將 DNS 回應的存留時間 (TTL) 指定為最低的 0。 這麼做表示 DNS 解析程式無法快取回應，每個新的查詢會取得回應，其中包含流量管理員具有的最新健康情況資訊。

藉由使用這些設定，流量管理員可以提供在端點變成狀況不良之後 10 秒內的容錯移轉，以及針對對應設定檔進行 DNS 查詢。

### <a name="how-can-i-specify-different-monitoring-settings-for-different-endpoints-in-a-profile"></a>如何為設定檔中不同的端點指定不同監視設定？

流量管理員監視設定是在各個設定檔層級。 如果您只需要為一個端點使用不同的監視設定，可以藉由讓該端點成為[巢狀設定檔](traffic-manager-nested-profiles.md)來完成，其監視設定與父代設定檔不同。

### <a name="what-host-header-do-endpoint-health-checks-use"></a>端點健全狀況檢查使用哪一個主機標頭？

流量管理員在 HTTP 和 HTTPS 的健康狀態檢查中使用主機標頭。 流量管理員所使用的主機標頭是設定檔中設定的端點目標名稱。 主機標頭中使用的值不能與目標屬性分開指定。

### <a name="what-are-the-ip-addresses-from-which-the-health-checks-originate"></a>健康情況檢查是從哪些 IP 位址產生？

按一下[這裡](https://azuretrafficmanagerdata.blob.core.windows.net/probes/azure/probe-ip-ranges.json)以檢視 JSON 檔案，此檔案會列出可發出流量管理員健康情況檢查的 IP 位址。 請檢閱 JSON 檔案中所列出的 IP，以確保端點會允許來自這些 IP 位址的連入連線，從而能夠檢查其健康情況狀態。

### <a name="how-many-health-checks-to-my-endpoint-can-i-expect-from-traffic-manager"></a>流量管理員會對端點進行多少健康情況檢查？

流量管理員對於您端點的健康情況檢查數目取決於下列項目：
- 您為監視間隔所設定的值 (較小間隔表示在任何指定時間期間內有更多要求進入端點)。
- 產生健康情況檢查的位置數目 (您可以預期這些檢查列在上述常見問題集的 IP 位址)。

### <a name="how-can-i-get-notified-if-one-of-my-endpoints-goes-down"></a>如果我的其中一個端點關閉，我如何取得通知？ 
流量管理員所提供的其中一個計量是設定檔中端點的健全狀態。 您可以將此視為設定檔中所有端點的彙總 (例如，您的端點中 75% 是健康的)，或是依端點層級的端點彙總。 流量管理員計量是透過 Azure 監視器公開，因此您可以使用它的[警示功能](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)，在端點的健全狀態變更時取得通知。 如需詳細資訊，請參閱[流量管理員的計量與警示](traffic-manager-metrics-alerts.md)。  

## <a name="traffic-manager-nested-profiles"></a>流量管理員巢狀設定檔

### <a name="how-do-i-configure-nested-profiles"></a>如何設定巢狀設定檔？

您可以使用 Azure Resource Manager 和傳統 Azure REST API、Azure PowerShell Cmdlet 及跨平台 Azure CLI 命令來設定巢狀流量管理員設定檔。 透過新的 Azure 入口網站也支援它們。

### <a name="how-many-layers-of-nesting-does-traffic-manger-support"></a>流量管理員支援幾層巢狀結構？

設定檔的巢狀結構深度最多可達 10 層。 不允許使用「迴圈」。

### <a name="can-i-mix-other-endpoint-types-with-nested-child-profiles-in-the-same-traffic-manager-profile"></a>在同一個「流量管理員」設定檔中，是否可以將其他端點類型與巢狀子設定檔混合使用？

是。 對於在設定檔內如何結合不同類型的端點，並沒有任何限制。

### <a name="how-does-the-billing-model-apply-for-nested-profiles"></a>巢狀設定檔如何套用計費模型？

使用巢狀設定檔並沒有計價上的負面影響。

流量管理員計費有兩個要素︰端點健康狀態檢查和數百萬個 DNS 查詢

* 端點健康情況檢查︰當子設定檔被設定為父設定檔中的端點時，並不會針對該子設定檔收費。 監視子設定檔中的端點將以一般方式計費。
* DNS 查詢：每個查詢只計算一次。 查詢父設定檔而從子設定檔傳回端點時，只計入父設定檔內。

如需完整的詳細資料，請參閱[流量管理員定價頁面](https://azure.microsoft.com/pricing/details/traffic-manager/)。

### <a name="is-there-a-performance-impact-for-nested-profiles"></a>巢狀設定檔是否會對效能造成影響？

否。 使用巢狀設定檔不會影響效能。

在處理每個 DNS 查詢時，流量管理員名稱伺服器會周遊設定檔階層內部。 父設定檔的 DNS 查詢可能會收到從子設定檔傳回端點的 DNS 回應。 不論您使用單一設定檔或巢狀設定檔，都只會使用單一 CNAME 記錄。 不需要為階層中的每個設定檔建立 CNAME 記錄。

### <a name="how-does-traffic-manager-compute-the-health-of-a-nested-endpoint-in-a-parent-profile"></a>流量管理員如何計算父設定檔中的巢狀端點健康狀態？

父設定檔不會直接對子系執行健康狀態檢查， 子設定檔端點的健康狀態會用來計算子設定檔的整體健康狀態。 這項資訊會在巢狀設定檔階層中往上傳播，以判斷巢狀端點的健康狀態。 父設定檔會使用此彙總健康狀態，以決定是否可以將流量導向子設定檔。

下表描述流量管理員檢查巢狀端點健康狀態時的行為。

| 子設定檔監視狀態 | 父端點監視狀態 | 注意 |
| --- | --- | --- |
| 已停用。 已停用子設定檔。 |已停止 |父端點狀態為「已停止」，不是「已停用」。 「已停用」狀態會保留，表示您已停用父設定檔中的端點。 |
| 已降級。 至少有一個子設定檔端點的狀態為「已降級」。 |線上︰子設定檔中「線上」端點的數目至少等於 MinChildEndpoints 的值。<BR>CheckingEndpoint︰子設定檔中「線上」加 CheckingEndpoint 端點的數目至少等於 MinChildEndpoints 的值。<BR>已降級︰其他情況。 |系統會將流量傳遞給狀態為 CheckingEndpoint 的端點。 如果 MinChildEndpoints 設定太高，則端點一律會降級。 |
| 線上。 至少有一個子設定檔端點處於「線上」狀態。 沒有端點處於「已降級」狀態。 |請參閱上方。 | |
| CheckingEndpoints。 至少有一個子設定檔端點是 'CheckingEndpoint'。 沒有端點是「線上」或「已降級」。 |同上。 | |
| 非使用中。 所有子設定檔端點不是「已停用」就是「已停止」，或者此設定檔沒有任何端點 |已停止 | |

## <a name="next-steps"></a>後續步驟：
- 深入了解「流量管理員」的 [端點監視和自動容錯移轉](../traffic-manager/traffic-manager-monitoring.md)。
- 深入了解「流量管理員」的 [流量路由方法](../traffic-manager/traffic-manager-routing-methods.md)。
