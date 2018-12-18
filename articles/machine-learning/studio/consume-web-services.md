---
title: 如何使用 Azure Machine Learning Web 服務 | Microsoft Docs
description: 部署機器學習服務之後，就可以使用 RESTFul Web 服務作為即時要求-回應服務或批次執行服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 804f8211-9437-4982-98e9-ca841b7edf56
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/02/2017
ms.openlocfilehash: b89fb0fbb499fa06c9e56f02937b1c586efde9b6
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833349"
---
# <a name="how-to-consume-an-azure-machine-learning-web-service"></a>如何使用 Azure Machine Learning Web 服務

將 Azure Machine Learning 預測模型部署為 Web 服務後，就可以使用 REST API 傳送資料並取得預測。 您可以即時或以批次模式傳送資料。

有關如何使用 Machine Learning Studio 建立和部署 Machine Learning Web 服務的詳細資訊，請參閱此處：

* 如需如何在 Machine Learning Studio 中建立實驗的教學課程，請參閱 [建立您的第一個實驗](create-experiment.md)。
* 如需如何部署 Web 服務的詳細資訊，請參閱[部署 Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。
* 如需 Machine Learning 的一般詳細資訊，請參閱 [Machine Learning 文件中心](https://azure.microsoft.com/documentation/services/machine-learning/)。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="overview"></a>概觀
使用 Azure Machine Learning Web 服務，外部應用程式會即時與機器學習服務工作流程計分模型通訊。 機器學習 Web 服務呼叫會將預測結果傳回外部應用程式。 若要進行機器學習 Web 服務呼叫，您可以傳遞部署預測時所建立的 API 金鑰。 機器學習服務 Web 服務以 REST 為基礎，這是一種常見的 Web 程式設計專案架構。

Azure Machine Learning 有兩種類型的服務：

* 要求-回應服務 (RRS) – 這是一種低延遲、調整性高的服務，針對從 Machine Learning Studio 建立和部署的無狀態模型提供介面。
* 批次執行服務 (BES) – 這是一種非同步的服務，為一批資料記錄進行計分。

如需 Machine Learning Web 服務的詳細資訊，請參閱[部署 Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。

## <a name="get-an-azure-machine-learning-authorization-key"></a>取得 Azure Machine Learning 授權金鑰
當您部署實驗時，會為 Web 服務產生 API 金鑰。 您可以從數個位置擷取金鑰。

### <a name="from-the-microsoft-azure-machine-learning-web-services-portal"></a>透過 Microsoft Azure Machine Learning Web 服務入口網站
登入 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net)入口網站。

若要擷取新 Machine Learning Web 服務的 API 金鑰︰

1. 在 Azure Machine Learning Web Services 入口網站中，按一下頂端功能表上的 [Web 服務]。
2. 按一下您要擷取金鑰的 Web 服務。
3. 在頂端功能表上，按一下 [取用] 。
4. 複製並儲存 [主要金鑰] 。

若要擷取傳統 Machine Learning Web 服務的 API 金鑰︰

1. 在 Azure Machine Learning Web Services 入口網站中，按一下頂端功能表上的 [傳統 Web 服務]。
2. 按一下您所使用的 Web 服務。
3. 按一下您要取得金鑰的端點。
4. 在頂端功能表上，按一下 [取用] 。
5. 複製並儲存 [主要金鑰] 。

### <a name="classic-web-service"></a>傳統 Web 服務
 您也可以從 Machine Learning Studio 擷取傳統 Web 服務的金鑰。

#### <a name="machine-learning-studio"></a>Machine Learning Studio
1. 在 Machine Learning Studio 中，按一下左側的 [Web 服務]  。
2. 按一下某個 Web 服務。 [API 金鑰] 位於 [儀表板] 索引標籤上。

## <a id="connect"></a>連線到機器學習 Web 服務
您可以使用任何支援 HTTP 要求和回應的程式設計語言，連線到機器學習 Web 服務。 您可以從機器學習 Web 服務說明頁面檢視 C#、Python 和 R 的範例。

**機器學習服務 API 說明**當您部署 Web 服務時，會建立機器學習服務 API 說明。 請參閱 [Azure 機器學習逐步解說 - 部署 Web 服務](walkthrough-5-publish-web-service.md)。
機器學習服務 API 說明包含有關預測 Web 服務的詳細資訊。

1. 按一下您所使用的 Web 服務。
2. 按一下您想要檢視 API 說明頁面的端點。
3. 在頂端功能表上，按一下 [取用] 。
4. 按一下 [要求-回應] 或 [批次執行] 端點底下的 [API 說明頁面]。

**檢視新 Web 服務的機器學習 API 說明**

在 [Azure Machine Learning Web 服務入口網站](https://services.azureml.net/)中：

1. 按一下頂端功能表上的 [Web 服務]  。
2. 按一下您要擷取金鑰的 Web 服務。

按一下 [使用 Web 服務] 取得要求-回應服務和批次執行服務的 URI，以及以 C#、R 和 Python 撰寫的範例程式碼。

按一下 [Swagger API] 從提供的 URI，取得所呼叫 API 的 Swagger 相關文件。

### <a name="c-sample"></a>C# 範例
若要連線到機器學習 Web 服務，請使用 **HttpClient** 傳遞 ScoreData。 ScoreData 包含 FeatureVector，這是代表 ScoreData 的數值特徵 N 維向量。 您要使用 API 金鑰向機器學習服務驗證。

若要連線到機器學習 Web 服務，必須安裝 **Microsoft.AspNet.WebApi.Client** NuGet 封裝。

**在 Visual Studio 中安裝 Microsoft.AspNet.WebApi.Client NuGet**

1. 發佈 Download dataset from UCI: Adult 2 class dataset 的 Web 服務。
2. 按一下 [工具] > [NuGet 套件管理員] > [套件管理員主控台]。
3. 選擇 [ **Install-package Microsoft.AspNet.WebApi.Client**]。

**執行程式碼範例**

1. 發佈機器學習服務範例集合中的 "Sample 1: Download dataset from UCI: Adult 2 class dataset" 實驗。
2. 使用來自 Web 服務的金鑰指派 apiKey。 請參閱前述的 **取得 Azure Machine Learning 授權金鑰** 。
3. 使用要求 URI 指派 serviceUri。

**以下是完整的要求內容。**
```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

namespace CallRequestResponseService
{
    class Program
    {
        static void Main(string[] args)
        {
            InvokeRequestResponseService().Wait();
        }

        static async Task InvokeRequestResponseService()
        {
            using (var client = new HttpClient())
            {
                var scoreRequest = new
                {
                    Inputs = new Dictionary<string, List<Dictionary<string, string>>> () {
                        {
                            "input1",
                            // Replace columns labels with those used in your dataset
                            new List<Dictionary<string, string>>(){new Dictionary<string, string>(){
                                    {
                                        "column1", "value1"
                                    },
                                    {
                                        "column2", "value2"
                                    },
                                    {
                                        "column3", "value3"
                                    }
                                }
                            }
                        },
                    },
                    GlobalParameters = new Dictionary<string, string>() {}
                };

                // Replace these values with your API key and URI found on https://services.azureml.net/
                const string apiKey = "<your-api-key>"; 
                const string apiUri = "<your-api-uri>";
                
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue( "Bearer", apiKey);
                client.BaseAddress = new Uri(apiUri);

                // WARNING: The 'await' statement below can result in a deadlock
                // if you are calling this code from the UI thread of an ASP.Net application.
                // One way to address this would be to call ConfigureAwait(false)
                // so that the execution does not attempt to resume on the original context.
                // For instance, replace code such as:
                //      result = await DoSomeTask()
                // with the following:
                //      result = await DoSomeTask().ConfigureAwait(false)

                HttpResponseMessage response = await client.PostAsJsonAsync("", scoreRequest);

                if (response.IsSuccessStatusCode)
                {
                    string result = await response.Content.ReadAsStringAsync();
                    Console.WriteLine("Result: {0}", result);
                }
                else
                {
                    Console.WriteLine(string.Format("The request failed with status code: {0}", response.StatusCode));

                    // Print the headers - they include the requert ID and the timestamp,
                    // which are useful for debugging the failure
                    Console.WriteLine(response.Headers.ToString());

                    string responseContent = await response.Content.ReadAsStringAsync();
                    Console.WriteLine(responseContent);
                }
            }
        }
    }
}
```

### <a name="python-sample"></a>Python 範例
若要連線至 Machine Learning Web 服務，請使用 Python 2.X 的 **urllib2** 程式庫或 Python 3.X 的 **urllib.request** 程式庫。 要傳遞的是 ScoreData，內含 FeatureVector，是代表 ScoreData 數值特徵的 N 維向量。 您要使用 API 金鑰向機器學習服務驗證。

**執行程式碼範例**

1. 部署機器學習服務範例集合中的 "Sample 1: Download dataset from UCI: Adult 2 class dataset" 實驗。
2. 使用來自 Web 服務的金鑰指派 apiKey。 請參閱本文章接近開始處的**取得 Azure Machine Learning 授權金鑰**。
3. 使用要求 URI 指派 serviceUri。

**以下是完整的要求內容。**
```python
import urllib2 # urllib.request for Python 3.X
import json

data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",   
                'column2': "value2",   
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters":  {}
}

body = str.encode(json.dumps(data))

# Replace this with the URI and API Key for your web service
url = '<your-api-uri>'
api_key = '<your-api-key>'
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}

# "urllib.request.Request(uri, body, headers)" for Python 3.X
req = urllib2.Request(url, body, headers)

try:
    # "urllib.request.urlopen(req)" for Python 3.X
    response = urllib2.urlopen(req)

    result = response.read()
    print(result)
# "urllib.error.HTTPError as error" for Python 3.X
except urllib2.HTTPError, error: 
    print("The request failed with status code: " + str(error.code))

    # Print the headers - they include the requert ID and the timestamp, which are useful for debugging the failure
    print(error.info())
    print(json.loads(error.read())) 
```

### <a name="r-sample"></a>R 範例

若要連線至 Machine Learning Web 服務，請使用 **RCurl** 和 **rjson** 程式庫提出要求並處理傳回的 JSON 回應。 要傳遞的是 ScoreData，內含 FeatureVector，是代表 ScoreData 數值特徵的 N 維向量。 您要使用 API 金鑰向機器學習服務驗證。

**以下是完整的要求內容。**
```r
library("RCurl")
library("rjson")

# Accept SSL certificates issued by public Certificate Authorities
options(RCurlOptions = list(cainfo = system.file("CurlSSL", "cacert.pem", package = "RCurl")))

h = basicTextGatherer()
hdr = basicHeaderGatherer()

req = list(
    Inputs = list(
            "input1" = list(
                list(
                        'column1' = "value1",
                        'column2' = "value2",
                        'column3' = "value3"
                    )
            )
        ),
        GlobalParameters = setNames(fromJSON('{}'), character(0))
)

body = enc2utf8(toJSON(req))
api_key = "<your-api-key>" # Replace this with the API key for the web service
authz_hdr = paste('Bearer', api_key, sep=' ')

h$reset()
curlPerform(url = "<your-api-uri>",
httpheader=c('Content-Type' = "application/json", 'Authorization' = authz_hdr),
postfields=body,
writefunction = h$update,
headerfunction = hdr$update,
verbose = TRUE
)

headers = hdr$value()
httpStatus = headers["status"]
if (httpStatus >= 400)
{
print(paste("The request failed with status code:", httpStatus, sep=" "))

# Print the headers - they include the requert ID and the timestamp, which are useful for debugging the failure
print(headers)
}

print("Result:")
result = h$value()
print(fromJSON(result))
```

### <a name="javascript-sample"></a>JavaScript 範例

若要連線至 Machine Learning Web 服務，請使用專案中的 **request** npm 套件。 也會用到 `JSON` 物件在輸入時添加格式，並用來剖析結果。 使用 `npm install request --save` 進行安裝，或將 `"request": "*"` 新增至 `dependencies` 下的 package.json 並執行 `npm install`。

**以下是完整的要求內容。**
```js
let req = require("request");

const uri = "<your-api-uri>";
const apiKey = "<your-api-key>";

let data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",
                'column2': "value2",
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters": {}
}

const options = {
    uri: uri,
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + apiKey,
    },
    body: JSON.stringify(data)
}

req(options, (err, res, body) => {
    if (!err && res.statusCode == 200) {
        console.log(body);
    } else {
        console.log("The request failed with status code: " + res.statusCode);
    }
});
```