---
title: 如何為您的應用程式服務應用程式設定 Google 驗證
description: 了解如何為您的應用程式服務應用程式設定 Google 驗證。
services: app-service
documentationcenter: ''
author: mattchenderson
manager: syntaxc4
editor: ''
ms.assetid: 2b2f9abf-9120-4aac-ac5b-4a268d9b6e2b
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/19/2018
ms.author: mahender
ms.openlocfilehash: f89ff3a030f1da75bca538eefaf2496e9be8e97b
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35233814"
---
# <a name="how-to-configure-your-app-service-application-to-use-google-login"></a>如何設定 App Service 應用程式以使用 Google 登入
[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主題說明如何設定 Azure App Service，以使用 Google 做為驗證提供者。

若要完成本主題的程序，您必須具有已通過電子郵件地址驗證的 Google 帳戶。 若要建立新的 Google 帳戶，請前往 [accounts.google.com](http://go.microsoft.com/fwlink/p/?LinkId=268302)。

## <a name="register"> </a>向 Google 註冊您的應用程式
1. 登入 [Azure 入口網站]，然後瀏覽到您的應用程式。 複製 **URL**，以供稍後用來設定 Google 應用程式。
2. 瀏覽至 [Google APIs](http://go.microsoft.com/fwlink/p/?LinkId=268303) 網站，以您的 Google 帳戶認證登入，按一下 [建立專案]，提供 [專案名稱]，然後按一下 [建立]。
3. 建立專案之後，請選取專案。 在專案儀表板中，按一下 [移至 API 概觀]。
4. 選取 [啟用 API 和服務]。 搜尋 **Google+ API**，並加以選取。 接著，按一下 [啟用] 。
5. 在左側導覽中，按一下 [憑證] > [OAuth 同意畫面]，然後選取您的 [電子郵件地址]，輸入 [產品名稱]，再按一下 [儲存]。
6. 在 [認證] 索引標籤中，按一下 [建立認證] > [OAuth 用戶端 ID]。
7. 在「建立用戶端識別碼」畫面上，選取 [Web 應用程式]。
8. 將您稍早複製的 App Service **URL** 貼到 [已授權的 JAVASCRIPT 來源]，然後將重新導向 URI 貼到 [授權的重新導向 URI]。 重新導向 URI 是您的應用程式 URL 加上路徑 /.auth/login/google/callback。 例如： `https://contoso.azurewebsites.net/.auth/login/google/callback`。 請確實使用 HTTPS 配置。 然後按一下 [建立] 。
9. 在下一個畫面上，記下用戶端識別碼和用戶端密碼的值。

    > [!IMPORTANT]
    > 用戶端密碼是重要的安全性認證。 請勿與任何人共用此密碼，或在用戶端應用程式中加以散發。


## <a name="secrets"> </a>將 Google 資訊新增至應用程式
1. 回到 [Azure 入口網站]，並瀏覽到您的應用程式。 依序按一下 [設定] 及 [驗證/授權]。
2. 如果 [驗證/授權] 功能未啟用，請切換到 [開] 。
3. 按一下 [Google] 。 貼上先前取得的應用程式識別碼與應用程式密碼值，然後選擇性啟用應用程式需要的任何範圍。 然後按一下 [確定] 。
   
   ![][1]
   
   App Service 預設會提供驗證，但不會限制對您網站內容和 API 的已授權存取。 您必須在應用程式程式碼中授權使用者。
4. (選擇性) 若要限制只有透過 Google 授權的使用者可以存取您的網站，請將 [要求未經驗證時所採取的動作] 設為 [Google]。 這會要求所有要求都需經過驗證，且所有未經驗證的要求都會重新導向至 Google 以進行驗證。
5. 按一下 [檔案] 。

現在，您已可在應用程式中使用 Google 進行驗證。

## <a name="related-content"> </a>相關內容
[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Anchors. -->

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-settings.png

<!-- URLs. -->

[Google apis]: http://go.microsoft.com/fwlink/p/?LinkId=268303

[Azure 入口網站]: https://portal.azure.com/

