---
title: 使用 U-SQL 指令碼轉換資料 - Azure | Microsoft Docs
description: 了解如何在 Azure Data Lake Analytics 計算服務上執行 U-SQL 指令碼來處理或轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.assetid: e17c1255-62c2-4e2e-bb60-d25274903e80
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/01/2017
ms.author: douglasl
robots: noindex
ms.openlocfilehash: 534fbeaa8ba3c27c8d3f3bbcc59717d8bdb5c654
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37050313"
---
# <a name="transform-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>在 Azure Data Lake Analytics 上執行 U-SQL 指令碼來轉換資料 
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-usql-activity.md)
> * [第 2 版 (目前的版本)](../transform-data-using-data-lake-analytics.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 U-SQL 活動](../transform-data-using-data-lake-analytics.md)。

Azure Data Factory 中的「管線」會使用連結的計算服務，來處理連結的儲存體服務中的資料。 它包含一系列活動，其中每個活動都會執行特定的處理作業。 本文將說明 **Data Lake Analytics U-SQL 活動**，它在 **Azure Data Lake Analytics** 計算連結的服務上執行 **U-SQL** 指令碼。 

使用 Data Lake Analytics「U-SQL 活動」來建立管線之前，請先建立 Azure Data Lake Analytics 帳戶。 若要了解 Azure Data Lake Analytics，請參閱 [開始使用 Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-get-started-portal.md)。

請檢閱 [建置您的第一個管線教學課程](data-factory-build-your-first-pipeline.md) ，以了解建立 Data Factory、連結服務、資料集和管線的詳細步驟。 您可以搭配「Data Factory 編輯器」、Visual Studio 或 Azure PowerShell 使用 JSON 程式碼片段來建立 Data Factory 實體。

## <a name="supported-authentication-types"></a>支援的驗證類型
U-SQL 活動支援以下針對 Data Lake Analytics 的驗證類型：
* 服務主體驗證
* 使用者認證 (OAuth) 驗證 

我們建議您使用服務主體驗證，特別是針對排程的 U-SQL 執行。 權杖到期行為會連同使用者認證驗證發生。 如需組態詳細資料，請參閱[連結服務屬性](#azure-data-lake-analytics-linked-service)一節。

## <a name="azure-data-lake-analytics-linked-service"></a>Azure Data Lake Analytics 連結服務
您需建立 **Azure Data Lake Analytics** 連結服務，來將 Azure Data Lake Analytics 計算服務連結到 Azure Data Factory。 管線中的 Data Lake Analytics U-SQL 活動會參考此連結服務。 

下表提供 JSON 定義中所使用之一般屬性的描述。 您可以在服務主體與使用者認證驗證之間進一步選擇。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| **type** |type 屬性應設為： **AzureDataLakeAnalytics**。 |yes |
| **accountName** |Azure Data Lake Analytics 帳戶名稱。 |yes |
| **dataLakeAnalyticsUri** |Azure Data Lake Analytics URI。 |否 |
| **subscriptionId** |Azure 訂用帳戶識別碼 |否 (如果未指定，便會使用 Data Factory 的訂用帳戶)。 |
| **resourceGroupName** |Azure 資源群組名稱 |否 (若未指定，便會使用 Data Factory 的資源群組)。 |

### <a name="service-principal-authentication-recommended"></a>服務主體驗證 (建議)
若要使用服務主體驗證，請在 Azure Active Directory (Azure AD) 中註冊應用程式實體，並授與其 Data Lake Store 存取權。 如需詳細的步驟，請參閱[服務對服務驗證](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 請記下以下的值，您可以使用這些值來定義連結服務：
* 應用程式識別碼
* 應用程式金鑰 
* 租用戶識別碼

指定下列屬性以使用服務主體驗證：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **servicePrincipalId** | 指定應用程式的用戶端識別碼。 | yes |
| **servicePrincipalKey** | 指定應用程式的金鑰。 | yes |
| **tenant** | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | yes |

**範例：服務主體驗證**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>使用者認證驗證
或者，您也可以藉由指定下列屬性，使用 Data Lake Analytics 的使用者認證驗證：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **authorization** | 按一下「資料處理站編輯器」中的 [授權] 按鈕，然後輸入您的認證，此動作會將自動產生的授權 URL 指派給此屬性。 | yes |
| **sessionId** | OAuth 授權工作階段的 OAuth 工作階段識別碼。 每個工作階段識別碼都是唯一的，只能使用一次。 當您使用「資料處理站編輯器」時便會自動產生此設定。 | yes |

**範例：使用者認證授權**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>權杖到期
您使用 [授權] 按鈕所產生的授權碼在一段時間後會到期。 請參閱下表以了解不同類型的使用者帳戶的到期時間。 當驗證**權杖到期**時，您可能會看到下列錯誤訊息：認證作業發生錯誤：invalid_grant - AADSTS70002：驗證認證時發生錯誤。 AADSTS70008：提供的存取授權已過期或撤銷。 追蹤識別碼：d18629e8-af88-43c5-88e3-d8419eb1fca1 相互關連識別碼：fac30a0c-6be6-4e02-8d69-a776d2ffefd7 時間戳記：2015-12-15 21:09:31Z

| 使用者類型 | 到期時間 |
|:--- |:--- |
| 不受 Azure Active Directory 管理的使用者帳戶 (@hotmail.com、@live.com 等) |12 小時 |
| 受 Azure Active Directory (AAD) 管理的使用者帳戶 |最後一次執行配量後的 14 天。 <br/><br/>如果以 OAuth 式連結服務為基礎的配量至少每 14 天執行一次，則為 90 天。 |

如果要避免/解決此錯誤，請在**權杖到期**時使用 [授權] 按鈕重新授權，然後重新部署連結服務。 您也可以如下使用程式碼，以程式設計方式產生 **sessionId** 和 **authorization** 屬性的值：

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

請參閱 [AzureDataLakeStoreLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)、[AzureDataLakeAnalyticsLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)和 [AuthorizationSessionGetResponse 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)主題，以取得在程式碼中使用的 Data Factory 類別的詳細資訊。 請針對 WindowsFormsWebAuthenticationDialog 類別，新增對下列項目的參考：Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll。 

## <a name="data-lake-analytics-u-sql-activity"></a>Data Lake Analytics U-SQL 活動
下列 JSON 片段會定義具有 Data Lake Analytics U-SQL 活動的管線。 活動定義具有您稍早建立的 Azure Data Lake Analytics 連結服務的參考。   

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This is a pipeline to compute events for en-gb locale and date less than 2012/02/19.",
        "activities": 
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs": 
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00Z",
        "end": "2015-08-08T01:00:00Z",
        "isPaused": false
    }
}
```

下表描述此活動特有的屬性之名稱和描述。 

| 屬性            | 說明                              | 必要                                 |
| :------------------ | :--------------------------------------- | :--------------------------------------- |
| type                | 類型屬性必須設為 DataLakeAnalyticsU-SQL 。 | yes                                      |
| 預設容器   | 參考 Azure Data Lake Analytics 註冊為 Data Factory 中的連結服務 | yes                                      |
| scriptPath          | 包含 U-SQL 指令碼的資料夾的路徑。 檔案的名稱有區分大小寫。 | 否 (如果您使用指令碼)                   |
| scriptLinkedService | 連結服務會連結包含 Data Factory 的指令碼的儲存體 | 否 (如果您使用指令碼)                   |
| script              | 指定內嵌指令碼而不是指定 scriptPath 和 scriptLinkedService。 例如：`"script": "CREATE DATABASE test"`。 | 否 (如果您使用 scriptPath 和 scriptLinkedService) |
| degreeOfParallelism | 同時用來執行作業的節點數目上限。 | 否                                       |
| 優先順序            | 判斷應該選取排入佇列的哪些工作首先執行。 編號愈低，優先順序愈高。 | 否                                       |
| parameters          | U-SQL 指令碼的參數          | 否                                       |
| runtimeVersion      | 所要使用之 U-SQL 引擎的執行階段版本 | 否                                       |
| compilationMode     | <p>U-SQL 的編譯模式。 必須是下列其中一個值：</p> <ul><li>**Semantic：** 僅執行語意檢查和必要的例行性檢查。</li><li>**Full：** 執行完整編譯，包括語法檢查、最佳化、程式碼產生等。</li><li>**SingleBox：** 在將 TargetType 設定為 SingleBox 的情況下，執行完整編譯。</li></ul><p>如果您沒有為此屬性指定值，伺服器將會判斷最佳的編譯模式。 </p> | 否                                       |

指令碼定義請參閱 [SearchLogProcessing.txt 指令碼定義](#sample-u-sql-script) 。 

## <a name="sample-input-and-output-datasets"></a>建立輸入和輸出資料集
### <a name="input-dataset"></a>輸入資料集
在此範例中，輸入的資料是位於 Azure Data Lake Store (datalake/input 資料夾中的 SearchLog.tsv 檔案)。 

```json
{
    "name": "DataLakeTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}    
```

### <a name="output-dataset"></a>輸出資料集
在此範例中，U-SQL 指令碼所產生的輸出資料會儲存在 Azure Data Lake Store (datalake/output 資料夾)。 

```json
{
    "name": "EventsByRegionTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="sample-data-lake-store-linked-service"></a>Data Lake Store 連結服務範例
以下是輸入/輸出資料集所使用的範例 Azure Data Lake Store 連結服務的定義。 

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
        }
    }
}
```

如需 JSON 屬性的描述，請參閱 [將資料移入和移除 Azure Data Lake Store](data-factory-azure-datalake-connector.md) 一文。 

## <a name="sample-u-sql-script"></a>U-SQL 指令碼範例

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv(nullEscape:"#NULL#");

@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";

@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start <= DateTime.Parse("2012/02/19");

OUTPUT @rs1   
    TO @out
      USING Outputters.Tsv(quoting:false, dateTimeFormat:null);
```

ADF 會使用 ‘parameters’ 區段來動態傳遞 U-SQL 指令碼中 **@in** 和 **@out** 參數的值。 請參閱管線定義中的 ‘parameters’ 區段。

您也可以在管線定義中，針對在 Azure Data Lake Analytics 服務上執行的作業，指定其他屬性 (例如 degreeOfParallelism 和 priority)。

## <a name="dynamic-parameters"></a>動態參數
在範例管線定義中，in 和 out 參數都被指派了硬式編碼值。 

```json
"parameters": {
    "in": "/datalake/input/SearchLog.tsv",
    "out": "/datalake/output/Result.tsv"
}
```

您可改為使用動態參數。 例如︰ 

```json
"parameters": {
    "in": "$$Text.Format('/datalake/input/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)",
    "out": "$$Text.Format('/datalake/output/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)"
}
```

在此例中，系統仍然會從 /datalake/input 資料夾挑選輸入檔，並在 /datalake/output 資料夾中產生輸出檔。 檔案名稱則是根據配量開始時間動態產生。  


