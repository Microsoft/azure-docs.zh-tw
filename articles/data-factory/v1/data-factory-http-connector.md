---
title: 從 HTTP 來源移動資料 - Azure | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從內部部署或雲端 HTTP 來源移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 29281843dc1b375182eb3dafe95ad86c89217671
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37052268"
---
# <a name="move-data-from-an-http-source-using-azure-data-factory"></a>使用 Azure Data Factory 來移動 HTTP 來源的資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-http-connector.md)
> * [第 2 版 (目前的版本)](../connector-http.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 HTTP 連接器](../connector-http.md)。


本文概述如何使用 Azure Data Factory 中的複製活動，將內部部署/雲端 HTTP 端點中的資料移動到支援的接收資料存放區。 本文是根據 [資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援作為來源/接收的資料存放區清單。

Data factory 目前只支援把 HTTP 來源的資料移動到其他資料存放區，但不支援把其他資料存放區的資料移動到 HTTP 目的地。

## <a name="supported-scenarios-and-authentication-types"></a>支援的案例和驗證類型
您可以使用這個 HTTP 連接器，藉由使用 HTTP **GET**或 **POST** 方法，從**雲端和內部部署 HTTP/s 端點擷取資料**。 支援下列驗證類型︰**匿名****基本**、**摘要**、**Windows**和 **ClientCertificate**。 請注意，此連接器和 [Web 資料表連接器](data-factory-web-table-connector.md)的差異是︰後者是用來擷取 HTML 網頁上的資料表內容。

從內部部署 HTTP 端點複製資料時，您需要在內部部署環境/Azure VM 中安裝資料管理閘道。 請參閱 [在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文來了解資料管理閘道和設定閘道的逐步指示。

## <a name="getting-started"></a>開始使用
您可以建立內含複製活動的管線，使用不同的工具/API 將資料移出 HTTP 來源。

- 若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

- 您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 若要將資料從 HTTP 來源複製到 Azure Blob 儲存體的 JSON 範例，請參閱本文的 [JSON 範例](#json-examples)一節。

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 HTTP 連結服務專屬 JSON 元素的說明。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type | 類型屬性必須設為：`Http`。 | yes |
| url | Web 伺服器的基本 URL | yes |
| authenticationType | 指定驗證類型。 允許的值為︰**匿名**、**基本**、**摘要**、**Windows**、**ClientCertificate**。 <br><br> 請分別參閱此關於更多屬性的下列資料表各節以及這些驗證類型的 JSON 範例。 | yes |
| enableServerCertificateValidation | 如果來源是 HTTPS Web 伺服器，指定是否啟用伺服器 SSL 憑證驗證。 當 HTTPS 伺服器使用自我簽署的憑證時，請將此屬性設定為 false。 | 否，預設值是 True |
| gatewayName | 連接至內部部署 HTTP 來源的「資料管理閘道」閘道。 | 如果從內部部署 HTTP 來源複製資料，則為是。 |
| encryptedCredential | 用來存取 HTTP 端點的加密認證。 當您在複製精靈或 ClickOnce 快顯對話方塊中設定驗證資訊時會自動產生。 | 否。 僅當從內部部署 HTTP 伺服器複製資料時才套用。 |

如需設定內部部署 HTTP 連接器資料來源認證的詳細資訊，請參閱[利用資料管理閘道在內部部署來源和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)。

### <a name="using-basic-digest-or-windows-authentication"></a>使用基本、摘要或 Windows 驗證

將 `authenticationType` 設定為 `Basic`、`Digest`或 `Windows`，並指定除了上面介紹的 HTTP 連接器泛用的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| username | 存取 HTTP 端點的使用者名稱。 | yes |
| password | 使用者 (使用者名稱) 的密碼。 | yes |

#### <a name="example-using-basic-digest-or-windows-authentication"></a>範例︰使用基本、摘要或 Windows 驗證

```JSON
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "basic",
            "url" : "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

### <a name="using-clientcertificate-authentication"></a>使用 ClientCertificate 驗證

若要使用基本驗證，將 `authenticationType` 設定為 `ClientCertificate`，並指定除了上面介紹的 HTTP 連接器泛用的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| embeddedCertData | Base 64 編碼的個人資訊交換 (PFX) 檔案之二進位資料內容。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| certThumbprint | 憑證指紋已安裝在您閘道器電腦的憑證存放區上。 僅當從內部部署 HTTP 來源複製資料時才套用。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| password | 與憑證相關聯的密碼。 | 否 |

如果您使用 `certThumbprint` 進行驗證且憑證已安裝在本機電腦的個人存放區中，您必須授與讀取權限給閘道器服務︰

1. 啟動 Microsoft Management Console (MMC)。 新增目標為 [本機電腦] 的 [憑證] 嵌入式管理單元。
2. 展開 [憑證]，[個人]，然後按一下 [憑證]。
3. 以滑鼠右鍵按一下個人存放區中的 [憑證]，然後選取 [所有工作]->[管理私用金鑰...]
3. 在 [安全性] 索引標籤上，新增資料管理閘道主機服務使用憑證讀取存取執行所在的使用者帳戶。  

#### <a name="example-using-client-certificate"></a>範例︰使用用戶端憑證
此連結服務會將您的資料處理站與內部部署 HTTP web 伺服器連結。 它會使用已安裝資料管理閘道的電腦上所安裝的用戶端憑證。

```JSON
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"

        }
    }
}
```

#### <a name="example-using-client-certificate-in-a-file"></a>範例︰在檔案中使用用戶端憑證
此連結服務會將您的資料處理站與內部部署 HTTP web 伺服器連結。 它會使用已安裝資料管理閘道的電腦上的用戶端憑證檔案。

```JSON
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "base64 encoded cert data",
            "password": "password of cert"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **Http** 類型資料集的 typeProperties 區段具有下列屬性

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 指定資料集的類型。 必須設為 `Http`。 | yes |
| relativeUrl | 包含資料之資源的相對 URL。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 <br><br> 若要建構動態 URL，您可以使用[Data Factory 函式與系統變數](data-factory-functions-variables.md)，例如 "relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)"。 | 否 |
| requestMethod | HTTP 方法。 允許的值為 **GET** 或 **POST**。 | 否。 預設值為 `GET`。 |
| additionalHeaders | 其他 HTTP 要求標頭。 | 否 |
| requestBody | HTTP 要求的內文。 | 否 |
| format | 如果您只想要**從 HTTP 端點依現狀擷取資料**而不剖析它，請略過此格式設定。 <br><br> 如果您想要在複製期間剖析 HTTP 回應內容，支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

### <a name="example-using-the-get-default-method"></a>範例︰使用 GET (預設值) 方法

```JSON
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "XXX/test.xml",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

### <a name="example-using-the-post-method"></a>範例︰使用 POST 方法

```JSON
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
           "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 **typeProperties** 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

目前，當複製活動中的來源類型為 **HttpSource** 時，支援下列屬性。

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| httpRequestTimeout | HTTP 的逾時 (TimeSpan) 要求取得回應。 逾時會取得回應，而非逾時讀取回應資料。 | 否。 預設值：00:01:40 |

## <a name="supported-file-and-compression-formats"></a>支援的檔案和壓縮格式
請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)文章以了解詳細資訊。

## <a name="json-examples"></a>JSON 範例
下列範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 這些範例示範如何將資料從 HTTP 來源複製到 Azure Blob 儲存體。 不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源 **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

### <a name="example-copy-data-from-http-source-to-azure-blob-storage"></a>範例：將資料從 HTTP 來源複製到 Azure Blob 儲存體
此範例的 Data Factory 方案包含下列資料處理站實體︰

1. 一個 [HTTP](#linked-service-properties) 類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [Http](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [HttpSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

範例會每隔一小時就把 HTTP 來源的資料複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

### <a name="http-linked-service"></a>HTTP 連結服務
此範例會使用匿名驗證來使用 HTTP 連結服務。 請參閱 [HTTP 連結服務](#linked-service-properties)一節，來了解您可以使用的不同驗證類型。

```JSON
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務

```JSON
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

### <a name="http-input-dataset"></a>HTTP 輸入資料集
將 **external** 設定為 **true** 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}

```

### <a name="azure-blob-output-dataset"></a>Azure Blob 輸出資料集

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。

```JSON
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/Movies"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

### <a name="pipeline-with-copy-activity"></a>具有複製活動的管線

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **HttpSource**，而 **sink** 類型設為 **BlobSink**。

請參閱 [HttpSource](#copy-activity-properties) 以取得 HttpSource 支援的屬性清單。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "HttpSourceToAzureBlob",
        "description": "Copy from an HTTP source to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "HttpSourceDataInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "HttpSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```

> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
