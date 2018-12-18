---
title: 要求單位和估計輸送量 - Azure Cosmos DB | Microsoft Docs
description: 了解如何在 Azure Cosmos DB 中了解、指定及預估要求單位需求。
services: cosmos-db
author: rimman
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 06/26/2018
ms.author: rimman
ms.openlocfilehash: 66beeb2cc724f75d17a4c155f1cdb888153e8fbf
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286760"
---
# <a name="request-units-in-azure-cosmos-db"></a>Azure Cosmos DB 中的要求單位

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 是 Microsoft 的全球分散式多模型資料庫。 有了 Azure Cosmos DB，您就不需要租用虛擬機器、部署軟體或監視資料庫。 Microsoft 頂尖工程師會負責操作並持續監視 Azure Cosmos DB，以提供世界級的可用性、效能和資料保護能力。 您可以使用選擇的 API (例如 [SQL](documentdb-introduction.md)、[MongoDB](mongodb-introduction.md) 和[資料表](table-introduction.md) API) 來存取資料，以及透過 [Gremlin API](graph-introduction.md) 來存取圖表。 所有 API 全都是原生支援的。 

Azure Cosmos DB 的貨幣是「要求單位 (RU)」。 使用要求單位，就不需要保留讀取/寫入容量，或是佈建 CPU、記憶體及 IOPS。 Azure Cosmos DB 支援各種 API 來執行不同的作業，範圍可從簡單地讀取及寫入到複雜的圖表查詢。 因為並非所有要求都一樣，所以系統會根據處理要求所需的計算量，為要求指派標準化的要求單位數量。 作業的要求單位數目是先決的。 您可以透過回應標頭來追蹤 Azure Cosmos DB 中任何作業所耗用的要求單位數目。 

若要提供可預測的效能，請保留每秒 100 RU 單位的輸送量。 您可以使用 Azure Cosmos DB [要求單位計算機](https://www.documentdb.com/capacityplanner) \(英文\)，以[評估您的輸送量需求](request-units.md#estimating-throughput-needs)。

![輸送量計算機][5]

閱讀本文後，您將能夠回答下列問題：

* Azure Cosmos DB 中的要求單位和要求費用是什麼？
* 如何指定 Azure Cosmos DB 中的容器或一組容器的要求單位容量？
* 如何估計應用程式的要求單位需求？
* 如果超過 Azure Cosmos DB 中的容器或一組容器的要求單位容量，會發生什麼情況？

因為 Azure Cosmos DB 是多模型資料庫，請務必注意本文適用於 Azure Cosmos DB 中的所有資料模型和 API。 本文使用通用詞彙，例如，使用「容器」來泛指集合或圖表，以及使用「項目」來泛指資料表、文件、節點或實體。

## <a name="request-units-and-request-charges"></a>要求單位和要求費用

Azure Cosmos DB 可保留資源來滿足應用程式的輸送量需求，以提供快速且可預測的效能。 應用程式負載和存取模式會隨著時間變更。 Azure Cosmos DB 可協助您輕鬆地增加或減少可供應用程式使用的保留輸送量。

有了 Azure Cosmos DB，就可根據每秒處理的要求單位來指定保留的輸送量。 您可以將要求單位視為輸送量貨幣。 您會以每秒為基礎，保留一些保證可供應用程式使用的要求單位。 Azure Cosmos DB 中的每個作業 (包括寫入文件、執行查詢，以及更新文件) 都會耗用 CPU、記憶體和 IOPS。 也就是說，每個作業都會產生要求費用，這會以要求單位來表示。 當您了解影響要求單位費用的因素，以及應用程式的輸送量需求時，就可儘量以最符合經濟效益的方式來執行應用程式。 

## <a name="throughput-isolation-in-globally-distributed-databases"></a>輸送量隔離是分散在世界各地的資料庫

如果您將資料庫複寫至多個區域，Azure Cosmos DB 就會提供輸送量隔離，以確保某一個區域中的要求單位使用量不會影響另一個區域的要求單位使用量。 例如，如果您將資料寫入至某一個區域，並從另一個區域讀取資料，則用於在區域 A 執行寫入作業的要求單位不會取自用於在區域 B 執行讀取作業的要求單位。要求單位不會分散於您已部署資料庫的區域中。 複製資料庫的每個區域都已佈建充分的要求單位數目。 如需全域複寫的詳細資訊，請參閱[如何使用 Azure Cosmos DB 在全域散發資料](distribute-data-globally.md)。

## <a name="request-unit-considerations"></a>要求單位的考量
當您估計要佈建的要求單位數目時，請務必考量下列變數：

* **項目大小**。 大小增加時，讀取或寫入資料所耗用的要求單位數目也會增加。
* **項目屬性計數**。 假設預設會編製所有屬性的索引，撰寫文件、節點或實體所耗用的單位數就會隨著屬性計數增加而增加。
* **資料一致性**。 當您使用像是「強式」或「限定過期」的資料一致性模型時，會耗用額外的要求單位來讀取項目。
* **已編製索引的屬性**。 每個容器上的索引原則會判定預設要編製哪些索引的屬性。 您可以限制已編製索引的屬性數目或讓索引延遲編製，以減少寫入作業的要求單位耗用量。
* **編製文件索引**。 預設會自動編製每個項目的索引。 如果您選擇不要編製某些項目的索引，則會耗用較少的要求單位。
* **查詢模式**。 查詢的複雜性會影響針對作業所耗用的要求單位數目。 查詢結果數目、述詞數目、述詞性質、使用者定義函數的數目、來源資料的大小及預測，全都會影響查詢作業的成本。
* **指令碼使用方式**。 查詢、預存程序和觸發程序會根據所執行作業的複雜度來耗用要求單位。 開發應用程式時，請檢查要求費用標頭，進一步了解每個作業耗用要求單位容量的方式。

## <a name="estimating-throughput-needs"></a>估計輸送量需求
要求單位是要求處理成本的標準化量值。 單一要求單位代表讀取 (透過自我連結或識別碼) 由 10 個唯一屬性值 (不包括系統屬性) 所組成的單一 1 KB 項目所需的處理容量。 建立 (插入)、取代或刪除相同項目的要求會耗用服務的更多處理，因此需要更多的要求單位。 

> [!NOTE]
> 1 KB 項目搭配 1 個要求單位的基準，可與依據項目自我連結或識別碼的簡單 GET 對應。
> 
> 

例如，下表顯示在三個不同大小 (1 KB、4 KB 和 64 KB) 的項目 ，以及兩個不同效能等級 (500 次讀取/秒 + 100 次寫入/秒和 500 次讀取/秒 + 500 次寫入/秒)，要佈建多少要求單位。 在此範例中，會將資料一致性設定為 [工作階段]，並將編製索引原則設定為 [無]。

| 項目大小 | 讀取/秒 | 寫入/秒 | 要求單位
| --- | --- | --- | --- |
| 1 KB | 500 | 100 | (500 * 1) + (100 * 5) = 1,000 RU/s
| 1 KB | 500 | 500 | (500 * 1) + (500 * 5) = 3,000 RU/s
| 4 KB | 500 | 100 | (500 * 1.3) + (100 * 7) = 1,350 RU/s
| 4 KB | 500 | 500 | (500 * 1.3) + (500 * 7) = 4,150 RU/s
| 64 KB | 500 | 100 | (500 * 10) + (100 * 48) = 9,800 RU/s
| 64 KB | 500 | 500 | (500 * 10) + (500 * 48) = 29,000 RU/s


### <a name="use-the-request-unit-calculator"></a>使用要求單位計算機
您可以使用網頁型[要求單位計算機](https://www.documentdb.com/capacityplanner) \(英文\)，以協助微調輸送量估計。 此計算機可協助您估計適用於一般作業的要求單位需求，包括：

* 項目建立 (寫入)
* 項目讀取
* 項目刪除
* 項目更新

此工具也支援根據您提供的範例項目來估計資料儲存需求。

使用此工具：

1. 上傳一或多個具代表性的項目 (例如範例 JSON 文件)。
   
    ![將項目上傳至要求單位計算機][2]
2. 若要估計資料儲存需求，請輸入預期要儲存的項目 (例如文件、資料列或頂點) 總數。
3. 輸入需要的建立、讀取、更新和刪除作業數 (以每秒為基礎)。 若要估計項目更新作業的要求單位費用，請上傳一份步驟 1 中包含一般欄位更新的範例項目。 例如，如果項目更新通常會修改名為 *lastLogin* 和 *userVisits* 的兩個屬性，則請複製範例項目、更新那兩個屬性的值，然後上傳複製的項目。
   
    ![在要求單位計算機中輸入輸送量需求][3]
4. 選取 [計算]，然後檢查結果。
   
    ![要求單位計算機結果][4]

> [!NOTE]
> 如果項目類型與已編製索引屬性的大小和數目截然不同，請將每個一般項目「類型」的範例上傳至工具，然後計算結果。
> 
> 

### <a name="use-the-azure-cosmos-db-request-charge-response-header"></a>使用 Azure Cosmos DB 要求費用回應標頭
Azure Cosmos DB 服務的每個回應都會包括自訂標頭 (`x-ms-request-charge`)，其中包含指定要求所耗用的要求單位。 您也可以透過 Azure Cosmos DB SDK 存取此標頭。 在 .NET SDK 中，**RequestCharge** 是 **ResourceResponse** 物件的屬性。 針對查詢，Azure 入口網站中的 Azure Cosmos DB 資料總管會提供已執行查詢的要求費用資訊。 若要了解如何使用不同的多模型 API 來取得及設定輸送量，請參閱[設定及取得 Azure Cosmos DB 的輸送量](set-throughput.md)一文。

若要估計應用程式所需的保留輸送量，其中一個方法為對照應用程式所使用的代表性項目，以記錄執行一般作業的相關要求單位費用。 然後，估計您預期每秒會執行的作業數目。 此外，務必要測量並包含一般查詢和 Azure Cosmos DB 指令碼使用量。

> [!NOTE]
> 如果項目類型與已編製索引屬性的大小和數目截然不同，請記錄一般項目每個「類型」相關的適用作業要求單位費用。
> 
> 

例如，這些是您可能採取的步驟：

1. 記錄建立 (插入) 一般項目的要求單位費用。 
2. 記錄讀取一般項目的要求單位費用。
3. 記錄更新一般項目的要求單位費用。
4. 記錄一般且常見項目查詢的要求單位費用。
5. 記錄應用程式所使用任何自訂指令碼 (預存程序、觸發程序、使用者定義的函數) 的要求單位費用。
6. 根據您預期每秒執行的作業估計數目，來計算必要的要求單位數。

## <a name="a-request-unit-estimate-example"></a>要求單位估計範例
請考慮下列文件 (大小約 1 KB)：

```json
{
 "id": "08259",
  "description": "Cereals ready-to-eat, KELLOGG, KELLOGG'S CRISPIX",
  "tags": [
    {
      "name": "cereals ready-to-eat"
    },
    {
      "name": "kellogg"
    },
    {
      "name": "kellogg's crispix"
    }
  ],
  "version": 1,
  "commonName": "Includes USDA Commodity B855",
  "manufacturerName": "Kellogg, Co.",
  "isFromSurvey": false,
  "foodGroup": "Breakfast Cereals",
  "nutrients": [
    {
      "id": "262",
      "description": "Caffeine",
      "nutritionValue": 0,
      "units": "mg"
    },
    {
      "id": "307",
      "description": "Sodium, Na",
      "nutritionValue": 611,
      "units": "mg"
    },
    {
      "id": "309",
      "description": "Zinc, Zn",
      "nutritionValue": 5.2,
      "units": "mg"
    }
  ],
  "servings": [
    {
      "amount": 1,
      "description": "cup (1 NLEA serving)",
      "weightInGrams": 29
    }
  ]
}
```

> [!NOTE]
> 文件在 Azure Cosmos DB 中會變小，因此系統針對上述文件計算出的大小會稍微小於 1 KB。
> 
> 

下表顯示關於此項目一般作業的概略要求單位費用。 (概略要求單位費用假設帳戶一致性層級會設定為 [工作階段]，並自動編製所有項目的索引)。

| 作業 | 要求單位費用 |
| --- | --- |
| 建立項目 |~15 RU |
| 讀取項目 |~1 RU |
| 依識別碼查詢項目 |~2.5 RU |

下表顯示應用程式中所使用一般查詢的概略要求單位費用：

| 查詢 | 要求單位費用 | 傳回項目的數量 |
| --- | --- | --- |
| 依識別碼選取食物 |~2.5 RU |1 |
| 依製造商選取食物 |~7 RU |7 |
| 依食物群組選取並依重量排序 |~70 RU |100 |
| 選取食物群組中的前 10 個食物 |~10 RU |10 |

> [!NOTE]
> 要求單位費用會根據傳回的項目數而有所不同。
> 
> 

有了此資訊，便可根據每秒所預期的作業數和查詢數，以估計此應用程式的要求單位需求：

| 作業/查詢 | 每秒估計的數目 | 必要的要求單位數 |
| --- | --- | --- |
| 建立項目 |10 |150 |
| 讀取項目 |100 |100 |
| 依製造商選取食物 |25 |175 |
| 依食物群組選取 |10 |700 |
| 選取前 10 個 |15 |總共 150 個 |

在此情況下，您預期平均輸送量需求為每秒 1,275 RU。 四捨五入至最接近 100 的數目，您會針對此應用程式的容器 (或一組容器) 佈建每秒 1,300 RU。

## <a id="RequestRateTooLarge"></a> 超過 Azure Cosmos DB 中保留的輸送量限制
要求單位耗用量會以每秒的速率來評估。 如果應用程式超過佈建的要求單位速率，要求便會受到速率限制，直到該速率降到佈建的輸送量層級以下。 當要求受到速率限制時，伺服器會預先使用 `RequestRateTooLargeException` (HTTP 狀態碼 429) 結束要求，並傳回 `x-ms-retry-after-ms` 標頭。 此標頭會指出使用者在重試要求之前必須等候的時間量 (以毫秒為單位)。

    HTTP Status 429
    Status Line: RequestRateTooLarge
    x-ms-retry-after-ms :100

如果您使用 .NET 用戶端 SDK 和 LINQ 查詢，則在大部分的情況下，都不必處理此例外狀況。 目前的 .NET 用戶端 SDK 版本會隱含地攔截此回應、遵守伺服器指定的 retry-after 標頭，並自動重試要求。 除非有多個用戶端同時存取您的帳戶，否則下次重試將會成功。

如果您有多個用戶端會以高於要求速率的方式累積運作，則預設的重試行為可能會不敷使用，且用戶端會將具有狀態碼 429 的 `DocumentClientException` 擲回到應用程式。 在類似這樣的情況下，可以考慮在應用程式的錯誤處理常式中處理重試行為和邏輯，或為容器 (或一組容器) 提高佈建的輸送量。

## <a name="next-steps"></a>後續步驟
 
- 了解如何使用 Azure 入口網站和 SDK 來[設定和取得 Azure Cosmos DB 中的輸送量](set-throughput.md)。
- 了解 [Azure Cosmos DB 的效能和規模測試](performance-testing.md)。
- 若要深入了解 Azure Cosmos DB 資料庫的保留輸送量，請參閱 [Azure Cosmos DB 定價](https://azure.microsoft.com/pricing/details/cosmos-db/)和[在 Azure Cosmos DB 中進行資料分割](partition-data.md)。
- 若要深入了解 Azure Cosmos DB，請參閱 [Azure Cosmos DB 文件](https://azure.microsoft.com/documentation/services/cosmos-db/)。 

[2]: ./media/request-units/RUEstimatorUpload.png
[3]: ./media/request-units/RUEstimatorDocuments.png
[4]: ./media/request-units/RUEstimatorResults.png
[5]: ./media/request-units/RUCalculator2.png

