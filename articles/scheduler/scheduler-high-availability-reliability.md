---
title: 高可用性和可靠性 - Azure 排程器
description: 了解 Azure 排程器中的高可用性和可靠性
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam
ms.assetid: 5ec78e60-a9b9-405a-91a8-f010f3872d50
ms.topic: article
ms.date: 08/16/2016
ms.openlocfilehash: d647de379972bac317a213e2f8925c0ff8c3372c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947919"
---
# <a name="high-availability-and-reliability-for-azure-scheduler"></a>Azure 排程器的高可用性和可靠性

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 會取代 Azure 排程器，Azure 排程器之後將無法使用。 若要排定作業，請[改為試用 Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md)。 

Azure 排程器可為作業同時提供[高可用性](https://docs.microsoft.com/azure/architecture/guide/pillars#availability)和可靠性。 如需詳細資訊，請參閱[排程器 SLA](https://azure.microsoft.com/support/legal/sla/scheduler)。

## <a name="high-availability"></a>高可用性

Azure 排程器高度可用，並且會使用異地備援服務部署和地理區域作業複寫。

### <a name="geo-redundant-service-deployment"></a>地理區域備援服務部署

[Azure 目前所支援的每個地理區域](https://azure.microsoft.com/global-infrastructure/regions/#services)幾乎都可在 Azure 入口網站中提供 Azure 排程器。 因此，如果所裝載區域中的 Azure 資料中心變得無法使用，您仍可使用 Azure 排程器，因為服務的容錯移轉功能可讓您從另一個資料中心使用排程器。

### <a name="geo-regional-job-replication"></a>地理區域工作複寫

您在 Azure 排程器中擁有的作業會複寫到各個 Azure 區域。 因此，如果某個區域發生中斷，Azure 排程器就會容錯移轉，並確保從配對地理區域中的另一個資料中心執行作業。

比方說，如果您在美國中南部建立作業，Azure 排程器就會在中北部自動複寫該作業。 如果美國中南部發生失敗，Azure 排程器就會在美國中北部執行該作業。 

![地理區域工作複寫](./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image1.png)

Azure 排程器也會確保如果 Azure 發生失敗，資料會保留在相同但更廣泛的地理區域中。 因此，如果您只想要高可用性，就不需要複製作業了。 Azure 排程器可自動為作業提供高可用性。

## <a name="reliability"></a>可靠性

Azure 排程器可保證其自身的高可用性，但對於使用者建立的作業則會採用不同的方法。 例如，假設作業叫用無法使用的 HTTP 端點。 Azure 排程器仍會藉由為您提供替代的失敗處理方式，來嘗試成功地執行作業： 

* 設定重試原則。
* 設定替代端點。

<a name="retry-policies"></a>

### <a name="retry-policies"></a>重試原則

Azure 排程器可讓您設定重試原則。 如果作業失敗，則根據預設，排程器會以 30 秒的間隔重新嘗試執行四次。 您可以讓此重試原則變得更積極 (例如，在 30 秒間隔內重試 10 次) 或更鬆散 (例如，每日間隔內重試兩次)。

例如，假設您建立每週作業來呼叫 HTTP 端點。 如果在作業執行時 HTTP 端點有幾個小時變得無法使用，您可能不會想要再等待一週才讓作業重新執行，之所以會如此，是因為預設的重試原則在此案例中不會有作用。 因此舉例來說，您可以變更標準重試原則，讓重試的發生頻率變成每三小時，而不是每 30 秒。 

若要了解如何設定重試原則，請參閱 [retryPolicy](scheduler-concepts-terms.md#retrypolicy)。

### <a name="alternate-endpoints"></a>替代端點

如果 Azure 排程器作業呼叫無法連線的端點，即使在遵循重試原則後，排程器仍會回復到可處理這類錯誤的替代端點。 因此，如果您設定此端點，而排程器呼叫該端點，則在發生失敗時您自己的作業將會有高可用性。

例如，下圖顯示排程器在呼叫紐約的 Web 服務時如何遵循重試原則。 如果重試失敗，排程器會檢查是否有替代端點。 如果有替代端點，排程器就會開始將要求傳送至替代端點。 相同的重試原則同時適用於原始動作和替代動作。

![重試原則和替代端點的排程器行為](./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image2.png)

替代動作的動作類型會隨著原始動作而有所不同。 例如，雖然原始動作呼叫 HTTP 端點，但替代動作可能會使用儲存體佇列、服務匯流排佇列或服務匯流排主題動作來記錄錯誤。

若要了解如何設定替代端點，請參閱 [errorAction](scheduler-concepts-terms.md#error-action)。

## <a name="see-also"></a>另請參閱

* [什麼是 Azure 排程器？](scheduler-intro.md)
* [概念、術語及實體階層](scheduler-concepts-terms.md)
* [建置複雜的排程和進階週期](scheduler-advanced-complexity.md)
* [限制、配額、預設值及錯誤碼](scheduler-limits-defaults-errors.md)
