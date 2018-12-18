---
title: 為函式建立 OpenAPI 定義 | Microsoft Docs
description: 建立 OpenAPI 定義，讓其他應用程式和服務可在 Azure 中呼叫您的函式。
services: functions
keywords: OpenAPI, Swagger, 雲端應用程式, 雲端服務,
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.topic: tutorial
ms.date: 12/15/2017
ms.author: glenga
ms.reviewer: sunayv
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: 70eda1d69bdbdc969c5d6bc1774820b50ddc7c83
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093387"
---
# <a name="create-an-openapi-definition-for-a-function"></a>為函式建立 OpenAPI 定義
REST API 通常會使用 OpenAPI 定義 (之前稱為 [Swagger](http://swagger.io/) 檔案) 來描述。 此定義包含有關 API 中可以使用哪些作業，以及應該如何結構化 API 之要求和回應資料的資訊。

在本教學課程中，您會建立一個函式，來判斷風力渦輪機的緊急修復是否符合成本效益。 然後，您會為函式應用程式建立 OpenAPI 定義，以便從其他應用程式和服務呼叫函式。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立函式
> * 使用 OpenAPI 工具來產生 OpenAPI 定義
> * 修改定義以提供其他中繼資料
> * 呼叫函式以測試定義

> [!IMPORTANT]
> 目前只有 1.x 執行階段中有提供 OpenAPI 預覽功能。 如需有關如何建立 1.x 函數應用程式的資訊，請[參閱這裡](./functions-versions.md#creating-1x-apps)。

## <a name="create-a-function-app"></a>建立函數應用程式

您必須擁有函式應用程式以便主控函式的執行。 函式應用程式可讓您將多個函式群組為邏輯單位，以方便您管理、部署、調整和共用資源。 

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]


## <a name="create-the-function"></a>建立函式

本教學課程使用 HTTP 所觸發的函式，該函式接受兩個參數：修復渦輪機的估計時間 (以小時為單位)，以及渦輪機的容量 (以千瓦為單位)。 然後，此函式會計算修復的費用，以及渦輪機在 24 小時內的收入。

1. 展開函式應用程式，然後選取 [函式] 旁的 [+] 按鈕。 如果這是您函式應用程式中的第一個函式，請選取 [自訂函式]。 這會顯示一組完整的函式範本。 

    ![Azure 入口網站中的 Functions 快速入門](media/functions-openapi-definition/add-first-function.png)

2. 在搜尋欄位中，輸入 `http`，然後針對 HTTP 觸發程序範本選擇 **C#**。 
 
    ![選擇 HTTP 觸發程序](./media/functions-openapi-definition/select-http-trigger-portal.png)

3. 輸入 `TurbineRepair` 作為函式 [名稱]，選擇 `Function` 作為 **[[驗證層級]](functions-bindings-http-webhook.md#http-auth)**，然後選取 [建立]。  

    ![建立由 HTTP 觸發的函式](./media/functions-openapi-definition/select-http-trigger-portal-2.png)

1. 使用下列程式碼取代 run.csx 檔案的內容，然後按一下 [儲存]：

    ```csharp
    #r "Newtonsoft.Json"

    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    const double revenuePerkW = 0.12; 
    const double technicianCost = 250; 
    const double turbineCost = 100;

    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {   
        //Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        int hours = data.hours;
        int capacity = data.capacity;

        //Formulas to calculate revenue and cost
        double revenueOpportunity = capacity * revenuePerkW * 24;  
        double costToFix = (hours * technicianCost) +  turbineCost;
        string repairTurbine;

        if (revenueOpportunity > costToFix){
            repairTurbine = "Yes";
        }
        else {
            repairTurbine = "No";
        }

        return (ActionResult) new OkObjectResult(new{
            message = repairTurbine,
            revenueOpportunity = "$"+ revenueOpportunity,
            costToFix = "$"+ costToFix         
        }); 
    }
    ```
    此函式程式碼會傳回 `Yes` 或 `No` 的訊息，指出緊急修復是否符合成本效益，以及渦輪機所代表的收入機會與修復渦輪機的成本。 

1. 若要測試函式，請按一下最右邊的 [測試]，將 [測試] 索引標籤展開。針對 [要求本文] 輸入下列值，然後按一下 [執行]。

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

    ![在 Azure 入口網站中測試函式](media/functions-openapi-definition/test-function.png)

    下列值會在回應本文中傳回。

    ```json
    {"message":"Yes","revenueOpportunity":"$7200","costToFix":"$1600"}
    ```

現在，您有一個函式，可判斷緊急修復是否符合成本效益。 接下來，您會針對函式應用程式產生並修改 OpenAPI 定義。

## <a name="generate-the-openapi-definition"></a>產生 OpenAPI 定義

您現在已經準備好產生 OpenAPI 定義。 此定義可供其他 Microsoft 技術使用 (例如 API Apps、[PowerApps](functions-powerapps-scenario.md) 和 [Microsoft Flow](../azure-functions/app-service-export-api-to-powerapps-and-flow.md))，以及供協力廠商開發人員工具使用 (例如 [Postman](https://www.getpostman.com/docs/importing_swagger) 和[許多其他封裝](http://swagger.io/tools/))。

1. 只選取您的 API 支援的「指令動詞」(在本例中為 POST)。 這會讓產生的 API 定義更清楚。

    1. 在新 HTTP 觸發程序函式的 [整合] 索引標籤中，將 [允許的 HTTP 方法] 變更為 [選取的方法]

    1. 在 [選取的 HTTP 方法] 中，清除 [POST] 以外的所有選項，然後按一下 [儲存]。

        ![選取的 HTTP 方法](media/functions-openapi-definition/selected-http-methods.png)
        
1. 按一下函式應用程式名稱 (例如 **function-demo-energy**) > [平台功能] > [API 定義]。

    ![API 定義](media/functions-openapi-definition/api-definition.png)

1. 在 [API 定義] 索引標籤上，按一下 [函式]。

    ![API 定義來源](media/functions-openapi-definition/api-definition-source.png)

    此步驟會為您的函式應用程式啟用一組 OpenAPI 選項，包含裝載來自您函式應用程式網域之 OpenAPI 檔案的端點、[OpenAPI 編輯器](http://editor.swagger.io)的內嵌複本，以及 API 定義範本產生器。

1. 按一下 [產生 API 定義範本] > [儲存]。

    ![產生 API 定義範本](media/functions-openapi-definition/generate-template.png)

    Azure 會掃描您的函式應用程式以尋找 HTTP 觸發程序函式，並使用 functions.json 中的資訊來產生 OpenAPI 定義。 以下是產生的定義：

    ```yaml
    swagger: '2.0'
    info:
    title: function-demo-energy.azurewebsites.net
    version: 1.0.0
    host: function-demo-energy.azurewebsites.net
    basePath: /
    schemes:
    - https
    - http
    paths:
    /api/TurbineRepair:
        post:
        operationId: /api/TurbineRepair/post
        produces: []
        consumes: []
        parameters: []
        description: >-
            Replace with Operation Object
            #http://swagger.io/specification/#operationObject
        responses:
            '200':
            description: Success operation
        security:
            - apikeyQuery: []
    definitions: {}
    securityDefinitions:
    apikeyQuery:
        type: apiKey
        name: code
        in: query
    ```

    此定義會描述為「範本」，因為它需要更多中繼資料才能成為完整的 OpenAPI 定義。 您將修改下一個步驟中的定義。

## <a name="modify-the-openapi-definition"></a>修改 OpenAPI 定義
現在您已擁有範本定義，您可以對其進行修改，以提供 API 作業和資料結構的其他相關中繼資料。 在 [API 定義] 中，刪除產生的定義，從 `post` 到定義的底部，在下列內容中貼上，然後按一下 [儲存]。

```yaml
    post:
      operationId: CalculateCosts
      description: Determines if a technician should be sent for repair
      summary: Calculates costs
      x-ms-summary: Calculates costs
      x-ms-visibility: important
      produces:
        - application/json
      consumes:
        - application/json
      parameters:
        - name: body
          in: body
          description: Hours and capacity used to calculate costs
          x-ms-summary: Hours and capacity
          x-ms-visibility: important
          required: true
          schema:
            type: object
            properties:
              hours:
                description: The amount of effort in hours required to conduct repair
                type: number
                x-ms-summary: Hours
                x-ms-visibility: important
              capacity:
                description: The max output of a turbine in kilowatts
                type: number
                x-ms-summary: Capacity
                x-ms-visibility: important
      responses:
        200:
          description: Message with cost and revenue numbers
          x-ms-summary: Message
          schema:
           type: object
           properties:
            message:
              type: string
              description: Returns Yes or No depending on calculations
              x-ms-summary: Message 
            revenueOpportunity:
              type: string
              description: The revenue opportunity cost
              x-ms-summary: RevenueOpportunity 
            costToFix:
              type: string
              description: The cost in $ to fix the turbine
              x-ms-summary: CostToFix
      security:
        - apikeyQuery: []
definitions: {}
securityDefinitions:
  apikeyQuery:
    type: apiKey
    name: code
    in: query
```

在此情況下，您可以只貼上更新的中繼資料，不過，請務必了解我們對預設範本所做的修改類型：

+ 指定 API 會產生並取用 JSON 格式的資料。

+ 指定必要的參數，以及其名稱和資料類型。

+ 指定成功回應的傳回值，以及其名稱和資料類型。

+ 針對 API 以及其作業和參數提供易記摘要和描述。 這對使用此函式的人員很重要。

+ 已新增 x-ms-summary 和 x-ms-visibility，可用於 Microsoft Flow 和 Logic Apps 的 UI 中。 如需詳細資訊，請參閱 [Microsoft Flow 中自訂 API 的 OpenAPI 延伸模組](https://preview.flow.microsoft.com/documentation/customapi-how-to-swagger/)。

> [!NOTE]
> 我們將安全性定義保留為 API 金鑰的預設驗證方法。 如果您使用不同類型的驗證，則會變更定義的這個區段。

如需定義 API 作業的詳細資訊，請參閱 [OpenAPI 規格](https://swagger.io/specification/#operationObject)。

## <a name="test-the-openapi-definition"></a>測試 OpenAPI 定義
使用 API 定義之前，建議先在 Azure Functions UI 中進行測試。

1. 在您函式的 [管理] 索引標籤中，複製 [主機金鑰] 下的**預設**金鑰。

    ![複製 API 金鑰](media/functions-openapi-definition/copy-api-key.png)

    > [!NOTE]
    >您可以使用此金鑰進行測試，也可以在從應用程式或服務呼叫 API 時使用。

1. 回到 API 定義：**function-demo-energy** > [平台功能] > [API 定義]。

1. 在右窗格中按一下 [驗證]，輸入您所複製的 API 金鑰，然後按一下 [驗證]。

    ![使用 API 金鑰進行驗證](media/functions-openapi-definition/authenticate-api-key.png)

1. 向下捲動並按一下 [Try this operation] \(嘗試此作業)。

    ![嘗試此作業](media/functions-openapi-definition/try-operation.png)

1. 輸入 [時數] 和 [容量] 的值。

    ![輸入參數](media/functions-openapi-definition/parameters.png)

    注意 UI 如何使用 API 定義中的描述。

1. 按一下 [傳送要求]，然後按一下 [Pretty] 索引標籤以查看輸出。

    ![傳送要求](media/functions-openapi-definition/send-request.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立函式
> * 使用 OpenAPI 工具來產生 OpenAPI 定義
> * 修改定義以提供其他中繼資料
> * 呼叫函式以測試定義

前進到下一個主題，了解如何建立使用您所建立之 OpenAPI 定義的 PowerApps 應用程式。
> [!div class="nextstepaction"]
> [從 PowerApps 呼叫函式](functions-powerapps-scenario.md)
