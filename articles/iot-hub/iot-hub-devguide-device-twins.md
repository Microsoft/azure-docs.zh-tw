---
title: 了解 Azure IoT 中樞裝置對應項 | Microsoft Docs
description: 開發人員指南 - 使用裝置對應項同步處理 IoT 中樞與裝置之間的狀態和組態資料
author: fsautomata
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: elioda
ms.openlocfilehash: 558bf0eb6ab4abb4ad16910ebe36fdb7c1e19374
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49310924"
---
# <a name="understand-and-use-device-twins-in-iot-hub"></a>了解和使用 Azure IoT 中樞的裝置對應項

*裝置對應項*是存放裝置狀態資訊的 JSON 文件，包括中繼資料、組態和條件。 Azure IoT 中樞會為連線到 IoT 中樞的每個裝置維持裝置對應項。 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

本文章說明：

* 裝置對應項的結構：*標籤*、*所需屬性*和*報告屬性*。
* 裝置應用程式和後端可以對裝置對應項執行的作業。

使用裝置對應項可以：

* 在雲端儲存裝置特定的中繼資料。 例如，販賣機的部署位置。

* 從您的裝置應用程式報告目前狀態資訊，例如可用功能和狀況。 例如，裝置會透過行動電話或 WiFi 連線到您的 IoT 中樞。

* 同步處理裝置與後端應用程式之間長時間執行之工作流程的狀態。 例如，當解決方案後端指定要安裝的新韌體版本時，以及裝置應用程式報告更新處理程序的各個階段時。

* 查詢裝置中繼資料、組態或狀態。

請參閱[裝置到雲端通訊指引](iot-hub-devguide-d2c-guidance.md)，以取得有關使用報告屬性、裝置到雲端訊息或檔案上傳的指引。

請參閱[雲端對裝置通訊指引](iot-hub-devguide-c2d-guidance.md)，以取得有關使用所需屬性、直接方法或雲端到裝置訊息的指引。

## <a name="device-twins"></a>裝置對應項

裝置對應項會儲存裝置相關資訊，以供︰

* 裝置和後端用來同步處理裝置的狀況和組態。

* 解決方案後端用來查詢和鎖定長時間執行的作業。

裝置對應項的生命週期會連結至對應的[裝置身分識別](iot-hub-devguide-identity-registry.md)。 在 IoT 中樞建立或刪除裝置身分識別時，會隱含地建立和刪除裝置對應項。

裝置的兩個是 JSON 文件，其中含有︰

* **標籤**。 解決方案後端可以讀取及寫入的 JSON 文件區段。 裝置應用程式看不到標籤。

* **所需屬性**。 搭配報告屬性使用，以便同步處理裝置的組態或狀況。 解決方案後端可以設定所需的屬性，以及裝置應用程式可以讀取它們。 裝置應用程式也可以接收所需屬性中的變更通知。

* **報告屬性**。 搭配所需屬性使用，以便同步處理裝置的組態或狀況。 裝置應用程式可以設定報告的屬性，以及解決方案後端可以讀取並查詢它們。

* **裝置身分識別屬性**。 裝置對應項 JSON 文件的根目錄包含來自對應之裝置身分識別的唯讀屬性，此身分識別儲存在[身分識別登錄](iot-hub-devguide-identity-registry.md)中。

![裝置對應項屬性的螢幕擷取畫面](./media/iot-hub-devguide-device-twins/twin.png)

以下範例顯示裝置對應項 JSON 文件︰

```json
{
    "deviceId": "devA",
    "etag": "AAAAAAAAAAc=", 
    "status": "enabled",
    "statusReason": "provisioned",
    "statusUpdateTime": "0001-01-01T00:00:00",
    "connectionState": "connected",
    "lastActivityTime": "2015-02-30T16:24:48.789Z",
    "cloudToDeviceMessageCount": 0, 
    "authenticationType": "sas",
    "x509Thumbprint": {     
        "primaryThumbprint": null, 
        "secondaryThumbprint": null 
    }, 
    "version": 2, 
    "tags": {
        "$etag": "123",
        "deploymentLocation": {
            "building": "43",
            "floor": "1"
        }
    },
    "properties": {
        "desired": {
            "telemetryConfig": {
                "sendFrequency": "5m"
            },
            "$metadata" : {...},
            "$version": 1
        },
        "reported": {
            "telemetryConfig": {
                "sendFrequency": "5m",
                "status": "success"
            },
            "batteryLevel": 55,
            "$metadata" : {...},
            "$version": 4
        }
    }
}
```

根物件中包含的是裝置身分識別屬性，以及 `tags` 和 `reported` 與 `desired` 兩屬性的容器物件。 `properties` 容器包含一些唯讀元素 (`$metadata`、`$etag` 和 `$version`)，其說明位於[裝置對應項中繼資料](iot-hub-devguide-device-twins.md#device-twin-metadata)和[開放式並行存取](iot-hub-devguide-device-twins.md#optimistic-concurrency)章節。

### <a name="reported-property-example"></a>報告屬性範例

在上述範例中，裝置對應項包含由裝置應用程式所報告的 `batteryLevel` 屬性。 此屬性可讓您根據最後一次報告的電池電量對裝置進行查詢和操作。 其他範例包含裝置應用程式報告功能或連線能力選項。

> [!NOTE]
> 當解決方案後端想要取得屬性的最後已知值時，報告屬性如何簡化這種情況。 如果解決方案後端需要以一連串時間戳記事件 (例如時間序列) 的形式處理裝置遙測，請使用[裝置到雲端訊息](iot-hub-devguide-messages-d2c.md)。

### <a name="desired-property-example"></a>所需屬性範例

在上述範例中，解決方案後端和裝置應用程式會使用 `telemetryConfig` 裝置對應項的所需屬性和報告屬性，以同步處理此裝置的遙測組態。 例如︰

1. 解決方案後端會以所需組態值來設定所需屬性。 以下是含有所需屬性集的文件部分︰

   ```json
   "desired": {
       "telemetryConfig": {
           "sendFrequency": "5m"
       },
       ...
   },
   ```

2. 裝置應用程式會在已連線或第一次重新連線時，立即收到變更通知。 裝置應用程式接著會報告更新的組態 (或使用 `status` 屬性的錯誤狀況)。 以下是報告屬性的部分︰

   ```json
   "reported": {
       "telemetryConfig": {
           "sendFrequency": "5m",
           "status": "success"
       }
       ...
   }
   ```

3. 解決方案後端可以[查詢](iot-hub-devguide-query-language.md)裝置對應項，以追蹤組態作業在許多裝置上的結果。

> [!NOTE]
> 上述程式碼片段舉例說明了用來編碼裝置組態和其狀態的方式，為了方便閱讀，其內容已經過最佳化。 IoT 中樞不會對裝置對應項中的裝置對應項所需屬性和報告屬性強制實行特定結構描述。
> 

您可以使用裝置對應項來同步處理長時間執行的作業，例如韌體更新。 如需如何使用屬性來跨裝置同步處理及追蹤長時間執行的作業，請參閱[使用所需屬性來設定裝置](tutorial-device-twins.md)。

## <a name="back-end-operations"></a>後端作業

解決方案後端會使用下列不可部分完成的作業 (透過 HTTPS 公開) 來對裝置對應項進行操作︰

* **依識別碼擷取裝置對應項**。 此作業會傳回裝置對應項文件，包括標籤以及所需屬性、報告屬性和系統屬性。

* **部份更新裝置對應項**。 此作業可讓解決方案後端局部地更新裝置對應項中的標籤或所需屬性。 部分更新會以 JSON 文件的形式來表示，以新增或更新任何屬性。 設定為 `null` 的屬性會遭到移除。 下列範例會以 `{"newProperty": "newValue"}` 值建立新的所需屬性、以 `"otherNewValue"` 覆寫 `existingProperty` 的現有值，並移除 `otherOldProperty`。 不會對現有的所需屬性或標籤進行任何變更︰

   ```json
   {
       "properties": {
           "desired": {
               "newProperty": {
                   "nestedProperty": "newValue"
               },
               "existingProperty": "otherNewValue",
               "otherOldProperty": null
           }
       }
   }
   ```

* **取代所需屬性**。 此作業可讓解決方案後端完全覆寫所有現有的所需屬性，並以新的 JSON 文件取代 `properties/desired`。

* **取代標籤**。 此作業可讓解決方案後端完全覆寫所有現有的標籤，並以新的 JSON 文件取代 `tags`。

* **接收對應項通知**。 這項作業可以在對應項修改時通知方案後端。 若要這樣做，您的 IoT 解決方案必須建立路由，並將資料來源設為等於 *twinChangeEvents*。 根據預設，預先不存在這類路由，因此不會傳送任何對應項通知。 如果變更率太高，或基於其他原因，例如內部失敗，IoT 中樞可能只會傳送一個包含所有變更的通知。 因此，如果您的應用程式需要可靠地稽核和記錄所有中間狀態，您應該使用裝置到雲端的訊息。 對應項通知訊息包含屬性和內文。

   - properties

   | 名稱 | 值 |
   | --- | --- |
   $content-type | application/json |
   $iothub-enqueuedtime |  傳送通知的時間 |
   $iothub-message-source | twinChangeEvents |
   $content-encoding | utf-8 |
   deviceId | 裝置的識別碼 |
   hubName | IoT 中樞名稱 |
   operationTimestamp | 作業的 [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) 時間戳記 |
   iothub-message-schema | deviceLifecycleNotification |
   opType | "replaceTwin" 或 "updateTwin" |

   訊息系統屬性前面會加上 `$` 符號。

   - body
        
   本節包含所有對應項變更 (JSON 格式)。 它使用的格式與修補程式的格式相同，差別在於它可以包含所有對應項區段︰tags、properties.reported、properties.desired，而且包含 “$metadata” 項目。 例如，

   ```json
   {
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

上述所有作業皆支援[開放式並行存取](iot-hub-devguide-device-twins.md#optimistic-concurrency)，而且需要 **ServiceConnect** 權限，如[控制 IoT 中樞的存取權](iot-hub-devguide-security.md)中所定義。

除了這些作業，解決方案後端還可以︰

* 使用類似 SQL 的 [IoT 中樞查詢語言](iot-hub-devguide-query-language.md)來查詢裝置對應項。

* 使用[作業](iot-hub-devguide-jobs.md)在大型裝置對應項集合上執行作業。

## <a name="device-operations"></a>裝置作業

裝置應用程式會使用下列不可部分完成的作業來操作裝置對應項︰

* **擷取裝置對應項**。 此作業會針對目前連線的裝置傳回裝置對應項文件 (包括標籤以及所需屬性、報告屬性和系統屬性)。

* **部分更新的報告屬性**。 此作業可針對目前連線裝置的報告屬性進行部分更新。 此作業使用的 JSON 更新格式，與解決方案後端用於局部更新所需屬性的格式相同。

* **觀察所需屬性**。 目前連線的裝置可以選擇在所需屬性進行更新時收到通知。 裝置會收到由解決方案後端執行的相同更新形式 (部分或完整取代)。

上述所有作業都需要 **DeviceConnect** 權限，如[控制 IoT 中樞的存取權](iot-hub-devguide-security.md)中所定義。

[Azure IoT 裝置 SDK](iot-hub-devguide-sdks.md) 可讓您透過許多語言和平台輕鬆使用上述作業。 如需 IoT 中樞之所需屬性同步處理基元的詳細資訊，請參閱[裝置重新連線流程](iot-hub-devguide-device-twins.md#device-reconnection-flow)。

## <a name="tags-and-properties-format"></a>標籤和屬性格式

標籤、所需屬性和報告屬性是具有下列限制的 JSON 物件：

* JSON 物件中的所有索引鍵是區分大小寫的 64 個位元組的 UTF-8 UNICODE 字串。 允許的字元會排除 UNICODE 控制字元 (區段 C0 和 C1)，以及 `.`、`$` 和 SP。

* JSON 物件中的所有值可以屬於下列 JSON 類型︰布林值、數字、字串、物件。 不允許使用陣列。 整數的最大值是 4503599627370495 和整數的最小值是 -4503599627370496。

* 標籤、所需屬性和報告屬性中所有 JSON 物件的深度上限為 5。 例如，下列物件有效：

   ```json
   {
       ...
       "tags": {
           "one": {
               "two": {
                   "three": {
                       "four": {
                           "five": {
                               "property": "value"
                           }
                       }
                   }
               }
           }
       },
       ...
   }
   ```

* 所有字串值的最大長度為 512 個位元組。

## <a name="device-twin-size"></a>裝置對應項大小

「IoT 中樞」會對 `tags`、`properties/desired` 和 `properties/reported` 的個別總計值 (排除唯讀元素) 分別強制執行 8 KB 大小的限制。

大小的計算方式是計算所有字元的數量，並排除在字串常數之外的 UNICODE 控制字元 (區段 C0 和 C1) 和空格。

IoT 中樞會拒絕 (並出現錯誤) 將會讓這些文件的大小增加到超過限制的所有作業。

## <a name="device-twin-metadata"></a>裝置對應項中繼資料

IoT 中樞會為裝置對應項所需屬性和報告屬性的每個 JSON 物件保有上次更新的時間戳記。 時間戳記採用 UTC 格式，並以 [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) 格式 `YYYY-MM-DDTHH:MM:SS.mmmZ` 進行編碼。

例如︰

```json
{
    ...
    "properties": {
        "desired": {
            "telemetryConfig": {
                "sendFrequency": "5m"
            },
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": {
                        "$lastUpdated": "2016-03-30T16:24:48.789Z"
                    },
                    "$lastUpdated": "2016-03-30T16:24:48.789Z"
                },
                "$lastUpdated": "2016-03-30T16:24:48.789Z"
            },
            "$version": 23
        },
        "reported": {
            "telemetryConfig": {
                "sendFrequency": "5m",
                "status": "success"
            }
            "batteryLevel": "55%",
            "$metadata": {
                "telemetryConfig": {
                    "sendFrequency": "5m",
                    "status": {
                        "$lastUpdated": "2016-03-31T16:35:48.789Z"
                    },
                    "$lastUpdated": "2016-03-31T16:35:48.789Z"
                }
                "batteryLevel": {
                    "$lastUpdated": "2016-04-01T16:35:48.789Z"
                },
                "$lastUpdated": "2016-04-01T16:24:48.789Z"
            },
            "$version": 123
        }
    }
    ...
}
```

此資訊會保留在每個層級 (而不只是 JSON 結構的分葉)，以保留可移除物件索引鍵的更新。

## <a name="optimistic-concurrency"></a>開放式並行存取

標籤、所需屬性和報告屬性全都支援開放式並行存取。
依據 [RFC7232](https://tools.ietf.org/html/rfc7232)，標籤會有一個 ETag 來表示該標籤的 JSON 表示法。 您可以從解決方案後端在條件式更新作業中使用 ETag，以確保一致性。

裝置對應項所需屬性和報告屬性沒有 ETag，但是有一定會遞增的 `$version` 值。 類似於 ETag，更新端可以使用版本強制達到更新的一致性。 例如，報告屬性的裝置應用程式或所需屬性的解決方案後端。

當觀察端代理程式 (例如，觀察所需屬性的裝置應用程式) 必須協調擷取作業結果與更新通知之間的競爭情況時，版本也相當有用。 [裝置重新連線流程](iot-hub-devguide-device-twins.md#device-reconnection-flow)一節會提供詳細資訊。

## <a name="device-reconnection-flow"></a>裝置重新連線流程

IoT 中樞不會保留已中斷連線之裝置的所需屬性更新通知。 因此，連線的裝置必須擷取完整的所需屬性文件，並訂閱更新通知。 由於更新通知和完整擷取之間可能會有競爭情況，因此必須確保下列流程︰

1. 裝置應用程式連線到 IoT 中樞。
2. 裝置應用程式訂閱所需屬性更新通知。
3. 裝置應用程式擷取所需屬性的完整文件。

裝置應用程式可以忽略 `$version` 小於或等於完整擷取文件版本的所有通知。 IoT 中樞保證版本一定會遞增，因此這是可行的方法。

> [!NOTE]
> [Azure IoT 裝置 SDK](iot-hub-devguide-sdks.md) 中已實作此邏輯。 只有當裝置應用程式無法使用任何 Azure IoT 裝置 SDK，因而必須直接為 MQTT 介面編寫程式時，此說明才有用。
> 

## <a name="additional-reference-material"></a>其他參考資料

IoT 中樞開發人員指南中的其他參考主題包括︰

* [IoT 中樞端點](iot-hub-devguide-endpoints.md)一文說明每個 IoT 中樞針對執行階段和管理作業所公開的各種端點。

* [節流和配額](iot-hub-devguide-quotas-throttling.md)一文說明適用於 IoT 中樞服務的配額，和使用服務時所預期的節流行為。

* [Azure IoT 裝置和服務 SDK](iot-hub-devguide-sdks.md) 一文列出各種語言 SDK，可供您在開發與「IoT 中樞」互動的裝置和服務應用程式時使用。

* [裝置對應項、作業和訊息路由的 IoT 中樞查詢語言](iot-hub-devguide-query-language.md)一文說明可用於從 IoT 中樞擷取有關裝置對應項和作業資訊的 IoT 中樞查詢語言。

* [IoT 中樞 MQTT 支援](iot-hub-mqtt-support.md)一文針對 MQTT 通訊協定提供 IoT 中樞支援的詳細資訊。

## <a name="next-steps"></a>後續步驟

現在您已了解裝置對應項，接下來您可能會對下列 IoT 中樞開發人員指南主題感興趣︰

* [了解和使用 IoT 中樞的模組對應項](iot-hub-devguide-module-twins.md)
* [在裝置上叫用直接方法](iot-hub-devguide-direct-methods.md)
* [排程多個裝置上的作業](iot-hub-devguide-jobs.md)

若要嘗試本文所述的一些概念，請參閱下列「IoT 中樞」教學課程：

* [如何使用裝置對應項](iot-hub-node-node-twin-getstarted.md)
* [如何使用裝置對應項屬性](tutorial-device-twins.md)
* [使用適用於 VS Code 的 Azure IoT 工具組管理裝置](iot-hub-device-management-iot-toolkit.md)
