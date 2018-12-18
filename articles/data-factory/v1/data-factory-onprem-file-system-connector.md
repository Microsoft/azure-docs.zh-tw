---
title: 使用 Azure Data Factory 從檔案系統來回複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從內部部署檔案系統來回複製資料。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: ce19f1ae-358e-4ffc-8a80-d802505c9c84
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/13/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 7ab38c689cb6445bc85a942fc350c2a1f5de7912
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37047038"
---
# <a name="copy-data-to-and-from-an-on-premises-file-system-by-using-azure-data-factory"></a>使用 Azure Data Factory 從內部部署檔案系統來回複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-onprem-file-system-connector.md)
> * [第 2 版 (目前的版本)](../connector-file-system.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的檔案系統連接器](../connector-file-system.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，從內部部署檔案系統來回複製資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

## <a name="supported-scenarios"></a>支援的案例
您可以從內部部署檔案系統將資料複製到下列資料存放區：

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製到內部部署檔案系統：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> 來源檔案成功複製至目的地後，「複製活動」不會將它刪除。 如果您需要在成功複製後刪除來源檔案，請建立自訂活動來刪除檔案，並在管道中使用該活動。 

## <a name="enabling-connectivity"></a>啟用連線
Data Factory 支援透過「資料管理閘道」連接到內部部署的檔案系統，或從內部部署的檔案系統連出。 您必須在內部部署環境安裝「資料管理閘道」，Data Factory 服務才能連接到任何支援的內部部署資料存放區 (包括檔案系統)。 如需有關資料管理閘道以及設定閘道的逐步指示，請參閱[利用資料管理閘道在內部部署資源和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)。 除了資料管理閘道，不需要安裝其他二進位檔即可和內部部署檔案系統進行通訊。 您必須安裝和使用「資料管理閘道」，即使檔案系統位於 Azure IaaS VM 中也一樣。 如需有關閘道的詳細資訊，請參閱[資料管理閘道](data-factory-data-management-gateway.md)。

若要使用 Linux 檔案共用，請在 Linux 伺服器上安裝 [Samba](https://www.samba.org/)，在 Windows 伺服器上安裝「資料管理閘道」。 不支援在 Linux 伺服器上安裝資料管理閘道。

## <a name="getting-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出檔案系統。

若要建立管線，最簡單的方式就是使用**複製精靈**。 如需使用複製資料精靈建立管線的快速逐步解說，請參閱 [教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md) 。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您從 Azure Blob 儲存體將資料複製到內部部署檔案系統，您會建立兩個連結服務，將內部部署檔案系統和 Azure 儲存體帳戶連結至資料處理站。 有關內部部署檔案系統專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。
3. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您可以建立資料集來指定包含輸入資料的 Blob 容器與資料夾。 同時，您會建立另一個資料集來指定檔案系統中的資料夾與檔案名稱 (選擇性)。 針對內部部署檔案系統專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
4. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 BlobSource 作為來源，以及使用 FileSystemSink 作為複製活動的接收器。 同樣地，如果您是從內部部署檔案系統複製到 Azure Blob 儲存體，則在複製活動中使用 FileSystemSource 和 BlobSink。 針對內部部署檔案系統專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，按一下上一節中資料存放區的連結。

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到檔案系統 (或從檔案系統複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-to-and-from-file-system)一節。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義檔案系統特定的 Data Factory 實體：

## <a name="linked-service-properties"></a>連結服務屬性
您可以利用「內部部署檔案伺服器」已連結服務，將內部部署的檔案系統連結到 Azure Data Factory。 下表說明內部部署檔案伺服器連結服務專屬的 JSON 元素。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |確保 type 屬性設為 **OnPremisesFileServer**。 |yes |
| host |指定想要複製之資料夾的根路徑。 字串中的特殊字元需使用逸出字元 ‘ \ ’。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。 |yes |
| userid |指定具有伺服器存取權之使用者的識別碼。 |否 (如果您選擇 encryptedCredential) |
| password |指定使用者 (userid) 的密碼。 |否 (如果您選擇 encryptedCredential) |
| encryptedCredential |指定可以透過執行 New-AzureRmDataFactoryEncryptValue Cmdlet 來取得的加密認證。 |否 (如果您選擇以純文字指定使用者識別碼和密碼) |
| gatewayName |指定 Data Factory 應該用來連接到內部部署檔案伺服器的閘道器名稱。 |yes |


### <a name="sample-linked-service-and-dataset-definitions"></a>範例連結服務和資料集定義
| 案例 | 連結服務定義中的主機 | 資料集定義中的 folderPath |
| --- | --- | --- |
| 資料管理閘道電腦上的本機資料夾︰ <br/><br/>範例：D:\\\* 或 D:\folder\subfolder\\* |D:\\\\ (適用於資料管理閘道 2.0 和更新版本) <br/><br/> localhost (適用於比資料管理閘道 2.0 更早的版本) |.\\\\ 或 folder\\\\subfolder (適用於資料管理閘道 2.0 和更新版本) <br/><br/>D:\\\\ 或 D:\\\\folder\\\\subfolder (適用低於閘道 2.0 的版本) |
| 遠端共用資料夾︰ <br/><br/>範例︰\\\\myserver\\share\\\* 或 \\\\myserver\\share\\folder\\subfolder\\* |\\\\\\\\myserver\\\\share |.\\\\ 或 folder\\\\subfolder |

>[!NOTE]
>透過 UI 撰寫時，您不需要像透過 JSON 那樣輸入雙反斜線 (`\\`) 來逸出，只需指定單一反斜線。

### <a name="example-using-username-and-password-in-plain-text"></a>範例：使用純文字的使用者名稱和密碼

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

### <a name="example-using-encryptedcredential"></a>範例：使用 encryptedcredential

```JSON
{
  "Name": " OnPremisesFileServerLinkedService ",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "D:\\",
      "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
      "gatewayName": "mygateway"
    }
  }
}
```

## <a name="dataset-properties"></a>資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型。

不同類型資料集的 TypeProperties 區段不同。 它提供各種資訊，例如資料存放區中資料的格式與位置。 **FileShare** 類型資料集的 typeProperties 區段有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |指定資料夾的子路徑。 針對字串中的特殊字元，請使用逸出字元 '\'。 不支援萬用字元篩選。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |yes |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>當未指定輸出資料集的 fileName，且在活動接收中未指定 preserveHierarchy 時，所產生檔案的名稱格式如下： <br/><br/>`Data.<Guid>.txt` (例如： Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。 <br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1："fileFilter": "*.log"<br/>範例 2："fileFilter": 2014-1-?.txt"<br/><br/>請注意，fileFilter 適用於輸入 FileShare 資料集。 |否 |
| partitionedBy |您可以使用 partitionedBy 來指定時間序列資料的動態 folderPath/fileName。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

> [!NOTE]
> 無法同時使用 fileName 和 fileFilter。

### <a name="using-partitionedby-property"></a>使用 partitionedBy 屬性
如上一節所述，您可以使用 **partitionedBy** 屬性、[Data Factory 函式及系統變數](data-factory-functions-variables.md)，來指定時間序列資料的動態 folderPath 和檔案名稱。

若要進一步了解時間序列資料集、排程和配量，請參閱[建立資料集](data-factory-create-datasets.md)、[排程和執行](data-factory-scheduling-and-execution.md)以及[建立管線](data-factory-create-pipelines.md)。

#### <a name="sample-1"></a>範例 1：

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

在此範例中，{Slice} 會取代成 Data Factory 系統變數 SliceStart 的值，其格式為 (YYYYMMDDHH)。 SliceStart 是指配量的開始時間。 每個配量的 folderPath 都不同。 例如：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104。

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

在此範例中，會擷取 SliceStart 的年、月、日和時間給 folderPath 和 fileName 屬性所使用的個別變數。

## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料集，以及原則) 適用於所有類型的活動。 而活動的 **typeProperties** 區段中，可用的屬性會隨著每個活動類型而有所不同。

就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。 如果您要從內部部署的檔案系統移動資料，需將複製活動中的來源類型設定為 **FileSystemSource**。 同樣地，如果您要將資料移到內部部署的檔案系統，則需將複製活動中的接收器類型設定為 **FileSystemSink**。 本節提供 FileSystemSource 和 FileSystemSink 所支援的屬性清單。

**FileSystemSource** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

**FileSystemSink** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| copyBehavior |當來源為 BlobSource 或 FileSystem 時，定義複製行為。 |**PreserveHierarchy：** 保留目標資料夾中的檔案階層。 亦即，來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/>**FlattenHierarchy：** 來源資料夾的中所有檔案都會建立在目標資料夾的第一層中。 建立的目標檔案會具有自動產生的名稱。<br/><br/>**MergeFiles：** 將來源資料夾的所有檔案合併為一個檔案。 如果有指定檔案/Blob 名稱，合併檔案的名稱會是指定的名稱。 否則，就會是自動產生的檔案名稱。 |否 |

### <a name="recursive-and-copybehavior-examples"></a>遞迴和 copyBehavior 範例
本節說明 recursive 和 copyBehavior 屬性在不同組合的情況下，複製作業所產生的行為。

| recursive 值 | copyBehavior 值 | 產生的行為 |
| --- | --- | --- |
| true |preserveHierarchy |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以與來源相同的結構，建立目標資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 有自動產生的名稱 |
| true |mergeFiles |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File 5 的內容會合併成一個檔案，並有自動產生的檔案名稱。 |
| false |preserveHierarchy |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |
| false |flattenHierarchy |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |
| false |mergeFiles |具有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的內容會合併成一個檔案，並有自動產生的檔案名稱。<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4、File5 的 Subfolder1。 |

## <a name="supported-file-and-compression-formats"></a>支援的檔案和壓縮格式
請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)文章以了解詳細資訊。

## <a name="json-examples-for-copying-data-to-and-from-file-system"></a>從檔案系統來回複製資料的 JSON 範例
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 它們會示範如何將資料複製到內部部署檔案系統和 Azure Blob 儲存體，以及複製其中的資料。 您可以使用 Azure Data Factory 中的複製活動，把資料「直接」複製到[支援的來源和接收器](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文中所述的任何接收器。

### <a name="example-copy-data-from-an-on-premises-file-system-to-azure-blob-storage"></a>範例：將資料從內部部署檔案系統複製到 Azure Blob 儲存體
此範例示範如何將資料從內部部署檔案系統複製到 Azure Blob 儲存體。 範例有下列 Data Factory 實體：

* [OnPremisesFileServer](#linked-service-properties)類型的連結服務。
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [FileShare](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [FileSystemSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

下列範例每小時都會將時間序列資料從內部部署檔案系統複製到 Azure Blob 儲存體。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

首先，依照[利用資料管理閘道在內部部署來源和雲端之間移動資料](data-factory-move-data-between-onprem-and-cloud.md)的指示設定資料管理閘道。

**內部部署檔案伺服器連結服務：**

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia.<region>.corp.<company>.com",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

我們建議使用 **encryptedCredential** 屬性，而不要使用 **userid** 和 **password** 屬性。 如需此連結服務的詳細資訊，請參閱[檔案系統連結服務](#linked-service-properties)。

**Azure 儲存體連結服務：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**內部部署檔案系統輸入資料集：**

每小時從新的檔案挑選資料。 會根據配量的開始時間來決定 folderPath 和 fileName 屬性。  

`"external": "true"` 設定會通知 Data Factory：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "OnpremisesFileSystemInput",
  "properties": {
    "type": " FileShare",
    "linkedServiceName": " OnPremisesFileServerLinkedService ",
    "typeProperties": {
      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
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
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Azure Blob 儲存體資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**具有檔案系統來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **FileSystemSource**，而 **sink** 類型設為 **BlobSink**。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-06-01T18:00:00",
    "end":"2015-06-01T19:00:00",
    "description":"Pipeline for copy activity",
    "activities":[  
      {
        "name": "OnpremisesFileSystemtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "OnpremisesFileSystemInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
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
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
     ]
   }
}
```

### <a name="example-copy-data-from-azure-sql-database-to-an-on-premises-file-system"></a>範例：將資料從 Azure SQL Database 複製到內部部署檔案系統
下列範例顯示︰

* [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties) 類型的已連結服務。
* [OnPremisesFileServer](#linked-service-properties)類型的連結服務。
* [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties) 類型的輸入資料集。
* [FileShare](#dataset-properties) 類型的輸出資料集。
* 具有使用 [SqlSource](data-factory-azure-sql-connector.md##copy-activity-properties) 和 [FileSystemSink](#copy-activity-properties) 之複製活動的管線。

此範例會每小時將時間序列資料從 Azure SQL 資料表複製到內部部署檔案系統。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL Database 已連結的服務：**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```

**內部部署檔案伺服器連結服務：**

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia.<region>.corp.<company>.com",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

我們建議使用 **encryptedCredential** 屬性，而不要使用 **userid** 和 **password** 屬性。 如需此連結服務的詳細資訊，請參閱[檔案系統連結服務](#linked-service-properties)。

**Azure SQL 輸入資料集：**

此範例假設您已在 SQL Azure 中建立 "MyTable" 資料表，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

``“external”: ”true”`` 設定會通知 Data Factory：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

```JSON
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**內部部署檔案系統輸出資料集：**

資料會每小時複製到新的檔案。 會根據配量的開始時間來決定 Blob 的 folderPath 和 fileName。

```JSON
{
  "name": "OnpremisesFileSystemOutput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": " OnPremisesFileServerLinkedService ",
    "typeProperties": {
      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
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
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**具有 SQL 來源和檔案系統接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **FileSystemSink**。 針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-06-01T18:00:00",
    "end":"2015-06-01T20:00:00",
    "description":"pipeline for copy activity",
    "activities":[  
      {
        "name": "AzureSQLtoOnPremisesFile",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSQLInput"
          }
        ],
        "outputs": [
          {
            "name": "OnpremisesFileSystemOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "FileSystemSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 3,
          "timeout": "01:00:00"
        }
      }
     ]
   }
}
```


您也可以在複製活動定義中，將來自來源資料集的資料行與來自接收資料集的資料行對應。 如需詳細資料，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>效能和微調
 若要了解 Azure Data Factory 中影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)。
