---
title: 使用複製活動來移動資料 | Microsoft Docs
description: 了解 Data Factory 管線中的資料移動︰雲端存放區之間和內部部署與雲端之間的資料移轉。 使用「複製活動」。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 67543a20-b7d5-4d19-8b5e-af4c1fd7bc75
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/05/2017
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 6b13c70d86af195e50190083aa562811236cdd4b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38299855"
---
# <a name="move-data-by-using-copy-activity"></a>使用複製活動來移動資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-data-movement-activities.md)
> * [第 2 版 (目前的版本)](../copy-activity-overview.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的複製活動](../copy-activity-overview.md)。

## <a name="overview"></a>概觀
在 Azure Data Factory 中，您可以使用「複製活動」在內部部署與雲端資料存放區之間複製資料。 複製資料之後，可以將它進一步轉換及進行分析。 您也可以使用「複製活動」來發佈商業智慧 (BI) 及應用程式使用情況的轉換和分析結果。

![複製活動的角色](media/data-factory-data-movement-activities/copy-activity.png)

「複製活動」是由安全、可靠、可調整且 [全域可用的服務](#global)提供技術支援。 本文提供關於在 Data Factory 和複製活動中移動資料的詳細資訊。

首先，讓我們看看在兩個雲端資料存放區之間，以及在內部部署資料存放區與雲端資料存放區之間，如何進行資料移轉。

> [!NOTE]
> 若要了解活動的大致情況，請參閱 [了解管線和活動](data-factory-create-pipelines.md)。
>
>

### <a name="copy-data-between-two-cloud-data-stores"></a>在兩個雲端資料存放區之間複製資料
當來源和接收資料存放區都在雲端時，「複製活動」就會經歷下列階段，將資料從來源複製到接收資料存放區。 為「複製活動」提供技術支援的雲端服務會：

1. 從來源資料存放區讀取資料。
2. 執行序列化/還原序列化、壓縮/解壓縮、資料行對應及類型轉換。 它會根據輸入資料集、輸出資料集及「複製活動」的組態執行這些作業。
3. 將資料寫入目的地資料存放區。

此服務會自動選擇最佳區域來執行資料移動。 此區域通常是最靠近接收資料存放區的區域。

![從雲端複製到雲端](./media/data-factory-data-movement-activities/cloud-to-cloud.png)

### <a name="copy-data-between-an-on-premises-data-store-and-a-cloud-data-store"></a>在內部部署資料存放區和雲端資料存放區之間複製資料
若要安全地在內部部署資料存放區與雲端資料存放區之間移動資料，請在內部部署機器上安裝「資料管理閘道」。 「資料管理閘道」是一個能夠啟用混合式資料移動及處理的代理程式。 您可以將它安裝在與資料存放區本身相同的機器上，或是安裝在可存取該資料存放區的另一部機器上。

在此案例中，「資料管理閘道」會執行序列化/還原序列化、壓縮/解壓縮、資料行對應及類型轉換。 資料不會流經 Azure Data Factory 服務。 取而代之的是，「資料管理閘道」會直接將資料寫入到目的地存放區。

![從內部部署複製到雲端](./media/data-factory-data-movement-activities/onprem-to-cloud.png)

如需簡介和逐步解說，請參閱 [在內部部署和雲端資料存放區之間移動資料](data-factory-move-data-between-onprem-and-cloud.md) 。 如需有關此代理程式的詳細資訊，請參閱 [資料管理閘道](data-factory-data-management-gateway.md) 。

您也可以使用「資料管理閘道」，將資料移出/移入 Azure IaaS 虛擬機器 (VM) 上所裝載的支援資料存放區。 在此情況下，您可以將「資料管理閘道」安裝在與資料存放區本身相同的 VM 上，或是安裝在可存取該資料存放區的另一部 VM 上。

## <a name="supported-data-stores-and-formats"></a>支援的資料存放區和格式
Data Factory 中的複製活動會將資料從來源資料存放區複製到接收資料存放區。 Data Factory 支援下列資料存放區。 可將來自任何來源的資料寫入任何接收器。 按一下資料存放區，即可了解如何將資料複製到該存放區，以及從該存放區複製資料。

> [!NOTE] 
> 如果您需要將資料移入/移出「複製活動」不支援的資料存放區，請在 Data Factory 中使用 **自訂活動** 搭配您自己的邏輯來複製/移動資料。 如需有關建立及使用自訂活動的詳細資料，請參閱 [在 Azure Data Factory 管線中使用自訂活動](data-factory-use-custom-activities.md)。

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> 具有 * 的資料存放區可以在內部部署環境或 Azure IaaS 上，並且需要您在內部部署/Azure IaaS 機器上安裝 [資料管理閘道](data-factory-data-management-gateway.md) 。

### <a name="supported-file-formats"></a>支援的檔案格式
您可以使用「複製活動」在兩個以檔案為基礎的資料存放區之間**依原樣複製檔案**，您可以在輸入和輸出資料集定義中略過[格式區段](data-factory-create-datasets.md)。 系統會有效率地複製資料，而不會進行任何序列化/還原序列化。

「複製活動」也會以指定的格式讀取和寫入檔案：**文字、JSON、Avro、ORC 和 Parquet**，並支援 **GZip、Deflate、BZip2 和 ZipDeflate** 壓縮轉碼器。 如需詳細資訊，請參閱[支援的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)。

例如，您可以執行下列複製活動：

* 複製內部部署 SQL Server 中的資料，然後以 ORC 格式寫入 Azure Data Lake Store。
* 從內部部署檔案系統複製文字 (CSV) 格式的檔案，然後以 Avro 格式寫入 Azure Blob 中。
* 從內部部署檔案系統複製壓縮檔，然後解壓縮到 Azure Data Lake Store。
* 從 Azure Blob 複製 GZip 壓縮文字 (CSV) 格式的資料，然後寫入 Azure SQL Database 中。

## <a name="global"></a>全域可用的資料移動
Azure Data Factory 只在美國西部、美國東部和北歐區域提供使用。 不過，支援複製活動的服務可在下列區域和地理位置全域提供使用。 全域可用的拓撲可確保進行有效率的資料移動，通常可避免發生跨區域躍點的情況。 如需了解某區域中是否有 Data Factory 和「資料移動」可供使用，請參閱 [依區域提供的服務](https://azure.microsoft.com/regions/#services) 。

### <a name="copy-data-between-cloud-data-stores"></a>在雲端資料存放區之間複製資料
當來源和接收資料存放區都位於雲端時，Data Factory 會使用區域中最接近相同地理位置之接收的服務部署來移動資料。 如需對應資訊，請參閱下表︰

| 目的地資料存放區的地理位置 | 目的地資料存放區的區域 | 用於資料移動的區域 |
|:--- |:--- |:--- |
| 美國 | 美國東部 | 美國東部 |
| &nbsp; | 美國東部 2 | 美國東部 2 |
| &nbsp; | 美國中部 | 美國中部 |
| &nbsp; | 美國中北部 | 美國中北部 |
| &nbsp; | 美國中南部 | 美國中南部 |
| &nbsp; | 美國中西部 | 美國中西部 |
| &nbsp; | 美國西部 | 美國西部 |
| &nbsp; | 美國西部 2 | 美國西部 2 |
| 加拿大 | 加拿大東部 | 加拿大中部 |
| &nbsp; | 加拿大中部 | 加拿大中部 |
| 巴西 | 巴西南部 | 巴西南部 |
| 歐洲 | 北歐 | 北歐 |
| &nbsp; | 西歐 | 西歐 |
| 英國 | 英國西部 | 英國南部 |
| &nbsp; | 英國南部 | 英國南部 |
| 亞太地區 | 東南亞 | 東南亞 |
| &nbsp; | 東亞 | 東南亞 |
| 澳大利亞 | 澳洲東部 | 澳洲東部 |
| &nbsp; | 澳大利亞東南部 | 澳大利亞東南部 |
| 印度 | 印度中部 | 印度中部 |
| &nbsp; | 印度西部 | 印度中部 |
| &nbsp; | 印度南部 | 印度中部 |
| 日本 | 日本東部 | 日本東部 |
| &nbsp; | 日本西部 | 日本東部 |
| 南韓 | 南韓中部 | 南韓中部 |
| &nbsp; | 南韓南部 | 南韓中部 |

或者，您可以明確指出要用來執行複製的 Data Factory 服務區域，方法是指定複製活動 `typeProperties` 底下的 `executionLocation`屬性。 這個屬性支援的值詳列於上述**用於資料移動的區域**資料行。 請注意，您的資料在複製期間會透過網路通過該區域。 例如，若要在南韓的 Azure 存放區之間複製，您可以將 `"executionLocation": "Japan East"` 指定為經過日本區域 (請參考[範例 JSON](#by-using-json-scripts))。

> [!NOTE]
> 如果目的地資料存放區的區域不在上述清單中，除非指定 `executionLocation`，否則「複製活動」預設將會失敗而不會搜查替代區域。 支援的區域清單將會隨著時間擴展。
>

### <a name="copy-data-between-an-on-premises-data-store-and-a-cloud-data-store"></a>在內部部署資料存放區和雲端資料存放區之間複製資料
在內部部署存放區 (或 Azure 虛擬機器/IaaS) 與雲端存放區之間複製資料時， [資料管理閘道](data-factory-data-management-gateway.md) 會在內部部署機器或虛擬機器上執行資料移動。 除非您使用 [分段複製](data-factory-copy-activity-performance.md#staged-copy) 功能，否則資料不會流經雲端服務。 在此情況下，資料會先流經預備環境 Azure Blob 儲存體，然後才寫入接收資料存放區。

## <a name="create-a-pipeline-with-copy-activity"></a>建立具有複製活動的管線
您可以使用幾個方式來建立具有「複製活動」的管線︰

### <a name="by-using-the-copy-wizard"></a>透過使用複製精靈
「Data Factory 複製精靈」可協助您建立具有「複製活動」的管線。 此管線可讓您在「不需要」為連結服務、資料集及管線「撰寫 JSON」  定義的情況下，將資料從支援的來源複製到目的地。 如需有關此精靈的詳細資料，請參閱 [Data Factory 複製精靈](data-factory-copy-wizard.md) 。  

### <a name="by-using-json-scripts"></a>透過使用 JSON 指令碼
您可以使用 Azure 入口網站中的「Data Factory 編輯器」、Visual Studio 或 Azure PowerShell 來建立管線的 JSON 定義 (透過使用「複製活動」)。 然後，您可以部署它以在 Data Factory 中建立管線。 如需含有逐步指示的教學課程，請參閱 [教學課程：在 Azure Data Factory 管線中使用複製活動](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) 。    

JSON 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。 活動的 `typeProperties` 區段中可用的屬性會因每個活動類型的不同而有所不同。

在複製活動中， `typeProperties` 區段會根據來源和接收器的類型而有所不同。 按一下 [支援來源和接收器](#supported-data-stores-and-formats) 一節中的來源/接收器，即可了解「複製活動」針對該資料存放區所支援的類型屬性。

以下是範例 JSON 定義︰

```json
{
  "name": "ADFTutorialPipeline",
  "properties": {
    "description": "Copy data from Azure blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy",
        "inputs": [
          {
            "name": "InputBlobTable"
          }
        ],
        "outputs": [
          {
            "name": "OutputSQLTable"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          },
          "executionLocation": "Japan East"          
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2016-07-12T00:00:00Z",
    "end": "2016-07-13T00:00:00Z"
  }
}
```
輸出資料集中定義的排程會決定活動的執行時間 (例如「每日」，即頻率為「日」，間隔為 **1**)。 活動會將資料從輸入資料集 (**source**) 複製到輸出資料集 (**sink**)。

您可以為「複製活動」指定多個輸入資料集。 系統會先使用它們來確認相依性，然後才執行活動。 不過，只有來自第一個資料集的資料會被複製到目的地資料集。 如需詳細資訊，請參閱 [排程和執行](data-factory-scheduling-and-execution.md)。  

## <a name="performance-and-tuning"></a>效能和微調
請參閱 [複製活動的效能及微調指南](data-factory-copy-activity-performance.md)，其中說明在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素。 它也列出在內部測試期間所觀察到的效能，並討論各種可將「複製活動」效能最佳化的方式。

## <a name="fault-tolerance"></a>容錯
根據預設，來源與接收之間出現不相容的資料時，複製活動會停止複製資料並傳回失敗；而您可以明確設定略過並記錄不相容的資料列，只複製那些相容的資料，使複製成功。 如需詳細資訊，請參閱[複製活動容錯](data-factory-copy-activity-fault-tolerance.md)。

## <a name="security-considerations"></a>安全性考量
請參閱[安全性考量](data-factory-data-movement-security-considerations.md)，說明 Azure Data Factory 中資料移動服務用來保護您資料的安全性基礎結構。

## <a name="scheduling-and-sequential-copy"></a>排程和循序複製
請參閱 [排程和執行](data-factory-scheduling-and-execution.md) ，以取得排程和執行在 Data Factory 中如何運作的詳細資訊。 您可以利用循序/排序的方式，逐一執行多個複製作業。 請參閱[循序複製](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)一節。

## <a name="type-conversions"></a>類型轉換
不同的資料存放區有不同的原生類型系統。 「複製活動」會藉由下列含有兩個步驟的方法，執行從來源類型轉換成接收類型的自動類型轉換：

1. 從原生來源類型轉換成 .NET 類型。
2. 從 .NET 類型轉換成原生接收類型。

資料存放區從原生類型系統到 .NET 類型的對應，位於個別的資料存放區文章中。 (按一下 [支援的資料存放區](#supported-data-stores) 表格中的特定連結)。 您可以在建立資料表時，使用這些對應來判斷適當的類型，以便讓「複製活動」能夠執行正確的轉換。

## <a name="next-steps"></a>後續步驟
* 若要深入了解複製活動，請參閱 [將資料從 Azure Blob 儲存體複製到 Azure SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
* 若要了解如何將資料從內部部署資料存放區移到雲端資料存放區，請參閱 [將資料從內部部署資料存放區移到雲端資料存放區](data-factory-move-data-between-onprem-and-cloud.md)。
