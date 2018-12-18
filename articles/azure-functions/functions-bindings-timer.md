---
title: Azure Functions 的計時器觸發程序
description: 了解如何在 Azure Functions 中使用計時器觸發程序。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
editor: ''
tags: ''
keywords: azure functions, 函數, 事件處理, 動態運算, 無伺服器架構
ms.assetid: d2f013d1-f458-42ae-baf8-1810138118ac
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/08/2018
ms.author: glenga
ms.custom: ''
ms.openlocfilehash: 270228e73243e6b2670e7ccb30765526a5db6463
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42144032"
---
# <a name="timer-trigger-for-azure-functions"></a>Azure Functions 的計時器觸發程序 

本文說明如何在 Azure Functions 中使用計時器觸發程序。 計時器觸發程序可讓您依照排程執行函式。 

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>套件 - Functions 1.x

[Microsoft.Azure.WebJobs.Extensions](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) NuGet 套件 2.x 版中提供計時器觸發程序。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Timers/) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-2x"></a>套件 - Functions 2.x

[Microsoft.Azure.WebJobs.Extensions](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) NuGet 套件 3.x 版中提供計時器觸發程序。 套件的原始程式碼位於 [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/) GitHub 存放庫中。

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="example"></a>範例

請參閱特定語言的範例：

* [C#](#trigger---c-example)
* [C# 指令碼 (.csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [JavaScript](#trigger---javascript-example)
* [Java](#trigger---java-example)

### <a name="c-example"></a>C# 範例

下列範例示範每五分鐘執行一次的 [C# 函式](functions-dotnet-class-library.md)：

```cs
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, TraceWriter log)
{
    if(myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

### <a name="c-script-example"></a>C# 指令碼範例

下列範例示範 function.json 檔案中的計時器觸發程序繫結，以及使用此繫結的 [C# 指令碼函式](functions-reference-csharp.md)。 此函式會寫入一項記錄，指出此函式引動過程是否由遺失的排程項目所造成。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

以下是 C# 指令碼程式碼：

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log)
{
    if(myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}" );  
}
```

### <a name="f-example"></a>F# 範例

下列範例示範 function.json 檔案中的計時器觸發程序繫結，以及使用此繫結的 [F# 指令碼函式](functions-reference-fsharp.md)。 此函式會寫入一項記錄，指出此函式引動過程是否由遺失的排程項目所造成。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

以下是 F# 指令碼程式碼：

```fsharp
let Run(myTimer: TimerInfo, log: TraceWriter ) =
    if (myTimer.IsPastDue) then
        log.Info("F# function is running late.")
    let now = DateTime.Now.ToLongTimeString()
    log.Info(sprintf "F# function executed at %s!" now)
```

### <a name="javascript-example"></a>JavaScript 範例

下列範例示範 function.json 檔案中的計時器觸發程序繫結，以及使用此繫結的 [JavaScript 函式](functions-reference-node.md)。 此函式會寫入一項記錄，指出此函式引動過程是否由遺失的排程項目所造成。

以下是 *function.json* 檔案中的繫結資料：

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

以下是 JavaScript 指令碼程式碼：

```JavaScript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if(myTimer.isPastDue)
    {
        context.log('Node.js is running late!');
    }
    context.log('Node.js timer trigger function ran!', timeStamp);   

    context.done();
};
```

### <a name="java-example"></a>Java 範例

下列範例函式會每五分鐘觸發並執行。 函式上的 `@TimerTrigger` 註釋會使用與 [CRON 運算式](http://en.wikipedia.org/wiki/Cron#CRON_expression)相同的字串格式來定義排程。

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 *&#47;5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

## <a name="attributes"></a>屬性

在 [C# 類別庫](functions-dotnet-class-library.md)中，使用 [TimerTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs)。

該屬性的建構函式會採用 CRON 運算式或是 `TimeSpan`。 僅當函式應用程式是在 App Service 方案中執行時，您才可以使用 `TimeSpan`。 下列範例顯示 CRON 運算式：

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, TraceWriter log)
{
    if (myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
}
 ```

## <a name="configuration"></a>組態

下表說明您在 *function.json* 檔案中設定的繫結設定屬性內容和 `TimerTrigger` 屬性。

|function.json 屬性 | 屬性內容 |說明|
|---------|---------|----------------------|
|**type** | n/a | 必須設定為 "timerTrigger"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | n/a | 必須設定為 "in"。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | n/a | 代表函式程式碼中計時器物件的變數名稱。 | 
|**schedule**|**ScheduleExpression**|[CRON 運算式](#cron-expressions)或 [TimeSpan](#timespan) 值。 `TimeSpan` 只能用於 App Service 方案上執行的函式應用程式。 您可以將排程運算式放在應用程式設定中，並將此屬性設定為以 **%** 符號包裝的應用程式設定名稱，如此範例所示："%ScheduleAppSetting%"。 |
|**runOnStartup**|**RunOnStartup**|如果為 `true`，當執行階段啟動時，會叫用函式。 例如，當函式應用程式因無活動而處於閒置狀態後再甦醒時、 當函式應用程式因函式變更而重新啟動時，以及當函式應用程式相應放大時，執行階段便會啟動。因此 **runOnStartup** 應該幾乎不會設定為 `true`，因為它會使程式碼在極度無法預期的時間執行。|
|**useMonitor**|**UseMonitor**|設定為 `true` 或 `false` 以表示是否應該監視排程。 排程監視會使排程持續進行，以協助確保即使在函式應用程式執行個體重新啟動時，排程也能正確地持續運作。 如果未明確設定，則循環間隔大於 1 分鐘的排程之預設值為 `true`。 若為每分鐘觸發超過一次的排程，預設值為 `false`。

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>使用量

叫用計時器觸發程序函式時，[計時器物件](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs)會傳遞至函式。 下列 JSON 是計時器物件的範例表示法。 

```json
{
    "Schedule":{
    },
    "ScheduleStatus": {
        "Last":"2016-10-04T10:15:00+00:00",
        "LastUpdated":"2016-10-04T10:16:00+00:00",
        "Next":"2016-10-04T10:20:00+00:00"
    },
    "IsPastDue":false
}
```

當目前函式引動過程晚於排程時，`IsPastDue` 屬性為 `true`。 例如，函式應用程式重新啟動可能會導致遺漏引動過程。

## <a name="cron-expressions"></a>CRON 運算式 

Azure Functions 會使用 [NCronTab](https://github.com/atifaziz/NCrontab) \(英文\) 程式庫來解譯 CRON 運算式。 CRON 運算式包含六個欄位：

`{second} {minute} {hour} {day} {month} {day-of-week}`

每個欄位可以具備下列類型的值：

|類型  |範例  |觸發時間  |
|---------|---------|---------|
|特定值 |<nobr>"0 5 * * * *"</nobr>|於 hh:05:00，其中 hh 是每小時 (一小時一次)|
|所有值 (`*`)|<nobr>"0 * 5 * * *"</nobr>|於每天 5:mm:00，其中 mm 是小時中的每一分鐘 (一天 60 次)|
|範圍 (`-` 運算子)|<nobr>"5-7 * * * * *"</nobr>|於 hh:mm:05、hh:mm:06 和 hh:mm:07，其中 hh: mm 是每小時的每一分鐘 (一分鐘 3 次)|  
|一組值 (`,` 運算子)|<nobr>"5,8,10 * * * * *"</nobr>|於 hh:mm:05、hh:mm:08 和 hh:mm:10，其中 hh: mm 是每小時的每一分鐘 (一分鐘 3 次)|
|間隔值 (`/` 運算子)|<nobr>"0 */5 * * * *"</nobr>|於 hh:05:00、hh:10:00、hh:15:00 以此類推，直到 hh:55:00，其中 hh 是每小時 (一小時 12 次)|

若要指定月或日，您可以使用數值、名稱，或是名稱的縮寫：

* 針對日，數值必須為 0 到 6，其中 0 為星期日。
* 名稱必須為英文。 例如：`Monday`, `January`。
* 名稱不區分大小寫。
* 名稱可為縮寫。 建議的縮寫長度為三個字母。  例如：`Mon`, `Jan`。 

### <a name="cron-examples"></a>CRON 範例

以下是您可以在 Azure Functions 中使用於計時器觸發程序的一些 CRON 運算式範例。

|範例|觸發時間  |
|---------|---------|
|`"0 */5 * * * *"`|每隔 5 分鐘一次|
|`"0 0 * * * *"`|每小時開始時一次|
|`"0 0 */2 * * *"`|每隔 2 小時一次|
|`"0 0 9-17 * * *"`|上午 9 點到下午 5 點之間每隔一小時一次|
|`"0 30 9 * * *"`|每天上午 9:30|
|`"0 30 9 * * 1-5"`|每個工作日上午 9:30|
|`"0 30 9 * Jan Mon"`|一月每個星期一上午 9:30|
>[!NOTE]   
>您可以在線上找到 CRON 運算式範例，但其中大部分都會省略 `{second}` 欄位。 如果您複製其中一個運算式，請新增遺漏的 `{second}` 欄位。 通常您在該欄位中需要零，而非星號。

### <a name="cron-time-zones"></a>CRON 時區

CRON 運算式中的數字代表時間和日期，而非時間範圍。 例如，`hour` 欄位中的 5 代表上午 5:00，而非每隔 5 小時。

CRON 運算式使用的預設時區是國際標準時間 (UTC)。 若要讓 CRON 運算式以另一個時區為基礎，請為名為 `WEBSITE_TIME_ZONE` 的函式應用程式建立應用程式設定。 將值設定為所需的時區名稱，如 [Microsoft 時區索引](https://technet.microsoft.com/library/cc749073)中所示。 

例如，*美加東部標準時間*是 UTC-05:00。 若要讓計時器觸發程序在每天上午 10:00 (美加東部標準時間) 觸發，您可以使用說明 UTC 時區的下列 CRON 運算式︰

```json
"schedule": "0 0 15 * * *"
``` 

或者為名為 `WEBSITE_TIME_ZONE` 的函式應用程式建立應用程式設定，並將值設為**美加東部標準時間**。  然後使用下列 CRON 運算式： 

```json
"schedule": "0 0 10 * * *"
``` 

當您使用 `WEBSITE_TIME_ZONE` 時，時間會隨特定時區的時間變更 (例如日光節約時間) 而調整。 

## <a name="timespan"></a>時間範圍

 `TimeSpan` 只能用於 App Service 方案上執行的函式應用程式。

不同於 CRON 運算式，`TimeSpan` 值會指定每個函式引動過程之間的時間間隔。 如果函式在執行時間超過指定時間間隔之後完成，計時器會立即再次叫用函式。

以字串表示，當 `hh` 低於 24 時，`TimeSpan` 格式為 `hh:mm:ss`。 當前兩個數字為 24 或更高時，格式為 `dd:hh:mm`。 這裡有一些範例：

|範例 |觸發時間  |
|---------|---------|
|"01:00:00" | 每小時        |
|"00:01:00"|每分鐘         |
|"24:00:00" | 每 24 天        |

## <a name="scale-out"></a>向外延展

如果函式應用程式相應放大至多個執行個體，則只有計時器觸發函式的單一執行個體會在所有執行個體中執行。

## <a name="function-apps-sharing-storage"></a>共用儲存體的函式應用程式

如果您在多個函式應用程式中共用儲存體帳戶，請確定每個函式應用程式在 host.json 中具有不同的 `id`。 您可以省略 `id` 屬性或將每個函式應用程式的 `id` 手動設定為不同的值。 計時器觸發程序會使用儲存體鎖定，以確保當函數應用程式相應放大至多個執行個體時，只會有一個計時器執行個體。 如果兩個函式應用程式共用相同的 `id`，且每一個都是使用計時器觸發程序，則只有一個計時器會執行。

## <a name="retry-behavior"></a>重試行為

不同於佇列觸發程序，計時器觸發程序在函式失敗後並不會重試。 當函式失敗時，必須等到下次排程的時間才會再次呼叫函式。

## <a name="troubleshooting"></a>疑難排解

如需有關當計時器觸發程序未如預期般運作時該怎麼做的資訊，請參閱[調查及報告計時器觸發的函式並未引發的問題](https://github.com/Azure/azure-functions-host/wiki/Investigating-and-reporting-issues-with-timer-triggered-functions-not-firing)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [移至使用計時器觸發程序的快速入門](functions-create-scheduled-function.md)

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
