---
title: 使用 Data Factory 將資料複製到 Azure Blob 儲存體或從該處複製資料 | Microsoft Docs
description: 了解如何使用 Data Factory 將資料從支援的來源資料存放區複製到「Azure Blob 儲存體」，或從「Blob 儲存體」複製到支援的接收資料存放區。
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/17/2018
ms.author: jingwang
ms.openlocfilehash: 46e12378812788d147c903046b50a93c13119f2f
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444583"
---
# <a name="copy-data-to-or-from-azure-blob-storage-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料複製到 Azure Blob 儲存體或從該處複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-azure-blob-connector.md)
> * [目前的版本](connector-azure-blob-storage.md)

本文概述如何使用 Azure Data Factory 中的「複製活動」，將資料複製到「Azure Blob 儲存體」及從該處複製資料。 本文是根據[複製活動概觀](copy-activity-overview.md)一文，該文提供複製活動的一般概觀。

## <a name="supported-capabilities"></a>支援的功能

您可以將資料從支援的來源資料存放區複製到 Blob 儲存體。 您也可以將資料從 Blob 儲存體複製到任何支援的接收資料存放區。 如需複製活動所支援作為來源或接收器的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md)表格。

具體而言，這個 Blob 儲存體連接器支援：

- 將 Blob 複製到一般用途 Azure 儲存體帳戶和經常性存取/非經常性存取 Blob 儲存體，或從這兩處複製 Blob。 
- 使用帳戶金鑰、服務共用存取簽章、服務主體或受控服務識別驗證來複製 Blob。
- 從區塊、附加或分頁 Blob 複製 Blob，以及將資料只複製到區塊 Blob。 不支援使用「Azure 進階儲存體」作為接收，因為它是以分頁 Blob 為後盾。
- 依原樣複製 Blob，或是使用[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md)來剖析或產生 Blob。

## <a name="get-started"></a>開始使用

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

下列各節提供屬性的相關詳細資料，這些屬性是用來定義 Blob 儲存體專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性

Azure Blob 連接器支援下列驗證類型，請參閱詳細資料的對應章節：

- [帳戶金鑰驗證](#account-key-authentication)
- [共用存取簽章驗證](#shared-access-signature-authentication)
- [服務主體驗證](#service-principal-authentication)
- [受控服務識別驗證](#managed-service-identity-authentication)

>[!NOTE]
>HDInsights、Azure Machine Learning 和 Azure SQL 資料倉儲 PolyBase 負載僅支援 Azure Blob 儲存體帳戶金鑰驗證。

### <a name="account-key-authentication"></a>帳戶金鑰驗證

若要使用儲存體帳戶金鑰驗證，以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為**AzureBlobStorage** (建議) 或 **AzureStorage** (請參閱下面附註)。 |是 |
| connectionString | 針對 connectionString 屬性指定連線到儲存體所需的資訊。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載 Integration Runtime (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>如果您使用 "AzureStorage" 類型連結服務，它仍然如同現狀受支援，但建議您從現在開始使用這個新的 "AzureBlobStorage" 連結服務類型。

**範例：**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "connectionString": {
                "type": "SecureString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="shared-access-signature-authentication"></a>共用存取簽章驗證

共用存取簽章可提供您儲存體帳戶中資源的委派存取。 您可以使用共用存取簽章來將儲存體帳戶中物件的有限權限授與用戶端，讓該用戶端可以在一段指定時間內使用。 您不需要共用您的帳戶存取金鑰。 共用存取簽章是一種 URI，此 URI 會在其查詢參數中包含對儲存體資源進行驗證式存取所需的一切資訊。 若要使用共用存取簽章存取儲存體資源，用戶端只需在適當的建構函式或方法中傳入共用存取簽章即可。 如需共用存取簽章的詳細資訊，請參閱[共用存取簽章：了解共用存取簽章模型](../storage/common/storage-dotnet-shared-access-signature-part-1.md)。

> [!NOTE]
> Data Factory 現已可支援**服務共用存取簽章**和**帳戶共用存取簽章**。 如需這兩種類型及其建構方式的詳細資訊，請參閱[共用存取簽章的類型](../storage/common/storage-dotnet-shared-access-signature-part-1.md#types-of-shared-access-signatures)。 

> [!TIP]
> 若要為儲存體帳戶產生服務共用存取簽章，您可以執行下列 PowerShell 命令。 取代預留位置，並授與所需的權限。
> `$context = New-AzureStorageContext -StorageAccountName <accountName> -StorageAccountKey <accountKey>`
> `New-AzureStorageContainerSASToken -Name <containerName> -Context $context -Permission rwdl -StartTime <startTime> -ExpiryTime <endTime> -FullUri`

若要使用共用存取簽章驗證，以下是支援的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為**AzureBlobStorage** (建議) 或 **AzureStorage** (請參閱下面附註)。 |是 |
| sasUri | 指定儲存體資源 (例如 Blob、容器或資料表) 的共用存取簽章 URI。 將此欄位標記為 SecureString，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載 Integration Runtime (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>如果您使用 "AzureStorage" 類型連結服務，它仍然如同現狀受支援，但建議您從現在開始使用這個新的 "AzureBlobStorage" 連結服務類型。

**範例：**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {
            "sasUri": {
                "type": "SecureString",
                "value": "<SAS URI of the Azure Storage resource>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

當您建立共用存取簽章 URI 時，請考慮下列各點：

- 根據連結的服務 (讀取、寫入、讀取/寫入) 在資料處理站中的使用方式，在物件上設定適當的讀取/寫入權限。
- 設定適當的 [過期時間]。 請確定儲存體物件的存取權不會在管線的作用中期間內過期。
- URI 應根據需要在正確的容器/Blob 或資料表層級建立。 Blob 的共用存取簽章 URI 可讓 Data Factory 存取該特定 Blob。 Blob 儲存體容器的共用存取簽章 URI 可讓 Data Factory 逐一查看該容器中 Blob。 若要在稍後提供更多或較少物件的存取權，或更新共用存取簽章 URI，請記得使用新的 URI 更新連結的服務。

### <a name="service-principal-authentication"></a>服務主體驗證

如需一般的 Azure 儲存體服務主體驗證，請參閱[使用 Azure Active Directory 驗證 Azure 儲存體的存取權](../storage/common/storage-auth-aad.md)。

若要使用服務主體驗證，請遵循下列步驟：

1. 遵循[使用 Azure AD 租用戶註冊應用程式](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant)，以在 Azure Active Directory (Azure AD) 中註冊應用程式實體。 請記下以下的值，您可以使用這些值來定義連結服務：

    - 應用程式識別碼
    - 應用程式金鑰
    - 租用戶識別碼

2. 在 Azure Blob 儲存體中授與服務主體適當權限。 請參閱[使用 RBAC 管理 Azure 儲存體資料的存取權限](../storage/common/storage-auth-aad-rbac.md)，以取得角色的更多詳細資訊。

    - **作為來源**，在存取控制 (IAM) 中，至少授與 [儲存體 Blob 資料讀取者] 角色。
    - **作為接收**，在存取控制 (IAM) 中，至少授與 [儲存體 Blob 資料參與者] 角色。

以下是支援 Azure Blob 儲存體連結服務的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為 **AzureStorageSas**。 |是 |
| serviceEndpoint | 指定模式為 `https://<accountName>.blob.core.windows.net/` 的 Azure Blob 儲存體服務端點。 |是 |
| servicePrincipalId | 指定應用程式的用戶端識別碼。 | 是 |
| servicePrincipalKey | 指定應用程式的金鑰。 將此欄位標記為 **SecureString**，將它安全地儲存在 Data Factory 中，或[參考 Azure Key Vault 中儲存的祕密](store-credentials-in-key-vault.md)。 | 是 |
| tenant | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | 是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載 Integration Runtime (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>服務主體驗證僅由 "AzureBlobStorage" 類型連結服務支援，但先前的 "AzureStorage" 類型連結服務不提供支援。

**範例：**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>" 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-service-identity-authentication"></a>受控服務識別驗證

資料處理站[受控服務識別](data-factory-service-identity.md)相關聯，用後者來表示此特定資料處理站。 您可以直接將此服務識別用於 Blob 儲存體驗證，類似於使用您自己的服務主體。 這可以讓這個指定的處理站從 Data Lake Store 存取及複製資料，或存取及複製資料至 Blob 儲存體。

如需一般的 Azure 儲存體 MSI 驗證，請參閱[使用 Azure Active Directory 驗證 Azure 儲存體的存取權](../storage/common/storage-auth-aad.md)。

若要使用受控服務識別 (MSI) 驗證，請遵循下列步驟：

1. [擷取資料處理站服務識別](data-factory-service-identity.md#retrieve-service-identity)，做法是複製與資料處理站一起產生的 "SERVICE IDENTITY APPLICATION ID"。

2. 在 Azure Blob 儲存體中授與服務主體適當權限。 請參閱[使用 RBAC 管理 Azure 儲存體資料的存取權限](../storage/common/storage-auth-aad-rbac.md)，以取得角色的更多詳細資訊。

    - **作為來源**，在存取控制 (IAM) 中，至少授與 [儲存體 Blob 資料讀取者] 角色。
    - **作為接收**，在存取控制 (IAM) 中，至少授與 [儲存體 Blob 資料參與者] 角色。

以下是支援 Azure Blob 儲存體連結服務的屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 類型屬性必須設定為 **AzureStorageSas**。 |是 |
| serviceEndpoint | 指定模式為 `https://<accountName>.blob.core.windows.net/` 的 Azure Blob 儲存體服務端點。 |是 |
| connectVia | 用來連線到資料存放區的[整合執行階段](concepts-integration-runtime.md)。 您可以使用 Azure Integration Runtime 或自我裝載 Integration Runtime (如果您的資料存放區位於私人網路中)。 如果未指定，就會使用預設的 Azure Integration Runtime。 |否 |

>[!NOTE]
>受控服務識別驗證僅由 "AzureBlobStorage" 類型連結服務支援，但先前的 "AzureStorage" 類型連結服務不提供支援。 

**範例：**

```json
{
    "name": "AzureBlobStorageLinkedService",
    "properties": {
        "type": "AzureBlobStorage",
        "typeProperties": {            
            "serviceEndpoint": "https://<accountName>.blob.core.windows.net/"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 Blob 儲存體資料集所支援的屬性清單。

若要將資料複製到 Blob 儲存體或從該處複製資料，請將資料集的類型屬性設定為 **AzureBlob**。 以下是支援的屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 資料集的 type 屬性必須設定為 **AzureBlob**。 |是 |
| folderPath | Blob 儲存體中容器和資料夾的路徑。 不支援萬用字元篩選。 範例是 myblobcontainer/myblobfolder/。 |是 |
| fileName | 在指定 "folderPath" 下之 Blob 的**名稱或萬用字元篩選**。 如果沒有為此屬性指定值，資料集就會指向資料夾中的所有 Blob。 <br/><br/>針對篩選，允許的萬用字元為：`*` (符合零或多個字元) 和 `?` (符合零或單一字元)。<br/>- 範例 1：`"fileName": "*.csv"`<br/>- 範例 2：`"fileName": "???20180427.txt"`<br/>如果實際檔案名稱內有萬用字元或逸出字元 `^`，請使用此逸出字元來逸出。<br/><br/>當沒有針對輸出資料集指定 fileName，且活動接收中沒有指定 **preserveHierarchy** 時，複製活動會自動使用以下模式產生 Blob 名稱："Data.[活動執行識別碼 GUID].[GUID (若為 FlattenHierarchy)].[格式 (若有設定)].[壓縮 (若有設定)]"。 範例："Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt.gz"。 |否 |
| format | 如果您想要在檔案型存放區之間依原樣複製檔案 (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。<br/><br/>如果您想要以特定格式來剖析或產生檔案，以下是支援的檔案格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat** 和 **ParquetFormat**。 將 [format] 下的 [type] 屬性設定為下列其中一個值。 如需詳細資訊，請參閱[文字格式](supported-file-formats-and-compression-codecs.md#text-format)、[JSON 格式](supported-file-formats-and-compression-codecs.md#json-format)、[Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、[Orc 格式](supported-file-formats-and-compression-codecs.md#orc-format)和 [Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)小節。 |否 (僅適用於二進位複製案例) |
| compression | 指定此資料的壓縮類型和層級。 如需詳細資訊，請參閱[支援的檔案格式和壓縮轉碼器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/>支援的類型為：GZip、Deflate、BZip2 及 ZipDeflate。<br/>支援的層級為 **Optimal** 和 **Fastest**。 |否 |

>[!TIP]
>若要複製資料夾下的所有 Blob，只要指定 **folderPath**。<br>若要複製指定名稱的單一 Blob，以資料夾部分指定 **folderPath**，並以檔案名稱指定 **fileName**。<br>若要複製資料夾下的 Blob 子集，以資料夾部分指定 **folderPath**，並以萬用字元篩選指定 **fileName**。 

**範例：**

```json
{
    "name": "AzureBlobDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": {
            "referenceName": "<Azure Blob storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "mycontainer/myfolder",
            "fileName": "myfile.csv.gz",
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

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 Blob 儲存體來源和接收所支援的屬性清單。

### <a name="blob-storage-as-a-source-type"></a>Blob 儲存體作為來源類型

若要從 Blob 儲存體複製資料，請將複製活動中的來源類型設定為 **BlobSource**。 複製活動的 [來源] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動來源的 type 屬性必須設定為 **BlobSource**。 |是 |
| 遞迴 | 指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 請注意，當遞迴設定為 true 且接收是檔案型存放區時，就不會在接收上複製或建立空的資料夾或子資料夾。<br/>允許的值為 **true** (預設值) 和 **false**。 | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyFromBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Blob input dataset name>",
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
                "type": "BlobSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="blob-storage-as-a-sink-type"></a>Blob 儲存體作為接收類型

若要將資料複製到 Blob 儲存體，請將複製活動中的接收類型設定為 **BlobSink**。 [接收] 區段支援下列屬性。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | 複製活動接收的 type 屬性必須設定為 **BlobSink**。 |是 |
| copyBehavior | 當來源是來自檔案型資料存放區的檔案時，會定義複製行為。<br/><br/>允許的值包括：<br/><b>- PreserveHierarchy (預設值)</b>：保留目標資料夾中的檔案階層。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><b>- FlattenHierarchy</b>：來自來源資料夾的所有檔案都在目標資料夾的第一層中。 目標檔案會有自動產生的名稱。 <br/><b>- MergeFiles</b>：將來源資料夾的所有檔案合併成一個檔案。 如果有指定檔案或 Blob 名稱，合併檔案的名稱會是指定的名稱。 否則，就會是自動產生的檔案名稱。 | 否 |

**範例：**

```json
"activities":[
    {
        "name": "CopyToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Blob output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "BlobSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

### <a name="some-recursive-and-copybehavior-examples"></a>一些遞迴和 copyBehavior 範例

本節說明遞迴和 copyBehavior 值在不同組合的情況下，複製作業所產生的行為。

| 遞迴 | copyBehavior | 來源資料夾結構 | 產生的目標 |
|:--- |:--- |:--- |:--- |
| true |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以與來源相同的結構，建立目標資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File1 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File2 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File3 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File4 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File5 自動產生的名稱 |
| true |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File5 的內容會合併成一個檔案，並具有自動產生的檔案名稱。 |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File1 自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;針對 File2 自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的內容會合併成一個檔案，並具有自動產生的檔案名稱。 針對 File1 自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |

## <a name="next-steps"></a>後續步驟
如需 Data Factory 中的複製活動所支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md##supported-data-stores-and-formats)。
