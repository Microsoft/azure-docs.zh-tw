---
title: Azure 隨選媒體編碼器的比較 | Microsoft Docs
description: 本主題比較**媒體編碼器標準**和**媒體編碼器高階工作流程**的編碼功能。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: a79437c0-4832-423a-bca8-82632b2c47cc
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/05/2017
ms.author: juliako;anilmur
ms.openlocfilehash: cf420c6b5f72b2109016bdb8b86d6cfcc506f4e5
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34639515"
---
# <a name="comparison-of-azure-on-demand-media-encoders"></a>Azure 隨選媒體編碼器的比較

本主題比較**媒體編碼器標準**和**媒體編碼器高階工作流程**的編碼功能。

## <a name="video-and-audio-processing-capabilities"></a>視訊和音訊處理功能

下表比較媒體編碼器標準 (MES) 和媒體編碼器高階工作流程 (MEPW) 之間的功能。 

|功能|Media Encoder Standard|媒體編碼器高階工作流程|
|---|---|---|
|編碼時套用條件式邏輯<br/>(例如，如果輸入是 HD，則編碼 5.1 音訊)|否|yes|
|隱藏式輔助字幕|否|[是](media-services-premium-workflow-encoder-formats.md#closed_captioning)|
|[Dolby® Professional Loudness Correction](http://www.dolby.com/us/en/technologies/dolby-professional-loudness-solutions.pdf)<br/> with Dialogue Intelligence™|否|yes|
|非交錯顯示、反轉電影|基本|廣播品質|
|偵測並移除黑色框線 <br/>(垂直黑邊、上下黑邊)|否|yes|
|縮圖產生|[是](media-services-dotnet-generate-thumbnail-with-mes.md)|[是](media-services-media-encoder-premium-workflow-tutorials.md#thumbnails_to__multibitrate_MP4)|
|裁剪/修剪和拼接視訊|[是](media-services-advanced-encoding-with-mes.md#trim_video)|yes|
|音訊或視訊的覆疊|[是](media-services-advanced-encoding-with-mes.md#overlay)|[是](media-services-media-encoder-premium-workflow-multiplefilesinput.md#example-1--overlay-an-image-on-top-of-the-video)|
|圖形的覆疊|從影像來源|從影像和文字來源|
|多重音訊語言曲目|限制|[是](media-services-media-encoder-premium-workflow-multiplefilesinput.md#example-2--multiple-audio-language-encoding)|

## <a id="billing"></a>每個編碼器所使用的計費計量表
| 媒體處理器名稱 | 適用的價格 | 注意 |
| --- | --- | --- |
| **Media Encoder Standard** |ENCODER |在 ENCODER 資料行底下，「編碼工作」會以[這裡][1]指定的費率，根據所有產生來輸出之媒體檔案的總持續時間 (以分鐘為單位) 計費。 |
| **Media Encoder Premium Workflow** |PREMIUM ENCODER |在 PREMIUM ENCODER 資料行底下，「編碼工作」會以[這裡][1]指定的費率，根據所有產生來輸出之媒體檔案的總持續時間 (以分鐘為單位) 計費。 |

## <a name="input-containerfile-formats"></a>輸入容器/檔案格式
| 輸入容器/檔案格式 | Media Encoder Standard | 媒體編碼器高階工作流程 |
| --- | --- | --- |
| Adobe® Flash® F4V |yes |yes |
| MXF/SMPTE 377M |yes |yes |
| GXF |yes |yes |
| MPEG-2 傳輸資料流 |yes |yes |
| MPEG-2 程式資料流 |yes |yes |
| MPEG-4/MP4 |yes |yes |
| Windows Media/ASF |yes |yes |
| AVI (未壓縮 8 位元/10 位元) |yes |yes |
| 3GPP/3GPP2 |yes |否 |
| Smooth Streaming 檔案格式 (PIFF 1.3) |yes |否 |
| [Microsoft Digital Video Recording(DVR-MS)](https://msdn.microsoft.com/library/windows/desktop/dd692984) |yes |否 |
| Matroska/WebM |yes |否 |
| QuickTime (.mov) |yes |否 |

## <a name="input-video-codecs"></a>輸入視訊轉碼器
| 輸入視訊轉碼器 | Media Encoder Standard | 媒體編碼器高階工作流程 |
| --- | --- | --- |
| AVC 8 位元/10 位元，高達 4:2:2，包括 AVCIntra |8 位元 4:2:0 和 4:2:2 |yes |
| Avid DNxHD (使用 MXF) |yes |yes |
| DVCPro/DVCProHD (使用 MXF) |yes |yes |
| JPEG2000 |yes |yes |
| MPEG-2 (高達 422 Profile 和 High Level，包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs ® 和 D10 等變種) |最高 422 設定檔 |yes |
| MPEG-1 |yes |yes |
| Windows Media 視訊/VC-1 |yes |yes |
| Canopus HQ/HQX |否 |否 |
| Mpeg-4 第 2 部分 |yes |否 |
| [Theora](https://en.wikipedia.org/wiki/Theora) |yes |否 |
| Apple ProRes 422 |yes |否 |
| Apple ProRes 422 LT |yes |否 |
| Apple ProRes 422 HQ |yes |否 |
| Apple ProRes Proxy |yes |否 |
| Apple ProRes 4444 |yes |否 |
| Apple ProRes 4444 XQ |yes |否 |
| HEVC/H.265|主要設定檔|主要和主要 10 設定檔|

## <a name="input-audio-codecs"></a>輸入音訊轉碼器
| 輸入音訊轉碼器 | Media Encoder Standard | 媒體編碼器高階工作流程 |
| --- | --- | --- |
| AES (SMPTE 331M 和 302M，AES3-2003) |否 |yes |
| Dolby® E |否 |yes |
| Dolby® Digital (AC3) |否 |yes |
| Dolby® Digital Plus (E-AC3) |否 |yes |
| AAC (AAC-LC、AAC-HE 和 AAC-HEv2；高達 5.1) |yes |yes |
| MPEG Layer 2 |yes |yes |
| MP3 (MPEG-1 音訊層 3) |yes |yes |
| Windows Media 音訊 |yes |yes |
| WAV/PCM |yes |yes |
| [FLAC](https://en.wikipedia.org/wiki/FLAC)</a> |yes |否 |
| [Opus](https://en.wikipedia.org/wiki/Opus_\(audio_format\)) |yes |否 |
| [Vorbis](https://en.wikipedia.org/wiki/Vorbis)</a> |yes |否 |

## <a name="output-containerfile-formats"></a>輸出容器/檔案格式
| 輸出容器/檔案格式 | Media Encoder Standard | 媒體編碼器高階工作流程 |
| --- | --- | --- |
| Adobe® Flash® F4V |否 |yes |
| MXF (OP1a、XDCAM 和 AS02) |否 |yes |
| DPP (包括 AS11) |否 |yes |
| GXF |否 |yes |
| MPEG-4/MP4 |yes |yes |
| MPEG-TS |yes |yes |
| Windows Media/ASF |否 |yes |
| AVI (未壓縮 8 位元/10 位元) |否 |yes |
| Smooth Streaming 檔案格式 (PIFF 1.3) |否 |yes |

## <a name="output-video-codecs"></a>輸出視訊轉碼器
| 輸出視訊轉碼器 | Media Encoder Standard | Media Encoder Premium Workflow |
| --- | --- | --- |
| AVC (H.264；8 位元；高達 High Profile、Level 5.2；4K Ultra HD；AVC Intra) |僅限 8 位元 4:2:0 |yes |
| HEVC (H.265；8 位元與 10 位元；)  |否 |yes |
| Avid DNxHD (使用 MXF) |否 |yes |
| MPEG-2 (高達 422 Profile 和 High Level，包括 XDCAM、XDCAM HD、XDCAM IMX、CableLabs ® 和 D10 等變種) |否 |yes |
| MPEG-1 |否 |yes |
| Windows Media 視訊/VC-1 |否 |yes |
| JPEG 縮圖建立 |yes |yes |
| PNG 縮圖建立 |yes |yes |
| BMP 縮圖建立 |yes |否 |

## <a name="output-audio-codecs"></a>輸出音訊轉碼器
| 輸出音訊轉碼器 | Media Encoder Standard | 媒體編碼器高階工作流程 |
| --- | --- | --- |
| AES (SMPTE 331M 和 302M，AES3-2003) |否 |yes |
| Dolby® Digital (AC3) |否 |yes |
| Dolby® Digital Plus (E-AC3) 高達 7.1 |否 |yes |
| AAC (AAC-LC、AAC-HE 和 AAC-HEv2；高達 5.1) |yes |yes |
| MPEG Layer 2 |否 |yes |
| MP3 (MPEG-1 音訊層 3) |否 |yes |
| Windows Media 音訊 |否 |yes |

>[!NOTE]
>如果編碼成 Dolby® Digital (AC3)，則輸出只能寫入到 ISO MP4 檔案。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>相關文章
* [透過自訂 Media Encoder Standard 預設值來執行進階編碼工作](media-services-custom-mes-presets-with-dotnet.md)
* [配額和限制](media-services-quotas-and-limitations.md)

<!--Reference links in article-->
[1]: http://azure.microsoft.com/pricing/details/media-services/
