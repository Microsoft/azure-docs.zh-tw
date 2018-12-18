---
title: 快速入門：專案 URL 預覽 Node.js
titlesuffix: Azure Cognitive Services
description: 在 Azure 的 Microsoft 認知服務中開始使用 URL 預覽。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-url-preview
ms.topic: quickstart
ms.date: 03/16/2018
ms.author: rosh
ms.openlocfilehash: b8410127ed9a0444ab400eb3242868a85bd5b2c9
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883562"
---
# <a name="quickstart-url-preview-with-nodejs"></a>快速入門：使用 Node.js 進行 URL 預覽 

下列 Node 範例會建立 SwiftKey 網站的 URL 預覽： https://swiftkey.com/en。

## <a name="prerequisites"></a>必要條件

取得免費試用版[認知服務實驗室](https://aka.ms/answersearchsubscription)的存取金鑰

## <a name="code-scenario"></a>程式碼案例 

下列程式碼會取得 URL 預覽資料。
實際進行步驟如下：
1. 宣告變數以依主機及路徑指定端點。
2. 指定要預覽的查詢 URL，並加入查詢參數。  
3. 建立回應的處理常式函式。
4. 定義建立要求的搜尋函式，並加入 *Ocp-Apim-Subscription-Key* 標頭。
5. 執行搜尋函式。 

此示範的完整程式碼如下：

````
'use strict';

let https = require('https');

// Replace the subscriptionKey string value with your valid subscription key.
let subscriptionKey = 'YOUR-ACCESS-KEY'; 
let host = 'api.labs.cognitive.microsoft.com';
let path = '/urlpreview/v7.0/search';

let mkt = 'en-US';
let q = 'https://swiftkey.com/';

let params = '?q=' + encodeURI(q);

let response_handler = function (response) {
    let body = '';
    response.on('data', function (d) {
        body += d;
    });
    response.on('end', function () {
        let body_ = JSON.parse(body);
        let body__ = JSON.stringify(body_, null, '  ');
        console.log(body__);
    });
    response.on('error', function (e) {
        console.log('Error: ' + e.message);
    });
};

let Search = function () {
    let request_params = {
        method: 'GET',
        hostname: host,
        path: path + params,
        headers: {
            'Ocp-Apim-Subscription-Key': subscriptionKey,
        }
    };

    let req = https.request(request_params, response_handler);
    req.end();
}

Search();

````

## <a name="next-steps"></a>後續步驟
- [C# 範例程式碼](csharp.md)
- [Java 快速入門](java-quickstart.md)
- [JavaScript 快速入門](javascript.md)
- [Python 快速入門](python-quickstart.md)