---
title: 如何為您的應用程式服務應用程式設定 Twitter 驗證
description: 了解如何為您的應用程式服務應用程式設定 Twitter 驗證。
services: app-service
documentationcenter: ''
author: mattchenderson
manager: syntaxc4
editor: ''
ms.assetid: c6dc91d7-30f6-448c-9f2d-8e91104cde73
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/19/2018
ms.author: mahender
ms.openlocfilehash: f6449f99fda9c1a612ed9f9134751ff76b25904c
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32149947"
---
# <a name="how-to-configure-your-app-service-application-to-use-twitter-login"></a>如何設定 App Service 應用程式以使用 Twitter 登入
[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主題說明如何設定 Azure App Service，以使用 Twitter 作為驗證提供者。

若要完成本主題的程序，您必須具有已驗證過電子郵件地址和電話號碼的 Twitter 帳戶。 若要建立新的 Twitter 帳戶，請前往 <a href="http://go.microsoft.com/fwlink/p/?LinkID=268287" target="_blank">twitter.com</a>。

## <a name="register"> </a>向 Twitter 註冊您的應用程式
1. 登入 [Azure 入口網站]，並瀏覽到您的應用程式。 複製您的 **URL**。 您將使用此 URL 設定您的 Twitter 應用程式。
2. 瀏覽至 [Twitter Developers] 網站，使用您的 Twitter 帳戶認證登入，然後按一下 [建立新的應用程式]。
3. 針對您新的應用程式輸入 [名稱]和 [說明]。 貼上您應用程式的 **URL** 作為 [網站]值。 然後，針對 [回呼 URL]，貼上您之前複製的 [回呼 URL]。 這是您已附加路徑 /.auth/login/twitter/callback 的行動應用程式閘道器。 例如： `https://contoso.azurewebsites.net/.auth/login/twitter/callback`。 請確實使用 HTTPS 配置。
4. 在頁面底部，閱讀並接受條款。 然後按一下 [ **建立 Twitter 應用程式**]。 這會註冊應用程式並顯示應用程式詳細資料。
5. 按一下 [設定] 索引標籤，勾選 [允許此應用程式用來以 Twitter 登入]，然後按一下 [更新設定]。
6. 選取 [ **金鑰和存取權杖** ] 索引標籤。記下 [消費者金鑰 (API 金鑰)] 和 [消費者密鑰 (API 密鑰)] 的值。
   
   > [!NOTE]
   > 消費者密碼是重要的安全性認證。 請勿將此密碼告訴任何人或隨應用程式一起散發。
   > 
   > 

## <a name="secrets"> </a>將 Twitter 資訊新增至應用程式
1. 回到 [Azure 入口網站]，並瀏覽到您的應用程式。 依序按一下 [設定] 及 [驗證/授權]。
2. 如果 [驗證/授權] 功能未啟用，請切換到 [開] 。
3. 按一下 [Twitter] 。 貼入您先前取得的應用程式識別碼和應用程式密鑰值。 然後按一下 [確定] 。
   
   ![][1]
   
   App Service 預設會提供驗證，但不會限制對您網站內容和 API 的已授權存取。 您必須在應用程式程式碼中授權使用者。
4. (選擇性) 若要限制只有透過 Twitter 授權的使用者可以存取您的網站，請將 [要求未經驗證時所採取的動作] 設為 [Twitter]。 這會要求所有要求都需經過驗證，且所有未經驗證的要求都會重新導向至 Twitter 以進行驗證。
5. 按一下 [檔案] 。

現在，您已可在應用程式中使用 Twitter 進行驗證。

## <a name="related-content"> </a>相關內容
[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-twitter-authentication/app-service-twitter-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-twitter-authentication/mobile-app-twitter-settings.png

<!-- URLs. -->

[Twitter Developers]: http://go.microsoft.com/fwlink/p/?LinkId=268300
[Azure 入口網站]: https://portal.azure.com/
[xamarin]: ../app-services-mobile-app-xamarin-ios-get-started-users.md
