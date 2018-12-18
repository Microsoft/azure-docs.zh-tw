---
title: 建立與 Azure Logic Apps 整合的函式 | Microsoft Docs
description: 建立可整合 Azure Logic Apps 和 Azure 認知服務的函數，以將推文情感進行分類，並在偵測到不佳的情感時傳送通知。
services: functions, logic-apps, cognitive-services
keywords: 工作流程, 雲端應用程式, 雲端服務, 商務程序, 系統整合, 企業應用程式整合, EAI
author: ggailey777
manager: jeconnoc
ms.assetid: 60495cc5-1638-4bf0-8174-52786d227734
ms.service: azure-functions
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: glenga
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: 0b2e0ff800ab80a2c638293ce23fc1911390f2dd
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221107"
---
# <a name="create-a-function-that-integrates-with-azure-logic-apps"></a>建立與 Azure Logic Apps 整合的函式

Azure Functions 與 Logic Apps 設計工具中的 Azure Logic Apps 進行整合。 這個整合可讓您搭配其他 Azure 和第三方服務，使用協調流程中的 Functions 計算能力。 

此教學課程說明如何使用 Functions 搭配 Logic Apps 和 Azure 上的 Microsoft 認知服務，分析來自 Twitter 貼文的情感。 HTTP 觸發函式會以情感分數作為基礎，將推文分類為綠色、黃色或紅色。 偵測到不佳的情感時，會傳送一封電子郵件。 

![此映像顯示邏輯應用程式設計工具中應用程式的前兩個步驟](media/functions-twitter-email/designer1.png)

在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立認知服務 API 資源。
> * 建立可將推文情感進行分類的函式。
> * 建立連線至 Twitter 的邏輯應用程式。
> * 將情感偵測新增至邏輯應用程式。 
> * 將邏輯應用程式連線至函式。
> * 以函式的回應作為基礎來傳送電子郵件。

## <a name="prerequisites"></a>先決條件

+ 使用中的 [Twitter](https://twitter.com/) 帳戶。 
+ [Outlook.com](https://outlook.com/) 帳戶 (用於傳送通知)。
+ 此主題使用[從 Azure 入口網站建立您的第一個函式](functions-create-first-azure-function.md)中所建立的資源作為起點。  
如果您尚未這麼做，請立即完成這些步驟，才能建立函式應用程式。

## <a name="create-a-cognitive-services-resource"></a>建立認知服務資源

可在 Azure 中使用認知服務 API 作為個別資源。 使用文字分析 API 來偵測受監視的人氣推文。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

2. 按一下 Azure 入口網站左上角的 [建立資源]。

3. 按一下 [AI + Analytics] > [文字分析 API]。 然後，使用資料表中指定的設定、接受條款，並勾選 [釘選到儀表板]。

    ![建立 [識別資源] 頁面](media/functions-twitter-email/cog_svcs_resource.png)

    | 設定      |  建議的值   | 說明                                        |
    | --- | --- | --- |
    | **名稱** | MyCognitiveServicesAccnt | 請選擇唯一的帳戶名稱。 |
    | **位置** | 美國西部 | 使用距離您最近的位置。 |
    | **定價層** | F0 | 從最低層開始。 如果您用完呼叫，請調整為較高層。|
    | **資源群組** | myResourceGroup | 此教學課程中所有的服務，都是使用相同的資源群組。|

4. 按一下 [建立] 以建立資源。 建立之後，請選取釘選到儀表板的新認知服務資源。 

5. 在左側瀏覽欄位中，按一下 [金鑰]，然後將**金鑰 1** 的值複製並加以儲存。 您可以使用此金鑰，將邏輯應用程式連線至認知服務 API。 
 
    ![金鑰](media/functions-twitter-email/keys.png)

## <a name="create-the-function-app"></a>建立函式應用程式

Functions 提供的絕佳方法，可讓您將 Logic Apps 工作流程中的處理工作進行卸載。 此教學課程會使用 HTTP 觸發函式來處理認知服務的推文情感分數，並將類別值傳回。  

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-an-http-triggered-function"></a>建立由 HTTP 觸發的函式  

1. 展開函式應用程式，然後按一下 [Functions] 旁的 [+] 按鈕。 如果這是您函式應用程式中的第一個函式，請選取 [自訂函式]。 這會顯示一組完整的函式範本。

    ![Azure 入口網站中的 Functions 快速入門](media/functions-twitter-email/add-first-function.png)

2. 在搜尋欄位中，輸入 `http`，然後針對 HTTP 觸發程序範本選擇 **C#**。 

    ![選擇 HTTP 觸發程序](./media/functions-twitter-email/select-http-trigger-portal.png)

    新增到函數應用程式的所有後續函式都使用 C# 語言範本。

3. 輸入您函式的 [名稱] ，針對 [[驗證等級](functions-bindings-http-webhook.md#http-auth)] 選擇 `Function`，然後選取 [建立]。 

    ![建立由 HTTP 觸發的函式](./media/functions-twitter-email/select-http-trigger-portal-2.png)

    這會使用 HTTP 觸發程序範本建立 C# 指令碼函式。 您的程式碼會在新視窗中顯示為 `run.csx`。

4. 使用下列程式碼取代 `run.csx` 檔案的內容，然後按一下 [儲存]：

    ```csharp
    #r "Newtonsoft.Json"
    
    using System;
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        string category = "GREEN";
    
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        log.LogInformation(string.Format("The sentiment score received is '{0}'.", requestBody));
    
        double score = Convert.ToDouble(requestBody);
    
        if(score < .3)
        {
            category = "RED";
        }
        else if (score < .6) 
        {
            category = "YELLOW";
        }
    
        return requestBody != null
            ? (ActionResult)new OkObjectResult(category)
            : new BadRequestObjectResult("Please pass a value on the query string or in the request body");
    }
    ```
    此函式程式碼會以要求中所收到的情感分數作為基礎，將色彩類別傳回。 

4. 若要測試函式，請按一下最右邊的 [測試]，將 [測試] 索引標籤展開。輸入 `0.2` 的值作為**要求本文**，然後按一下 [執行]。 會在回應本文中傳回**紅色**的值。 

    ![在 Azure 入口網站中測試函式](./media/functions-twitter-email/test.png)

現在您的函式可將情感分數進行分類。 接下來，您可以建立邏輯應用程式，將您的函式與 Twitter 和認知服務 API 進行。 

## <a name="create-a-logic-app"></a>建立邏輯應用程式   

1. 在 Azure 入口網站中，按一下 Azure 入口網站左上角的 [新增] 按鈕。

2. 按一下 [企業整合] > [邏輯應用程式]。 然後，使用資料表中指定的設定、勾選 [釘選到儀表板]，然後按一下 [建立]。
 
4. 然後，輸入**名稱** (例如 `TweetSentiment`)、使用資料表中指定的設定、接受條款，並勾選 [釘選到儀表板]。

    ![在 Azure 入口網站中建立邏輯應用程式](./media/functions-twitter-email/new_logic_app.png)

    | 設定      |  建議的值   | 說明                                        |
    | ----------------- | ------------ | ------------- |
    | **名稱** | TweetSentiment | 為您的應用程式選擇適當名稱。 |
    | **資源群組** | myResourceGroup | 選擇與之前相同的現有資源群組。 |
    | **位置** | 美國東部 | 選擇接近您的位置。 |    

4. 選擇 [釘選到儀表板]，然後按一下 [建立] 以建立邏輯應用程式。 

5. 建立應用程式之後，按一下釘選到儀表板的新邏輯應用程式。 接著，在 Logic Apps 設計工具中，向下捲動並按一下 [空白的邏輯應用程式] 範本。 

    ![空白的 Logic Apps 範本](media/functions-twitter-email/blank.png)

您現在可以使用 Logic Apps 設計工具，將服務和觸發程序新增至您的應用程式。

## <a name="connect-to-twitter"></a>連接到 Twitter

首先，建立您 Twitter 帳戶的連線。 邏輯應用程式會輪詢推文，從而觸發應用程式加以執行。

1. 在設計工具中，按一下 [Twitter] 服務，然後按一下 [張貼新推文時] 觸發程序。 登入您的 Twitter 帳戶，並授權 Logic Apps 以使用您的帳戶。

2. 使用表格中指定的 Twitter 觸發程序設定。 

    ![Twitter 連接器設定](media/functions-twitter-email/azure_tweet.png)

    | 設定      |  建議的值   | 說明                                        |
    | ----------------- | ------------ | ------------- |
    | **搜尋文字** | #Azure | 使用熱門程度足夠在指定間隔內產生新推文的雜湊標記。 當您是使用免費層且您的雜湊標記太熱門時，可能很快就會將認知服務 API 中的交易配額用完。 |
    | **頻率** | 分鐘 | 用於輪詢 Twitter 的頻率單位。  |
    | **間隔** | 15 | Twitter 要求之間所經過的時間 (以頻率為單位)。 |

3.  按一下 [儲存] 可連線到您的 Twitter 帳戶。 

現在，您的應用程式已連線到 Twitter。 接下來，您要連線至文字分析來偵測收集推文的情感。

## <a name="add-sentiment-detection"></a>新增情感偵測

1. 依序按一下 [新增步驟]、[新增動作]。

    ![選取 [新增步驟]，然後選取 [新增動作]](media/functions-twitter-email/new_step.png)

2. 在 [選擇動作] 中，按一下 [文字分析]，然後按一下 [偵測情感] 動作。

    ![偵測情感](media/functions-twitter-email/detect_sent.png)

3. 輸入連線名稱 (例如 `MyCognitiveServicesConnection`)、將您所儲存的認知服務 API 金鑰貼上，然後按一下 [建立]。  

4. 按一下 [要分析的文字] > [推文文字]，然後按一下 [儲存]。  

    ![偵測情感](media/functions-twitter-email/ds_tta.png)

現在您已設定情感偵測，可將連線新增至使用情感分數輸出的函式。

## <a name="connect-sentiment-output-to-your-function"></a>將情感輸出連線至您的函式

1. 在 Logic Apps 設計工具中，按一下 [新增步驟] > [新增動作]，然後按一下 [Azure Functions]。 

2. 按一下 [選擇 Azure 函式]，選取您稍早建立的 **CategorizeSentiment** 函式。  

    ![顯示 [選擇 Azure 函式] 的 Azure 函式方塊](media/functions-twitter-email/choose_fun.png)

3. 在**要求本文**中，依序按一下 [分數] 和 [儲存]。

    ![分數](media/functions-twitter-email/trigger_score.png)

現在，當邏輯應用程式傳送出情感分數時，就會將您的函式觸發。 函式會將以色彩標示的類別傳回至邏輯應用程式。 接下來，您要將函式傳回**紅色**的情感值時所傳送的電子郵件通知加以新增。 

## <a name="add-email-notifications"></a>新增電子郵件通知

工作流程的最後一個部分，是當情感計分為_紅色_時，要將電子郵件觸發。 此主題是使用 Outlook.com 連接器。 您可以執行類似的步驟，來使用 Gmail 或 Office 365 Outlook 連接器。   

1. 在 Logic Apps 設計工具中，按一下 [新增步驟] > [新增條件]。 

2. 按一下 [選擇值]，然後按一下 [本文]。 選取 [等於]、按一下 [選擇值] 並輸入 `RED`，然後按一下 [儲存]。 

    ![將條件新增至邏輯應用程式。](media/functions-twitter-email/condition.png)

3. 在 [若為是] 中、按一下 [新增動作]、搜尋 `outlook.com`、按一下 [傳送電子郵件]，然後登入您的 Outlook.com 帳戶。
    
    ![選擇條件的動作。](media/functions-twitter-email/outlook.png)

    > [!NOTE]
    > 如果您沒有 Outlook.com 帳戶，可以選擇另一個連接器，例如 Gmail 或 Office 365 Outlook

4. 在**傳送電子郵件**動作，使用資料表中指定的電子郵件設定。 

    ![設定傳送電子郵件動作的電子郵件。](media/functions-twitter-email/send_email.png)

    | 設定      |  建議的值   | 說明  |
    | ----------------- | ------------ | ------------- |
    | **To** | 輸入您的電子郵件地址 | 接收通知電子郵件地址。 |
    | **主旨** | 偵測到負面的推文情感  | 電子郵件通知的主旨列。  |
    | **內文** | 推文文字、位置 | 按一下 [推文文字] 和 [位置] 參數。 |

5.  按一下 [檔案] 。

現在，工作流程已完成，您可以啟用邏輯應用程式，並查看工作的函式。

## <a name="test-the-workflow"></a>測試工作流程

1. 在邏輯應用程式設計工具中，按一下 [執行] 可啟動應用程式。

2. 在左欄中，按一下 [概觀] 可查看邏輯應用程式的狀態。 
 
    ![邏輯應用程式執行狀態](media/functions-twitter-email/over1.png)

3. (選擇性) 按下其中一個執行，可查看執行的詳細資料。

4. 請移至您的函式、檢視記錄，並確認已收到情感值並已加以處理。
 
    ![檢視函式記錄](media/functions-twitter-email/sent.png)

5. 偵測到潛在的負面情感時，您會收到一封電子郵件。 如果您尚未收到電子郵件，可以將函式程式碼變更為每次傳回「紅色」︰

        return req.CreateResponse(HttpStatusCode.OK, "RED");

    確認電子郵件通知之後，請變更回原始的程式碼︰

        return req.CreateResponse(HttpStatusCode.OK, category);

    > [!IMPORTANT]
    > 完成此教學課程之後，您應停用邏輯應用程式。 您可以透過停用應用程式，在執行及用完認知服務 API 中的交易時可避免支付費用。

現在您已經知道要將 Functions 整合至 Logic Apps 工作流程有多麼輕鬆。

## <a name="disable-the-logic-app"></a>停用邏輯應用程式

若要停用邏輯應用程式，請依序按一下螢幕頂端的 [概觀] 及 [停用]。 這樣無須刪除應用程式，就可以使邏輯應用程式停止執行及產生費用。 

![函式記錄](media/functions-twitter-email/disable-logic-app.png)

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立認知服務 API 資源。
> * 建立可將推文情感進行分類的函式。
> * 建立連線至 Twitter 的邏輯應用程式。
> * 將情感偵測新增至邏輯應用程式。 
> * 將邏輯應用程式連線至函式。
> * 以函式的回應作為基礎來傳送電子郵件。

前進至下一個教學課程，了解如何針對函式建立無伺服器 API。

> [!div class="nextstepaction"] 
> [使用 Azure Functions 建立無伺服器 API](functions-create-serverless-api.md)

若要深入了解 Logic Apps，請參閱 [Azure Logic Apps](../logic-apps/logic-apps-overview.md)。

