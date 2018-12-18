---
title: Java 快速入門 - 預測意圖 - LUIS
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，使用可用的公用 LUIS 應用程式，從交談文字判斷使用者的意圖。 使用 Java，以文字形式將使用者的意圖傳送至公用應用程式的 HTTP 預測端點。 在端點，LUIS 會套用公用應用程式的模型來分析自然語言文字的意義、判斷整體意圖，並擷取與應用程式主體網域相關的資料。
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 03fd376e58062ae9b4593fcaf7904200157c4dc3
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031419"
---
# <a name="quickstart-get-intent-using-java"></a>快速入門：使用 Java 來取得意圖

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

<a name="create-luis-subscription-key"></a>

## <a name="prerequisites"></a>必要條件

* [JDK SE](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (英文) (Java 開發套件，標準版)
* [Visual Studio Code](https://code.visualstudio.com/)
* 公用應用程式識別碼：df67dcdb-c37d-46af-88e1-8b97951ca1c2


[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-luis-repo-note.md)]

## <a name="get-luis-key"></a>取得 LUIS 金鑰

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-get-key-para.md)]

## <a name="get-intent-with-browser"></a>使用瀏覽器取得意圖

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-browser-para.md)]

## <a name="get-intent-programmatically"></a>以程式設計方式取得意圖 

您可以使用 Java 來存取您在上一個步驟的瀏覽器視窗中看到的相同結果。 

1. 複製下列程式碼，以在名為 `LuisGetRequest.java` 的檔案中建立類別：

   [!code-java[Console app code that calls a LUIS endpoint](~/samples-luis/documentation-samples/quickstarts/analyze-text/java/call-endpoint.java)]

2. 以您的 LUIS 金鑰取代 `YOUR-KEY` 變數的值。

3. 使用 `javac -cp ":lib/*" LuisGetRequest.java` 來編譯 java 程式。 

4. 使用 `java -cp ":lib/*" LuisGetRequest.java`執行應用程式。 它會顯示您稍早在瀏覽器視窗中看到的相同 JSON。

    ![主控台視窗會顯示來自 LUIS 的 JSON 結果](./media/luis-get-started-java-get-intent/console-turn-on.png)
    
## <a name="luis-keys"></a>LUIS 金鑰

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-key-usage-para.md)]

## <a name="clean-up-resources"></a>清除資源

刪除 Java 檔案。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [新增表達方式](luis-get-started-java-add-utterance.md)