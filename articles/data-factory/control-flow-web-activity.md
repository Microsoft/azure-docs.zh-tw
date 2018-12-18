---
title: Azure Data Factory 中的 Web 活動 | Microsoft Docs
description: 了解如何使用 Web 活動 (Data Factory 支援的其中一個控制流程活動) 從管線叫用 REST 端點。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/14/2018
ms.author: shlo
ms.openlocfilehash: 71e89828645cadbbbf60527fca9968fd8ed568ff
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37055478"
---
# <a name="web-activity-in-azure-data-factory"></a>Azure Data Factory 中的 Web 活動
使用 Web 活動可以從 Data Factory 管線呼叫自訂的 REST 端點。 您可以傳遞資料集和連結服務，以供活動取用和存取。 

## <a name="syntax"></a>語法

```json
{  
   "name":"MyWebActivity",
   "type":"WebActivity",
   "typeProperties":{  
      "method":"Post",
      "url":"<URLEndpoint>",
      "headers":{  
         "Content-Type":"application/json"
      },
      "authentication":{  
         "type":"ClientCertificate",  
         "pfx":"****",
         "password":"****"
      },
      "datasets":[  
         {  
            "referenceName":"<ConsumedDatasetName>",
            "type":"DatasetReference",
            "parameters":{  
               ...
            }
         }
      ],
      "linkedServices":[  
         {  
            "referenceName":"<ConsumedLinkedServiceName>",
            "type":"LinkedServiceReference"
         }
      ]
   }
}

```

## <a name="type-properties"></a>類型屬性

屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
name | Web 活動的名稱 | 字串 | yes
type | 必須設定為 **WebActivity**。 | 字串 | yes
method | 目標端點的 Rest API 方法。 | 字串。 <br/><br/>支援的類型："GET"、"POST"、"PUT" | yes
url | 目標端點和路徑 | 字串 (或含有字串之 resultType 的運算式)。 如果活動未在 1 分鐘內收到來自端點的回應，就會發生逾時並出現錯誤。 | yes
headers | 傳送至要求的標頭。 例如，若要對要求設定語言和類型︰`"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }`。 | 字串 (或含有字串之 resultType 的運算式) | 是，Content-type 標頭是必要的。 `"headers":{ "Content-Type":"application/json"}`
body | 代表傳送至端點的承載。  | 字串 (或含有字串之 resultType 的運算式)。 <br/><br/>請在[要求乘載結構描述](#request-payload-schema)一節中查看要求乘載的結構描述。 | POST/PUT 方法的必要項。
驗證 | 呼叫端點所使用的驗證方法。 支援的類型為「基本」或 ClientCertificate。 如需詳細資訊，請參閱[驗證](#authentication)一節。 如果不需要驗證，請排除這個屬性。 | 字串 (或含有字串之 resultType 的運算式) | 否
資料集 | 傳遞至端點的資料集清單。 | 資料集參考的陣列。 可以是空陣列。 | yes
linkedServices | 傳遞至端點的連結服務清單。 | 連結服務參考的陣列。 可以是空陣列。 | yes

> [!NOTE]
> Web 活動叫用的 REST 端點必須傳回 JSON 類型的回應。 如果活動未在 1 分鐘內收到來自端點的回應，就會發生逾時並出現錯誤。

下表顯示 JSON 內容的需求：

| 值類型 | Request body | Response body |
|---|---|---|
|JSON 物件 | 支援 | 支援 |
|JSON 陣列 | 支援 <br/>(目前，JSON 陣列因為錯誤的結果無法運作。 正在執行修正。) | 不支援 |
| JSON 值 | 支援 | 不支援 |
| 非 JSON 型別 | 不支援 | 不支援 |
||||

## <a name="authentication"></a>驗證

### <a name="none"></a>None
如果不需要驗證，請勿包含 authentication 屬性。

### <a name="basic"></a>基本
指定要搭配基本驗證使用的使用者名稱和密碼。 

```json
"authentication":{  
   "type":"Basic,
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>用戶端憑證
指定以 base64 編碼的 PFX 檔案和密碼內容。 

```json
"authentication":{  
   "type":"ClientCertificate",
   "pfx":"****",   
   "password":"****"
}
```
## <a name="request-payload-schema"></a>要求承載結構描述
當您使用 POST/PUT 方法時，主體屬性代表傳送至端點的承載。 您可以將連結服務和資料集傳遞為承載的一部分。 以下是承載的結構描述： 

```json
{
    "body": {
        "myMessage": "Sample",
        "datasets": [{
            "name": "MyDataset1",
            "properties": {
                ...
            }
        }],
        "linkedServices": [{
            "name": "MyStorageLinkedService1",
            "properties": {
                ...
            }
        }]
    }
} 
```

## <a name="example"></a>範例
在此範例中，管線中的 Web 活動會呼叫 REST 端點。 它會將 Azure SQL 連結服務和 Azure SQL 資料集傳遞至端點。 REST 端點使用 Azure SQL 連接字串連接到 Azure SQL Server，並傳回 SQL Server 執行個體的名稱。 

### <a name="pipeline-definition"></a>管線定義

```json
{
    "name": "<MyWebActivityPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyWebActivity>",
                "type": "WebActivity",
                "typeProperties": {
                    "method": "Post",
                    "url": "@pipeline().parameters.url",
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "authentication": {
                        "type": "ClientCertificate",
                        "pfx": "*****",
                        "password": "*****"
                    },
                    "datasets": [
                        {
                            "referenceName": "MySQLDataset",
                            "type": "DatasetReference",
                            "parameters": {
                                "SqlTableName": "@pipeline().parameters.sqlTableName"
                            }
                        }
                    ],
                    "linkedServices": [
                        {
                            "referenceName": "SqlLinkedService",
                            "type": "LinkedServiceReference"
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "sqlTableName": {
                "type": "String"
            },
            "url": {
                "type": "String"
            }
        }
    }
}

```

### <a name="pipeline-parameter-values"></a>管線參數值

```json
{
    "sqlTableName": "department",
    "url": "https://adftes.azurewebsites.net/api/execute/running"
}

```

### <a name="web-service-endpoint-code"></a>Web 服務端點程式碼

```csharp

[HttpPost]
public HttpResponseMessage Execute(JObject payload)
{
    Trace.TraceInformation("Start Execute");

    JObject result = new JObject();
    result.Add("status", "complete");

    JArray datasets = payload.GetValue("datasets") as JArray;
    result.Add("sinktable", datasets[0]["properties"]["typeProperties"]["tableName"].ToString());

    JArray linkedServices = payload.GetValue("linkedServices") as JArray;
    string connString = linkedServices[0]["properties"]["typeProperties"]["connectionString"].ToString();

    System.Data.SqlClient.SqlConnection sqlConn = new System.Data.SqlClient.SqlConnection(connString);

    result.Add("sinkServer", sqlConn.DataSource);

    Trace.TraceInformation("Stop Execute");

    return this.Request.CreateResponse(HttpStatusCode.OK, result);
}

```

## <a name="next-steps"></a>後續步驟
請參閱 Data Factory 支援的其他控制流程活動： 

- [執行管線活動](control-flow-execute-pipeline-activity.md)
- [For Each 活動](control-flow-for-each-activity.md)
- [取得中繼資料活動](control-flow-get-metadata-activity.md)
- [查閱活動](control-flow-lookup-activity.md)
