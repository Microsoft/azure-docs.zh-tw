---
title: 使用 Azure 通知中樞將通知推送至 Chrome 應用程式 | Microsoft Docs
description: 了解如何使用 Azure 通知中樞將推播通知傳送至 Chrome 應用程式。
services: notification-hubs
keywords: 行動推播通知,推播通知,推播通知,chrome 推播通知
documentationcenter: ''
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 75d4ff59-d04a-455f-bd44-0130a68e641f
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-chrome
ms.devlang: JavaScript
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 7bdc692104194bff4a25e6974ba72971af543cbf
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38698155"
---
# <a name="tutorial-push-notifications-to-chrome-apps-with-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至 Chrome 應用程式
[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

本教學課程將逐步引導您建立通知中樞，並使用 [Google 雲端通訊 (GCM)](https://developers.google.com/cloud-messaging/) 將推播通知傳送至範例 Google Chrome 應用程式。 Chrome 應用程式會在 Google Chrome 瀏覽器的內容中執行，並向通知中樞註冊。 

> [!NOTE]
> Chrome 應用程式的推播通知不是一般的瀏覽器中通知，而是瀏覽器擴充性模型所特有 (如需詳細資訊，請參閱 [Chrome 應用程式概觀] )。 Chrome 應用程式除了在桌面瀏覽器中執行外，也可透過 Apache Cordova 在行動裝置 (Android 和 iOS) 上執行。 如需詳細資訊，請參閱[行動裝置上的 Chrome 應用程式]。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * [啟用 Google Cloud Messaging](#register)
> * [設定您的通知中樞](#configure-hub)
> * [將您的 Chrome 應用程式連接到通知中樞](#connect-app)
> * [傳送推播通知給您的 Chrome 應用程式](#send)

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a id="register"></a>啟用 Google Cloud Messaging
1. 瀏覽至 [Google 雲端主控台](https://console.cloud.google.com/cloud-resource-manager)網站，然後使用您的 Google 帳戶認證登入
2. 選取工具列上的 [建立專案]。 

    ![建立專案按鈕](media/notification-hubs-chrome-get-started/create-project-button.png)
1. 提供適當的 [專案名稱]，然後按一下 [建立] 按鈕。
2. 選取工具列上的 [通知] 圖示 (鈴鐺)，然後選取 [建立專案] 訊息。 

    ![專案建立通知](media/notification-hubs-chrome-get-started/project-creation-notification.png)
1. 在 [專案] 頁面上，記下您建立之專案的 [專案編號]。 您會以專案編號作為 Chrome 應用程式中的 [GCM 寄件者識別碼] ，向 GCM 註冊。
   
    ![Google 雲端主控台 - 專案編號](media/notification-hubs-chrome-get-started/gcm-project-number.png)
3. 在儀表板中，選取 [移至 API 概觀]。 

    ![移至 API 概觀按鈕](media/notification-hubs-chrome-get-started/go-to-apis-overview-button.png)
1. 在 [API 和服務] 頁面中，選取 [啟用 API 和服務]。 

    ![啟用 API 和服務](media/notification-hubs-chrome-get-started/enable-apis-and-services.png)
1. 搜尋含有**雲端通訊**關鍵字的清單。 在已篩選的清單中選取 [Google 雲端通訊]。 

    ![Google 雲端通訊 API](media/notification-hubs-chrome-get-started/google-cloud-messaging-api.png)
1. 在 [Google 雲端通訊] 頁面上，選取 [啟用]。

    ![啟用 GCM](media/notification-hubs-chrome-get-started/enable-gcm.png)
1. 在 [API 和服務] 頁面中，切換至 [認證] 索引標籤。選取 [建立認證]，然後選取 [API 金鑰]。 

    ![建立 API 金鑰按鈕](media/notification-hubs-chrome-get-started/create-api-key-button.png)
1. 在 [已建立的 API 金鑰] 對話方塊方塊中選取 [複製] 按鈕，將金鑰複製到剪貼簿。 請將其儲存至某處。 您在下一節中會使用此值來設定通知中樞，讓它能夠將推播通知傳送至 GCM。

    ![API 頁面](media/notification-hubs-chrome-get-started/api-created-page.png)
12. 在 [API 金鑰] 清單中選取您的 API 金鑰。 在 [API 金鑰] 頁面中，選取 [IP 位址] \(Web 伺服器、cron 作業等\)，並輸入**0.0.0.0/0** 作為 IP 位址，然後按一下 [儲存]。 

    ![啟用 IP 位址](media/notification-hubs-chrome-get-started/enable-ip-addresses.png)

## <a id="configure-hub"></a>建立您的通知中樞
[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

6. 在 [通知設定] 類別中選取 [Google] \(GCM\)，輸入 GCM 專案的 [API 金鑰]，然後按一下 [儲存]。

      ![Azure 通知中樞 - Google (GCM)](media/notification-hubs-chrome-get-started/configure-gcm-api-key.png)

## <a id="connect-app"></a>將您的 Chrome 應用程式連接到通知中樞
現在已將您的通知中樞設定成使用 GCM，而且您已擁有可用來註冊應用程式以接收和傳送推播通知的連接字串。

### <a name="create-a-new-chrome-app"></a>建立新的 Chrome 應用程式
下列範例以 [Chrome 應用程式 GCM 範例]為基礎，並使用建議的方式建立 Chrome 應用程式。 這一節特別說明 Azure 通知中樞專用的步驟。 

> [!NOTE]
> 我們建議您從 [Chrome 應用程式通知中樞範例]下載此 Chrome 應用程式的原始碼。 

Chrome 應用程式是透過 JavaScript 建立的，您可以使用任何慣用的文字編輯器加以建立。 下圖顯示 Chrome 應用程式的外觀：

![Google Chrome 應用程式][15]

1. 建立資料夾，並將其命名為 `ChromePushApp`。 您不一定要取為這個名稱，但如果您將其命名為不同名稱，請務必要替換掉必要程式碼片段中的路徑。
2. 在第二個步驟建立的資料夾中，下載 [crypto-js 程式庫] 。 此程式庫資料夾包含兩個子資料夾：`components` 和 `rollups`。
3. 建立 `manifest.json` 檔案。 所有 Chrome 應用程式都會受到包含應用程式中繼資料的資訊清單檔案，以及最重要的是，使用者安裝應用程式時授與應用程式的所有權限所支援。
   
        {
          "name": "NH-GCM Notifications",
          "description": "Chrome platform app.",
          "manifest_version": 2,
          "version": "0.1",
          "app": {
            "background": {
              "scripts": ["background.js"]
            }
          },
          "permissions": ["gcm", "storage", "notifications", "https://*.servicebus.windows.net/*"],
          "icons": { "128": "gcm_128.png" }
        }
   
    請留意 `permissions` 元素，它會指定此 Chrome 應用程式能夠從 GCM 接收推播通知。 此範例應用程式也會使用圖示檔案 `gcm_128.png`，您會在取自原始 GCM 範例的重複使用原始碼中發現此檔案。 您可以用此圖示檔案來替換任何符合 [圖示準則](https://developer.chrome.com/apps/manifest/icons)的影像。
4. 建立名為 `background.js` 的檔案並加入下列程式碼：
   
        // Returns a new notification ID used in the notification.
        function getNotificationId() {
          var id = Math.floor(Math.random() * 9007199254740992) + 1;
          return id.toString();
        }
   
        function messageReceived(message) {
          // A message is an object with a data property that
          // consists of key-value pairs.
   
          // Concatenate all key-value pairs to form a display string.
          var messageString = "";
          for (var key in message.data) {
            if (messageString != "")
              messageString += ", "
            messageString += key + ":" + message.data[key];
          }
          console.log("Message received: " + messageString);
   
          // Pop up a notification to show the GCM message.
          chrome.notifications.create(getNotificationId(), {
            title: 'GCM Message',
            iconUrl: 'gcm_128.png',
            type: 'basic',
            message: messageString
          }, function() {});
        }
   
        var registerWindowCreated = false;
   
        function firstTimeRegistration() {
          chrome.storage.local.get("registered", function(result) {
   
            registerWindowCreated = true;
            chrome.app.window.create(
              "register.html",
              {  width: 520,
                 height: 500,
                 frame: 'chrome'
              },
              function(appWin) {}
            );
          });
        }
   
        // Set up a listener for GCM message event.
        chrome.gcm.onMessage.addListener(messageReceived);
   
        // Set up listeners to trigger the first-time registration.
        chrome.runtime.onInstalled.addListener(firstTimeRegistration);
        chrome.runtime.onStartup.addListener(firstTimeRegistration);
   
    這是快顯 Chrome 應用程式視窗 HTML (**register.html**) 的檔案，且該檔案也會定義處理常式 **messageReceived** 來處理內送的推播通知。
5. 建立名為 `register.html` 的檔案，此檔案會定義 Chrome 應用程式的 UI。 
   
   > [!NOTE]
   > 此範例使用 **CryptoJS v3.1.2**。 如果您下載了另一個版本的程式庫，請務必要正確替換 `src` 路徑中的版本。
   > 
   > 
   
        <html>
   
        <head>
        <title>GCM Registration</title>
        <script src="register.js"></script>
        <script src="CryptoJS v3.1.2/rollups/hmac-sha256.js"></script>
        <script src="CryptoJS v3.1.2/components/enc-base64-min.js"></script>
        </head>
   
        <body>
   
        Sender ID:<br/><input id="senderId" type="TEXT" size="20"><br/>
        <button id="registerWithGCM">Register with GCM</button>
        <br/>
        <br/>
        <br/>
   
        Notification Hub Name:<br/><input id="hubName" type="TEXT" style="width:400px"><br/><br/>
        Connection String:<br/><textarea id="connectionString" type="TEXT" style="width:400px;height:60px"></textarea>
   
        <br/>
   
        <button id="registerWithNH" disabled="true">Register with Azure Notification Hubs</button>
   
        <br/>
        <br/>
   
        <textarea id="console" type="TEXT" readonly style="width:500px;height:200px;background-color:#e5e5e5;padding:5px"></textarea>
   
        </body>
   
        </html>
6. 使用此步驟中程式碼，建立名為 `register.js` 的檔案。 此檔案會指定 `register.html`後面的指令碼。 Chrome 應用程式並不允許內嵌執行，因此您必須為 UI 建立個別的備份指令碼。
   
        var registrationId = "";
        var hubName        = "", connectionString = "";
        var originalUri    = "", targetUri = "", endpoint = "", sasKeyName = "", sasKeyValue = "", sasToken = "";
   
        window.onload = function() {
           document.getElementById("registerWithGCM").onclick = registerWithGCM;  
           document.getElementById("registerWithNH").onclick = registerWithNH;
           updateLog("You have not registered yet. Please provider sender ID and register with GCM and then with  Notification Hubs.");
        }
   
        function updateLog(status) {
          currentStatus = document.getElementById("console").innerHTML;
          if (currentStatus != "") {
            currentStatus = currentStatus + "\n\n";
          }
   
          document.getElementById("console").innerHTML = currentStatus  + status;
        }
   
        function registerWithGCM() {
          var senderId = document.getElementById("senderId").value.trim();
          chrome.gcm.register([senderId], registerCallback);
   
          // Prevent register button from being clicked again before the registration finishes.
          document.getElementById("registerWithGCM").disabled = true;
        }
   
        function registerCallback(regId) {
          registrationId = regId;
          document.getElementById("registerWithGCM").disabled = false;
   
          if (chrome.runtime.lastError) {
            // When the registration fails, handle the error and retry the
            // registration later.
            updateLog("Registration failed: " + chrome.runtime.lastError.message);
            return;
          }
   
          updateLog("Registration with GCM succeeded.");
          document.getElementById("registerWithNH").disabled = false;
   
          // Mark that the first-time registration is done.
          chrome.storage.local.set({registered: true});
        }
   
        function registerWithNH() {
          hubName = document.getElementById("hubName").value.trim();
          connectionString = document.getElementById("connectionString").value.trim();
          splitConnectionString();
          generateSaSToken();
          sendNHRegistrationRequest();
        }
   
        // From http://msdn.microsoft.com/library/dn495627.aspx
        function splitConnectionString()
        {
          var parts = connectionString.split(';');
          if (parts.length != 3)
          throw "Error parsing connection string";
   
          parts.forEach(function(part) {
            if (part.indexOf('Endpoint') == 0) {
            endpoint = 'https' + part.substring(11);
            } else if (part.indexOf('SharedAccessKeyName') == 0) {
            sasKeyName = part.substring(20);
            } else if (part.indexOf('SharedAccessKey') == 0) {
            sasKeyValue = part.substring(16);
            }
          });
   
          originalUri = endpoint + hubName;
        }
   
        function generateSaSToken()
        {
          targetUri = encodeURIComponent(originalUri.toLowerCase()).toLowerCase();
          var expiresInMins = 10; // 10 minute expiration
   
          // Set expiration in seconds.
          var expireOnDate = new Date();
          expireOnDate.setMinutes(expireOnDate.getMinutes() + expiresInMins);
          var expires = Date.UTC(expireOnDate.getUTCFullYear(), expireOnDate
            .getUTCMonth(), expireOnDate.getUTCDate(), expireOnDate
            .getUTCHours(), expireOnDate.getUTCMinutes(), expireOnDate
            .getUTCSeconds()) / 1000;
          var tosign = targetUri + '\n' + expires;
   
          // Using CryptoJS.
          var signature = CryptoJS.HmacSHA256(tosign, sasKeyValue);
          var base64signature = signature.toString(CryptoJS.enc.Base64);
          var base64UriEncoded = encodeURIComponent(base64signature);
   
          // Construct authorization string.
          sasToken = "SharedAccessSignature sr=" + targetUri + "&sig="
                          + base64UriEncoded + "&se=" + expires + "&skn=" + sasKeyName;
        }
   
        function sendNHRegistrationRequest()
        {
          var registrationPayload =
          "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
          "<entry xmlns=\"http://www.w3.org/2005/Atom\">" +
              "<content type=\"application/xml\">" +
                  "<GcmRegistrationDescription xmlns:i=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns=\"http://schemas.microsoft.com/netservices/2010/10/servicebus/connect\">" +
                      "<GcmRegistrationId>{GCMRegistrationId}</GcmRegistrationId>" +
                  "</GcmRegistrationDescription>" +
              "</content>" +
          "</entry>";
   
          // Update the payload with the registration ID obtained earlier.
          registrationPayload = registrationPayload.replace("{GCMRegistrationId}", registrationId);
   
          var url = originalUri + "/registrations/?api-version=2014-09";
          var client = new XMLHttpRequest();
   
          client.onload = function () {
            if (client.readyState == 4) {
              if (client.status == 200) {
                updateLog("Notification Hub Registration succesful!");
                updateLog(client.responseText);
              } else {
                updateLog("Notification Hub Registration did not succeed!");
                updateLog("HTTP Status: " + client.status + " : " + client.statusText);
                updateLog("HTTP Response: " + "\n" + client.responseText);
              }
            }
          };
   
          client.onerror = function () {
                updateLog("ERROR - Notification Hub Registration did not succeed!");
          }
   
          client.open("POST", url, true);
          client.setRequestHeader("Content-Type", "application/atom+xml;type=entry;charset=utf-8");
          client.setRequestHeader("Authorization", sasToken);
          client.setRequestHeader("x-ms-version", "2014-09");
   
          try {
              client.send(registrationPayload);
          }
          catch(err) {
              updateLog(err.message);
          }
        }
   
    此指令碼具有下列重要參數：
   
   * **window.onload** 會在 UI 上定義兩個按鈕的按鈕點擊事件。 第一個按鈕點擊事件處理常式會向 GCM 註冊，另一個則使用向 GCM 註冊後所傳回的註冊識別碼來向 Azure 通知中樞註冊。
   * **updateLog** 是允許使用程式碼記錄資訊的函式。 
   * **registerWithGCM** 是第一個按鈕點擊處理常式，可向 GCM 進行 `chrome.gcm.register` 呼叫，以註冊目前的 Chrome 應用程式執行個體。
   * **registerCallback** 是回呼函數，會在 GCM 註冊呼叫傳回時受到呼叫。
   * **registerWithNH** 是第二個按鈕點擊處理常式，會向通知中樞進行註冊。 它會取得使用者已指定的 `hubName` 和 `connectionString`，並製作通知中樞註冊 REST API 呼叫。
   * **splitConnectionString** 和 **generateSaSToken** 是代表 SaS 權杖建立程序之 JavaScript 實作的協助程式，其必須用於所有 REST API 呼叫中。 如需詳細資訊，請參閱 [一般概念](http://msdn.microsoft.com/library/dn495627.aspx)。
   * **sendNHRegistrationRequest** 是對 Azure 通知中樞發出 HTTP REST 呼叫的函式。
   * **registrationPayload** 會定義註冊 XML 裝載。 如需詳細資訊，請參閱 [建立註冊 NH REST API]。 請以接收自 GCM 的值來更新其中的註冊識別碼。
   * **client** 是應用程式用來發出 HTTP POST 要求的 **XMLHttpRequest** 執行個體。 使用 `sasToken` 更新 `Authorization` 標頭。 成功完成此呼叫後，即會向 Azure 通知中樞註冊此 Chrome 應用程式執行個體。

        此專案的整體資料夾結構應與下列結構類似︰![Google Chrome 應用程式 - 資料夾結構][21]

### <a name="set-up-and-test-your-chrome-app"></a>設定和測試 Chrome 應用程式
1. 開啟 Chrome 瀏覽器。 開啟 [Chrome 擴充功能]，並啟用 [開發人員模式]。
   
    ![Google Chrome - 啟用開發人員模式][16]
2. 按一下 [載入未封裝的擴充功能]，並瀏覽至您在其中建立檔案的資料夾。 您也可以選擇性地使用 **Chrome Apps & Extensions Developer Tool**。 此工具本身為 Chrome 應用程式 (從 Chrome 線上應用程式商店進行安裝)，且提供 Chrome 應用程式開發進階偵錯功能。
   
    ![Google Chrome - 載入已解壓縮的擴充功能][17]
3. 如果該 Chrome 應用程式在建立時未發生任何錯誤，您會看見該應用程式顯示。
   
    ![Google Chrome - Chrome 應用程式顯示][18]
4. 輸入您先前從 **Google 雲端主控台** 取得的 **[專案編號]**，做為寄件者識別碼，然後按一下 **[向 GCM 註冊]**。 您必須看見 **Registration with GCM succeeded.** 訊息。
   
    ![Google Chrome - Chrome 應用程式自訂][19]
5. 輸入您先前從 Azure 傳統入口網站取得的 [通知中樞名稱] 和 [DefaultListenSharedAccessSignature]，然後按一下 [向 Azure 通知中樞註冊]。 您必須看見 **Registration with GCM succeeded.** 訊息 和註冊回應的詳細資料，其中包含 Azure 通知中樞註冊識別碼。
   
    ![Google Chrome - 指定通知中樞詳細資料][20]  

## <a name="send"></a>傳送通知給您的 Chrome 應用程式
為了進行測試，請使用 .NET 主控台應用程式傳送 Chrome 推播通知。 

> [!NOTE]
> 您可以透過公用 <a href="http://msdn.microsoft.com/library/windowsazure/dn223264.aspx">REST 介面</a>，使用通知中樞從任何後端傳送推播通知。 如需其他跨平台範例，請查看[說明文件入口網站](https://azure.microsoft.com/documentation/services/notification-hubs/)。
> 
> 

1. 在 Visual Studio 的 [檔案] 功能表中，選取 [新增]，然後選取 [專案]。 在 [Visual C#] 下方，按一下 [Windows] 和 [主控台應用程式]，然後按一下 [確定]。  此步驟會建立新的主控台應用程式專案。
2. 在 [工具] 功能表中按一下 [NuGet 套件管理員]，然後按一下 [套件管理員主控台]。 您會在視窗底部看見 [套件管理員主控台]。
3. 在主控台視窗中，執行下列命令：
   
        Install-Package Microsoft.Azure.NotificationHubs
   
   具有 <a href="http://nuget.org/packages/WindowsAzure.ServiceBus/">WindowsAzure.ServiceBus NuGet 套件</a>的 Azure 服務匯流排 SDK 的參考會自動新增至專案。
4. 開啟 `Program.cs` 並新增下列 `using` 陳述式：
   
        using Microsoft.Azure.NotificationHubs;
5. 在 `Program` 類別中，新增下列方法：
   
        private static async void SendNotificationAsync()
        {
            NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString("<connection string with full access>", "<hub name>");
            String message = "{\"data\":{\"message\":\"Hello Chrome from Azure Notification Hubs\"}}";
            await hub.SendGcmNativeNotificationAsync(message);
        }
   
    請務必將 `<hub name>` 預留位置取代為在[入口網站](https://portal.azure.com)中出現於您的 [通知中樞] 頁面上的通知中樞名稱。 此外，請將連接字串預留位置取代為您在＜通知中樞組態＞一節中取得，且名為 `DefaultFullSharedAccessSignature` 的連接字串。
   
    > [!NOTE]
    > 請確定您使用的連接字串具有 [完整] 存取權，而非 [接聽] 存取權。 [接聽]  存取權連接字串未授與傳送推播通知的權限。
6. 在 `Main` 方法中新增下列呼叫︰
   
         SendNotificationAsync();
         Console.ReadLine();
7. 確定 Chrome 正在執行，並執行主控台應用程式。
8. 您應該會在桌面上看見下列通知快顯視窗。
   
    ![Google Chrome - 通知][13]
9. 當 Chrome 執行時，您也可以使用工作列 (在 Windows 中) 內的 [Chrome 通知] 視窗來查看所有的通知。
   
    ![Google Chrome - 通知清單](./media/notification-hubs-chrome-get-started/ChromeNotificationWindow.png)

> [!NOTE]
> 您不需要執行 Chrome 應用程式，也不需要在瀏覽器中開啟 (但 Chrome 瀏覽器本身必須為執行狀態)。 您也可以在 [Chrome 通知] 視窗中整合檢視您所有的通知。
> 
> 

## <a name="next-steps"> </a>後續步驟
在本教學課程中，您已將廣播通知傳送至所有向後端註冊的用戶端。 若想了解如何將通知推送至特定裝置，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[將通知推送至特定裝置](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)

<!-- Images. -->
[3]: ./media/notification-hubs-chrome-get-started/EnableGCM.png
[4]: ./media/notification-hubs-chrome-get-started/CreateServerKey.png
[5]: ./media/notification-hubs-chrome-get-started/ServerKey.png
[6]: ./media/notification-hubs-chrome-get-started/CreateNH.png
[7]: ./media/notification-hubs-chrome-get-started/NHNamespace.png
[8]: ./media/notification-hubs-chrome-get-started/NamespaceConfigure.png
[9]: ./media/notification-hubs-chrome-get-started/NHConfigure.png
[10]: ./media/notification-hubs-chrome-get-started/NHConfigureGCM.png
[11]: ./media/notification-hubs-chrome-get-started/NHDashboard.png
[12]: ./media/notification-hubs-chrome-get-started/NHConnString.png
[13]: ./media/notification-hubs-chrome-get-started/ChromeNotification.png
[14]: ./media/notification-hubs-chrome-get-started/ChromeNotificationWindow.png
[15]: ./media/notification-hubs-chrome-get-started/ChromeApp.png
[16]: ./media/notification-hubs-chrome-get-started/ChromeExtensions.png
[17]: ./media/notification-hubs-chrome-get-started/ChromeLoadExtension.png
[18]: ./media/notification-hubs-chrome-get-started/ChromeAppLoaded.png
[19]: ./media/notification-hubs-chrome-get-started/ChromeAppGCM.png
[20]: ./media/notification-hubs-chrome-get-started/ChromeAppNH.png
[21]: ./media/notification-hubs-chrome-get-started/FinalFolderView.png

<!-- URLs. -->
[Chrome 應用程式通知中樞範例]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToChromeApps
[Notification Hubs Overview]: notification-hubs-push-notification-overview.md
[Chrome 應用程式概觀]: https://developer.chrome.com/apps/about_apps
[Chrome 應用程式 GCM 範例]: https://github.com/GoogleChrome/chrome-app-samples/tree/master/samples/gcm-notifications
[Installable Web Apps]: https://developers.google.com/chrome/apps/docs/
[行動裝置上的 Chrome 應用程式]: https://developer.chrome.com/apps/chrome_apps_on_mobile
[建立註冊 NH REST API]: http://msdn.microsoft.com/library/azure/dn223265.aspx
[crypto-js 程式庫]: http://code.google.com/p/crypto-js/
[GCM with Chrome Apps]: https://developer.chrome.com/apps/cloudMessaging
[Google Cloud Messaging for Chrome]: https://developer.chrome.com/apps/cloudMessagingV1
[Azure Notification Hubs Notify Users]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Azure Notification Hubs breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
