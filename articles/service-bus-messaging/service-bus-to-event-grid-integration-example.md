---
title: Azure 服務匯流排與 Event Grid 的整合範例 | Microsoft Docs
description: 本文提供服務匯流排傳訊與 Event Grid 的整合範例。
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: f99766cb-8f4b-4baf-b061-4b1e2ae570e4
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 02/15/2018
ms.author: spelluru
ms.openlocfilehash: ca4a6bfcc0f1041e76c07b3b240d7c641ef8f587
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699859"
---
# <a name="azure-service-bus-to-azure-event-grid-integration-examples"></a>Azure 服務匯流排與 Azure Event Grid 的整合範例

在本文中，您可了解如何設定 Azure 函式和邏輯應用程式，以根據從 Azure Event Grid 接收的事件來接收訊息。 您將執行下列動作：
 
* 建立簡單[測試 Azure 函式](#test-function-setup)，以便偵錯及檢視來自 Event Grid 的初始事件流程。 不論您是否執行其他步驟，都要執行此步驟。
* 建立 [Azure 函式來接收和處理 Azure 服務匯流排訊息](#receive-messages-using-azure-function) (以 Event Grid 事件為基礎)。
* 利用 [Azure App Service 的 Logic Apps 功能](#receive-messages-using-azure-logic-app)。

您建立的範例假設服務匯流排主題有兩個訂用帳戶。 此範例還假設所建立的 Event Grid 訂用帳戶只會針對一個服務匯流排訂用帳戶傳送事件。 

在此範例中，您會將訊息傳送至服務匯流排主題，然後確認已針對此服務匯流排訂用帳戶產生事件。 函式或邏輯應用程式會從服務匯流排訂用帳戶接收訊息，然後完成它。

## <a name="prerequisites"></a>必要條件
開始之前，請確定您已完成下面兩節中的步驟。

### <a name="create-a-service-bus-namespace"></a>建立服務匯流排命名空間

建立服務匯流排進階命名空間，並建立具有兩個訂用帳戶的服務匯流排主題。

### <a name="send-a-message-to-the-service-bus-topic"></a>將訊息傳送至服務匯流排主題

您可以使用任何方法，將訊息傳送至服務匯流排主題。 此程序結尾的範例程式碼假設您目前使用 Visual Studio 2017。

1. 複製 [GitHub azure-service-bus 存放庫](https://github.com/Azure/azure-service-bus/)。

1. 在 Visual Studio 中，移至 \samples\DotNet\Microsoft.ServiceBus.Messaging\ServiceBusEventGridIntegration 資料夾，然後開啟 SBEventGridIntegration.sln 檔案。

1. 移至 **MessageSender** 專案，然後選取 **Program.cs**。

   ![8][]

1. 填入主題名稱和連接字串，然後執行下列主控台應用程式程式碼：

    ```CSharp
    const string ServiceBusConnectionString = "YOUR CONNECTION STRING";
    const string TopicName = "YOUR TOPIC NAME";
    ```

## <a name="set-up-a-test-function"></a>設定測試函式

在處理整個案例之前，至少設定一個小型測試函式，以便用來偵錯及觀察哪些事件正在流動。

1. 在 Azure 入口網站中，建立新的 Azure Functions 應用程式。 若要了解 Azure Functions 的基本概念，請參閱 [Azure Functions 文件](https://docs.microsoft.com/azure/azure-functions/)。

1. 在您新建立的函式中，選取加號 (+) 以新增 HTTP 觸發程序函式：

    ![2][]
    
    [搭配預先建立的函式快速開始使用] 視窗隨即開啟。

    ![3][]

1. 選取 [Webhook + API] 按鈕，選取 [CSharp]，然後選取 [建立這個函式]。
 
1. 將下列程式碼貼到函式中：

    ```CSharp
    #r "Newtonsoft.Json"
    using System.Net;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    
    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        log.Info("C# HTTP trigger function processed a request.");
        // parse query parameter
        var content = req.Content;
    
        string jsonContent = await content.ReadAsStringAsync(); 
        log.Info($"Received Event with payload: {jsonContent}");
    
    IEnumerable<string> headerValues;
    if (req.Headers.TryGetValues("Aeg-Event-Type", out headerValues))
    {
    var validationHeaderValue = headerValues.FirstOrDefault();
    if(validationHeaderValue == "SubscriptionValidation")
    {
    var events = JsonConvert.DeserializeObject<GridEvent[]>(jsonContent);
         var code = events[0].Data["validationCode"];
         return req.CreateResponse(HttpStatusCode.OK,
         new { validationResponse = code });
    }
    }
    
        return jsonContent == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Pass a name on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, "Hello " + jsonContent);
    }
    
    public class GridEvent
    {
        public string Id { get; set; }
        public string EventType { get; set; }
        public string Subject { get; set; }
        public DateTime EventTime { get; set; }
        public Dictionary<string, string> Data { get; set; }
        public string Topic { get; set; }
    }
    ```

1. 選取 [儲存並執行]。

## <a name="connect-the-function-and-namespace-via-event-grid"></a>透過 Event Grid 連線函式和命名空間

在這一節中，您會將函式與服務匯流排命名空間繫結在一起。 在此範例中，使用 Azure 入口網站。 若要了解如何使用 PowerShell 或 Azure CLI 來執行此程序，請參閱 [Azure 服務匯流排與 Azure Event Grid 的整合概觀](service-bus-to-event-grid-integration-concept.md)。

若要建立 Azure Event Grid 訂用帳戶，請執行下列作業：
1. 在 Azure 入口網站中，移至您的命名空間，然後在左窗格中選取 [Event Grid]。  
    您的命名空間視窗隨即開啟，其右窗格中顯示兩個 Event Grid 訂用帳戶。

    ![20][]

1. 選取 [事件訂用帳戶]。  
    [事件訂用帳戶] 視窗隨即開啟。 下圖顯示一個表單，可供訂閱 Azure 函式或 Webhook，而不需套用任何篩選條件。

    ![21][]

1. 完成所示的表單，然後記得在 [尾碼篩選條件] 方塊中，輸入相關的篩選條件。

1. 選取 [建立] 。

1. 如「必要條件」一節所述，將傳送訊息至服務匯流排主題，並透過 Azure Functions 的 [監視] 功能來確認事件正在流動。

下一個步驟是將函式與服務匯流排命名空間繫結在一起。 在此範例中，使用 Azure 入口網站。 若要了解如何使用 PowerShell 或 Azure CLI 來執行此步驟，請參閱 [Azure 服務匯流排與 Azure Event Grid 的整合概觀](service-bus-to-event-grid-integration-concept.md)。

![9][]

### <a name="receive-messages-by-using-azure-functions"></a>使用 Azure Functions 接收訊息

在前一節中，您已觀察到簡單測試和偵錯案例，並確定事件正在流動。 

在這一節中，您將了解如何在收到事件之後接收和處理訊息。

您會如下列範例所示新增 Azure 函式，因為 Azure Functions 內的服務匯流排函式原本不支援新的 Event Grid 整合。

1. 在您於必要條件中開啟的相同 Visual Studio 方案中，選取 **ReceiveMessagesOnEvent.cs**。 

    ![10][]

1. 在下列程式碼中輸入您的連接字串：

    ```Csharp
    const string ServiceBusConnectionString = "YOUR CONNECTION STRING";
    ```

1. 在 Azure 入口網站中，針對您在「設定測試函式」一節中建立的 Azure 函式下載發行設定檔。

    ![11][]

1. 在 Visual Studio 中，以滑鼠右鍵按一下 [SBEventGridIntegration]，然後選取 [發佈]。 

1. 在您先前下載之發行設定檔的 [發佈] 窗格中，選取 [匯入設定檔]，然後選取 [發佈]。

    ![12][]

1. 發佈新的 Azure 函式之後，建立新的 Azure Event Grid 訂用帳戶，以指向新的 Azure 函式。  
    在 [結尾為] 方塊中，務必套用正確的篩選條件，這應該是您的服務匯流排訂用帳戶名稱。

1. 將訊息傳送到您先前建立的 Azure 服務匯流排主題，然後在 Azure 入口網站中監視 Azure Functions 記錄，確保事件正在流動並已收到訊息。

    ![12-1][]

### <a name="receive-messages-by-using-logic-apps"></a>使用 Logic Apps 接收訊息

執行下列作業，將邏輯應用程式與 Azure 服務匯流排和 Azure Event Grid 連在一起：

1. 在 Azure 入口網站中建立新的邏輯應用程式，然後選取 [Event Grid] 作為啟動動作。

    ![13][]

    Logic Apps 設計工具視窗隨即開啟。

    ![14][]

1. 透過下列方式新增您的資訊︰

    a. 在 [資源名稱] 方塊中，輸入您自己的命名空間名稱。 

    b. 在 [進階選項] 下的 [尾碼篩選條件] 方塊中，輸入您的訂用帳戶。

1. 新增服務匯流排接收動作，以從主題訂用帳戶接收訊息。  
    最後一個動作如下圖所示：

    ![15][]

1. 新增完整事件，如下圖所示：

    ![16][]

1. 如「必要條件」一節所述，儲存邏輯應用程式，並將訊息傳送至服務匯流排主題。  
    觀察邏輯應用程式的執行狀況。 若要檢視更多執行資料，請選取 [概觀]，然後檢視 [執行歷程記錄] 下的資料。

    ![17][]

    ![18][]

## <a name="next-steps"></a>後續步驟

* 深入了解 [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/)。
* 深入了解 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/)。
* 深入了解 [Azure App Service 的 Logic Apps 功能](https://docs.microsoft.com/azure/logic-apps/)。
* 深入了解 [Azure 服務匯流排](https://docs.microsoft.com/azure/service-bus/)。


[2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid2.png
[3]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid3.png
[7]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid7.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[10]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid10.png
[11]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid11.png
[12]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12.png
[12-1]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-1.png
[12-2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-2.png
[13]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid13.png
[14]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid14.png
[15]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid15.png
[16]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid16.png
[17]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid17.png
[18]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid18.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png
