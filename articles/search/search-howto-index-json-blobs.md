---
title: 使用 Azure 搜尋服務 Blob 索引子編製索引 JSON Blob
description: 使用 Azure 搜尋服務 Blob 索引子編製索引 JSON Blob
author: chaosrealm
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: eugenesh
ms.openlocfilehash: 752df29200a5e020ccf10f511ae2f02c0d72bd48
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34362997"
---
# <a name="indexing-json-blobs-with-azure-search-blob-indexer"></a>使用 Azure 搜尋服務 Blob 索引子編製索引 JSON Blob
本文說明如何設定 Azure 搜尋服務 Blob 索引子，從 Azure Blob 儲存體中的 JSON blob 擷取結構化的內容。

Azure Blob 儲存體中的 JSON blob 通常是單一 JSON 文件或 JSON 陣列。 Azure 搜尋服務中的 Blob 索引子可以剖析其中一種結構，取決於您如何設定在要求上的 **parsingMode** 參數。

| JSON 文件 | parsingMode | 說明 | 可用性 |
|--------------|-------------|--------------|--------------|
| 一個 blob 一個 | `json` | 將 JSON blob 當作單一文字區塊來剖析。 每一個 JSON blob 會變成單一 Azure 搜尋服務文件。 | 通常可以在 [REST](https://docs.microsoft.com/rest/api/searchservice/indexer-operations) 和 [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer) API 中使用。 |
| 一個 blob 多個 | `jsonArray` | 剖析 blob 中的 JSON 陣列，陣列的每個元素會變成不同的 Azure 搜尋服務文件。  | 預覽版中，使用 [REST api-version=`2017-11-11-Preview`](search-api-2017-11-11-preview.md) 和 [.NET SDK Preview](https://aka.ms/search-sdk-preview)。 |

> [!Note]
> 預覽版 API 是針對測試與評估，不應該用於生產環境。
>

## <a name="setting-up-json-indexing"></a>設定 JSON 編製索引
JSON blob 的索引編製類似於一般文件擷取，是 Azure 搜尋服務中所有索引子通用的三步驟工作流程。

### <a name="step-1-create-a-data-source"></a>步驟 1：建立資料來源

第一個步驟是提供索引子使用的資料來源連線資訊。 資料來源類型 (在此指定為 `azureblob`) 決定了索引子會叫用哪一些資料擷取行為。 針對編製 JSON blob 索引，JSON 文件和陣列的資料來源定義是相同的。 

    POST https://[service name].search.windows.net/datasources?api-version=2017-11-11
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }   

### <a name="step-2-create-a-target-search-index"></a>步驟 2：建立目標搜尋索引 

索引子要搭配索引結構描述。 如果您使用 API (而不是入口網站)，請事先準備好索引，以便在索引子作業中指定它。 

> [!Note]
> 在入口網站中透過 **import** 動作公開索引子，可公開有限數目的正式運作索引子。 通常，import 工作流程可以根據來源中的中繼資料建構初步索引。 如需詳細資訊，請參閱[在入口網站中將資料匯入 Azure 搜尋服務](search-import-data-portal.md)。

### <a name="step-3-configure-and-run-the-indexer"></a>步驟 3：設定及執行索引子

到目前為止，資料來源和索引的定義都符合不限定 parsingMode 的情況。 不過，在索引子設定的第 3 個步驟，做法將分道揚鑣，取決於您在 Azure 搜尋服務索引中如何剖析及結構化 JSON blob 內容。

呼叫索引子時，請這麼做：

+ 將 **parsingMode** 參數設為 `json` (將每個 blob 當作單一文件加以索引) ，或設為 `jsonArray` (如果您的 blob 包含 JSON 陣列，且您需要將陣列的每個元素視為個別文件)。

+ 選擇性使用**欄位對應**挑選用來填入目標搜尋索引的來源 JSON 文件屬性。 若採用 JSON 陣列，如果陣列為較低層級屬性，您可以設定文件根目錄，指出陣列放在 blob 中的位置。

> [!IMPORTANT]
> 當您使用 `json` 或 `jsonArray` 剖析模式時，Azure 搜尋服務會假設資料來源中的所有 blob 都包含 JSON。 如果您需要支援在相同的資料來源中混用 JSON 和非 JSON Blob，請在 [UserVoice 網站](https://feedback.azure.com/forums/263029-azure-search)上讓我們知道。


## <a name="how-to-parse-single-json-blobs"></a>如何剖析單一 JSON blob

根據預設， [Azure 搜尋服務 Blob 索引子](search-howto-indexing-azure-blob-storage.md) 會將 JSON Blob 剖析為單一的文字區塊。 通常，您會想要保留 JSON 文件的結構。 例如，假設您在 Azure Blob 儲存體中有下列 JSON 文件：

    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13"
            "tags" : [ "search", "storage", "howto" ]    
        }
    }

### <a name="indexer-definition-for-single-json-blobs"></a>單一 JSON blob 的索引子定義

使用 Azure 搜尋服務 blob 索引子，類似先前範例的 JSON 文件會被剖析成單一 Azure 搜尋服務文件。 索引子會比對來源中的 "text"、"datePublished"、"tags"，找出與目標欄位具有相同名稱和類型的索引，然後載入索引。

設定則在索引子作業的主體中提供。 前文提過預先定義的資料來源物件，，會指定資料來源類型和連線資訊。 此外，目標索引在您的服務中也必須是空的容器。 排程和參數為選擇性，如果省略它們，索引子會立即執行，使用 `json` 作為剖析模式。

完整指定的要求看起來如下：

    POST https://[service name].search.windows.net/indexers?api-version=2017-11-11
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } }
    }

如前所述，不一定要使用欄位對應。 指定索引的 "text"、"datePublished"、"tags" 欄位，blob 索引子可以推斷正確的對應，故要求中不需要欄位對應。

## <a name="how-to-parse-json-arrays-preview"></a>如何剖析 JSON 陣列 (預覽版)

或者，您可以選擇 JSON 陣列預覽版功能。 當 blob 包含「JSON 物件的陣列」，且您希望陣列的每個元素成為個別的 Azure 搜尋服務文件，這個功能特別實用。 以下列 JSON blob 為例，您可以將 Azure 搜尋服務索引填入三個不同的文件，每個都具有 "id" 和 "text" 欄位。  

    [
        { "id" : "1", "text" : "example 1" },
        { "id" : "2", "text" : "example 2" },
        { "id" : "3", "text" : "example 3" }
    ]

### <a name="indexer-definition-for-a-json-array"></a>JSON 陣列的索引子定義

若採用 JSON 陣列，索引子要求會使用預覽版 API 和 `jsonArray` 剖析器。 這兩個是編製 JSON blob 索引時的陣列專屬必要做法。

    POST https://[service name].search.windows.net/indexers?api-version=2017-11-11-Preview
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }

同樣的，如前所述，不一定要使用欄位對應。 指定索引的 "id" 和 "text" 欄位，blob 索引子可以推斷正確的對應，不需要欄位對應清單。

<a name="nested-json-arrays"></a>

### <a name="nested-json-arrays"></a>巢狀的 JSON 陣列
如果您想要為 JSON 物件陣列編製索引，但是該陣列以巢狀方式位於文件中的某處？ 您可以使用 `documentRoot` 組態屬性選擇哪一個屬性包含陣列。 例如，如果您的 Blob 看起來像這樣︰

    {
        "level1" : {
            "level2" : [
                { "id" : "1", "text" : "Use the documentRoot property" },
                { "id" : "2", "text" : "to pluck the array you want to index" },
                { "id" : "3", "text" : "even if it's nested inside the document" }  
            ]
        }
    }

使用這個設定來索引 `level2` 屬性中包含的陣列：

    {
        "name" : "my-json-array-indexer",
        ... other indexer properties
        "parameters" : { "configuration" : { "parsingMode" : "jsonArray", "documentRoot" : "/level1/level2" } }
    }

## <a name="using-field-mappings-to-build-search-documents"></a>使用欄位對應建置搜尋文件

當來源和目標的欄位不能完全對齊時，您可以在要求主體中定義欄位對應區段，明確指定欄位對欄位的關聯。

目前，Azure 搜尋服務無法直接編製索引任意 JSON 文件，因為它只支援基本資料類型、字串陣列和 GeoJSON 點。 不過，您可以使用 **欄位對應** 挑選 JSON 文件的幾個部分，並將它們「上移」到搜尋文件的最上層欄位。 如需了解欄位對應的基本知識，請參閱 [Azure 搜尋服務索引子中的欄位對應](search-indexer-field-mappings.md)。

回到我們的範例 JSON 文件︰

    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13"
            "tags" : [ "search", "storage", "howto" ]    
        }
    }

假設搜尋索引有下列欄位︰`Edm.String` 類型的 `text`、`Edm.DateTimeOffset` 類型的 `date`、`Collection(Edm.String)` 類型的 `tags`。 請注意來源中的 "datePublished" 與索引中的 `date` 欄位之間的差異。 若要將 JSON 對應到所需形狀，請使用下列欄位對應︰

    "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
      ]

對應中的來源欄位名稱是使用 [JSON 指標](http://tools.ietf.org/html/rfc6901) 標記法指定。 以正斜線開始參考 JSON 文件的根目錄，然後使用正斜線分隔的路徑挑選所需的屬性 (使用任意層級的巢狀結構)。

您也可以使用以零為起始的索引來參考個別陣列元素。 比方說，若要從上述範例挑選 "tags" 陣列的第一個元素，請如下所示使用欄位對應︰

    { "sourceFieldName" : "/article/tags/0", "targetFieldName" : "firstTag" }

> [!NOTE]
> 如果欄位對應路徑中的來源欄位名稱參考在 JSON 中不存在的屬性，則會略過該對應且不會產生錯誤。 這麼做是為了讓我們可支援具有不同結構描述 (這是常見的使用案例) 的文件。 因為沒有任何驗證，您必須小心避免在欄位對應規格中出現錯字。
>
>

## <a name="example-indexer-request-with-field-mappings"></a>範例：索引子要求搭配欄位對應

下列範例是完整指定的索引子內容，包括欄位對應：

    POST https://[service name].search.windows.net/indexers?api-version=2017-11-11
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } },
      "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
        ]
    }


## <a name="help-us-make-azure-search-better"></a>協助我們改進 Azure 搜尋服務
如果您有功能要求或改進的想法，請在我們的 [UserVoice 網站](https://feedback.azure.com/forums/263029-azure-search/)與我們連絡。

## <a name="see-also"></a>另請參閱

+ [Azure 搜尋服務中的索引子](search-indexer-overview.md)
+ [使用 Azure 搜尋服務編製 Azure Blob 儲存體的索引](search-howto-index-json-blobs.md)
+ [使用 Azure 搜尋服務 blob 索引子編製 CSV blob 的索引](search-howto-index-csv-blobs.md)
+ [教學課程：搜尋 Azure Blob 儲存體中的半結構化資料](search-semi-structured-data.md)