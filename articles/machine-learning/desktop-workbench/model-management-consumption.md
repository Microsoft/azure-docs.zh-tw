---
title: Azure Machine Learning 模型管理 Web 服務取用 | Microsoft Docs
description: 本文件說明在 Azure Machine Learning 中取用以模型管理所部署之 Web 服務有關的步驟和概念。
services: machine-learning
author: raymondlaghaeian
ms.author: raymondl
manager: hjerez
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/06/2017
ROBOTS: NOINDEX
ms.openlocfilehash: c01198a78a32c460bd147e1e160358271b80eef5
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950615"
---
# <a name="consuming-web-services"></a>取用 Web 服務

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


將模型部署為即時 Web 服務後，您就可以對它傳送資料，並從各種不同的平台和應用程式取得預測。 即時 Web 服務會公開用來取得預測的 REST API。 您可以透過單一資料列或多重資料列格式將資料傳送給 Web 服務，以便一次取得一或多個預測。

使用 [Azure Machine Learning Web 服務](model-management-service-deploy.md)，外部應用程式會藉由對服務 URL 發出 HTTP POST 呼叫，來與預測模型進行同步通訊。 若要發出 Web 服務呼叫，用戶端應用程式必須指定您在部署預測時所建立的 API 金鑰，並將要求資料放入 POST 要求本文。

> [!NOTE]
> 請注意，叢集部署模式才會有 API 金鑰。 本機 Web 服務不會有金鑰。

## <a name="service-deployment-options"></a>服務部署選項
Azure Machine Learning Web 服務可以部署到雲端式叢集以供生產和測試案例使用，也可以部署到使用 Docker 引擎的本機工作站。 這兩種情況下的預測模型功能會一樣。 叢集式部署會提供以 Azure Container Service 為基礎的解決方案，這個解決方案不僅可擴充，也會有優異的效能，至於本機部署則可用來偵錯。 

Azure Machine Learning CLI 和 API 提供了方便的命令，讓您使用 ```az ml env``` 選項來建立及管理計算環境以供部署服務。 

## <a name="list-deployed-services-and-images"></a>列出已部署的服務和映像
您可以使用 CLI 命令 ```az ml service list realtime -o table``` 來列出目前已部署的服務和 Docker 映像。 請注意，此命令永遠都可以在目前計算環境的內容中運作。 它不會顯示在未設為目前環境的環境中部署的服務。 若要設定環境，請使用 ```az ml env set```。 

## <a name="get-service-information"></a>取得服務資訊
成功部署 Web 服務後，請使用下列命令來取得服務 URL 和其他詳細資料以便呼叫服務端點。 

```
az ml service usage realtime -i <web service id>
```

如果在部署時提供了服務 API 結構描述，此命令便會印出可用來呼叫服務的服務 URL、必要的要求標頭、Swagger URL 和資料範例。

只要輸入 CLI 命令範例和輸入資料，您就可以直接從 CLI 測試服務，而不必撰寫 HTTP 要求：

```
az ml service run realtime -i <web service id> -d "Your input data"
```

## <a name="get-the-service-api-key"></a>取得服務的 API 金鑰
若要取得 Web 服務金鑰，請使用下列命令：

```
az ml service keys realtime -i <web service id>
```
在建立 HTTP 要求時，請使用授權標頭 "Authorization": "Bearer <key>" 中的金鑰

## <a name="get-the-service-swagger-description"></a>取得服務 Swagger 的說明
如果有提供服務 API 結構描述，則服務端點會在 ```http://<ip>/api/v1/service/<service name>/swagger.json``` 中公開 Swagger 文件。 Swagger 文件可用來自動產生服務用戶端，並瀏覽預期的輸入資料和關於服務的其他詳細資料。

## <a name="get-service-logs"></a>取得服務記錄
若要了解服務的行為並診斷問題，有幾種方式可供用來擷取服務記錄：
- CLI 命令 ```az ml service logs realtime -i <service id>```。 此命令同時適用於叢集和本機模式。
- 如果您已在部署時啟用服務記錄功能，系統也會將服務記錄傳送給 AppInsight。 CLI 命令 ```az ml service usage realtime -i <service id>``` 會顯示 AppInsight URL。 請注意，AppInsight 記錄可能會有 2-5 分鐘的延遲。
- 您可以透過在使用 ```az ml env set``` 設定目前的叢集環境時所連線的 Kubernetes 主控台來檢視叢集記錄
- 如果服務是在本機執行，則可透過 Docker 引擎記錄來取得本機的 Docker 記錄。

## <a name="call-the-service-using-c"></a>使用 C# 呼叫服務
請使用服務 URL 從 C# 主控台應用程式傳送要求。 

1. 在 Visual Studio 中，建立新的主控台應用程式： 
    * 在功能表中，按一下 [檔案] -> [新增] -> [專案]
    * 在 Visual Studio C# 底下，按一下 [Windows 類別桌面]，然後選取 [主控台應用程式]。
2. 輸入 `MyFirstService` 作為專案的名稱，然後按一下 [確定]。
3. 在 [專案參考] 中，將參考設定為 `System.Net` 和 `System.Net.Http`。
4. 按一下 [工具] -> [NuGet 套件管理員] -> [套件管理員主控台]，然後安裝 **Microsoft.AspNet.WebApi.Client** 套件。
5. 開啟 **Program.cs** 檔案，並以下列程式碼取代其中的程式碼：
6. 使用從 Web 服務取得的資訊來更新 `SERVICE_URL` 和 `API_KEY` 參數。
7. 執行專案。

```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;
using Newtonsoft.Json;

namespace MyFirstService
{
    class Program
    {
        // Web Service URL (update it with your service url)
        private const string SERVICE_URL = "http://<service ip address>:80/api/v1/service/<service name>/score";
        private const string API_KEY = "your service key";

        static void Main(string[] args)
        {
            Program.PostRequest();
            Console.ReadLine();
        }

        private static void PostRequest()
        {
            HttpClient client = new HttpClient();
            client.BaseAddress = new Uri(SERVICE_URL);
            //For local web service, comment out this line.
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", API_KEY);

            var inputJson = new List<RequestPayload>();
            RequestPayload payload = new RequestPayload();
            List<InputDf> inputDf = new List<InputDf>();
            inputDf.Add(new InputDf()
            {
                feature1 = value1,
                feature2 = value2,
            });
            payload.Input_df_list = inputDf;
            inputJson.Add(payload);

            try
            {
                var request = new HttpRequestMessage(HttpMethod.Post, string.Empty);
                request.Content = new StringContent(JsonConvert.SerializeObject(payload));
                request.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
                var response = client.SendAsync(request).Result;

                Console.Write(response.Content.ReadAsStringAsync().Result);
            }
            catch (Exception e)
            {
                Console.Out.WriteLine(e.Message);
            }
        }
        public class InputDf
        {
            public double feature1 { get; set; }
            public double feature2 { get; set; }
        }
        public class RequestPayload
        {
            [JsonProperty("input_df")]
            public List<InputDf> Input_df_list { get; set; }
        }
    }
}
```

## <a name="call-the-web-service-using-python"></a>使用 Python 呼叫 Web 服務
您可以使用 Python 將要求傳送給即時 Web 服務。 

1. 將下列程式碼範例複製到新的 Python 檔案。
2. 更新資料、url 和 api_key 參數。 若為本機 Web 服務，請移除 'Authorization' 標頭。
3. 執行程式碼。 

```python
import requests
import json

data = "{\"input_df\": [{\"feature1\": value1, \"feature2\": value2}]}"
body = str.encode(json.dumps(data))

url = 'http://<service ip address>:80/api/v1/service/<service name>/score'
api_key = 'your service key' 
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}

resp = requests.post(url, body, headers=headers)
resp.text
```
