---
title: 如何使用適用於 Azure Mobile Apps 的 Apache Cordova 外掛程式
description: 如何使用適用於 Azure Mobile Apps 的 Apache Cordova 外掛程式
services: app-service\mobile
documentationcenter: javascript
author: conceptdev
manager: crdun
editor: ''
ms.assetid: a56a1ce4-de0c-4f3c-8763-66252c52aa59
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: article
ms.date: 10/30/2016
ms.author: crdun
ms.openlocfilehash: 6fb8be96c9793e96f1f7d2ad8e212d056d7e9ba5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38467881"
---
# <a name="how-to-use-apache-cordova-client-library-for-azure-mobile-apps"></a>如何使用適用於 Azure Mobile Apps 的 Apache Cordova 用戶端程式庫
[!INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南說明如何使用最新的 [適用於 Azure Mobile Apps 的 Apache Cordova 外掛程式]執行一般案例。 如果您是 Azure Mobile Apps 的新手，請先完成 [Azure Mobile Apps 快速啟動] 以建立後端、建立資料表及下載預先建置的 Apache Cordova 專案。 在本指南中，我們會著重於用戶端 Apache Cordova 外掛程式。

## <a name="supported-platforms"></a>支援的平台
此 SDK 可在 iOS、Android 和 Windows 裝置上支援 Apache Cordova v6.0.0 和更新版本。  平台支援如下︰

* Android API 19-24 (KitKat 至 Nougat)。
* iOS 8.0 版和更新版本。
* Windows Phone 8.1。
* 通用 Windows 平台。

## <a name="Setup"></a>設定和必要條件
本指南假設您已建立包含資料表的後端。 本指南假設資料表的結構描述與這些教學課程中的資料表相同。 本指南也假設您已將 Apache Cordova 外掛程式加入至程式碼。  如果您還沒有這樣做，您可以在命令列將 Apache Cordova 外掛程式新增至您的專案：

```
cordova plugin add cordova-plugin-ms-azure-mobile-apps
```

如需有關建立 [第一個 Apache Cordova 應用程式]的詳細資訊，請參閱其文件。

## <a name="ionic"></a>設定 Ionic v2 應用程式

若要適當地定 Ionic v2 專案，請先建立基本應用程式，並新增 Cordova 外掛程式：

```
ionic start projectName --v2
cd projectName
ionic plugin add cordova-plugin-ms-azure-mobile-apps
```

新增下列行到 `app.component.ts` 以建立用戶端專案：

```
declare var WindowsAzure: any;
var client = new WindowsAzure.MobileServiceClient("https://yoursite.azurewebsites.net");
```

您現在可以在瀏覽器中建置並執行專案：

```
ionic platform add browser
ionic run browser
```

Azure Mobile Apps Cordova 外掛程式同時支援 Ionic v1 與 v2 應用程式。  只有 Ionic v2 應用程式需要 `WindowsAzure` 物件的其他宣告。

[!INCLUDE [app-service-mobile-html-js-library.md](../../includes/app-service-mobile-html-js-library.md)]

## <a name="auth"></a>作法：驗證使用者
Azure App Service 支援使用各種外部識別提供者 (Facebook、Google、Microsoft 帳戶及 Twitter) 來驗證與授權應用程式使用者。 您可以在資料表上設定權限，以限制僅有通過驗證使用者可以存取特定操作。 您也可以在伺服器指令碼中，使用驗證的使用者的身分識別來實作授權規則。 如需詳細資訊，請參閱 [開始使用驗證] 教學課程。

在 Apache Cordova 應用程式中使用驗證時，下列 Cordova 外掛程式必須可用：

* [cordova-plugin-device]
* [cordova-plugin-inappbrowser]

支援兩個驗證流程：伺服器流程和用戶端流程。  由於伺服器流程採用提供者的 Web 驗證介面，因此所提供的驗證體驗也最為簡單。 用戶端流程依賴提供者專屬的裝置專用 SDK，可以與裝置特有的功能深入整合，例如單一登入。

[!INCLUDE [app-service-mobile-html-js-auth-library.md](../../includes/app-service-mobile-html-js-auth-library.md)]

### <a name="configure-external-redirect-urls"></a>做法︰設定行動 App Service 以使用外部重新導向 URL。
有數種類型的 Apache Cordova 應用程式會使用回送功能來處理 OAuth UI 流程。  驗證服務預設只知道如何使用您的服務，因此 localhost 上的 OAuth UI 流程會引發問題。  有問題的 OAuth UI 流程範例包括︰

* Ripple 模擬器。
* Ionic 的即時重新載入。
* 在本機執行行動後端
* 在不同於提供驗證的 Azure App Service 中執行行動後端。

請遵循下列指示來將您的本機設定加入組態︰

1. 登入 [Azure 入口網站]
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下行動應用程式的名稱。
3. 按一下 [工具] 
4. 按一下 [觀察] 功能表中的 [資源總管]，然後按一下 [前往]。  新的視窗或索引標籤隨即開啟。
5. 在左側導覽中，展開網站的 [config]、[authsettings] 節點。
6. 按一下 [編輯] 
7. 尋找「allowedExternalRedirectUrls」元素。  它可能會設定為 null 或值陣列。  將此值變更為下列值︰

         "allowedExternalRedirectUrls": [
             "http://localhost:3000",
             "https://localhost:3000"
         ],

    使用您服務的 URL 來取代 URL。  範例包括 "http://localhost:3000" (適用於 Node.js 範例服務) 或 "http://localhost:4400" (適用於 Ripple 服務)。  不過，這些 URL 都是範例 - 您的情況 (包括範例中提及的服務) 可能會有差異。
8. 按一下螢幕右上角的 [讀/寫]  按鈕。
9. 按一下綠色 [PUT]  按鈕。

此時即會儲存設定。  在設定完成儲存之前，請勿關閉瀏覽器視窗。
此外，請將這些回送 URL 新增至 App Service 的 CORS 設定：

1. 登入 [Azure 入口網站]
2. 選取 [所有資源] 或 [應用程式服務]，然後按一下行動應用程式的名稱。
3. [設定] 刀鋒視窗隨即自動開啟。  如果沒有，請按一下 [所有設定] 。
4. 按一下 API 功能表下方的 [CORS]  。
5. 輸入在提供的方塊中輸入您希望加入的 URL，然後按 Enter 鍵。
6. 視需要輸入其他的 URL。
7. 按一下 [儲存]  來儲存這些設定。

大約需要 10-15 秒的時間，才能使新的設定生效。

## <a name="register-for-push"></a>作法：註冊推播通知
安裝 [phonegap-plugin-push] 來處理推播通知。  在命令列中使用 `cordova plugin add` 命令，或在 Visual Studio 內透過 Git 外掛程式安裝程式，即可輕鬆新增此外掛程式。  以下在 Apache Cordova 應用程式中的程式碼會為您的裝置註冊推播通知：

```
var pushOptions = {
    android: {
        senderId: '<from-gcm-console>'
    },
    ios: {
        alert: true,
        badge: true,
        sound: true
    },
    windows: {
    }
};
pushHandler = PushNotification.init(pushOptions);

pushHandler.on('registration', function (data) {
    registrationId = data.registrationId;
    // For cross-platform, you can use the device plugin to determine the device
    // Best is to use device.platform
    var name = 'gcm'; // For android - default
    if (device.platform.toLowerCase() === 'ios')
        name = 'apns';
    if (device.platform.toLowerCase().substring(0, 3) === 'win')
        name = 'wns';
    client.push.register(name, registrationId);
});

pushHandler.on('notification', function (data) {
    // data is an object and is whatever is sent by the PNS - check the format
    // for your particular PNS
});

pushHandler.on('error', function (error) {
    // Handle errors
});
```

使用通知中樞 SDK 從伺服器傳送推播通知。  永遠不要直接從用戶端傳送推播通知。 這種方式會被用來對通知中樞或 PNS 觸發阻斷服務攻擊。  PNS 可能會因為這類攻擊而禁止您的流量。

## <a name="more-information"></a>詳細資訊

您可以在 [API 文件](http://azure.github.io/azure-mobile-apps-js-client/)中找到 API 詳細資訊。

<!-- URLs. -->
[Azure 入口網站]: https://portal.azure.com
[Azure Mobile Apps 快速啟動]: app-service-mobile-cordova-get-started.md
[開始使用驗證]: app-service-mobile-cordova-get-started-users.md
[Add authentication to your app]: app-service-mobile-cordova-get-started-users.md

[適用於 Azure Mobile Apps 的 Apache Cordova 外掛程式]: https://www.npmjs.com/package/cordova-plugin-ms-azure-mobile-apps
[第一個 Apache Cordova 應用程式]: http://cordova.apache.org/#getstarted
[phonegap-facebook-plugin]: https://github.com/wizcorp/phonegap-facebook-plugin
[phonegap-plugin-push]: https://www.npmjs.com/package/phonegap-plugin-push
[cordova-plugin-device]: https://www.npmjs.com/package/cordova-plugin-device
[cordova-plugin-inappbrowser]: https://www.npmjs.com/package/cordova-plugin-inappbrowser
[Query object documentation]: https://msdn.microsoft.com/library/azure/jj613353.aspx
