---
title: 使用 U-SQL 指令碼轉換資料 - Azure | Microsoft Docs
description: 了解如何在 Azure Data Lake Analytics 計算服務上執行 U-SQL 指令碼來處理或轉換資料。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: abnarain
ms.openlocfilehash: d406c7f7714e011126be67ad3f65938db62e7bbe
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39412843"
---
# <a name="transform-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>在 Azure Data Lake Analytics 上執行 U-SQL 指令碼來轉換資料 
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-usql-activity.md)
> * [目前的版本](transform-data-using-data-lake-analytics.md)

Azure Data Factory 中的「管線」會使用連結的計算服務，來處理連結的儲存體服務中的資料。 它包含一系列活動，其中每個活動都會執行特定的處理作業。 本文將說明 **Data Lake Analytics U-SQL 活動**，它在 **Azure Data Lake Analytics** 計算連結的服務上執行 **U-SQL** 指令碼。 

使用 Data Lake Analytics「U-SQL 活動」來建立管線之前，請先建立 Azure Data Lake Analytics 帳戶。 若要了解 Azure Data Lake Analytics，請參閱 [開始使用 Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md)。


## <a name="azure-data-lake-analytics-linked-service"></a>Azure Data Lake Analytics 已連結的服務
您需建立 **Azure Data Lake Analytics** 連結服務，來將 Azure Data Lake Analytics 計算服務連結到 Azure Data Factory。 管線中的 Data Lake Analytics U-SQL 活動會參考此連結服務。 

下表提供 JSON 定義中所使用之一般屬性的描述。 

| 屬性                 | 說明                              | 必要                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| **type**                 | type 屬性應設為： **AzureDataLakeAnalytics**。 | 是                                      |
| **accountName**          | Azure Data Lake Analytics 帳戶名稱。  | 是                                      |
| **dataLakeAnalyticsUri** | Azure Data Lake Analytics URI。           | 否                                       |
| **subscriptionId**       | Azure 訂用帳戶識別碼                    | 否                                       |
| **resourceGroupName**    | Azure 資源群組名稱                | 否                                       |

### <a name="service-principal-authentication"></a>服務主體驗證
Azure Data Lake Analytics 已連結的服務需要服務主體驗證，才能連接到 Azure Data Lake Analytics 服務。 若要使用服務主體驗證，請在 Azure Active Directory (Azure AD) 中註冊應用程式實體，並授與其使用之 Data Lake Analytics 和 Data Lake Store 存取權。 如需詳細的步驟，請參閱[服務對服務驗證](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 請記下以下的值，您可以使用這些值來定義連結服務：

* 應用程式識別碼
* 應用程式金鑰 
* 租用戶識別碼

使用[新增使用者精靈](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#add-a-new-user)對您的 Azure Data Lake 分析授與服務主體權限。

指定下列屬性以使用服務主體驗證：

| 屬性                | 說明                              | 必要 |
| :---------------------- | :--------------------------------------- | :------- |
| **servicePrincipalId**  | 指定應用程式的用戶端識別碼。     | 是      |
| **servicePrincipalKey** | 指定應用程式的金鑰。           | 是      |
| **tenant**              | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | 是      |

**範例：服務主體驗證**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "<azure data lake analytics URI>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant ID>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }       
    }
}
```

若要深入了解已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)。

## <a name="data-lake-analytics-u-sql-activity"></a>Data Lake Analytics U-SQL 活動
下列 JSON 片段會定義具有 Data Lake Analytics U-SQL 活動的管線。 活動定義具有您稍早建立的 Azure Data Lake Analytics 連結服務的參考。 為了執行 Data Lake Analytics U-SQL 指令碼，Data Factory 會將您指定的指令碼提交給 Data Lake Analytics，必要的輸入和輸出會在 Data Lake Analytics 的指令碼中定義，以擷取及輸出。 

```json
{
    "name": "ADLA U-SQL Activity",
    "description": "description",
    "type": "DataLakeAnalyticsU-SQL",
    "linkedServiceName": {
        "referenceName": "<linked service name of Azure Data Lake Analytics>",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "<linked service name of Azure Data Lake Store or Azure Storage which contains the U-SQL script>",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
        "degreeOfParallelism": 3,
        "priority": 100,
        "parameters": {
            "in": "/datalake/input/SearchLog.tsv",
            "out": "/datalake/output/Result.tsv"
        }
    }   
}
```

下表描述此活動特有的屬性之名稱和描述。 

| 屬性            | 說明                              | 必要 |
| :------------------ | :--------------------------------------- | :------- |
| name                | 管線中的活動名稱     | 是      |
| 說明         | 說明活動用途的文字。  | 否       |
| type                | 對於 Data Lake Analytics U-SQL 活動，活動類型為 **DataLakeAnalyticsU-SQL**。 | 是      |
| 預設容器   | Azure Data Lake Analytics 之已連結的服務。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。  |是       |
| scriptPath          | 包含 U-SQL 指令碼的資料夾的路徑。 檔案的名稱有區分大小寫。 | 是      |
| scriptLinkedService | 連結服務會將包含指令碼的 **Azure Data Lake Store** 或 **Azure 儲存體**連結至資料處理站 | 是      |
| degreeOfParallelism | 同時用來執行作業的節點數目上限。 | 否       |
| 優先順序            | 判斷應該選取排入佇列的哪些工作首先執行。 編號愈低，優先順序愈高。 | 否       |
| parameters          | 要傳遞到 U-SQL 指令碼的參數。    | 否       |
| runtimeVersion      | 所要使用之 U-SQL 引擎的執行階段版本。 | 否       |
| compilationMode     | <p>U-SQL 的編譯模式。 必須是下列其中一個值：**Semantic：** 僅執行語意檢查和必要的例行性檢查、**Full：** 執行完整編譯，包括語法檢查、最佳化、程式碼產生等等，**SingleBox：** 執行完整編譯，TargetType 設定為 SingleBox。 如果您沒有為此屬性指定值，伺服器將會判斷最佳的編譯模式。 | 否 |

Data Factory 針對指令碼定義提交「請參閱 [SearchLogProcessing.txt 指令碼定義](#sample-u-sql-script)」。 

## <a name="sample-u-sql-script"></a>範例 U-SQL 指令碼

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int,
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

在上述指令碼範例中，指令碼的輸入和輸出是在 **@in** 和 **@out** 參數中定義。 Data Factory 會使用 ‘parameters’ 區段來動態傳遞 U-SQL 指令碼中 **@in** 和 **@out** 參數的值。 

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
    "in": "/datalake/input/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/data.tsv",
    "out": "/datalake/output/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/result.tsv"
}
```

在此例中，系統仍然會從 /datalake/input 資料夾挑選輸入檔，並在 /datalake/output 資料夾中產生輸出檔。 檔案名稱會根據觸發管線時傳入的範圍開始時間而變化。  

## <a name="next-steps"></a>後續步驟
請參閱下列文章，其說明如何以其他方式轉換資料： 

* [Hive 活動](transform-data-using-hadoop-hive.md)
* [Pig 活動](transform-data-using-hadoop-pig.md)
* [MapReduce 活動](transform-data-using-hadoop-map-reduce.md)
* [Hadoop 串流活動](transform-data-using-hadoop-streaming.md)
* [Spark 活動](transform-data-using-spark.md)
* [.NET 自訂活動](transform-data-using-dotnet-custom-activity.md)
* [Machine Learning 批次執行活動](transform-data-using-machine-learning.md)
* [預存程序活動](transform-data-using-stored-procedure.md)
