---
title: 了解 Azure IoT 中樞查詢語言 | Microsoft Docs
description: 開發人員指南 - 說明類似 SQL 的 IoT 中樞查詢語言，用於從 IoT 中樞擷取裝置/模組對應項和作業的相關資訊。
author: fsautomata
manager: ''
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/26/2018
ms.author: elioda
ms.openlocfilehash: 2e4b356fec642e06e3223700967eeacd19f1c49c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952472"
---
# <a name="iot-hub-query-language-for-device-and-module-twins-jobs-and-message-routing"></a>裝置與模組對應項、作業和訊息路由的 IoT 中樞查詢語言

IoT 中樞提供功能強大、類似 SQL 的語言，來擷取有關[裝置對應項][lnk-twins]、[作業][lnk-jobs]和[訊息路由][lnk-devguide-messaging-routes]的資訊。 本文提供︰

* IoT 中樞查詢語言主要功能的簡介，以及
* 語言的詳細說明。 如需訊息路由查詢語言的詳細資訊，請參閱[訊息路由中的查詢](../iot-hub/iot-hub-devguide-routing-query-syntax.md)。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="device-and-module-twin-queries"></a>裝置與模組對應項查詢
[裝置對應項][lnk-twins]與模組對應項可以包含標籤和屬性形式的任意 JSON 物件。 IoT 中樞可讓您以包含所有對應項資訊的單一 JSON 文件形式查詢裝置對應項與模組對應項。
例如，假設 IoT 中樞裝置對應項有下列結構 (模組對應項很類似，只是具有額外的 moduleId)：

```json
{
    "deviceId": "myDeviceId",
    "etag": "AAAAAAAAAAc=",
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00",    
    "connectionState": "Disconnected",    
    "lastActivityTime": "0001-01-01T00:00:00",
    "cloudToDeviceMessageCount": 0,
    "authenticationType": "sas",    
    "x509Thumbprint": {    
        "primaryThumbprint": null,
        "secondaryThumbprint": null
    },
    "version": 2,
    "tags": {
        "location": {
            "region": "US",
            "plant": "Redmond43"
        }
    },
    "properties": {
        "desired": {
            "telemetryConfig": {
                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
                "sendFrequencyInSecs": 300
            },
            "$metadata": {
            ...
            },
            "$version": 4
        },
        "reported": {
            "connectivity": {
                "type": "cellular"
            },
            "telemetryConfig": {
                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
                "sendFrequencyInSecs": 300,
                "status": "Success"
            },
            "$metadata": {
            ...
            },
            "$version": 7
        }
    }
}
```

### <a name="device-twin-queries"></a>裝置對應項查詢

IoT 中樞可以將裝置對應項公開為稱為**裝置**的文件集合。
因此，下列查詢會擷取整組裝置對應項︰

```sql
SELECT * FROM devices
```

> [!NOTE]
> [Azure IoT SDK][lnk-hub-sdks] 支援將大型結果分頁。

IoT 中樞允許擷取使用任意條件進行的裝置對應項篩選。 例如，若要收到 **location.region** 標記設定為 **US** 的裝置對應項，請使用下列查詢：

```sql
SELECT * FROM devices
WHERE tags.location.region = 'US'
```

布林運算子和算術比較也受到支援。 例如，若要擷取位於美國且遙測傳送頻率設定為比每分鐘還低的裝置對應項，請使用下列查詢：

```sql
SELECT * FROM devices
WHERE tags.location.region = 'US'
    AND properties.reported.telemetryConfig.sendFrequencyInSecs >= 60
```

為了方便起見，您也可以使用陣列常數搭配 **IN** 和 **NIN** (不在) 運算子。 例如，若要擷取回報 WiFi 或有線連線的裝置對應項，請使用下列查詢：

```sql
SELECT * FROM devices
WHERE properties.reported.connectivity IN ['wired', 'wifi']
```

您通常必須識別包含特定屬性的所有裝置對應項。 為了實現此目的，IoT 中樞支援 `is_defined()` 函式。 例如，若要擷取定義 `connectivity` 屬性的裝置對應項，請使用下列查詢：

```SQL
SELECT * FROM devices
WHERE is_defined(properties.reported.connectivity)
```

請參閱 [WHERE 子句][lnk-query-where]一節，以取得篩選功能的完整參考。

群組和彙總也受到支援。 例如，若要尋找處於各個遙測組態狀態的裝置計數，請使用下列查詢：

```sql
SELECT properties.reported.telemetryConfig.status AS status,
    COUNT() AS numberOfDevices
FROM devices
GROUP BY properties.reported.telemetryConfig.status
```

這個群組查詢會傳回類似以下範例的結果：

```json
[
    {
        "numberOfDevices": 3,
        "status": "Success"
    },
    {
        "numberOfDevices": 2,
        "status": "Pending"
    },
    {
        "numberOfDevices": 1,
        "status": "Error"
    }
]
```

在此範例中，三個裝置回報設定成功，兩個仍在套用設定，還有一個回報發生錯誤。

投影查詢可讓開發人員只傳回他們關心的屬性。 例如，若要擷取所有已中斷連線之裝置的最後一個活動時間，請使用下列查詢：

```sql
SELECT LastActivityTime FROM devices WHERE status = 'enabled'
```

### <a name="module-twin-queries"></a>模組對應項查詢

查詢模組對應項和查詢裝置對應項很類似，但使用不同的集合/命名空間，也就是不用您可以查詢的 “from devices”

```sql
SELECT * FROM devices.modules
```

我們不允許 devices 與 devices.modules 集合之間的聯結。 如果您想要跨裝置查詢模組對應項，可以根據標籤來執行。 此查詢會傳回所有裝置上具有 scanning 狀態的所有模組對應項：

```sql
Select * from devices.modules where properties.reported.status = 'scanning'
```

此查詢會傳回具有 scanning 狀態的所有模組對應項，但僅以指定的裝置子集為限。

```sql
Select * from devices.modules where properties.reported.status = 'scanning' and deviceId IN ('device1', 'device2')  
```

### <a name="c-example"></a>C# 範例
查詢功能由 [C# 服務 SDK][lnk-hub-sdks] 在 **RegistryManager** 類別中公開。
以下是簡單查詢的範例︰

```csharp
var query = registryManager.CreateQuery("SELECT * FROM devices", 100);
while (query.HasMoreResults)
{
    var page = await query.GetNextAsTwinAsync();
    foreach (var twin in page)
    {
        // do work on twin object
    }
}
```

**query** 物件已搭配頁面大小 (最高可達 100) 來具現化。 接著，會藉由呼叫 **GetNextAsTwinAsync** 方法多次來擷取多個頁面。

query 物件會根據查詢所需的還原序列化選項，公開多個 **Next** 值。 例如，裝置對應項或作業物件，或是單純 JSON (使用投影時)。

### <a name="nodejs-example"></a>Node.js 範例
查詢功能由[適用於 Node.js 的 Azure IoT 服務 SDK][lnk-hub-sdks] 在 **Registry** 物件中公開。
以下是簡單查詢的範例︰

```nodejs
var query = registry.createQuery('SELECT * FROM devices', 100);
var onResults = function(err, results) {
    if (err) {
        console.error('Failed to fetch the results: ' + err.message);
    } else {
        // Do something with the results
        results.forEach(function(twin) {
            console.log(twin.deviceId);
        });

        if (query.hasMoreResults) {
            query.nextAsTwin(onResults);
        }
    }
};
query.nextAsTwin(onResults);
```

**query** 物件已搭配頁面大小 (最高可達 100) 來具現化。 接著，會藉由呼叫 **nextAsTwin** 方法多次來擷取多個頁面。

query 物件會根據查詢所需的還原序列化選項，公開多個 **Next** 值。 例如，裝置對應項或作業物件，或是單純 JSON (使用投影時)。

### <a name="limitations"></a>限制

> [!IMPORTANT]
> 根據裝置對應項中的最新值，查詢結果可能會有數分鐘的延遲。 如果要依據識別碼來查詢個別裝置對應項，請使用擷取裝置對應項 API。 此 API 一律會包含最新的值，並且具有較高的節流限制。

目前僅支援在基本類型 (沒有物件) 之間進行比較，例如 `... WHERE properties.desired.config = properties.reported.config` 只會在這些屬性具有基本值時才受到支援。

## <a name="get-started-with-jobs-queries"></a>開始使用作業查詢

[作業][lnk-jobs]可提供方法來對裝置組執行作業。 每個裝置對應項皆包含屬於 **jobs** 集合一部分之作業的資訊。
在邏輯上，

```json
{
    "deviceId": "myDeviceId",
    "etag": "AAAAAAAAAAc=",
    "tags": {
        ...
    },
    "properties": {
        ...
    },
    "jobs": [
        {
            "deviceId": "myDeviceId",
            "jobId": "myJobId",
            "jobType": "scheduleTwinUpdate",
            "status": "completed",
            "startTimeUtc": "2016-09-29T18:18:52.7418462",
            "endTimeUtc": "2016-09-29T18:20:52.7418462",
            "createdDateTimeUtc": "2016-09-29T18:18:56.7787107Z",
            "lastUpdatedDateTimeUtc": "2016-09-29T18:18:56.8894408Z",
            "outcome": {
                "deviceMethodResponse": null
            }
        },
        ...
    ]
}
```

目前，此集合可在 IoT 中樞查詢語言中以 **devices.jobs** 的形式來查詢。

> [!IMPORTANT]
> 目前，在查詢裝置對應項時，一律不會傳回 jobs 屬性。 亦即包含 'FROM devices' 的查詢。 存取 jobs 屬性時，只能使用 `FROM devices.jobs` 直接透過查詢來存取。
>
>

例如，若要取得影響單一裝置的所有作業 (過去的和排程的)，您可以使用下列查詢︰

```sql
SELECT * FROM devices.jobs
WHERE devices.jobs.deviceId = 'myDeviceId'
```

請注意這個查詢如何為每個傳回的作業提供裝置特定的狀態 (可能的話還會提供直接方法回應)。
您也可以在 **devices.jobs** 集合的所有物件屬性上，使用任意布林值條件進行篩選。
例如，若要擷取 2016 年 9 月之後為特定裝置建立的所有已完成裝置對應項更新作業，請使用下列查詢：

```sql
SELECT * FROM devices.jobs
WHERE devices.jobs.deviceId = 'myDeviceId'
    AND devices.jobs.jobType = 'scheduleTwinUpdate'
    AND devices.jobs.status = 'completed'
    AND devices.jobs.createdTimeUtc > '2016-09-01'
```

您也可以擷取單一作業的每一裝置結果。

```sql
SELECT * FROM devices.jobs
WHERE devices.jobs.jobId = 'myJobId'
```

### <a name="limitations"></a>限制
**devices.jobs** 上的查詢目前不支援︰

* 投影，因此只有 `SELECT *` 是可行的。
* 參照裝置對應項 (作業屬性除外) 的條件 (請參閱上一節)。
* 執行彙總，例如計數、平均、分組依據。

## <a name="basics-of-an-iot-hub-query"></a>IoT 中樞查詢的基本概念
每個「IoT 中樞」查詢都包含 SELECT 和 FROM 子句，以及選擇性的 WHERE 和 GROUP BY 子句。 每個查詢都會在 JSON 文件的集合上執行，例如裝置對應項。 FROM 子句會指出要在其上反覆運算的文件集合 (**devices** 或 **devices.jobs**)。 然後，會套用 WHERE 子句中的篩選。 使用彙總時，此步驟的結果會依照 GROUP BY 子句中所指定的方式進行分組。 針對每個群組，會依照 SELECT 子句中所指定的方式產生一個資料列。

```sql
SELECT <select_list>
FROM <from_specification>
[WHERE <filter_condition>]
[GROUP BY <group_specification>]
```

## <a name="from-clause"></a>FROM 子句
**FROM <from_specification>** 子句只能採用兩個值︰**FROM devices** (用來查詢裝置對應項) 或 **FROM devices.jobs** (用來查詢每一裝置的作業詳細資料)。

## <a name="where-clause"></a>WHERE 子句
**WHERE <filter_condition>** 子句是選擇性的。 它會指定一或多個條件，而且 FROM 集合中的 JSON 文件必須滿足這些條件，才能納入為結果的一部分。 任何 JSON 文件都必須將指定的條件評估為 "true"，才能併入結果。

[運算式和條件][lnk-query-expressions]一節中會說明允許的條件。

## <a name="select-clause"></a>SELECT 子句
**SELECT <select_list>** 是必要子句，可指定要從查詢擷取的值。 它會指定用來產生新 JSON 物件的 JSON 值。
針對已篩選 (及視需要已分組) 之 FROM 集合子集的每個項目，投影階段會產生一個新的 JSON 物件。 此物件會以 SELECT 子句中所指定的值來建構。

以下是 SELECT 子句的文法︰

```
SELECT [TOP <max number>] <projection list>

<projection_list> ::=
    '*'
    | <projection_element> AS alias [, <projection_element> AS alias]+

<projection_element> :==
    attribute_name
    | <projection_element> '.' attribute_name
    | <aggregate>

<aggregate> :==
    count()
    | avg(<projection_element>)
    | sum(<projection_element>)
    | min(<projection_element>)
    | max(<projection_element>)
```

**attribute_name** 指的是 FROM 集合中 JSON 文件的任何屬性。 您可以在[開始使用裝置對應項查詢][lnk-query-getstarted]一節中找到一些 SELECT 子句範例。

目前，只有在裝置對應項的彙總查詢中，才支援與 **SELECT*** 不同的選取範圍子句。

## <a name="group-by-clause"></a>GROUP BY 子句
**GROUP BY <group_specification>** 子句是一個選擇性步驟，會在 WHERE 子句中指定的篩選之後、SELECT 中指定的投影之前執行。 它會根據屬性值來分組文件。 這些群組可用來產生 SELECT 子句中所指定的彙總值。

使用 GROUP BY 的查詢範例如下︰

```sql
SELECT properties.reported.telemetryConfig.status AS status,
    COUNT() AS numberOfDevices
FROM devices
GROUP BY properties.reported.telemetryConfig.status
```

GROUP BY 的正式語法如下︰

```
GROUP BY <group_by_element>
<group_by_element> :==
    attribute_name
    | < group_by_element > '.' attribute_name
```

**attribute_name** 指的是 FROM 集合中 JSON 文件的任何屬性。

目前，GROUP BY 子句只支援在查詢裝置對應項時使用。

## <a name="expressions-and-conditions"></a>運算式和條件
概括而言，運算式：

* 會評估為 JSON 類型 (例如布林值、數字、字串、陣列或物件) 的執行個體。
* 定義方式是使用內建運算子和函式處理來自裝置 JSON 文件和常數的資料。

「條件」是評估為布林值的運算式。 任何與布林值 **true** 不同的常數都會被視為 **false**。 此規則包括 **null****undefined**、任何物件或陣列執行個體、任何字串，以及布林值 **false**。

運算式的語法如下︰

```
<expression> ::=
    <constant> |
    attribute_name |
    <function_call> |
    <expression> binary_operator <expression> |
    <create_array_expression> |
    '(' <expression> ')'

<function_call> ::=
    <function_name> '(' expression ')'

<constant> ::=
    <undefined_constant>
    | <null_constant>
    | <number_constant>
    | <string_constant>
    | <array_constant>

<undefined_constant> ::= undefined
<null_constant> ::= null
<number_constant> ::= decimal_literal | hexadecimal_literal
<string_constant> ::= string_literal
<array_constant> ::= '[' <constant> [, <constant>]+ ']'
```

若要了解運算式語法中每個符號所代表的意義，請參閱下表：

| 符號 | 定義 |
| --- | --- |
| attribute_name | **FROM** 集合中 JSON 文件的任何屬性。 |
| binary_operator | [運算子](#operators)一節中所列的任何二元運算子。 |
| function_name| [函式](#functions)一節中所列的任何函式。 |
| decimal_literal |以小數點標記法表示的浮點數。 |
| hexadecimal_literal |以字串 '0x' 後面接著十六進位數字的字串所表示的數字。 |
| string_literal |字串常值是由零個或多個 Unicode 字元序列或逸出序列所表示的 Unicode 字串。 字串常值會以單引號或雙引號括起來。 允許的逸出︰`\'`、`\"`、`\\`、`\uXXXX` (適用於由 4 個十六進位數字所定義的 Unicode 字元)。 |

### <a name="operators"></a>運算子
支援下列運算子：

| 系列 | 運算子 |
| --- | --- |
| 算術 |+, -, *, /, % |
| 邏輯 |AND、OR、NOT |
| 比較 |=、!=、<、>、<=、>=、<> |

### <a name="functions"></a>函式
查詢對應項和作業時唯一支援的函式為：

| 函式 | 說明 |
| -------- | ----------- |
| IS_DEFINED(property) | 傳回布林值，表示屬性是否已經指派值 (包含 `null`)。 |

在路由條件中，支援下列比對函式：

| 函式 | 說明 |
| -------- | ----------- |
| ABS(x) | 傳回指定之數值運算式的絕對 (正) 值。 |
| EXP(x) | 傳回指定之數值運算式 (e^x) 的指數值。 |
| POWER(x,y) | 將指定之運算式的值傳回給指定的乘冪 (x^y)。|
| SQUARE(x) | 傳回指定之數值的平方。 |
| CEILING(x) | 傳回大於或等於指定之數值運算式的最小整數值。 |
| FLOOR(x) | 傳回小於或等於指定之數值運算式的最大整數。 |
| SIGN(x) | 傳回指定之數值運算式的正數 (+1)、零 (0) 或負數 (-1) 符號。|
| SQRT(x) | 傳回指定之數值的平方根。 |

在路由條件中，支援下列類型檢查和轉換函式：

| 函式 | 說明 |
| -------- | ----------- |
| AS_NUMBER | 將輸入字串轉換為數字。 如果輸入是一個數字則為 `noop`；如果字串不是數字則為 `Undefined`。|
| IS_ARRAY | 傳回布林值，表示指定之運算式的類型為陣列。 |
| IS_BOOL | 傳回布林值，表示指定之運算式的類型為布林值。 |
| IS_DEFINED | 傳回布林值，表示屬性是否已經指派值。 |
| IS_NULL | 傳回布林值，表示指定之運算式的類型為 null。 |
| IS_NUMBER | 傳回布林值，表示指定之運算式的類型為數字。 |
| IS_OBJECT | 傳回布林值，表示指定之運算式的類型為 JSON 物件。 |
| IS_PRIMITIVE | 傳回布林值，可指出所指定之運算式的類型是否為基本類型 (字串、布林值、數值或 `null`)。 |
| IS_STRING | 傳回布林值，表示指定之運算式的類型為字串。 |

在路由條件中，支援下列字串函式：

| 函式 | 說明 |
| -------- | ----------- |
| CONCAT(x, y, …) | 傳回字串，該字串是串連兩個或多個字串值的結果。 |
| LENGTH(x) | 傳回指定字串運算式的字元數目。|
| LOWER(x) | 傳回將大寫字元資料轉換成小寫之後的字串運算式。 |
| UPPER(x) | 傳回將小寫字元資料轉換成大寫之後的字串運算式。 |
| SUBSTRING(string, start [, length]) | 傳回字串運算式的部分，從指定字元以零為起始的位置開始，直到指定的長度，或直到字串的結尾。 |
| INDEX_OF(string, fragment) | 傳回第一個指定的字串運算式中，第二個字串運算式第一次出現的開始位置，或者如果找不到字串，則為 -1。|
| STARTS_WITH(x, y) | 傳回布林值，表示第一個字串運算式是否以第二個字串運算式開頭。 |
| ENDS_WITH(x, y) | 傳回布林值，表示第一個字串運算式是否以第二個字串運算式結尾。 |
| CONTAINS(x,y) | 傳回布林值，表示第一個字串運算式是否包含第二個字串運算式。 |

## <a name="next-steps"></a>後續步驟
了解如何使用 [Azure IoT SDK][lnk-hub-sdks] 在應用程式中執行查詢。

[lnk-query-where]: iot-hub-devguide-query-language.md#where-clause
[lnk-query-expressions]: iot-hub-devguide-query-language.md#expressions-and-conditions
[lnk-query-getstarted]: iot-hub-devguide-query-language.md#get-started-with-device-twin-queries

[lnk-twins]: iot-hub-devguide-device-twins.md
[lnk-jobs]: iot-hub-devguide-jobs.md
[lnk-devguide-endpoints]: iot-hub-devguide-endpoints.md
[lnk-devguide-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-devguide-mqtt]: iot-hub-mqtt-support.md
[lnk-devguide-messaging-routes]: iot-hub-devguide-messages-d2c.md
[lnk-devguide-messaging-format]: iot-hub-devguide-messages-construct.md

[lnk-hub-sdks]: iot-hub-devguide-sdks.md
