---
title: 包含檔案
description: 包含檔案，其中包含用於建立後端 ASP .NET WebAPI 專案的程式碼。
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/04/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 634bb14cfef3df2cf944eeafbfa8d671afa4ac98
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "33835792"
---
## <a name="create-the-webapi-project"></a>建立 WebAPI 專案
下列各節討論如何建立新的 ASP.NET WebAPI 後端。 此程序有三個主要用途：

- **驗證用戶端**：您可以新增訊息處理常式來驗證用戶端要求，並將使用者與要求建立關聯。
- **使用 WebAPI 後端註冊通知**：您可新增一個控制器來處理新的註冊，以便用戶端裝置接收通知。 經過驗證的使用者名稱會自動新增至註冊作為 [標記](../articles/notification-hubs/notification-hubs-tags-segment-push-message.md)。
- **傳送通知給用戶端**：您可新增一個控制器，以便使用者對與標籤建立關聯的裝置和用戶端觸發安全的推送。 

執行下列動作，建立新的 ASP.NET WebAPI 後端： 

> [!IMPORTANT]
> 如果您使用 Visual Studio 2015 或更新版本，在開始本教學課程之前，請確定您已安裝適用於 Visual Studio 的最新版 NuGet 套件管理員。 
>
>若要檢查版本，請啟動 Visual Studio。 在 [工具] 功能表上，選取 [擴充功能和更新]。 搜尋您的 Visual Studio 版本中的 **NuGet Package Manager**，然後確定您已安裝最新版本。 如果您的版本不是最新版本，請將它解除安裝，然後重新安裝 NuGet 套件管理員。
 
![][B4]

> [!NOTE]
> 確定您已安裝 Visual Studio [Azure SDK](https://azure.microsoft.com/downloads/) 以進行網站部署。 
> 

1. 啟動 Visual Studio 或 Visual Studio Express。 

2. 選取 [伺服器總管] ，然後登入您的 Azure 帳戶。 若要在您的帳戶上建立網站資源，您必須登入。

3. 在 Visual Studio 中，以滑鼠右鍵按一下 Visual Studio 解決方案，並指向 [新增]，然後按一下 [新增專案]。 
4. 展開 [Visual C#]，並選取 [Web]，然後按一下 [ASP.NET Web 應用程式]。

4. 在 [名稱] 方塊中，輸入 **AppBackend**，然後選取 [確定]。 
   
    ![[新增專案] 視窗][B1]

5. 在 [新增 ASP.NET 專案] 視窗中，選取 [Web API] 核取方塊，然後選取 [確定]。
   
    ![[新增 ASP.NET 專案] 視窗][B2]

6. 在 [設定 Microsoft Azure Web 應用程式] 視窗中，選取訂用帳戶，然後在 [App Service 方案] 清單中，執行下列其中一個動作：

    * 選取您已建立的 App Service 方案。 
    * 選取 [建立新的 App Service 方案]，然後建立一個新方案。 
    
  在此教學課程中您不需要資料庫。 在您選取 App Service 方案之後，選取 [確定]  來建立專案。
   
    ![[定 Microsoft Azure Web 應用程式] 視窗][B5]

## <a name="authenticate-clients-to-the-webapi-backend"></a>對 WebAPI 後端驗證用戶端
在本節中，您會為新的後端建立名為 **AuthenticationTestHandler** 的新訊息處理常式類別。 這個類別衍生自 [DelegatingHandler](https://msdn.microsoft.com/library/system.net.http.delegatinghandler.aspx) 並新增為訊息處理常式，以便處理進入後端的所有要求。 

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [AppBackend] 專案，然後依序選取 [新增] 和 [類別]。 
 
2. 將新類別命名為 **AuthenticationTestHandler.cs**，然後選取 [新增] 以產生類別。 為了簡單起見，此類別使用「基本驗證」來驗證使用者。 您的應用程式可以使用任何驗證結構描述。

3. 在 AuthenticationTestHandler.cs 中，加入下列 `using` 陳述式：
   
    ```csharp
    using System.Net.Http;
    using System.Threading;
    using System.Security.Principal;
    using System.Net;
    using System.Text;
    using System.Threading.Tasks;
    ```

4. 在 AuthenticationTestHandler.cs 中，以下列程式碼取代 `AuthenticationTestHandler` 類別定義： 
   
    下列三個條件都成立時，此處理常式會授權要求：
   
   * 要求包含「授權」標頭。 
   * 要求使用 *基本* 驗證。 
   * 使用者名稱字串和密碼字串是相同的字串。
     
  否則，會拒絕此要求。 此驗證不是真正的驗證和授權方法。 這只是本教學課程的簡單範例。
     
  如果要求訊息已經由 `AuthenticationTestHandler` 驗證及授權，則基本驗證使用者會附加至 [HttpContext](https://msdn.microsoft.com/library/system.web.httpcontext.current.aspx) 上的目前要求。 稍後另一個控制器 (RegisterController) 會使用 HttpContext 中的使用者資訊，將 [標記](https://msdn.microsoft.com/library/azure/dn530749.aspx) 新增至通知註冊要求。

    ```csharp     
    public class AuthenticationTestHandler : DelegatingHandler
    {
        protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
        {
            var authorizationHeader = request.Headers.GetValues("Authorization").First();
    
            if (authorizationHeader != null && authorizationHeader
                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
            {
                string authorizationUserAndPwdBase64 =
                    authorizationHeader.Substring("Basic ".Length);
                string authorizationUserAndPwd = Encoding.Default
                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
                string user = authorizationUserAndPwd.Split(':')[0];
                string password = authorizationUserAndPwd.Split(':')[1];
    
                if (verifyUserAndPwd(user, password))
                {
                    // Attach the new principal object to the current HttpContext object
                    HttpContext.Current.User =
                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
                    System.Threading.Thread.CurrentPrincipal =
                        System.Web.HttpContext.Current.User;
                }
                else return Unauthorized();
            }
            else return Unauthorized();
    
            return base.SendAsync(request, cancellationToken);
        }
    
        private bool verifyUserAndPwd(string user, string password)
        {
            // This is not a real authentication scheme.
            return user == password;
        }
    
        private Task<HttpResponseMessage> Unauthorized()
        {
            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
            var tsc = new TaskCompletionSource<HttpResponseMessage>();
            tsc.SetResult(response);
            return tsc.Task;
        }
    }
    ``` 
    > [!NOTE]
    > 安全性注意事項：`AuthenticationTestHandler` 類別未提供真正的驗證。 它僅可用於模仿基本驗證而且並不安全。 您必須在生產應用程式和服務中實作安全的驗證機制。                
5. 若要註冊訊息處理常式，請在 **App_Start/WebApiConfig.cs** 類別中 `Register` 方法的結尾新增下列程式碼：

    ```csharp   
    config.MessageHandlers.Add(new AuthenticationTestHandler());
    ```
6. 儲存您的變更。

## <a name="register-for-notifications-by-using-the-webapi-backend"></a>使用 WebAPI 後端來註冊通知
在本節中，您會將新的控制器新增至 WebAPI 後端來處理要求，以使用通知中樞的用戶端程式庫為使用者和裝置註冊通知。 控制器會對已由 `AuthenticationTestHandler` 驗證並附加至 HttpContext 的使用者，新增使用者標記。 此標籤具有以下字串格式：`"username:<actual username>"`。

1. 在 [方案總管] 中，以滑鼠右鍵按一下 [AppBackend] 專案，然後選取 [管理 NuGet 套件]。

2. 在左窗格中，選取 [線上]，然後在 [搜尋] 方塊中輸入 **Microsoft.Azure.NotificationHubs**。

3. 選取結果清單中的 [Microsoft Azure 通知中樞]，然後選取 [安裝]。 請完成安裝，然後關閉 [NuGet Package Manager] 視窗。
   
    此動作會使用 <a href="http://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/">Microsoft.Azure.Notification Hubs NuGet 套件</a>來新增對 Azure 通知中樞 SDK 的參考。

4. 建立新的類別檔案，代表與用來傳送通知的通知中樞間的連線。 在 [方案總管] 中，以滑鼠右鍵按一下 **Models** 資料夾，選取 [新增]，然後選取 [類別]。 將新類別命名為 **Notifications.cs**，然後選取 [新增] 以產生類別。 
   
    ![[新增項目] 視窗][B6]

5. 在 Notifications.cs 中，將下列 `using` 陳述式新增在檔案頂端：
   
    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

6. 以下列程式碼取代 `Notifications` 類別定義，並以通知中樞的連接字串 (含完整存取權) 和中心名稱 (可在 [Azure 入口網站](http://portal.azure.com)取代) 取代兩個預留位置：
   
    ```csharp
    public class Notifications
    {
        public static Notifications Instance = new Notifications();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>", 
                                                                            "<hub name>");
        }
    }
    ```
7. 接下來，建立名為 **RegisterController** 的新控制器。 在 [方案總管] 中，以滑鼠右鍵按一下 **Controllers** 資料夾，選取 [新增]，然後選取 [控制器]。 

8. 選取 [Web API 2 控制器 - 空的]，然後選取 [新增]。
   
    ![[新增 Scaffold] 視窗][B7]
   
9. 在 [控制器名稱] 方塊中，輸入 **RegisterController** 為新的類別命名，然後選取 [新增]。

    ![[新增控制器] 視窗][B8]

10. 在 RegisterController.cs 中，加入下列 `using` 陳述式：
   
    ```csharp
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Azure.NotificationHubs.Messaging;
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
11. 在 `RegisterController` 類別定義中加入下列程式碼。 在此程式碼中，您會為已附加至 HttpContext 的使用者新增使用者標籤。 您新增的訊息篩選 `AuthenticationTestHandler` 會驗證此使用者並附加至 HttpContext。 您也可以新增選擇性檢查，以驗證使用者是否有權註冊所要求的標籤。
   
    ```csharp
    private NotificationHubClient hub;

    public RegisterController()
    {
        hub = Notifications.Instance.Hub;
    }

    public class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    // POST api/register
    // This creates a registration id
    public async Task<string> Post(string handle = null)
    {
        string newRegistrationId = null;

        // make sure there are no existing registrations for this push handle (used for iOS and Android)
        if (handle != null)
        {
            var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

            foreach (RegistrationDescription registration in registrations)
            {
                if (newRegistrationId == null)
                {
                    newRegistrationId = registration.RegistrationId;
                }
                else
                {
                    await hub.DeleteRegistrationAsync(registration);
                }
            }
        }

        if (newRegistrationId == null) 
            newRegistrationId = await hub.CreateRegistrationIdAsync();

        return newRegistrationId;
    }

    // PUT api/register/5
    // This creates or updates a registration (with provided channelURI) at the specified id
    public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
    {
        RegistrationDescription registration = null;
        switch (deviceUpdate.Platform)
        {
            case "mpns":
                registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "wns":
                registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "apns":
                registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                break;
            case "gcm":
                registration = new GcmRegistrationDescription(deviceUpdate.Handle);
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }

        registration.RegistrationId = id;
        var username = HttpContext.Current.User.Identity.Name;

        // add check if user is allowed to add these tags
        registration.Tags = new HashSet<string>(deviceUpdate.Tags);
        registration.Tags.Add("username:" + username);

        try
        {
            await hub.CreateOrUpdateRegistrationAsync(registration);
        }
        catch (MessagingException e)
        {
            ReturnGoneIfHubResponseIsGone(e);
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }

    // DELETE api/register/5
    public async Task<HttpResponseMessage> Delete(string id)
    {
        await hub.DeleteRegistrationAsync(id);
        return Request.CreateResponse(HttpStatusCode.OK);
    }

    private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
    {
        var webex = e.InnerException as WebException;
        if (webex.Status == WebExceptionStatus.ProtocolError)
        {
            var response = (HttpWebResponse)webex.Response;
            if (response.StatusCode == HttpStatusCode.Gone)
                throw new HttpRequestException(HttpStatusCode.Gone.ToString());
        }
    }
    ```
12. 儲存您的變更。

## <a name="send-notifications-from-the-webapi-backend"></a>從 WebAPI 後端傳送通知
在本節中，您會新增控制器，以便用戶端裝置傳送通知。 此通知是以使用者名稱標籤為基礎，其使用 ASP.NET WebAPI 後端中的 Azure 通知中樞 .NET 程式庫。

1. 以您在上一節中建立 **RegisterController** 的相同方式，建立另一個名為 **NotificationsController**的新控制器。

2. 在 NotificationsController.cs 中，加入下列 `using` 陳述式：
   
    ```csharp
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
3. 在 **NotificationsController** 類別中新增下列方法：
   
    此程式碼會傳送以平台通知服務 (PNS) `pns` 參數為基礎的通知類型。 `to_tag` 的值用來設定訊息上的 *username* 標記。 此標記必須符合作用中通知中樞註冊的使用者名稱標記。 通知訊息是取自 POST 要求主體，並針對目標 PNS 格式化。 
   
    視您的支援裝置用來接收通知的 PNS 而言，可支援各種格式的通知。 例如在 Windows 裝置上，您可以搭配 WNS 使用不受其他 PNS 直接支援的[快顯通知](https://msdn.microsoft.com/library/windows/apps/br230849.aspx)。 在這類情況下，您的後端必須針對您打算支援的裝置 PNS，將通知格式化為支援的通知。 然後在 [NotificationHubClient 類別](https://msdn.microsoft.com/library/azure/microsoft.azure.notificationhubs.notificationhubclient_methods.aspx) 上使用適當的傳送 API。
   
    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
    {
        var user = HttpContext.Current.User.Identity.Name;
        string[] userTag = new string[2];
        userTag[0] = "username:" + to_tag;
        userTag[1] = "from:" + user;

        Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
        HttpStatusCode ret = HttpStatusCode.InternalServerError;

        switch (pns.ToLower())
        {
            case "wns":
                // Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                break;
            case "apns":
                // iOS
                var alert = "{\"aps\":{\"alert\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                break;
            case "gcm":
                // Android
                var notif = "{ \"data\" : {\"message\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendGcmNativeNotificationAsync(notif, userTag);
                break;
        }

        if (outcome != null)
        {
            if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
            {
                ret = HttpStatusCode.OK;
            }
        }

        return Request.CreateResponse(ret);
    }
    ```
4. 若要執行應用程式並確保工作到目前為止的準確性，請選取 **F5** 鍵。 應用程式會開啟網頁瀏覽器並顯示於 ASP.NET 首頁上。 

## <a name="publish-the-new-webapi-backend"></a>發佈新的 WebAPI 後端
接下來，您可將應用程式部署到 Azure 網站，讓它得以從所有裝置存取。 

1. 以滑鼠右鍵按一下 **AppBackend** 專案，然後選取 [發佈]。

2. 選取 [Microsoft Azure App Service] 作為發佈目標，然後選取 [發佈]。 [建立 App Service] 視窗隨即開啟。 您可以在此建立在 Azure 中執行 ASP.NET Web 應用程式所需的所有 Azure 資源。

    ![[Microsoft Azure App Service] 圖格][B15]

3. 在 [建立 App Service] 視窗中，選取您的 Azure 帳戶。 選取 [變更類型] > [Web 應用程式]。 保留預設 [Web 應用程式名稱]，然後選取 [訂用帳戶]、[資源群組] 和 [App Service 方案]。 

4. 選取 [建立] 。

5. 記下 [摘要] 區段中的 [網站 URL] 屬性。 此 URL 是您在本教學課程中稍後使用的「後端端點」。 

6. 選取 [發佈] 。

精靈完成後，它會將 ASP.NET Web 應用程式發佈至 Azure，然後在預設瀏覽器中開啟應用程式。  您的應用程式可在 Azure App Service 中檢視。

URL 會使用您稍早指定的 Web 應用程式名稱，其格式為 http://<app_name>.azurewebsites.net。

[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/publish-to-app-service.png
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG
