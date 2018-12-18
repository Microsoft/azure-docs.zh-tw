---
title: 將資產載入 Azure Media Clipper | Microsoft Docs
description: 將資產載入 Azure Media Clipper 的步驟
services: media-services
keywords: clip;subclip;encoding;media;剪輯;子剪輯;編碼;媒體
author: dbgeorge
manager: jasonsue
ms.author: dwgeo
ms.date: 11/10/2017
ms.topic: article
ms.service: media-services
ms.openlocfilehash: 6a479218ff8bd5addf4273b23c06380859e0ea08
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782897"
---
# <a name="loading-assets-into-azure-media-clipper"></a>將資產載入 Azure Media Clipper
您可以透過兩個方法將資產載入 Azure Media Clipper：
1. 以靜態方式傳遞資產庫
2. 透過 API 以動態方式產生資產清單

## <a name="statically-load-videos-into-clipper"></a>以靜態方式將影片載入到 Clipper
以靜態方式將影片載入到 Clipper，讓使用者能夠建置剪輯，而不需從資產選取面板中選取影片。

在此情況下，您會將一組靜態的資產傳入 Clipper。 每個資產都包括 AMS 資產/篩選識別碼、名稱，已發行的串流 URL。 如果適用的話，也可能會傳入內容保護驗證權杖，或是縮圖 URL 的陣列。 如果傳入的話，則會將縮圖填入到介面中。 在資產庫為靜態且較小的案例中，您可以針對資產庫中的每個資產傳入資產合約。

> [!NOTE]
> 以靜態方式將資產載入到 Clipper 時，即會將資產**直接新增至時間軸**，而且**不會呈現資產窗格**。 第一個資產會新增至時間軸，而其餘的資產會堆疊於時間軸右側。

若要載入靜態資產庫，請使用 **load** 方法來傳入每個資產的 JSON 表示法。 下列程式碼範例說明某一個資產的 JSON 表示法。

```javascript
var assets = [
    {
      /* Required: represents the Azure Media Services asset Id when "type" === "asset"; otherwise, represents the dynamic manifest asset filter Id ("type" === "filter")  */
      "id": "my-asset-or-dynamic-manifest-asset-filter-id",
    
      /* Required: represents the asset name as shown in the Clipper interface */
      "name": "My Asset or Dynamic Manifest Asset Filter Name",
    
      /* Required: must be one of the following values: "asset" or "filter" */
      /* NOTE: "asset" type represents a rendered asset; "filter" type represents a dynamic manifest asset filter */
      "type": "asset",
    
      /* Required */
      "source": {
    
        /* Required: represents the asset streaming locator, the base Smooth Streaming URL */
        "src": "//amssamples.streaming.mediaservices.windows.net/91492735-c523-432b-ba01-faba6c2206a2/AzureMediaServicesPromo.ism/manifest",
    
        /* Optional: default value "application/vnd.ms-sstr+xml" */
        "type": "application/vnd.ms-sstr+xml",
    
        /* Required: If the asset has content protection applied, then you must include an array with the different protection types along with the token to request the license/key; otherwise, provide an empty array */
        "protectionInfo": [{
            "type": "AES",
            "authenticationToken": "Bearer aes-token-placeholder"
          },
          {
            "type": "PlayReady",
            "authenticationToken": "Bearer playready-token-placeholder"
          },
          {
            "type": "Widevine",
            "authenticationToken": "Bearer widevine-token-placeholder"
          },
          {
            "type": "FairPlay",
            "certificateUrl": "//example/path/to/myfairplay.der",
            "authenticationToken": "Bearer fairplay-token-placeholder"
          }
        ]
      },
    
      /* Optional: array containing thumbnail URLs for the video. */
      /* NOTE: For the thumbnail URLs to work as expected in the Clipper timeline they must be evenly distributed across the video (based on the duration) and in chronological order within the array. */
      "thumbnails": [
        "//example/path/thumbnail_001.jpg",
        "//example/path/thumbnail_002.jpg",
        "//example/path/thumbnail_003.jpg",
        "//example/path/thumbnail_004.jpg",
        "//example/path/thumbnail_005.jpg"
        ]
    }
];

var subclipper = new subclipper({
    selector: '#root',
    restVersion: '2.0',
    submitSubclipCallback: onSubmitSubclip,
});
subclipper.ready(function () {
    subclipper.load(assets);
});

```

> [!NOTE]
> 建議使用 ready(handler) 方法來鏈結 load() 方法呼叫，如上述範例所示。 上述範例保證小工具會在載入資產之前就緒。

> [!NOTE]
> 若要讓縮圖 URL 能夠在 Clipper 時間軸中正確運作，它們必須平均地分散在影片中 (以持續時間為基礎)，並在陣列內以時間順序排序。 您可以使用下列 JSON 預設程式碼片段，作為使用「媒體編碼器標準」處理器產生影像的範例參考：

```json
{
  "Start": "0",
  "Step": "00:00:05",
  "Range": "100%",
  "Type": "PngImage",
  "PngLayers": [
    {
      "Type": "PngLayer",
      "Width": 48,
      "Height": 26
    }
  ]
}
```

## <a name="dynamically-load-videos-in-clipper"></a>以動態方式將影片載入到 Clipper
以動態方式將影片載入到 Clipper，讓使用者能夠從資產選取面板中選取要據以剪輯的影片。

或者，您可以透過回呼以動態方式載入資產。 在資產是以動態方式產生或資產庫較大的案例中，您應該透過回呼載入資產。 若要以動態方式載入資產，您必須實作選擇性的 onLoadAssets 回呼函式。 此函式會在初始化時傳入到 Clipper 中。 經過解析的資產應該遵守與靜態載入之資產相同的合約。 下列程式碼範例說明方法簽章、預期的輸入和預期的輸出。

> [!NOTE]
> 以動態方式將資產載入到 Clipper 時，會在**資產選取面板**中呈現資產。

```javascript
// Video Assets Pane Callback
    //
    // Filter Parameters:
    // - search: string value term that will be used in the back-end to filter assets by name.
    // - skip: int value used for pagination in the back-end that allows skipping a number of assets in the response.
    // - take: int value used for pagination in the back-end that allows defining the number of assets to include in the response.
    // - type: ('filter', 'asset') value that will be used in the back-end to filter assets by type.
    //
    // Returns: a Promise object that, when resolved, retuns an object containing an array of assets (input contract)
    //          that satisfies the filter parameters, plus optionally the total types of files available:
    // {
    //  total: 100,
    //  assets: [{...}],
    // }
    var onLoadAssets = function (search, skip, take, type) {
        var promise = new Promise(function (resolve, reject) {
            // TODO: implement the getAssetsFromBackend method to get the assets from the back-end using the filter parameters (search, skip, take, type).
            getAssetsFromBackend(search, skip, take, type)
                .then(function (assets) {
                    resolve({
                        total: assets.length,
                        assets: assets
                    });
                }).catch(function () {
                    reject(Error("error details"));
                });
        });
    
        return promise;
    };

    // Create widget instance:
    // - using a root element selector
    // - enabling the Video Assets panel by registering a callback in the 'assetsPanelLoaderCallback' option parameter.
    var widget = new subclipper({
        selector: '#root',

        // Enable the Video Assets panel in the widget to automatically load assets (input contract)
        assetsPanelLoaderCallback: onLoadAssets
    });

    // ...
    // The widget will automatically invoke the 'assetsPanelLoaderCallback' callback with the filter parameters specified by the user 
    // and load assets returned by the Promise into the Video Assets panel.
    // ...
```