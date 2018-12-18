---
title: LUIS 和 QnAMaker - Bot 整合
titleSuffix: Azure Cognitive Services
description: 在 Bot 中整合 QnA Maker 與 LUIS 的逐步教學課程。
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: 53e46fa84bcd7b96403dcb0ec70b45b800bc4acb
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042001"
---
# <a name="integrate-qna-maker-and-luis-to-distribute-your-knowledge-base"></a>整合 QnA Maker 與 LUIS 散發您的知識庫
隨著 QnA Maker 知識庫變得愈來愈大，整體而言相當難以維護，因此需要將知識庫分割成較小的邏輯區塊。

雖然可以直接在 QnA Maker 中建立多個知識庫，但是您需要一些邏輯將內送的問題傳送到適當的知識庫。 您可以使用 LUIS 來達成此目的。

## <a name="architecture"></a>架構

![QnA Maker LUIS 架構](../media/qnamaker-tutorials-qna-luis/qnamaker-luis-architecture.PNG)

在上述情況中，QnA Maker 會先從 LUIS 模型時取得內送問題的意圖，然後使用該意圖將問題傳送到正確的 QnA Maker 知識庫。

## <a name="prerequisites"></a>必要條件
- 登入 [LUIS](https://www.luis.ai/) 入口網站並[建立應用程式](https://docs.microsoft.com/azure/cognitive-services/luis/create-new-app)。
- 按照情節[新增意圖](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents)。
- [訓練](https://docs.microsoft.com/azure/cognitive-services/luis/luis-how-to-train)和[發佈](https://docs.microsoft.com/azure/cognitive-services/luis/publishapp) LUIS 應用程式。
- 登入 [QnA Maker](https://qnamaker.ai)，並根據情節[建立](https://www.qnamaker.ai/Create)知識庫。
- 測試並發佈知識庫。

## <a name="qna-maker--luis-bot"></a>QnA Maker + LUIS 聊天機器人
1. 首先使用 LUIS 範本建立 Web 應用程式聊天機器人，並將聊天機器人連結到稍早建立的 LUIS 應用程式，然後修改意圖。 關於詳細步驟，請參閱[這裡](https://docs.microsoft.com/azure/cognitive-services/luis/luis-csharp-tutorial-build-bot-framework-sample)。

2. 將相依性加入至檔案的頂端，並加入其他相依性：

    ```
    using RestSharp;
    using System.Collections.Generic;
    using Newtonsoft.Json;
    ```
3. 加入呼叫 QnA Maker 服務的下列類別：

    ```
        /// <summary>
        /// QnAMakerService is a wrapper over the QnA Maker REST endpoint
        /// </summary>
        [Serializable]
        public class QnAMakerService
        {
            private string qnaServiceHostName;
            private string knowledgeBaseId;
            private string endpointKey;
    
            /// <summary>
            /// Initialize a particular endpoint with it's details
            /// </summary>
            /// <param name="hostName">Hostname of the endpoint</param>
            /// <param name="kbId">Knowledge base ID</param>
            /// <param name="ek">Endpoint Key</param>
            public QnAMakerService(string hostName, string kbId, string ek)
            {
                qnaServiceHostName = hostName;
                knowledgeBaseId = kbId;
                endpointKey = ek;
            }
    
            /// <summary>
            /// Call the QnA Maker endpoint and get a response
            /// </summary>
            /// <param name="query">User question</param>
            /// <returns></returns>
            public string GetAnswer(string query)
            {
                var client = new RestClient( qnaServiceHostName + "/qnamaker/knowledgebases/" + knowledgeBaseId + "/generateAnswer");
                var request = new RestRequest(Method.POST);
                request.AddHeader("authorization", "EndpointKey " + endpointKey);
                request.AddHeader("content-type", "application/json");
                request.AddParameter("application/json", "{\"question\": \"" + query + "\"}", ParameterType.RequestBody);
                IRestResponse response = client.Execute(request);
    
                // Deserialize the response JSON
                QnAAnswer answer = JsonConvert.DeserializeObject<QnAAnswer>(response.Content);
    
                // Return the answer if present
                if (answer.answers.Count > 0)
                    return answer.answers[0].answer;
                else
                    return "No good match found.";
            }
        }
    
        /* START - QnA Maker Response Class */
        public class Metadata
        {
            public string name { get; set; }
            public string value { get; set; }
        }
    
        public class Answer
        {
            public IList<string> questions { get; set; }
            public string answer { get; set; }
            public double score { get; set; }
            public int id { get; set; }
            public string source { get; set; }
            public IList<object> keywords { get; set; }
            public IList<Metadata> metadata { get; set; }
        }
    
        public class QnAAnswer
        {
            public IList<Answer> answers { get; set; }
        }
        /* END - QnA Maker Response Class */
    ```

3. 在對應的知識庫中，移至 [https://qnamaker.ai] -> [我的知識庫] -> 檢視程式碼。 取得下列資訊：

    ![QnA Maker HTTP 要求](../media/qnamaker-tutorials-qna-luis/qnamaker-http-request.png)

4. 對於每個知識庫將 QnAMakerService 類別具現化：
    ```
            // Instantiate the knowledge bases
            public QnAMakerService hrQnAService = new QnAMakerService("https://hrkb.azurewebsites.net", "<HR knowledge base id>", "<HR endpoint key>");
            public QnAMakerService payrollQnAService = new QnAMakerService("https://payrollkb.azurewebsites.net", "<Payroll knowledge base id>", "<Payroll endpoint key>");
            public QnAMakerService financeQnAService = new QnAMakerService("https://financekb.azurewebsites.net", "<Finance knowledge base id>", "<Finance endpoint key>");
    ```

5. 對於意圖呼叫對應的知識庫。
    ```
            // HR Intent
            [LuisIntent("HR")]
            public async Task CancelIntent(IDialogContext context, LuisResult result)
            {
                // Ask the HR knowledge base
                await context.PostAsync(hrQnAService.GetAnswer(result.Query));
            }
    
            // Payroll intent
            [LuisIntent("Payroll")]
            public async Task GreetingIntent(IDialogContext context, LuisResult result)
            {
                // Ask the payroll knowledge base
                await context.PostAsync(payrollQnAService.GetAnswer(result.Query));
            }
    
            // Finance intent
            [LuisIntent("Finance")]
            public async Task HelpIntent(IDialogContext context, LuisResult result)
            {
                // Ask the finance knowledge base
                await context.PostAsync(financeQnAService.GetAnswer(result.Query));
            }
    ```

## <a name="build-the-bot"></a>組建聊天機器人
1. 在程式碼編輯器中，以滑鼠右鍵按一下 [`build.cmd`] 並選取 [從主控台執行]。

    ![從主控台執行](../media/qnamaker-tutorials-qna-luis/run-from-console.png)

2. 程式碼檢視畫面會改換成顯示組建進度和結果的終端機視窗。

    ![主控台組建](../media/qnamaker-tutorials-qna-luis/console-build.png)

## <a name="test-the-bot"></a>測試聊天機器人
在 Azure 入口網站中，選取 [Test in Web Chat]\(在網路聊天中測試\) 以測試 Bot。 鍵入不同意圖的訊息，以便取得對應的知識庫之中的回應。

![Web 交談測試](../media/qnamaker-tutorials-qna-luis/qnamaker-web-chat.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [建立 QnA Maker 的商務持續性方案](../How-To/business-continuity-plan.md)
