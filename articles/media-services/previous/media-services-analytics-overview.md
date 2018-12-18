---
title: 媒體服務平台上的媒體分析 | Microsoft Docs
description: 媒體分析公開預覽、企業規模的語音和電腦視覺服務集合、相容性、安全性和遍及全球的觸角概觀
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: c56e3781-8510-4f7f-b5ff-a218c1bb6f4c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 06/29/2017
ms.author: milanga;juliako;johndeu
ms.openlocfilehash: 0ac8b9fad35267ceaec5b5acec4722b6005f68a9
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783587"
---
# <a name="media-analytics-on-the-media-services-platform"></a>媒體服務平台上的媒體分析
## <a name="overview"></a>概觀
越來越多組織採用影片做為慣用的媒體，用來訓練員工、連絡客戶與記錄商務功能。 雖然雲端運算可供儲存、串流及存取大型媒體檔案， 但隨著公司影片庫內容的成長，需要同樣有效的方法深入分析內容。 

為了滿足不斷成長的需求，Azure 媒體服務提供 Azure 媒體分析。 媒體分析是一組語音與視覺元件，可讓組織或企業從其影片檔輕鬆地產生能採取行動的見解。 媒體分析服務是使用核心媒體服務平台元件建置而成，因此可隨時進行一天的大規模媒體處理。

有了媒體分析，開發人員可以快速地將進階影片功能帶入應用程式。 其提供的企業環境具備大型組織所需的完整規模、相容性、安全性和遍及全球的觸角。

下圖顯示媒體分析和媒體服務平台的其他主要部分。 

![VoD 工作流程](./media/media-services-analytics-overview/media-services-analytics-overview01.png)

媒體分析的媒體處理器會產生 MP4 檔案或 JSON 檔案。 如果媒體處理器產生 MP4 檔案，您可以漸進式下載檔案。 如果媒體處理器產生 JSON 檔案，您可以從 Azure Blob 儲存體下載檔案。 

## <a name="media-analytics-services"></a>媒體分析服務

### <a name="indexer"></a>索引器
Azure 媒體索引器可讓您的內容可供搜尋，並且產生隱藏式輔助字幕。 相較於舊版，Azure 媒體索引器 2 (預覽版) 速度更快，支援更多的語言。 支援的語言包括英文、西班牙文、法文、德文、義大利文、中文、葡萄牙文和阿拉伯文。 如需詳細資訊和範例，請參閱[利用 Azure 媒體索引器 2 處理影片](media-services-process-content-with-indexer2.md)。
### <a name="hyperlapse"></a>Hyperlapse
Microsoft Hyperlapse 將影像防震與縮時攝影功能結合在一起，能從完整的內容中建立快速、消費性的影片。 除了縮時攝影外，您也可以使用 Hyperlapse，從透過行動電話及攝影機拍攝的晃動影片中產生不晃動的影片。 如需詳細資訊和範例，請參閱 [Hyperlapse 媒體檔案與 Azure 媒體超縮時攝影](media-services-hyperlapse-content.md)。
### <a name="motion-detector"></a>動作偵測
您可以使用動作偵測在影片中偵測背景靜止的動作。 如此一來，便能夠檢查監視攝影機所偵測到的動作事件中是否有誤判。 如需詳細資訊和範例，請參閱 [Azure 媒體分析的動作偵測](media-services-motion-detection.md)。
### <a name="face-detector"></a>臉部偵測
透過使用臉部偵測，您便能偵測人臉及其表情，包括快樂、悲傷及驚喜。 這項服務有數個實用的產業應用程式 (將於稍後說明)，包括彙總並分析事件參與人員的反應。 如需詳細資訊和範例，請參閱 [Azure 媒體分析的臉部和情緒偵測](media-services-face-and-emotion-detection.md)。
### <a name="video-summarization"></a>影片摘要
視訊摘要可自動選取來源視訊的有趣片段，協助您建立較長視訊的摘要。 針對片長較長的影片，如果您想要提供精彩內容的快速概觀，此功能非常有用。 如需詳細資訊和範例，請參閱[使用 Azure 媒體影片縮圖建立影片摘要](media-services-video-summarization.md)。
### <a name="optical-character-recognition"></a>光學字元辨識
Azure 媒體 OCR (光學字元辨識) 可讓您將影片檔中的文字內容轉換成可編輯、可搜尋的數位文字。 接著您便可從媒體的影片訊號中自動擷取出有意義的中繼資料。
### <a name="scalable-face-redaction"></a>可調整式臉部修訂
Azure 媒體修訂器是媒體分析媒體處理器，可在雲端提供可調整式臉部修訂。 您可以使用臉部修訂功能修改影片，將所選人物的臉部變得模糊。 您可能會想要在新聞媒體中或涉及公共安全時，使用臉部修訂服務。 當要手動修訂多個臉部時，即使片長只有數分鐘，也會花上數小時修訂，但若使用此服務，則只需要幾個簡單的步驟就能完成臉部修訂。 如需詳細資訊，請參閱[使用 Azure 媒體分析修訂臉部](media-services-face-redaction.md)一文。
### <a name="content-moderation"></a>內容仲裁
Azure 內容仲裁可讓您針對影片使用機器輔助仲裁。 例如，您可能想偵測影片中潛在的成人和辛辣內容，並由您的人力仲裁小組檢閱加上旗標的內容。 但手動仲裁不需要的影片內容是很耗費時間和成本的工作。 透過此服務和相關聯的檢閱工具，您可以結合機器輔助仲裁和人機互動 (human-in-the-loop) 功能，以有效率且符合成本效益的方式獲得最佳結果。 若要進一步了解，請參閱[使用 Azure 內容仲裁處理您的視訊](media-services-content-moderation.md)一文。

## <a name="common-scenarios"></a>常見案例
媒體分析可協助組織和企業從影片中得到新的見解，並更有效地管理大量的影片內容。 以下是幾個案例︰

* **客服中心**。 即使社交媒體出現，但客服中心仍能協助處理大量的客戶服務交易。 客服中心的語音資料經編碼後可成為大量的客戶資訊，企業可分析這些資訊以提高客戶滿意度。 組織可使用媒體索引器擷取文字，及建置搜尋索引與儀表板。 接著便可以從一般客訴、客訴來源及其他相關資料中獲取情報。
* **仲裁使用者產生的內容**。 從新聞媒體到警察局，許多組織都有對外公開的入口網站可接受使用者產生的媒體，例如影片和影像。 內容的數量可能會因為未預期事件而激增。 在這些狀況下，很難有效地人工審查內容是否適當。 客戶可以依賴內容仲裁服務將重點放在適當的內容。
* **監視錄影**. 隨著 IP 攝影機的使用愈來愈普遍，監視影片的存量也隨之成長。 手動檢閱監視視訊既耗時又容易發生人為錯誤。 媒體分析提供數種服務，例如動作偵測、臉部偵測和 Hyperlapse，讓審查、管理和建立衍生物件的程序變得更容易。

## <a name="media-analytics-media-processors"></a>媒體分析媒體處理器
本節列出所有媒體分析媒體處理器，並示範如何使用 .NET 或 REST 來取得媒體處理器 (MP) 物件。

### <a name="mp-names"></a>MP 名稱
* Azure 媒體索引器 2 預覽
* Azure Media Indexer
* Azure Media Hyperlapse
* Azure 媒體臉部偵測器
* Azure 媒體動作偵測器
* Azure 媒體視訊縮圖
* Azure 媒體 OCR
* Azure 媒體內容仲裁

### <a name="net"></a>.NET
下列函數會採用其中一個指定的 MP 名稱，並傳回 MP 物件。

    static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
    {
        var processor = _context.MediaProcessors
            .Where(p => p.Name == mediaProcessorName)
            .ToList()
            .OrderBy(p => new Version(p.Version))
            .LastOrDefault();

        if (processor == null)
            throw new ArgumentException(string.Format("Unknown media processor",
                                                       mediaProcessorName));

        return processor;
    }


### <a name="rest"></a>REST
要求：

    GET https://media.windows.net/api/MediaProcessors()?$filter=Name%20eq%20'Azure%20Media%20OCR' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    User-Agent: Microsoft ADO.NET Data Services
    Authorization: Bearer <token>
    x-ms-version: 2.12
    Host: media.windows.net

回應：

    . . .

    {  
       "odata.metadata":"https://media.windows.net/api/$metadata#MediaProcessors",
       "value":[  
          {  
             "Id":"nb:mpid:UUID:074c3899-d9fb-448f-9ae1-4ebcbe633056",
             "Description":"Azure Media OCR",
             "Name":"Azure Media OCR",
             "Sku":"",
             "Vendor":"Microsoft",
             "Version":"1.1"
          }
       ]
    }

## <a name="demos"></a>示範
請參閱 [Azure 媒體分析示範](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)。

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-articles"></a>相關文章
請參閱[媒體服務分析公告](https://azure.microsoft.com/blog/introducing-azure-media-analytics/)。

<!-- Images -->

[overview]: ./media/media-services-video-on-demand-workflow/media-services-video-on-demand.png

## <a name="next-steps"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
