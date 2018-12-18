---
title: 快速入門：Bing 實體搜尋 SDK，Node
titleSuffix: Azure Cognitive Services
description: 使用 Node 設定實體搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: quickstart
ms.date: 02/12/2018
ms.author: v-gedod
ms.openlocfilehash: 1f2a5f6a1473cde40928ada6e30f6bd9b780543d
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48814877"
---
# <a name="quickstart-bing-entity-search-sdk-with-node"></a>快速入門：搭配使用 Bing 實體搜尋 SDK 與 Node

Bing 實體搜尋 SDK 包含用於實體查詢和剖析結果的 REST API 功能。 

[C# Bing 實體搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples/blob/master/Samples/entitySearch.js)可從 Git Hub 取得。
## <a name="application-dependencies"></a>應用程式相依性

若要使用 Bing 實體搜尋 SDK 來設定主控台應用程式，請在您的開發環境中執行 `npm install azure-cognitiveservices-entitysearch`。

## <a name="entity-search-client"></a>實體搜尋用戶端
在「搜尋」下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 建立 `CognitiveServicesCredentials` 的執行個體：
```
const CognitiveServicesCredentials = require('ms-rest-azure').CognitiveServicesCredentials;
let credentials = new CognitiveServicesCredentials('YOUR-ACCESS-KEY');
```
然後，將用戶端具現化，並搜尋結果：
```
const EntitySearchAPIClient = require('azure-cognitiveservices-entitysearch');

let entitySearchApiClient = new EntitySearchAPIClient(credentials);

entitySearchApiClient.entitiesOperations.search('seahawks').then((result) => {
    console.log(result.queryContext);
    console.log(result.entities.value);
    console.log(result.entities.value[0].description);
}).catch((err) => {
    throw err;
});

```
程式碼會將 `result.value` 項目列印到主控台，而不會剖析任何文字。  各個類別的結果 (如果有的話) 將會包含：
- _type：'Thing'
- _type：'ImageObject'

<!-- Removing until we can replace with a sanitized version.
![Entity results](media/entity-search-sdk-node-quickstart-results.png)
-->

## <a name="next-steps"></a>後續步驟

[認知服務 Node.js SDK 範例](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples)