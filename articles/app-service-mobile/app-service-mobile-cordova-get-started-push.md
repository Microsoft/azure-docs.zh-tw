---
title: 使用 Azure App Service 的 Mobile Apps 功能新增推播通知至 Apache Cordova 應用程式 | Microsoft Docs
description: 了解如何使用 Mobile Apps 將推播通知傳送至 Apache Cordova 應用程式。
services: app-service\mobile
documentationcenter: javascript
manager: crdun
editor: ''
author: conceptdev
ms.assetid: 92c596a9-875c-4840-b0e1-69198817576f
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: article
ms.date: 10/30/2016
ms.author: crdun
ms.openlocfilehash: cb36c6fea20f5976f159780f3482d9aae669f951
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42817951"
---
# <a name="add-push-notifications-to-your-apache-cordova-app"></a>新增推播通知至您的 Apache Cordova 應用程式

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## <a name="overview"></a>概觀

在本教學課程中，您會將推播通知新增至 [Apache Cordova 快速入門][5]專案，以便在每次插入一筆記錄時，將推播通知傳送至裝置。

如果您未使用下載的快速入門伺服器專案，則需要推播通知擴充套件。 如需詳細資訊，請參閱[使用適用於 Mobile Apps 的 .NET 後端伺服器 SDK][1]。

## <a name="prerequisites"></a>必要條件

本教學課程假設您擁有以 Visual Studio 2015 開發的 Apache Cordova 應用程式。 此裝置應在 Google Android Emulator、Android 裝置、Windows 裝置或 iOS 裝置上執行。

若要完成本教學課程，您需要：

* 具有 [Visual Studio Community 2015][2] 或更新版本的電腦
* [Visual Studio Tools for Apache Cordova][4]
* [使用中的 Azure 帳戶][3]
* 已完成的 [Apache Cordova 快速入門][5]專案
* (Android) 具有已驗證電子郵件地址的 [Google 帳戶][6]
* (iOS) [Apple Developer Program 成員資格][7]和 iOS 裝置 (iOS 模擬器不支援推播通知)
* (Windows) [Microsoft 市集開發人員帳戶][8]和 Windows 10 裝置

## <a name="configure-hub"></a>設定通知中樞

[!INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

[觀看示範本節步驟的影片][9]。

## <a name="update-the-server-project"></a>更新伺服器專案

[!INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

## <a name="add-push-to-app"></a>修改您的 Cordova 應用程式

若要確保您的 Apache Cordova 應用程式專案能夠處理推播通知，請安裝 Cordova 推播外掛程式，還有平台特定的所有推播服務。

#### <a name="update-the-cordova-version-in-your-project"></a>更新您的專案中的 Cordova 版本。

如果您的專案使用 6.1.1 版以前的 Apache Cordova，請更新用戶端專案。 若要更新專案，請採取下列步驟：

* 以滑鼠右鍵按一下 `config.xml` 開啟設定設計工具。
* 選取 [平台] 索引標籤。
* 在 [Cordova CLI] 文字方塊中選取 [6.1.1]。 
* 若要更新專案，選取 [建置]，然後選取 [建置解決方案]。

#### <a name="install-the-push-plugin"></a>安裝推播外掛程式

Apache Cordova 應用程式原本就不會處理裝置或網路功能。  這些功能是由 [npm][10] 或 GitHub 上發佈的外掛程式所提供。 `phonegap-plugin-push` 外掛程式會處理網路推播通知。

您可透過下列其中一種方式安裝推播外掛程式：

**從命令提示：**

執行以下命令：

    cordova plugin add phonegap-plugin-push

**從 Visual Studio 內：**

1. 在方案總管中開啟 `config.xml` 檔案。 接著，選取 [外掛程式] > [自訂]。 然後選取 [Git] 作為安裝來源。

2. 輸入 `https://github.com/phonegap/phonegap-plugin-push` 作為來源。

    ![在方案總管中開啟 config.xml 檔案][img1]

3. 選取安裝來源旁邊的箭頭。

4. 在 **SENDER_ID** 中，如果您已經有 Google 開發人員主控台專案的數值專案識別碼，您可以在此將它新增。 否則，請輸入預留位置值，例如 777777。 如果您是以 Android 為目標，您可以稍後在 config.xml 檔案中更新此值。

    >[!NOTE]
    >自 2.0.0 版開始，google-services.json 必須安裝在專案的根資料夾中，以設定寄件者識別碼。 如需詳細資訊，請參閱[安裝文件。](https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/INSTALLATION.md)

5. 選取 [新增] 。

推撥外掛程式現已安裝。

#### <a name="install-the-device-plugin"></a>安裝裝置外掛程式

遵循您安裝推播外掛程式時的相同程序。 從核心外掛程式清單中新增裝置外掛程式。 (選取 [外掛程式] > [核心] 以尋找它。)您需要此外掛程式才能取得平台名稱。

#### <a name="register-your-device-when-the-application-starts"></a>在應用程式啟動時註冊您的裝置 

最初，我們包含一些適用於 Android 的最少程式碼。 之後您可以將應用程式修改為在 iOS 或 Windows 10 上執行。

1. 在登入程序的回呼期間將呼叫新增至 **registerForPushNotifications**。 或者，您可以在 **onDeviceReady** 方法底部新增它：

    ```javascript
    // Log in to the service.
    client.login('google')
        .then(function () {
            // Create a table reference.
            todoItemTable = client.getTable('todoitem');

            // Refresh the todoItems.
            refreshDisplay();

            // Wire up the UI Event Handler for the Add Item.
            $('#add-item').submit(addItemHandler);
            $('#refresh').on('click', refreshDisplay);

                // Added to register for push notifications.
            registerForPushNotifications();

        }, handleError);
    ```

    這個範例示範在成功驗證之後呼叫 **registerForPushNotifications**。 您可以依需要經常呼叫 `registerForPushNotifications()`。

2. 加入新的 **registerForPushNotifications** 方法，如下所示︰

    ```javascript
    // Register for push notifications. Requires that phonegap-plugin-push be installed.
    var pushRegistration = null;
    function registerForPushNotifications() {
        pushRegistration = PushNotification.init({
            android: { senderID: 'Your_Project_ID' },
            ios: { alert: 'true', badge: 'true', sound: 'true' },
            wns: {}
        });

    // Handle the registration event.
    pushRegistration.on('registration', function (data) {
        // Get the native platform of the device.
        var platform = device.platform;
        // Get the handle returned during registration.
        var handle = data.registrationId;
        // Set the device-specific message template.
        if (platform == 'android' || platform == 'Android') {
            // Register for GCM notifications.
            client.push.register('gcm', handle, {
                mytemplate: { body: { data: { message: "{$(messageParam)}" } } }
            });
        } else if (device.platform === 'iOS') {
            // Register for notifications.
            client.push.register('apns', handle, {
                mytemplate: { body: { aps: { alert: "{$(messageParam)}" } } }
            });
        } else if (device.platform === 'windows') {
            // Register for WNS notifications.
            client.push.register('wns', handle, {
                myTemplate: {
                    body: '<toast><visual><binding template="ToastText01"><text id="1">$(messageParam)</text></binding></visual></toast>',
                    headers: { 'X-WNS-Type': 'wns/toast' } }
            });
        }
    });

    pushRegistration.on('notification', function (data, d2) {
        alert('Push Received: ' + data.message);
    });

    pushRegistration.on('error', handleError);
    }
    ```
3. (Android) 在上述程式碼中，從 [Google Developer Console][18] 使用應用程式的數字專案識別碼取代 `Your_Project_ID`。

## <a name="optional-configure-and-run-the-app-on-android"></a>(選擇性) 在 Android 上設定和執行應用程式

完成本節可以為 Android 啟用推播通知。

#### <a name="enable-gcm"></a>啟用 Firebase 雲端傳訊

因為一開始是以 Google Android 平台為目標，所以您必須啟用 Firebase 雲端通訊。

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

#### <a name="configure-backend"></a>設定行動應用程式後端以使用 FCM 傳送推送要求

[!INCLUDE [app-service-mobile-android-configure-push](../../includes/app-service-mobile-android-configure-push.md)]

#### <a name="configure-your-cordova-app-for-android"></a>針對 Android 設定您的 Cordova 應用程式

在 Cordova 應用程式中，開啟 **config.xml**。 然後從 [Google Developer Console][18] 使用應用程式的數字專案識別碼取代 `Your_Project_ID`。

```xml
<plugin name="phonegap-plugin-push" version="1.7.1" src="https://github.com/phonegap/phonegap-plugin-push.git">
    <variable name="SENDER_ID" value="Your_Project_ID" />
</plugin>
```

開啟 **index.js**。 然後更新程式碼以使用您的數值專案識別碼。

```javascript
pushRegistration = PushNotification.init({
    android: { senderID: 'Your_Project_ID' },
    ios: { alert: 'true', badge: 'true', sound: 'true' },
    wns: {}
});
```

#### <a name="configure-device"></a>設定 Android 裝置進行 USB 偵錯

在您可以將應用程式部署到您的 Android 裝置之前，您需要啟用 USB 偵錯。 在您的 Android 手機上執行下列步驟：

1. 移至 [設定] > [關於手機]。 然後點選 [組建編號]，直到啟用開發人員模式為止 (大約七次)。
2. 回到 [設定] > [開發人員選項]，啟用 [USB 偵錯]。 然後使用 USB 纜線將您的 Android 手機連接到開發電腦。

我們測試時使用的是執行 Android 6.0 (Marshmallow) 的 Google Nexus 5 X 裝置。 不過，這些技術在任何現代化 Android 版本中都是相同的。

#### <a name="install-google-play-services"></a>安裝 Google Play 服務

推播外掛程式仰賴 Android Google Play 服務來進行推播通知。

1. 在 Visual Studio 中，選取 [工具] > [Android] > [Android SDK 管理員]。 然後展開 [Extras] 資料夾。 請檢查適當的方塊，確定已安裝所有下列的 SDK：

   * Android 2.3 或更新版本
   * Google Repository 版本 27 或更高版本
   * Google Play 服務 9.0.2 或更高版本

2. 選取 [安裝套件]。 然後等待安裝完成。

[phonegap-plugin-push 安裝文件][19]中列出目前必要的程式庫。

#### <a name="test-push-notifications-in-the-app-on-android"></a>在 Android 上的應用程式中測試推播通知

您現在可以執行應用程式，在 TodoItem 資料表中插入項目，以測試推播通知。 只要是使用相同的後端，您可以從相同的裝置或第二部裝置來測試。 以下列方法之一在 Android 平台上測試 Cordova 應用程式︰

* *在實體裝置上︰* 使用 USB 纜線將 Android 裝置連接至開發電腦。  請選取 [裝置]，不要選取 [Google Android 模擬器]。 Visual Studio 會將應用程式部署至裝置，並執行應用程式。 您接著可以在裝置上與應用程式互動。

  [Mobizen][20] 等螢幕畫面分享應用程式可協助您開發 Android 應用程式。 Mobizen 會將您的 Android 畫面投射至電腦上的網頁瀏覽器。

* *在 Android 模擬器上︰* 使用模擬器時，還需要其他設定步驟。

    請確定您是部署至已將 Google API 設為目標的虛擬裝置，如 Android 虛擬裝置 (AVD) 管理員中所示。

    ![Android 虛擬裝置管理員](./media/app-service-mobile-cordova-get-started-push/google-apis-avd-settings.png)

    如果您想要使用更快速的 x86 模擬器，請[安裝 HAXM 驅動程式][11]，並設定模擬器來使用它。

    選取 [應用程式] > [設定] > [新增帳戶] 將 Google 帳戶加入 Android 裝置。 然後遵循提示。

    ![將 Google 帳戶新增至 Android 裝置](./media/app-service-mobile-cordova-get-started-push/add-google-account.png)

    按照先前的方法執行 todolist 應用程式，然後插入新的 todo 項目。 這次，通知圖示會顯示在通知區域中。 您可以開啟通知抽屜來檢視通知的完整內容。

    ![檢視通知](./media/app-service-mobile-cordova-get-started-push/android-notifications.png)

## <a name="optional-configure-and-run-on-ios"></a>(選擇性) 在 iOS 上設定和執行

本節適用於在 iOS 裝置上執行 Cordova 專案。 如果未使用 iOS 裝置，可以略過這一節。

#### <a name="install-and-run-the-ios-remote-build-agent-on-a-mac-or-cloud-service"></a>在 Mac 或雲端服務上安裝及執行 iOS 遠端組建代理程式

您必須先進行 [iOS 安裝指南][12]中的步驟來安裝和執行遠端組建代理程式，才可以使用 Visual Studio 在 iOS 上執行 Cordova 應用程式。

確定您可以建置適用於 iOS 的應用程式。 必須執行安裝指南中的步驟才能從 Visual Studio 針對 iOS 建置應用程式。 如果您沒有 Mac，您可以在 MacInCloud 之類的服務上使用遠端組建代理程式來針對 iOS 建置。 如需詳細資訊，請參閱[在雲端中執行 iOS 應用程式][21]。

> [!NOTE]
> 必須有 Xcode 7 或更新版本，才能在 iOS 上使用推播外掛程式。

#### <a name="find-the-id-to-use-as-your-app-id"></a>尋找要做為應用程式識別碼的識別碼

在針對推播通知註冊您的應用程式之前，在 Cordova 應用程式中開啟 config.xml，在 widget 元素中尋找 `id` 屬性值，並加以複製以供稍後使用。 在下列 XML 中，識別碼為 `io.cordova.myapp7777777`。

```xml
<widget defaultlocale="en-US" id="io.cordova.myapp7777777"
    version="1.0.0" windows-packageVersion="1.1.0.0" xmlns="http://www.w3.org/ns/widgets"
    xmlns:cdv="http://cordova.apache.org/ns/1.0" xmlns:vs="http://schemas.microsoft.com/appx/2014/htmlapps">
```

稍後，當您在 Apple 開發人員入口網站上建立應用程式識別碼時，請使用這個識別碼。 如果您在開發人員入口網站上建立不同的應用程式識別碼，在本教學課程稍後，您必須採取一些額外的步驟。 Widget 元素中的識別碼必須符合開發人員入口網站上的應用程式識別碼。

#### <a name="register-the-app-for-push-notifications-on-apples-developer-portal"></a>在 Apple 的開發人員入口網站註冊應用程式以取得推播通知

[!INCLUDE [Enable Apple Push Notifications](../../includes/enable-apple-push-notifications.md)]

[觀看示範類似步驟的影片](https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-5-Set-up-apns-for-push)

#### <a name="configure-azure-to-send-push-notifications"></a>設定 Azure 來傳送推播通知

[!INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

#### <a name="verify-that-your-app-id-matches-your-cordova-app"></a>確認您的應用程式識別碼符合 Cordova 應用程式

如果您在 Apple 開發人員帳戶中建立的應用程式識別碼已符合 config.xml 檔案中 widget 元素的識別碼，即可略過此步驟。 不過，如果識別碼不相符，請採取下列步驟︰

1. 從您的專案中刪除 platforms 資料夾。
2. 從您的專案中刪除 plugins 資料夾。
3. 從您的專案中刪除 node_modules 資料夾。
4. 在 config.xml 檔案中更新 widget 元素的 id 屬性，以使用您在 Apple 開發人員帳戶中建立的應用程式識別碼。
5. 重建您的專案。

##### <a name="test-push-notifications-in-your-ios-app"></a>在 iOS 應用程式中測試推播通知

1. 在 Visual Studio 中，請確定已選取 **iOS** 作為部署目標。 然後選取 [裝置] 以在連接的 iOS 裝置上執行推播通知。

    您可以在連接至 PC 與 iTunes 的 iOS 裝置上執行推播通知。 iOS 模擬器不支援推播通知。

2. 在 Visual Studio 中選取 [執行] 按鈕或 **F5** 以建置專案，並在 iOS 裝置上啟動應用程式。 然後，選取 [確定] 以接收推播通知。

   > [!NOTE]
   > 應用程式在第一次執行期間會要求您確認推播通知。

3. 在應用程式中輸入一項工作，然後選取加號 **(+)** 圖示。
4. 驗證已收到通知。 然後選取 [確定] 關閉通知。

## <a name="optional-configure-and-run-on-windows"></a>(選擇性) 在 Windows 上設定和執行

本節說明如何在 Windows 10 裝置 (Windows 10 支援 PhoneGap 推播外掛程式) 上執行 Apache Cordova 應用程式專案。 如果未使用 Windows 裝置，可以略過這一節。

#### <a name="register-your-windows-app-for-push-notifications-with-wns"></a>向 WNS 註冊 Windows 應用程式以使用推播通知

若要使用 Visual Studio 中的 [存放區] 選項，請從 [解決方案平台] 清單中選取 Windows 目標，例如 **Windows-x64** 或 **Windows-x86**。 (避免 **Windows-AnyCPU** 使用推播通知。)

[!INCLUDE [app-service-mobile-register-wns](../../includes/app-service-mobile-register-wns.md)]

[觀看示範類似步驟的影片][13]

#### <a name="configure-the-notification-hub-for-wns"></a>設定 WNS 的通知中樞

[!INCLUDE [app-service-mobile-configure-wns](../../includes/app-service-mobile-configure-wns.md)]

#### <a name="configure-your-cordova-app-to-support-windows-push-notifications"></a>設定 Cordova 應用程式以支援 Windows 推播通知

以滑鼠右鍵按一下 **config.xml** 開啟設定設計工具。 然後選取 [檢視設計工具]。 接下來，選取 [Windows] 索引標籤，然後在 [Windows 目標版本] 底下選取 [Windows 10]。

若要在您的預設 (偵錯) 組建中支援推播通知，請開啟 **build.json** 檔案。 然後將 "release" 設定複製到您的偵錯設定。

```json
"windows": {
    "release": {
        "packageCertificateKeyFile": "res\\native\\windows\\CordovaApp.pfx",
        "publisherId": "CN=yourpublisherID"
    }
}
```

更新之後，**build.json** 檔案應該包含下列程式碼︰

```json
"windows": {
    "release": {
        "packageCertificateKeyFile": "res\\native\\windows\\CordovaApp.pfx",
        "publisherId": "CN=yourpublisherID"
        },
    "debug": {
        "packageCertificateKeyFile": "res\\native\\windows\\CordovaApp.pfx",
        "publisherId": "CN=yourpublisherID"
        }
    }
```

建置應用程式並確認沒有錯誤。 您的用戶端應用程式現在應註冊來自 Mobile Apps 後端的通知。 針對方案中每個 Windows 專案重複操作這一節。

#### <a name="test-push-notifications-in-your-windows-app"></a>在 Windows 應用程式中測試推播通知

在 Visual Studio 中，確定已選取 Windows 平台做為部署目標，例如 **Windows-x64** 或 **Windows-x86**。 若要在裝載 Visual Studio 的 Windows 10 電腦上執行應用程式，請選擇 [本機電腦]。

1. 選取 [執行] 按鈕，以建置專案並啟動應用程式。

2. 在應用程式中輸入新 todoitem 的名稱，然後選取加號 **(+)** 圖示來加入它。

確認在加入項目時收到通知。

## <a name="next-steps"></a>後續步驟

* 請參閱[通知中樞][17]，以了解推播通知的相關資訊。
* 繼續教學課程，在 Apache Cordova 應用程式中[新增驗證][14] (如果尚未這麼做)。

了解如何使用下列 SDK：

* [Apache Cordova SDK][15]
* [ASP.NET Server SDK][1]
* [Node.js Server SDK][16]

<!-- Images -->
[img1]: ./media/app-service-mobile-cordova-get-started-push/add-push-plugin.png

<!-- URLs -->
[1]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[2]: http://www.visualstudio.com/
[3]: https://azure.microsoft.com/pricing/free-trial/
[4]: https://www.visualstudio.com/en-us/features/cordova-vs.aspx
[5]: app-service-mobile-cordova-get-started.md
[6]: http://go.microsoft.com/fwlink/p/?LinkId=268302
[7]: https://developer.apple.com/programs/
[8]: https://developer.microsoft.com/en-us/store/register
[9]: https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-3-Create-azure-notification-hub
[10]: https://www.npmjs.com/
[11]: https://taco.visualstudio.com/en-us/docs/run-app-apache/#HAXM
[12]: http://taco.visualstudio.com/en-us/docs/ios-guide/
[13]: https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-6-Set-up-wns-for-push
[14]: app-service-mobile-cordova-get-started-users.md
[15]: app-service-mobile-cordova-how-to-use-client-library.md
[16]: app-service-mobile-node-backend-how-to-use-server-sdk.md
[17]: ../notification-hubs/notification-hubs-push-notification-overview.md
[18]: https://console.developers.google.com/home/dashboard
[19]: https://github.com/phonegap/phonegap-plugin-push/blob/master/docs/INSTALLATION.md
[20]: https://www.mobizen.com/
[21]: http://taco.visualstudio.com/en-us/docs/build_ios_cloud/
