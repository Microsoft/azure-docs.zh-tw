---
title: Azure 流量管理員端點監視 | Microsoft Docs
description: 本文有助於您了解流量管理員如何使用端點監視和自動端點容錯移轉，以協助 Azure 客戶部署高可用性應用程式
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/22/2021
ms.author: duau
ms.openlocfilehash: 455eeb83ef5a9608c077de24b8d3d4722d26a822
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98742704"
---
# <a name="traffic-manager-endpoint-monitoring"></a>流量管理員端點監視

Azure 流量管理員包含內建的端點監視和自動端點容錯移轉。 此功能可協助您提供能夠從端點故障中恢復的高可用性應用程 式，包括 Azure 區域失敗。

## <a name="configure-endpoint-monitoring"></a>設定端點監視

若要設定端點監視，您必須在流量管理員設定檔中指定下列設定︰

* **通訊協定**。 選擇 HTTP、HTTPS 或 TCP 作為通訊協定，流量管理員會在探查端點以檢查其健康情況時使用。 HTTPS 監視不會驗證您的 TLS/SSL 憑證是否有效--它只會檢查憑證是否存在。
* **連接埠**。 選擇要求所使用的連接埠。
* **Path**。 組態設定僅適用於 HTTP 和 HTTPS 通訊協定，這些通訊協定需要指定路徑設定。 為 TCP 監視通訊協定提供這項設定會導致錯誤。 對於 HTTP 和 HTTPS 通訊協定，提供監視存取之網頁或檔案的相對路徑和名稱。 正斜線 (/) 是相對路徑的有效項目。 這個值表示檔案位於根目錄 (預設值)。
* **自訂標頭設定**。 此設定設定可協助您將特定的 HTTP 標頭新增至流量管理員傳送至設定檔下端點的健康情況檢查。 您可以在設定檔層級上指定自訂標頭，以套用到該設定檔中的所有端點，以及/或在端點層級上指定，讓標頭僅套用到該端點。 您可以在多租使用者環境中，使用自訂標頭來進行端點的健康情況檢查。 如此一來，就可以藉由指定主機標頭，正確地將它路由傳送到目的地。 您也可以使用此設定來新增唯一標頭，以用來識別流量管理員產生的 HTTP(S) 要求，並以不同方式處理這些要求。 您最多可以指定八個標頭：值組，並以逗號分隔。 例如，"header1.xml： value1，h 2： value2"。 
* **預期的狀態碼範圍**。 這項設定可讓您以200-299、301-301 格式指定多個成功碼範圍。 當健康狀態檢查完成時，如果已將這些狀態碼視為來自端點的回應，流量管理員會將這些端點標示為狀況良好。 您可以指定最多8個狀態碼範圍。 此設定僅適用於 HTTP 和 HTTPS 通訊協定和所有端點。 此設定位於流量管理員設定檔層級，而且根據預設，值 200 會定義為成功狀態碼。
* **探查間隔**。 這個值會指定流量管理員探查代理程式檢查端點健康情況的頻率。 您可以指定以下兩個值：30 秒 (一般探查) 和 10 秒 (快速探查)。 如果沒有提供任何值，設定檔會設定為預設值 30 秒。 請瀏覽[流量管理員定價](https://azure.microsoft.com/pricing/details/traffic-manager)分頁以深入了解快速探查定價的詳細資訊。
* **容許的失敗次數**。 這個值會指定在將該端點標示為狀況不良之前，流量管理員探查代理程式可以容許多少次失敗。 此值必須介於 0 到 9 的範圍內。 值為 0 表示單一監視失敗就會導致將該端點標示為狀況不良。 如果未指定任何值，它會使用預設值 3。
* **探查超時**。 這個屬性會指定在將健康情況探查檢查視為失敗之前，流量管理員探查代理程式應等候的時間量。 如果探查間隔設定為 30 秒，則您可以設定介於 5 到 10 秒的逾時值。 如果未指定任何值，它會使用預設值 10 秒。 如果探查間隔設定為 10 秒，則您可以設定介於 5 到 9 秒的逾時值。 如果未指定任何逾時值，它會使用預設值 9 秒。

    ![流量管理員端點監視](./media/traffic-manager-monitoring/endpoint-monitoring-settings.png)

    **圖：流量管理員端點監視**

## <a name="how-endpoint-monitoring-works"></a>端點監視的運作方式

當監視通訊協定設定為 HTTP 或 HTTPS 時，流量管理員探查代理程式會使用指定的通訊協定、埠和相對路徑，向端點提出 GET 要求。 如果探查代理程式收到 200-OK 回應，或在 **預期狀態碼 \* 範圍** 中設定的任何回應，則會將端點視為狀況良好。 如果回應是不同的值，或在超時期間內未收到回應，流量管理員探查代理程式會根據「容許的失敗次數」設定來重新嘗試。 如果此設定為0，則不會執行任何重新嘗試。 如果連續失敗次數大於「容許的失敗次數」設定，則端點會標示為狀況不良。

當監視通訊協定是 TCP 時，流量管理員探查代理程式會使用指定的埠建立 TCP 連接要求。 如果端點以回應來回應要求以建立連線，則該健康情況檢查會標示為成功。 流量管理員探查代理程式會重設 TCP 連接。 如果回應是不同的值，或在超時期間內未收到回應，流量管理員探查代理程式會根據「容許的失敗次數」設定來重新嘗試。 如果此設定為0，則不會進行任何重新嘗試。 如果連續失敗次數大於「容許的失敗次數」設定，則該端點會標示為狀況不良。

在所有情況下，流量管理員會從多個位置進行探查。 連續的失敗會決定每個區域內發生的情況。 這就是為什麼端點接收來自流量管理員的健康情況探查，其頻率高於探查間隔所用的設定。

>[!NOTE]
>對於 HTTP 或 HTTPS 監視通訊協定，端點端上常見的做法是在您的應用程式內實作自訂分頁，例如，/health.aspx。 使用此路徑進行監視，您可以執行應用程式特定的檢查，例如檢查效能計數器或驗證資料庫可用性。 根據這些自訂檢查，頁面會傳回適當的 HTTP 狀態碼。

流量管理員設定檔中的所有端點都共用監視設定。 如果您需要對不同端點使用不同的監視設定，您可以建立[巢狀流量管理員設定檔](traffic-manager-nested-profiles.md#example-5-per-endpoint-monitoring-settings)。

## <a name="endpoint-and-profile-status"></a>端點和設定檔狀態

您可以啟用和停用流量管理員設定檔和端點。 不過，端點狀態的變更也可能是因為流量管理員的自動化設定和處理常式所造成。

### <a name="endpoint-status"></a>端點狀態

您可以啟用或停用特定端點。 基礎服務可能仍然狀況良好，不受影響。 變更端點狀態可控制流量管理員設定檔中端點的可用性。 當端點狀態為停用時，流量管理員不會檢查其健全狀況，而且端點不會包含在 DNS 回應中。

### <a name="profile-status"></a>設定檔狀態

使用設定檔狀態設定，您可以啟用或停用特定的設定檔。 儘管端點狀態只會影響單一端點，設定檔狀態仍會影響整個設定檔，包括所有端點。 當您停用設定檔時，不會檢查端點的健康情況，而且 DNS 回應中不會包含任何端點。 會針對 DNS 查詢傳回 [NXDOMAIN](https://tools.ietf.org/html/rfc2308) 回應碼。

### <a name="endpoint-monitor-status"></a>端點監視狀態

端點監視狀態是流量管理員所產生的值，會顯示端點的狀態。 您無法手動變更此設定。 端點監視狀態是端點監視的結果以及設定端點狀態的組合。 下表顯示端點監視狀態的可能值：

| 設定檔狀態 | 端點狀態 | 端點監視狀態 | 備註 |
| --- | --- | --- | --- |
| 已停用 |啟用 |非使用中 |已停用設定檔。 儘管端點狀態為「已啟用」，但會優先採用設定檔狀態 (「已停用」)。 停用的設定檔中的端點不會受到監視。 會針對 DNS 查詢傳回 NXDOMAIN 回應碼。 |
| &lt;any&gt; |已停用 |已停用 |端點已停用。 停用的端點不會受到監視。 端點不會包含在 DNS 回應中，因此不會接收流量。 |
| 啟用 |啟用 |線上 |端點受到監視且狀況良好。 它包含在 DNS 回應中，而且可以接收流量。 |
| 啟用 |啟用 |已降級 |端點監視健全狀況檢查失敗。 端點不會包含在 DNS 回應中，且不會接收流量。 <br>例外狀況是如果所有端點都已降級。 在這種情況下，會將它們視為在查詢回應中傳回) 。</br>|
| 啟用 |啟用 |正在檢查端點 |端點受到監視，但尚未收到第一個探查的結果。 CheckingEndpoint 是暫時性狀態，其通常會在新增或啟用設定檔中的端點之後立即發生。 處於此狀態的端點會包含於 DNS 回應中，因此可接收流量。 |
| 啟用 |啟用 |已停止 |端點指向的 web 應用程式未執行。 檢查 web 應用程式設定。 如果端點的類型為嵌套端點，且子設定檔已停用或為非使用中，也可能會發生此狀態。 <br>處於停止狀態的端點不會受到監視。 它不會包含在 DNS 回應中，也不會接收流量。 例外狀況是如果所有端點都已降級。 在這種情況下，所有這些專案都會被視為要在查詢回應中傳回。</br>|

如需如何計算巢狀端點的端點監視狀態詳細資訊，請參閱[巢狀流量管理員設定檔](traffic-manager-nested-profiles.md)。

>[!NOTE]
> 如果準層或以上層級中的 Web 應用程式未執行，App Service 可能會發生「已停止」端點監視狀態。 如需詳細資訊，請參閱[流量管理員與 App Service 整合](../app-service/web-sites-traffic-manager.md)。

### <a name="profile-monitor-status"></a>設定檔監視狀態

設定檔監視狀態是所有端點的端點監視狀態值，以及設定之設定檔狀態所形成的組合。 下表說明可用的值：

| 設定檔狀態 (依設定) | 端點監視狀態 | 設定檔監視狀態 | 備註 |
| --- | --- | --- | --- |
| Disabled |&lt;任何&gt;或未定義端點的設定檔。 |Disabled |已停用設定檔。 |
| 已啟用 |至少有一個端點的狀態為「已降級」。 |已降級 |檢閱個別端點狀態值，以判斷哪些端點需要進一步注意。 |
| 已啟用 |至少有一個端點的狀態為「線上」。 沒有狀態為「已降級」的端點。 |線上 |服務正在接收流量。 不需要進行其他動作。 |
| 已啟用 |至少有一個端點的狀態為「正在檢查端點」。 沒有狀態為「線上」或「已降級」的端點。 |正在檢查端點 |建立或啟用設定檔時，就會發生此轉換狀態。 正在初次檢查端點健康情況。 |
| 已啟用 |設定檔中的所有端點狀態為「已停用」或「已停止」，或者設定檔並未定義任何端點。 |非使用中 |沒有作用中的端點，但設定檔仍為「已啟用」。 |

## <a name="endpoint-failover-and-recovery"></a>端點容錯移轉和復原

流量管理員會定期檢查每個端點的健康狀態，包括狀況不良的端點健康情況。 當端點變成狀況良好時，流量管理員會偵測到此狀況，並且將它恢復至輪替。

發生下列任何一種狀況時，端點為狀況不良：

- 如果監視通訊協定是 HTTP 或 HTTPS：
    - 非200回應，或未包含 **預期狀態碼範圍** 設定所指定之狀態範圍的回應。  (包括不同的2xx 程式碼或301/302 重新導向) 。
- 如果監視通訊協定是 TCP： 
    - 收到 ACK 或 SYN 通知以外的回應，以回應流量管理員所傳送的 SYN 要求，以嘗試建立連接。
- 逾時。 
- 導致無法連線到端點的任何其他連線問題。

如需疑難排解失敗檢查的詳細資訊，請參閱 [疑難排解 Azure 流量管理員上的已降級狀態](traffic-manager-troubleshooting-degraded.md)。 

下圖中的時間軸是具有下列設定的流量管理員端點監視程式的詳細描述：

* 監視通訊協定為 HTTP。
* 探查間隔為30秒。
* 容許的失敗次數為3。
* Timeout 值為10秒。
* DNS TTL 為30秒。

![流量管理員端點容錯移轉和容錯回復順序](./media/traffic-manager-monitoring/timeline.png)

**圖：流量管理員端點容錯移轉和復原順序**

1. **GET**。 針對每個端點，流量管理員監視系統會對監視設定中指定的路徑執行 GET 要求。
2. **200 確定或自訂程式碼範圍指定的流量管理員設定檔監視設定**。 監視系統預期在監視設定中指定的範圍內，會在10秒內傳回 HTTP 200 OK 或狀態碼。 收到此回應時，就能確認服務可供使用。
3. **每次檢查之間間隔 30 秒**。 每隔 30 秒重複進行端點健康狀態檢查。
4. **服務無法使用**。 服務變為無法使用。 流量管理員在下一次健康情況檢查之前將無法得知。
5. **嘗試存取監視路徑**。 監視系統會執行 GET 要求，但不會在10秒的超時時間內收到回應。 接著每隔 30 秒執行一次，再嘗試三次。 如果其中一次嘗試成功，會重設嘗試次數。
6. **將狀態設為已降級**。 在第四次連續失敗之後，監視系統會將無法使用的端點狀態標示為「已降級」。
7. **流量轉向其他端點**。 流量管理員 DNS 名稱伺服器已更新，而流量管理員在回應 DNS 查詢時不再傳回端點。 新的連接會導向至其他可用的端點。 不過，先前包含此端點的 DNS 回應仍可能由遞迴 DNS 伺服器和 DNS 用戶端所快取。 用戶端會繼續使用端點，直到 DNS 快取過期為止。 當 DNS 快取過期時，用戶端會提出新的 DNS 查詢，並導向至不同的端點。 快取持續時間是由流量管理員設定檔中的 TTL 設定所控制，例如 30 秒。
8. **健全狀況檢查繼續**。 儘管端點處於「已降級」狀態，流量管理員還是會繼續檢查它的健全狀況。 當端點恢復成狀況良好的狀態時，流量管理員會偵測到此情況。
9. **服務重新上線**。 服務變為可供使用。 端點會將其降級狀態保留在流量管理員中，直到監視系統執行下一個健康情況檢查為止。
10. **服務的流量恢復**。 流量管理員傳送 GET 要求，並收到 200 OK 狀態回應。 此服務已恢復狀況良好的狀態。 流量管理員名稱伺服器會更新，並會開始在 DNS 回應中送出服務的 DNS 名稱。 流量會傳回端點，因為傳回其他端點的快取 DNS 回應會過期，而且與其他端點的現有連接即將結束。

    > [!NOTE]
    > 由於流量管理員是在 DNS 層級運作，所以無法影響任何端點的現有連接。 在引導端點之間的流量時 (透過變更設定檔設定，或是在容錯移轉或容錯回復期間)，流量管理員會將新的連接導向至可用的端點。 不過，其他端點可透過現有的連接繼續接收流量，直到這些工作階段終止為止。 若要將現有連接的流量排出，應用程式應該限制每個端點使用的工作階段持續時間。

## <a name="traffic-routing-methods"></a>流量路由方法

當端點的狀態為「已降級」時，它就不會再傳回以回應 DNS 查詢。 而是選擇並傳回替代端點。 設定檔中設定的流量路由方法會決定如何選擇替代端點。

* **優先權**。 端點會形成優先順序清單。 永遠傳回清單上第一個可用的端點。 如果端點狀態為「已降級」，則會傳回下一個可用的端點。
* **加權**。 任何可用的端點都會根據其指派的加權和其他可用端點的加權，隨機播放。
* **效能**。 會傳回最接近使用者的端點。 如果該端點無法使用，流量管理員會將流量移至最接近的下一個 Azure 區域中的端點。 您可以使用[巢狀流量管理員設定檔](traffic-manager-nested-profiles.md#example-4-controlling-performance-traffic-routing-between-multiple-endpoints-in-the-same-region)來設定效能流量路由的替代容錯移轉計劃。
* **地理**。 會傳回根據查詢要求 IP，對地理位置提供服務的已對應端點。 如果該端點無法使用，則不會選取其他端點進行容錯移轉，因為地理位置只能對應至設定檔中的一個端點。  (詳細資料位於 [常見問題](traffic-manager-FAQs.md#traffic-manager-geographic-traffic-routing-method)) 。 使用地理路由時的最佳做法是，建議客戶使用具有多個端點作為設定檔端點的巢狀流量管理員設定檔。
* **多重值** 傳回對應到 IPv4/IPv6 位址的多個端點。 收到此設定檔的查詢時，會根據您所指定的回應值 **中最大記錄計數** 來傳回狀況良好的端點。 回應的預設數目是兩個端點。
* **子網路** 傳回對應至一組 IP 位址範圍的端點。 從該 IP 位址收到要求時，傳回的端點會是該 IP 位址對應的端點。 

如需詳細資訊，請參閱 [流量管理員流量路由方法](traffic-manager-routing-methods.md)。

> [!NOTE]
> 當所有合格的端點狀態為「已降級」時，正常的流量路由行為會發生例外狀況。 流量管理員會「竭盡所能」進行嘗試，且「彷彿所有已降級狀態的端點實際上是處於線上狀態一樣地回應」 。 此行為比在 DNS 回應中不傳回任何端點的另一種做法更好。 「已停用」或「已停止」端點不會受到監視，因此，不會將它們視為符合流量資格。
>
> 這種情況通常是因為服務組態不正確所造成的，例如：
>
> * 存取控制清單 [ACL] 封鎖流量管理員健康情況檢查。
> * 流量管理員設定檔中監視連接埠或通訊協定的設定不正確。
>
> 此行為的結果是當流量管理員健康狀態檢查未正確設定時，從流量路由傳送看起來，流量管理員似乎運作正常。 不過，在此情況下，不會發生影響整體應用程式可用性的端點容錯移轉。 請務必檢查設定檔會顯示為「線上」狀態，而不是「已降級」狀態。 「線上」狀態表示流量管理員的健康狀態檢查如預期般地運作。

如需疑難排解無法進行健康狀態檢查的詳細資訊，請參閱[疑難排解 Azure 流量管理員上的已降級狀態](traffic-manager-troubleshooting-degraded.md)。

## <a name="faqs"></a>常見問題集

* [流量管理員有能力從 Azure 區域失敗中恢復嗎？](./traffic-manager-faqs.md#is-traffic-manager-resilient-to-azure-region-failures)

* [資源群組位置的選擇如何影響流量管理員？](./traffic-manager-faqs.md#how-does-the-choice-of-resource-group-location-affect-traffic-manager)

* [如何判斷每個端點目前的健全狀況？](./traffic-manager-faqs.md#how-do-i-determine-the-current-health-of-each-endpoint)

* [我可以監視 HTTPS 端點嗎？](./traffic-manager-faqs.md#can-i-monitor-https-endpoints)

* [新增端點時，使用 IP 位址還是 DNS 名稱？](./traffic-manager-faqs.md#do-i-use-an-ip-address-or-a-dns-name-when-adding-an-endpoint)

* [新增端點時，可以使用哪些類型的 IP 位址？](./traffic-manager-faqs.md#what-types-of-ip-addresses-can-i-use-when-adding-an-endpoint)

* [我可以在單一設定檔內使用不同的端點定址類型嗎？](./traffic-manager-faqs.md#can-i-use-different-endpoint-addressing-types-within-a-single-profile)

* [傳入查詢的記錄類型和與端點定址類型建立關聯的記錄類型不同時，會發生什麼事？](./traffic-manager-faqs.md#what-happens-when-an-incoming-querys-record-type-is-different-from-the-record-type-associated-with-the-addressing-type-of-the-endpoints)

* [我可以在巢狀設定檔中使用具有 IPv4/IPv6 定址端點的設定檔嗎？](./traffic-manager-faqs.md#can-i-use-a-profile-with-ipv4--ipv6-addressed-endpoints-in-a-nested-profile)

* [我已停止流量管理員設定檔中的 web 應用程式端點，但即使在重新開機後，也不會收到任何流量。如何修正此問題？](./traffic-manager-faqs.md#i-stopped-an-web-application-endpoint-in-my-traffic-manager-profile-but-i-am-not-receiving-any-traffic-even-after-i-restarted-it-how-can-i-fix-this)

* [即使我的應用程式不支援 HTTP 或 HTTPS，也可以使用流量管理員嗎？](./traffic-manager-faqs.md#can-i-use-traffic-manager-even-if-my-application-does-not-have-support-for-http-or-https)

* [使用 TCP 監視時需要端點的哪些特定回應？](./traffic-manager-faqs.md#what-specific-responses-are-required-from-the-endpoint-when-using-tcp-monitoring)

* [流量管理員將我的使用者從狀況不良的端點移開時間有多快？](./traffic-manager-faqs.md#how-fast-does-traffic-manager-move-my-users-away-from-an-unhealthy-endpoint)

* [如何為設定檔中不同的端點指定不同監視設定？](./traffic-manager-faqs.md#how-can-i-specify-different-monitoring-settings-for-different-endpoints-in-a-profile)

* [如何將 HTTP 標頭指派給我端點的流量管理員健康狀態檢查？](./traffic-manager-faqs.md#how-can-i-assign-http-headers-to-the-traffic-manager-health-checks-to-my-endpoints)

* [端點健全狀況檢查使用哪一個主機標頭？](./traffic-manager-faqs.md#what-host-header-do-endpoint-health-checks-use)

* [健康情況檢查是從哪些 IP 位址產生？](./traffic-manager-faqs.md#what-are-the-ip-addresses-from-which-the-health-checks-originate)

* [流量管理員會對端點進行多少健康情況檢查？](./traffic-manager-faqs.md#how-many-health-checks-to-my-endpoint-can-i-expect-from-traffic-manager)

* [如果我的其中一個端點關閉，我如何取得通知？](./traffic-manager-faqs.md#how-can-i-get-notified-if-one-of-my-endpoints-goes-down)

## <a name="next-steps"></a>後續步驟

了解 [流量管理員的運作方式](traffic-manager-how-it-works.md)

深入了解流量管理員支援的 [流量路由方法](traffic-manager-routing-methods.md)

了解如何 [建立流量管理員設定檔](traffic-manager-manage-profiles.md)

[疑難排解流量管理員端點上的已降級狀態](traffic-manager-troubleshooting-degraded.md)