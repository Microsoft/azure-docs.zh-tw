---
title: 在 Azure 搜尋服務中呼叫認知搜尋 API 的教學課程 | Microsoft Docs
description: 在此教學課程中，將逐步說明資料擷取和轉換的 Azure 搜尋服務索引中，資料擷取、自然語言和影像 AI 處理的範例。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: luisca
ms.openlocfilehash: 4694d7a580c9544e43cf0b56b192b55c02257531
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730659"
---
# <a name="tutorial-learn-how-to-call-cognitive-search-apis-preview"></a>教學課程：了解如何呼叫認知搜尋 API (預覽)

在此教學課程中，您將了解在 Azure 搜尋服務中使用*認知技能*進行資料擴充程式設計的機制。 認知技能是會擷取影像的文字和文字表示法，並偵測語言、實體、關鍵片語等項目的自然語言處理 (NLP) 和影像分析作業。 其最終結果是，認知搜尋索引管線會在 Azure 搜尋服務索引中建立豐富的額外內容。 

在此教學課程中，您會發出 REST API 呼叫以執行下列工作：

> [!div class="checklist"]
> * 建立對索引路由中的範例資料進行擴充的索引管線
> * 套用內建的技能：實體辨識、語言偵測、文字操作和關鍵片語擷取
> * 了解如何藉由將技能集的輸入對應至輸出，將多個技術串聯在一起
> * 執行要求並檢閱結果
> * 重設索引和索引子以進行進一步開發

Azure 搜尋服務的輸出是全文檢索的可搜尋索引。 您可以使用其他標準功能來強化索引，例如[同義字](search-synonyms.md)、[評分設定檔](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)、[分析器](search-analyzers.md)和[篩選](search-filters.md)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

> [!NOTE]
> 認知搜尋目前為公開預覽狀態。 技能集執行、映像擷取及正規化目前為免費提供。 我們將在不久後宣布這些功能的定價。 

## <a name="prerequisites"></a>先決條件

剛開始使用認知搜尋嗎？ 請閱讀[「什麼是認知搜尋？」](cognitive-search-concept-intro.md) 以取得概念，或嘗試以[入口網站快速入門](cognitive-search-quickstart-blob.md)取得重要概念的實際操作簡介。

若要對 Azure 搜尋服務進行 REST 呼叫，請使用 PowerShell 或 Web 測試工具 (例如 Telerik Fiddler 或 Postman) 來編寫 HTTP 要求的公式。 如果您未曾使用過這些工具，請參閱[使用 Fiddler 或 Postman 探索 Azure 搜尋服務 REST API](search-fiddler.md)。

使用 [Azure 入口網站](https://portal.azure.com/)建立在端對端工作流程中使用的服務。 

### <a name="set-up-azure-search"></a>設定 Azure 搜尋服務

首先，請註冊 Azure 搜尋服務。 

1. 使用您的 Azure 帳戶登入 [Azure 入口網站](https://portal.azure.com)。

1. 按一下 [建立資源]，搜尋「Azure Search 搜尋服務」，然後按一下 [建立]。 如果您是第一次設定搜尋服務，請參閱[在入口網站中建立 Azure 搜尋服務](search-create-service-portal.md)。

  ![儀表板入口網站](./media/cognitive-search-tutorial-blob/create-service-full-portal.png "在入口網站中建立 Azure 搜尋服務")

1. 針對資源群組建立資源群組，以包含您在此教學課程中建立的所有資源。 這可讓您在完成教學課程後能夠更輕鬆地清除資源。

1. 針對 [位置]，選擇 [美國中南部] 或 [西歐]。 預覽版本目前只能在這些區域中使用。

1. 針對 [定價層]，您可以建立 [免費] 服務以完成教學課程和快速入門。 若要使用您自己的資料進行深入調查，請建立[付費服務](https://azure.microsoft.com/pricing/details/search/)，例如**基本**或**標準**。 

  「免費」服務僅限使用 3 個索引、上限為 16 MB 的 Blob 大小，以及 2 分鐘的索引編製，這對執行完整認知搜尋功能而言是不夠的。 若要檢視不同層級的限制，請參閱[服務限制](search-limits-quotas-capacity.md)。

  > [!NOTE]
  > 認知搜尋目前為公開預覽狀態。 目前，在所有層級中都可執行技能集，包括免費層。 我們將在不久後宣布此功能的定價。

1. 將服務釘選到儀表板，以快速存取服務資訊。

  ![入口網站中的服務定義頁面](./media/cognitive-search-tutorial-blob/create-search-service.png "入口網站中的服務定義頁面")

1. 建立服務之後，請收集下列資訊：[概觀] 頁面中的 [URL]，以及 [金鑰] 頁面中的 [API 金鑰] (主要或次要)。

  ![入口網站中的端點和金鑰資訊](./media/cognitive-search-tutorial-blob/create-search-collect-info.png "入口網站中的端點和金鑰資訊")

### <a name="set-up-azure-blob-service-and-load-sample-data"></a>設定 Azure Blob 服務並載入範例資料

擴充管線會從 Azure 資料來源中提取資料。 來源資料必須來自 [Azure 搜尋服務索引子](search-indexer-overview.md)支援的資料來源類型。 針對此練習，我們會使用 Blob 儲存體來展現多個內容類型。

1. [載入範例資料](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4)。 下載由不同類型的小型檔案集組成的範例資料。 

1. 註冊 Azure Blob 儲存體、建立儲存體帳戶、登入儲存體總管，並建立名為 `basicdemo` 的容器。 如需關於上述所有步驟的指示，請參閱 [Azure 儲存體總管快速入門](../storage/blobs/storage-quickstart-blobs-storage-explorer.md)。

1. 使用 Azure 儲存體總管，在您建立的 容器中按一下 [上傳]`basicdemo`，以上傳範例檔案。

1. 範例檔案載入之後，請取得 Blob 儲存體的容器名稱和連接字串。 您可以瀏覽至 Azure 入口網站中的儲存體帳戶，以執行此動作。 在 [存取金鑰] 上，複製 [連接字串] 欄位。

  連接字串應為類似於下列範例的 URL：

      ```http
      DefaultEndpointsProtocol=https;AccountName=cogsrchdemostorage;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

此外也有其他方式可指定連接字串，例如提供共用存取簽章。 若要深入了解資料來源認證，請參閱[編製 Azure Blob 儲存體的索引](search-howto-indexing-azure-blob-storage.md#Credentials)。

## <a name="create-a-data-source"></a>建立資料來源

現在，您的服務和來源檔案皆已備妥，您可以開始組合索引管線的元件。 首先請從[資料來源物件](https://docs.microsoft.com/rest/api/searchservice/create-data-source)開始；此物件會指示 Azure 搜尋服務如何擷取外部來源資料。

在此教學課程中，請使用可編寫及傳送 HTTP 要求的 REST API 和工具，例如 PowerShell、Postman 或 Fiddler。 在要求標頭中，提供您在建立 Azure 搜尋服務時所使用的服務名稱，以及為您的搜尋服務產生的 API 金鑰。 在要求本文中，指定 Blob 容器名稱和連接字串。

### <a name="sample-request"></a>範例要求
```http
POST https://[service name].search.windows.net/datasources?api-version=2017-11-11-Preview
Content-Type: application/json  
api-key: [admin key]  
```
#### <a name="request-body-syntax"></a>要求本文的語法
```json
{   
    "name" : "demodata",  
    "description" : "Demo files to demonstrate cognitive search capabilities.",  
    "type" : "azureblob",
    "credentials" :
    { "connectionString" :
      "DefaultEndpointsProtocol=https;AccountName=<your account name>;AccountKey=<your account key>;"
    },  
    "container" : { "name" : "<your blob container name>" }
}  
```
傳送要求。 Web 測試工具應會傳回確認成功的狀態碼 201。 

由於這是您第一次的要求，請查看 Azure 入口網站，以確認已在 Azure 搜尋服務中建立資料來源。 在搜尋服務儀表板頁面上，確認 [資料來源] 圖格有新的項目。 您可能需要等候幾分鐘，讓入口網站重新整理頁面。 

  ![入口網站中的資料來源圖格](./media/cognitive-search-tutorial-blob/data-source-tile.png "入口網站中的資料來源圖格")

如果發生 403 或 404 錯誤，請檢查要求建構：`api-version=2017-11-11-Preview` 位於端點上，`api-key` 應位於標頭中的 `Content-Type` 後面，且其值必須是適用於搜尋服務的值。 您可以在此教學課程的其餘步驟中重複使用該標頭。

> [!TIP]
> 在您執行多個工作之前，建議您於此時確認搜尋服務正在其中一個提供預覽功能的支援位置上執行：美國中南部或西歐。

## <a name="create-a-skillset"></a>建立技能集

在此步驟中，您會定義一組要套用至資料的擴充步驟。 我們將每個擴充步驟稱為*技能*，一組擴充步驟則稱之為*技能集*。 此教學課程會使用為技能集[預先定義的認知技能](cognitive-search-predefined-skills.md)：

+ [語言偵測](cognitive-search-skill-language-detection.md)，用以識別內容的語言。

+ [文字分割](cognitive-search-skill-textsplit.md)，用以在呼叫關鍵片語擷取技能之前，將大型內容分成較小的區塊。 關鍵片語擷取可接受不超過 50,000 個字元的輸入。 此有些範例檔案需要進行分割，以符合此限制。

+ [具名實體辨識](cognitive-search-skill-named-entity-recognition.md)，用以從 Blob 容器中的內容擷取組織名稱。

+ [關鍵片語擷取](cognitive-search-skill-keyphrases.md)，用以提取最高排名的關鍵片語。 

### <a name="sample-request"></a>範例要求
在您進行此 REST 呼叫前，如果您的工具不會在呼叫間保留要求標頭，請記得在下方的要求中取代服務名稱和管理金鑰。 

此要求會建立技能集。 在此教學課程的其餘部分請參考技能集名稱 ```demoskillset```。

```http
PUT https://[servicename].search.windows.net/skillsets/demoskillset?api-version=2017-11-11-Preview
api-key: [admin key]
Content-Type: application/json
```
#### <a name="request-body-syntax"></a>要求本文的語法
```json
{
  "description": 
  "Extract entities, detect language and extract key-phrases",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations", "targetName": "organizations"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "languageCode",
          "targetName": "languageCode"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "textSplitMode" : "pages", 
      "maximumPageLength": 4000,
      "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      },
      { 
        "name": "languageCode",
        "source": "/document/languageCode"
      }
    ],
    "outputs": [
      {
            "name": "textItems",
            "targetName": "pages"
      }
    ]
  },
  {
      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
      "context": "/document/pages/*",
      "inputs": [
        {
          "name": "text", "source": "/document/pages/*"
        },
        {
          "name":"languageCode", "source": "/document/languageCode"
        }
      ],
      "outputs": [
        {
          "name": "keyPhrases",
          "targetName": "keyPhrases"
        }
      ]
    }
  ]
}
```

傳送要求。 Web 測試工具應會傳回確認成功的狀態碼 201。 

#### <a name="about-the-request"></a>關於要求

請留意每個頁面套用關鍵片語擷取技能的情形。 透過將內容設定為 ```"document/pages/*"```，將可擴充每個文件/頁面陣列成員 (文件中的每個頁面) 的此執行。

每項技術會分別對文件的內容執行。 在處理期間，Azure 搜尋服務會萃取每份文件，以讀取不同檔案格式的內容。 找到來自來源檔案的文字時，會將文字放入產生的 ```content``` 欄位中，每份文件一個欄位。 據此，請將輸入設定為 ```"/document/content"```。

以下顯示技能集的圖形化表示法。 

![了解技能集](media/cognitive-search-tutorial-blob/skillset.png "了解技能集")

輸出可以對應至索引、作為下游技能的輸入，或在使用語言代碼時同時作為對應和輸入。 在索引中，語言代碼可用於篩選。 作為輸入時，文字分析技能會使用語言代碼指出斷字方面的語言規則。

如需技能集基本概念的詳細資訊，請參閱[如何定義技能集](cognitive-search-defining-skillset.md)。

## <a name="create-an-index"></a>建立索引

在本節中，您會藉由指定要在可搜尋索引中包含哪些欄位以及每個欄位的搜尋屬性，來定義索引結構描述。 欄位具有類型，並且可取用決定如何使用欄位 (可搜尋、可排序等等) 的屬性。 索引中的欄位名稱不需要完全符合來源中的欄位名稱。 在後續步驟中，您可以在索引子中新增欄位對應以連接來源-目的地欄位。 針對此步驟，請使用與搜尋應用程式相關的欄位命名慣例來定義索引。

此練習會使用下列欄位和欄位類型：

| 欄位名稱： | id       | 內容   | languageCode | keyPhrases         | 組織     |
|--------------|----------|-------|----------|--------------------|-------------------|
| 欄位類型： | Edm.String|Edm.String| Edm.String| List<Edm.String>  | List<Edm.String>  |


### <a name="sample-request"></a>範例要求
在您進行此 REST 呼叫前，如果您的工具不會在呼叫間保留要求標頭，請記得在下方的要求中取代服務名稱和管理金鑰。 

此要求會建立索引。 在此教學課程的其餘部分請使用索引名稱 ```demoindex```。

```http
PUT https://[servicename].search.windows.net/indexes/demoindex?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```
#### <a name="request-body-syntax"></a>要求本文的語法

```json
{
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "key": true,
      "searchable": true,
      "filterable": false,
      "facetable": false,
      "sortable": true
    },
    {
      "name": "content",
      "type": "Edm.String",
      "sortable": false,
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "languageCode",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "keyPhrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```
傳送要求。 Web 測試工具應會傳回確認成功的狀態碼 201。 

若要深入了解如何定義索引，請參閱[建立索引 (Azure 搜尋服務 REST API)](https://docs.microsoft.com/rest/api/searchservice/create-index)。


## <a name="create-an-indexer-map-fields-and-execute-transformations"></a>建立索引子、對應欄位，並執行轉換

至此，您已建立資料來源、技能集和索引。 這三項元件構成了[索引子](search-indexer-overview.md)的一部分，可將各項資料一併提取到多階段的單一作業中。 若要在索引子中結合這些項目，您必須定義欄位對應。 欄位對應是索引子定義的一部分，可在您提交要求時執行轉換。

針對非擴充索引，如果欄位名稱或資料類型不會精確比對，或是想要使用函式，則索引子定義會提供選用的 *fieldMappings* 區段。

對於具有擴充管線的認知搜尋工作負載，索引子必須要有 *outputFieldMappings*。 當內部程序 (擴充管線) 為欄位值的來源時，就會使用這些對應。 *outputFieldMappings* 特有的行為包括能夠處理在擴充 (透過塑形器技能) 的過程中建立的複雜類型。 此外，每份文件可能會有多個元素 (例如，一份文件中有多個組織)。 *outputFieldMappings* 建構可以指示系統 將元素的集合壓平合併為單一記錄。

### <a name="sample-request"></a>範例要求

在您進行此 REST 呼叫前，如果您的工具不會在呼叫間保留要求標頭，請記得在下方的要求中取代服務名稱和管理金鑰。 

此外也請提供索引子的名稱。 在此教學課程的其餘部分，您可以將其參照為 ```demoindexer```。

```http
PUT https://[servicename].search.windows.net/indexers/demoindexer?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```
#### <a name="request-body-syntax"></a>要求本文的語法

```json
{
  "name":"demoindexer", 
  "dataSourceName" : "demodata",
  "targetIndexName" : "demoindex",
  "skillsetName" : "demoskillset",
  "fieldMappings" : [
        {
          "sourceFieldName" : "metadata_storage_path",
          "targetFieldName" : "id",
          "mappingFunction" : 
            { "name" : "base64Encode" }
        },
        {
          "sourceFieldName" : "content",
          "targetFieldName" : "content"
        }
   ],
  "outputFieldMappings" : 
  [
        {
          "sourceFieldName" : "/document/organizations", 
          "targetFieldName" : "organizations"
        },
        {
          "sourceFieldName" : "/document/pages/*/keyPhrases/*", 
          "targetFieldName" : "keyPhrases"
        },
        {
            "sourceFieldName": "/document/languageCode",
            "targetFieldName": "languageCode"
        }      
  ],
  "parameters":
  {
    "maxFailedItems":-1,
    "maxFailedItemsPerBatch":-1,
    "configuration": 
    {
        "dataToExtract": "contentAndMetadata",
        "imageAction": "generateNormalizedImages"
        }
  }
}
```

傳送要求。 Web 測試工具應會傳回確認處理成功的狀態碼 201。 

預期此步驟需要幾分鐘的時間才能完成。 即使資料集很小，分析技能仍需要大量計算。 某些技能 (例如影像分析) 需要長時間執行。

> [!TIP]
> 建立索引子時會叫用管線。 資料的存取、輸入和輸出的對應或作業順序若有問題，都會在這個階段中出現。 若要透過程式碼或指令碼的變更重新執行管線，您可能需要先卸除物件。 如需詳細資訊，請參閱[重設並重新執行](#reset)。

### <a name="explore-the-request-body"></a>探索要求本文

指令碼會將 ```"maxFailedItems"``` 設定為 -1，這會指示索引引擎在資料匯入期間忽略錯誤。 此設定有其用處，因為示範資料來源中只有少量文件。 若要有較大的資料來源，您應將值設定為大於 0。

同時也請注意組態參數中的 ```"dataToExtract":"contentAndMetadata"``` 陳述式。 此陳述式會指示索引子自動擷取不同檔案格式的內容，以及每個檔案的相關中繼資料。 

在擷取內容時，您可以設定 ```ImageAction```，以從在資料來源中找到的影像擷取文字。 ```"ImageAction":"generateNormalizedImages"``` 會指示索引子從影像中擷取文字 (例如，從「停」交通號誌中擷取「停」這個字)，並將其內嵌為內容欄位的一部分。 此行為適用於內嵌在文件中的影像 (例如 PDF 內的影像)，以及在資料來源中找到的影像 (例如 JPG 檔案)。

在此預覽中，```"ImageAction"``` 唯一的有效值為 ```"generateNormalizedImages"```。

## <a name="check-indexer-status"></a>檢查索引子狀態

索引子經定義後，將會在您提交要求時自動執行。 根據您所定義的認知技能，索引編製所需的時間可能會超出您的預期。 若要確認索引子是否仍在執行，請傳送下列要求以檢查索引子狀態。

```http
GET https://[servicename].search.windows.net/indexers/demoindexer/status?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

回應會指出索引子是否正在執行。 索引編製完成後，請對 STATUS 端點使用另一個 HTTP GET (如上所列)，以查看在擴充期間是否發生任何錯誤和警告的報告。  

某些來源檔案和技能的組合常會出現警告，這並不一定表示有問題。 在此教學課程中，警告是良性的 (例如，沒有來自 JPEG 檔案的文字輸入)。 您可以檢閱狀態回應，以取得在索引編製期間所發出警告的詳細資訊。
 
## <a name="verify-content"></a>驗證內容

索引編製完成後，請執行會傳回個別欄位內容的查詢。 根據預設，Azure 搜尋服務會傳回前 50 項結果。 範例資料很小，因此預設值即足堪使用。 不過，在使用較大的資料集時，您可能需要在查詢字串中加上參數，以傳回較多結果。 如需相關指示，請參閱[如何在 Azure 搜尋服務中對結果分頁](search-pagination-page-layout.md)。

在驗證步驟中，您會查詢所有欄位的索引。

```http
GET https://[servicename].search.windows.net/indexes/demoindex?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

輸出將是索引結構描述，附有每個欄位的名稱、類型和屬性。

提交第二個 `"*"` 查詢，以傳回單一欄位的所有內容，例如 `organizations`。

```http
GET https://[servicename].search.windows.net/indexes/demoindex/docs?search=*&$select=organizations&api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

對其他欄位重複前述步驟：此練習中的內容、語言、關鍵片語和組織。 您可以透過使用逗號分隔清單的 `$select` 傳回多個欄位。

您可以使用 GET 或 POST，視查詢字串的複雜度和長度而定。 如需詳細資訊，請參閱[使用 REST API 進行查詢](https://docs.microsoft.com/azure/search/search-query-rest-api)。

<a name="access-enriched-document"></a>

## <a name="accessing-the-enriched-document"></a>存取擴充的文件

認知搜尋可讓您查看擴充文件的結構。 擴充的文件是在擴充期間建立的暫時性結構，在程序完成後即會刪除。

若要擷取在索引編製期間建立之擴充文件的快照集，請將名為 ```enriched``` 的欄位新增至您的索引。 索引子會自動在該欄位中傾印該文件所有擴充項目的字串表示法。

```enriched``` 欄位將會包含一個字串，作為記憶體內部的擴充文件在 JSON 中的邏輯表示法。  不過，該欄位的值是有效的 JSON 文件。 引號會逸出，因此您必須將 `\"` 取代為 `"`，才能以格式化 JSON 的形式檢視文件。  

```enriched``` 欄位的用途是偵錯，只是為了幫助您了解以運算式進行評估之內容的邏輯圖形。 它可以作為了解和偵錯技能集的實用工具。

重複前述練習 (包括 `enriched` 欄位)，以擷取擴充文件的內容：

### <a name="request-body-syntax"></a>要求本文的語法
```json
{
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "key": true,
      "searchable": true,
      "filterable": false,
      "facetable": false,
      "sortable": true
    },
    {
      "name": "content",
      "type": "Edm.String",
      "sortable": false,
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "languageCode",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "keyPhrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "sortable": false,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "enriched",
      "type": "Edm.String",
      "searchable": false,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```
<a name="reset"></a>

## <a name="reset-and-rerun"></a>重設並重新執行

在管線開發的早期實驗階段中，設計反覆項目最實用的方法是從 Azure 搜尋服務中刪除物件，並讓您的程式碼重建它們。 資源名稱是唯一的。 刪除物件可讓您使用相同的名稱重新建立它。

若要使用新的定義為您的文件重新編製索引：

1. 刪除索引以移除保存的資料。 刪除索引子以在服務上重新建立它。
2. 修改技能集和索引定義。
3. 在服務上重新建立索引和索引子以執行管線。 

您可以使用入口網站來刪除索引和索引子。 技能集只能透過 HTTP 命令來刪除 (如果您決定將其刪除)。

```http
DELETE https://[servicename].search.windows.net/skillsets/demoskillset?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

成功刪除時會傳回狀態碼 204。

當您的程式碼成熟時，您可以精簡重建策略。 如需詳細資訊，請參閱[如何重建索引](search-howto-reindex.md)。

## <a name="takeaways"></a>重要心得

此教學課程示範了藉由建立下列元件組件來建置擴充索引管線的基本步驟：資料來源、技能集、索引和索引子。

[預先定義的技能](cognitive-search-predefined-skills.md)已透過輸入和輸出連同技能集定義和鏈結技能的機制一起導入。 您也已了解在將管線中的擴充值路由至 Azure 搜尋服務上的可搜尋索引時，索引子定義中必須要有 `outputFieldMappings`。

最後，您了解到如何測試結果並重設系統，以進行進一步的反覆運算。 您已了解對索引發出查詢，會傳回由擴充的索引管線建立的輸出。 此版本提供了檢視內部建構 (由系統建立的擴充文件) 的機制。 您也已了解如何檢查索引子狀態，以及在重新執行管線之前應刪除哪些物件。

## <a name="clean-up-resources"></a>清除資源

在完成教學課程後，最快速的清除方式是刪除包含 Azure 搜尋服務和 Azure Blob 服務的資源群組。 如果您將這兩項服務放在相同群組中，此時刪除資源群組，將會永久刪除其中的所有內容，包括服務與您為此教學課程建立的任何已儲存內容。 在入口網站中，資源群組名稱位在每個服務的 [概觀] 頁面上。

## <a name="next-steps"></a>後續步驟

以自訂技能自訂或擴充管線。 建立自訂技能並將其新增至技能集，以便讓您自行撰寫的文字或影像分析上線。 

> [!div class="nextstepaction"]
> [範例：建立自訂技能](cognitive-search-create-custom-skill-example.md)
