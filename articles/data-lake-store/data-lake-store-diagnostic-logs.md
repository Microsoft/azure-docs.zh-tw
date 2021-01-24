---
title: 檢視 Azure Data Lake Storage Gen1 的診斷記錄 | Microsoft Docs
description: '了解如何設定及存取 Azure Data Lake Storage Gen1 的診斷記錄 '
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: f6e75eb1-d0ae-47cf-bdb8-06684b7c0a94
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: 07bf22cfc683d8c6f2c765364334ed1594e2fdaa
ms.sourcegitcommit: 4d48a54d0a3f772c01171719a9b80ee9c41c0c5d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/24/2021
ms.locfileid: "98745879"
---
# <a name="accessing-diagnostic-logs-for-azure-data-lake-storage-gen1"></a>存取 Azure Data Lake Storage Gen1 的診斷記錄
了解如何啟用 Azure Data Lake Storage Gen1 帳戶的診斷記錄，以及如何檢視針對您帳戶收集的記錄。

組織可以針對其 Azure Data Lake Storage Gen1 帳戶啟用診斷記錄，以收集資料存取審核記錄，以提供存取資料的使用者清單、存取資料的頻率、帳戶中儲存的資料量等資訊。啟用時，診斷和/或要求會以最大的方式登入。 只有在對服務端點提出要求時，才會建立要求和診斷記錄項目。

## <a name="prerequisites"></a>Prerequisites
* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* **Azure Data Lake Storage Gen1 帳戶**。 請遵循[透過 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示。

## <a name="enable-diagnostic-logging-for-your-data-lake-storage-gen1-account"></a>啟用 Data Lake Storage Gen1 帳戶的診斷記錄
1. 登入新的 [Azure 入口網站](https://portal.azure.com)。
2. 開啟 Data Lake Storage Gen1 帳戶，接著在 Data Lake Storage Gen1 帳戶刀鋒視窗中，按一下 [診斷設定]。
3. 在 [診斷設定] 刀鋒視窗中，按一下 [開啟診斷]。

    ![已呼叫 [診斷設定] 選項和 [開啟診斷] 選項的 Data Lake Storage Gen 1 帳戶螢幕擷取畫面。](./media/data-lake-store-diagnostic-logs/turn-on-diagnostics.png "啟用診斷記錄")

3. 在 [診斷設定] 刀鋒視窗中，進行下列變更以設定診斷記錄。
   
    ![[診斷設定] 區段的螢幕擷取畫面，其中包含 [名稱] 文字方塊和 [儲存] 選項已被呼叫。](./media/data-lake-store-diagnostic-logs/enable-diagnostic-logs.png "啟用診斷記錄")
   
   * 針對 [名稱]，輸入診斷記錄設定的值。
   * 您可以選擇不同的資料儲存/處理方法。
     
        * 選取 [封存至儲存體帳戶] 選項可將記錄儲存到 Azure 儲存體帳戶。 如果您想要保存資料以供日後批次處理，可以使用此選項。 如果您選取此選項，必須提供用來儲存記錄的 Azure 儲存體帳戶。
        
        * 選取 [串流至事件中樞] 選項可將記錄資料串流到 Azure 事件中樞。 如果您有即時分析內送記錄的下游處理管線，很可能會使用這個選項。 如果您選取此選項，必須提供要使用的 Azure 事件中樞詳細資料。

        * 選取要 **傳送至 Log Analytics** 的選項，以使用 Azure 監視器服務來分析所產生的記錄資料。 如果您選取此選項，必須提供要用來執行記錄分析的 Log Analytics 工作區詳細資料。 如需使用 Azure 監視器記錄檔的詳細資訊，請參閱使用 [Azure 監視器記錄檔來查看或分析收集的資料](../azure-monitor/log-query/log-analytics-tutorial.md) 。
     
   * 指定要取得稽核記錄、要求記錄或兩者。
   * 指定的資料的保留天數。 只有在您使用 Azure 儲存體帳戶來封存記錄資料時，才適用保留期。
   * 按一下 [儲存]。

一旦您啟用了診斷設定，即可在 [診斷記錄]  索引標籤中查看記錄。

## <a name="view-diagnostic-logs-for-your-data-lake-storage-gen1-account"></a>檢視 Data Lake Storage Gen1 帳戶的診斷記錄
檢視 Data Lake Storage Gen1 帳戶的記錄資料有兩種方式。

* 從 Data Lake Storage Gen1 帳戶設定檢視
* 從儲存資料的 Azure 儲存體帳戶

### <a name="using-the-data-lake-storage-gen1-settings-view"></a>使用 Data Lake Storage Gen1 設定檢是
1. 從 Data Lake Storage Gen1 帳戶的 [設定] 刀鋒視窗中，按一下 [診斷記錄]。
   
    ![檢視診斷記錄](./media/data-lake-store-diagnostic-logs/view-diagnostic-logs.png "檢視診斷記錄") 
2. 在 [診斷記錄] 刀鋒視窗中，您應該會看到依照 [稽核記錄] 和 [要求記錄] 分類的記錄。
   
   * 要求記錄會擷取所有以 Data Lake Storage Gen1 帳戶提出的 API 要求。
   * 稽核記錄與要求記錄相似，不過能針對以 Data Lake Storage Gen1 帳戶執行之作業提供更詳細的明細。 例如，要求記錄中的一個上傳 API 呼叫可能會致使稽核記錄出現多個「附加」作業。
3. 若要下載記錄，請針對每個記錄項目按一下 [下載] 連結。

### <a name="from-the-azure-storage-account-that-contains-log-data"></a>從包含記錄資料的 Azure 儲存體帳戶
1. 開啟與與用於記錄的 Data Lake Storage Gen1 關聯的 Azure 儲存體帳戶刀鋒視窗，然後按一下 [Blob]。 [Blob 服務]  刀鋒視窗會列出兩個容器。
   
    ![Data Lake Storage Gen 1] 分頁的螢幕擷取畫面，其中已選取 [Blob] 選項，而 [Blog service] 分頁則包含兩個稱為「blob 服務」的名稱。](./media/data-lake-store-diagnostic-logs/view-diagnostic-logs-storage-account.png "檢視診斷記錄")
   
   * 容器 **insights-logs-audit** 包含稽核記錄。
   * 容器 **insights-logs-requests** 包含要求記錄。
2. 在這些容器中，記錄會儲存在下列結構底下。
   
    ![記錄結構儲存在容器中的螢幕擷取畫面。](./media/data-lake-store-diagnostic-logs/view-diagnostic-logs-storage-account-structure.png "檢視診斷記錄")
   
    例如，稽核記錄檔的完整路徑可能是 `https://adllogs.blob.core.windows.net/insights-logs-audit/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKESTORE/ACCOUNTS/mydatalakestorage/y=2016/m=07/d=18/h=04/m=00/PT1H.json`
   
    同樣的，要求記錄檔的完整路徑可能是 `https://adllogs.blob.core.windows.net/insights-logs-requests/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKESTORE/ACCOUNTS/mydatalakestorage/y=2016/m=07/d=18/h=14/m=00/PT1H.json`

## <a name="understand-the-structure-of-the-log-data"></a>了解記錄資料的結構
稽核和要求記錄採用 JSON 格式。 在本節中，我們要探討要求和稽核記錄的 JSON 結構。

### <a name="request-logs"></a>要求記錄
以下是採用 JSON 格式之要求記錄中的範例項目。 每個 Blob 會一個名為 **記錄** 的根物件，其中包含記錄檔物件的陣列。

```json
{
"records": 
  [        
    . . . .
    ,
    {
        "time": "2016-07-07T21:02:53.456Z",
        "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKESTORE/ACCOUNTS/<data_lake_storage_gen1_account_name>",
        "category": "Requests",
        "operationName": "GETCustomerIngressEgress",
        "resultType": "200",
        "callerIpAddress": "::ffff:1.1.1.1",
        "correlationId": "4a11c709-05f5-417c-a98d-6e81b3e29c58",
        "identity": "1808bd5f-62af-45f4-89d8-03c5e81bac30",
        "properties": {"HttpMethod":"GET","Path":"/webhdfs/v1/Samples/Outputs/Drivers.csv","RequestContentLength":0,"StoreIngressSize":0 ,"StoreEgressSize":4096,"ClientRequestId":"3b7adbd9-3519-4f28-a61c-bd89506163b8","StartTime":"2016-07-07T21:02:52.472Z","EndTime":"2016-07-07T21:02:53.456Z","QueryParameters":"api-version=<version>&op=<operationName>"}
    }
    ,
    . . . .
  ]
}
```

#### <a name="request-log-schema"></a>要求記錄的結構描述
| 名稱 | 類型 | 描述 |
| --- | --- | --- |
| time |String |記錄的時間戳記 (UTC 時間) |
| resourceId |String |作業發生之資源的識別碼 |
| category |String |記錄類別。 例如， **要求**。 |
| operationName |String |記錄的作業名稱。 例如，getfilestatus。 |
| resultType |String |作業的狀態。例如，200。 |
| callerIpAddress |String |提出要求之用戶端的 IP 位址 |
| correlationId |String |用來將一組相關記錄項目分組在一起的記錄識別碼 |
| 身分識別 |Object |產生記錄的身分識別 |
| properties |JSON |如需詳細資料，請參閱下文 |

#### <a name="request-log-properties-schema"></a>要求記錄屬性結構描述
| 名稱 | 類型 | 描述 |
| --- | --- | --- |
| HttpMethod |String |作業使用的 HTTP 方法。 例如，GET。 |
| 路徑 |String |執行作業的所在路徑 |
| RequestContentLength |int |HTTP 要求的內容長度 |
| ClientRequestId |String |可唯一識別此要求的識別碼 |
| StartTime |String |伺服器接收到要求的時間 |
| EndTime |String |伺服器傳送回應的時間 |
| StoreIngressSize |long |輸入至 Data Lake Store 的大小（以位元組為單位） |
| StoreEgressSize |long |從 Data Lake Store 輸出的位元組大小 |
| QueryParameters |String |描述：這些是 HTTP 查詢參數。 範例1： api-version = 2014-01-01&op = getfilestatus 範例2： op = APPEND&append = true&syncFlag = DATA&filesessionid = bee3355a-4925-4435-bb4d-ceea52811aeb&leaseid = bee3355a-4925-4435-bb4d-ceea52811aeb&offset = 28313319&api-version = 2017-08-01 |

### <a name="audit-logs"></a>稽核記錄
以下是採用 JSON 格式之稽核記錄中的範例項目。 每個 blob 都有一個名為「 **記錄** 」的根物件，其中包含記錄物件的陣列。

```json
{
"records": 
  [        
    . . . .
    ,
    {
        "time": "2016-07-08T19:08:59.359Z",
        "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKESTORE/ACCOUNTS/<data_lake_storage_gen1_account_name>",
        "category": "Audit",
        "operationName": "SeOpenStream",
        "resultType": "0",
        "resultSignature": "0",
        "correlationId": "381110fc03534e1cb99ec52376ceebdf;Append_BrEKAmg;25.66.9.145",
        "identity": "A9DAFFAF-FFEE-4BB5-A4A0-1B6CBBF24355",
        "properties": {"StreamName":"adl://<data_lake_storage_gen1_account_name>.azuredatalakestore.net/logs.csv"}
    }
    ,
    . . . .
  ]
}
```

#### <a name="audit-log-schema"></a>稽核記錄的結構描述
| 名稱 | 類型 | 描述 |
| --- | --- | --- |
| time |String |記錄的時間戳記 (UTC 時間) |
| resourceId |String |作業發生之資源的識別碼 |
| category |String |記錄類別。 例如， **稽核**。 |
| operationName |String |記錄的作業名稱。 例如，getfilestatus。 |
| resultType |String |作業的狀態。例如，200。 |
| resultSignature |String |作業的其他詳細資料。 |
| correlationId |String |用來將一組相關記錄項目分組在一起的記錄識別碼 |
| 身分識別 |Object |產生記錄的身分識別 |
| properties |JSON |如需詳細資料，請參閱下文 |

#### <a name="audit-log-properties-schema"></a>稽核記錄屬性結構描述
| 名稱 | 類型 | 描述 |
| --- | --- | --- |
| StreamName |String |執行作業的所在路徑 |

## <a name="samples-to-process-the-log-data"></a>處理記錄資料的範例
從 Azure Data Lake Storage Gen1 將記錄傳送到 Azure 監視器記錄 (請參閱 [查看或分析以 Azure 監視器記錄檔收集的資料](../azure-monitor/log-query/log-analytics-tutorial.md) Azure 監視器使用) 記錄檔的詳細資訊，下列查詢會傳回資料表，其中包含使用者顯示名稱清單、事件時間，以及事件時間與視覺化圖表的事件計數。 您可以輕鬆地進行修改，以顯示使用者 GUID 或其他屬性：

```
search *
| where ( Type == "AzureDiagnostics" )
| summarize count(TimeGenerated) by identity_s, TimeGenerated
```


Azure Data Lake Storage Gen1 會提供有關如何處理和分析記錄資料的範例。 您可以在中找到此範例 [https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample](https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample) 。 

## <a name="see-also"></a>另請參閱
* [Azure Data Lake Storage Gen1 概觀](data-lake-store-overview.md)
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
