---
title: 媒體編碼器標準 (MES) 的工作預設 | Microsoft Docs
description: 本主題提供「媒體編碼器標準」(MES) 之服務所定義範例預設的概觀。
author: Juliako
manager: cfow
editor: johndeu
services: media-services
documentationcenter: ''
ms.assetid: f243ed1c-ac9c-4300-a5f7-f092cf9853b9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/01/2017
ms.author: juliako
ms.openlocfilehash: 4c518771d95dcb571da294f8a752e7b5740ef19f
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783317"
---
# <a name="sample-presets-for-media-encoder-standard-mes"></a>媒體編碼器標準 (MES) 的範例預設

**媒體編碼器標準**會定義一組預先定義的系統編碼預設，供您在建立編碼作業時使用。 如果您想要將視訊編碼以使用媒體服務進行串流處理，建議使用「彈性資料流」預設。 當您指定這個預設時，媒體編碼器標準將[自動產生位元速率階梯](media-services-autogen-bitrate-ladder-with-mes.md)。 

### <a name="creating-custom-presets-from-samples"></a>從範例建立自訂預設
「媒體服務」可完整支援自訂預設中的所有值，以滿足您的特定編碼需要和需求。 如果您需要自訂編碼預設，應該從本節所提供的下列其中一個系統預設著手，使用它來作為您自訂組態的範本。 如需這些預設中每個元素的意義說明，請參閱[媒體編碼器標準結構描述](media-services-mes-schema.md)主題。  
  
> [!NOTE]
>  使用 4k 編碼的預設值時，您應該取得 `S3` 保留單元類型。 如需詳細資訊，請參閱 [如何調整編碼](https://azure.microsoft.com/documentation/articles/media-services-portal-encoding-units)。  

#### <a name="video-rotation-default-setting-in-presets"></a>預設中的影片旋轉預設設定：
使用「媒體編碼器標準」時，預設會啟用影片旋轉。 如果您的影片是在行動裝置上以「直向」模式錄製的，則這些預設會先將影片旋轉至「橫向」模式，再進行編碼。
 
## <a name="available-presets"></a>可用的預設： 

 [H264 多重位元速率 1080p 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-1080p-Audio-5.1.md) 會產生一組 8 個對齊 GOP 的 MP4 檔案 (範圍從 6000 kbps 到 400 kbps) 和 AAC 5.1 音訊。  
  
 [H264 多重位元速率 1080p](media-services-mes-preset-H264-Multiple-Bitrate-1080p.md) 會產生一組 8 個對齊 GOP 的 MP4 檔案 (範圍從 6000 kbps 到 400 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 16x9 (適用於 iOS)](media-services-mes-preset-H264-Multiple-Bitrate-16x9-for-iOS.md) 會產生一組 8 個對齊 GOP 的 MP4 檔案 (範圍從 8500 kbps 到 200 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 16x9 SD 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD-Audio-5.1.md) 會產生一組 5 個對齊 GOP 的 MP4 檔案 (範圍從 1900 kbps 到 400 kbps) 和 AAC 5.1 音訊。  
  
 [H264 多重位元速率 16x9 SD](media-services-mes-preset-H264-Multiple-Bitrate-16x9-SD.md) 會產生一組 5 個對齊 GOP 的 MP4 檔案 (範圍從 1900 kbps 到 400 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 4K 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4K-Audio-5.1.md) 會產生一組 12 個對齊 GOP 的 MP4 檔案 (範圍從 20000 kbps 到 1000 kbps) 和 AAC 5.1 音訊。  
  
 [H264 多重位元速率 4K](media-services-mes-preset-H264-Multiple-Bitrate-4K.md) 會產生一組 12 個對齊 GOP 的 MP4 檔案 (範圍從 20000 kbps 到 1000 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 4x3 (適用於 iOS)](media-services-mes-preset-H264-Multiple-Bitrate-4x3-for-iOS.md) 會產生一組 8 個對齊 GOP 的 MP4 檔案 (範圍從 8500 kbps 到 200 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 4x3 SD 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD-Audio-5.1.md) 會產生一組 5 個對齊 GOP 的 MP4 檔案 (範圍從 1600 kbps 到 400 kbps) 和 AAC 5.1 音訊。  
  
 [H264 多重位元速率 4x3 SD](media-services-mes-preset-H264-Multiple-Bitrate-4x3-SD.md) 會產生一組 5 個對齊 GOP 的 MP4 檔案 (範圍從 1600 kbps 到 400 kbps) 和立體聲 AAC 音訊。  
  
 [H264 多重位元速率 720p 音訊 5.1](media-services-mes-preset-H264-Multiple-Bitrate-720p-Audio-5.1.md) 會產生一組 6 個對齊 GOP 的 MP4 檔案 (範圍從 3400 kbps 到 400 kbps) 和 AAC 5.1 音訊。  
  
 [H264 多重位元速率 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) 會產生一組 6 個對齊 GOP 的 MP4 檔案 (範圍從 3400 kbps 到 400 kbps) 和立體聲 AAC 音訊。  
  
 [H264 單一位元速率 1080p 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-1080p-Audio-5.1.md) 會產生位元速率為 6750 kbps 的單一 MP4 檔案，而且是 AAC 5.1 音訊。  
  
 [H264 單一位元速率 1080p](media-services-mes-preset-H264-Single-Bitrate-1080p.md) 會產生位元速率為 6750 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率 4K 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-4K-Audio-5.1.md) 會產生位元速率為 18000 kbps 的單一 MP4 檔案，而且是 AAC 5.1 音訊。  
  
 [H264 單一位元速率 4K](media-services-mes-preset-H264-Single-Bitrate-4K.md) 會產生位元速率為 18000 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率 4x3 SD 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-4x3-SD-Audio-5.1.md) 會產生位元速率為 1800 kbps 的單一 MP4 檔案，而且是 AAC 5.1 音訊。  
  
 [H264 單一位元速率 4x3 SD](media-services-mes-preset-H264-Single-Bitrate-4x3-SD.md) 會產生位元速率為 1800 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率 16x9 SD 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-16x9-SD-Audio-5.1.md) 會產生位元速率為 2200 kbps 的單一 MP4 檔案，而且是 AAC 5.1 音訊。  
  
 [H264 單一位元速率 16x9 SD](media-services-mes-preset-H264-Single-Bitrate-16x9-SD.md) 會產生位元速率為 2200 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率 720p 音訊 5.1](media-services-mes-preset-H264-Single-Bitrate-720p-Audio-5.1.md) 會產生位元速率為 4500 kbps 的單一 MP4 檔案，而且是 AAC 5.1 音訊。  
  
 [H264 單一位元速率 720p (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-720p-for-Android.md) 預設會產生位元速率為 2000 kbps 的單一 MP4 檔案，而且是立體聲 AAC。  
  
 [H264 單一位元速率 720p](media-services-mes-preset-H264-Single-Bitrate-720p.md) 會產生位元速率為 4500 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率高品質 SD (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-High-Quality-SD-for-Android.md) 會產生位元速率為 500 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 [H264 單一位元速率低品質 SD (適用於 Android)](media-services-mes-preset-H264-Single-Bitrate-Low-Quality-SD-for-Android.md) 會產生位元速率為 56 kbps 的單一 MP4 檔案，而且是立體聲 AAC 音訊。  
  
 如需與媒體服務編碼器相關的詳細資訊，請參閱[使用 Azure 媒體服務隨選編碼](https://azure.microsoft.com/documentation/articles/media-services-encode-asset/)。
