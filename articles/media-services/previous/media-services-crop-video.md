---
title: 如何利用媒體編碼器標準裁剪影片 - Azure | Microsoft Docs
description: 本文說明如何利用 Media Encoder Standard 裁剪影片。
services: media-services
documentationcenter: ''
author: anilmur
manager: cfowler
editor: ''
ms.assetid: 7628f674-2005-4531-8b61-d7a4f53e46ba
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 08/09/2017
ms.author: anilmur;juliako;
ms.openlocfilehash: 2592316481c9e265fbae20b832beb21ae4905b14
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782947"
---
# <a name="crop-videos-with-media-encoder-standard"></a>以 Media Encoder Standard 裁剪影片
若要裁剪輸入影片，您可以使用 Media Encoder Standard (MES)。 裁剪是指在視訊畫面內選取矩形視窗，並只編碼該視窗內之像素的程序。 下圖有助於說明此程序。

![裁剪影片](./media/media-services-crop-video/media-services-crop-video01.png)

假設做為輸入之影片的解析度為 1920x1080 像素 (16:9 外觀比例)，不過因為左側和右側有黑色長條 (圓柱方塊)，導致包含使用中影片的區域只有 4:3 視窗或 1440x1080 像素。 您可以使用 MES 來裁剪或刪去黑色長條，再編碼 1440x1080 區域。

MES 中的裁剪是前置處理階段，因此編碼預設值中的裁剪參數適用於原始輸入影片。 編碼是後續的階段，因此寬度/高度設定適用於 *經過前置處理* 的影片，而非原始影片。 在設計預設值時，您需要執行下列動作︰(a) 根據原始的輸入影片選取裁剪參數；(b) 根據裁剪後的影片選取編碼設定。 如果編碼設定與裁剪後的影片不符，輸出將會不如預期。

[下列](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) 主題說明如何使用 MES 建立編碼編碼，以及如何為編碼編碼指定自訂預設值。 

## <a name="creating-a-custom-preset"></a>建立自訂預設值
在下圖所示的範例中：

1. 原始輸入為 1920x1080
2. 我們需要將影片裁剪為 1440x1080，使其能位於輸入畫面的中央
3. 這表示 X 位移為 (1920 – 1440)/2 = 240，Y 位移為零
4. 裁剪矩形的高度和寬度分別為 1440 和 1080
5. 在編碼階段中，我們的任務是產生三層分別為 1440x1080、960x720 和 480x360 的解析度

### <a name="json-preset"></a>JSON 預設值
    {
      "Version": 1.0,
      "Sources": [
        {
          "Streams": [],
          "Filters": {
            "Crop": {
                "X": 240,
                "Y": 0,
                "Width": 1440,
                "Height": 1080
            }
          },
          "Pad": true
        }
      ],
      "Codecs": [
        {
          "KeyFrameInterval": "00:00:02",
          "H264Layers": [
            {
              "Profile": "Auto",
              "Level": "auto",
              "Bitrate": 3400,
              "MaxBitrate": 3400,
              "BufferWindow": "00:00:05",
              "Width": 1440,
              "Height": 1080,
              "BFrames": 3,
              "ReferenceFrames": 3,
              "AdaptiveBFrame": true,
              "Type": "H264Layer",
              "FrameRate": "0/1"
            },
            {
              "Profile": "Auto",
              "Level": "auto",
              "Bitrate": 2250,
              "MaxBitrate": 2250,
              "BufferWindow": "00:00:05",
              "Width": 960,
              "Height": 720,
              "BFrames": 3,
              "ReferenceFrames": 3,
              "AdaptiveBFrame": true,
              "Type": "H264Layer",
              "FrameRate": "0/1"
            },
            {
              "Profile": "Auto",
              "Level": "auto",
              "Bitrate": 1250,
              "MaxBitrate": 1250,
              "BufferWindow": "00:00:05",
              "Width": 480,
              "Height": 360,
              "BFrames": 3,
              "ReferenceFrames": 3,
              "AdaptiveBFrame": true,
              "Type": "H264Layer",
              "FrameRate": "0/1"
            }
          ],
          "Type": "H264Video"
        },
        {
          "Profile": "AACLC",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128,
          "Type": "AACAudio"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
          "Format": {
            "Type": "MP4Format"
          }
        }
      ]
    }


## <a name="restrictions-on-cropping"></a>裁剪的限制
裁剪功能僅供手動操作。 您需要將輸入影片載入適當的編輯工具，以便選取感興趣的畫面、放置游標以判斷裁剪矩形的位移、判斷針對特定影片微調的編碼預設值等。這項功能的本意並非實現如自動偵測及移除輸入影片之黑邊等這類功能。

裁剪功能受到以下限制的約束。 如果不符合以下情況，編碼工作可能會失敗或產生非預期的輸出。

1. 裁剪矩形的座標和大小不得超出輸入影片
2. 如前文所述，編碼設定的寬度和高度必須與裁剪後的影片對應
3. 裁剪適用於以橫向模式擷取的影片 (亦即，不適用於以垂直方向或直立模式手持之智慧型手機錄製的影片)
4. 最適合用於以方形像素擷取的漸進式影片

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>後續步驟
請參閱 Azure 媒體服務學習途徑，以了解 AMS 所提供的強大功能。  

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
