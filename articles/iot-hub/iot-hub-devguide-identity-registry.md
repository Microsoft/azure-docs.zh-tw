---
title: 了解 Azure IoT 中樞身分識別登錄 | Microsoft Docs
description: 開發人員指南 - 說明 IoT 中樞身分識別登錄和如何用來管理裝置。 包含大量匯入和匯出裝置識別身分的相關資訊。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: dobett
ms.openlocfilehash: 041eed3a65faeb4e6c19cd9220a9e6393e18532a
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452202"
---
# <a name="understand-the-identity-registry-in-your-iot-hub"></a>了解 IoT 中樞的身分識別登錄

每個 IoT 中樞都有身分識別登錄，可儲存允許連線至 IoT 中樞裝置和模組的相關資訊。 若要讓裝置或模組可以連線到 IoT 中樞，IoT 中樞的身分識別登錄中必須先有該裝置或模組的項目。 裝置或模組也必須根據身分識別登錄中儲存的認證，向 IoT 中樞進行驗證。

儲存在身分識別登錄的裝置或模組識別碼會區分大小寫。

總括來說，身分識別登錄是支援 REST 的裝置或模組身分識別資源集合。 當您在身分識別登錄中新增項目時，IoT 中樞會建立一組每一裝置資源，例如，包含傳遞中雲端到裝置訊息的佇列。

當您需要執行下列作業時，請使用身分識別登錄：

* 佈建裝置或模組來連線到 IoT 中樞。
* 針對中樞的裝置或面對模組的端點，控制依裝置或依模組的存取。

> [!NOTE]
> 身分識別登錄不包含任何應用程式特有中繼資料。

## <a name="identity-registry-operations"></a>身分識別登錄作業

IoT 中樞身分識別登錄會公開下列作業︰

* 建立裝置或模組身分識別
* 更新裝置或模組身分識別
* 依識別碼擷取裝置或模組身分識別
* 刪除裝置或模組身分識別
* 列出多達 1000 個識別
> 模組身分識別與模組對應項都處於公開預覽階段。 當模組身分識別正式運作時，可支援下列功能。
* 將裝置身分識別匯出至 Azure Blob 儲存體
* 從 Azure Blob 儲存體匯入裝置身分識別

上述所有作業均可使用 [RFC7232](https://tools.ietf.org/html/rfc7232) 中指定的開放式並行存取。

> [!IMPORTANT]
> 如果要擷取 IoT 中樞的身分識別登錄中的所有身分識別，唯一方法是使用[匯出](iot-hub-devguide-identity-registry.md#import-and-export-device-identities)功能。

IoT 中樞身分識別登錄：

* 不包含任何應用程式中繼資料。
* 可以將 **deviceId** 或 **moduleId** 做為索引鍵來存取，就像字典一樣。
* 不支援表達式查詢。

IoT 方案通常具有不同的方案專屬存放區，其中包含應用程式特定的中繼資料。 例如，智慧建置方案中的解決方案專用存放區會記錄部署溫度感應器的空間。

> [!IMPORTANT]
> 只能將身分識別登錄用於裝置管理和佈建作業。 執行階段的高輸送量作業不應該仰賴在身分識別登錄中執行作業。 例如，在傳送命令前先檢查裝置的連線狀態就不是支援的模式。 請務必檢查身分識別登錄的[節流速率](iot-hub-devguide-quotas-throttling.md)以及[裝置活動訊號](iot-hub-devguide-identity-registry.md#device-heartbeat)模式。

## <a name="disable-devices"></a>停用裝置

您可以在身分識別登錄中更新身分識別的 [狀態] 屬性來停用裝置。 您通常會在兩種情況下使用此屬性：

* 在佈建協調程序期間。 如需詳細資訊，請參閱[裝置佈建](iot-hub-devguide-identity-registry.md#device-provisioning)。

* 如果因為任何原因，您認為裝置遭到入侵，或變成未經授權。

模組無法使用這項功能。

## <a name="import-and-export-device-identities"></a>匯入和匯出裝置身分識別

請使用 [IoT 中樞資源提供者端點](iot-hub-devguide-endpoints.md)上的非同步作業，從 IoT 中樞的身分識別登錄大量匯出裝置身分識別。 匯出是長時間執行的作業，其使用客戶提供的 Blob 容器來儲存讀取自身分識別登錄的裝置身分識別資料。

請使用 [IoT 中樞資源提供者端點](iot-hub-devguide-endpoints.md)上的非同步作業，將裝置身分識別大量匯入至 IoT 中樞的身分識別登錄。 匯入是長時間執行的作業，其使用客戶提供的 Blob 容器中的資料，將裝置身分識別資料寫入至身分識別登錄。

如需有關匯入和匯出 API 的詳細資訊，請參閱 [IoT 中樞資源提供者 REST API](/rest/api/iothub/iothubresource)。 若要深入了解如何執行匯入和匯出作業，請參閱[大量管理 IoT 中樞的裝置身分識別](iot-hub-bulk-identity-mgmt.md)。

## <a name="device-provisioning"></a>裝置佈建

給定的 IoT 解決方案儲存的裝置資料取決於該解決方案的特定需求。 但是解決方案至少必須儲存裝置身分識別和驗證金鑰。 Azure IoT 中樞包含身分識別登錄，其可以儲存每個裝置的值，例如識別碼、驗證金鑰和狀態碼。 解決方案可以使用其他 Azure 服務 (例如表格儲存體、Blob 儲存體或 Cosmos DB) 來儲存任何其他裝置資料。

*裝置佈建* 是將初始裝置資料加入至解決方案中存放區的程序。 若要讓新裝置能夠連接到您的中樞，必須將裝置識別碼和金鑰新增至「IoT 中樞」身分識別登錄。 做為佈建程序的一部分，您可能需要初始化其他解決方案存放區中的裝置特定資料。 您也可以使用 Azure IoT 中樞裝置佈建服務，以無須人為介入的方式對一或多個 IoT 中樞進行 Just-In-Time 自動佈建。 若要深入了解，請參閱[佈建服務文件](https://azure.microsoft.com/documentation/services/iot-dps)。

## <a name="device-heartbeat"></a>裝置活動訊號

IoT 中樞身分識別登錄包含稱為 **connectionState** 的欄位。 請只在進行開發和偵錯時才使用 **connectionState**。 IoT 解決方案不應該在執行階段查詢欄位。 例如，請勿查詢 **connectionState** 欄位從而在傳送雲端到裝置訊息或 SMS 之前先確認裝置是否已連線。 我們建議您訂閱事件方格上的[**裝置中斷連線**事件](iot-hub-event-grid.md#event-types)來取得警示，並監視裝置連線狀態。 使用此[教學課程](iot-hub-how-to-order-connection-state-events.md)可了解如何將 IoT 中樞的「裝置連線」和「裝置中斷連線」事件整合至 IoT 解決方案中。

如果 IoT 解決方案需要知道裝置是否已連線，可實作「活動訊號模式」。
在活動訊號模式中，裝置每隔固定時間就會至少傳送一次裝置到雲端的訊息 (例如，每小時至少一次)。 因此，即使裝置沒有任何要傳送的資料，它仍會傳送空的裝置到雲端的訊息 (通常具有可識別它是活動訊號的屬性)。 此解決方案會在服務端保有一份對應，其中有針對每個裝置收到的最後一次活動訊號。 此解決方案若未在預期時間內收到裝置傳來的活動訊號訊息，就會假設該裝置發生問題。

更複雜的實作可以包含來自 [Azure 監視器](../azure-monitor/index.yml)和 [Azure 資源健康狀態](../service-health/resource-health-overview.md)的資訊，以便識別嘗試連線或通訊但失敗的裝置，請查看[使用診斷進行監視](iot-hub-monitor-resource-health.md)指南。 當您實作活動訊號模式時，請務必檢查 [IoT 中樞配額與節流](iot-hub-devguide-quotas-throttling.md)。

> [!NOTE]
> 如果 IoT 解決方案僅使用連線狀態來判斷是否要傳送雲端到裝置訊息，而且訊息未廣播到大量的裝置組合，請考慮使用較簡單的「短暫的到期時間」模式。 此模式所達到的效果與使用活動訊號模式來維護裝置連線狀態登錄相同，但更有效率。 如果您要求訊息收條，IoT 中樞會通知您哪些裝置能夠收到訊息，哪些裝置不能收到。

## <a name="device-and-module-lifecycle-notifications"></a>裝置與模組的生命週期通知

IoT 中樞在身分識別建立或刪除時，可透過傳送生命週期通知，以通知 IoT 解決方案。 若要這樣做，您的 IoT 解決方案必須建立路由，並將資料來源設為等於 *DeviceLifecycleEvents* 或 *ModuleLifecycleEvents*。 根據預設，不會傳送任何生命週期通知，亦即沒有預先存在的這類路由。 通知訊息包含屬性和內文。

屬性：訊息系統屬性前面會加上 `$` 符號。

裝置的通知訊息：

| 名稱 | 值 |
| --- | --- |
|$content-type | application/json |
|$iothub-enqueuedtime |  傳送通知的時間 |
|$iothub-message-source | deviceLifecycleEvents |
|$content-encoding | utf-8 |
|opType | **createDeviceIdentity** 或 **deleteDeviceIdentity** |
|hubName | IoT 中樞名稱 |
|deviceId | 裝置的識別碼 |
|operationTimestamp | 作業的 ISO8601 時間戳記 |
|iothub-message-schema | deviceLifecycleNotification |

主體：本節為 JSON 格式，它表示所建立裝置身分識別的對應項。 例如，

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```
模組的通知訊息：

| 名稱 | 值 |
| --- | --- |
$content-type | application/json |
$iothub-enqueuedtime |  傳送通知的時間 |
$iothub-message-source | moduleLifecycleEvents |
$content-encoding | utf-8 |
opType | **createModuleIdentity** 或 **deleteModuleIdentity** |
hubName | IoT 中樞名稱 |
moduleId | 模組的識別碼 |
operationTimestamp | 作業的 ISO8601 時間戳記 |
iothub-message-schema | moduleLifecycleNotification |

主體：本節為 JSON 格式，它表示所建立模組身分識別的對應項。 例如，

```json
{
    "deviceId":"11576-ailn-test-0-67333793211",
    "moduleId":"tempSensor",
    "etag":"AAAAAAAAAAE=",
    "properties": {
        "desired": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        },
        "reported": {
            "$metadata": {
                "$lastUpdated": "2016-02-30T16:24:48.789Z"
            },
            "$version": 1
        }
    }
}
```

## <a name="device-identity-properties"></a>裝置身分識別屬性

裝置身分識別會以具有下列屬性的 JSON 文件表示：

| 屬性 | 選項 | 說明 |
| --- | --- | --- |
| deviceId |必要，只能讀取更新 |區分大小寫的字串，最長為 128 個字元，可使用 ASCII 7 位元英數字元和某些特殊字元：`- . + % _ # * ? ! ( ) , = @ $ '`。 |
| generationId |必要，唯讀 |IoT 中樞產生的區分大小寫字串，最長為 128 個字元。 此值可用來在刪除並重建裝置時，區分具有相同 **deviceId** 的裝置。 |
| etag |必要，唯讀 |依據 [RFC7232](https://tools.ietf.org/html/rfc7232)，此字串代表裝置身分識別的弱式 ETag。 |
| auth |選用 |包含驗證資訊和安全性資料的複合物件。 |
| auth.symkey |選用 |包含主要和次要金鑰 (以 base64 格式儲存) 的複合物件。 |
| status |必要 |存取指示器。 可以是 [已啟用] 或 [已停用]。 如果為 [已啟用] ，則允許連接裝置。 如果為 [已停用] ，此裝置無法存取任何裝置面向的端點。 |
| statusReason |選用 |長度為 128 個字元的字串，用來儲存裝置身分識別狀態的原因。 允許所有 UTF-8 字元。 |
| statusUpdateTime |唯讀 |暫時指示器，顯示上次狀態更新的日期和時間。 |
| connectionState |唯讀 |指出連線狀態的欄位︰**已連線**或**已中斷連線**。 這個欄位代表裝置連線狀態的 IoT 中樞檢視。 **重要事項**：此欄位只應用於開發/偵錯用途。 只有針對使用 MQTT 或 AMQP 的裝置才會更新連線狀態。 此外，它是以通訊協定層級的偵測 (MQTT 偵測或 AMQP 偵測) 為基礎，而且最多只能有 5 分鐘的延遲。 基於這些理由，其中可能會有誤判的情形，例如將裝置回報為已連線，但卻已中斷連線。 |
| connectionStateUpdatedTime |唯讀 |暫時指示器，顯示上次更新連線狀態的日期和時間。 |
| lastActivityTime |唯讀 |暫時指示器，顯示裝置上次連接、接收或傳送訊息的日期和時間。 |

> [!NOTE]
> 連線狀態只能代表連線狀態的 IoT 中樞檢視。 根據網路狀況和組態而定，可能會延遲此狀態的更新。

## <a name="module-identity-properties"></a>模組身分識別屬性

模組身分識別會以具有下列屬性的 JSON 文件表示：

| 屬性 | 選項 | 說明 |
| --- | --- | --- |
| deviceId |必要，只能讀取更新 |區分大小寫的字串，最長為 128 個字元，可使用 ASCII 7 位元英數字元和某些特殊字元：`- . + % _ # * ? ! ( ) , = @ $ '`。 |
| moduleId |必要，只能讀取更新 |區分大小寫的字串，最長為 128 個字元，可使用 ASCII 7 位元英數字元和某些特殊字元：`- . + % _ # * ? ! ( ) , = @ $ '`。 |
| generationId |必要，唯讀 |IoT 中樞產生的區分大小寫字串，最長為 128 個字元。 此值可用來在刪除並重建裝置時，區分具有相同 **deviceId** 的裝置。 |
| etag |必要，唯讀 |依據 [RFC7232](https://tools.ietf.org/html/rfc7232)，此字串代表裝置身分識別的弱式 ETag。 |
| auth |選用 |包含驗證資訊和安全性資料的複合物件。 |
| auth.symkey |選用 |包含主要和次要金鑰 (以 base64 格式儲存) 的複合物件。 |
| status |必要 |存取指示器。 可以是 [已啟用] 或 [已停用]。 如果為 [已啟用] ，則允許連接裝置。 如果為 [已停用] ，此裝置無法存取任何裝置面向的端點。 |
| statusReason |選用 |長度為 128 個字元的字串，用來儲存裝置身分識別狀態的原因。 允許所有 UTF-8 字元。 |
| statusUpdateTime |唯讀 |暫時指示器，顯示上次狀態更新的日期和時間。 |
| connectionState |唯讀 |指出連線狀態的欄位︰**已連線**或**已中斷連線**。 這個欄位代表裝置連線狀態的 IoT 中樞檢視。 **重要事項**：此欄位只應用於開發/偵錯用途。 只有針對使用 MQTT 或 AMQP 的裝置才會更新連線狀態。 此外，它是以通訊協定層級的偵測 (MQTT 偵測或 AMQP 偵測) 為基礎，而且最多只能有 5 分鐘的延遲。 基於這些理由，其中可能會有誤判的情形，例如將裝置回報為已連線，但卻已中斷連線。 |
| connectionStateUpdatedTime |唯讀 |暫時指示器，顯示上次更新連線狀態的日期和時間。 |
| lastActivityTime |唯讀 |暫時指示器，顯示裝置上次連接、接收或傳送訊息的日期和時間。 |

## <a name="additional-reference-material"></a>其他參考資料

IoT 中樞開發人員指南中的其他參考主題包括︰

* [IoT 中樞端點](iot-hub-devguide-endpoints.md)說明每個 IoT 中樞公開給執行階段和管理作業的各種端點。

* [節流和配額](iot-hub-devguide-quotas-throttling.md)描述適用於 IoT 中樞服務的配額和節流行為。

* [Azure IoT 裝置和服務 SDK](iot-hub-devguide-sdks.md) 列出各種語言 SDK，可供您在開發與「IoT 中樞」互動的裝置和服務應用程式時使用。

* [IoT 中樞查詢語言](iot-hub-devguide-query-language.md)描述可用來從 IoT 中樞擷取有關裝置對應項和作業之資訊的查詢語言。

* [IoT 中樞 MQTT 支援](iot-hub-mqtt-support.md)針對 MQTT 通訊協定提供 IoT 中樞支援的詳細資訊。

## <a name="next-steps"></a>後續步驟

現在您已了解如何使用 IoT 中樞身分識別登錄，接下來您可能對下列 IoT 中樞開發人員指南主題感興趣：

* [控制 IoT 中樞的存取權](iot-hub-devguide-security.md)

* [使用裝置對應項同步處理狀態和組態](iot-hub-devguide-device-twins.md)

* [在裝置上叫用直接方法](iot-hub-devguide-direct-methods.md)

* [排程多個裝置上的作業](iot-hub-devguide-jobs.md)

若要嘗試本文所述的一些概念，請參閱下列「IoT 中樞」教學課程：

* [開始使用 Azure IoT 中樞](quickstart-send-telemetry-dotnet.md)

若要探索使用 IoT 中樞裝置佈建服務進行 Just-In-Time 自動佈建，請參閱： 

* [Azure IoT 中樞裝置佈建服務](https://azure.microsoft.com/documentation/services/iot-dps)
