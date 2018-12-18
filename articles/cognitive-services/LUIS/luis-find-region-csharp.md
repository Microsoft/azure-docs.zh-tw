---
title: 在 LUIS 中尋找使用 C# 的端點區域
titleSuffix: Azure Cognitive Services
description: 使用 LUIS 的端點金鑰和應用程式識別碼，以程式設計方式尋找發佈區域。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: 53c3d1abb24ae0d5b33a2a100dda07fd20ae92d1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47039627"
---
# <a name="find-endpoint-region-with-c"></a>尋找使用 C# 的端點區域 
如果您有 LUIS 應用程式識別碼和 LUIS 訂用帳戶識別碼，可以尋找哪個區域要用於端點查詢。

> [!NOTE] 
> 從 [**LUIS-Samples** Github 存放庫](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/find-region/csharp/)可以取得完整 C# 解決方案。

## <a name="luis-endpoint-query-strategy"></a>LUIS 端點查詢策略
每個 LUIS 端點查詢都需要：

* 端點金鑰
* 應用程式識別碼
* 區域

如果 LUIS 端點查詢使用正確的端點金鑰和應用程式識別碼，但使用錯誤的區域，則回應碼為 401。 401 要求不會計入訂用帳戶配額。 將此要求變成策略，以輪詢所有區域，進而尋找正確的區域。 正確的區域是唯一傳回 2xx 狀態碼的要求。 

|Response code|參數|
|--|--|
|2xx|正確的端點金鑰<br>正確的應用程式識別碼<br>正確的主機區域|
|401|正確的端點金鑰<br>正確的應用程式識別碼<br>_正確的_主機區域|

## <a name="c-class-code-to-find-region"></a>用以尋找區域的 C# 程式碼
主控台應用程式會採用 LUIS 應用程式識別碼和端點金鑰，並傳回與其相關聯的所有區域。 目前，端點金鑰是依照區域來建立的，所以只能傳回一個區域。

包括 .Net 程式庫相依性：

[!code-csharp[Add the dependencies](~/samples-luis/documentation-samples/find-region/csharp/ConsoleAppLUISRegion/Program.cs?range=1-6 "Add the dependencies")]

新增這個為了尋找區域所建立的自訂 LUIS 類別。 將 `luisAppId` 和 `luisSubscriptionKey` 的變數值取代為您自己的值。 所有傳回 401 的區域都會寫入至偵錯主控台。 

[!code-csharp[Add the LUIS class](~/samples-luis/documentation-samples/find-region/csharp/ConsoleAppLUISRegion/Program.cs?range=10-83 "Add the LUIS class")]

這是在主控台應用程式的主方法中呼叫自訂 LUIS 類別的範例：

[!code-csharp[Call the LUIS class](~/samples-luis/documentation-samples/find-region/csharp/ConsoleAppLUISRegion/Program.cs?range=85-101 "Call the LUIS class")]

執行應用程式時，主控台會顯示應用程式識別碼的區域。

![顯示 LUIS 區域的主控台應用程式螢幕擷取畫面](./media/find-region-csharp/console.png)

## <a name="next-steps"></a>後續步驟

深入了解 LUIS [區域](luis-reference-regions.md)。
