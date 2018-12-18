---
title: Azure 事件中樞常見問題集 | Microsoft Docs
description: Azure 事件中樞常見問題集 (FAQ)
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.topic: article
ms.date: 08/07/2018
ms.author: shvija
ms.openlocfilehash: 50491515c7a553107666e29dcba2b4bd0836f409
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746224"
---
# <a name="event-hubs-frequently-asked-questions"></a>事件中樞常見問題集

## <a name="general"></a>一般

### <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>事件中樞基本層和標準層之間的差異為何？

Azure 事件中樞的標準層提供比基本層更多的功能。 標準層包含下列功能︰

* 較長的事件保留期
* 其他代理連線，超過包含的數目時支付超額費用
* 超過單一[消費者群組](event-hubs-features.md#consumer-groups)
* [擷取](event-hubs-capture-overview.md)

如需有關定價層的詳細資訊，包括專用事件中樞，請參閱[事件中樞定價詳細資料](https://azure.microsoft.com/pricing/details/event-hubs/)。

### <a name="what-are-event-hubs-throughput-units"></a>事件中樞輸送量單位是什麼？

您明確選取事件中樞輸送量單位，可以者透過 Azure 入口網站或事件中樞 Resource Manager 範本。 輸送量單位會套用到事件中樞命名空間內的所有事件中樞，以及賦予命名空間下列功能的每個輸送量單位：

* 最高每秒 1MB 的輸入事件 (傳送到事件中樞的事件)，但不超過每秒 1000 個輸入事件、管理作業或控制 API 呼叫。
* 最高每秒 2MB 的輸出事件 (從事件中樞取用的事件)，但是不超過 4096 個輸出事件。
* 最多 84GB 的事件儲存體 (足以應付預設的 24 小時保留期限)。

事件中樞輸送量單位以小時計費，基準為在指定時段內選取之單位的最大數目。 您可以隨著使用量的增加，使用[自動擴充來自動增加輸送量單位數](event-hubs-auto-inflate.md)。

### <a name="how-are-event-hubs-throughput-unit-limits-enforced"></a>事件中樞的輸送量單位限制如何實施？

如果命名空間內所有事件中樞的輸入輸送量總計或輸入事件率總計超過彙總輸送量單位額度，傳送者將遭受節流處置並會收到錯誤，指出已超過輸入配額。

如果命名空間內所有事件中樞的輸出輸送量總計或輸出事件率總計超過彙總輸送量單位額度，接收者將遭受節流處置並會收到錯誤，指出已超過輸出配額。 輸入和輸出配額採單獨實施，如此一來，傳送者無法使事件取用速率變慢，接收者也不能阻止事件傳送到事件中樞。

### <a name="is-there-a-limit-on-the-number-of-throughput-units-that-can-be-selected"></a>可供選取的輸送量單位數目是否有限制？

每個命名空間的預設配額為 20 個輸送量單位。 您可以藉由提出支援票證來要求較大的輸送量單位配額。 除了 20 個輸送量單位的限制之外，還有 20 和 100 個輸送量單位的組合。 請注意，使用 20 個以上的輸送量單位會排除不需要提出支援票證即可變更輸送量單位數目的能力。

使用[自動擴充](event-hubs-auto-inflate.md)功能時，您可以隨著使用量的增加，自動增加輸送量單位數。

### <a name="can-i-use-a-single-amqp-connection-to-send-and-receive-from-multiple-event-hubs"></a>我是否可以使用單一 AMQP 連線，從多個事件中樞進行傳送及接收？

是。只要所有事件中樞都位於相同的命名空間內即可。

### <a name="what-is-the-maximum-retention-period-for-events"></a>事件的最大保留期間是多少？

事件中樞標準層目前支援的最大保留期間為 7 天。 請注意，事件中樞的立意並非作為永久的資料存放區。 大於 24 小時的保留期間乃專為方便地在同一系統上重新執行事件串流的案例而設計。例如，根據現有資料來訓練或驗證新機器學習模型。 如果您需要將訊息保留超過 7 天，請在事件中樞上啟用[事件中樞擷取](event-hubs-capture-overview.md)，這會將資料從事件中樞提取到您選擇的儲存體帳戶或 Azure Data Lake 服務帳戶。 啟用擷取將會產生費用，費用根據您購買的輸送量單位而定。

### <a name="where-is-azure-event-hubs-available"></a>哪裡可以取得 Azure 事件中樞？

在所有支援的 Azure 區域皆提供 Azure 事件中樞。 如需清單，請瀏覽 [Azure 區域](https://azure.microsoft.com/regions/)頁面。  

## <a name="best-practices"></a>最佳作法

### <a name="how-many-partitions-do-i-need"></a>我需要多少個分割區？

請注意，事件中樞上的分割區計數在設定之後便無法修改。 因此，請務必在開始之前先想清楚您需要多少個分割區。 

事件中樞的設計目的是讓每個取用者群組均可擁有單一分割區讀取器。 在大多數使用案例中，四個分割區的預設設定就已足夠。 如果您想要調整事件處理，可能就要考慮加入額外的分割區。 分割區上並沒有任何特定的輸送量限制，但是，您命名空間中的彙總輸送量會受限於輸送量單位。 當您提高命名空間中的輸送量單位數目時，可能需要額外的分割區，讓並行讀取器能夠達成它們自己的最大輸送量。

不過，如果在您的模型中應用程式對特定分割區具有同質性，則提高分割區數目可能不會為您帶來任何好處。 如需此概念的詳細資訊，請參閱[可用性和一致性](event-hubs-availability-and-consistency.md)。

## <a name="pricing"></a>價格

### <a name="where-can-i-find-more-pricing-information"></a>哪裡可以找到更多定價資訊？

如需事件中樞定價的完整資訊，請參閱[事件中樞定價詳細資料](https://azure.microsoft.com/pricing/details/event-hubs/)。

### <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>保留事件中樞事件超過 24 小時需要計費嗎？

事件中樞標準層允許 24 小時以上的訊息保留期間，最多達 7 天。 如果儲存之事件總數的大小超過選定輸送量單位數目的儲存額度 (每個輸送量單位 84 GB)，超過額度的大小將以已發佈的 Azure Blob 儲存體費率計費。 即使您已將輸送量單位的最大輸入額度用盡，每個輸送量單位的儲存額度依然涵蓋 24 小時保留期間 (預設值) 的所有儲存費用。

### <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>事件中樞儲存空間大小如何計算及收費？

儲存在所有事件中樞內之事件，包括事件標頭的所有內部負荷或磁碟儲存結構的所有內部負荷，是以全天為單位來測量。 我們會在一天結束時計算儲存空間大小峰值。 每日儲存額度的計算乃以當天選定之輸送量單位的最小數目為基準 (每個輸送量單位提供 84 GB 的額度)。 如果總大小超過計算出來的每日儲存額度，我們會採用 Azure Blob 儲存體費率來計算超出的儲存空間 (依照**本地備援儲存體**費率)。

### <a name="how-are-event-hubs-ingress-events-calculated"></a>事件中樞輸入事件的計算方式為何？

每個傳送到事件中樞的事件都算是可計費訊息。 *輸入事件*的定義為小於或等於 64 KB 的資料單位。 任何大小小於或等於 64 KB 的事件均視為一個可計費事件。 如果事件大於 64 KB，可計費事件的數目乃根據事件大小來計算 (64 KB 的倍數)。 例如，一個傳送到事件中樞的 8 KB 事件將視為一個事件來計費，不過一則傳送到事件中樞的 96 KB 訊息則視為兩個事件來計費。

自事件中樞取用的事件，以及管理作業和控制呼叫 (如檢查點)，不會計入可計費輸入事件，但會累積在輸送量單位額度內。

### <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>代理連線費用適用於事件中樞嗎？

只有在使用 AMQP 通訊協定時才需要支付連線費用。 不論傳送系統或裝置的數目多寡，使用 HTTP 來傳送事件不需要支付連線費用。 如果您打算使用 AMQP (例如，為了實現更有效率的事件串流，或針對 IoT 命令和控制案例啟用雙向通訊)，請參閱[事件中樞定價資訊](https://azure.microsoft.com/pricing/details/event-hubs/)分頁，以取得關於每個服務層中包含多少個連線的詳細資訊。

### <a name="how-is-event-hubs-capture-billed"></a>事件中樞擷取如何計費？

當命名空間中的任何事件中樞有啟用擷取選項時，即會啟用擷取。 「事件中樞擷取」會依據購買的輸送量單位每小時計費。 「事件中樞擷取」計費會隨著輸送量單位計數的增減，以一整個小時為增加量反映這些變更。 如需事件中樞擷取計費的詳細資訊，請參閱[事件中樞定價資訊](https://azure.microsoft.com/pricing/details/event-hubs/)。

### <a name="will-i-be-billed-for-the-storage-account-i-select-for-event-hubs-capture"></a>我是否需要針對我為事件中樞擷取所選取的儲存體帳戶付費？

當在事件中樞上啟用擷取時，會使用您提供的儲存體帳戶。 因為這是您的儲存體帳戶，所以針對此設定的任何變更都將由您的 Azure 訂用帳戶支付相關費用。

## <a name="quotas"></a>配額

### <a name="are-there-any-quotas-associated-with-event-hubs"></a>是否有任何與事件中樞相關聯的配額？

如需所有事件中樞配額的清單，請參閱[配額](event-hubs-quotas.md)。

## <a name="troubleshooting"></a>疑難排解

### <a name="what-are-some-of-the-exceptions-generated-by-event-hubs-and-their-suggested-actions"></a>事件中樞所產生的例外狀況有哪些，其建議的動作為何？

如需可能的事件中樞例外狀況清單，請參閱[例外狀況概觀](event-hubs-messaging-exceptions.md)。

### <a name="diagnostic-logs"></a>診斷記錄檔

事件中樞支援兩種類型的[診斷記錄](event-hubs-diagnostic-logs.md) - 擷取錯誤記錄和作業記錄 - 兩種記錄都是以 JSON 格式代表，且可以透過 Azure 入口網站開啟。

### <a name="support-and-sla"></a>支援與 SLA

事件中樞的技術支援可透過[社群論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=servbus)取得。 計費及訂用帳戶管理支援均為免費提供。

若要深入了解 SLA，請參閱[服務等級協定](https://azure.microsoft.com/support/legal/sla/)頁面。

## <a name="next-steps"></a>後續步驟

您可以造訪下列連結以深入了解事件中樞︰

* [事件中心概觀](event-hubs-what-is-event-hubs.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞自動擴大](event-hubs-auto-inflate.md)
