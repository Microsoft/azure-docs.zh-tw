---
title: 了解 Azure IoT 中樞配額和節流 | Microsoft Docs
description: 開發人員指南 - 說明適用於 IoT 中樞的配額和預期的節流行為。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: dobett
ms.openlocfilehash: 11cec9621ad72cfeaee45e4cd466430e64b9b836
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42141879"
---
# <a name="reference---iot-hub-quotas-and-throttling"></a>參考 - IoT 中樞配額和節流

## <a name="quotas-and-throttling"></a>配額和節流
每個 Azure 訂用帳戶最多可以有 50 個 IoT 中樞，以及最多 1 個可用中樞。

佈建每個 IoT 中樞時，都會為其佈建特定層級中的特定單位數。 層級和單位數會決定您每天可傳送的訊息配額上限。 對於免費層中樞，用來計算每日配額的訊息大小為 0.5 KB，而對於所有其他層的訊息大小則為 4 KB。 如需詳細資訊，請參閱 [Azure IoT 中樞價格][lnk-pricing]。

層級也會決定「IoT 中樞」在所有作業上強制執行的節流限制。

## <a name="operation-throttles"></a>作業節流
作業節流是在分鐘範圍內套用的速率限制，用來防止不當使用。 「IoT 中樞」會試著儘可能避免傳回錯誤，但如果違反節流太久，就會開始傳回例外狀況。

您可以隨時增加 IoT 中樞中佈建的單位數來提高配額或節流限制。

下表顯示強制執行的節流。 個別中心的值如下。

| 節流 | 免費、B1 及 S1 | B2 和 S2 | B3 和 S3 | 
| -------- | ------- | ------- | ------- |
| 身分識別登錄作業 (建立、擷取、列出、更新、刪除) | 1.67/秒/單位 (100/分鐘/單位) | 1.67/秒/單位 (100/分鐘/單位) | 83.33/秒/單位 (5000/分鐘/單位) |
| 新的裝置連線 (這項限制適用於「新連線」的建立速率，而非連線總數) | 100/秒或 12/秒/單位較高者 <br/> 例如，兩個 S1 單位是 2\*12 = 24 個新連線/秒，但是您在所有單位上至少擁有 100 個新連線/秒。 如果有九個 S1 單位，您的全部單位就會擁有 108 個新連線/秒 (9\*12)。 | 120 個新連線/秒/單位 | 6000 個新連線/秒/單位 |
| 裝置到雲端傳送 | 100/秒或 12/秒/單位較高者 <br/> 例如，兩個 S1 單位是 2\*12 = 24/秒，但是您在所有單位上至少擁有 100/秒。 如果有九個 S1 單位，您的全部單位就會擁有 108/秒 (9\*12)。 | 120/秒/單位 | 6000/秒/單位 |
| 雲端到裝置的傳送<sup>1</sup> | 1.67/秒/單位 (100/分鐘/單位) | 1.67/秒/單位 (100/分鐘/單位) | 83.33/秒/單位 (5000/分鐘/單位) |
| 雲端到裝置的接收<sup>1</sup> <br/> (僅限裝置使用 HTTPS 時)| 16.67/秒/單位 (1000/分鐘/單位) | 16.67/秒/單位 (1000/分鐘/單位) | 833.33/秒/單位 (50000/分鐘/單位) |
| 檔案上傳 | 1.67 檔案上傳通知/秒/單位 (100/分鐘/單位) | 1.67 檔案上傳通知/秒/單位 (100/分鐘/單位) | 83.33 檔案上傳通知/秒/單位 (5000/分鐘/單位) |
| 直接方法<sup>1</sup> | 160 KB/秒/單位<sup>2</sup> | 480 KB/秒/單位<sup>2</sup> | 24 MB/秒/單位<sup>2</sup> | 
| 對應項 (裝置與模組) 讀取<sup>1</sup> | 10/秒 | 10/秒或 1/秒/單位較高者 | 50/秒/單位 |
| 對應項更新 (裝置與模組)<sup>1</sup> | 10/秒 | 10/秒或 1/秒/單位較高者 | 50/秒/單位 |
| 作業操作<sup>1</sup> <br/> (建立、更新、列出、刪除) | 1.67/秒/單位 (100/分鐘/單位) | 1.67/秒/單位 (100/分鐘/單位) | 83.33/秒/單位 (5000/分鐘/單位) |
| 作業裝置操作<sup>1</sup> <br/> (更新對應項，叫用直接方法) | 10/秒 | 10/秒或 1/秒/單位較高者 | 50/秒/單位 |
| 設定與 Edge 部署<sup>1</sup> <br/> (建立、更新、列出、刪除) | 0.33/秒/單位 (20/分鐘/單位) | 0.33/秒/單位 (20/分鐘/單位) | 0.33/秒/單位 (20/分鐘/單位) |


<sup>1</sup>「IoT 中樞」的基本層中不提供此功能。 如需詳細資訊，請參閱[如何選擇適合的 IoT 中樞](iot-hub-scaling.md)。 <br/><sup>2</sup>節流計量大小為 8 KB。

「裝置連線」節流會控制新裝置可與 IoT 中樞建立連線的速率。 「裝置連線」節流不會控制同時連線裝置的數目上限。 節流受制於為 IoT 中樞佈建的單位數。

例如，若您購買單一 S1 單位，則得到每秒 100 個連線的節流。 因此，要連線到 100,000 個裝置，至少需要 1000 秒 (約 16 分鐘)。 不過，若您已將裝置登錄在您的身分識別登錄中，則可以有任意數量的同時連線裝置。

如需有關 IoT 中樞節流行為的深入討論，請參閱 [IoT Hub throttling and you (IoT 中樞節流和您)][lnk-throttle-blog] 部落格文章。

> [!IMPORTANT]
> 身分識別登錄作業是用於裝置管理與佈建案例中的執行階段用途。 透過[匯入和匯出作業][lnk-importexport]，即可支援讀取或更新大量的裝置身分識別。
> 
> 

## <a name="other-limits"></a>其他限制

IoT 中樞會強制執行其他操作限制：

| 作業 | 限制 |
| --------- | ----- |
| 檔案上傳 URI | 10000 個 SAS URI 可以讓儲存體帳戶一次用盡。 <br/> 10 個 SAS URI/裝置可以一次用盡。 |
| 作業<sup>1</sup> | 作業歷程記錄最多保留 30 天 <br/> 並行作業數上限為 1 個 (適用於免費版和 S1)、5 個 (適用於 S2)、10 個 (適用於 S3)。 |
| 額外端點 | 付費 SKU 中樞包含 10 個額外端點。 免費 SKU 中樞包含 1 個額外端點。 |
| 訊息路由規則 | 付費 SKU 中樞包含 100 個路由規則。 免費 SKU 中樞包含 5 個路由規則。 |
| 裝置到雲端傳訊 | 訊息大小上限為 256 KB |
| 雲端到裝置的傳訊<sup>1</sup> | 訊息大小上限為 64 KB。 擱置傳遞的訊息數上限為 50。 |
| 直接方法<sup>1</sup> | 直接方法承載的大小上限為 128 KB。 |
| 組態 | 每個中樞 20 個設定。 |
| Edge 部署 | 每個中樞 20 個部署。 每個部署 20 個模組。 |
| 對應項 | 每個對應項區段 (標籤、所需屬性、報告屬性) 的大小上限為 8 KB |

<sup>1</sup>「IoT 中樞」的基本層中不提供此功能。 如需詳細資訊，請參閱[如何選擇適合的 IoT 中樞](iot-hub-scaling.md)。

> [!NOTE]
> 您目前可以連線到單一 IoT 中樞的裝置數目上限為 500,000 個。 如果您想要上調此限制，請連絡 [Microsoft 支援服務](https://azure.microsoft.com/support/options/)。

## <a name="latency"></a>Latency
IoT 中樞會努力地為所有作業提供低延遲的服務。 不過，由於網路狀況及其他無法預測的因素，其無法保證最大延遲。 設計您的解決方案時，您應該：

* 避免進行有關任何 IoT 中樞作業最大延遲的任何假設。
* 在與您的裝置最靠近的 Azure 區域中佈建 IoT 中樞。
* 考慮使用 Azure IoT Edge 在裝置或靠近裝置的閘道上執行無法容忍延遲的作業。

如先前所述，多個 IoT 中樞單位既會影響節流功能，又不會提供額外的延遲好處或保證。
如果您注意到作業的延遲時間莫名其妙增加，請連絡 [Microsoft 支援服務](https://azure.microsoft.com/support/options/)。

## <a name="next-steps"></a>後續步驟
此 IoT 中樞開發人員指南中的其他參考主題包括︰

* [IoT 中樞端點][lnk-devguide-endpoints]
* [裝置對應項、作業和訊息路由的 IoT 中樞查詢語言][lnk-devguide-query]
* [IoT 中樞的 MQTT 支援][lnk-devguide-mqtt]

[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub
[lnk-throttle-blog]: https://azure.microsoft.com/blog/iot-hub-throttling-and-you/
[lnk-importexport]: iot-hub-devguide-identity-registry.md#import-and-export-device-identities

[lnk-devguide-endpoints]: iot-hub-devguide-endpoints.md
[lnk-devguide-query]: iot-hub-devguide-query-language.md
[lnk-devguide-mqtt]: iot-hub-mqtt-support.md
