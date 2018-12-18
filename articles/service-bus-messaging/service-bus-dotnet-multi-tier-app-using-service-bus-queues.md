---
title: 使用 Azure 服務匯流排的 .NET 多層應用程式 | Microsoft Docs
description: 協助您在 Azure 中開發多層式應用程式，以使用服務匯流排佇列在各層之間進行通訊的 .NET 教學課程。
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
ms.service: service-bus-messaging
ms.devlang: dotnet
ms.topic: article
ms.date: 09/05/2018
ms.author: spelluru
ms.openlocfilehash: b905feef4d3a5445ba27b92637448443c3bbb4a3
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855607"
---
# <a name="net-multi-tier-application-using-azure-service-bus-queues"></a>使用 Azure 服務匯流排佇列的 .NET 多層應用程式

使用 Visual Studio 和免費 Azure SDK for .NET 開發 Microsoft Azure 很容易。 本教學課程將逐步引導您完成相關步驟，以建立在本機環境中執行、使用多個 Azure 資源的應用程式。

您將了解下列內容：

* 如何透過單一下載和安裝，讓電腦適合用於進行 Azure 開發。
* 如何使用 Visual Studio 進行 Azure 相關開發。
* 如何使用 Web 與背景工作角色，在 Azure 建立多層式應用程式。
* 如何使用服務匯流排佇列在階層間通訊。

[!INCLUDE [create-account-note](../../includes/create-account-note.md)]

在本教學課程中，您將在 Azure 雲端服務中建置和執行多層式應用程式。 前端為 ASP.NET MVC Web 角色，而後端為使用服務匯流排佇列的背景工作角色。 您可以建立相同的多層應用程式，並使其前端作為部署至 Azure 網站而非雲端服務的 Web 專案。 您也可以試試 [.NET 內部部署/雲端混合式應用程式](../service-bus-relay/service-bus-dotnet-hybrid-app-using-service-bus-relay.md)教學課程。

下列螢幕擷取畫面顯示完成的應用程式：

![][0]

## <a name="scenario-overview-inter-role-communication"></a>案例概觀：內部角色通訊
為了提交訂單進行處理，Web 角色中執行的前端 UI 元件必須與背景工作角色中執行的中間層互動。 此範例使用服務匯流排傳訊在各層之間進行通訊。

在 Web 層與中間層之間使用服務匯流排傳訊，可使這兩個元件彼此脫鉤。 相較於直接傳訊 (亦即，TCP 或 HTTP)，Web 層不會直接連線至中間層，而是透過訊息的形式，將工作單位推播至服務匯流排，而後者會可靠地保管工作單位，直到中間層準備好取用並處理這些工作單位為止。

服務匯流排提供兩個實體來支援代理的傳訊，也就是：佇列和主題。 使用佇列時，每個傳送至佇列的訊息都是由單一接收者取用。 主題可支援發佈/訂用帳戶模式，亦即每個發佈的訊息會提供給在主題註冊的訂用帳戶。 每個訂閱在邏輯上會維護自己的訊息佇列。 訂用帳戶也可以設定成使用篩選規則，以將傳遞至訂用帳戶佇列的訊息限制為符合篩選條件的訊息。 下列範例使用服務匯流排佇列。

![][1]

此通訊機制具有數個超越直接傳訊的優點：

* **暫時分離。** 使用非同步傳訊模式時，生產者和取用者不需要同時處於線上狀態。 服務匯流排會可靠地保管訊息，直到取用方準備好接收訊息為止。 如此一來，分散式應用程式的元件即使中斷連線 (例如，基於維護原因而計劃性中斷，或基於元件損毀而意外中斷)，也不會影響整體系統。 此外，取用方應用程式只需要在一天中的特定時間處於線上狀態即可。
* **負載調節。** 在許多應用程式中，系統負載會隨時間改變，而處理每個工作單位所需的時間卻通常固定不變。 有佇列作為訊息生產者與取用者之間的中間者後，就只需佈建取用方應用程式 (背景工作) 來容納平均負載而非尖峰負載。 佇列的深度會隨著連入負載的改變而增加和縮短。 就處理應用程式負載所需的基礎結構數量而言，如此可直接節省金錢。
* **負載平衡。** 當負載增加時，可以新增更多的背景工作程序來讀取佇列中的訊息。 每個訊息僅由其中一個背景工作程序處理。 此外，這個提取型負載平衡機制可讓背景工作電腦獲得最佳利用，即使背景工作電腦的處理能力有所不同也一樣，因為背景工作電腦將以自己的最大速率提取訊息。 此模式通常稱為「競爭取用者」模式。
  
  ![][2]

下列幾節討論實作此架構的程式碼。

## <a name="create-a-namespace"></a>建立命名空間

第一個步驟是建立「命名空間」，並取得該命名空間的[共用存取簽章 (SAS) 金鑰](service-bus-sas.md)。 命名空間會為每個透過服務匯流排公開的應用程式提供應用程式界限。 建立命名空間時，系統會產生 SAS 金鑰。 命名空間名稱與 SAS 金鑰的結合提供一個認證，供服務匯流排驗證對應用程式的存取權。

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="create-a-web-role"></a>建立 Web 角色

在本節中，您會建置應用程式的前端。 首先，您會建立應用程式顯示的頁面。
之後，新增程式碼以提交項目給服務匯流排佇列，並顯示佇列的狀態資訊。

### <a name="create-the-project"></a>建立專案

1. 使用系統管理員權限啟動 Visual Studio：在 **Visual Studio** 程式圖示上按一下滑鼠右鍵，然後按一下 [以系統管理員身分執行]。 這篇文章稍後討論的 Azure 計算模擬器需要 Visual Studio 以系統管理員權限啟動。
   
   在 Visual Studio 的 [檔案] 功能表，按一下 [新增]，然後按一下 [專案]。
2. 從 [已安裝的範本] 的 [Visual C#] 下，按一下 [雲端]，然後按一下 [Azure 雲端服務]。 將專案命名為 **MultiTierApp**。 然後按一下 [確定] 。
   
   ![][9]
3. 從 [角色] 窗格，按兩下 [ASP.NET Web 角色]。
   
   ![][10]
4. 將滑鼠移至 [Azure 雲端服務解決方案] 下的 [WebRole1]，按一下鉛筆圖示，將 Web 角色重新命名為 **FrontendWebRole**。 然後按一下 [確定] 。 (請確定您輸入 "Frontend"，其中 e 為小寫，而不是 "FrontEnd"。)
   
   ![][11]
5. 在 [新增 ASP.NET 專案] 對話方塊的 [選取範本] 清單中，按一下 [MVC]。
   
   ![][12]
6. 還是在 [新增 ASP.NET 專案] 對話方塊中，按一下 [變更驗證] 按鈕。 在 [變更驗證] 對話方塊中，確定已選取 [不需要驗證]，然後按一下 [確定]。 在本教學課程中，您會部署不需要使用者登入的應用程式。
   
    ![][16]
7. 回到 [新增 ASP.NET 專案] 對話方塊中，按一下 [確定] 以建立專案。
8. 在 [方案總管] 的 [FrontendWebRole] 專案中，以滑鼠右鍵按一下 [參考]，然後按一下 [管理 NuGet 套件]。
9. 按一下 [瀏覽] 索引標籤，然後搜尋 **WindowsAzure.ServiceBus**。 選取 **WindowsAzure.ServiceBus** 套件，按一下 [安裝]，然後接受使用規定。
   
   ![][13]
   
   請注意，現在已參考必要的用戶端組件，並已新增一些新的程式碼檔案。
10. 在 [方案總管] 中，於 [模型] 上按一下滑鼠右鍵、按一下 [新增]，再按一下 [類別]。 在 [名稱] 方塊中，輸入名稱 **OnlineOrder.cs**。 然後按一下 [ **新增**]。

### <a name="write-the-code-for-your-web-role"></a>撰寫 Web 角色的程式碼
在本節中，您將建立應用程式顯示的各個頁面。

1. 在 Visual Studio 的 OnlineOrder.cs 檔案中，將現有的命名空間定義取代為下列程式碼：
   
   ```csharp
   namespace FrontendWebRole.Models
   {
       public class OnlineOrder
       {
           public string Customer { get; set; }
           public string Product { get; set; }
       }
   }
   ```
2. 在 [方案總管] 中，按兩下 [Controllers\HomeController.cs]。 在檔案頂端新增下列 **using** 陳述式，以納入您剛建立之模型的命名空間，以及服務匯流排。
   
   ```csharp
   using FrontendWebRole.Models;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   ```
3. 同時在 Visual Studio 的 HomeController.cs 檔案中，也將現有的命名空間定義取代為下列程式碼。 此程式碼包含將項目提交給佇列的處理方法。
   
   ```csharp
   namespace FrontendWebRole.Controllers
   {
       public class HomeController : Controller
       {
           public ActionResult Index()
           {
               // Simply redirect to Submit, since Submit will serve as the
               // front page of this application.
               return RedirectToAction("Submit");
           }
   
           public ActionResult About()
           {
               return View();
           }
   
           // GET: /Home/Submit.
           // Controller method for a view you will create for the submission
           // form.
           public ActionResult Submit()
           {
               // Will put code for displaying queue message count here.
   
               return View();
           }
   
           // POST: /Home/Submit.
           // Controller method for handling submissions from the submission
           // form.
           [HttpPost]
           // Attribute to help prevent cross-site scripting attacks and
           // cross-site request forgery.  
           [ValidateAntiForgeryToken]
           public ActionResult Submit(OnlineOrder order)
           {
               if (ModelState.IsValid)
               {
                   // Will put code for submitting to queue here.
   
                   return RedirectToAction("Submit");
               }
               else
               {
                   return View(order);
               }
           }
       }
   }
   ```
4. 從 [建置] 功能表中，按一下 [建置方案] 來測試您的工作到目前為止是否正確無誤。
5. 現在，為先前建立的 `Submit()` 方法建立檢視。 在 `Submit()` 方法 (未採用任何參數的 `Submit()` 的多載) 內按一下滑鼠右鍵，然後選擇 [新增檢視]。
   
   ![][14]
6. 隨即出現對話方塊，供您建立檢視。 在 [範本] 清單中，選擇 [建立]。 在 [模型類別] 清單中，選取 [OnlineOrder] 類別。
   
   ![][15]
7. 按一下 [新增] 。
8. 現在，變更應用程式的顯示名稱。 在 [方案總管] 中，按兩下 **Views\Shared\\_Layout.cshtml** 檔案以在 Visual Studio 編輯器中開啟。
9. 將所有出現的 [我的 ASP.NET 應用程式] 取代為 [Northwind Traders 產品]。
10. 移除 [首頁]、[關於] 和 [連絡人] 連結。 刪除反白顯示的程式碼：
    
    ![][28]
11. 最後，修改提交頁面，以納入佇列的一些相關資訊。 在 [方案總管] 中，按兩下 [Views\Home\Submit.cshtml] 檔案以在 Visual Studio 編輯器中開啟。 在 `<h2>Submit</h2>` 後面新增下列一行。 目前 `ViewBag.MessageCount` 是空的。 稍後您將在其中填入資料。
    
    ```html
    <p>Current number of orders in queue waiting to be processed: @ViewBag.MessageCount</p>
    ```
12. 您現在已實作 UI。 您可以按 **F5** 執行應用程式，確認它的外觀與預期一樣。
    
    ![][17]

### <a name="write-the-code-for-submitting-items-to-a-service-bus-queue"></a>撰寫增程式碼以提交項目給服務匯流排佇列
現在，將新增程式碼以提交項目給佇列。 首先，您會建立一個類別，並使其包含服務匯流排佇列連接資訊。 接著，從 Global.aspx.cs 初始化您的連線。 最後，更新稍早在 HomeController.cs 建立的提交程式碼，以將項目實際提交給服務匯流排佇列。

1. 在 [方案總管] 中，於 [FrontendWebRole] 上按一下滑鼠右鍵 (在專案而非角色上按一下滑鼠右鍵)。 按一下 [新增]，然後按一下 [類別]。
2. 將類別命名為 **QueueConnector.cs**。 按一下 [新增] 以建立類別。
3. 現在，新增可封裝連線資訊、並初始化與服務匯流排佇列連線的程式碼。 以下列程式碼取代 QueueConnector.cs 的整個內容，並將值輸入 `your Service Bus namespace` (命名空間名稱) 和 `yourKey`，後者是先前取自 Azure 入口網站的**主要金鑰**。
   
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Web;
   using Microsoft.ServiceBus.Messaging;
   using Microsoft.ServiceBus;
   
   namespace FrontendWebRole
   {
       public static class QueueConnector
       {
           // Thread-safe. Recommended that you cache rather than recreating it
           // on every request.
           public static QueueClient OrdersQueueClient;
   
           // Obtain these values from the portal.
           public const string Namespace = "your Service Bus namespace";
   
           // The name of your queue.
           public const string QueueName = "OrdersQueue";
   
           public static NamespaceManager CreateNamespaceManager()
           {
               // Create the namespace manager which gives you access to
               // management operations.
               var uri = ServiceBusEnvironment.CreateServiceUri(
                   "sb", Namespace, String.Empty);
               var tP = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                   "RootManageSharedAccessKey", "yourKey");
               return new NamespaceManager(uri, tP);
           }
   
           public static void Initialize()
           {
               // Using Http to be friendly with outbound firewalls.
               ServiceBusEnvironment.SystemConnectivity.Mode =
                   ConnectivityMode.Http;
   
               // Create the namespace manager which gives you access to
               // management operations.
               var namespaceManager = CreateNamespaceManager();
   
               // Create the queue if it does not exist already.
               if (!namespaceManager.QueueExists(QueueName))
               {
                   namespaceManager.CreateQueue(QueueName);
               }
   
               // Get a client to the queue.
               var messagingFactory = MessagingFactory.Create(
                   namespaceManager.Address,
                   namespaceManager.Settings.TokenProvider);
               OrdersQueueClient = messagingFactory.CreateQueueClient(
                   "OrdersQueue");
           }
       }
   }
   ```
4. 現在，請確定會呼叫您的 **Initialize** 方法。 在 [方案總管] 中，按兩下 [Global.asax\Global.asax.cs]。
5. 在 **Application_Start** 方法的結尾新增下列程式碼行。
   
   ```csharp
   FrontendWebRole.QueueConnector.Initialize();
   ```
6. 最後，更新稍早建立的 Web 程式碼，以提交項目給佇列。 在 [方案總管] 中，按兩下 [Controllers\HomeController.cs]。
7. 如下所示更新 `Submit()` 方法 (未採用任何參數的多載)，以取得佇列的訊息計數。
   
   ```csharp
   public ActionResult Submit()
   {
       // Get a NamespaceManager which allows you to perform management and
       // diagnostic operations on your Service Bus queues.
       var namespaceManager = QueueConnector.CreateNamespaceManager();
   
       // Get the queue, and obtain the message count.
       var queue = namespaceManager.GetQueue(QueueConnector.QueueName);
       ViewBag.MessageCount = queue.MessageCount;
   
       return View();
   }
   ```
8. 如下所示更新 `Submit(OnlineOrder order)` 方法 (採用一個參數的多載)，以提交訂單資訊給佇列。
   
   ```csharp
   public ActionResult Submit(OnlineOrder order)
   {
       if (ModelState.IsValid)
       {
           // Create a message from the order.
           var message = new BrokeredMessage(order);
   
           // Submit the order.
           QueueConnector.OrdersQueueClient.Send(message);
           return RedirectToAction("Submit");
       }
       else
       {
           return View(order);
       }
   }
   ```
9. 現在您可以重新執行應用程式。 每次您一提交訂單，訊息計數便會增加。
   
   ![][18]

## <a name="create-the-worker-role"></a>建立背景工作角色
您現在將建立背景工作角色來處理所提交的訂單。 本範例使用 **Worker Role with Service Bus Queue** Visual Studio 專案範本。 您已經從入口網站取得必要的認證。

1. 請確定您已將 Visual Studio 連接到您的 Azure 帳戶。
2. 在 Visual Studio 的 [方案總管] 中，於 [MultiTierApp] 專案下的 [角色] 資料夾上按一下滑鼠右鍵。
3. 按一下 [新增]，然後按一下 [新的背景工作角色專案]。 [新增新的角色專案] 對話方塊隨即出現。
   
   ![][26]
4. 在 [新增新的角色專案] 對話方塊中，按一下 [具有服務匯流排佇列的背景工作角色]。
   
   ![][23]
5. 在 [名稱] 方塊中，將專案命名為 **OrderProcessingRole**。 然後按一下 [ **新增**]。
6. 將您在＜建立服務匯流排命名空間＞一節的步驟 9 中取得的連接字串複製到剪貼簿。
7. 在 [方案總管] 中，請以滑鼠右鍵按一下您在步驟 5 中所建立的 **OrderProcessingRole** (請確定您是在 [角色] 下而不是類別下的 **OrderProcessingRole** 按一下滑鼠右鍵)。 然後按一下 [屬性]。
8. 在 [屬性] 對話方塊的 [設定] 索引標籤中，按一下 **Microsoft.ServiceBus.ConnectionString** 的 [值] 方塊內部，然後貼上您在步驟 6 中複製的端點值。
   
   ![][25]
9. 建立 **OnlineOrder**，以代表您從佇列處理它們時的順序。 您可以重複使用已建立的類別。 在 [方案總管] 中，以滑鼠右鍵按一下 **OrderProcessingRole** 類別 (在類別圖示而不是角色上按一下滑鼠右鍵)。 按一下 [新增]，然後按一下 [現有項目]。
10. 瀏覽至 **FrontendWebRole\Models** 的子資料夾，並按兩下 [OnlineOrder.cs]，以將其新增至此專案。
11. 在 **WorkerRole.cs** 中，將 **QueueName** 變數值從 `"ProcessingQueue"` 變更為 `"OrdersQueue"`，如下列程式碼所示。
    
    ```csharp
    // The name of your queue.
    const string QueueName = "OrdersQueue";
    ```
12. 在 WorkerRole.cs 檔案頂端新增下列 using 陳述式。
    
    ```csharp
    using FrontendWebRole.Models;
    ```
13. 在 `Run()` 函式的 `OnMessage()` 呼叫內，以下列程式碼取代 `try` 子句的內容。
    
    ```csharp
    Trace.WriteLine("Processing", receivedMessage.SequenceNumber.ToString());
    // View the message as an OnlineOrder.
    OnlineOrder order = receivedMessage.GetBody<OnlineOrder>();
    Trace.WriteLine(order.Customer + ": " + order.Product, "ProcessingMessage");
    receivedMessage.Complete();
    ```
14. 您已完成應用程式。 您可以測試完整的應用程式，方法是以滑鼠右鍵按一下 [方案總管] 中的 MultiTierApp 專案、選取 [設定為啟始專案]，然後按下 F5。 請注意，訊息計數不會增加，因為背景工作角色會處理佇列中的項目，並將它們標示為完成。 您可以檢視 Azure 計算模擬器 UI，來查看背景工作角色的追蹤輸出。 作法為在工作列的通知區中的計算模擬器圖示上按一下滑鼠右鍵，並選取 [顯示計算模擬器 UI]。
    
    ![][19]
    
    ![][20]

## <a name="next-steps"></a>後續步驟
若要深入了解服務匯流排，請參閱下列資源：  

* [開始使用服務匯流排佇列][sbacomqhowto]
* [服務匯流排服務頁面][sbacom]  

若要深入了解多層式案例，請參閱︰  

* [使用儲存體資料表、佇列與 Blob 的 .NET 多層式應用程式][mutitierstorage]  

[0]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[1]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-100.png
[2]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-101.png
[9]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-10.png
[10]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-11.png
[11]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-02.png
[12]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-12.png
[13]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-13.png
[14]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-33.png
[15]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-34.png
[16]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-14.png
[17]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app.png
[18]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-app2.png

[19]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-38.png
[20]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-39.png
[23]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRole1.png
[25]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBWorkerRoleProperties.png
[26]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/SBNewWorkerRole.png
[28]: ./media/service-bus-dotnet-multi-tier-app-using-service-bus-queues/getting-started-multi-tier-40.png

[sbacom]: https://azure.microsoft.com/services/service-bus/  
[sbacomqhowto]: service-bus-dotnet-get-started-with-queues.md  
[mutitierstorage]: https://code.msdn.microsoft.com/Windows-Azure-Multi-Tier-eadceb36
