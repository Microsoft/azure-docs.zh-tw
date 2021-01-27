---
title: Azure 媒體服務中的 LiveEvent 低延遲設定
description: 本主題概要說明 LiveEvent 低延遲設定，並示範如何設定低延遲。
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 023b0f4d7f0367882e0a5bb2be89c485c18bc03c
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98897827"
---
# <a name="live-event-low-latency-settings"></a>實況活動低延遲設定

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

本文說明如何在[實況活動](/rest/api/media/liveevents)上設定低延遲。 它也會討論您在各種播放器中使用低延遲設定時會看到的典型結果。 結果會隨著 CDN 和網路延遲而有所不同。

若要使用新的 **LowLatency** 功能，您可以在 **LiveEvent** 上將 **StreamOptionsFlag** 設定為 **LowLatency**。 在建立 [LiveOutput](/rest/api/media/liveoutputs) 以便播放 HLS 時，請將 [LiveOutput.Hls.fragmentsPerTsSegment](/rest/api/media/liveoutputs/create#hls) 設定為 1。 串流啟動並執行後，您可以使用 [Azure 媒體播放機](https://ampdemo.azureedge.net/) (AMP 示範頁面) ，並將播放選項設定為使用「低延遲啟發學習法設定檔」。

> [!NOTE]
> 目前，Azure 媒體播放機中的 LowLatency HeuristicProfile 是設計用來播放以 MPEG 虛線通訊協定表示的串流，其中 CSF 或 CMAF 格式 (例如 `format=mdp-time-csf` 或 `format=mdp-time-cmaf`) 。 

下列 .NET 範例會示範如何在 **LiveEvent** 上設定 **LowLatency**：

```csharp
LiveEvent liveEvent = new LiveEvent(
            location: mediaService.Location, 
            description: "Sample LiveEvent for testing",
            vanityUrl: false,
            encoding: new LiveEventEncoding(
                        // Set this to Standard to enable a transcoding LiveEvent, and None to enable a pass-through LiveEvent
                        encodingType:LiveEventEncodingType.None, 
                        presetName:null
                    ),
            input: new LiveEventInput(LiveEventInputProtocol.RTMP,liveEventInputAccess), 
            preview: liveEventPreview,
            streamOptions: new List<StreamOptionsFlag?>()
            {
                // Set this to Default or Low Latency
                // To use low latency optimally, you should tune your encoder settings down to 1 second "Group Of Pictures" (GOP) length instead of 2 seconds.
                StreamOptionsFlag.LowLatency
            }
        );
```                

參閱完整範例：[MediaV3LiveApp](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs#L126)。

## <a name="live-events-latency"></a>實況活動延遲

下表會顯示媒體服務中延遲 (啟用 LowLatency 旗標時) 的典型結果，其測量方式為從發佈摘要抵達服務到檢視器在播放器上看見播放之間的時間。 若要以最佳方式使用低延遲，您應該將您的編碼器設定下調為 1 秒「圖片群組」(GOP) 長度。 使用較高的 GOP 長度時，您可在相同的畫面播放速率下，將頻寬耗用量降至最低並減少位元速率。 這對動作較少的影片特別有幫助。

### <a name="pass-through"></a>傳遞 

||已啟用 2 秒 GOP 低延遲|已啟用 1 秒 GOP 低延遲|
|---|---|---|
|**AMP 中的 DASH**|10 秒|8 秒|
|**原生 iOS 播放程式上的 HLS**|14 秒|10 秒|

### <a name="live-encoding"></a>即時編碼

||已啟用 2 秒 GOP 低延遲|已啟用 1 秒 GOP 低延遲|
|---|---|---|
|**AMP 中的 DASH**|14 秒|10 秒|
|**原生 iOS 播放程式上的 HLS**|18 秒|13 秒|

> [!NOTE]
> 端對端延遲可能會因區域網路狀況或引進 CDN 快取層而有所不同。 建議您測試實際的設定。

## <a name="next-steps"></a>後續步驟

- [即時串流概觀](live-streaming-overview.md)
- [即時串流教學課程](stream-live-tutorial-with-api.md)
