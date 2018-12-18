---
title: 如何為您的應用程式服務應用程式設定 Microsoft 帳戶驗證
description: 了解如何為您的應用程式服務應用程式設定 Microsoft 帳戶驗證。
author: mattchenderson
services: app-service
documentationcenter: ''
manager: syntaxc4
editor: ''
ms.assetid: ffbc6064-edf6-474d-971c-695598fd08bf
ms.service: app-service
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/19/2018
ms.author: mahender
ms.openlocfilehash: abf09444e92c6faded42a9143b4b5c849a4cf41d
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48853261"
---
# <a name="how-to-configure-your-app-service-application-to-use-microsoft-account-login"></a>如何設定 App Service 應用程式以使用 Microsoft 帳戶登入
[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主題說明如何設定 Azure App Service，以使用 Microsoft 帳戶作為驗證提供者。 

## <a name="register-microsoft-account"> </a>使用 Microsoft 帳戶註冊應用程式
1. 登入 [Azure 入口網站]，然後瀏覽到您的應用程式。 複製 **URL**，以供稍後使用 Microsoft 帳戶來設定應用程式。
2. 瀏覽到 Microsoft 帳戶開發人員中心的 [我的應用程式] 頁面，並視需要使用您的 Microsoft 帳戶登入。
3. 按一下 [新增應用程式]，然後輸入應用程式名稱，再按一下 [建立]。
4. 請記下 [應用程式識別碼] ，因為稍後您將會用到此資訊。 
5. 在 [平台] 下，按一下 [新增平台]  ，然後選取 [網站]。
6. 在 [重新導向 URI] 下，提供應用程式的端點，然後按一下 [儲存] 。 
   
   > [!NOTE]
   > 重新導向 URI 是您的應用程式 URL 加上路徑 /.auth/login/microsoftaccount/callback。 例如， `https://contoso.azurewebsites.net/.auth/login/microsoftaccount/callback`。   
   > 請確實使用 HTTPS 配置。
   
7. 在 [應用程式密碼] 下，按一下 [產生新密碼] 。 請記下出現的值。 一旦您離開此頁面，就不會再次顯示。

    > [!IMPORTANT]
    > 密碼是重要的安全性認證。 請勿與任何人共用密碼，或在用戶端應用程式內散佈密碼。
    
8. 按一下 [儲存] 

## <a name="secrets"> </a>將 Microsoft 帳戶資訊新增至 App Service 應用程式
1. 回到 [Azure 入口網站]，瀏覽至您的應用程式，按一下 [設定] > [驗證/授權]。
2. 如果 [驗證/授權] 功能未啟用，請切換至 [開啟] 。
3. 按一下 [Microsoft 帳戶] 。 貼上先前取得的 [應用程式識別碼] 與 [應用程式密碼] 值，然後選擇性啟用應用程式需要的任何範圍。 然後按一下 [確定] 。
   
    ![][1]
   
    App Service 預設會提供驗證，但不會限制對您網站內容和 API 的已授權存取。 您必須在應用程式程式碼中授權使用者。
4. (選擇性) 若要限制只有透過 Microsoft 帳戶授權的使用者可以存取您的網站，請將 [要求未經驗證時所採取的動作] 設為 [Microsoft 帳戶]。 這會要求所有的要求都經過驗證，且所有未經驗證的要求會重新導向至 Microsoft 帳戶以進行驗證。
5. 按一下 [檔案] 。

現在，您已可在應用程式中使用 Microsoft 帳戶進行驗證。

## <a name="related-content"> </a>相關內容
[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/app-service-microsoftaccount-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-microsoft-authentication/mobile-app-microsoftaccount-settings.png

<!-- URLs. -->

[我的應用程式]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure 入口網站]: https://portal.azure.com/
