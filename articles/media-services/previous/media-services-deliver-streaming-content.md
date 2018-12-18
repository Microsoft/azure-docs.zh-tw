---
title: 使用 .NET 發佈 Azure 媒體服務內容 | Microsoft Docs
description: 了解如何建立定位器，用來建置串流 URL。 程式碼範例以 C# 撰寫，並使用 Media Services SDK for .NET。
author: juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: c53b1f83-4cb1-4b09-840f-9c145b7d6f8d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2017
ms.author: juliako
ms.openlocfilehash: 224c9cf5ef9925645de1d94dc5bc03c15ba91432
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783207"
---
# <a name="publish-azure-media-services-content-using-net"></a>使用 .NET 發佈 Azure 媒體服務內容
> [!div class="op_single_selector"]
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [入口網站](media-services-portal-publish.md)
> 
> 

## <a name="overview"></a>概觀
您可以建立隨選串流定位器及建置串流 URL，串流處理調適性位元速率 MP4 集。 [為資產編碼](media-services-encode-asset.md) 主題說明如何編碼為調適性位元速率 MP4 集。 

> [!NOTE]
> 如果您的內容已加密，請在建立定位器之前設定資產傳遞原則 (如 [這個](media-services-dotnet-configure-asset-delivery-policy.md) 主題中所述)。 
> 
> 

您也可以使用隨選串流定位器來建置指向可漸進式下載之 MP4 檔案的 URL。  

本主題說明如何建立隨選串流定位器，以發佈資產及建置 Smooth、MPEG DASH 和 HLS 串流 URL。 它也會示範如何建置漸進式下載 URL。 

## <a name="create-an-ondemand-streaming-locator"></a>建立隨選串流定位器
若要建立隨選串流定位器並取得 URL，您需要執行下列動作：

1. 如果內容已加密，請定義存取原則。
2. 建立隨選串流定位器。
3. 如果您想要串流處理，請取得資產內的串流資訊清單檔案 (.ism)。 
   
   如果您想要漸進式地下載，請取得資產中的 MP4 檔案名稱。  
4. 建置資訊清單檔或 MP4 檔案的 URL。 


>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的天數 / 存取權限，則應該使用相同的原則識別碼。 例如，為預定要長時間維持就地 (非上傳原則) 的定位器原則。 如需詳細資訊，請參閱[此主題](media-services-dotnet-manage-entities.md#limit-access-policies)。

### <a name="use-media-services-net-sdk"></a>使用 Media Services .NET SDK
建置串流 URL 

```csharp
    private static void BuildStreamingURLs(IAsset asset)
    {

        // Create a 30-day readonly access policy. 
          // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create a locator to the streaming content on an origin. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get a reference to the streaming manifest file from the  
        // collection of files in the asset. 
        var manifestFile = asset.AssetFiles.Where(f => f.Name.ToLower().
                                    EndsWith(".ism")).
                                    FirstOrDefault();

        // Create a full URL to the manifest file. Use this for playback
        // in streaming media clients. 
        string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
        Console.WriteLine("URL to manifest for client streaming using Smooth Streaming protocol: ");
        Console.WriteLine(urlForClientStreaming);
        Console.WriteLine("URL to manifest for client streaming using HLS protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=m3u8-aapl)");
        Console.WriteLine("URL to manifest for client streaming using MPEG DASH protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=mpd-time-csf)"); 
        Console.WriteLine();
    }
```

輸出：

    URL to manifest for client streaming using Smooth Streaming protocol:
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest
    URL to manifest for client streaming using HLS protocol:
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)
    URL to manifest for client streaming using MPEG DASH protocol:
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)


> [!NOTE]
> 您也可以透過 SSL 連線串流您的內容。 若要執行這個方法，請確定您的串流 URL 是以 HTTPS 開頭。 目前 AMS 不支援使用 SSL 搭配自訂網域。
> 
> 

建置漸進式下載 URL 

```csharp
    private static void BuildProgressiveDownloadURLs(IAsset asset)
    {
        // Create a 30-day readonly access policy. 
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create an OnDemandOrigin locator to the asset. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get MP4 files.
        IEnumerable<IAssetFile> mp4AssetFiles = asset
            .AssetFiles
            .ToList()
            .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Create a full URL to the MP4 files. Use this to progressively download files.
        foreach (var pd in mp4AssetFiles)
            Console.WriteLine(originLocator.Path + pd.Name);
    }
```
輸出：

    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4
    http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4

    . . . 

### <a name="use-media-services-net-sdk-extensions"></a>使用 Media Services .NET SDK 延伸模組
下列程式碼會呼叫 .NET SDK 擴充功能方法，以針對調適性串流建立定位器並產生 Smooth Streaming、HLS 和 MPEG-DASH URL。
```csharp
    // Create a loctor.
    _context.Locators.Create(
        LocatorType.OnDemandOrigin,
        inputAsset,
        AccessPermissions.Read,
        TimeSpan.FromDays(30));

    // Get the streaming URLs.
    Uri smoothStreamingUri = inputAsset.GetSmoothStreamingUri();
    Uri hlsUri = inputAsset.GetHlsUri();
    Uri mpegDashUri = inputAsset.GetMpegDashUri();

    Console.WriteLine(smoothStreamingUri);
    Console.WriteLine(hlsUri);
    Console.WriteLine(mpegDashUri);
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>後續步驟
* [下載資產](media-services-deliver-asset-download.md)
* [設定資產傳遞原則](media-services-dotnet-configure-asset-delivery-policy.md)

