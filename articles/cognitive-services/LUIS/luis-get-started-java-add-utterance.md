---
title: Java 快速入門 - 變更模型及訓練 LUIS 應用程式
titleSuffix: Azure Cognitive Services
description: 在此 Java 快速入門中，會將範例語句新增至「家庭自動化」應用程式，並且將應用程式定型。 範例語句是對應到意圖的交談使用者文字。 您可以藉由提供與意圖相關的範例語句，教導 LUIS 哪些種類的使用者提供文字屬於哪種意圖。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 8500dfdd639b4f1de46a0b79b87a8564d338b6ab
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042019"
---
# <a name="quickstart-change-model-using-java"></a>快速入門：使用 Java 來變更模型 

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

[!INCLUDE [Quickstart prerequisites for endpoint](../../../includes/cognitive-services-luis-qs-change-model-prereq.md)]
* [JDK SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (英文) (Java 開發套件，標準版)
* [Google 的 GSON JSON 文件庫](https://github.com/google/gson)。

[!INCLUDE [Quickstart note about code repository](../../../includes/cognitive-services-luis-qs-change-model-luis-repo-note.md)]

## <a name="example-utterances-json-file"></a>範例語句 JSON 檔案

[!INCLUDE [Quickstart explanation of example utterance JSON file](../../../includes/cognitive-services-luis-qs-change-model-json-ex-utt.md)]

## <a name="create-quickstart-code"></a>建立快速入門程式碼

1. 將 Java 相依性新增至名為 `AddUtterances.java` 的檔案。

   [!code-java[Java Dependencies](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=23-26 "Java Dependencies")]

2. 建立 `AddUtterances` 類別。 此類別包含之後的所有程式碼片段。

    ```Java
    public class AddUtterances {
        // Insert code here
    }
    ```

3. 將 LUIS 常數新增至此類別。 複製下列程式碼，並且變更為您的撰寫金鑰、應用程式識別碼和版本識別碼。

   [!code-java[LUIS-based IDs](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=33-44 "LUIS-based IDs")]

4. 將用來呼叫 LUIS API 的方法，新增至 AddUtterances 類別。 

   [!code-java[HTTP request to LUIS](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=46-168 "HTTP request to LUIS")]

5. 將用來從 LUIS API 取得 HTTP 回應的方法，新增至 AddUtterances 類別。

   [!code-java[HTTP response from LUIS](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=170-202 "HTTP response from LUIS")]

6. 將例外狀況處理新增至 AddUtterances 類別。 

   [!code-java[Exception Handling](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=205-243 "Exception Handling")]

7. 將 main 函式新增至 AddUtterances 類別。

   [!code-java[Add main function](~/samples-luis/documentation-samples/quickstarts/change-model/java/AddUtterances.java?range=245-278 "Add main function")]

## <a name="build-code"></a>建置程式碼

使用相依性編譯 AddUtterance

```CMD
> javac -classpath gson-2.8.2.jar AddUtterances.java
```

## <a name="run-code"></a>執行程式碼
不使用任何引數來呼叫 `AddUtterance`，會將 LUIS 語句新增至此應用程式，但不會加以訓練。

```CMD
> java -classpath .;gson-2.8.2.jar AddUtterances
```

此命令列會顯示呼叫新增語句 API 的結果。 

[!INCLUDE [Quickstart response from API calls](../../../includes/cognitive-services-luis-qs-change-model-json-results.md)]

## <a name="clean-up-resources"></a>清除資源
當您完成快速入門時，請移除在本快速入門中建立的所有檔案。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [以程式設計方式建置 LUIS 應用程式](luis-tutorial-node-import-utterances-csv.md) 