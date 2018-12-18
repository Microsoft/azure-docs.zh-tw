---
title: Azure 媒體服務常見問題集 | Microsoft Docs
description: 常見問題集 (FAQ)
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: juliako
ms.openlocfilehash: d4b7d8ec5cb162e5fc844f107fbd5eb08fb00639
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353524"
---
# <a name="frequently-asked-questions"></a>常見問題集

本文解說「Azure 媒體服務」(AMS) 使用者社群所提出的常見問題集。

## <a name="general-ams-faqs"></a>一般 AMS 常見問題集

問：如何串流至 Apple iOS 裝置

答：將 "(format=m3u8-aapl)" 路徑新增至 URL 的 "/Manifest" 部分，來告訴串流原始伺服器傳回 HLS 內容以供在 Apple iOS 原生裝置上取用 (如需詳細資料，請參閱 (提供內容)[media-services-deliver-content-overview.md])。

問：如何調整索引？

答：編碼與編製索引工作的保留單位都相同。 請依照 [如何調整編碼保留單位](media-services-scale-media-processing-overview.md)的指示進行。 **請注意** ，Indexer 效能不受保留單元類型影響。

問：我已上傳、編碼以及發佈視訊。 當我試著串流處理視頻時，為什麼不會播放視頻？

答：其中一個最常見的原因就是您嘗試從中進行播放的串流端點不是處於 [執行中] 狀態。  

問：我可以編輯即時資料流？

答：Azure 媒體服務目前尚未開放即時資料流的編輯功能，所以請您事先在電腦上編輯。

問：Azure CDN 可以搭配即時資料流使用？

答：媒體服務支援 Azure CDN 整合 (如需詳細資訊，請參閱 [如何在媒體服務帳戶中管理串流端點](media-services-portal-manage-streaming-endpoints.md))。  即時資料流可以搭配 CDN 使用。 Azure 媒體服務提供 Smooth Streaming、HLS 和 MPEG-DASH 輸出。 所有這些格式都使用 HTTP 來傳送資料以及獲得 HTTP 快取的優點。 在即時資料流實際視訊/音訊資料是分割成片段，而且這個獨立片段會快取到 CDN。 唯一要重新整理的資料是資訊清單資料。 CDN 會定期重新整理資訊清單資料。

問：Azure 媒體服務支援儲存影像？

答：如果您只想要儲存 JPEG 或 PNG 影像，請儲存至 Azure Blob 儲存體。 除非您想維持影像與視訊或音訊資產之間的關聯，否則將影像保存在媒體服務帳戶中，實際上一點用處也沒有。 或者，當您需要在視訊編碼器中將影像作為重疊時才有必要。媒體編碼器標準支援在視訊上層重疊影像，因此會將 JPEG 和 PNG 列為支援的輸入格式。 如需詳細資訊，請參閱 [建立重疊](media-services-advanced-encoding-with-mes.md#overlay)。

問：我如何將資產從一個媒體服務帳戶複製到另一個帳戶。

答：若要使用 .NET 將資產從某個媒體服務帳戶複製到另一個帳戶，請使用 [Azure Media Services .NET SDK Extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions/) (Azure 媒體服務 .NET SDK 擴充功能) 儲存機制中可用的 [IAsset.Copy](https://github.com/Azure/azure-sdk-for-media-services-extensions/blob/dev/MediaServices.Client.Extensions/IAssetExtensions.cs#L354) 擴充功能方法。 如需詳細資訊，請參閱 [這個](https://social.msdn.microsoft.com/Forums/azure/28912d5d-6733-41c1-b27d-5d5dff2695ca/migrate-media-services-across-subscription?forum=MediaServices) 論壇執行緒。

問：使用 AMS 時，有哪些支援的字元可用來命名檔案？

答：建置串流內容的 URL (例如， http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters) 時，媒體服務會使用 IAssetFile.Name 屬性的值。基於這個理由，不允許 percent-encoding。 **Name** 屬性的值不能有下列任何[百分比編碼保留字元](http://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)：!*'();:@&=+$,/?%#[]"。 此外，只能有一個 ‘.’ 在檔案名稱的副檔名。

問：如何使用 REST 連線？

答：如需連線至 AMS API 的詳細資訊，請參閱[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)。 

問：如何在編碼過程中旋轉影片？

答：[Media Encoder Standard](media-services-dotnet-encode-with-media-encoder-standard.md) 支援 90/180/270 度的旋轉角度。 預設行為是「自動」，此時它會嘗試偵測內送之 MP4/MOV 檔案的旋轉中繼資料並加以補償。 包括以下 **Sources** 元素至[這裡](media-services-mes-presets-overview.md)所定義的其中一個 json 預設項目：

    "Version": 1.0,
    "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Rotation": "90"
      }
    }
    ],
    "Codecs": [

    ...


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
