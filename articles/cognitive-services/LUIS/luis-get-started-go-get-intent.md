---
title: Go 語言快速入門 - 預測意圖 - LUIS
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，使用可用的公用 LUIS 應用程式，從交談文字判斷使用者的意圖。 使用 GO，以文字形式將使用者的意圖傳送至公用應用程式的 HTTP 預測端點。 在端點，LUIS 會套用公用應用程式的模型來分析自然語言文字的意義、判斷整體意圖，並擷取與應用程式主體網域相關的資料。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: f19c46fcc7dcff2e02786bb3a03082f831e880a2
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032728"
---
# <a name="quickstart-get-intent-using-go"></a>快速入門：使用 Go 來取得意圖

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

* [Go](https://golang.org/) (英文) 程式設計語言  
* [Visual Studio Code](https://code.visualstudio.com/)
* 公用應用程式識別碼：df67dcdb-c37d-46af-88e1-8b97951ca1c2

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-luis-repo-note.md)]

## <a name="get-luis-key"></a>取得 LUIS 金鑰

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-get-key-para.md)]

## <a name="get-intent-with-browser"></a>使用瀏覽器取得意圖

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-browser-para.md)]

## <a name="get-intent-programmatically"></a>以程式設計方式取得意圖

您可以使用 Go，來存取您在上一個步驟的瀏覽器視窗中所看到的相同結果。 

1. 建立名為 `endpoint.go` 的新檔案。 新增下列程式碼：
    
   [!code-go[Go code that calls a LUIS endpoint](~/samples-luis/documentation-samples/quickstarts/analyze-text/go/endpoint.go?range=36-98)]

2. 在您建立檔案所在的相同目錄中使用命令提示字元，輸入 `go build endpoint.go` 以編譯 Go 檔案。 命令提示字元不會傳回成功組建的任何資訊。

3. 藉由在命令提示字元中輸入下列文字，從命令列執行 Go 應用程式： 

    ```CMD
    go run endpoint.go -appID df67dcdb-c37d-46af-88e1-8b97951ca1c2 -endpointKey <add-your-key> -region westus
    ```
    
    使用您的金鑰值取代 `<add-your-key>`。  
    
    命令提示字元回應為： 
    
    ```CMD
    appID has value df67dcdb-c37d-46af-88e1-8b97951ca1c2
    endpointKey has value xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    region has value westus
    utterance has value turn on the bedroom light
    response
    {
        "query": "turn on the bedroom light",
        "topScoringIntent": {
            "intent": "HomeAutomation.TurnOn",
            "score": 0.809439957
        },
        "entities": [
            {
            "entity": "bedroom",
            "type": "HomeAutomation.Room",
            "startIndex": 12,
            "endIndex": 18,
            "score": 0.8065475
            }
        ]
    }
    ```
    
## <a name="luis-keys"></a>LUIS 金鑰

[!INCLUDE [Use authoring key for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-key-usage-para.md)]

## <a name="clean-up-resources"></a>清除資源
關閉 Go 檔案，並且從檔案系統中移除。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [新增表達方式](luis-get-started-go-add-utterance.md)