---
title: 了解 Azure IoT 中樞訊息路由 | Microsoft Docs
description: 開發人員指南 - 如何使用訊息路由來傳送裝置到雲端訊息。 包含關於傳送遙測和非遙測資料的資訊。
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: asrastog
ms.openlocfilehash: 7c36ab2f0d4d3e5c772f8ef62c13161a2649362f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966736"
---
# <a name="use-message-routing-to-send-device-to-cloud-messages-to-different-endpoints"></a>使用訊息路由將裝置到雲端訊息傳送到不同的端點

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

訊息路由可讓您以自動化、可調整且可靠的方式，將裝置的訊息傳送到雲端服務。 訊息路由可用於： 

* **傳送裝置遙測訊息以及事件** (也就是裝置生命週期事件和裝置對應項變更事件) 到內建端點和自訂端點。 深入了解[路由端點](##routing-endpoints)。

* 套用多種查詢**篩選資料再將資料傳送到眾多端點**。 訊息路由可讓您對訊息屬性和訊息本文，以及裝置對應項標記和裝置對應項屬性進行查詢。 進一步了解使用[訊息路由中的查詢](../iot-hub/iot-hub-devguide-routing-query-syntax.md)。

IoT 中樞需要這些服務端點的寫入權限，才能將訊息路由傳送至工作。 如果您透過 Azure 入口網站設定您的端點，則會為您新增必要的權限。 請務必設定您的服務，以支援預期的輸送量。 當您第一次設定您的 IoT 解決方案時，您可能需要監視其他端點，然後對實際負載進行必要的調整。

IoT 中樞會針對所有裝置到雲端訊息定義[常見格式](../iot-hub/iot-hub-devguide-messages-construct.md)，以在通訊協定之間提供互通性。 如果訊息符合多個指向相同端點的路由，則 IoT 中樞只會將訊息傳遞至該端點一次。 因此，您不需要在您的服務匯流排佇列或主題上設定重複資料刪除。 在分割佇列中，分割區同質性可保證訊息排序。 本教學課程可用來了解如何 [設定訊息路由] (https://docs.microsoft.com/azure/iot-hub/tutorial-routing)。

## <a name="routing-endpoints"></a>路由端點

IoT 中樞都有與事件中樞相容的預設內建端點 (**訊息/事件**)。 您可以將訂用帳戶中的其他服務連結到 IoT 中樞，來建立要路由傳送訊息的目標[自訂端點](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-endpoints#custom-endpoints)。 IoT 中樞目前支援下列服務做為自訂端點︰

### <a name="built-in-endpoint"></a>內建端點
您可以使用標準[事件中樞整合和 SDK](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin) 接收內建端點 (**訊息/事件**) 的裝置到雲端訊息。 請注意，除非建立通向該端點的路由，否則在建立路由後，資料會停止流向內建端點。

### <a name="azure-blob-storage"></a>Azure Blob 儲存體
「IoT 中樞」僅支援以 [Apache Avro](http://avro.apache.org/) 格式將資料寫入「Azure Blob 儲存體」。 「IoT 中樞」會批次處理訊息，然後在批次達到特定大小或經過一段特定時間之後，就將資料寫入至 Blob。

「IoT 中樞」預設會採用下列檔案命名慣例：
```
{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}
```

您可以使用任何檔案命名慣例，不過，必須使用所有列出的語彙基元。 如果沒有資料可供寫入，IoT 中樞會寫入空的 Blob。

### <a name="service-bus-queues-and-service-bus-topics"></a>服務匯流排佇列和服務匯流排主題
做為 IoT 中樞端點的服務匯流排佇列和主題不能啟用 [工作階段] 或 [重複偵測]。 如果已啟用其中一個選項，端點會在 Azure 入口網站中顯示為 [無法連線]。

### <a name="event-hubs"></a>事件中樞
除了內建事件中樞相容端點之外，您也可以將資料傳送至事件中樞類型的自訂端點。 

當您使用路由及自訂端點時，不符合任何規則的訊息只會傳遞至內建端點。 若要將訊息傳遞至內建端點以及自訂端點，請新增會將訊息傳送至事件端點的路由。

## <a name="reading-data-that-has-been-routed"></a>讀取已傳送的資料
您可以遵循此[教學課程](https://docs.microsoft.com/azure/iot-hub/tutorial-routing)設定路由。

下列教學課程可用來了解如何從端點讀取訊息。

* 從[內建端點](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-node)讀取
* 從 [Blob 儲存體](https://docs.microsoft.com/azure/storage/blobs/storage-blob-event-quickstart)讀取
* 從[事件中樞](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send)讀取
* 從[服務匯流排佇列](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)讀取
* 從[服務匯流排主題](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions)讀取

## <a name="fallback-route"></a>後援路由
後援路由會將與任何現有路由查詢條件不符的所有訊息，傳送至與[事件中樞](https://docs.microsoft.com/azure/event-hubs/)相容的內建事件中樞 (**訊息/事件**)。 如果訊息路由已開啟，您即可啟用後援路由功能。 請注意，除非建立通向該端點的路由，否則在建立路由後，資料會停止流向內建端點。 如果內建端點沒有路由，而且後援路由已啟用，則與路由的任何查詢條件不符的訊息會傳送至內建端點。 此外，如果刪除所有的現有路由，必須啟用後援路由接收內建端點的所有資料。 

在 Azure 入口網站 -> [訊息路由] 刀鋒視窗 中，您可以啟用/停用後援路由。 您也可以對於 [FallbackRouteProperties](https://docs.microsoft.com/rest/api/iothub/iothubresource/createorupdate#fallbackrouteproperties) 使用 Azure 資源管理員來使用後援路由的自訂端點。

## <a name="non-telemetry-events"></a>非遙測事件
除了裝置遙測之外，訊息路由也可啟用傳送裝置對應項變更事件和裝置生命週期事件。 例如，如果隨同設定為**裝置對應項變更事件**的資料來源建立路由，IoT 中樞會將訊息傳送至包含裝置對應項變更的端點。 同樣地，如果隨同設定為**裝置生命週期事件**的資料來源建立路由，IoT 中樞會傳送訊息，指出是否已刪除或建立裝置。 
[IoT 中樞也與 Azure Event Grid 整合](iot-hub-event-grid.md)發佈裝置事件，以根據這些事件支援即時整合和工作流程自動化。 請參閱[訊息路由與事件格線之間的關鍵差異](iot-hub-event-grid-routing-comparison.md)了解何者最適合您的情況。

## <a name="testing-routes"></a>測試路由
您建立新的路由或編輯現有的路由時，應該使用範例訊息測試路由查詢。 您可以測試個別的路由，也可以一次測試所有路由，而且沒有任何訊息在測試期間傳送至端點。 Azure 入口網站、Azure Resource Manager、Azure PowerShell 和 Azure CLI 均可用於測試。 結果有助於識別範例訊息是否符合查詢、訊息不符合查詢，或由於範例訊息或查詢語法不正確而無法執行測試。 若要深入了解，請參閱[測試路由](https://docs.microsoft.com/rest/api/iothub/iothubresource/testroute)和[測試所有路由](https://docs.microsoft.com/rest/api/iothub/iothubresource/testallroutes)。

## <a name="latency"></a>Latency
當您使用內建端點來路由傳送裝置到雲端的遙測訊息時，在建立第一個路由之後，端對端延遲會稍微增加。

在大部分情況下，平均增加的延遲會少於 500 毫秒。 您可以使用**路由：訊息/事件的訊息延遲**或 **d2c.endpoints.latency.builtIn.events** IoT 中樞計量來監視延遲。 建立或刪除第一個之後的任何路由並不會影響端對端延遲。

## <a name="monitoring-and-troubleshooting"></a>監視與疑難排解
IoT 中樞提供數個路由和端點相關計量，以讓您概略了解中樞的健全狀況和傳送的訊息。 您可以結合多個計量的資訊，以識別問題的根本原因。 例如，使用計量**路由：捨棄的遙測訊息**或 **d2c.telemetry.egress.dropped** 識別與任何路由的查詢不符且後援路由已停用而捨棄的訊息數。 [IoT 中樞計量](https://docs.microsoft.com/azure/iot-hub/iot-hub-metrics)列出對於 IoT 中樞預設啟用的所有計量。

使用 Azure 監視器[診斷設定](https://docs.microsoft.com/azure/iot-hub/iot-hub-monitor-resource-health)中的**路由**診斷記錄，即可追蹤評估路由查詢期間發生的錯誤，以及 IoT 中樞察覺的端點健全狀況，例如端點無作用的喬況。 這些診斷記錄可傳送到 Log Analytics、事件中樞或 Azure 儲存體來進行自訂處理。

## <a name="next-steps"></a>後續步驟
* 如需了解如何建立訊息路由，請參閱[使用路由處理 IoT 中樞的裝置到雲端訊息](../iot-hub/tutorial-routing.md)教學課程。
* [快速入門](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-node)示範如何從模擬裝置傳送裝置到雲端的訊息。
* 如需您可以用來傳送裝置對雲端訊息之 SDK 的資訊，請參閱 [Azure IoT SDK](../iot-hub/iot-hub-devguide-sdks.md)。
