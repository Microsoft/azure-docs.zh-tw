---
title: 使用 Azure Data Factory 從 Amazon Simple Storage Service 複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 將資料從 Amazon Simple Storage Service (S3) 複製到支援的接收資料存放區。
services: data-factory
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/25/2018
ms.author: jingwang
ms.openlocfilehash: 3635e8bf1d9ba4061da5b8f416a3b755f7064000
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045631"
---
# <a name="copy-data-from-amazon-simple-storage-service-using-azure-data-factory"></a>使用 Azure Data Factory 從 Amazon Simple Storage Service 複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-amazon-simple-storage-service-connector.md)
> * [目前的版本](connector-amazon-simple-storage-service.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，從 Amazon S3 複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從 Amazon S3 複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

具體而言，這個 Amazon S3 連接器支援依原樣複製檔案，或使用[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md)來剖析檔案。

## <a name="required-permissions"></a>所需的權限

若要從 Amazon S3 複製資料，請確定您已獲得下列權限︰

- 適用於 Amazon S3 物件作業的 `s3:GetObject` 和 `s3:GetObjectVersion`。
- 適用於 Amazon S3 貯體作業的 `s3:ListBucket` 或 `s3:GetBucketLocation`。 如果您要使用「Data Factory 複製精靈」，則也需要 `s3:ListAllMyBuckets`。

如需有關完整 Amazon S3 權限清單的詳細資料，請參閱[在原則中指定權限 (英文)](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html)。

## <a name="getting-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)] 

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Amazon S3 專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

以下是針對 Amazon S3 已連結服務支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為：**AmazonS3**。 | yes |
| accessKeyId | 密碼存取金鑰的識別碼。 |yes |
| secretAccessKey | 密碼存取金鑰本身。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |yes |
| connectVia | 用來連線到資料存放區的 [Integration Runtime](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或「自我裝載 Integration Runtime」(如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>此連接器需要 IAM 帳戶的存取金鑰，才能從 Amazon S3 複製資料。 不支援[暫存安全性認證](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)。
>

下列是一個範例：

```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AmazonS3",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                    "type": "SecureString",
                    "value": "<secret access key>"
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

如需可用來定義資料集的區段和屬性完整清單，請參閱資料集文章。 本節提供 Amazon S3 資料集所支援的屬性清單。

若要從 Amazon S3 複製資料，請將資料集的類型屬性設定為 **AmazonS3Object**。 以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的類型屬性必須設定為：**AmazonS3Object** |yes |
| bucketName | S3 貯體名稱。 不支援萬用字元篩選。 |yes |
| 索引鍵 | 指定貯體下 S3 物件索引鍵的**名稱或萬用字元篩選**。 未指定 "prefix" 屬性時才適用。 <br/><br/>僅檔案名稱部分支援萬用字元篩選，資料夾部分不支援。 允許的萬用字元為：`*` (符合零或多個字元) 和 `?` (符合零或單一字元)。<br/>- 範例 1：`"key": "rootfolder/subfolder/*.csv"`<br/>- 範例 2：`"key": "rootfolder/subfolder/???20180427.txt"`<br/>如果實際檔案名稱內有萬用字元或逸出字元 `^`，請使用此逸出字元來逸出。 |否 |
| prefix | S3 物件索引鍵的前置詞。 系統會選取索引鍵以此前置詞開頭的物件。 未指定 "key" 屬性時才適用。 |否 |
| version | 如果已啟用 S3 版本設定功能，則為 S3 物件的版本。 |否 |
| format | 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。<br/><br/>如果您想要以特定格式來剖析或產生檔案，以下是支援的檔案格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](supported-file-formats-and-compression-codecs.md#text-format)、[Json 格式](supported-file-formats-and-compression-codecs.md#json-format)、[Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)章節。 |否 (僅適用於二進位複製案例) |
| compression | 指定此資料的壓縮類型和層級。 如需詳細資訊，請參閱[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/>支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。<br/>支援的層級為：**Optimal** 和 **Fastest**。 |否 |

>[!TIP]
>若要複製資料夾下的所有檔案，以貯體指定 **bucketName**，並以資料夾部分指定 **prefix**。<br>若要使用指定的名稱複製單一檔案，以貯體指定 **bucketName**，並以資料夾部分加上檔案名稱指定 **key**。<br>若要複製資料夾下的檔案的子集，以貯體指定 **bucketName**，並以資料夾部分加上萬用字元指定 **key**。

**範例：使用前置詞**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3Object",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "prefix": "testFolder/test",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

**範例：使用索引鍵和版本 (選擇性)**

```json
{
    "name": "AmazonS3Dataset",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "bucketName": "testbucket",
            "key": "testFolder/testfile.csv.gz",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Amazon S3 來源所支援的屬性清單。

### <a name="amazon-s3-as-source"></a>Amazon S3 作為來源

若要從 Amazon S3 複製資料，請將複製活動中的來源類型設定為 **FileSystemSource** (其中包含 Amazon S3)。 複製活動的 **source** 區段支援下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的類型屬性必須設定為：**FileSystemSource** |yes |
| 遞迴 | 表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 請注意，當 recursive 設定為 true，而接收器為檔案型存放區時，系統不會在接收器複製/建立空資料夾/子資料夾。<br/>允許的值為：**true** (預設值)、**false** | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromAmazonS3",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Amazon S3 input dataset name>",
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
                "type": "FileSystemSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="next-steps"></a>後續步驟
如需 Azure Data Factory 中的複製活動所支援作為來源和接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
