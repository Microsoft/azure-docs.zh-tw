---
title: 如何停用 Azure Functions 中的函式
description: 了解如何停用和啟用 Azure Functions 1.x 和 2.x 中的函式。
services: functions
documentationcenter: ''
author: tdykstra
manager: cfowler
editor: ''
ms.service: functions
ms.workload: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: tdykstra
ms.openlocfilehash: be30a97c9c94031ea526385326c220fbf0d316b7
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40007100"
---
# <a name="how-to-disable-functions-in-azure-functions"></a>如何停用 Azure Functions 中的函式

本文說明如何停用 Azure Functions 中的函式。 「停用」函式表示讓執行階段忽略為此函式定義的自動觸發程序。 您這麼做的方式取決於執行階段版本和程式設計語言：

* Functions 1.x
  * 指令碼語言
  * C# 類別庫
* Functions 2.x
  * 一種方法適用於所有語言
  * C# 類別庫的選擇性方法

## <a name="functions-1x---scripting-languages"></a>Functions 1.x - 指令碼語言

對於 C# 指令碼和 JavaScript 之類的指令碼語言，您可使用 *function.json* 檔案的 `disabled` 屬性來告訴執行階段不要觸發函式。 這個屬性可以設定為 `true` 或應用程式設定的名稱：

```json
{
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ],
    "disabled": true
}
```
或 

```json
    "bindings": [
        ...
    ],
    "disabled": "IS_DISABLED"
```

在第二個範例中，如有名為 IS_DISABLED 且設為 `true` 或 1 的應用程式設定，此函式就會停用。

您可以在 Azure 入口網站中編輯此檔案，或使用函式的 [管理] 索引標籤上的 [函式狀態] 切換。此入口網站切換可藉由變更 *function.json* 檔案來產生作用。

![函式狀態切換](media/disable-function/function-state-switch.png)

## <a name="functions-1x---c-class-libraries"></a>Functions 1.x - C# 類別庫

在 Functions 1.x 類別庫中，您可使用 `Disable` 屬性來防止觸發函式。 您可以使用此屬性 (沒有建構函式參數)，如下列範例所示：

```csharp
public static class QueueFunctions
{
    [Disable]
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
}
```

沒有建構函式參數的屬性會要求您重新編譯並重新部署專案，以變更函式的已停用狀態。 若要更彈性地使用此屬性，請包含參照布林值應用程式設定的建構函式參數，如下列範例所示：

```csharp
public static class QueueFunctions
{
    [Disable("MY_TIMER_DISABLED")]
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
}
```

這個方法可讓您藉由變更應用程式設定來啟用及停用此函式，而不需重新編譯或重新部署。 變更應用程式設定會導致函式應用程式重新啟動，因此會立即辨識已停用的狀態變更。

> [!IMPORTANT]
> `Disabled` 屬性是停用類別庫函式的唯一方式。 您不應該直接編輯為類別庫函式所產生的 *function.json* 檔案。 如果您編輯該檔案，無論您對 `disabled` 屬性做什麼都不會有任何作用。
>
> [管理] 索引標籤上的 [函式狀態] 切換也是如此，因為它可藉由變更 *function.json* 檔案來產生作用。
>
> 另請注意，入口網站可能會在此函式並未停用時，指出它已停用。



## <a name="functions-2x---all-languages"></a>Functions 2.x - 所有語言

在 Functions 2.x 中，您可使用應用程式設定來停用函式。 例如，若要停用名為 `QueueTrigger` 的函式，您可建立名為 `AzureWebJobs.QueueTrigger.Disabled` 的應用程式設定，並將它設定為 `true`。 若要啟用此函式，請將應用程式設定設為 `false`。 您也可以使用函式的 [管理] 索引標籤上的 [函式狀態] 切換。此切換可藉由建立和刪除 `AzureWebJobs.<functionname>.Disabled` 應用程式設定來產生作用。

![函式狀態切換](media/disable-function/function-state-switch.png)

## <a name="functions-2x---c-class-libraries"></a>Functions 2.x - C# 類別庫

在 Functions 2.x 類別庫中，我們建議您使用所有語言都適用的方法。 但如有需要，您可以[使用如 Functions 1.x 中的 Disable 屬性](#functions-1x---c-class-libraries)。

## <a name="next-steps"></a>後續步驟

本文關於如何停用自動觸發程序。 如需觸發程序的詳細資訊，請參閱[觸發程序與繫結](functions-triggers-bindings.md)。
