---
title: 使用 Azure AD 驗證搭配 .NET 存取 Azure 媒體服務 API | Microsoft Docs
description: 本主題說明如何使用 Azure Active Directory (Azure AD) 驗證搭配 .NET 存取 Azure 媒體服務 (AMS) API。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/27/2018
ms.author: juliako
ms.openlocfilehash: b8f58f4010590dc40d5e8dc7ac1b634f161a807d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33784567"
---
# <a name="use-azure-ad-authentication-to-access-azure-media-services-api-with-net"></a>使用 Azure AD 驗證搭配 .NET 存取 Azure 媒體服務 API

從 windowsazure.mediaservices 4.0.0.4 開始，Azure 媒體服務支援以 Azure Active Directory (Azure AD) 為主的驗證。 本主題說明如何使用 Azure AD 驗證搭配 Microsoft .NET 存取 Azure 媒體服務 API。

## <a name="prerequisites"></a>先決條件

- 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
- 媒體服務帳戶。 如需詳細資訊，請參閱[使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。
- 最新的 [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices) 封裝。
- 熟讀[使用 AAD 驗證存取 Azure 媒體服務 API 概觀](media-services-use-aad-auth-to-access-ams-api.md)主題。 

使用 Azure AD 驗證搭配 Azure 媒體服務時，您可以下列其中一種方式進行驗證：

- **使用者驗證**可驗證使用應用程式與 Azure 媒體服務資源互動的人員。 互動式應用程式應該會先提示使用者輸入認證。 例如，授權的使用者用來監控編碼工作或即時串流的管理主控台應用程式。 
- **服務主體驗證**會驗證服務。 通常使用這種驗證方法的應用程式有執行精靈服務、中介層服務或排程的工作的應用程式：例如，Web 應用程式、函數應用程式、邏輯應用程式、API 或微服務。

>[!IMPORTANT]
>Azure 媒體服務目前支援 Azure 存取控制服務驗證模型。 不過，存取控制授權將在 2018 年 6 月 22 日被取代。 建議您儘速移轉至 Azure Active Directory 驗證模型。

## <a name="get-an-azure-ad-access-token"></a>取得 Azure AD 存取權杖

若要使用 Azure AD 驗證連線到 Azure 媒體服務 API，用戶端應用程式必須要求 Azure AD 存取權杖。 當您使用媒體服務 .NET 用戶端 SDK 時，許多有關如何取得 Azure AD 存取權杖的詳細資料已為您包裝並簡化於 [AzureAdTokenProvider](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenProvider.cs) 和 [AzureAdTokenCredentials](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenCredentials.cs) 類別中。 

例如，您不需要提供 Azure AD 授權單位、媒體服務資源 URI 或原生 Azure AD 應用程式詳細資料。 這些是已由 Azure AD 存取權杖提供者類別設定的已知值。 

如果您未使用 Azure 媒體服務 .NET SDK，建議您使用 [Azure AD 驗證程式庫](../../active-directory/develop/active-directory-authentication-libraries.md)。 若要取得搭配 Azure AD 驗證程式庫使用所需之參數的值，請參閱[使用 Azure 入口網站存取 Azure AD 驗證設定](media-services-portal-get-started-with-aad.md)。

您也可以選擇使用您自己的實作來取代預設的 **AzureAdTokenProvider** 實作。

## <a name="install-and-configure-azure-media-services-net-sdk"></a>安裝及設定 Azure 媒體服務 .NET SDK

>[!NOTE] 
>若要使用 Azure AD 驗證搭配媒體服務 .NET SDK，您需要最新的 [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices) 封裝。 此外，請將參考加入 **Microsoft.IdentityModel.Clients.ActiveDirectory** 組件。 如果您使用現有的應用程式，請包含 **Microsoft.WindowsAzure.MediaServices.Client.Common.Authentication.dll** 組件。 

1. 在 Visual Studio 中，建立新的 C# 主控台應用程式。
2. 使用 [windowsazure.mediaservices](https://www.nuget.org/packages/windowsazure.mediaservices) NuGet 封裝來安裝 **Azure 媒體服務 .NET SDK**。 

    若要使用 NuGet 加入參考，請採取下列步驟︰在 [方案總管] 中，以滑鼠右鍵按一下專案名稱，然後選取 [管理 NuGet 封裝]。 接著，搜尋 **windowsazure.mediaservices**，然後選取 [安裝]。
    
    -或-

    在 Visual Studio 的 [封裝管理員主控台] 中，執行下列命令。

        Install-Package windowsazure.mediaservices -Version 4.0.0.4

3. 新增 **using** 至您的原始程式碼。

        using Microsoft.WindowsAzure.MediaServices.Client; 

## <a name="use-user-authentication"></a>使用使用者驗證

若要利用使用者驗證選項連線到 Azure 媒體服務 API，用戶端應用程式必須使用下列參數要求 Azure AD 權杖：  

- Azure AD 租用戶端點。 租用戶資訊可從 Azure 入口網站擷取。 將滑鼠游標停留在右上角登入的使用者。
- 媒體服務資源 URI。
- 媒體服務 (原生) 應用程式用戶端識別碼。 
- 媒體服務 (原生) 應用程式重新導向 URI。 

這些參數的值位於 **AzureEnvironments.AzureCloudEnvironment**。 **AzureEnvironments.AzureCloudEnvironment** 常數是 .NET SDK 中的協助程式，用來取得公用 Azure 資料中心正確的環境變數設定。 

它包含預先定義的環境設定，只用於存取公用資料中心的媒體服務。 對於 sovereign 或政府雲端區域，分別可以使用 **AzureChinaCloudEnvironment****AzureUsGovernmentEnvrionment** 或 **AzureGermanCloudEnvironment**。

下列程式碼範例會建立權杖：
    
    var tokenCredentials = new AzureAdTokenCredentials("microsoft.onmicrosoft.com", AzureEnvironments.AzureCloudEnvironment);
    var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
  
若要開始針對媒體服務進行程式設計，您需要建立可呈現伺服器內容的 **CloudMediaContext** 執行個體。 **CloudMediaContext** 包含重要集合的參考，包括工作、資產、檔案、存取原則和定位器。 

您還需要將**媒體 REST 服務的資源 URI** 傳遞至 **CloudMediaContext** 建構函式。 若要取得媒體 REST 服務的資源 URI，請登入 Azure 入口網站，選取您的 Azure 媒體服務帳戶，選取 [API 存取權]，然後選取 [使用使用者驗證連線到 Azure 媒體服務]。 

下列程式碼範例會建立 **CloudMediaContext** 執行個體：

    CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

下列範例說明如何建立 Azure AD 權杖和內容：

    namespace AADAuthSample
    {
        class Program
        {
            static void Main(string[] args)
            {
                // Specify your Azure AD tenant domain, for example "microsoft.onmicrosoft.com".
                var tokenCredentials = new AzureAdTokenCredentials("{YOUR AAD TENANT DOMAIN HERE}", AzureEnvironments.AzureCloudEnvironment);
    
                var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
    
                // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
                CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
    
                var assets = context.Assets;
                foreach (var a in assets)
                {
                    Console.WriteLine(a.Name);
                }
            }
    
        }
    }

>[!NOTE]
>如果您收到指出「遠端伺服器傳回一個錯誤: (401) 未經授權」的例外狀況，請參閱使用 Azure AD 驗證存取 Azure 媒體服務 API 概觀的[存取控制](media-services-use-aad-auth-to-access-ams-api.md#access-control)一節。

## <a name="use-service-principal-authentication"></a>使用服務主體驗證
    
若要利用服務主體選項連線到 Azure 媒體服務 API，您的中介層應用程式 (Web API 或 Web 應用程式) 必須要求具有下列參數的 Azure AD 權杖：  

- Azure AD 租用戶端點。 租用戶資訊可從 Azure 入口網站擷取。 將滑鼠游標停留在右上角登入的使用者。
- 媒體服務資源 URI。
- Azure AD 應用程式的值：**用戶端識別碼**和**用戶端祕密**。

**用戶端識別碼**和**用戶端祕密**參數的值可以在 Azure 入口網站中找到。 如需詳細資訊，請參閱[利用 Azure 入口網站開始使用 Azure AD 驗證](media-services-portal-get-started-with-aad.md)。

下列程式碼範例會使用以 **AzureAdClientSymmetricKey** 做為參數的 **AzureAdTokenCredentials** 建構函式建立權杖： 
    
    var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", 
                                new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"), 
                                AzureEnvironments.AzureCloudEnvironment);

    var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

您也可以指定以 **AzureAdClientCertificate** 做為參數的 **AzureAdTokenCredentials** 建構函式。 

如需有關如何以可供 Azure AD 使用之形式建立及設定憑證的指示，請參閱[在精靈應用程式中使用憑證向 Azure AD 驗證 - 手動設定步驟](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential/blob/master/Manual-Configuration-Steps.md)。

    var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", 
                                new AzureAdClientCertificate("{YOUR CLIENT ID HERE}", "{YOUR CLIENT CERTIFICATE THUMBPRINT}"), 
                                AzureEnvironments.AzureCloudEnvironment);

若要開始針對媒體服務進行程式設計，您需要建立可呈現伺服器內容的 **CloudMediaContext** 執行個體。 您還需要將**媒體 REST 服務的資源 URI** 傳遞至 **CloudMediaContext** 建構函式。 您也可以從 Azure 入口網站取得**媒體 REST 服務的資源 URI** 值。

下列程式碼範例會建立 **CloudMediaContext** 執行個體：

    CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
    
下列範例說明如何建立 Azure AD 權杖和內容：

    namespace AADAuthSample
    {
    
        class Program
        {
            static void Main(string[] args)
            {
                var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", 
                                            new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"), 
                                            AzureEnvironments.AzureCloudEnvironment);
            
                var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
    
                // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".      
                CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
    
                var assets = context.Assets;
                foreach (var a in assets)
                {
                    Console.WriteLine(a.Name);
                }
    
                Console.ReadLine();
            }
    
        }
    }

## <a name="next-steps"></a>後續步驟

開始使用[上傳檔案至您的帳戶](media-services-dotnet-upload-files.md)。
