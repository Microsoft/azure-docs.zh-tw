---
title: 提交 Azure Media Clipper 剪輯作業 | Microsoft Docs
description: 從 Azure Media Clipper 提交剪輯作業的步驟
services: media-services
keywords: clip;subclip;encoding;media;剪輯;子剪輯;編碼;媒體
author: dbgeorge
manager: jasonsue
ms.author: dwgeo
ms.date: 11/10/2017
ms.topic: article
ms.service: media-services
ms.openlocfilehash: 8372c405087c0dc7a000a65265bb99c395c3a8d6
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783177"
---
# <a name="submit-clipping-jobs-from-azure-media-clipper"></a>從 Azure Media Clipper 提交剪輯作業
Azure Media Clipper 需要實作 **submitSubclipCallback** 方法，以處理剪輯作業提交。 此函式是用於將 Clipper 輸出的 HTTP POST 實作至 Web 服務。 此 Web 服務是您可以提交編碼作業的位置。 Clipper 的輸出是為轉譯作業的媒體編碼器標準編碼預設，或是動態資訊清單篩選條件呼叫的 REST API 裝載。 需要這個傳遞模型，因為媒體服務帳戶認證在用戶端瀏覽器中不安全。

下列順序圖表說明瀏覽器用戶端、您的 Web 服務與 Azure 媒體服務之間的工作流程：![Azure Media Clipper 順序圖表](media/media-services-azure-media-clipper-submit-job/media-services-azure-media-clipper-sequence-diagram.PNG)

上圖中的四個實體為：使用者的瀏覽器、您的 Web 服務、裝載 Clipper 資源的 CDN 端點，以及 Azure 媒體服務。 當使用者瀏覽您的網頁時，該網頁就會從裝載 CDN 端點取得 Clipper JavaScript 和 CSS 資源。 終端使用者會從他們的瀏覽器設定剪輯作業或動態資訊清單篩選建立呼叫。 當終端使用者提交作業或篩選建立呼叫時，瀏覽器會將作業承載放置於您必須部署的 Web 服務中。 此 Web 服務最後會使用您的媒體服務帳戶認證，將剪輯作業或篩選建立呼叫提交至 Azure 媒體服務。

下列程式碼範例會說明範例 **submitSubclipCallback** 方法。 在此方法中，您會實作媒體編碼器標準編碼預設的 HTTP POST。 如果 POST 成功 (**結果**)，則**承諾**已解決，否則就會拒絕且具有錯誤詳細資料。

```javascript
// Submit Subclip Callback
//
// Parameter:
// - subclip: object that represents the subclip (output contract).
//
// Returns: a Promise object that, when resolved, retuns true if the operation was accept in the back-end; otherwise, returns false.
var onSubmitSubclip = function (subclip) {
    var promise = new Promise(function (resolve, reject) {
        // TODO: perform the back-end AJAX request to submit the subclip job.
        var result = true;

        if (/* everything turned out fine */ result) {
            resolve(result);
        }
        else {
            reject(Error("error details"));
        }
    });

    return promise;
};

var subclipper = new subclipper({
    selector: '#root',
    restVersion: '2.0',
    submitSubclipCallback: onSubmitSubclip,
});
```
作業提交的輸出可以是轉譯作業的媒體編碼器標準編碼預設，或是動態資訊清單篩選的 REST API 承載。

## <a name="submitting-encoding-job-to-create-video"></a>提交編碼作業來建立影片
您可以提交媒體編碼器標準編碼作業，來建立畫面精準的影片剪輯。 編碼作業會產生轉譯的影片，一個新的分散式 MP4 檔案。

轉譯剪輯的作業輸出合約是具有下列屬性的 JSON 物件：

```json
{
  /* Required */
  "name": "My New Subclip Output Name",

  /* Required: string specifying the format of the Json file.
   */
  /* NOTE: The REST API version, currently 2.0 is supported
   */
  "restVersion": "2.0"

  /* Required: array containing all the input Ids (both "asset" or "filter" type) used in the subclipper timeline;
  it must contain at least one item. */
  /* NOTE: when "output"."type" === "job", all items must match the "inputsIds"[i]."type" === "asset" condition;
  this array can be used in the back-end to get the input assets for the subclipping job with the Media Encoder Standard (MES) processor. */
  /* NOTE: when "output"."type" === "filter", there must be a single item in the array ("inputsIds".length === 1)
  that can be used in the back-end to get the source for the dynamic manifest filter (create or update). */
  "inputsIds": [{
    /* Required */
    "id": "my-asset-id-1",

    /* Required: must be one of the following values: "asset" or "filter" */
    "type": "asset"
  }, {
    /* Required */
    "id": "my-asset-id-2",

    /* Required: must be one of the following values: "asset" or "filter" */
    "type": "asset"
  }],

  /* Required */
  "output": {

    /* Required: must be one of the following values: "job" or "filter" */
    "type": "job",

    /* Required if "type" === "job" */
    /* NOTE: This is the preset for the Media Encoder Standard (MES) processor that can be used in the back-end to sumit the subclip job.
    The encoding profile ("Codecs" property) depends on the "singleBitrateMp4Profile" and "multiBitrateMp4Profile" option parameters
    specified when creating the widget instance. */
    /* REFERENCE: https://docs.microsoft.com/azure/media-services/media-services-advanced-encoding-with-mes */
    "job": {
      "Version": 1.0,
      "Codecs": [{
          "KeyFrameInterval": "00:00:02",
          "SceneChangeDetection": true,
          "H264Layers": [{
            "Profile": "Auto",
            "Level": "auto",
            "Bitrate": 6750,
            "MaxBitrate": 6750,
            "BufferWindow": "00:00:05",
            "Width": 1920,
            "Height": 1080,
            "BFrames": 3,
            "ReferenceFrames": 3,
            "AdaptiveBFrame": true,
            "Type": "H264Layer",
            "FrameRate": "0/1"
          }],
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
      "Outputs": [{
        "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
        "Format": {
          "Type": "MP4Format"
        }
      }],
      "Sources": [{
        "AssetId": "my-asset-id-1",
        "StartTime": "0.00:01:15.000",
        "Duration": "0.00:00:25.000"
      }, {
        "AssetId": "my-asset-id-2",
        "StartTime": "0.00:20:04.000",
        "Duration": "0.00:33:07.500"
      }]
    }
  }
}
```

若要執行編碼作業，請使用相關聯的預設提交媒體編碼器標準編碼作業。 請參閱本文，以了解使用 [.NET SDK](https://docs.microsoft.com/azure/media-services/media-services-dotnet-encode-with-media-encoder-standard) 或 [REST API](https://docs.microsoft.com/azure/media-services/media-services-rest-encode-asset) 提交編碼作業的詳細資料。

## <a name="quickly-creating-video-clips-without-encoding"></a>不需編碼即可快速建立影片剪輯
除了建立編碼作業，您還能使用 Azure Media Clipper 來建立動態資訊清單篩選。 篩選不需編碼，並且可在新資產未建立時快速建立。 篩選條件剪輯的輸出合約是具有下列屬性的 JSON 物件：

```json
{
  /* Required: Filter name (Alphanumeric characters and hyphens only, no whitespace) */
  "name": "FilterName",

  /* Required: string specifying the format of the Json file.
   */
  /* NOTE: This subclipper version always returns '2.0' in this field.
   */
  "restVersion": "2.0"

  /* Required: array containing all the input Ids (both "asset" or "filter" type) used in the subclipper timeline;
  it must contain at least one item. */
  /* NOTE: when "output"."type" === "job", all items must match the "inputsIds"[i]."type" === "asset" condition;
  this array can be used in the back-end to get the input assets for the subclipping job with the Media Encoder Standard (MES) processor. */
  /* NOTE: when "output"."type" === "filter", there must be a single item in the array ("inputsIds".length === 1)
  that can be used in the back-end to get the source for the dynamic manifest filter (create or update). */
  "inputsIds": [{
    /* Required */
    "id": "my-asset-id",

    /* Required: must be one of the following values: "asset" or "filter" */
    "type": "asset"
  }],

  /* Required */
  "output": {

    /* Required: must be one of the following values: "job" or "filter" */
    "type": "filter",

    /* Required if "type" === "filter" */
    /* REFERENCE: https://docs.microsoft.com/rest/api/media/operations/assetfilter */
    "filter": {
      "PresentationTimeRange": {
        "StartTimestamp": "340000000",
        "EndTimestamp": "580000000",
        "Timescale": "10000000"
      },
      "Tracks": [{
        "PropertyConditions": [{
          "Property": "Type",
          "Value": "video",
          "Operator": "Equal"
        }]
      }, {
        "PropertyConditions": [{
          "Property": "Type",
          "Value": "audio",
          "Operator": "Equal"
        }, {
          "Property": "Name",
          "Value": "audio-track-1",
          "Operator": "Equal"
        }]
      }, {
        "PropertyConditions": [{
          "Property": "Type",
          "Value": "text",
          "Operator": "Equal"
        }, {
          "Property": "Language",
          "Value": "en",
          "Operator": "Equal"
        }]
      }]
    }
  }
}
```

若要提交 REST 呼叫以建立動態資訊清單篩選，請使用 [REST API](https://docs.microsoft.com/azure/media-services/media-services-rest-dynamic-manifest) 提交相關聯的篩選承載。