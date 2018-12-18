---
title: 使用 Azure Data Factory 從 HTTP 來源複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 管線中的複製活動，從雲端或內部部署 HTTP 來源將資料複製到支援的接收資料存放區。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/24/2018
ms.author: jingwang
ms.openlocfilehash: 5afb2fccd5c7b8ca306079941837d854c0b21349
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43091712"
---
# <a name="copy-data-from-http-endpoint-using-azure-data-factory"></a>使用 Azure Data Factory 從 HTTP 端點複製資料 | Microsoft Docs
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-http-connector.md)
> * [目前的版本](connector-http.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 HTTP 端點複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 HTTP 來源複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源/接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，此 HTTP 連接器支援：

- 使用 HTTP **GET** 或 **POST** 方法，從 HTTP/s 端點擷取資料。
- 使用下列驗證擷取資料︰**匿名**、**基本**、**摘要**、**Windows**和 **ClientCertificate**。
- 依原樣複製 HTTP 回應，或使用[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md)來剖析回應。

此連接器和 [Web 資料表連接器](connector-web-table.md)的差異是︰後者是用來擷取 HTML 網頁上的資料表內容。

>[!TIP]
>若要在 ADF 中設定 HTTP 連接器之前測試 HTTP 資料擷取要求，您可以從 API 規格取得標頭和本文需求，再使用 Postman 或網頁瀏覽器之類的工具進行驗證。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 HTTP 連接器專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 HTTP 連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**HttpServer**。 | 是 |
| url | Web 伺服器的基本 URL | 是 |
| enableServerCertificateValidation | 指定是否在連線到 HTTP 端點時啟用伺服器 SSL 憑證驗證。 當 HTTPS 伺服器使用自我簽署的憑證時，請將此屬性設定為 false。 | 否，預設值是 True |
| authenticationType | 指定驗證類型。 允許的值為︰**匿名**、**基本**、**摘要**、**Windows**、**ClientCertificate**。 <br><br> 請分別參閱此關於更多屬性的下列資料表各節以及這些驗證類型的 JSON 範例。 | 是 |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或「自我裝載 Integration Runtime」(如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

### <a name="using-basic-digest-or-windows-authentication"></a>使用基本、摘要或 Windows 驗證

將「authenticationType」屬性設定為**基本**、**摘要**或 **Windows**，並連同上一個區段所述的一般屬性指定下列內容：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| userName | 存取 HTTP 端點的使用者名稱。 | 是 |
| password | 使用者 (userName) 的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |

**範例**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "Basic",
            "url" : "<HTTP endpoint>",
            "userName": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="using-clientcertificate-authentication"></a>使用 ClientCertificate 驗證

若要使用 ClientCertificate 驗證，請將「authenticationType」屬性設定為 **ClientCertificate**，並連同上一個區段所述的一般屬性指定下列內容：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| embeddedCertData | Base64 編碼的憑證資料。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| certThumbprint | 憑證指紋已安裝在自我裝載整合執行階段機器的憑證存放區上。 在 connectVia 中指定整合執行階段的自我裝載類型時才適用。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| password | 與憑證相關聯的密碼。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 否 |

如果您使用 "certThumbprint" 進行驗證且憑證已安裝在本機電腦的個人存放區中，您必須授與讀取權限給自我裝載整合執行階段︰

1. 啟動 Microsoft Management Console (MMC)。 新增目標為 [本機電腦] 的 [憑證] 嵌入式管理單元。
2. 展開 [憑證]，[個人]，然後按一下 [憑證]。
3. 以滑鼠右鍵按一下個人存放區中的 [憑證]，然後選取 [所有工作] -> [管理私用金鑰...]
3. 在 [安全性] 索引標籤上，新增使用憑證讀取存取全執行整合執行階段主機服務 (DIAHostService) 的使用者帳戶。

**範例 1：使用 certThumbprint**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "certThumbprint": "<thumbprint of certificate>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**範例 2：使用 embeddedCertData**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "embeddedCertData": "<base64 encoded cert data>",
            "password": {
                "type": "SecureString",
                "value": "password of cert"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 HTTP 資料集所支援的屬性清單。

若要從 HTTP 複製資料，請將資料集的類型屬性設定為 **HttpFile**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**HttpFile** | 是 |
| relativeUrl | 包含資料之資源的相對 URL。 未指定此屬性時，則只會使用在連結服務定義中指定的 URL。 | 否 |
| requestMethod | HTTP 方法。<br/>允許的值為 **Get** (預設值) 或 **Post**。 | 否 |
| additionalHeaders | 其他 HTTP 要求標頭。 | 否 |
| requestBody | HTTP 要求的內文。 | 否 |
| format | 如果您想要**依原樣擷取 HTTP 端點的資料**而不剖析資料，並將資料複製到以檔案為基礎的存放區，請略過輸入和輸出資料集定義中的格式區段。<br/><br/>如果您想要在複製期間剖析 HTTP 回應內容，支援下列檔案格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱 [Json 格式](supported-file-formats-and-compression-codecs.md#json-format)、[文字格式](supported-file-formats-and-compression-codecs.md#text-format)、[Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)章節。 |否 |
| compression | 指定此資料的壓縮類型和層級。 如需詳細資訊，請參閱[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/>支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。<br/>支援的層級為：**Optimal** 和 **Fastest**。 |否 |

>[!NOTE]
>支援的 HTTP 要求承載大小是大約 500KB。 如果您希望傳遞至 Web 端點的承載大小大於這個值，請考慮批次分成較小區塊。

**範例 1：使用 Get 方法 (預設值)**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        }
    }
}
```

**範例 2：使用 Post 方法**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "requestMethod": "Post",
            "requestBody": "<body for POST HTTP request>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 HTTP 來源所支援的屬性清單。

### <a name="http-as-source"></a>HTTP 作為來源

若要從 HTTP 複製資料，請將複製活動中的來源類型設定為 **HttpSource**。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**HttpSource** | 是 |
| httpRequestTimeout | HTTP 的逾時 (TimeSpan) 要求取得回應。 逾時會取得回應，而非逾時讀取回應資料。<br/> 預設值為：00:01:40  | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromHTTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<HTTP input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "HttpSource",
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```


## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。
