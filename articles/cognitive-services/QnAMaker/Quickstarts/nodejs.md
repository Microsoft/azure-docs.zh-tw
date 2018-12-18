---
title: 快速入門：適用於 QnA Maker API 的 Node.js (V4)
titleSuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您在 Azure 上快速開始使用 Microsoft 認知服務中的 Microsoft Translator Text API。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 09/12/2018
ms.author: diberry
ms.openlocfilehash: bbcd031f80010f10ab36440b703ac6dd55a8ced2
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387407"
---
# <a name="quickstart-for-microsoft-qna-maker-api-with-nodejs"></a>使用 Node.js 搭配 Microsoft QnA Maker API 的快速入門 
<a name="HOLTop"></a>

本文說明如何搭配使用 [Microsoft QnA Maker API](../Overview/overview.md) 和 Node.js，以執行以下動作。

- [建立新的知識庫。](#Create)
- [更新現有的知識庫。](#Update)
- [取得建立或更新知識庫要求的狀態。](#Status)
- [發佈現有的知識庫。](#Publish)
- [取代現有知識庫的內容。](#Replace)
- [下載知識庫的內容。](#GetQnA)
- [使用知識庫來取得問題的答案。](#GetAnswers)
- [取得知識庫的相關資訊。](#GetKB)
- [取得屬於指定使用者的所有知識庫相關資訊。](#GetKBsByUser)
- [刪除知識庫。](#Delete)
- [取得目前的端點金鑰。](#GetKeys)
- [重新產生目前的端點金鑰。](#PutKeys)
- [取得目前的文字變異形式集合。](#GetAlterations)
- [取代目前的文字變異形式集合。](#PutAlterations)

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-nodejs-repo-note.md)]

## <a name="prerequisites"></a>必要條件

您將需要 [Node.js 6](https://nodejs.org/en/download/) 才能執行此程式碼。

您必須擁有包含 **Microsoft QnA Maker API** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 您將需要來自 [Azure 儀表板](https://portal.azure.com/#create/Microsoft.CognitiveServices)的付費訂用帳戶金鑰。

<a name="Create"></a>

## <a name="create-knowledge-base"></a>建立知識庫

以下程式碼使用 [Create](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff) 方法來建立新的知識庫。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/create';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the POST request.
let post = function (path, content, callback) {
    let request_params = {
        method : 'POST',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases/create POST method.
let create_kb = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the POST request.
    post (path, req, function (response) {
// Extract the data we want from the POST response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

// callback is the function to call when we have the response from the GET request to check the status.
let check_status = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ wait : response.headers['retry-after'], response : response.body });
    });
}

let req = {
  "name": "QnA Maker FAQ",
  "qnaList": [
    {
      "id": 0,
      "answer": "You can use our REST APIs to manage your Knowledge Base. See here for details: https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600",
      "source": "Custom Editorial",
      "questions": [
        "How do I programmatically update my Knowledge Base?"
      ],
      "metadata": [
        {
          "name": "category",
          "value": "api"
        }
      ]
    }
  ],
  "urls": [
    "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
    "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
  ],
  "files": []
};

var path = service + method;
// Convert the request to a string.
let content = JSON.stringify(req);
create_kb (path, content, function (result) {
// Write out the response from the /knowledgebases/create method.
    console.log (pretty_print(result.response));
// Loop until the operation is complete.
    let loop = function () {
        path = service + result.operation;
// Check the status of the operation.
        check_status (path, function (status) {
// Write out the status.
            console.log (pretty_print(status.response));
// Convert the status into an object and get the value of the operationState field.
            var state = (JSON.parse(status.response)).operationState;
// If the operation isn't complete, wait and query again.
            if (state == 'Running' || state == 'NotStarted') {
                console.log ('Waiting ' + status.wait + ' seconds...');
                setTimeout(loop, status.wait * 1000);
            }
        });
    }
// Begin the loop.
    loop();
});
```

**建立知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:30Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
...
{
  "operationState": "Running",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:30Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:52:30Z",
  "lastActionTimestamp": "2018-04-13T01:52:46Z",
  "resourceLocation": "/knowledgebases/b0288f33-27b9-4258-a304-8b9f63427dad",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "e88b5b23-e9ab-47fe-87dd-3affc2fb10f3"
}
```

[回到頁首](#HOLTop)

<a name="Update"></a>

## <a name="update-knowledge-base"></a>更新知識庫

以下程式碼使用 [Update](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) 方法來更新現有的知識庫。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the PATCH request.
let patch = function (path, content, callback) {
    let request_params = {
        method : 'PATCH',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases PATCH method.
let update_kb = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the PATCH request.
    patch (path, req, function (response) {
// Extract the data we want from the PATCH response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

// callback is the function to call when we have the response from the GET request to check the status.
let check_status = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ wait : response.headers['retry-after'], response : response.body });
    });
}

let req = {
  'add': {
    'qnaList': [
      {
        'id': 1,
        'answer': 'You can change the default message if you use the QnAMakerDialog. See this for details: https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle',
        'source': 'Custom Editorial',
        'questions': [
          'How can I change the default message from QnA Maker?'
        ],
        'metadata': []
      }
    ],
    'urls': [
      'https://docs.microsoft.com/azure/cognitive-services/Emotion/FAQ'
    ]
  },
  'update' : {
    'name' : 'New KB Name'
  },
  'delete': {
    'ids': [
      0
    ]
  }
};

var path = service + method + kb;
// Convert the request to a string.
let content = JSON.stringify(req);
update_kb (path, content, function (result) {
    console.log (pretty_print(result.response));
// Loop until the operation is complete.
    let loop = function () {
        path = service + result.operation;
// Check the status of the operation.
        check_status (path, function (status) {
// Write out the status.
            console.log (pretty_print(status.response));
// Convert the status into an object and get the value of the operationState field.
            var state = (JSON.parse(status.response)).operationState;
// If the operation isn't complete, wait and query again.
            if (state == 'Running' || state == 'NotStarted') {
                console.log ('Waiting ' + status.wait + ' seconds...');
                setTimeout(loop, status.wait * 1000);
            }
        });
    }
// Begin the loop.
    loop();
});
```

**更新知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:48Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:50Z",
  "resourceLocation": "/knowledgebases/140a46f3-b248-4f1b-9349-614bfd6e5563",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
Press any key to continue.
```

[回到頁首](#HOLTop)

<a name="Status"></a>

## <a name="get-request-status"></a>取得要求狀態

您可以呼叫 [Operation](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails) 方法，以查看建立或更新知識庫要求的狀態。 若要了解如何使用此方法，請參閱 [Create](#Create) 或 [Update](#Update) 方法的範例程式碼。

[回到頁首](#HOLTop)

<a name="Publish"></a>

## <a name="publish-knowledge-base"></a>發佈知識庫

以下程式碼使用 [Publish](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fe) 方法來發行現有的知識庫。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the POST request.
let post = function (path, content, callback) {
    let request_params = {
        method : 'POST',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases POST method.
let publish_kb = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the POST request.
    post (path, req, function (response) {
// Extract the data we want from the POST response and pass it to the callback function.
        if (response.status == '204') {
            let result = {'result':'Success'};
            callback (JSON.stringify(result));
        }
        else {
            callback (response.body);
        }
    });
}

var path = service + method + kb;
publish_kb (path, '', function (result) {
    console.log (pretty_print(result));
});
```

**發佈知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "result": "Success."
}
```

[回到頁首](#HOLTop)

<a name="Replace"></a>

## <a name="replace-knowledge-base"></a>取代知識庫

以下程式碼使用 [Replace](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_publish) 方法來取代指定知識庫的內容。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the PUT request.
let put = function (path, content, callback) {
    let request_params = {
        method : 'PUT',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases PUT method.
let replace_kb = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the PUT request.
    put (path, req, function (response) {
// Extract the data we want from the PUT response and pass it to the callback function.
        if (response.status == '204') {
            let result = {'result':'Success'};
            callback (JSON.stringify(result));
        }
        else {
            callback (response.body);
        }
    });
}

let req = {
  'qnaList': [
    {
      'id': 0,
      'answer': 'You can use our REST APIs to manage your Knowledge Base. See here for details: https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600',
      'source': 'Custom Editorial',
      'questions': [
        'How do I programmatically update my Knowledge Base?'
      ],
      'metadata': [
        {
          'name': 'category',
          'value': 'api'
        }
      ]
    }
  ]
};

var path = service + method + kb;
// Convert the request to a string.
let content = JSON.stringify(req);
replace_kb (path, content, function (result) {
    console.log (pretty_print(result));
});
```

**取代知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
    "result": "Success."
}
```

[回到頁首](#HOLTop)

<a name="GetQnA"></a>

## <a name="download-the-contents-of-a-knowledge-base"></a>下載知識庫的內容

以下程式碼使用 [Download knowledge base](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_download) 方法來下載指定知識庫的內容。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

// NOTE: Replace this with "test" or "prod".
let env = "test";

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = `/knowledgebases/${kb}/${env}/qna/`;

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases GET method.
let get_qna = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

var path = service + method;
get_qna (path, function (result) {
    console.log (pretty_print(result.response));
});
```

**下載知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "qnaDocuments": [
    {
      "id": 1,
      "answer": "You can use our REST APIs to manage your Knowledge Base. See here for details: https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600",
      "source": "Custom Editorial",
      "questions": [
        "How do I programmatically update my Knowledge Base?"
      ],
      "metadata": [
        {
          "name": "category",
          "value": "api"
        }
      ]
    },
    {
      "id": 2,
      "answer": "QnA Maker provides an FAQ data source that you can query from your bot or application. Although developers will find this useful, content owners will especially benefit from this tool. QnA Maker is a completely no-code way of managing the content that powers your bot or application.",
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "questions": [
        "Who is the target audience for the QnA Maker tool?"
      ],
      "metadata": []
    },
...
  ]
}
```

[回到頁首](#HOLTop)

<a name="GetAnswers"></a>

## <a name="get-answers-to-a-question-using-a-knowledge-base"></a>使用知識庫來取得問題的答案

以下程式碼使用 **Generate answers** 方法，以使用特定知識庫來取得問題的答案。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
1. 新增下方提供的程式碼。
1. 以 QnA Maker 訂用帳戶的網站名稱來取代 `host` 值。 如需詳細資訊，請參閱[建立 QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)。
1. 以訂用帳戶的有效端點金鑰來取代 `endpoint_key` 值。 請注意，此金鑰不同於您的訂用帳戶金鑰。 您可以使用 [Get endpoint keys](#GetKeys) 方法來取得端點金鑰。
1. 針對要查詢答案的知識庫，以知識庫的識別碼來取代 `kb` 值。 請注意此知識庫必須已使用 [Publish](#Publish) 方法來發行。
1. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// NOTE: Replace this with a valid host name.
let host = "ENTER HOST HERE";

// NOTE: Replace this with a valid endpoint key.
// This is not your subscription key.
// To get your endpoint keys, call the GET /endpointkeys method.
let endpoint_key = "ENTER KEY HERE";

// NOTE: Replace this with a valid knowledge base ID.
// Make sure you have published the knowledge base with the
// POST /knowledgebases/{knowledge base ID} method.
let kb = "ENTER KB ID HERE";

let method = "/qnamaker/knowledgebases/" + kb + "/generateAnswer";

let question = {
    'question': 'Is the QnA Maker Service free?',
    'top': 3
};

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the POST request.
let post = function (path, content, callback) {
    let request_params = {
        method : 'POST',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Authorization' : 'EndpointKey ' + endpoint_key,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases POST method.
let get_answers = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the POST request.
    post (path, req, function (response) {
        callback (response.body);
    });
}

// Convert the request to a string.
let content = JSON.stringify(question);
get_answers (method, content, function (result) {
// Write out the response from the /knowledgebases/create method.
    console.log (pretty_print(result));
});
```

**取得答案的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "answers": [
    {
      "questions": [
        "Can I use BitLocker with the Volume Shadow Copy Service?"
      ],
      "answer": "Yes. However, shadow copies made prior to enabling BitLocker will be automatically deleted when BitLocker is enabled on software-encrypted drives. If you are using a hardware encrypted drive, the shadow copies are retained.",
      "score": 17.3,
      "id": 62,
      "source": "https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-frequently-asked-questions",
      "metadata": []
    },
...
  ]
}
```

[回到頁首](#HOLTop)

<a name="GetKB"></a>

## <a name="get-information-about-a-knowledge-base"></a>取得知識庫的相關資訊

以下程式碼使用 [Get knowledge base details](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_getknowledgebasedetails) 方法來取得特定知識庫的相關資訊。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases GET method.
let get_kb = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

var path = service + method + kb;
get_kb (path, function (result) {
    console.log (pretty_print(result.response));
});
```

**取得知識庫詳細資料的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "id": "140a46f3-b248-4f1b-9349-614bfd6e5563",
  "hostName": "https://qna-docs.azurewebsites.net",
  "lastAccessedTimestamp": "2018-04-12T22:58:01Z",
  "lastChangedTimestamp": "2018-04-12T22:58:01Z",
  "name": "QnA Maker FAQ",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "urls": [
    "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
    "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
  ],
  "sources": [
    "Custom Editorial"
  ]
}
```

[回到頁首](#HOLTop)

<a name="GetKBsByUser"></a>

## <a name="get-all-knowledge-bases-for-a-user"></a>取得使用者的所有知識庫

以下程式碼使用 [Get knowledge bases for user](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_getknowledgebasesforuser) 方法，以取得指定使用者的所有知識庫相關資訊。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases GET method.
let get_kbs = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

var path = service + method;
get_kbs (path, function (result) {
    console.log (pretty_print(result.response));
});
```

**取得使用者知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "knowledgebases": [
    {
      "id": "081c32a7-bd05-4982-9d74-07ac736df0ac",
      "hostName": "https://qna-docs.azurewebsites.net",
      "lastAccessedTimestamp": "2018-04-12T11:51:58Z",
      "lastChangedTimestamp": "2018-04-12T11:51:58Z",
      "name": "QnA Maker FAQ",
      "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
      "urls": [],
      "sources": []
    },
    {
      "id": "140a46f3-b248-4f1b-9349-614bfd6e5563",
      "hostName": "https://qna-docs.azurewebsites.net",
      "lastAccessedTimestamp": "2018-04-12T22:58:01Z",
      "lastChangedTimestamp": "2018-04-12T22:58:01Z",
      "name": "QnA Maker FAQ",
      "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
      "urls": [
        "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
        "https://docs.microsoft.com/bot-framework/resources-bot-framework-faq"
      ],
      "sources": [
        "Custom Editorial"
      ]
    },
...
  ]
}
Press any key to continue.
```

[回到頁首](#HOLTop)

<a name="Delete"></a>

## <a name="delete-a-knowledge-base"></a>刪除知識庫

以下程式碼使用 [Delete knowledge base](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/knowledgebases_delete) 方法來刪除指定的知識庫。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with a valid knowledge base ID.
let kb = 'ENTER ID HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/knowledgebases/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the DELETE request.
let http_delete = function (path, content, callback) {
    let request_params = {
        method : 'DELETE',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases DELETE method.
let delete_kb = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the DELETE request.
    http_delete (path, req, function (response) {
// Extract the data we want from the DELETE response and pass it to the callback function.
        if (response.status == '204') {
            let result = {'result':'Success'};
            callback (JSON.stringify(result));
        }
        else {
            callback (response.body);
        }
    });
}

var path = service + method + kb;
delete_kb (path, '', function (result) {
    console.log (pretty_print(result));
});
```

**刪除知識庫的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "result": "Success."
}
```

[回到頁首](#HOLTop)

<a name="GetKeys"></a>

## <a name="get-endpoint-keys"></a>取得端點金鑰

以下程式碼使用 [Get endpoint keys](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/endpointkeys_getendpointkeys) 方法來取得目前的端點金鑰。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/endpointkeys/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases GET method.
let get_keys = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

var path = service + method;
get_keys (path, function (result) {
    console.log (pretty_print(result.response));
});
```

**取得端點金鑰的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "primaryEndpointKey": "ac110bdc-34b7-4b1c-b9cd-b05f9a6001f3",
  "secondaryEndpointKey": "1b4ed14e-614f-444a-9f3d-9347f45a9206"
}
```

[回到頁首](#HOLTop)

<a name="PutKeys"></a>

## <a name="refresh-endpoint-keys"></a>重新整理端點金鑰

以下程式碼使用 [Refresh endpoint keys](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/endpointkeys_refreshendpointkeys) 方法來重新產生目前的端點金鑰。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

// NOTE: Replace this with "PrimaryKey" or "SecondaryKey."
let key_type = "PrimaryKey";

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/endpointkeys/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the PATCH request.
let patch = function (path, content, callback) {
    let request_params = {
        method : 'PATCH',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases PATCH method.
let refresh_keys = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the PATCH request.
    patch (path, req, function (response) {
// Extract the data we want from the PATCH response and pass it to the callback function.
        if (response.status == '204') {
            let result = {'result':'Success'};
            callback (JSON.stringify(result));
        }
        else {
            callback (response.body);
        }
    });
}

let req = {
  'wordAlterations': [
    {
      'alterations': [
        'botframework',
        'bot frame work'
      ]
    }
  ]
};

var path = service + method + key_type;
// Convert the request to a string.
let content = JSON.stringify(req);
refresh_keys (path, content, function (result) {
    console.log (pretty_print(result));
});
```

**重新整理端點金鑰的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "primaryEndpointKey": "ac110bdc-34b7-4b1c-b9cd-b05f9a6001f3",
  "secondaryEndpointKey": "1b4ed14e-614f-444a-9f3d-9347f45a9206"
}
```

[回到頁首](#HOLTop)

<a name="GetAlterations"></a>

## <a name="get-word-alterations"></a>取得文字變異形式

以下程式碼使用 [Download alterations](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fc) 方法來取得目前的文字變異形式。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/alterations/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the GET request.
let get = function (path, callback) {
    let request_params = {
        method : 'GET',
        hostname : host,
        path : path,
        headers : {
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases GET method.
let get_alterations = function (path, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the GET request.
    get (path, function (response) {
// Extract the data we want from the GET response and pass it to the callback function.
        callback ({ operation : response.headers.location, response : response.body });
    });
}

var path = service + method;
get_alterations (path, function (result) {
    console.log (pretty_print(result.response));
});
```

**取得文字變異形式的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "wordAlterations": [
    {
      "alterations": [
        "botframework",
        "bot frame work"
      ]
    }
  ]
}
```

[回到頁首](#HOLTop)

<a name="PutAlterations"></a>

## <a name="replace-word-alterations"></a>取代文字變異形式

以下程式碼使用 [Replace alterations](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fd) 方法來取代目前的文字變異形式。

1. 在您偏好的 IDE 中建立新的 Node.js 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```nodejs
'use strict';

let fs = require ('fs');
let https = require ('https');

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace this with a valid subscription key.
let subscriptionKey = 'ENTER KEY HERE';

let host = 'westus.api.cognitive.microsoft.com';
let service = '/qnamaker/v4.0';
let method = '/alterations/';

let pretty_print = function (s) {
    return JSON.stringify(JSON.parse(s), null, 4);
}

// callback is the function to call when we have the entire response.
let response_handler = function (callback, response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
// Call the callback function with the status code, headers, and body of the response.
        callback ({ status : response.statusCode, headers : response.headers, body : body });
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};

// Get an HTTP response handler that calls the specified callback function when we have the entire response.
let get_response_handler = function (callback) {
// Return a function that takes an HTTP response, and is closed over the specified callback.
// This function signature is required by https.request, hence the need for the closure.
    return function (response) {
        response_handler (callback, response);
    }
}

// callback is the function to call when we have the entire response from the PUT request.
let put = function (path, content, callback) {
    let request_params = {
        method : 'PUT',
        hostname : host,
        path : path,
        headers : {
            'Content-Type' : 'application/json',
            'Content-Length' : Buffer.byteLength(content),
            'Ocp-Apim-Subscription-Key' : subscriptionKey,
        }
    };

// Pass the callback function to the response handler.
    let req = https.request (request_params, get_response_handler (callback));
    req.write (content);
    req.end ();
}

// callback is the function to call when we have the response from the /knowledgebases PUT method.
let put_alterations = function (path, req, callback) {
    console.log ('Calling ' + host + path + '.');
// Send the PUT request.
    put (path, req, function (response) {
// Extract the data we want from the PUT response and pass it to the callback function.
        if (response.status == '204') {
            let result = {'result':'Success'};
            callback (JSON.stringify(result));
        }
        else {
            callback (response.body);
        }
    });
}

let req = {
  'wordAlterations': [
    {
      'alterations': [
        'botframework',
        'bot frame work'
      ]
    }
  ]
};

var path = service + method;
// Convert the request to a string.
let content = JSON.stringify(req);
put_alterations (path, content, function (result) {
    console.log (pretty_print(result));
});
```

**取代文字變異形式的回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "result": "Success."
}
```

[回到頁首](#HOLTop)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)

## <a name="see-also"></a>另請參閱 

[QnA Maker 概觀](../Overview/overview.md)
