---
title: 如何使用 .NET 設定用於媒體服務開發的電腦
description: 了解將 Media Services SDK for .NET 用於媒體服務的必要條件。 同時了解如何建立 Visual Studio 應用程式。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: ec2804c7-c656-4fbf-b3e4-3f0f78599a7f
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 639d1d6af169a0bb459dd8d6c778503b60c48e2c
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783497"
---
# <a name="media-services-development-with-net"></a>使用 .NET 進行媒體服務開發
[!INCLUDE [media-services-selector-setup](../../../includes/media-services-selector-setup.md)]

本文章討論如何使用 .NET 開始開發媒體服務應用程式。

**Azure Media Services .NET SDK 程式庫** 可讓您使用 .NET 對媒體服務進行程式設計。 為了讓使用 .NET 進行開發更為簡單，會提供 **Azure Media Services .NET SDK 延伸模組** 程式庫。 此程式庫包含一組延伸方法和協助程式函數，以簡化 .NET 程式碼。 這兩個程式庫都是透過 **NuGet** 和 **GitHub** 取得。

## <a name="prerequisites"></a>先決條件
* 新的或現有 Azure 訂用帳戶中的媒體服務帳戶。 請參閱文章[如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* 作業系統：Windows 10、Windows 7、Windows 2008 R2 或 Windows 8。
* .NET Framework 4.5 或更新版本。
* Visual Studio。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案
本節說明如何在 Visual Studio 中建立專案並設定來進行媒體服務開發。  在此案例中，專案是 C# Windows 主控台應用程式，但這裡顯示的設定步驟也適用於可以為媒體服務應用程式建立的其他專案類型 (例如，Windows Forms 應用程式或 ASP.NET Web 應用程式)。

本節顯示如何使用 **NuGet** 新增 Media Services .NET SDK 延伸模組和其他相依程式庫。

或者，您可以從 GitHub 取得最新 Media Services .NET SDK 位元 ([github.com/Azure/azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services) 或 [github.com/Azure/azure-sdk-for-media-services-extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions))、建置方案，並新增至用戶端專案的參考。 所有必要相依性皆會自動下載並解壓縮。

1. 在 Visual Studio 中，建立新的 C# 主控台應用程式。 輸入 [名稱]、[位置] 和 [方案名稱]，然後按一下 [確定]。
2. 建置方案。
3. 使用 **NuGet** 來安裝和新增 **Azure 媒體服務 .NET SDK 延伸模組** (**windowsazure.mediaservices.extensions**)。 安裝這個封裝，也會安裝 **Media Services .NET SDK** ，並新增所有其他必要相依性。
   
    確定您已安裝 NuGet 的最新版本。 如需詳細資訊和安裝指示，請參閱 [NuGet](http://nuget.codeplex.com/)。

    1. 在 [方案總管] 中，於專案名稱上按一下滑鼠右鍵，然後選擇 [管理 NuGet 套件]。

    2. [管理 NuGet 封裝] 對話方塊隨即出現。

    3. 在線上資源庫中，搜尋「Azure MediaServices 擴充功能」，選擇 [Azure Media Services .NET SDK 擴充功能] \(**windowsazure.mediaservices.extensions**)，然後按一下 [安裝] 按鈕。
   
    4. 會修改專案，並新增 Media Services .NET SDK 延伸模組、Media Services .NET SDK 和其他相依組件的參考。
4. 若要提升更乾淨的開發環境，請考慮啟用 [NuGet 封裝還原]。 如需詳細資訊，請參閱 [NuGet 封裝還原](http://docs.nuget.org/consume/package-restore)。
5. 加入 **System.Configuration** 組件的參考。 此組件包含用來存取組態檔 (例如 App.config) 的 System.Configuration.**ConfigurationManager** 類別。
   
    1. 若要使用 [管理參考] 對話方塊新增參考，請以滑鼠右鍵按一下 [方案總管] 中的專案名稱。 接著，按一下 [新增]，然後按一下 [參考]。
   
    2. [管理參考] 對話方塊隨即出現。
    3. 在 .NET Framework 組件底下，尋找並選取 System.Configuration 組件，然後按 [確定]。
6. 開啟 App.config 檔案並將 **appSettings** 區段新增至檔案。 設定連接媒體服務 API 時所需的值。 如需詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

    設定使用**服務主體**驗證方法連接時所需的值。

        ```csharp
                <configuration>
                ...
                    <appSettings>
                        <add key="AMSAADTenantDomain" value="tenant"/>
                        <add key="AMSRESTAPIEndpoint" value="endpoint"/>
                        <add key="AMSClientId" value="id"/>
                        <add key="AMSClientSecret" value="secret"/>
                    </appSettings>
                </configuration>
        ```

7. 將 **System.Configuration** 參考新增至專案。
8. 在 Program.cs 檔案的開頭，使用下列程式碼來覆寫現有的 **using** 陳述式：

    ```csharp      
            using System;
            using System.Configuration;
            using System.IO;
            using Microsoft.WindowsAzure.MediaServices.Client;
            using System.Threading;
            using System.Collections.Generic;
            using System.Linq;
    ```

    現在，您可以開始開發媒體服務應用程式。    

## <a name="example"></a>範例

以下是一個小範例，它會連接到 AMS API，並列出所有可用的媒體處理器。

```csharp
        class Program
        {
            // Read values from the App.config file.

            private static readonly string _AADTenantDomain =
                ConfigurationManager.AppSettings["AMSAADTenantDomain"];
            private static readonly string _RESTAPIEndpoint =
                ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
            private static readonly string _AMSClientId =
                ConfigurationManager.AppSettings["AMSClientId"];
            private static readonly string _AMSClientSecret =
                ConfigurationManager.AppSettings["AMSClientSecret"];
        
            private static CloudMediaContext _context = null;
            static void Main(string[] args)
            {
                AzureAdTokenCredentials tokenCredentials = 
                    new AzureAdTokenCredentials(_AADTenantDomain,
                        new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                        AzureEnvironments.AzureCloudEnvironment);

                var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

                _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
        
                // List all available Media Processors
                foreach (var mp in _context.MediaProcessors)
                {
                    Console.WriteLine(mp.Name);
                }
        
            }
 ```

## <a name="next-steps"></a>後續步驟

現在[您可以連接到 AMS API](media-services-use-aad-auth-to-access-ams-api.md)並開始[開發](media-services-dotnet-get-started.md)。


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

