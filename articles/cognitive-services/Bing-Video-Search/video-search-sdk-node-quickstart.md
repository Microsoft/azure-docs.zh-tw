---
title: 快速入門：Bing 影片搜尋 SDK (Node)
titleSuffix: Azure Cognitive Services
description: 設定 Bing 影片搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: quickstart
ms.date: 02/12/2018
ms.author: rosh
ms.openlocfilehash: 4dcc9220d4d38bfe34514edd6a3ad47c7a7d4ba8
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225611"
---
# <a name="quickstart-bing-video-search-sdk-with-node"></a>快速入門：使用 Node 的 Bing 影片搜尋 SDK

Bing 影片搜尋 SDK 包含用於影片查詢和剖析結果的 REST API 功能。 

[節點 Bing 影片搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/videoSearch.js)可從 Git Hub 取得。

## <a name="application-dependencies"></a>應用程式相依性

若要使用 Bing 影片搜尋 SDK 來設定主控台應用程式，請在您的開發環境中執行 `npm install azure-cognitiveservices-videosearch`。

## <a name="video-search-client"></a>影片搜尋用戶端
在「搜尋」下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 建立 `CognitiveServicesCredentials` 的執行個體：
```
const CognitiveServicesCredentials = require('ms-rest-azure').CognitiveServicesCredentials;
let credentials = new CognitiveServicesCredentials('YOUR-ACCESS-KEY');
```
然後，將用戶端具現化：
```
const VideoSearchAPIClient = require('azure-cognitiveservices-videosearch');
let client = new VideoSearchAPIClient(credentials);
```
搜尋結果。
```
client.videosOperations.search('Interstellar Trailer').then((result) => {
    console.log(result.value);
}).catch((err) => {
    throw err;
});

```

<!-- Remove until the response can be replace with a sanitized version.
The code prints `result.value` items to the console without parsing any text. The results will be:
- _type: 'VideoObjectElementType'

![Video results](media/video-search-sdk-node-results.png)
-->

## <a name="next-steps"></a>後續步驟

[認知服務 Node.js SDK 範例](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples)
