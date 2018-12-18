---
title: 了解 Azure IoT 中樞模組對應項 | Microsoft Docs
description: 開發人員指南 - 使用模組對應項來同步處理 IoT 中樞與裝置之間的狀態和設定資料
author: chrissie926
manager: ''
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: menchi
ms.openlocfilehash: 0df1170079e66bda95e38bcf17dcce738269eeb0
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47039372"
---
# <a name="understand-and-use-module-twins-in-iot-hub"></a>了解和使用 IoT 中樞的模組對應項

本文假設您已先讀過[了解和使用 Azure IoT 中樞的裝置對應項][lnk-devguide-device-twins]。 在 IoT 中樞的每個裝置身分識別下，您可以建立最多 20 個模組身分識別。 每個模組身分識別都會隱含地產生模組對應項。 模組對應項和裝置對應項非常類似，它們是存放模組狀態資訊 (包括中繼資料、設定和條件) 的 JSON 文件。 Azure IoT 中樞會為您連線到 IoT 中樞的每個模組維護模組對應項。 

在裝置端，IoT 中樞裝置 SDK 可讓您建立會個別向 IoT 中樞建立獨立連線的模組。 這可讓您針對裝置上的不同元件使用不同的命名空間。 例如，您的販賣機具有三個不同的感應器。 每個感應器都由公司中的不同部門控制。 您可以為每個感應器建立一個模組。 如此一來，每個部門只能將作業或直接方法傳送至其所控制的感應器，以避免衝突和使用者錯誤。

 模組身分識別和模組對應項所提供的功能，和裝置身分識別與裝置對應項所提供的功能相同，但是更為細微。 如此更細微的控制，可讓相容裝置 (例如管理多個元件的作業系統型裝置或韌體裝置) 隔離那些元件的設定和條件。 模組身分識別和模組對應項在搭配具有模組化軟體元件的 IoT 裝置使用時，能隔離管理上所需關注的各種事項。 我們的目標是透過模組對應項的正式運作，在模組對應項層級上支援所有裝置對應項功能。 

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

本文章說明：

* 模組對應項的結構：*標籤*、*所需屬性*和*報告屬性*。
* 模組和後端可以對模組對應項執行的作業。

請參閱[裝置到雲端通訊指引][lnk-d2c-guidance]，以取得有關使用報告屬性、裝置到雲端訊息或檔案上傳的指引。
請參閱[雲端對裝置通訊指引][lnk-c2d-guidance]，以取得有關使用所需屬性、直接方法或雲端到裝置訊息的指引。

## <a name="module-twins"></a>模組對應項
模組對應項會儲存模組相關資訊，以供︰

* 裝置與 IoT 中樞上的模組用來同步處理模組的狀況和設定。
* 解決方案後端用來查詢和鎖定長時間執行的作業。

模組對應項的生命週期會連結至對應的[模組身分識別][lnk-identity]。 在 IoT 中樞建立或刪除模組身分識別時，系統會隱含地建立和刪除模組對應項。

模組對應項是 JSON 文件，其中含有︰

* **標籤**。 解決方案後端可以讀取及寫入的 JSON 文件區段。 裝置上的模組看不到標籤。 標籤是基於查詢用途所設定。
* **所需屬性**。 搭配報告屬性使用，以便同步處理模組的設定或條件。 解決方案後端可以設定所需屬性，而模組應用程式則可以讀取它們。 模組應用程式也可以接收所需屬性中的變更通知。
* **報告屬性**。 搭配所需屬性使用，以便同步處理模組的設定或條件。 模組應用程式可以設定報告屬性，而解決方案後端則可以讀取並查詢它們。
* **模組身分識別屬性**。 模組對應項 JSON 文件的根目錄包含來自對應模組身分識別的唯讀屬性，此身分識別儲存在[身分識別登錄][lnk-identity]中。

![][img-module-twin]

以下範例顯示模組對應項 JSON 文件︰

```json
{
    "deviceId": "devA",
    "moduleId": "moduleA",
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
            }
            "batteryLevel": 55,
            "$metadata" : {...},
            "$version": 4
        }
    }
}
```

根物件中包含的是模組身分識別屬性，以及 `tags` 和 `reported` 與 `desired` 屬性的容器物件。 `properties` 容器包含一些唯讀元素 (`$metadata`、`$etag` 和 `$version`)，其說明位於[模組對應項中繼資料][lnk-module-twin-metadata]和[開放式並行存取][lnk-concurrency]小節。

### <a name="reported-property-example"></a>報告屬性範例
在上述範例中，模組對應項包含由模組應用程式所報告的 `batteryLevel` 屬性。 此屬性可讓您根據最後一次報告的電池電量對模組進行查詢和操作。 其他範例包含模組應用程式報告模組功能或連線能力選項。

> [!NOTE]
> 當解決方案後端想要取得屬性的最後已知值時，報告屬性如何簡化這種情況。 如果解決方案後端需要以一連串時間戳記事件 (例如時間序列) 的形式處理模組遙測，請使用[裝置到雲端的訊息][lnk-d2c]。

### <a name="desired-property-example"></a>所需屬性範例
在上述範例中，解決方案後端和模組應用程式會使用 `telemetryConfig` 模組對應項的所需屬性和報告屬性，以同步處理此模組的遙測設定。 例如︰

1. 解決方案後端會以所需組態值來設定所需屬性。 以下是含有所需屬性集的文件部分︰

    ```json
    ...
    "desired": {
        "telemetryConfig": {
            "sendFrequency": "5m"
        },
        ...
    },
    ...
    ```

2. 模組應用程式會在已連線或第一次重新連線時，立即收到變更通知。 模組應用程式接著會報告更新的設定 (或使用 `status` 屬性報告錯誤狀況)。 以下是報告屬性的部分︰

    ```json
    ...
    "reported": {
        "telemetryConfig": {
            "sendFrequency": "5m",
            "status": "success"
        }
        ...
    }
    ...
    ```

3. 解決方案後端可以[查詢][lnk-query]模組對應項，以追蹤設定作業在許多模組上的結果。

> [!NOTE]
> 上述程式碼片段舉例說明用來編碼模組設定和其狀態的方式，並針對方便閱讀而進行最佳化。 IoT 中樞不會對模組對應項中的模組對應項所需屬性和報告屬性強制實行特定結構描述。
> 
> 

## <a name="back-end-operations"></a>後端作業
解決方案後端會使用下列不可部分完成的作業 (透過 HTTPS 公開) 來對模組對應項進行操作︰

* **依識別碼擷取模組對應項**。 此作業會傳回模組對應項文件，包括標籤，以及所需及報告的系統屬性。
* **局部更新模組對應項**。 此作業可讓解決方案後端局部地更新模組對應項中的標籤或所需屬性。 部分更新會以 JSON 文件的形式來表示，以新增或更新任何屬性。 設定為 `null` 的屬性會遭到移除。 下列範例會以 `{"newProperty": "newValue"}` 值建立新的所需屬性、以 `"otherNewValue"` 覆寫 `existingProperty` 的現有值，並移除 `otherOldProperty`。 不會對現有的所需屬性或標籤進行任何變更︰

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
* **接收對應項通知**。 這項作業可以在對應項修改時通知方案後端。 若要這樣做，您的 IoT 解決方案必須建立路由，並將資料來源設為等於 *twinChangeEvents*。 根據預設，不會傳送任何對應項通知，亦即預先不存在這類路由。 如果變更率太高，或基於其他原因，例如內部失敗，IoT 中樞可能只會傳送一個包含所有變更的通知。 因此，如果您的應用程式需要可靠地稽核和記錄所有中間狀態，您應該使用裝置到雲端的訊息。 對應項通知訊息包含屬性和內文。

    - properties

    | 名稱 | 值 |
    | --- | --- |
    $content-type | application/json |
    $iothub-enqueuedtime |  傳送通知的時間 |
    $iothub-message-source | twinChangeEvents |
    $content-encoding | utf-8 |
    deviceId | 裝置的識別碼 |
    moduleId | 模組的識別碼 |
    hubName | IoT 中樞名稱 |
    operationTimestamp | 作業的 [ISO8601] 時間戳記 |
    iothub-message-schema | deviceLifecycleNotification |
    opType | "replaceTwin" 或 "updateTwin" |

    訊息系統屬性前面會加上 `'$'` 符號。

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

上述所有作業皆支援[開放式並行存取][lnk-concurrency]，而且需要 **ServiceConnect** 權限，如[安全性][lnk-security]一文所定義。

除了這些作業，解決方案後端還可以︰

* 使用類似 SQL 的 [IoT 中樞查詢語言][lnk-query]來查詢模組對應項。

## <a name="module-operations"></a>模組作業
模組應用程式會使用下列不可部分完成的作業來操作模組對應項︰

* **擷取模組對應項**。 此作業會針對目前連線的模組傳回模組對應項文件 (包括標籤，以及所需及報告的系統屬性)。
* **部分更新的報告屬性**。 此作業可針對目前所連線模組的報告屬性進行局部更新。 此作業使用的 JSON 更新格式，與解決方案後端用於局部更新所需屬性的格式相同。
* **觀察所需屬性**。 目前連線的模組可以選擇在所需屬性進行更新時收到通知。 模組會收到由解決方案後端執行的相同更新形式 (部分或完整取代)。

上述所有作業都需要 **ModuleConnect** 權限，如[安全性][lnk-security]一文所定義。

[Azure IoT 裝置 SDK][lnk-sdks] 可讓您透過許多語言和平台輕鬆使用上述作業。

## <a name="tags-and-properties-format"></a>標籤和屬性格式
標籤、所需屬性和報告屬性是具有下列限制的 JSON 物件：

* JSON 物件中的所有索引鍵是區分大小寫的 64 個位元組的 UTF-8 UNICODE 字串。 允許的字元會排除 UNICODE 控制字元 (區段 C0 和 C1)，以及 `'.'`、`' '` 和 `'$'`。
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

## <a name="module-twin-size"></a>模組對應項大小
「IoT 中樞」會對 `tags`、`properties/desired` 和 `properties/reported` 的個別總計值 (排除唯讀元素) 分別強制執行 8 KB 大小的限制。
大小的計算方式是計算所有字元的數量，並排除在字串常數之外的 UNICODE 控制字元 (區段 C0 和 C1) 和空格。
IoT 中樞會拒絕 (並出現錯誤) 將會讓這些文件的大小增加到超過限制的所有作業。

## <a name="module-twin-metadata"></a>模組對應項中繼資料
IoT 中樞會為模組對應項所需屬性和報告屬性中的每個 JSON 物件保有上次更新的時間戳記。 時間戳記採用 UTC 格式，並以 [ISO8601] 格式 `YYYY-MM-DDTHH:MM:SS.mmmZ` 進行編碼。
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
依據 [RFC7232]，標籤會有一個 ETag 來表示該標籤的 JSON 表示法。 您可以從解決方案後端在條件式更新作業中使用 ETag，以確保一致性。

模組對應項所需屬性和報告屬性沒有 ETag，但是有一定會遞增的 `$version` 值。 類似於 ETag，更新端可以使用版本強制達到更新的一致性。 例如報告屬性的模組應用程式，或是所需屬性的解決方案後端。

當觀察端代理程式 (例如，觀察所需屬性的模組應用程式) 必須協調擷取作業結果與更新通知之間的競爭情況時，版本也相當有用。 [裝置重新連線流程][lnk-reconnection]一節會提供詳細資訊。 

## <a name="next-steps"></a>後續步驟
若要嘗試本文所述的一些概念，請參閱下列「IoT 中樞」教學課程：

* [透過 .NET 後端和 .NET 裝置開始使用 IoT 中樞模組身分識別和模組對應項][lnk-module-twin-tutorial]

<!-- links and images -->

[lnk-devguide-device-twins]: iot-hub-devguide-device-twins.md
[lnk-endpoints]: iot-hub-devguide-endpoints.md
[lnk-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-sdks]: iot-hub-devguide-sdks.md
[lnk-query]: iot-hub-devguide-query-language.md
[lnk-identity]: iot-hub-devguide-identity-registry.md
[lnk-d2c]: iot-hub-devguide-messages-d2c.md
[lnk-methods]: iot-hub-devguide-direct-methods.md
[lnk-security]: iot-hub-devguide-security.md
[lnk-c2d-guidance]: iot-hub-devguide-c2d-guidance.md
[lnk-d2c-guidance]: iot-hub-devguide-d2c-guidance.md
[ISO8601]: https://en.wikipedia.org/wiki/ISO_8601
[RFC7232]: https://tools.ietf.org/html/rfc7232

[lnk-module-twin-tutorial]: iot-hub-csharp-csharp-module-twin-getstarted.md
[lnk-module-twin-metadata]: iot-hub-devguide-module-twins.md#module-twin-metadata
[lnk-concurrency]: iot-hub-devguide-device-twins.md#optimistic-concurrency
[img-module-twin]: media/iot-hub-devguide-device-twins/module-twin.jpg
