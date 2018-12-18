---
title: 使用 Azure Data Factory 從 FTP 伺服器移動資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 FTP 伺服器移動資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: eea3bab0-a6e4-4045-ad44-9ce06229c718
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: bbbbaab6090941141abd7a2bbd2eac6dbf9fd354
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37051537"
---
# <a name="move-data-from-an-ftp-server-by-using-azure-data-factory"></a>使用 Azure Data Factory 從 FTP 伺服器移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-ftp-connector.md)
> * [第 2 版 (目前的版本)](../connector-ftp.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 FTP 連接器](../connector-ftp.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，從 FTP 伺服器移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從 FTP 伺服器複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)資料表。 資料處理站目前只支援將資料從 FTP 伺服器移到其他資料存放區，而不支援將資料從其他資料存放區移到 FTP 伺服器。 它支援內部部署和雲端 FTP 伺服器。

> [!NOTE]
> 來源檔案成功複製至目的地後，「複製活動」不會將它刪除。 如果您需要在成功複製後刪除來源檔案，請建立自訂活動來刪除檔案，並在管道中使用該活動。 

## <a name="enable-connectivity"></a>啟用連線能力
如果您要將資料從內部部署 FTP 伺服器移到雲端資料存放區 (例如：移到 Azure Blob 儲存體)，請安裝並使用資料管理閘道。 資料管理閘道是安裝在內部部署電腦上的用戶端代理程式，允許雲端服務連接至內部部署資源。 如需詳細資訊，請參閱[資料管理閘道](data-factory-data-management-gateway.md)文章。 請參閱[在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)一文來了解設定和使用閘道器的逐步說明。 即使伺服器位於 Azure 架構即服務 (IaaS) 虛擬機器 (VM) 上，您還是可以使用閘道器連接至 FTP 伺服器。

您可以將閘道器安裝在與 FTP 伺服器相同的內部部署電腦或 IaaS VM 上。 不過，建議您將閘道器安裝在個別的機器 或 IaaS VM 上，除了可避免發生資源爭用的情況之外，也可獲得較佳的效能。 當您在不同電腦上安裝閘道時，電腦應該能夠存取 FTP 伺服器。

## <a name="get-started"></a>開始使用
您可以藉由使用不同的工具 或 API，建立內含複製活動的管線，以便從 FTP 來源移動資料。

建立管線的最簡單方式就是使用「資料處理站複製精靈」。 如需快速逐步解說，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

您也可以使用下列工具來建立管線︰Azure 入口網站、Visual Studio、PowerShell、Azure Resource Manager 範本、.NET API 及 REST API。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具或 API (.NET API 除外) 時，您需使用 JSON 格式來定義這些資料處理站實體。 如需相關範例，其中含有用來從 FTP 資料存放區複製資料的資料處理站實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從 FTP 伺服器複製到 Azure Blob](#json-example-copy-data-from-ftp-server-to-azure-blob) 一節。

> [!NOTE]
> 如需可用的支援檔案和壓縮格式的詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 FTP 特定的資料處理站實體：

## <a name="linked-service-properties"></a>連結服務屬性
下表提供 FTP 連結服務專屬的 JSON 元素說明。

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| type |設定為 FtpServer。 |yes |&nbsp; |
| host |指定 FTP 伺服器的名稱或 IP 位址。 |yes |&nbsp; |
| authenticationType |指定驗證類型。 |yes |基本或匿名 |
| username |指定擁有 FTP 伺服器存取權限的使用者。 |否 |&nbsp; |
| password |指定使用者 (使用者名稱) 的密碼。 |否 |&nbsp; |
| encryptedCredential |指定用來存取 FTP 伺服器的加密認證。 |否 |&nbsp; |
| gatewayName |指定資料管理閘道中連結至內部部署 FTP 伺服器的閘道器名稱。 |否 |&nbsp; |
| 連接埠 |指定 FTP 伺服器所接聽的連接埠 |否 |21 |
| enableSsl |指定是否使用透過 SSL/TLS 的 FTP 通道。 |否 |true |
| enableServerCertificateValidation |指定是否在使用透過 SSL/TLS 的 FTP 通道時啟用伺服器 SSL 憑證驗證。 |否 |true |

>[!NOTE]
>FTP 連接器支援以未加密或明確的 SSL/TLS 加密方式存取 FTP 伺服器；它不支援隱含的 SSL/TLS 加密。

### <a name="use-anonymous-authentication"></a>使用匿名驗證

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {        
            "authenticationType": "Anonymous",
              "host": "myftpserver.com"
        }
    }
}
```

### <a name="use-username-and-password-in-plain-text-for-basic-authentication"></a>使用純文字的使用者名稱和密碼進行基本驗證

```JSON
{
    "name": "FTPLinkedService",
      "properties": {
    "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
      }
}
```

### <a name="use-port-enablessl-enableservercertificatevalidation"></a>使用連接埠、enableSsl、enableServerCertificateValidation

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",    
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

### <a name="use-encryptedcredential-for-authentication-and-gateway"></a>針對驗證和閘道器使用 encryptedCredential

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "mygateway"
        }
      }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型。

不同類型資料集的 **typeProperties** 區段不同。 它提供資料集類型的特定資訊。 FileShare 類型資料集的 typeProperties 區段有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的子路徑。 使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 partitionBy ，讓資料夾路徑以配量開始和結束日期時間為基礎。 |yes |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。<br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1：`"fileFilter": "*.log"`<br/>範例 2：`"fileFilter": 2014-1-?.txt"`<br/><br/> fileFilter 適用於輸入 FileShare 資料集。 這個屬性不支援 Hadoop 分散式檔案系統 (HDFS)。 |否 |
| partitionedBy |用來指定時間序列資料的動態 folderPath 和 filename。 例如，您可以指定每小時資料參數化的 **folderPath**。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間依原樣複製檔案 (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：GZip、Deflate、BZip2 和 ZipDeflate，而支援的層級為：最佳和最快。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |
| useBinaryTransfer |指定是否使用二進位傳輸模式。 值對二進位模式為真 (為預設值)，對 ASCII 為則為假。 只有在相關聯的連結服務類型的類型為 FtpServer 時，才可以使用這個屬性。 |否 |

> [!NOTE]
> 無法同時使用 fileName 和 fileFilter。

### <a name="use-the-partionedby-property"></a>使用 partionedBy 屬性
如上一節所述，您可以利用 partitionedBy 屬性指定時間序列資料的動態 folderPath 和 fileName。

若要了解時間序列資料集、排程和配量，請參閱[建立資料集](data-factory-create-datasets.md)、[排程和執行](data-factory-scheduling-and-execution.md)以及[建立管線](data-factory-create-pipelines.md)等文章。

#### <a name="sample-1"></a>範例 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
在此範例中，{Slice} 會取代成資料處理站系統變數 SliceStart 的值 (使用指定的格式 (YYYYMMDDHH))。 SliceStart 是指配量的開始時間。 每個配量的資料夾路徑都不同。 (例如：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104。)

#### <a name="sample-2"></a>範例 2

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
如需可用來定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就複製活動而言，類型屬性會根據來源和接收的類型而有所不同。

在複製活動中，如果來源的類型為 FileSystemSource，則 typeProperties 區段會有下列可用屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

## <a name="json-example-copy-data-from-ftp-server-to-azure-blob"></a>JSON 範例：將資料從 FTP 伺服器複製到 Azure Blob
此範例示範如何將資料從 FTP 伺服器複製至 Azure Blob 儲存體。 不過，您可以使用資料處理站中的複製活動，把資料直接複製到 [支援的資料存放區及格式](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文中所述的任何接收器。  

以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。

* [FtpServer](#linked-service-properties) 類型的連結服務
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務
* [FileShare](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)
* 具有使用 [FileSystemSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)

範例會每隔一小時就把 FTP 伺服器的資料複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

### <a name="ftp-linked-service"></a>FTP 連結服務

此範例運用純文字使用者名稱和密碼來使用基本驗證。 您也可以使用下列其中一種方式：

* 匿名驗證
* 基本驗證與加密認證
* 透過 SSL/TLS 的 FTP (FTPS)

請參閱 [FTP 連結服務](#linked-service-properties)一節，來了解您可以使用的不同驗證類型。

```JSON
{
    "name": "FTPLinkedService",
    "properties": {
    "type": "FtpServer",
    "typeProperties": {
        "host": "myftpserver.com",           
        "authenticationType": "Basic",
        "username": "Admin",
        "password": "123456"
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
### <a name="ftp-input-dataset"></a>FTP 輸入資料集

此資料及係指 FTP 資料夾 `mysharedfolder` 和 `test.csv` 檔案。 管線會將檔案複製至目的地。

設定 external 為 true 會通知資料處理站服務：這是資料處理站外部的資料集而且不是由資料處理站中的活動所產生。

```JSON
{
  "name": "FTPFileInput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": "FTPLinkedService",
    "typeProperties": {
      "folderPath": "mysharedfolder",
      "fileName": "test.csv",
      "useBinaryTransfer": true
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

### <a name="azure-blob-output-dataset"></a>Azure Blob 輸出資料集

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/ftp/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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


### <a name="a-copy-activity-in-a-pipeline-with-file-system-source-and-blob-sink"></a>具有檔案系統來源和 blob 接收器的管線中複製活動

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，source 類型設為 FileSystemSource，而 sink 類型設為 BlobSink。

```JSON
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
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
        "start": "2016-08-24T18:00:00Z",
        "end": "2016-08-24T19:00:00Z"
    }
}
```
> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="next-steps"></a>後續步驟
請參閱下列文章：

* 若要了解影響 Data Factory 中資料移動 (複製活動) 效能的關鍵因素，以及各種最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。

* 如需使用複製活動來建立管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
