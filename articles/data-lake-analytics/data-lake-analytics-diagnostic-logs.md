---
title: 啟用及檢視 Azure Data Lake Analytics 的診斷記錄
description: 了解如何設定及存取 Azure Data Lake Analytics 的診斷記錄
services: data-lake-analytics
ms.service: data-lake-analytics
author: jasonwhowell
ms.author: jasonh
ms.assetid: cf5633d4-bc43-444e-90fc-f90fbd0b7935
ms.topic: conceptual
ms.date: 02/12/2018
ms.openlocfilehash: 0bade9f393d879123b7b1485052f70924d9c9b9c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045476"
---
# <a name="accessing-diagnostic-logs-for-azure-data-lake-analytics"></a>存取 Azure Data Lake Analytics 的診斷記錄

診斷記錄可讓您收集資料存取稽核線索。 這些記錄檔可提供如下資訊︰

* 資料的存取使用者清單。
* 資料的存取頻率。
* 帳戶中儲存的資料量。

## <a name="enable-logging"></a>啟用記錄

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 開啟 Data Lake Analytics 帳戶，然後從 [監視] 區段選取 [診斷記錄]。 接下來，選取 [開啟診斷]。

    ![開啟診斷以收集稽核和要求記錄](./media/data-lake-analytics-diagnostic-logs/turn-on-logging.png)

3. 在 [診斷設定] 中，輸入此記錄組態的 [名稱]，然後選取記錄選項。

    ![開啟診斷以收集稽核和要求記錄](./media/data-lake-analytics-diagnostic-logs/enable-diagnostic-logs.png "啟用診斷記錄")

   * 您可以選擇三種不同的資料儲存/處理方法。

     * 選取 [封存至儲存體帳戶] 可將記錄儲存到 Azure 儲存體帳戶。 如果您想要封存資料，請使用此選項。 如果您選取此選項，必須提供用來儲存記錄的 Azure 儲存體帳戶。

     * 選取 [串流至事件中樞] 可將記錄資料串流到 Azure 事件中樞。 如果您有即時分析內送記錄的下游處理管線，請使用此選項。 如果您選取此選項，必須提供要使用的 Azure 事件中樞詳細資料。

     * 選取 [傳送至 Log Analytics] 可將資料傳送至 Log Analytics 服務。 如果您想要使用 Log Analytics 來收集和分析記錄，請使用此選項。
   * 指定要取得稽核記錄、要求記錄或兩者。  要求記錄會擷取每個應用程式開發介面 (API) 的要求。 稽核記錄則會記錄該 API 要求觸發的所有作業。

   * 針對 [封存至儲存體帳戶]，請指定要保留資料的天數。

   * 按一下 [檔案] 。

        > [!NOTE]
        > 您必須先選取 [封存至儲存體帳戶]、[串流至事件中樞] 或 [傳送至 Log Analytics]，再按一下 [儲存] 按鈕。

### <a name="use-the-azure-storage-account-that-contains-log-data"></a>使用包含記錄資料的 Azure 儲存體帳戶

1. 若要顯示存放記錄資料的 blob 容器，開啟 Data Lake Analytics 記錄所使用的 Azure 儲存體帳戶，然後按一下 [Blob]。

   * 容器 **insights-logs-audit** 包含稽核記錄檔。
   * 容器 **insights-logs-requests** 包含要求記錄檔。

2. 在這些容器中，記錄都會儲存在下列結構底下：

        resourceId=/
          SUBSCRIPTIONS/
            <<SUBSCRIPTION_ID>>/
              RESOURCEGROUPS/
                <<RESOURCE_GRP_NAME>>/
                  PROVIDERS/
                    MICROSOFT.DATALAKEANALYTICS/
                      ACCOUNTS/
                        <DATA_LAKE_ANALYTICS_NAME>>/
                          y=####/
                            m=##/
                              d=##/
                                h=##/
                                  m=00/
                                    PT1H.json

   > [!NOTE]
   > [Blob 服務] `##` 項目包含記錄檔的建立年、月、日和小時。 Data Lake Analytics 每小時會建立一個檔案，因此 `m=` 一律會包含 `00` 值。

    例如，稽核記錄檔的完整路徑可能是：

        https://adllogs.blob.core.windows.net/insights-logs-audit/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/mydatalakeanalytics/y=2016/m=07/d=18/h=04/m=00/PT1H.json

    同樣的，要求記錄檔的完整路徑可能是：

        https://adllogs.blob.core.windows.net/insights-logs-requests/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/mydatalakeanalytics/y=2016/m=07/d=18/h=14/m=00/PT1H.json

## <a name="log-structure"></a>記錄檔結構

稽核和要求記錄採用結構化 JSON 格式。

### <a name="request-logs"></a>要求記錄

以下是採用 JSON 格式之要求記錄中的範例項目。 每個 Blob 會一個名為 **記錄** 的根物件，其中包含記錄檔物件的陣列。

    {
    "records":
      [        
        . . . .
        ,
        {
             "time": "2016-07-07T21:02:53.456Z",
             "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/<data_lake_analytics_account_name>",
             "category": "Requests",
             "operationName": "GetAggregatedJobHistory",
             "resultType": "200",
             "callerIpAddress": "::ffff:1.1.1.1",
             "correlationId": "4a11c709-05f5-417c-a98d-6e81b3e29c58",
             "identity": "1808bd5f-62af-45f4-89d8-03c5e81bac30",
             "properties": {
                 "HttpMethod":"POST",
                 "Path":"/JobAggregatedHistory",
                 "RequestContentLength":122,
                 "ClientRequestId":"3b7adbd9-3519-4f28-a61c-bd89506163b8",
                 "StartTime":"2016-07-07T21:02:52.472Z",
                 "EndTime":"2016-07-07T21:02:53.456Z"
                 }
        }
        ,
        . . . .
      ]
    }

#### <a name="request-log-schema"></a>要求記錄的結構描述

| 名稱 | 類型 | 說明 |
| --- | --- | --- |
| 分析 |字串 |記錄的時間戳記 (UTC 時間) |
| ResourceId |字串 |執行作業所在資源的識別碼 |
| category |字串 |記錄類別。 例如， **要求**。 |
| operationName |字串 |記錄的作業名稱。 例如，GetAggregatedJobHistory。 |
| resultType |字串 |作業的狀態。例如，200。 |
| callerIpAddress |字串 |提出要求之用戶端的 IP 位址 |
| correlationId |字串 |角色的識別碼。 此值可用來群組一組相關的記錄項目。 |
| 身分識別 |Object |產生記錄的身分識別 |
| properties |JSON |請參閱下一節 (要求記錄檔屬性結構描述) 以取得詳細資訊 |

#### <a name="request-log-properties-schema"></a>要求記錄屬性結構描述

| 名稱 | 類型 | 說明 |
| --- | --- | --- |
| HttpMethod |字串 |作業使用的 HTTP 方法。 例如，GET。 |
| Path |字串 |執行作業的所在路徑 |
| RequestContentLength |int |HTTP 要求的內容長度 |
| ClientRequestId |字串 |可唯一識別此要求的識別碼 |
| StartTime |字串 |伺服器接收到要求的時間 |
| EndTime |字串 |伺服器傳送回應的時間 |

### <a name="audit-logs"></a>稽核記錄

以下是採用 JSON 格式之稽核記錄中的範例項目。 每個 Blob 會一個名為 **記錄** 的根物件，其中包含記錄檔物件的陣列。

    {
    "records":
      [        
        . . . .
        ,
        {
             "time": "2016-07-28T19:15:16.245Z",
             "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/<data_lake_ANALYTICS_account_name>",
             "category": "Audit",
             "operationName": "JobSubmitted",
             "identity": "user@somewhere.com",
             "properties": {
                 "JobId":"D74B928F-5194-4E6C-971F-C27026C290E6",
                 "JobName": "New Job",
                 "JobRuntimeName": "default",
                 "SubmitTime": "7/28/2016 7:14:57 PM"
                 }
        }
        ,
        . . . .
      ]
    }

#### <a name="audit-log-schema"></a>稽核記錄的結構描述

| 名稱 | 類型 | 說明 |
| --- | --- | --- |
| 分析 |字串 |記錄的時間戳記 (UTC 時間) |
| ResourceId |字串 |執行作業所在資源的識別碼 |
| category |字串 |記錄類別。 例如， **稽核**。 |
| operationName |字串 |記錄的作業名稱。 例如，JobSubmitted。 |
| resultType |字串 |作業狀態 (operationName) 的子狀態。 |
| resultSignature |字串 |作業狀態 (operationName) 的其他詳細資料。 |
| 身分識別 |字串 |要求作業的使用者。 例如： susan@contoso.com。 |
| properties |JSON |請參閱下一節 (稽核記錄檔屬性結構描述) 以取得詳細資訊 |

> [!NOTE]
> **resultType** 和 **resultSignature** 會提供作業結果的相關資訊，並且只會在作業完成時才包含值。 例如，只有當 **operationName** 包含 **JobStarted** 或 **JobEnded** 的值時，它們才會包含值。
>
>

#### <a name="audit-log-properties-schema"></a>稽核記錄屬性結構描述

| 名稱 | 類型 | 說明 |
| --- | --- | --- |
| JobId |字串 |指派給作業的識別碼 |
| JobName |字串 |為作業提供的名稱 |
| JobRunTime |字串 |用來處理作業的執行階段 |
| SubmitTime |字串 |作業提交時間 (UTC 格式) |
| StartTime |字串 |作業在提交後開始執行的時間 (UTC 格式) |
| EndTime |字串 |作業結束時間 |
| 平行處理原則 |字串 |在提交期間為此作業要求的 Data Lake Analytics 單位數目 |

> [!NOTE]
> **SubmitTime**、**StartTime**、**EndTime** 和 **Parallelism** 提供作業的相關資訊。 這些項目只會在作業啟動或完成時才包含值。 例如，**SubmitTime** 只會在 **operationName** 具有 **JobSubmitted** 值之後包含值。

## <a name="process-the-log-data"></a>處理記錄資料

Azure Data Lake Analytics 會提供有關如何處理和分析記錄資料的範例。 您可以在 [https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample](https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample) 找到範例。

## <a name="next-steps"></a>後續步驟
* [Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
