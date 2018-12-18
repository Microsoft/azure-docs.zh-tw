---
title: 發佈原生用戶端應用程式 - Azure AD | Microsoft Docs
description: 涵蓋如何讓原生用戶端應用程式與 Azure AD 應用程式 Proxy 連接器通訊，為內部部署的應用程式提供安全的遠端存取。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/31/2018
ms.author: barbkess
ms.reviewer: harshja
ms.custom: it-pro
ms.openlocfilehash: 91f75fdbe7bc7c66bbe581e9d4b295467acd4cd5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989663"
---
# <a name="how-to-enable-native-client-apps-to-interact-with-proxy-applications"></a>如何讓原生用戶端應用程式與 Proxy 應用程式互動

除了 Web 應用程式，Azure Active Directory 應用程式 Proxy 也可以用於發佈以 Azure AD Authentication Library (ADAL) 設定的原生用戶端應用程式。 原生用戶端應用程式與 Web 應用程式不同，因為這種應用程式會安裝在裝置上，而 Web 應用程式則是透過瀏覽器存取。 

應用程式 Proxy 藉由接受在標頭中傳送的 Azure AD 發行權杖，來支援原生用戶端應用程式。 應用程式 Proxy 服務會代表使用者執行驗證。 此解決方案不使用應用程式權杖進行驗證。 

![使用者、Azure Active Directory 和已發佈應用程式之間的關係](./media/application-proxy-configure-native-client-application/richclientflow.png)

請使用 Azure AD 驗證程式庫來發佈原生應用程式，該程式庫會處理驗證並支援許多用戶端環境。 應用程式 Proxy 融入 [原生應用程式到 Web API 案例](../develop/native-app.md)。 

本文引導您完成使用應用程式 Proxy 和 Azure AD 驗證程式庫發佈原生應用程式的四個步驟。 

## <a name="step-1-publish-your-application"></a>步驟 1：發佈您的應用程式
如同任何其他應用程式一般，發佈您的 Proxy 應用程式，並指派使用者以存取您的應用程式。 如需詳細資訊，請參閱[使用應用程式 Proxy 發佈應用程式](application-proxy-publish-azure-portal.md)。

## <a name="step-2-configure-your-application"></a>步驟 2：設定您的應用程式
以下列方式設定原生應用程式：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 巡覽至 [Azure Active Directory] > [應用程式註冊]。
3. 選取 [新增應用程式註冊]。
4. 指定應用程式的名稱，選取 [原生] 作為應用程式類型，並提供應用程式的重新導向 URI。 

   ![建立新的應用程式註冊](./media/application-proxy-configure-native-client-application/create.png)
5. 選取 [建立] 。

如需建立新應用程式註冊的詳細資訊，請參閱[整合應用程式與 Azure Active Directory](../develop/quickstart-v1-integrate-apps-with-azure-ad.md)。


## <a name="step-3-grant-access-to-other-applications"></a>步驟 3：授與其他應用程式存取權
啟用要公開給您的目錄中的其他應用程式的原生應用程式：

1. 仍在 [應用程式註冊] 中，選取您剛才建立的新原生應用程式。
2. 選取 [必要權限]。
3. 選取 [新增] 。
4. 開啟第一個步驟 [選取 API]。
5. 使用搜尋列，尋找您在第一節中發佈的「應用程式 Proxy」應用程式。 選擇該應用程式，然後按一下 [選取]。 

   ![搜尋 Proxy 應用程式](./media/application-proxy-configure-native-client-application/select_api.png)
6. 開啟第二個步驟 [選取權限]。
7. 使用核取方塊授與 Proxy 應用程式的原生應用程式存取權，然後按一下 [選取]。

   ![授與 Proxy 應用程式存取權](./media/application-proxy-configure-native-client-application/select_perms.png)
8. 選取 [完成] 。


## <a name="step-4-edit-the-active-directory-authentication-library"></a>步驟 4：編輯 Active Directory 驗證程式庫
在 Active Directory 驗證程式庫 (ADAL) 的驗證內容中編輯原生應用程式程式碼，以包含下列文字：

```
// Acquire Access Token from AAD for Proxy Application
AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/<Tenant ID>");
AuthenticationResult result = await authContext.AcquireTokenAsync("< External Url of Proxy App >",
        "<App ID of the Native app>",
        new Uri("<Redirect Uri of the Native App>"),
        PromptBehavior.Never);

//Use the Access Token to access the Proxy Application
HttpClient httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);
HttpResponseMessage response = await httpClient.GetAsync("< Proxy App API Url >");
```

範例程式碼中的變數應取代如下：

* **租用戶識別碼**可以在 Azure 入口網站中找到。 巡覽至 [Azure Active Directory] > [屬性]，然後複製目錄識別碼。 
* [外部 URL] 是您在 Proxy 應用程式中輸入的前端 URL。 若要尋找此值，請巡覽至 Proxy 應用程式的 [應用程式 Proxy] 區段。
* 原生應用程式的**應用程式識別碼**可以在原生應用程式的 [屬性] 頁面上找到。
* **原生應用程式的重新導向 URI** 可以在原生應用程式的 [重新導向 URI] 頁面上找到。

使用這些參數編輯 ADAL 後，即使您的使用者在公司網路之外，也應當能夠向原生用戶端應用程式進行驗證。 

## <a name="next-steps"></a>後續步驟

如需原生應用程式流程的詳細資訊，請參閱[原生應用程式到 Web API](../develop/native-app.md)。

深入了解設定[應用程式 Proxy 的單一登入](application-proxy-single-sign-on.md)
