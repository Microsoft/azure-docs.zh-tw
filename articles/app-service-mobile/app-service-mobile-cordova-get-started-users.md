---
title: 使用 Mobile Apps 在 Apache Cordova 上新增驗證 | Microsoft Docs
description: 了解如何在 Azure App Service 中使用 Mobile Apps，透過眾多識別提供者驗證 Apache Cordova 應用程式使用者，包括 Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: javascript
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 10dd6dc9-ddf5-423d-8205-00ad74929f0d
ms.service: app-service-mobile
ms.workload: na
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: article
ms.date: 10/30/2016
ms.author: crdun
ms.openlocfilehash: b5cce832ae7ae83552c2a5ded2f5f5bda0ac76bf
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27591941"
---
# <a name="add-authentication-to-your-apache-cordova-app"></a>新增驗證至您的 Apache Cordova 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

## <a name="summary"></a>總結
在本教學課程中，您可以使用支援的身分識別提供者，將驗證加入 Apache Cordova 上的 TodoList 快速入門專案。 本教學課程以[開始使用 Mobile Apps] 為基礎，您必須先完成該教學課程。

## <a name="register"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

[觀看示範類似步驟的影片](https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-8-Azure-authentication)

## <a name="permissions"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

現在，您可以驗證是否已停用後端的匿名存取。 在 Visual Studio 中：

* 開啟您完成[開始使用 Mobile Apps] 教學課程時所建立的專案。
* 在 **Google Android 模擬器**中執行應用程式。
* 應用程式啟動後，確認有顯示「非預期的連接失敗」。

接下來，將應用程式更新為在使用者從行動應用程式後端要求資源之前，先驗證使用者。

## <a name="add-authentication"></a>將驗證新增至應用程式
1. 在 **Visual Studio** 中開啟您的專案，然後開啟 `www/index.html` 檔案進行編輯。
2. 找出 head 區段中的 `Content-Security-Policy` 中繼標籤。  將 OAuth 主機新增至允許的來源清單。

   | 提供者 | SDK 提供者名稱 | OAuth 主機 |
   |:--- |:--- |:--- |
   | Azure Active Directory | aad | https://login.microsoftonline.com |
   | Facebook | Facebook | https://www.facebook.com |
   | Google | Google | https://accounts.google.com |
   | Microsoft | microsoftaccount | https://login.live.com |
   | Twitter | Twitter | https://api.twitter.com |

    content-security-policy (針對 Azure Active Directory 實作) 的範例如下所示：

        <meta http-equiv="Content-Security-Policy" content="default-src 'self'
            data: gap: https://login.microsoftonline.com https://yourapp.azurewebsites.net; style-src 'self'">

    使用上表中的 OAuth 主機取代 `https://login.microsoftonline.com`。  如需 content-security-policy 中繼標籤的詳細資訊，請參閱 [Content-Security-Policy 文件]。

    在適當的行動裝置上使用時，某些驗證提供者不需要變更 Content-Security-Policy。  例如，在 Android 裝置上使用 Google 驗證時不需要 Content-Security-Policy 變更。

3. 開啟 `www/js/index.js` 檔案進行編輯，找出 `onDeviceReady()` 方法，並在用戶端建立程式碼底下新增下列程式碼：

        // Login to the service
        client.login('SDK_Provider_Name')
            .then(function () {

                // BEGINNING OF ORIGINAL CODE

                // Create a table reference
                todoItemTable = client.getTable('todoitem');

                // Refresh the todoItems
                refreshDisplay();

                // Wire up the UI Event Handler for the Add Item
                $('#add-item').submit(addItemHandler);
                $('#refresh').on('click', refreshDisplay);

                // END OF ORIGINAL CODE

            }, handleError);

    此程式碼會取代現有用於建立資料表參考和重新整理 UI 的程式碼。

    login () 方法會開始向提供者驗證。 login() 方法是會傳回 JavaScript Promise 的非同步函式。  初始化作業的其餘部分會置於承諾回應中，如此就不會在 login() 方法完成之前執行。

4. 在您剛才加入的程式碼中，使用您的登入提供者名稱取代 `SDK_Provider_Name` 。 例如，針對 Azure Active Directory，請使用 `client.login('aad')`。
5. 執行專案。  當專案完成初始化時，您的應用程式會針對選擇的驗證提供者顯示 OAuth 登入頁面。

## <a name="next-steps"></a>後續步驟
* 深入了解 Azure App Service [驗證相關資訊] 。
* 將 [推播通知] 新增至 Apache Cordova 應用程式，以繼續本教學課程。

了解如何使用 SDK。

* [Apache Cordova SDK]
* [ASP.NET Server SDK]
* [Node.js Server SDK]

<!-- URLs. -->
[開始使用 Mobile Apps]: app-service-mobile-cordova-get-started.md
[Content-Security-Policy 文件]: https://cordova.apache.org/docs/en/latest/guide/appdev/whitelist/index.html
[推播通知]: app-service-mobile-cordova-get-started-push.md
[驗證相關資訊]: app-service-mobile-auth.md
[Apache Cordova SDK]: app-service-mobile-cordova-how-to-use-client-library.md
[ASP.NET Server SDK]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[Node.js Server SDK]: app-service-mobile-node-backend-how-to-use-server-sdk.md
