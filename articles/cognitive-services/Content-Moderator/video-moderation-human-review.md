---
title: 使用人工審核進行影片仲裁 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用電腦輔助影片審核和人力審查工具來審核不當的內容
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: conceptual
ms.date: 01/20/2018
ms.author: sajagtap
ms.openlocfilehash: 33956e1986265b36f6df9a2d19909cb974bd3197
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222075"
---
# <a name="video-moderation-with-human-review"></a>使用人工審核進行影片仲裁

使用 Content Moderator 的電腦輔助[影片審核](video-moderation-api.md)和[人力審查工具](Review-Tool-User-Guide/human-in-the-loop.md)來審核成人 (明確) 和不雅 (暗示性) 內容的影片和文字記錄，以取得適合您業務的最佳結果。

## <a name="video-trained-classifier-preview"></a>經過影片訓練的分類器 (預覽)

利用經過影像訓練的模型或經過影片訓練的模型來進行電腦輔助影片分類。 與經過影像訓練的分類器不同，Microsoft 的成人和不雅影片分類器是以影片進行訓練。 這個方法會產生較佳的相符品質。

## <a name="shot-detection"></a>分鏡偵測

在輸出分類詳細資料時，其他影片智慧對分析影片的彈性有助幫助。 不只是輸出畫面，Microsoft 的影片審查服務還會提供分鏡層級的資訊。 您現在可以選擇在分鏡層級和畫面層級分析您的影片。
 
## <a name="key-frame-detection"></a>主畫面偵測

影片審查服務只會識別和輸出可能完整 (良好) 的畫面，而不會定期輸出畫面。 此功能能夠有效地產生畫面，以便進行畫面層級的成人和不雅分析。

下列摘錄顯示部分的回應，其中包含潛在分鏡、主畫面及成人和不雅分數：

    "fragments": [
    {
      "start": 0,
      "duration": 18000
    },
    {
      "start": 18000,
      "duration": 3600,
      "interval": 3600,
      "events": [
        [
          {
            "reviewRecommended": false,
            "adultScore": 0.00001,
            "racyScore": 0.03077,
            "index": 5,
            "timestamp": 18000,
            "shotIndex": 0
          }
        ]
      ]
    },
    {
      "start": 18386372,
      "duration": 119149,
      "interval": 119149,
      "events": [
        [
          {
            "reviewRecommended": true,
            "adultScore": 0.00000,
            "racyScore": 0.91902,
            "index": 5085,
            "timestamp": 18386372,
            "shotIndex": 62
          }
        ]
      ]


## <a name="visualization-for-human-reviews"></a>人力審核的視覺效果

如需更細微的案例，企業需要人力審核解決方案，以便轉譯影片、其畫面和機器指派的標記。 審核影片和畫面的人力審核者會取得完整的見解觀點、變更標記，以及提出他們的決策。

![影片審查工具預設檢視](images/video-review-default-view.png)

## <a name="player-view-for-video-level-review"></a>影片層級審核的播放程式檢視

透過可顯示潛在成人和不雅畫面的影片播放程式檢視，促成影片層級的二進位決策。 人力審查者會利用各種速度選項來瀏覽影片，進而檢查場景。 他們可藉由切換標記來確認其決策。

![影片審查工具播放程式檢視](images/video-review-player-view.PNG)

## <a name="frames-view-for-detailed-reviews"></a>詳細審核的畫面檢視

透過以畫面為基礎的檢視，促成逐畫面分析的詳細影片審核。 人力審查者可審查及選取一或多個畫面，並切換標記以確認他們的決策。 下一個選擇性步驟校訂具冒犯性的畫面或內容。

![影片審查工具畫面檢視](images/video-review-frames-view-apply-tags.PNG)

## <a name="transcript-moderation"></a>文字記錄審查

對於帶有語音的影片，通常也需要對具冒犯性的語音進行審查。 您可使用 Azure 媒體索引器服務將語音轉換成文字，並使用 Content Moderator 的審查 API 來提交文字記錄，以便在審查工具內進行文字審核。

![影片審查工具的文字記錄檢視](images/video-review-transcript-view.png)

## <a name="next-steps"></a>後續步驟

開始使用[影片審查快速入門](video-moderation-api.md)。 

了解如何從已審核的輸出為您的人力審查者產生[影片審查](video-reviews-quickstart-dotnet.md)。

將[影片文字記錄審核](video-transcript-reviews-quickstart-dotnet.md)新增至您的影片審核。

查看有關如何開發[完整影片審查解決方案](video-transcript-moderation-review-tutorial-dotnet.md)的詳細教學課程。 
