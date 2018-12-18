---
title: 來自 Verizon 的核心報告 | Microsoft Docs
description: 您可以使用下列報告檢視 CDN 的使用模式：頻寬、傳輸的資料、點擊、快取狀態、快取點擊率、已傳輸的 IPV4/IPV6 資料。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 5a0d9018-8bdb-48ff-84df-23648ebcf763
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: fa828bfa736d677fb4881e5cc2628c0e03eb8749
ms.sourcegitcommit: 7f1ce8be5367d492f4c8bb889ad50a99d85d9a89
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2017
ms.locfileid: "26345242"
---
# <a name="core-reports-from-verizon"></a>來自 Verizon 的核心報告

[!INCLUDE[cdn-verizon-only](../../includes/cdn-verizon-only.md)]

透過 Verizon 設定檔的管理入口網站使用 Verizon 核心報告，您可以使用下列報告檢視 CDN 的使用模式：

* 頻寬
* 傳輸的資料
* 點擊
* 快取狀態
* 快取點擊率
* 已轉送的 IPV4/IPV6 資料

## <a name="accessing-verizon-core-reports"></a>存取 Verizon 核心報告
1. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [管理]  按鈕。
   
    ![CDN 設定檔 [管理] 按鈕](./media/cdn-reports/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
2. 將滑鼠移至 [分析] 索引標籤上，然後將滑鼠移至 [核心報告] 飛出視窗上。 按一下功能表中的報告。
   
    ![CDN 管理入口網站 - 核心報告功能表](./media/cdn-reports/cdn-core-reports.png)

3. 針對每個報告，請從 [日期範圍] 清單選取日期範圍。 您可以選取預先定義的日期範圍，例如 [今天] 或 [本週]，或者可以選取 [自訂] 並按一下日曆圖示，手動輸入日期範圍。 

4. 選取日期範圍之後，按一下 [執行] 以產生報告。 

4. 如果您需要以 Excel 格式匯出資料，請按一下 [執行] 按鈕上面的 Excel 圖示。

## <a name="bandwidth"></a>頻寬
頻寬報告由圖形和資料表所組成，指出 HTTP 與 HTTPS 在特定期間的 CDN 頻寬使用量 (以 Mbp 為單位)。 您可以跨所有 POP 或針對特定 POP 檢視頻寬使用量。 此報告可讓您檢視流量尖峰和 POP 的分佈。

從 [邊緣節點] 清單中，選取 [所有邊緣節點]，從所有節點查看流量，或選擇特定的區域。

報告每五分鐘更新一次。

![頻寬報告](./media/cdn-reports/cdn-bandwidth.png)

## <a name="data-transferred"></a>傳輸的資料
此報告由圖形和資料表所組成，指出 HTTP 與 HTTPS 在特定期間的 CDN 流量使用量 (以 GB 為單位)。 您可以跨所有 POP 或針對特定 POP 檢視流量使用量。 此報告可讓您檢視整個 POP 的流量尖峰與分佈。

從 [邊緣節點] 清單中，選取 [所有邊緣節點]，從所有節點查看流量，或選擇特定的區域。

報告每五分鐘更新一次。

![傳輸的資料報告](./media/cdn-reports/cdn-data-transferred.png)

## <a name="hits-status-codes"></a>點擊 (狀態碼)
這份報告描述內容之要求狀態碼的分佈。 內容的每個要求都會產生 HTTP 狀態碼。 狀態碼描述邊緣 POP 如何處理要求。 例如，2xx 狀態碼指出已成功向用戶端提出要求，而 4xx 狀態碼指出發生錯誤。 如需 HTTP 狀態碼的詳細資訊，請參閱 [HTTP 狀態碼清單](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)。

![點擊報告](./media/cdn-reports/cdn-hits.png)

## <a name="cache-statuses"></a>快取狀態
這份報告描述快取點擊的分佈和用戶端要求的快取遺漏。 因為最快的效能是來自快取命中，您可以將快取遺漏及過期快取命中的情況降至最低，以最佳化資料傳送速度。 

若要減少快取遺漏，請設定您的原始伺服器，將下列項目的使用降至最低： 
 * `no-cache` 回應標頭
 * 查詢字串快取，除非絕對必要  
 * 無法快取的回應碼

若要減少過期的快取命中，請將資產的 `max-age` 設定為長期間，可將原始伺服器的要求數目降至最低。

![快取狀態報告](./media/cdn-reports/cdn-cache-statuses.png)

### <a name="main-cache-statuses-include"></a>主要的快取狀態包括：
* TCP_HIT：從邊緣伺服器提供。 此物件在快取中且不超過其 max-age。
* TCP_MISS：從原始伺服器提供。 此物件不在快取中，但回應會回到來源。
* TCP_EXPIRED_MISS：利用來源重新驗證之後，從原始伺服器提供。 此物件在快取中但超過其 max-age。 利用來源進行重新驗證會導致來自來源的新回應取代快取物件。
* TCP_EXPIRED _HIT：利用來源重新驗證之後從邊緣提供。 此物件在快取中但超過其 max-age。 利用原始伺服器進行重新驗證會導致未經修改的快取物件。

### <a name="full-list-of-cache-statuses"></a>快取狀態的完整清單
* TCP_HIT - 直接從 POP 提供要求給用戶端時會回報此狀態。 當資產在最接近用戶端的 POP 上快取且具有有效的存留時間 (TTL) 時，它會立即由 POP 提供。 TTL 由下列回應標頭決定：
  
  * Cache-Control: s-maxage
  * Cache-Control: max-age
  * Expires
* TCP_MISS：這個狀態指出最接近用戶端的 POP 上找不到要求資產的快取版本。 資產會從原始伺服器或原始保護盾伺服器要求。 如果原始伺服器或原始保護盾伺服器傳回資產，它會提供給用戶端並在用戶端及邊緣伺服器上快取。 否則，會傳回非 200 狀態碼 (例如，403 禁止或 404 找不到)。
* TCP_EXPIRED_HIT：當要求的目標是 TTL 已過期的資產，且該要求會直接從 POP 提供給用戶端時，就會回報此狀態。 例如，當資產的 max-age 過期時。 
  
   過期的要求通常會導致對原始伺服器提出重新驗證要求。 為了讓 TCP_EXPIRED_HIT 狀態發生，原始伺服器必須指出較新版的資產不存在。 這種情況通常會導致更新資產的快取控制項和過期標頭。
* TCP_EXPIRED_MISS：當較新版本的過期快取資產從 POP 提供給用戶端時，就會回報此狀態。 當快取資產的 TTL 過期 (例如過期的 max-age) 且原始伺服器傳回較新版本的資產時，就會發生此狀態。 新版的資產將提供給用戶端而不是快取版本。 此外，它將會在邊緣伺服器和用戶端上快取。
* CONFIG_NOCACHE：這個狀態指出邊緣 POP 上的客戶專屬組態會防止快取資產。
* NONE - 這個狀態指出快取內容的有效性檢查並未執行。
* TCP_CLIENT_REFRESH_MISS：當 HTTP 用戶端 (例如瀏覽器) 強制邊緣 POP 從原始伺服器擷取新版的過時資產時，就會傳回此狀態。 根據預設，伺服器可以防止 HTTP 用戶端強制邊緣伺服器從原始伺服器擷取新版的資產。
* TCP_PARTIAL_HIT：當位元組範圍要求導致部分快取資產的點擊時，就會傳回此狀態。 要求的位元組範圍會立即從 POP 提供給用戶端。
* UNCACHEABLE：當資產的 `Cache-Control` 和 `Expires` 標頭指出不應該在 POP 上或由 HTTP 用戶端將其快取時，就會傳回此狀態。 這些要求類型會由原始伺服器提供。

## <a name="cache-hit-ratio"></a>快取點擊率
此報告指出直接從快取處理的快取要求百分比。

此報告提供下列詳細資料：

* 要求的內容是在最接近要求者的 POP 上加以快取。
* 直接從網路邊緣處理要求。
* 要求不需要利用原始伺服器重新驗證。

報告不包含：

* 因為國家 (地區) 篩選選項而拒絕的要求。
* 資產的要求，其標頭指出他們不應該快取。 例如，`Cache-Control: private`、`Cache-Control: no-cache` 或 `Pragma: no-cache` 標頭可防止資產受到快取。
* 部分快取內容的位元組範圍要求。

公式為：(TCP_ HIT/(TCP_ HIT+TCP_MISS))*100

![快取點擊率報告](./media/cdn-reports/cdn-cache-hit-ratio.png)

## <a name="ipv4ipv6-data-transferred"></a>已轉送的 IPV4/IPV6 資料
此報告會顯示在 IPV4 與 IPV6 中流量使用量的分佈。

![已轉送的 IPV4/IPV6 資料](./media/cdn-reports/cdn-ipv4-ipv6.png)

## <a name="considerations"></a>考量
報告只會產生於過去 18 個月內。

