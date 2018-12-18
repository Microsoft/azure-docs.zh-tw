---
title: 在 Azure Blob 儲存體來回複製資料 | Microsoft Docs
description: 了解如何在 Azure Data Factory 複製 Blob 資料。 使用我們的範例：如何在 Azure Blob 儲存體和 Azure SQL Database 將資料複製進來和複製出去。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: bec8160f-5e07-47e4-8ee1-ebb14cfb805d
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 2efc20d5a2248fed69f38880a9e75a6ccb2403dd
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42144257"
---
# <a name="copy-data-to-or-from-azure-blob-storage-using-azure-data-factory"></a>使用 Azure Data Factory 在 Azure Blob 儲存體來回複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-azure-blob-connector.md)
> * [第 2 版 (目前的版本)](../connector-azure-blob-storage.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的 Azure Blob 儲存體連接器](../connector-azure-blob-storage.md)。


本文說明如何使用 Azure Data Factory 中的「複製活動」，在 Azure Blob 儲存體來回複製資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文提供使用複製活動來移動資料的一般概觀。

## <a name="overview"></a>概觀
您可以將資料從任何支援的來源資料存放區複製到「Azure Blob 儲存體」，或從「Azure Blob 儲存體」複製到任何支援的接收資料存放區。 下表提供複製活動所支援作為來源或接收器的資料存放區清單。 例如，您可以將資料**從** SQL Server 資料庫或 Azure SQL Database 移**到** Azure Blob 儲存體。 而且，您可以將資料「從」 Azure Blob 儲存體複製「到」 Azure SQL 資料倉儲或 Azure Cosmos DB 集合。 

## <a name="supported-scenarios"></a>支援的案例
您可以將資料從 Azure Blob 儲存體儲存到下列資料存放區：

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製到 Azure Blob 儲存體：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]
 
> [!IMPORTANT]
> 複製活動支援在一般用途的 Azure 儲存體帳戶以及經常性存取/非經常性存取 Blob 儲存體來回複製資料。 此活動支援從區塊、附加或分頁 Blob 讀取，但只支援寫入至區塊 Blob。 不支援使用「Azure 進階儲存體」作為接收器，因為它是以分頁 Blob 為後盾。
> 
> 資料成功複製至目的地後，複製活動不會從來源刪除資料。 如果您需要在成功複製後刪除來源資料，請建立[自訂活動](data-factory-use-custom-activities.md)來刪除資料，然後在管道中使用該活動。 如需範例，請參閱 [刪除 GitHub 上的 Blob 或資料夾範例 (英文)](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/DeleteBlobFileFolderCustomActivity)。 

## <a name="get-started"></a>開始使用
您可以建立內含複製活動的管線，使用不同的工具/API 將資料移進/移出 Azure Blob 儲存體。

若要建立管線，最簡單的方式就是使用**複製精靈**。 這篇文章中有建立管線的[逐步解說](#walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage)，可將資料從 Azure Blob 儲存體位置複製到另一個 Azure Blob 儲存體位置。 如需建立管線以將資料從 Azure Blob 儲存體複製到 Azure SQL Database 的教學課程，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫，您會建立兩個連結服務，將 Azure 儲存體帳戶和 Azure SQL 資料庫連結至資料處理站。 有關 Azure Blob 儲存體專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。 
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您會建立資料集來指定 blob 容器和包含輸入資料的資料夾。 同時建立另一個資料集來指定 Azure SQL 資料庫中的 SQL 資料表，以保存從 Blob 儲存體複製的資料。 如需 Azure Blob 儲存體專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 BlobSource 作為來源，以及使用 SqlSink 作為複製活動的接收器。 同樣地，如果您是從 Azure SQL Database 複製到 Azure Blob 儲存體，則在複製活動中使用 SqlSource 和 BlobSink。 針對 Azure Blob 儲存體專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，按一下上一節中資料存放區的連結。  

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到「Azure Blob 儲存體」(或從「Azure Blob 儲存體」複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-to-and-from-blob-storage  )一節。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Azure Blob 儲存體專屬的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性
您可以使用兩種類型的已連結服務，將「Azure 儲存體」連結到 Azure Data Factory。 它們是：**AzureStorage** 連結服務和 **AzureStorageSas** 連結服務。 Azure 儲存體連結服務可將 Azure 儲存體的全域存取權提供給 Data Factory。 而 Azure 儲存體 SAS (共用存取簽章) 連結服務會將 Azure 儲存體的受限制/時間繫結存取權提供給 Data Factory。 這兩個連結服務之間沒有其他差異。 選擇符合您需求的連結服務。 以下章節提供這兩個連結服務的詳細資料。

[!INCLUDE [data-factory-azure-storage-linked-services](../../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="dataset-properties"></a>資料集屬性
若要指定資料集以代表「Azure Blob 儲存體」中的輸入或輸出資料，您需將資料集的類型屬性設定成：**AzureBlob**。 請將資料集的 **linkedServiceName** 屬性設定成「Azure 儲存體」或「Azure 儲存體 SAS」已連結服務的名稱。  資料集的類型屬性會指定 Blob 儲存體中的「Blob 容器」和「資料夾」。

如需定義資料集的 JSON 區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

Data Factory 支援使用下列符合 CLS 規範的 .NET 型類型值，在 “structure” 中針對在讀取時驗證結構描述 (schema-on-read) 的資料來源 (例如 Azure Blob) 提供類型資訊：Int16、Int32、Int64、Single、Double、Decimal、Byte[]、Bool、String、Guid、Datetime、Datetimeoffset、Timespan。 Data Factory 會在將資料從來源資料存放區移到接收資料存放區時，自動執行類型轉換。

每個資料集類型的 **TypeProperties** 區段都不同，可提供資料存放區中資料的位置、格式等相關資訊。 **AzureBlob** 類型資料集的 typeProperties 區段具有下列屬性：

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |Blob 儲存體中容器和資料夾的路徑。 範例：myblobcontainer\myblobfolder\ |是 |
| fileName |Blob 的名稱。 fileName 是選擇性的，而且區分大小寫。<br/><br/>如果您指定檔案名稱，活動 (包括複製) 適用於特定的 Blob。<br/><br/>如果您未指定 fileName，複製會包含 folderPath 中的所有 Blob 以做為輸入資料集。<br/><br/>當沒有為輸出資料集指定 fileName，且沒有為活動接收器指定 preserveHierarchy 時，所產生檔案的名稱會採用此格式：Data.<Guid>.txt (例如 Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| partitionedBy |partitionedBy 是選擇性的屬性。 您可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 如需詳細資訊和範例，請參閱 [使用 partitionedBy 屬性](#using-partitionedBy-property) 一節。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

### <a name="using-partitionedby-property"></a>使用 partitionedBy 屬性
如上一節所述，您可以使用 **partitionedBy** 屬性、[Data Factory 函式及系統變數](data-factory-functions-variables.md)，來指定時間序列資料的動態 folderPath 和檔案名稱。

如需時間序列資料集、排程和配量的詳細資訊，請參閱[建立資料集](data-factory-create-datasets.md)和[排程和執行](data-factory-scheduling-and-execution.md)等文章。

#### <a name="sample-1"></a>範例 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

在此範例中，{Slice} 會取代成 Data Factory 系統變數 SliceStart 的值 (使用指定的格式 (YYYYMMDDHH))。 SliceStart 是指配量的開始時間。 每個配量的 folderPath 都不同。 例如：wikidatagateway/wikisampledataout/2014100103 或 wikidatagateway/wikisampledataout/2014100104

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
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料集，以及原則) 適用於所有類型的活動。 而活動的 **typeProperties** 區段中，可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。 如果您要將資料從 Azure Blob 儲存體移出，請將複製活動中的來源類型設定為 **BlobSource**。 同樣地，如果您要將資料移入 Azure Blob 儲存體，請將複製活動中的接收類型設定為 **BlobSink**。 本節提供 BlobSource 和 BlobSink 支援的屬性清單。

**BlobSource** 在 **typeProperties** 區段中支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True (預設值)、False |否 |

**BlobSink** 在 **typeProperties** 區段中支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| copyBehavior |當來源為 BlobSource 或 FileSystem 時，定義複製行為。 |<b>PreserveHierarchy</b>：保留目標資料夾中的檔案階層。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/><b>FlattenHierarchy</b>：來自來源資料夾的所有檔案都在目標資料夾的第一層中。 目標檔案會有自動產生的名稱。 <br/><br/><b>MergeFiles</b>：將來源資料夾的所有檔案合併為一個檔案。 如果已指定檔案/Blob 名稱，合併檔案名稱會是指定的名稱；否則，就會是自動產生的檔案名稱。 |否 |

**BlobSource** 也支援這兩個屬性以提供回溯相容性。

* **treatEmptyAsNull**：指定是否將 null 或空字串視為 null 值。
* **skipHeaderLineCount** - 指定需略過多少行。 僅適用於輸入資料集使用 TextFormat 時。

同樣地， **BlobSink** 支援下列屬性以提供回溯相容性。

* **blobWriterAddHeader**︰指定是否要在寫入至輸出資料集時新增資料行定義的標頭。

資料集現在支援下列會實作相同功能的屬性︰**treatEmptyAsNull**、**skipLineCount**、**firstRowAsHeader**。

下表提供有關使用新的資料集屬性來取代這些 Blob 來源/接收屬性的指引。

| 複製活動屬性 | 資料集屬性 |
|:--- |:--- |
| BlobSource 上的 skipHeaderLineCount |skipLineCount 和 firstRowAsHeader。 會先略過行，然後讀取第一個資料列做為標頭。 |
| BlobSource 上的 treatEmptyAsNull |輸入資料集上的 treatEmptyAsNull |
| BlobSink 上的 blobWriterAddHeader |輸出資料集上的 firstRowAsHeader |

如需這些屬性的詳細資訊，請參閱 [指定 TextFormat](data-factory-supported-file-and-compression-formats.md#text-format) 一節。    

### <a name="recursive-and-copybehavior-examples"></a>遞迴和 copyBehavior 範例
本節說明遞迴和 copyBehavior 值在不同組合的情況下，複製作業所產生的行為。

| 遞迴 | copyBehavior | 產生的行為 |
| --- | --- | --- |
| true |preserveHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以與來源相同的結構，建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 有自動產生的名稱 |
| true |mergeFiles |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File 5 的內容會合併成一個檔案，並有自動產生的檔案名稱 |
| false |preserveHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |
| false |flattenHierarchy |對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/><br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |
| false |mergeFiles |對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的內容會合併成一個檔案，並有自動產生的檔案名稱。 File1 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |

## <a name="walkthrough-use-copy-wizard-to-copy-data-tofrom-blob-storage"></a>逐步解說︰使用複製精靈將資料複製到 Blob 儲存體或從此儲存體複製資料
讓我們看看如何將資料快速複製到 Azure Blob 儲存體或從此儲存體複製資料。 在這個逐步解說中，來源和目的地資料存放區的類型都是：Azure Blob 儲存體。 此逐步解說中的管線會將資料從一個資料夾複製到相同 Blob 容器中的另一個資料夾。 此逐步解說刻意設計得很簡單，為的是示範使用「Blob 儲存體」作為來源或接收器時的設定或屬性。 

### <a name="prerequisites"></a>必要條件
1. 建立一個一般用途的「Azure 儲存體帳戶」(如果您還沒有此帳戶)。 在這個逐步解說中，您將使用 Blob 儲存體同時作為「來源」和「目的地」資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md) 一文以取得建立步驟。
2. 在儲存體帳戶中建立一個名為 **adfblobconnector** 的 Blob 容器。 
4. 在 **adfblobconnector** 容器中建立一個名為 **input** 的資料夾。
5. 以下列內容建立一個名為 **emp.txt** 的檔案，然後使用 [Azure 儲存體總管](https://azurestorageexplorer.codeplex.com/)之類的工具將它上傳到 [input] 資料夾
    ```json
    John, Doe
    Jane, Doe
    ```
### <a name="create-the-data-factory"></a>建立 Data Factory
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左上角的 [建立資源]，按一下 [智慧 + 分析]，然後按一下 [Data Factory]。
3. 在 [新增 Data Factory] 窗格中：   
    1. 針對 [名稱]，輸入 **ADFBlobConnectorDF**。 Azure Data Factory 的名稱在全域必須是唯一的。 如果您收到錯誤：`*Data factory name “ADFBlobConnectorDF” is not available`，請變更 Data Factory 的名稱 (例如 yournameADFBlobConnectorDF)，然後嘗試重新建立。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。
    2. 選取您的 Azure **訂用帳戶**。
    3. 針對「資源群組」，選取 [使用現有的] 來選取現有的資源群組 (或) 選取 [建立新項目] 來輸入資源群組的名稱。
    4. 選取 Data Factory 的 [位置]。
    5. 選取刀鋒視窗底部的 [釘選到儀表板] 核取方塊。
    6. 按一下頁面底部的 [新增] 。
3. 建立完成之後，您會看到 [Data Factory] 刀鋒視窗，如下圖所示：![Data Factory 首頁](./media/data-factory-azure-blob-connector/data-factory-home-page.png)

### <a name="copy-wizard"></a>複製精靈
1. 在 Data Factory 首頁上，按一下 [資料複製] 圖格，以在個別索引標籤中啟動 [複製資料精靈]。    
    
    > [!NOTE]
    >    如果您看到網頁瀏覽器停留在「授權中...」，請停用/取消核取 [封鎖第三方 Cookie 和站台資料] 設定 (或) 將它保持啟用並為 **login.microsoftonline.com** 建立例外狀況，然後再次嘗試啟動精靈。
2. 在 [屬性]  頁面︰
    1. 針對 [工作名稱]，輸入 **CopyPipeline**。 工作名稱是您 Data Factory 中管線的名稱。
    2. 輸入工作的「描述」(選擇性)。
    3. 針對 [工作頻率或工作排程]，保留 [依排程定期執行] 選項。 如果您想要僅執行此工作一次，而不要依排程重複執行，請選取 [立即執行一次]。 如果您選取 [立即執行一次] 選項，系統就會建立一個[單次管線](data-factory-create-pipelines.md#onetime-pipeline)。 
    4. 保留 [週期性模式] 的設定。 此工作會在您於下一個步驟中指定的開始和結束時間之間每天執行。
    5. 將 [開始日期時間] 變更為 [2017 年 4 月 21 日]。 
    6. 將 [結束日期時間] 變更為 [2017 年 4 月 25 日]。 您可以輸入日期，而不瀏覽行事曆。     
    8. 按 [下一步] 。
      ![複製工具 - 屬性頁面](./media/data-factory-azure-blob-connector/copy-tool-properties-page.png) 
3. 在 [來源資料存放區] 頁面上，按一下 [Azure Blob 儲存體] 圖格。 您可以使用此頁面來指定複製工作的來源資料存放區。 您可以使用現有的資料存放區連結服務或指定新的資料存放區。 若要使用現有的已連結服務，您需選取 [從現有的連結服務]，然後選取正確的已連結服務。 
    ![複製工具 - 來源資料存放區頁面](./media/data-factory-azure-blob-connector/copy-tool-source-data-store-page.png)
4. 在 [指定 Azure Blob 儲存體帳戶]  頁面︰
   1. 針對 [連線名稱]，保留自動產生的名稱。 此連線名稱是「Azure 儲存體」類型之已連結服務的名稱。 
   2. 確認已針對 [帳戶選取方法] 選取 [從 Azure 訂用帳戶] 選項。
   3. 針對 [Azure 訂用帳戶]，選取您的 Azure 訂用帳戶或保留 [全選]。   
   4. 從所選訂用帳戶中可用的 Azure 儲存體帳戶清單中，選取 [Azure 儲存體帳戶]。 您也可以選擇手動輸入儲存體帳戶設定，方法是針對 [帳戶選取方法] 選取 [手動輸入] 選項。
   5. 按 [下一步] 。 
      ![複製工具 - 指定 Azure Blob 儲存體帳戶](./media/data-factory-azure-blob-connector/copy-tool-specify-azure-blob-storage-account.png)
5. 在 [選擇輸入檔案或資料夾]  頁面︰
   1. 按兩下 [adfblobcontainer]。
   2. 選取 [input]，然後按一下 [選擇]。 在本逐步解說中，您會選取 [input] 資料夾。 您也可以改為選取資料夾中的 emp.txt 檔案。 
      ![複製工具 - 選擇輸入檔案或資料夾](./media/data-factory-azure-blob-connector/copy-tool-choose-input-file-or-folder.png)
6. 在 [選擇輸入檔案或資料夾] 頁面上︰
    1. 確定 [檔案或資料夾] 已設定為 [adfblobconnector/input]。 如果檔案位於子資料夾 (例如 2017/04/01、2017/04/02 等)，請針對檔案或資料夾輸入 adfblobconnector/input/{year}/{month}/{day}。 當您按 TAB 從文字方塊中移出時，會看到三個可供選取年 (yyyy)、月 (MM) 和日 (dd) 格式的下拉式清單。 
    2. 請勿設定 [以遞迴方式複製檔案]。 選取此選項可以用遞迴方式周遊資料夾來尋找要複製到目的地的檔案。 
    3. 請勿設定 [二進位複製] 選項。 選取此選項可以執行將來源檔案複製到目的地的二進位複製。 請勿為此逐步解說選取這個選項，以便您可以在後續頁面中看到更多選項。 
    4. 確認 [壓縮類型] 已設定為 [無]。 如果您的來源檔案是以其中一種支援的格式壓縮的，則請為此選項選取一個值。 
    5. 按 [下一步] 。
    ![複製工具 - 選擇輸入檔案或資料夾](./media/data-factory-azure-blob-connector/chose-input-file-folder.png) 
7. 在 [檔案格式設定] 頁面上，您會看到分隔符號以及精靈藉由剖析檔案自動偵測到的結構描述。 
    1. 確認下列選項：a. [檔案格式] 已設定為 [文字格式]。 您可以在下拉式清單中看到所有支援的格式。 例如：JSON、Avro、ORC、Parquet。
        b. [資料行分隔符號] 已設定為 [`Comma (,)`]。 您可以在下拉式清單中看到 Data Factory 支援的其他資料行分隔符號。 您也可以指定自訂的分隔符號。
        c. [資料列分隔符號] 已設定為 [`Carriage Return + Line feed (\r\n)`]。 您可以在下拉式清單中看到 Data Factory 支援的其他資料列分隔符號。 您也可以指定自訂的分隔符號。
        d. [略過行數] 已設定為 [0]。 如果您希望略過檔案開頭的幾行，則請在這裡輸入數字。
        e.  未設定 [第一個資料列包含資料行名稱]。 如果來源檔案的第一個資料列包含資料行名稱，則請選取此選項。
        f. 已設定 [將空白資料行值視為 Null] 選項。
    2. 展開 [進階設定] 以查看可用的進階選項。
    3. 在頁面底部，查看來自 emp.txt 檔案之資料的 [預覽]。
    4. 按一下底部的 [結構描述] 索引標籤，以查看複製精靈藉由查看來源檔案中的資料所推斷出的結構描述。
    5. 檢閱分隔符號並預覽資料之後，請按 [下一步]。
    ![複製工具 - 檔案格式設定](./media/data-factory-azure-blob-connector/copy-tool-file-format-settings.png)  
8. 在 [目的地資料存放區] 頁面上，選取 [Azure Blob 儲存體]，然後按 [下一步]。 在這個逐步解說中，您將使用「Azure Blob 儲存體」同時作為來源和目的地資料存放區。    
    ![複製精靈 - 選取目的地資料存放區](media/data-factory-azure-blob-connector/select-destination-data-store.png)
9. 在 [指定 Azure Blob 儲存體帳戶] 頁面上︰
   1. 針對 [連線名稱] 欄位，輸入 **AzureStorageLinkedService**。
   2. 確認已針對 [帳戶選取方法] 選取 [從 Azure 訂用帳戶] 選項。
   3. 選取您的 Azure **訂用帳戶**。  
   4. 選取您的 Azure 儲存體帳戶。 
   5. 按 [下一步] 。     
10. 在 [選擇輸出檔案或資料夾] 頁面上︰ 
    6. 將 [資料夾路徑] 指定為 **adfblobconnector/output/{year}/{month}/{day}**。 輸入 **TAB**。
    7. 針對 [年]，選取 [yyyy]。
    8. 針對 [月]，確定它已設定為 [MM]。
    9. 針對 [日]，確定它已設定為 [dd]。
    10. 確認 [壓縮類型] 已設定為 [無]。
    11. 確認 [複製行為] 已設定為 [合併檔案]。 如果已經有同名的輸出檔案存在，則新內容會新增到該相同檔案的結尾。
    12. 按 [下一步] 。
    ![複製工具 - 選擇輸出檔案或資料夾](media/data-factory-azure-blob-connector/choose-the-output-file-or-folder.png)
11. 在 [檔案格式設定] 頁面上，檢閱設定，然後按 [下一步]。 這裡的其中一額外選項是為輸出檔案新增標頭。 如果您選取該選項，就會新增標頭資料列，其中會含有來自來源結構描述的資料行名稱。 您可以在檢視來源的結構描述時，重新命名預設的資料行名稱。 例如，您可以將第一個資料行變更為「名字」，而將第二個資料行變更為「姓氏」。 接著，系統就會產生含有以這些名稱作為資料行名稱之標頭的輸出檔案。 
    ![複製工具 - 目的地的檔案格式設定](media/data-factory-azure-blob-connector/file-format-destination.png)
12. 在 [效能設定] 頁面上，確認 [雲端單位] 和 [平行複製] 已設定為 [自動]，然後按 [下一步]。 如需有關這些設定的詳細資料，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md#parallel-copy)。
    ![複製工具 - 效能設定](media/data-factory-azure-blob-connector/copy-performance-settings.png) 
14. 在 [摘要] 頁面上，檢閱所有設定 (工作屬性、來源和目的地的設定，以及複製設定)，然後按 [下一步]。
    ![複製工具 - 摘要頁面](media/data-factory-azure-blob-connector/copy-tool-summary-page.png)
15. 在 [摘要] 頁面中檢閱資訊，然後按一下 [完成]。 此精靈會在 Data Factory (從您啟動複製精靈的位置) 中建立兩個連結服務、兩個資料集 (輸入和輸出)，以及一個管線。
    ![複製工具 - 部署頁面](media/data-factory-azure-blob-connector/copy-tool-deployment-page.png)

### <a name="monitor-the-pipeline-copy-task"></a>監視管線 (複製工作)

1. 在 [部署] 頁面上，按一下 [`Click here to monitor copy pipeline`] 連結。 
2. 您應該會在個別的索引標籤中看到 [監視及管理應用程式]。![監視及管理應用程式](media/data-factory-azure-blob-connector/monitor-manage-app.png)
3. 將頂端的 [開始時間] 變更為 [`04/19/2017`]，將 [結束時間] 變更為 [`04/27/2017`]，然後按一下 [套用]。 
4. 您應該會在 [活動時段] 清單中看到五個活動時段。 [時段開始] 時間應該涵蓋從管線開始到管線結束時間的所有日子。 
5. 按一下 [活動時段] 清單的 [重新整理] 按鈕幾次，直到您看到所有活動時段的狀態都設定為 [就緒] 為止。 
6. 現在，確認是否已在 adfblobconnector 容器的輸出資料夾中產生輸出檔案。 您應該會在輸出資料夾中看到以下資料夾結構： 
    ```
    2017/04/21
    2017/04/22
    2017/04/23
    2017/04/24
    2017/04/25    
    ```
如需有關監視及管理 Data Factory 的詳細資訊，請參閱[監視和管理 Data Factory 管線](data-factory-monitor-manage-app.md)一文。 
 
### <a name="data-factory-entities"></a>Data Factory 實體
現在，切換回含有 Data Factory 首頁的索引標籤。 請注意，您的 Data Factory 現在有兩個已連結的服務、兩個資料集，以及一條管線。 

![含有實體的 Data Factory 首頁](media/data-factory-azure-blob-connector/data-factory-home-page-with-numbers.png)

按一下 [編寫及部署] 以啟動「Data Factory 編輯器」。 

![Data Factory 編輯器](media/data-factory-azure-blob-connector/data-factory-editor.png)

您應該會在 Data Factory 中看到下列 Data Factory 實體： 

 - 兩個已連結的服務。 一個用於來源，另一個用於目的地。 在這個逐步解說中，兩個已連結的服務都參考相同的「Azure 儲存體」帳戶。 
 - 兩個資料集。 一個輸入資料集和一個輸出資料集。 在這個逐步解說中，兩者使用相同的 Blob 容器，但參考不同的資料夾 (輸入和輸出)。
 - 一條管線。 此管線包含一個複製活動，此活動會使用 Blob 來源和 Blob 接收器，將資料從 Azure Blob 位置複製到另一個 Azure Blob 位置。 

下列各節會提供關於這些實體的詳細資訊。 

#### <a name="linked-services"></a>連結的服務
您應該會看到兩個已連結的服務。 一個用於來源，另一個用於目的地。 在這個逐步解說中，除了名稱之外，兩個定義看起來相同。 已連結服務的 **type** 是設定為 **AzureStorage**。 已連結服務的最重要屬性是 **connectionString**，Data Factory 會使用此屬性在執行階段連接到您的「Azure 儲存體」帳戶。 請在定義中忽略 hubName 屬性。 

##### <a name="source-blob-storage-linked-service"></a>來源 Blob 儲存體連結服務
```json
{
    "name": "Source-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

##### <a name="destination-blob-storage-linked-service"></a>目的地 Blob 儲存體連結服務

```json
{
    "name": "Destination-BlobStorage-z4y",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=**********"
        }
    }
}
```

如需有關「Azure 儲存體」連結服務的詳細資訊，請參閱[連結服務屬性](#linked-service-properties)一節。 

#### <a name="datasets"></a>資料集
有兩個資料集：一個輸入資料集和一個輸出資料集。 兩者的資料集類型都設定為 **AzureBlob**。 

輸入資料集會指向 **adfblobconnector** 容器的 [input] 資料夾。 此資料集的 **external** 屬性是設定為 **true**，因為產生資料的管線不是含有以此資料集作為輸入之複製活動的管線。 

輸出資料集會指向相同 Blob 容器的 [output] 資料夾。 輸出資料集也使用 **SliceStart** 系統變數的年、月和日來動態評估輸出檔案的路徑。 如需 Data Factory 所支援的函式與系統變數清單，請參閱 [Data Factory 函式與系統變數](data-factory-functions-variables.md) 。 **external** 屬性是設定為 **false**，因為此資料集是由管線所產生。 

如需有關 Azure Blob 資料集所支援屬性的詳細資訊，請參閱[資料集屬性](#dataset-properties)一節。

##### <a name="input-dataset"></a>輸入資料集

```json
{
    "name": "InputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Source-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/input/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

##### <a name="output-dataset"></a>輸出資料集

```json
{
    "name": "OutputDataset-z4y",
    "properties": {
        "structure": [
            { "name": "Prop_0", "type": "String" },
            { "name": "Prop_1", "type": "String" }
        ],
        "type": "AzureBlob",
        "linkedServiceName": "Destination-BlobStorage-z4y",
        "typeProperties": {
            "folderPath": "adfblobconnector/output/{year}/{month}/{day}",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            },
            "partitionedBy": [
                { "name": "year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
                { "name": "month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
                { "name": "day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } }
            ]
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        },
        "external": false,
        "policy": {}
    }
}
```

#### <a name="pipeline"></a>管線
此管線只有一個活動。 此活動的 **type** 是設定為 **Copy**。  在此活動類型屬性中，有兩個區段，一個用於來源，另一個用於接收器。 來源類型是設定為 **BlobSource**，因為此活動會從 Blob 儲存體複製資料。 接收器類型是設定為 **BlobSink**，因為此活動會將資料複製到 Blob 儲存體。 此複製活動會以 InputDataset-z4y 作為輸入，並以 OutputDataset-z4y 作為輸出。 

如需有關 BlobSource 和 BlobSink 所支援屬性的詳細資訊，請參閱[複製活動屬性](#copy-activity-properties)一節。 

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "MergeFiles",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset-z4y"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset-z4y"
                    }
                ],
                "policy": {
                    "timeout": "1.00:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 3,
                    "longRetry": 0,
                    "longRetryInterval": "00:00:00"
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "Activity-0-Blob path_ adfblobconnector_input_->OutputDataset-z4y"
            }
        ],
        "start": "2017-04-21T22:34:00Z",
        "end": "2017-04-25T05:00:00Z",
        "isPaused": false,
        "pipelineMode": "Scheduled"
    }
}
```

## <a name="json-examples-for-copying-data-to-and-from-blob-storage"></a>從 Blob 儲存體來回複製資料的 JSON 範例  
以下範例提供可用來使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 建立管線的範例 JSON 定義。 它們會示範如何將資料複製到 Azure Blob 儲存體和 Azure SQL Database，以及複製其中的資料。 不過，您可以在 Azure Data Factory 中使用複製活動，從任何來源 **直接** 將資料複製到 [這裡](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 所說的任何接收器。

### <a name="json-example-copy-data-from-blob-storage-to-sql-database"></a>JSON 範例：將資料從 Blob 儲存體複製到 SQL Database
下列範例顯示︰

1. [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties)類型的連結服務。
2. [AzureStorage](#linked-service-properties)類型的連結服務。
3. [AzureBlob](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [BlobSource](#copy-activity-properties) 和 [SqlSink](data-factory-azure-sql-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料從 Azure Blob 複製到 Azure SQL 資料表。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 連結服務：**

```json
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
**Azure 儲存體連結服務：**

```json
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
Azure Data Factory 支援兩種類型的 Azure 儲存體連結服務：**AzureStorage** 和 **AzureStorageSas**。 對於前者指定連接字串，包含帳戶金鑰，對於後者指定共用存取簽章 (SAS) Uri。 請參閱 [連結服務](#linked-service-properties) 章節以取得詳細資料。  

**Azure Blob 輸入資料集：**

每小時從新的 Blob 挑選資料 (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑使用開始時間的年、月、日部分，而檔案名稱使用開始時間的小時部分。 “external”: “true” 設定會通知 Data Factory：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
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
**Azure SQL 輸出資料集：**

此範例會將資料複製到 Azure SQL Database 中名為 "MyTable" 的資料表。 請在Azure SQL Database 中建立此資料表，其資料行的數目如您預期 Blob CSV 檔案要包含的數目。 此資料表會每小時加入新的資料列。

```json
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**具有 Blob 來源和 SQL 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlSink**。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "AzureBlobtoSQL",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
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
### <a name="json-example-copy-data-from-azure-sql-to-azure-blob"></a>JSON 範例：將資料從 Azure SQL 複製到 Azure Blob
下列範例顯示︰

1. [AzureSqlDatabase](data-factory-azure-sql-connector.md#linked-service-properties)類型的連結服務。
2. [AzureStorage](#linked-service-properties)類型的連結服務。
3. [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [SqlSource](data-factory-azure-sql-connector.md#copy-activity-properties) 和 [BlobSink](#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例會每小時將時間序列資料從 Azure SQL 資料表複製到 Azure Blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

**Azure SQL 連結服務：**

```json
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
**Azure 儲存體連結服務：**

```json
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
Azure Data Factory 支援兩種類型的 Azure 儲存體連結服務：**AzureStorage** 和 **AzureStorageSas**。 對於前者指定連接字串，包含帳戶金鑰，對於後者指定共用存取簽章 (SAS) Uri。 請參閱 [連結服務](#linked-service-properties) 章節以取得詳細資料。  

**Azure SQL 輸入資料集：**

此範例假設您已在 SQL Azure 中建立資料表 "MyTable"，其中包含時間序列資料的資料行 (名稱為 "timestampcolumn")。

設定 “external”: ”true” 會通知 Data Factory 服務：這是 Data Factory 外部的資料表而且不是由 Data Factory 中的活動所產生。

```json
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

**Azure Blob 輸出資料集：**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 資料夾路徑會使用開始時間的年、月、日和小時部分。

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
      "partitionedBy": [
        {
          "name": "Year",
          "value": { "type": "DateTime",  "date": "SliceStart", "format": "yyyy" } },
        { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
        { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
        { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" } }
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

**具有 SQL 來源和 Blob 接收器的管線中複製活動：**

此管線包含複製活動，該活動已設定為使用輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **SqlSource**，而 **sink** 類型設為 **BlobSink**。 針對 **SqlReaderQuery** 屬性指定的 SQL 查詢會選取過去一小時內要複製的資料。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
              {
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureSQLInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureBlobOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
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
