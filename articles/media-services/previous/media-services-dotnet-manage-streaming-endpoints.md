---
title: 使用 .NET SDK 來管理串流端點。 | Microsoft Docs
description: 本文章說明如何透過 Azure 入口網站管理串流端點。
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: cfowler
editor: ''
ms.assetid: 0da34a97-f36c-48d0-8ea2-ec12584a2215
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 741eb35c58fb723985a60f6ac071892c02d08412
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782927"
---
# <a name="manage-streaming-endpoints-with-net-sdk"></a>使用 .NET SDK 來管理串流端點

>[!NOTE]
>請務必檢閱[概觀](media-services-streaming-endpoints-overview.md)一文。 此外，也請檢閱 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。

本文章中的程式碼示範如何使用「Azure 媒體服務 .NET SDK」來執行下列工作：

- 檢查預設串流端點。
- 建立/新增新的串流端點。

    如果您打算有不同的 CDN 或一個 CDN 和直接存取，您可能會想要有多個串流端點。

    > [!NOTE]
    > 只有當串流端點處於執行中狀態時，才會向您收取費用。
    
- 更新串流端點。
    
    請務必呼叫 Update() 函式。

- 刪除串流端點。

    >[!NOTE]
    >預設串流端點不可刪除。

如需調整串流端點的相關資訊，請參閱[這篇](media-services-portal-scale-streaming-endpoints.md)文章。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

## <a name="add-code-that-manages-streaming-endpoints"></a>新增可管理串流端點的程式碼
    
以下列程式碼取代 Program.cs 中的程式碼：

```csharp
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.Live;

namespace AMSStreamingEndpoint
{
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

            var defaultStreamingEndpoint = _context.StreamingEndpoints.Where(s => s.Name.Contains("default")).FirstOrDefault();
            ExamineStreamingEndpoint(defaultStreamingEndpoint);

            IStreamingEndpoint newStreamingEndpoint = AddStreamingEndpoint();
            UpdateStreamingEndpoint(newStreamingEndpoint);
            DeleteStreamingEndpoint(newStreamingEndpoint);
        }

        static public void ExamineStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            Console.WriteLine(streamingEndpoint.Name);
            Console.WriteLine(streamingEndpoint.StreamingEndpointVersion);
            Console.WriteLine(streamingEndpoint.FreeTrialEndTime);
            Console.WriteLine(streamingEndpoint.ScaleUnits);
            Console.WriteLine(streamingEndpoint.CdnProvider);
            Console.WriteLine(streamingEndpoint.CdnProfile);
            Console.WriteLine(streamingEndpoint.CdnEnabled);
        }

        static public IStreamingEndpoint AddStreamingEndpoint()
        {
            var name = "StreamingEndpoint" + DateTime.UtcNow.ToString("hhmmss");
            var option = new StreamingEndpointCreationOptions(name, 1)
            {
                StreamingEndpointVersion = new Version("2.0"),
                CdnEnabled = true,
                CdnProfile = "CdnProfile",
                CdnProvider = CdnProviderType.PremiumVerizon
            };

            var streamingEndpoint = _context.StreamingEndpoints.Create(option);

            return streamingEndpoint;
        }

        static public void UpdateStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            if (streamingEndpoint.StreamingEndpointVersion == "1.0")
                streamingEndpoint.StreamingEndpointVersion = "2.0";

            streamingEndpoint.CdnEnabled = false;
            streamingEndpoint.Update();
        }

        static public void DeleteStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
        {
            streamingEndpoint.Delete();
        }
    }
}
```

## <a name="next-steps"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

