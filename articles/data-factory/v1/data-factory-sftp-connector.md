---
title: 使用 Azure Data Factory 從 SFTP 伺服器移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從內部部署或雲端 SFTP 伺服器移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/12/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: c22d2cba23e8bae965fa7c5746c9fff69ad3fa9e
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054410"
---
# <a name="move-data-from-an-sftp-server-using-azure-data-factory"></a>使用 Azure Data Factory 從 SFTP 伺服器移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-sftp-connector.md)
> * [第 2 版 (目前的版本)](../connector-sftp.md)

> [!NOTE]
> 本文適用於第 1 版的 Data Factory。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 SFTPconnector](../connector-sftp.md)。

本文概述如何使用 Azure Data Factory 中的複製活動，將內部部署/雲端 SFTP 伺服器中的資料移動到支援的接收資料存放區。 本文是根據 [資料移動活動](data-factory-data-movement-activities.md)一文，該文呈現使用複製活動移動資料的一般概觀以及支援作為來源/接收的資料存放區清單。

資料處理站目前只支援將資料從 SFTP 伺服器移到其他資料存放區，而不支援將資料從其他資料存放區移到 SFTP 伺服器。 它支援內部部署和雲端 SFTP 伺服器。

> [!NOTE]
> 來源檔案成功複製至目的地後，「複製活動」不會將它刪除。 如果您需要在成功複製後刪除來源檔案，請建立自訂活動來刪除檔案，並在管道中使用該活動。 

## <a name="supported-scenarios-and-authentication-types"></a>支援的案例和驗證類型
您可以使用此 SFTP 連接器，從**雲端 SFTP 伺服器和內部部署 SFTP 伺服器**複製資料。 連線至 SFTP 伺服器時支援 **Basic** 和 **SshPublicKey** 驗證類型。

從內部部署 SFTP 伺服器複製資料時，您需要在內部部署環境/Azure VM 中安裝資料管理閘道。 如需資料管理閘的詳細資料，請參閱[資料管理閘道](data-factory-data-management-gateway.md)。 請參閱[在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文來了解設定和使用閘道的逐步指示。

## <a name="getting-started"></a>開始使用
您可以建立內含複製活動的管線，使用不同的工具/API 將資料移出 SFTP 來源。

- 若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

- 您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 若要將資料從 SFTP 伺服器複製到 Azure Blob 儲存體的 JSON 範例，請參閱本文的 [JSON 範例：將資料從 SFTP 伺服器複製到 Azure Blob](#json-example-copy-data-from-sftp-server-to-azure-blob)一節。

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 FTP 連結服務專屬 JSON 元素的說明。

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| type | 類型屬性必須設為 `Sftp`。 |yes |
| host | SFTP 伺服器的名稱或 IP 位址。 |yes |
| 連接埠 |SFTP 伺服器所接聽的連接埠。 預設值：21 |否 |
| authenticationType |指定驗證類型。 允許的值︰**Basic**、**SshPublicKey**。 <br><br> 請參閱[使用基本驗證](#using-basic-authentication)和[使用 SSH 公開金鑰驗證](#using-ssh-public-key-authentication)章節，分別取得更多屬性和 JSON 範例。 |yes |
| skipHostKeyValidation | 指定是否略過主機金鑰驗證。 | 否。 預設值：false |
| hostKeyFingerprint | 指定主機金鑰的指紋。 | 如果 `skipHostKeyValidation` 設為 false，則為 [是]。  |
| gatewayName |要連線至內部部署 SFTP 伺服器的資料管理閘道名稱。 | 如果從內部部署 SFTP 伺服器複製資料，則為 [是]。 |
| encryptedCredential | 用來存取 SFTP 伺服器的加密認證。 當您在複製精靈或 ClickOnce 快顯對話方塊中指定基本驗證 (使用者名稱 + 密碼) 或 SshPublicKey 驗證 (使用者名稱 + 私密金鑰路徑或內容) 時自動產生。 | 否。 僅當從內部部署 SFTP 伺服器複製資料時才套用。 |

### <a name="using-basic-authentication"></a>使用基本驗證

若要使用基本驗證，將 `authenticationType` 設定為 `Basic`，然後指定上一節中介紹的 SFTP 連接器泛用以外的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| username | 可存取 SFTP 伺服器的使用者。 |yes |
| password | 使用者 (使用者名稱) 的密碼。 | yes |

#### <a name="example-basic-authentication"></a>範例：基本驗證
```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>範例：採用加密認證的基本驗證

```JSON
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
      }
}
```

### <a name="using-ssh-public-key-authentication"></a>使用 SSH 公用金鑰驗證

若要使用 SSH 公開金鑰驗證，將 `authenticationType` 設定為 `SshPublicKey`，然後指定上一節中介紹的 SFTP 連接器泛用以外的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| username |可存取 SFTP 伺服器的使用者 |yes |
| privateKeyPath | 指定閘道可以存取之私密金鑰檔案的絕對路徑。 | 指定 `privateKeyPath` 或 `privateKeyContent`。 <br><br> 僅當從內部部署 SFTP 伺服器複製資料時才套用。 |
| privateKeyContent | 私密金鑰內容的序列化字串。 複製精靈可以讀取私密金鑰檔案，並自動解壓縮私密金鑰內容。 如果您使用任何其他工具/SDK，請改為使用 privateKeyPath 屬性。 | 指定 `privateKeyPath` 或 `privateKeyContent`。 |
| passPhrase | 如果金鑰檔案受到複雜密碼保護，請指定複雜密碼/密碼以將私密金鑰解密。 | 如果私密金鑰檔案受到複雜密碼保護，則為 [是]。 |

> [!NOTE]
> SFTP 連接器支援 RSA/DSA OpenSSH 金鑰。 確定您的金鑰檔案內容是以 "-----BEGIN [RSA/DSA] PRIVATE KEY-----" 開頭。 如果私密金鑰檔案是 ppk 格式檔案，請使用 Putty 工具，從 .ppk 轉換為 OpenSSH 格式。

#### <a name="example-sshpublickey-authentication-using-private-key-filepath"></a>範例︰使用私密金鑰 filePath 的 SshPublicKey 驗證

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "mygateway"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>範例︰使用私密金鑰內容的 SshPublicKey 驗證

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型。

不同類型資料集的 **typeProperties** 區段不同。 它提供資料集類型的特定資訊。 **FileShare** 資料集類型的 typeProperties 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的子路徑。 使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |yes |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data.<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。<br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1：`"fileFilter": "*.log"`<br/>範例 2：`"fileFilter": 2014-1-?.txt"`<br/><br/> fileFilter 適用於輸入 FileShare 資料集。 這個屬性不支援使用 HDFS。 |否 |
| partitionedBy |partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |
| useBinaryTransfer |指定是否使用二進位傳輸模式。 二進位模式為 true，ASCII 則為 false。 預設值：True。 只有在相關聯的連結服務類型的類型為 FtpServer 時，才可以使用這個屬性。 |否 |

> [!NOTE]
> 無法同時使用檔名和 fileFilter。

### <a name="using-partionedby-property"></a>使用 partionedBy 屬性
如上一節所述，您可以利用 partitionedBy 指定時間序列資料的動態 folderPath 和 filename。 您可以使用 Data Factory 巨集和系統變數以及可指出給定資料配量的邏輯時間週期的 SliceStart、SliceEnd 來這麼做。

若要了解時間序列資料集、排程和配量，請參閱[建立資料集](data-factory-create-datasets.md)、[排程和執行](data-factory-scheduling-and-execution.md)以及[建立管線](data-factory-create-pipelines.md)等文章。

#### <a name="sample-1"></a>範例 1：

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
在此範例中，{Slice} 會取代成 Data Factory 系統變數 SliceStart 的值 (使用指定的格式 (YYYYMMDDHH))。 SliceStart 是指配量的開始時間。 每個配量的 folderPath 都不同。 範例：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104。

#### <a name="sample-2"></a>範例 2：

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
在此範例中，SliceStart 的年、月、日和時間會擷取到 folderPath 和 fileName 屬性所使用的個別變數。

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

然而，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就複製活動而言，類型屬性會根據來源和接收的類型而有所不同。

[!INCLUDE [data-factory-file-system-source](../../../includes/data-factory-file-system-source.md)]

## <a name="supported-file-and-compression-formats"></a>支援的檔案和壓縮格式
請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)文章以了解詳細資訊。

## <a name="json-example-copy-data-from-sftp-server-to-azure-blob"></a>JSON 範例：將資料從 SFTP 伺服器複製到 Azure Blob
下列範例提供您使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線時，可使用的範例 JSON 定義。 這些範例示範如何將資料從 SFTP 來源複製到 Azure Blob 儲存體。 不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源 **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

> [!IMPORTANT]
> 此範例提供 JSON 程式碼片段。 其中並不包含建立 Data Factory 的逐步指示。 如需逐步指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文。

此範例具有下列 Data Factory 實體：

* [sftp](#linked-service-properties) 類型的已連結服務。
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [FileShare](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [FileSystemSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

範例會每隔一小時就把 SFTP 伺服器的資料複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**SFTP 連結服務**

此範例會利用純文字使用者名稱和密碼來使用基本驗證。 您也可以使用下列其中一種方式：

* 基本驗證與加密認證
* SSH 公用金鑰驗證

請參閱 [FTP 連結服務](#linked-service-properties)一節，來了解您可以使用的不同驗證類型。

```JSON

{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver",
            "port": 22,
            "authenticationType": "Basic",
            "username": "myuser",
            "password": "mypassword",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "mygateway"
        }
    }
}
```
**Azure 儲存體連結服務**

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
**SFTP 輸入資料集**

此資料集是指 SFTP 資料夾 `mysharedfolder` 和 `test.csv` 檔案。 管線會將檔案複製至目的地。

設定 "external": "true" 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "SFTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "SftpLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/sftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            },
            "partitionedBy": [
                {
                    "name": "Year",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "yyyy"
                    }
                },
                {
                    "name": "Month",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "MM"
                    }
                },
                {
                    "name": "Day",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "dd"
                    }
                },
                {
                    "name": "Hour",
                    "value": {
                        "type": "DateTime",
                        "date": "SliceStart",
                        "format": "HH"
                    }
                }
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **FileSystemSource**，而 **sink** 類型設為 **BlobSink**。

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
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
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2017-02-20T18:00:00Z",
        "end": "2017-02-20T19:00:00Z"
    }
}
```

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。

## <a name="next-steps"></a>後續步驟
請參閱下列文章：

* [複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) ，以取得使用「複製活動」來建立管線的逐步指示。
