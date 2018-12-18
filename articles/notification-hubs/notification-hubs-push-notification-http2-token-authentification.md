---
title: 適用於 Azure 通知中樞 APNS 的權杖型 (HTTP/2) 驗證 | Microsoft Docs
description: 本主題說明如何運用適用於 APNS 的新權杖驗證
services: notification-hubs
documentationcenter: .net
author: dimazaid
manager: kpiteira
editor: spelluru
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: ca86130e9c184576fc44119190d6224a363c6561
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777887"
---
# <a name="token-based-http2-authentication-for-apns"></a>適用於 APNS 的權杖型 (HTTP/2) 驗證
## <a name="overview"></a>概觀
本文說明如何將新的 APNS HTTP/2 通訊協定與權杖型驗證搭配使用。

使用新通訊協定的主要優點包括：
-   產生權杖相對輕而易舉 (相較於憑證)
-   不再需要顧慮到期日 – 您可以控制驗證權杖及其撤銷
-   目前的承載上限為 4 KB
- 同步的意見反應
-   您會使用 Apple 的最新通訊協定 – 憑證仍使用已遭淘汰的二進位通訊協定

在幾分鐘內透過兩個步驟完成使用此新機制的動作：
1.  從 Apple 開發人員帳戶入口網站取得必要資訊
2.  使用新資訊設定您的通知中樞

通知中樞目前已準備就緒，可將新驗證系統與 APNS 搭配使用。 

請注意，如果您從使用適用於 APNS 的憑證認證進行移轉：
- 權杖屬性會在我們的系統覆寫您的憑證，
- 但您的應用程式仍可順利接收通知。

## <a name="obtaining-authentication-information-from-apple"></a>從 Apple 取得驗證資訊
若要啟用權杖型驗證，您需要從 Apple 開發人員帳戶取得下列屬性：
### <a name="key-identifier"></a>金鑰識別碼
您可以從 Apple 開發人員帳戶中的 [Keys]\(金鑰\) 頁面取得金鑰識別碼

![](./media/notification-hubs-push-notification-http2-token-authentification/obtaining-auth-information-from-apple.png)

### <a name="application-identifier--application-name"></a>應用程式識別碼和應用程式名稱
您可以透過開發人員帳戶中的 [App IDs]\(應用程式識別碼\) 頁面取得應用程式名稱。 
![](./media/notification-hubs-push-notification-http2-token-authentification/app-name.png)

您可以透過開發人員帳戶中的成員資格詳細資料頁面取得應用程式識別碼。
![](./media/notification-hubs-push-notification-http2-token-authentification/app-id.png)


### <a name="authentication-token"></a>驗證權杖
為應用程式產生權杖之後，您便可以下載該驗證權杖。 如需有關如何產生此權杖的詳細資訊，請參閱 [Apple 開發人員文件](http://help.apple.com/xcode/mac/current/#/dev11b059073?sub=dev1eb5dfe65) (英文)。

## <a name="configuring-your-notification-hub-to-use-token-based-authentication"></a>將您的通知中樞設定為使用權杖型驗證
### <a name="configure-via-the-azure-portal"></a>透過 Azure 入口網站進行設定
若要在入口網站中啟用權杖型驗證，請登入 Azure 入口網站，並移至 [通知中樞] > [Notification Services] > [APNS] 面板。 

我們已提供新屬性 – 驗證模式。 選取權杖可讓您透過所有相關的權杖屬性更新您的中樞。

![](./media/notification-hubs-push-notification-http2-token-authentification/azure-portal-apns-settings.png)

- 輸入您從 Apple 開發人員帳戶擷取的屬性， 
- 選擇您的應用程式模式 (「生產」或「沙箱」) 
- 按一下 [儲存] 以更新 APNS 認證。 

### <a name="configure-via-management-api-rest"></a>透過管理 API (REST) 進行設定

您可以使用我們的[管理 API](https://msdn.microsoft.com/library/azure/dn495827.aspx)，將您的通知中樞更新為使用權杖型驗證。
根據您設定的是的「沙箱」或「生產」應用程式 (在 Apple 開發人員帳戶中指定)，使用其中一個對應端點：

- 沙箱端點：[https://api.development.push.apple.com:443/3/device](https://api.development.push.apple.com:443/3/device)
- 生產環境端點： [https://api.push.apple.com:443/3/device](https://api.push.apple.com:443/3/device)

> [!IMPORTANT]
> 權杖型驗證所需的 API 版本：**2017-04 或更新版本**。
> 
> 

透過權杖型驗證更新中樞的 PUT 要求範例如下：


        PUT https://{namespace}.servicebus.windows.net/{Notification Hub}?api-version=2017-04
          "Properties": {
            "ApnsCredential": {
              "Properties": {
                "KeyId": "<Your Key Id>",
                "Token": "<Your Authentication Token>",
                "AppName": "<Your Application Name>",
                "AppId": "<Your Application Id>",
                "Endpoint":"<Sandbox/Production Endpoint>"
              }
            }
          }
        

### <a name="configure-via-the-net-sdk"></a>透過 .NET SDK 進行設定
您可以使用我們的[最新用戶端 SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/1.0.8)，將中樞設定為使用權杖型驗證。 

示範正確使用方式的程式碼範例如下：


        NamespaceManager nm = NamespaceManager.CreateFromConnectionString(_endpoint);
        string token = "YOUR TOKEN HERE";
        string keyId = "YOUR KEY ID HERE";
        string appName = "YOUR APP NAME HERE";
        string appId = "YOUR APP ID HERE";
        NotificationHubDescription desc = new NotificationHubDescription("PATH TO YOUR HUB");
        desc.ApnsCredential = new ApnsCredential(token, keyId, appId, appName);
        desc.ApnsCredential.Endpoint = @"https://api.development.push.apple.com:443/3/device";
        nm.UpdateNotificationHubAsync(desc);

## <a name="reverting-to-using-certificate-based-authentication"></a>還原為使用憑證式驗證
您可以使用任何上述方法並傳遞憑證而不是權杖屬性，隨時還原為使用憑證式驗證。 該動作會覆寫先前儲存的認證。
