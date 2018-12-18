---
title: 使用 Azure 媒體服務在雲端中編碼 | Microsoft Docs
description: 本主題說明使用 Azure 媒體服務時的編碼程序
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 04/21/2018
ms.author: juliako
ms.openlocfilehash: 69c5516ee503d774b143bb2d83f09ea863a00b31
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091544"
---
# <a name="encoding-with-azure-media-services"></a>使用 Azure 媒體服務進行編碼

Azure 媒體服務可讓您將高品質數位媒體檔案編碼成可在各種不同的瀏覽器和裝置上播放的格式。 例如，您可能會想要串流處理 Apple 的 HLS 或 MPEG DASH 格式的內容。 媒體服務也可讓您分析視訊或音訊內容。 本主題會指導您如何使用媒體服務 v3 將內容編碼。

若要使用媒體服務 v3 來編碼，您需要建立轉換和作業。 轉換可定義編碼的設定和輸出配方，作業則是配方的執行個體。 如需詳細資訊，請參閱[轉換和作業](transform-concept.md)

使用 Azure 媒體服務來編碼時，需使用預設來告訴編碼器應該如何處理輸入媒體檔案。 例如，您可以指定所編碼內容中需要的視訊解析度及/或音訊聲道數目。 

您可以使用依據業界最佳做法所建議的其中一個內建預設來快速開始，也可以選擇建置以特定案例或裝置需求為標的的自訂預設。 

如需編碼器的詳細資料，請參閱 [OpenAPI 規格](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/preview/2018-03-30-preview)。 

## <a name="built-in-presets"></a>內建預設

媒體服務目前支援下列內建編碼預設：  

|**預設名稱**|**案例**|**詳細資料**|
|---|---|---|
|**AudioAnalyzerPreset**|分析音訊|此預設會套用一組預先定義的 AI 型分析作業，包括語音轉譯。 此預設目前支援處理具有單一音訊播放軌的內容。<br/>您可以使用「語言標記-區域」的 BCP-47 格式 (例如 'en-US')，為輸入中的音訊承載指定語言。 所支援語言的清單有 'en-US'、'en-GB'、'es-ES'、'es-MX'、'fr-FR'、'it-IT'、'ja-JP'、'pt-BR'、'zh-CN'。|
|**VideoAnalyzerPreset**|分析音訊和視訊|從音訊和視訊擷取見解 (豐富的中繼資料)，並輸出 JSON 格式檔案。 您可以指定在處理視訊檔案時，是否只想擷取音訊見解。 如需詳細資訊，請參閱[分析視訊](analyze-videos-tutorial-with-api.md)。|
|**BuiltInStandardEncoderPreset**|串流|用來設定內建預設，以供使用標準編碼器為輸入視訊編碼時使用。 <br/>目前支援的預設如下：<br/>**EncoderNamedPreset.AdaptiveStreaming** (建議)。 如需詳細資訊，請參閱[自動產生位元速率階梯](autogen-bitrate-ladder.md)。<br/>**EncoderNamedPreset.AACGoodQualityAudio** - 會產生只包含立體聲音訊 (以 192 kbps 編碼) 的單一 MP4 檔案。<br/>**EncoderNamedPreset.H264MultipleBitrate1080p** - 會產生一組 8 個對齊 GOP 的 MP4 檔案 (範圍從 6000 kbps 到 400 kbps) 和立體聲 AAC 音訊。 解析度起自 1080p，下至 360p。<br/>**EncoderNamedPreset.H264MultipleBitrate720p** - 會產生一組 6 個對齊 GOP 的 MP4 檔案 (範圍從 3400 kbps 到 400 kbps) 和立體聲 AAC 音訊。 解析度起自 720p，下至 360p。<br/>**EncoderNamedPreset.H264MultipleBitrateSD** - 會產生一組 5 個對齊 GOP 的 MP4 檔案 (範圍從 1600 kbps 到 400 kbps) 和立體聲 AAC 音訊。 解析度起自 480p，下至 360p。<br/><br/>如需詳細資訊，請參閱[上傳、編碼和串流檔案](stream-files-tutorial-with-api.md)。|
|**StandardEncoderPreset**|串流|說明在使用標準編碼器為輸入視訊編碼時，所要使用的設定。 <br/>在自訂轉換預設時，請使用此預設。 如需詳細資訊，請參閱[如何自訂轉換預設](customize-encoder-presets-how-to.md)。|

## <a name="custom-presets"></a>自訂預設

「媒體服務」可完整支援自訂預設中的所有值，以滿足您的特定編碼需要和需求。 在自訂轉換預設時，請使用 **StandardEncoderPreset** 預設。 如需詳細說明和範例，請參閱[如何自訂編碼器預設](customize-encoder-presets-how-to.md)。

## <a name="scaling-encoding-in-v3"></a>在 v3 中調整編碼

目前，客戶必須使用 Azure 入口網站或 AMS v2 API 來設定 RU (如[調整媒體處理](../previous/media-services-scale-media-processing-overview.md)所述)。 

## <a name="next-steps"></a>後續步驟

### <a name="tutorials"></a>教學課程

下列教學課程說明如何使用媒體服務將內容編碼：

* [使用 Azure 媒體服務來上傳、編碼和串流](stream-files-tutorial-with-api.md)
* [使用 Azure 媒體服務分析影片](analyze-videos-tutorial-with-api.md)

### <a name="code-samples"></a>程式碼範例

下列程式碼範例包含的程式碼會說明如何使用媒體服務來編碼：

* [.NET Core](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/tree/master/NETCore)
* [Azure CLI](https://github.com/Azure/azure-docs-cli-python-samples/tree/master/media-services)

### <a name="sdks"></a>SDK

您可以使用下列受支援的媒體服務 v3 SDK 來將內容編碼。

* [Azure CLI](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
* [REST](https://docs.microsoft.com/rest/api/media/transforms)
* [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/mediaservices/management?view=azure-dotnet)
* [Java](https://docs.microsoft.com/java/api/overview/azure/mediaservices)
* [Python](https://docs.microsoft.com/python/api/overview/azure/media-services?view=azure-python)

