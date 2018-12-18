---
title: 從內部部署的 HDFS 移動資料 | Microsoft Docs
description: 深入了解如何使用 Azure Data Factory 從內部部署的 HDFS 移動資料
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 3215b82d-291a-46db-8478-eac1a3219614
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: ac9ba682079f735aa2fdd416070c5d206d526ad4
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629696"
---
# <a name="move-data-from-on-premises-hdfs-using-azure-data-factory"></a>使用 Azure Data Factory 從內部部署的 HDFS 移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-hdfs-connector.md)
> * [第 2 版 (目前的版本)](../connector-hdfs.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版的 HDFS 連接器](../connector-hdfs.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署的 HDFS 移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

您可以將資料從 HDFS 複製到任何支援的接收資料存放區。 如需複製活動所支援作為接收器的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表格。 Data Factory 目前只支援將資料從內部部署 HDFS 移動到其他資料存放區，而不支援將資料從其他資料存放區移動到內部部署 HDFS。

> [!NOTE]
> 來源檔案成功複製至目的地後，「複製活動」不會將它刪除。 如果您需要在成功複製後刪除來源檔案，請建立自訂活動來刪除檔案，並在管道中使用該活動。 

## <a name="enabling-connectivity"></a>啟用連線
Data Factory 服務支援使用資料管理閘道器連接至內部部署 HDFS。 請參閱 [在內部部署位置與雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 一文來了解資料管理閘道和設定閘道的逐步指示。 即使 HDFS 裝載於 Azure IaaS VM 中，仍請使用閘道器與其連接。

> [!NOTE]
> 請確定資料管理閘道器可以存取 Hadoop 叢集的**所有** [名稱節點伺服器]:[名稱節點連接埠] 和 [資料節點伺服器]:[資料節點連接埠]。 預設 [名稱節點連接埠] 是 50070，而預設 [資料節點連接埠] 是 50075。

雖然您可以在與 HDFS 相同的內部部署機器或 Azure VM 上安裝閘道器，仍建議您在個別的機器/Azure IaaS VM 上安裝。 在個別機器上安裝閘道器可減少資源爭用並改善效能。 當您在不同機器上安裝閘道器時，機器應該能夠存取具有 HDFS 的機器。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以從 HDFS 來源移動資料。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。
2. 建立**資料集**，代表複製作業的輸入和輸出資料。
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來從 HDFS 資料存放區複製資料之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例：將資料從內部部署 HDFS 複製到 Azure Blob](#json-example-copy-data-from-on-premises-hdfs-to-azure-blob) 一節。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 HDFS 特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
已連結的服務會將資料存放區連結到 Data Factory。 您需建立 **Hdfs** 類型的已連結服務，以將內部部署的 HDFS 連結到 Data Factory。 下表提供 HDFS 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **Hdfs** |是 |
| Url |到 HDFS 的 URL |是 |
| authenticationType |匿名或 Windows。 <br><br> 若要對 HDFS 連接器使用 **Kerberos 驗證**，請參閱[此章節](#use-kerberos-authentication-for-hdfs-connector)來據以設定您的內部部署環境。 |是 |
| userName |Windows 驗證的使用者名稱。 Kerberos 驗證請指定 `<username>@<domain>.com`。 |是 (適用於 Windows 驗證) |
| password |Windows 驗證的密碼。 |是 (適用於 Windows 驗證) |
| gatewayName |Data Factory 服務應該用來連接到 HDFS 的閘道器名稱。 |是 |
| encryptedCredential |[New-AzureRMDataFactoryEncryptValue](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactoryencryptvalue) 輸出。 |否 |

### <a name="using-anonymous-authentication"></a>使用匿名驗證

```JSON
{
    "name": "hdfs",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "userName": "hadoop",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```

### <a name="using-windows-authentication"></a>使用 Windows 驗證

```JSON
{
    "name": "hdfs",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": "password",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```
## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **FileShare** 資料集類型 (包含 HDFS 資料集) 的 typeProperties 區段具有下列屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的路徑。 範例： `myfolder`<br/><br/>使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 例如︰若為 folder\subfolder，請指定 folder\\\\subfolder；若為 d:\samplefolder，請指定 d:\\\\samplefolder。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |是 |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data<Guid>.txt (例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| partitionedBy |partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。 範例：folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

> [!NOTE]
> 無法同時使用檔名和 fileFilter。

### <a name="using-partionedby-property"></a>使用 partionedBy 屬性
如上一節所述，您可以使用 **partitionedBy** 屬性、[Data Factory 函式及系統變數](data-factory-functions-variables.md)，來指定時間序列資料的動態 folderPath 和檔案名稱。

若要進一步了解時間序列資料集、排程和配量，請參閱[建立資料集](data-factory-create-datasets.md)、[排程和執行](data-factory-scheduling-and-execution.md)以及[建立管線](data-factory-create-pipelines.md)等文章。

#### <a name="sample-1"></a>範例 1：

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
在此範例中，{Slice} 會取代成 Data Factory 系統變數 SliceStart 的值 (使用指定的格式 (YYYYMMDDHH))。 SliceStart 是指配量的開始時間。 每個配量的 folderPath 都不同。 例如：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104。

#### <a name="sample-2"></a>範例 2：

```JSON
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

而活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

在複製活動中，如果來源的類型為 **FileSystemSource** ，則 typeProperties 區段會有下列可用屬性：

**FileSystemSource** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

## <a name="supported-file-and-compression-formats"></a>支援的檔案和壓縮格式
請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)文章以了解詳細資訊。

## <a name="json-example-copy-data-from-on-premises-hdfs-to-azure-blob"></a>JSON 範例：將資料從內部部署 HDFS 複製到 Azure Blob
此範例示範如何將資料從內部部署 HDFS 複製到 Azure Blob 儲存體。 不過，您可以在 Azure Data Factory 中使用複製活動， **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。  

此範例提供下列 Data Factory 實體的 JSON 定義。 您可以透過 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)，使用這些定義來建立管線，以將資料從 HDFS 複製到「Azure Blob 儲存體」。

1. [OnPremisesHdfs](#linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
3. [FileShare](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [FileSystemSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每個小時將資料從內部部署 HDFS 複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

第一步是設定資料管理閘道。 如需相關指示，請參閱 [在內部部署位置和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 。

**HDFS 已連結的服務：** 此範例使用 Windows 驗證。 請參閱 [HDFS 連結服務](#linked-service-properties) 章節以了解您可以使用的各種不同類型的驗證。

```JSON
{
    "name": "HDFSLinkedService",
    "properties":
    {
        "type": "Hdfs",
        "typeProperties":
        {
            "authenticationType": "Windows",
            "userName": "Administrator",
            "password": "password",
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "mygateway"
        }
    }
}
```

**Azure 儲存體連結服務：**

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

**HDFS 輸入資料集：** 此資料集係指 HDFS 資料夾 DataTransfer/UnitTest/。 管線會將此資料夾中的所有檔案複製到目的地。

設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
    "name": "InputDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "HDFSLinkedService",
        "typeProperties": {
            "folderPath": "DataTransfer/UnitTest/"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
    "name": "OutputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "mycontainer/hdfs/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**具有檔案系統來源和 Blob 接收器的管線中複製活動：**

此管線包含「複製活動」，該活動已設定為使用這些輸入和輸出資料集，並且排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **FileSystemSource**，而 **sink** 類型設為 **BlobSink**。 針對 **query** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```JSON
{
    "name": "pipeline",
    "properties":
    {
        "activities":
        [
            {
                "name": "HdfsToBlobCopy",
                "inputs": [ {"name": "InputDataset"} ],
                "outputs": [ {"name": "OutputDataset"} ],
                "type": "Copy",
                "typeProperties":
                {
                    "source":
                    {
                        "type": "FileSystemSource"
                    },
                    "sink":
                    {
                        "type": "BlobSink"
                    }
                },
                "policy":
                {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "00:05:00"
                }
            }
        ],
        "start": "2014-06-01T18:00:00Z",
        "end": "2014-06-01T19:00:00Z"
    }
}
```

## <a name="use-kerberos-authentication-for-hdfs-connector"></a>對 HDFS 連接器使用 Kerberos 驗證
有兩種選項可以設定內部部署環境，以便在 HDFS 連接器中使用 Kerberos 驗證。 您可以選擇較適合您案例的選項。
* 選項 1：[將閘道電腦加入 Kerberos 領域](#kerberos-join-realm)
* 選項 2：[啟用 Windows 網域和 Kerberos 領域之間的相互信任](#kerberos-mutual-trust)

### <a name="kerberos-join-realm"></a>選項 1：將閘道電腦加入 Kerberos 領域

#### <a name="requirement"></a>需求：

* 閘道電腦必須加入 Kerberos 領域，且不可加入任何 Windows 網域。

#### <a name="how-to-configure"></a>如何設定︰

**在閘道電腦上︰**

1.  執行 **Ksetup** 公用程式來設定 Kerberos KDC 伺服器和領域。

    電腦必須設定為工作群組的成員，因為 Kerberos 領域與 Windows 網域不同。 若要進行此操作，您可以設定 Kerberos 領域並新增 KDC 伺服器，如下所示。 視需要以您自己的個別領域取代 *REALM.COM*。

            C:> Ksetup /setdomain REALM.COM
            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>

    在執行這 2 個命令後**重新啟動**電腦。

2.  使用 **Ksetup** 命令確認組態。 輸出應該會像這樣：

            C:> Ksetup
            default realm = REALM.COM (external)
            REALM.com:
                kdc = <your_kdc_server_address>

**在 Azure Data Factory 中：**

* 使用 **Windows 驗證**以及您用來連線到 HDFS 資料來源的 Kerberos 主體名稱和密碼，來設定 HDFS 連接器。 檢查組態詳細資料上的 [HDFS 連結服務屬性](#linked-service-properties)區段。

### <a name="kerberos-mutual-trust"></a>選項 2：啟用 Windows 網域和 Kerberos 領域之間的相互信任

#### <a name="requirement"></a>需求：
*   閘道電腦必須加入 Windows 網域。
*   您需要可更新網域控制站設定的權限。

#### <a name="how-to-configure"></a>如何設定︰

> [!NOTE]
> 視需要，以您自己的個別領域和網域控制站取代下列教學課程中的 REALM.COM 和 AD.COM。

**在 KDC 伺服器上︰**

1.  編輯 **krb5.conf** 檔案中的 KDC 組態，讓 KDC 信任參考下列組態範本的 Windows 網域。 根據預設，此組態位於 **/etc/krb5.conf**。

            [logging]
             default = FILE:/var/log/krb5libs.log
             kdc = FILE:/var/log/krb5kdc.log
             admin_server = FILE:/var/log/kadmind.log

            [libdefaults]
             default_realm = REALM.COM
             dns_lookup_realm = false
             dns_lookup_kdc = false
             ticket_lifetime = 24h
             renew_lifetime = 7d
             forwardable = true

            [realms]
             REALM.COM = {
              kdc = node.REALM.COM
              admin_server = node.REALM.COM
             }
            AD.COM = {
             kdc = windc.ad.com
             admin_server = windc.ad.com
            }

            [domain_realm]
             .REALM.COM = REALM.COM
             REALM.COM = REALM.COM
             .ad.com = AD.COM
             ad.com = AD.COM

            [capaths]
             AD.COM = {
              REALM.COM = .
             }

  設定之後**重新啟動** KDC 服務。

2.  使用下列命令，在 KDC 伺服器中準備名為 **krbtgt/REALM.COM@AD.COM** 的主體︰

            Kadmin> addprinc krbtgt/REALM.COM@AD.COM

3.  在 **hadoop.security.auth_to_local** HDFS 服務組態檔中，新增 `RULE:[1:$1@$0](.*@AD.COM)s/@.*//`。

**在網域控制站上：**

1.  執行下列 **Ksetup** 命令來新增領域項目：

            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
            C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM

2.  建立 Windows 網域到 Kerberos 領域的信任關係。 [password] 是主體 **krbtgt/REALM.COM@AD.COM** 的密碼。

            C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /passwordt:[password]

3.  選取 Kerberos 所使用的加密演算法。

    1. 移至 [伺服器管理員] > [群組原則管理] > [網域] > [群組原則物件] > [預設或作用中的網域原則]，然後進行編輯。

    2. 在 [群組原則管理編輯器] 快顯視窗中，移至 [電腦設定] > [原則] > [Windows 設定] > [安全性設定] > [本機原則] > [安全性選項]，並設定 [網路安全性: 設定 Kerberos 允許的加密類型]。

    3. 選取您想要在連線至 KDC 時使用的加密演算法。 一般來說，您可以直接選取所有選項。

        ![設定 Kerberos 的加密類型](media/data-factory-hdfs-connector/config-encryption-types-for-kerberos.png)

    4. 使用 **Ksetup** 命令，指定要用於特定 REALM 的加密演算法。

                C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96

4.  建立網域帳戶與 Kerberos 主體之間的對應，以便在 Windows 網域中使用 Kerberos 主體。

    1. 啟動 [系統管理工具] > [Active Directory 使用者和電腦]。

    2. 按一下 [檢視] > [進階功能] 來設定進階功能。

    3. 找到您要用以建立對應的帳戶，然後按一下滑鼠右鍵以檢視 [名稱對應] > 按一下 [Kerberos 名稱] 索引標籤。

    4. 從領域中新增主體。

        ![對應安全性身分識別](media/data-factory-hdfs-connector/map-security-identity.png)

**在閘道電腦上︰**

* 執行下列 **Ksetup** 命令來新增領域項目。

            C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
            C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM

**在 Azure Data Factory 中：**

* 使用 **Windows 驗證**以及您用來連線到 HDFS 資料來源的網域帳戶或 Kerberos 主體，來設定 HDFS 連接器。 檢查組態詳細資料上的 [HDFS 連結服務屬性](#linked-service-properties)區段。

> [!NOTE]
> 若要將來源資料集中的資料行對應至接收資料集中的資料行，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。


## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。
