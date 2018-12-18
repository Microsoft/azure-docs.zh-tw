---
title: 為您的應用程式服務應用程式設定 Azure Active Directory 驗證
description: 了解如何為您的應用程式服務應用程式設定 Azure Active Directory 驗證。
author: mattchenderson
services: app-service
documentationcenter: ''
manager: syntaxc4
editor: ''
ms.assetid: 6ec6a46c-bce4-47aa-b8a3-e133baef22eb
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/19/2018
ms.author: mahender
ms.openlocfilehash: 0c2ae7e4cf0b19ab9e1c276504892a2a8aaa8dfc
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297378"
---
# <a name="configure-your-app-service-app-to-use-azure-active-directory-login"></a>設定 App Service 應用程式使用 Azure Active Directory 登入
[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本文說明如何設定 Azure 應用程式服務，以使用 Azure Active Directory 作為驗證提供者。

## <a name="express"> </a>使用快速設定設定 Azure Active Directory
1. 在 [Azure 入口網站]中，瀏覽至您的 App Service 應用程式。 在左側瀏覽區域中，選取 [驗證/授權]。
2. 如果 [驗證/授權] 並未啟用，請選取 [開啟]。
3. 選取 [Azure Active Directory]，然後選取 [管理模式] 下方的 [快速]。
4. 選取 [確定] 以在 Azure Active Directory 中註冊 App Service 應用程式。 這樣會建立新的應用程式註冊。 如果您想改為選擇現有的應用程式註冊，請按一下 [選取現有的 App]，然後在您的租用戶內搜尋先前建立之應用程式註冊的名稱。
   按一下應用程式註冊以選取它，然後按一下 [確定]。 然後在 Azure Active Directory 設定頁面上按一下 [確定]。
   App Service 預設會提供驗證，但不會限制對您網站內容和 API 的已授權存取。 您必須在應用程式程式碼中授權使用者。
5. (選擇性) 若要限制只有透過 Azure Active Directory 授權的使用者可以存取您的網站，請將 [要求未經驗證時所採取的動作] 設為 [使用 Azure Active Directory 登入]。 這會要求所有要求都需經過驗證，且所有未經驗證的要求都會重新導向至 Azure Active Directory 以進行驗證。
6. 按一下 [檔案] 。

現在，您已可在 App Service 應用程式中使用 Azure Active Directory 進行驗證。

## <a name="advanced"> </a>(替代方法) 使用進階設定手動設定 Azure Active Directory
您也可以選擇手動提供組態設定。 如果您想使用的 AAD 租用戶不同於您登入 Azure 所用的租用戶，這會是較佳的解決方案。 若要完成組態，您必須先在 Azure Active Directory 中建立註冊，接著必須提供一些註冊詳細資料給 App Service。

### <a name="register"> </a>向 Azure Active Directory 註冊您的 App Service 應用程式
1. 登入 [Azure 入口網站]，然後瀏覽到您的 App Service 應用程式。 複製您的應用程式 **URL**。 您將使用此資訊設定 Azure Active Directory 應用程式註冊。
2. 瀏覽至 [Active Directory]，然後選取 [應用程式註冊]，然後按一下上方的 [新增應用程式註冊] 以啟動新的應用程式註冊。 
3. 在 [建立] 頁面中，輸入應用程式註冊的**名稱**、選取 [Web 應用程式/API] 類型，在 [登入 URL] 方塊中貼上應用程式 URL (來自步驟 1)。 然後按一下 [建立]。
4. 幾秒鐘之後，您應該會看到剛才建立的新應用程式註冊。
5. 新增應用程式註冊之後，按一下應用程式註冊名稱，按一下上方的 [設定]，然後按一下 [屬性] 
6. 在 [應用程式識別碼 URI] 方塊中，貼上應用程式 URL (來自步驟 1)，同時在 [首頁 URL] 中貼上應用程式 URL (來自步驟 1)，然後按一下 [儲存]
7. 現在按一下 [回覆 URL]、編輯 [回覆 URL]、貼上應用程式 URL (來自步驟 1)、修改通訊協定以確定您有 **https://** 通訊協定 (而不是 http://)，然後將 */.auth/login/aad/callback* 附加到 URL 結尾 (例如 `https://contoso.azurewebsites.net/.auth/login/aad/callback`)。 按一下 [檔案] 。   
8.  此時，複製應用程式的 [應用程式識別碼]。 保留此資訊以供之後使用。 您會需要它來設定 App Service 應用程式。
9. 關閉 [註冊的應用程式] 頁面。 按一下 [應用程式註冊] 頁面上方的 [端點] 按鈕，然後複製 [同盟中繼資料文件] URL。 
10. 開啟新的瀏覽器視窗，貼上 URL 並瀏覽至 XML 頁面來瀏覽至 URL。 在文件頂端的是 **EntityDescriptor** 元素。 尋找 **entityID** 屬性，並複製其值。 此值會作為您的**簽發者 URL**。 稍後您會設定您的應用程式使用它。

### <a name="secrets"> </a>將 Azure Active Directory 資訊新增至 App Service 應用程式
1. 回到 [Azure 入口網站]中，瀏覽至您的 App Service 應用程式。 按一下 [驗證/授權]。 如果 [驗證/授權] 功能未啟用，請切換到 [ **開啟**]。 在 [驗證提供者] 下按一下 [Azure Active Directory] 來設定應用程式。 (選擇性) App Service 預設會提供驗證，但不會限制對您網站內容和 API 的已授權存取。 您必須在應用程式程式碼中授權使用者。 將 [當要求未經驗證時所要採取的動作] 設定為 [使用 Azure Active Directory 登入]。 這個選項會要求所有要求都需經過驗證，且所有未經驗證的要求都會重新導向至 Azure Active Directory 以進行驗證。
2. 在 [Active Directory 驗證] 設定中，在 [管理模式]下按一下 [進階]。 將應用程式識別碼貼入 [用戶端識別碼] 方塊中 (來自步驟 8)，並將 entityId (來自步驟 10) 貼入 [簽發者 URL] 值。 然後按一下 [確定] 。
3. 在 [Active Directory 驗證設定] 頁面上，按一下 [儲存]。

現在，您已可在 App Service 應用程式中使用 Azure Active Directory 進行驗證。

## <a name="optional-configure-a-native-client-application"></a>(選擇性步驟) 設定原生用戶端應用程式
Azure Active Directory 也可讓您註冊更能控制權限對應的原生用戶端。 若您想要使用 **Active Directory Authentication Library**等程式庫執行登入，您會需要該功能。

1. 請瀏覽到 [Azure 入口網站]中的 [Azure Active Directory]。
2. 在左側瀏覽區域中，選取 [應用程式註冊]。 按一下上方的 [新增應用程式註冊]。
4. 在 [建立] 頁面中，輸入應用程式註冊的**名稱**。 在 [應用程式類型] 中選取 [原生]。
5. 在 [重新導向 URI] 方塊中，使用 HTTPS 配置輸入您網站的 /.auth/login/done 端點。 此值應該類似 https://contoso.azurewebsites.net/.auth/login/done。 如果是建立 Windows 應用程式，請改用 [封裝 SID](../app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library.md#package-sid) 作為 URI。
5. 按一下頁面底部的 [新增] 。
6. 新增應用程式註冊之後，選取它以開啟。 找到 [應用程式識別碼]，並記下此值。
7. 按一下 [所有設定] > [必要權限] > [新增] > [選取 API]。
8. 輸入您稍早註冊的 App Service 應用程式名稱來搜尋它，然後選取它並按一下 [選取]。 
9. 選取 [存取 \<app_name>]。 然後按一下 [選取] 。 然後按一下 [完成]。

您現在已設定了可以存取您 App Service 應用程式的原生用戶端應用程式。

## <a name="related-content"> </a>相關內容
[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-webapp-url.png
[1]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-app_registration.png
[2]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-app-registration-create.png
[3]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-app-registration-config-appidurl.png
[4]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-app-registration-config-replyurl.png
[5]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-endpoints.png
[6]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-aad-endpoints-fedmetadataxml.png
[7]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-webapp-auth.png
[8]: ./media/app-service-mobile-how-to-configure-active-directory-authentication/app-service-webapp-auth-config.png



<!-- URLs. -->

[Azure 入口網站]: https://portal.azure.com/
[alternative method]:#advanced
