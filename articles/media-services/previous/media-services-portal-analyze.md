---
title: 使用 Azure 入口網站分析您的媒體 | Microsoft Docs
description: 本主題討論如何使用 Azure 入口網站，以媒體分析媒體處理器 (MP) 處理您的媒體。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 18213fc1-74f5-4074-a32b-02846fe90601
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2017
ms.author: juliako
ms.openlocfilehash: d8c3bb07c88dc96b7ca779ca0f4dfe09052ab290
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666165"
---
# <a name="analyze-your-media-using-the-azure-portal"></a>使用 Azure 入口網站分析您的媒體
> [!NOTE]
> 若要完成此教學課程，您需要 Azure 帳戶。 如需詳細資料，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。 
> 
> 

## <a name="overview"></a>概觀
Azure 媒體服務分析是語音和視覺元件的集合 (具企業規模、相容性、安全性和遍及全球的觸角)，讓組織和企業從其影片檔輕鬆製作出能採取行動的深入見解內容。 如需更為詳細的 Azure 媒體服務分析概觀，請參閱[此主題](media-services-analytics-overview.md)。 

本主題討論如何使用 Azure 入口網站，以媒體分析媒體處理器 (MP) 處理您的媒體。 媒體分析 MP 會產生 MP4 檔案或 JSON 檔案。 如果媒體處理器產生了 MP4 檔案，您可以漸進式下載檔案。 如果媒體處理器產生了 JSON 檔案，您可以從 Azure Blob 儲存體下載檔案。 

## <a name="choose-an-asset-that-you-want-to-analyze"></a>選擇您要分析的資產
1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 在 [設定] 視窗中，選取 [資產]。  
   
    ![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze001.png)
3. 選取您要分析的資產，並按下 [分析] 按鈕。
   
    ![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze002.png)
4. 在 [以媒體分析處理媒體資產] 視窗中，選取處理器。 
   
    本文其餘部分會說明各個處理器的使用原因和方式。 
5. 按下 [建立] 來啟動作業。

## <a name="azure-media-indexer"></a>Azure Media Indexer
**Azure 媒體索引器**媒體處理器可讓您將媒體檔案和內容設為可供搜尋，並產生隱藏式輔助字幕追蹤。 本節會提供一些可為此 MP 指定之選項的詳細資料。

![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze003.png)

### <a name="language"></a>語言
要在多媒體檔案中辨識的自然語言。 例如，英文或西班牙文。 

### <a name="captions"></a>標題
您可以選擇要從內容產生的標題格式。 索引工作可以產生下列格式的隱藏式輔助字幕檔案：  

* **SAMI**
* **TTML**
* **WebVTT**

這些格式的隱藏式輔助字幕 (CC) 檔案可以用來讓具有聽力障礙的人存取音訊和視訊檔案。

### <a name="aib-file"></a>AIB 檔案
如果您想要產生音訊索引 Blob 檔案以便與自訂 SQL Server IFilter 搭配使用，請選取此選項。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/blog/using-aib-files-with-azure-media-indexer-and-sql-server/) 。

### <a name="keywords"></a>關鍵字
如果您想要產生關鍵字 XML 檔案，請選取此選項。 此檔案包含從語音內容擷取的關鍵字，以及關鍵字的頻率和位移資訊。

### <a name="job-name"></a>作業名稱
可讓您識別作業的易記名稱。 [本文](media-services-portal-check-job-progress.md)說明如何監視作業進度。 

### <a name="output-file"></a>輸出檔案
可讓您識別輸出內容的易記名稱。 

## <a name="azure-media-hyperlapse"></a>Azure Media Hyperlapse
Azure Media Hyperlapse 是能夠利用第一人稱視角或運動攝影的內容，來建立流暢縮時影片的 MP。  如需詳細資訊，請參閱[此主題](media-services-hyperlapse-content.md)。 本節會提供一些可為此 MP 指定之選項的詳細資料。

![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze004.png)

### <a name="speed"></a>速度
指定用來讓輸入影片加速的速度。 輸出是輸入影片經過穩定和縮時轉譯的成果。

### <a name="job-name"></a>作業名稱
可讓您識別作業的易記名稱。 [本文](media-services-portal-check-job-progress.md)說明如何監視作業進度。 

### <a name="output-file"></a>輸出檔案
可讓您識別輸出內容的易記名稱。 

## <a name="azure-media-face-detector"></a>Azure 媒體臉部偵測器
**Azure 媒體臉部偵測器** 媒體處理器 (MP) 能讓您透過臉部表情針對對象的參與情形做出計算、追蹤移動，甚至進行量測。 此服務包含兩個功能： 

* **臉部偵測**
  
    臉部偵測能夠找出並追蹤影片中的臉部。 可以同時追蹤多個臉部，隨著對象移動持續進行追蹤，並將時間和位置的中繼資料以 JSON 檔案的格式回傳。 追蹤期間，服務會在人員於螢幕上四處移動時，嘗試為他們的臉部賦予相同的識別碼，就算他們被擋住或暫時離開畫面也一樣。
  
  > [!NOTE]
  > 此服務並不會執行臉部辨識。 臉部離開畫面或被擋住太久的人員，將會在回來時被賦予新的識別碼。
  > 
  > 
* **情緒偵測**
  
    「情緒偵測」是臉部偵測媒體處理器的選擇性元件，它會根據已偵測的臉部，傳回多個情緒屬性的分析，包括快樂、悲傷、恐懼、憤怒等等。 

![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze005.png)

### <a name="detection-mode"></a>偵測模式
處理器可使用下列其中一個模式︰

* 臉部偵測
* 每一人臉情緒偵測
* 彙總情緒偵測

### <a name="job-name"></a>作業名稱
可讓您識別作業的易記名稱。 [本文](media-services-portal-check-job-progress.md)說明如何監視作業進度。 

### <a name="output-file"></a>輸出檔案
可讓您識別輸出內容的易記名稱。 

## <a name="azure-media-motion-detector"></a>Azure 媒體動作偵測器
**Azure 媒體動作偵測器** 媒體處理器 (MP) 能讓您在冗長且平淡的影片中，有效識別出感興趣的區段。 動作偵測可以用來在靜態攝影機資料上識別影片中有動作發生的區段。 它會產生 JSON 檔案，其中包含具有時間戳記的中繼資料，以及發生事件的週框區域。

如果將此技術用於監視器的影片輸出，它將能把動作分類為相關事件及誤判情況 (例如陰影或光源變更)。 這能讓您在不用檢視無止境的不相關事件之下，便能從攝影機輸出產生安全性警示，並從時間相當長的監視影片中擷取感興趣的片段。

![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze006.png)

## <a name="azure-media-video-thumbnails"></a>Azure 媒體視訊縮圖
此處理器可自動選取來源視訊的有趣片段，協助您建立較長視訊的摘要。 針對片長較長的視訊，如果您想要提供精彩內容的快速概觀，這非常有用。 如需詳細資訊和範例，請參閱 [使用 Azure 媒體視訊縮圖建立視訊摘要](media-services-video-summarization.md)

![分析影片](./media/media-services-portal-analyze/media-services-portal-analyze008.png)

### <a name="job-name"></a>作業名稱
可讓您識別作業的易記名稱。 [本文](media-services-portal-check-job-progress.md)說明如何監視作業進度。 

### <a name="output-file"></a>輸出檔案
可讓您識別輸出內容的易記名稱。 

## <a name="azure-media-content-moderator"></a>Azure 媒體內容仲裁
此處理器可協助您偵測影片中潛在的成人和猥褻內容。 處理器會自動偵測影片中的畫面和主要畫面格。 它會對主要畫面格是否有成人或猥褻內容評分，並且根據預設閾值建議檢閱。 如需詳細資訊和範例，請參閱[使用 Azure 媒體內容仲裁以仲裁影片](media-services-content-moderation.md)

![仲裁影片](./media/media-services-portal-analyze/media-services-portal-analyze-content-moderator.PNG)

### <a name="version"></a>版本 
使用 "2.0"。

### <a name="mode"></a>Mode
2.0 版忽略 `Mode` 設定。

## <a name="next-steps"></a>後續步驟
檢視媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
