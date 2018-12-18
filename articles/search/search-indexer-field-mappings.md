---
title: Azure 搜尋服務索引子中的欄位對應
description: 設定 Azure 搜尋服務索引子欄位對應交代欄位名稱和資料表示的差異
author: chaosrealm
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 08/30/2017
ms.author: eugenesh
ms.openlocfilehash: 51fa689030c4a8ce4e900ecd600cdd0524aa13d9
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42139803"
---
# <a name="field-mappings-in-azure-search-indexers"></a>Azure 搜尋服務索引子中的欄位對應
使用 Azure 搜尋服務索引子時，您偶爾會發現置身於輸入資料不完全符合目標索引結構描述的情況中。 在這些情況下，您可以使用 **欄位對應** 將您的資料轉換成所需的形狀。

欄位對應在某些情況下很有用︰

* 您的資料來源有 `_id`欄位，但 Azure 搜尋服務不允許以底線開頭的欄位名稱。 欄位對應允許您「重新命名」欄位。
* 例如，您想要用相同的資料來源資料填入索引中的多個欄位，因為您想要將不同的分析器套用到這些欄位。 欄位對應讓您「分岔」資料來源欄位。
* 您必須以 Base64 格式編碼或解碼資料。 欄位對應支援數個 **對應函式**，包括 Base64 編碼和解碼的函式。   

## <a name="setting-up-field-mappings"></a>設定欄位對應
您可以在建立新的索引子時，使用 [建立索引子](https://msdn.microsoft.com/library/azure/dn946899.aspx) API 加入欄位對應。 您可以使用 [更新索引子](https://msdn.microsoft.com/library/azure/dn946892.aspx) API 管理編製索引子的欄位對應。

欄位對應包含三個部分︰

1. `sourceFieldName`，表示資料來源中的欄位。 這是必要屬性。
2. 選擇性的 `targetFieldName`，表示搜尋索引中的欄位。 如果省略，則會使用資料來源中的相同名稱。
3. 選擇性的 `mappingFunction`，可以使用數個預先定義的函式之一轉換您的資料。 函式的完整清單 [如下](#mappingFunctions)。

欄位對應會加入索引子定義上的 `fieldMappings` 陣列中。

例如，以下是容納欄位名稱差異的方式︰

```JSON

PUT https://[service name].search.windows.net/indexers/myindexer?api-version=[api-version]
Content-Type: application/json
api-key: [admin key]
{
    "dataSourceName" : "mydatasource",
    "targetIndexName" : "myindex",
    "fieldMappings" : [ { "sourceFieldName" : "_id", "targetFieldName" : "id" } ]
}
```

索引子可以有多個欄位對應。 例如，以下是「分岔」欄位的方式︰

```JSON

"fieldMappings" : [
    { "sourceFieldName" : "text", "targetFieldName" : "textStandardEnglishAnalyzer" },
    { "sourceFieldName" : "text", "targetFieldName" : "textSoundexAnalyzer" }
]
```

> [!NOTE]
> Azure 搜尋服務會使用不區分大小寫的比較，來解析欄位對應的欄位和函式名稱。 這很方便 (大小寫不需要完全正確)，但這表示資料來源或索引不能有只以大小寫區分的欄位。  
>
>

<a name="mappingFunctions"></a>

## <a name="field-mapping-functions"></a>欄位對應函式
目前支援的函式如下︰

* [base64Encode](#base64EncodeFunction)
* [base64Decode](#base64DecodeFunction)
* [extractTokenAtPosition](#extractTokenAtPositionFunction)
* [jsonArrayToStringCollection](#jsonArrayToStringCollectionFunction)

<a name="base64EncodeFunction"></a>

## <a name="base64encode"></a>base64Encode
執行輸入字串的安全 URL  Base64 編碼。 假設輸入以 UTF-8 編碼。

### <a name="sample-use-case---document-key-lookup"></a>範例使用案例 - 文件索引鍵查閱
Azure 搜尋服務文件索引鍵只顯示安全的 URL 字元 (因為，例如，客戶必須能夠使用[查閱 API](https://docs.microsoft.com/rest/api/searchservice/lookup-document) 處理文件)。 如果資料包含不安全的 URL 字元，而且您想要使用它填入搜尋索引中的索引鍵欄位，請使用這個函式。 索引鍵一旦編碼，您可以使用 base64 解碼擷取其原始值。 如需詳細資訊，請參閱[base64 編碼和解碼](#base64details)一節。

#### <a name="example"></a>範例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : { "name" : "base64Encode" }
  }]
```

### <a name="sample-use-case---retrieve-original-key"></a>範例使用案例 - 擷取原始索引鍵
您有 Blob 索引子，會以 Blob 路徑中繼資料作為文件索引鍵來索引 Blob。 擷取編碼過的文件索引鍵後，要解碼路徑及下載 Blob。

#### <a name="example"></a>範例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : { "name" : "base64Encode", "parameters" : { "useHttpServerUtilityUrlTokenEncode" : false } }
  }]
 ```

如果您不需要依索引鍵查閱文件，也不需要將已編碼的內容解碼，您可以省去對應函式的 `parameters`，它會將 `useHttpServerUtilityUrlTokenEncode` 預設為 `true`。 否則，請參閱 [base64 詳細資料](#base64details)一節以決定要使用的設定。

<a name="base64DecodeFunction"></a>

## <a name="base64decode"></a>base64Decode
執行輸入字串的 Base64 解碼。 輸入假定為安全的 URL  Base64 編碼字串。

### <a name="sample-use-case"></a>範例使用案例
Blob 的自訂中繼資料值必須以 ASCII 編碼。 您可以使用 Base64 編碼表現 Blob 自訂中繼資料中任意的 UTF-8 字串。 不過，若要進行有意義的搜尋，您可以在填入搜尋索引時，使用這個函式將編碼的資料變回「一般」字串。

#### <a name="example"></a>範例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "Base64EncodedMetadata",
    "targetFieldName" : "SearchableMetadata",
    "mappingFunction" : { "name" : "base64Decode", "parameters" : { "useHttpServerUtilityUrlTokenDecode" : false } }
  }]
```

如果您未指定任何 `parameters`，則 `useHttpServerUtilityUrlTokenDecode` 的預設值為 `true`。 請參閱 [base64 詳細資料](#base64details)一節以決定要使用的設定。

<a name="base64details"></a>

### <a name="details-of-base64-encoding-and-decoding"></a>base64 編碼和解碼的詳細資訊
Azure 搜尋服務支援兩種 base64 編碼：HttpServerUtility URL 權杖以及 沒有填補的 URL 安全 base64 編碼。 如果您想將文件編碼供查閱、將值編碼讓索引子解碼、或將由索引子編碼的欄位解碼，必須使用和對應函式相同的編碼。

如果用於編碼和解碼的 `useHttpServerUtilityUrlTokenEncode` 或 `useHttpServerUtilityUrlTokenDecode` 參數分別設定為 `true`，則 `base64Encode` 的行為會像 [HttpServerUtility.UrlTokenEncode](https://msdn.microsoft.com/library/system.web.httpserverutility.urltokenencode.aspx) 且 `base64Decode` 的行為會像 [HttpServerUtility.UrlTokenDecode](https://msdn.microsoft.com/library/system.web.httpserverutility.urltokendecode.aspx)。

如果您未使用完整的 .NET Framework (亦即，您使用 .NET Core 或其他程式設計環境) 來產生索引鍵值以模擬 Azure 搜尋服務的行為，則請將 `useHttpServerUtilityUrlTokenEncode` 和 `useHttpServerUtilityUrlTokenDecode` 設定為 `false`。 base64 編碼和解碼公用程式函式可能會與 Azure 搜尋服務不同，取決於您使用的程式庫。

下表比較 `00>00?00` 字串的不同 base64 編碼。 若要判斷您的 base64 函式需要的額外處理 (如果有的話)，將您的程式庫編碼函式套用在 `00>00?00` 字串，然後比較輸出與預期的輸出 `MDA-MDA_MDA`。

| 編碼 | Base64 編碼的輸出 | 程式庫編碼之後的額外處理 | 程式庫解碼之前的額外處理 |
| --- | --- | --- | --- |
| Base64 加填補 | `MDA+MDA/MDA=` | 使用 URL 安全的字元，並移除填補 | 使用標準 base64 字元，並加上填補 |
| Base64 無填補 | `MDA+MDA/MDA` | 使用 URL 安全的字元 | 使用標準 base64 字元 |
| URL 安全的 base64 填補加填補 | `MDA-MDA_MDA=` | 移除填補 | 加上填補 |
| URL 安全的 base64 無填補 | `MDA-MDA_MDA` | None | None |

<a name="extractTokenAtPositionFunction"></a>

## <a name="extracttokenatposition"></a>extractTokenAtPosition
使用指定的分隔符號分割字串欄位，並在分割結果的指定位置挑選權杖。

例如，如果輸入是 `Jane Doe`，而 `delimiter` 是 `" "` (空格) 且 `position` 為 0，則結果是 `Jane`；如果 `position` 為 1，則結果是 `Doe`。 如果位置參考不存在的權杖，會傳回錯誤。

### <a name="sample-use-case"></a>範例使用案例
資料來源包含 `PersonName` 欄位，而您想要將它編製為 `FirstName` 和 `LastName` 兩個不同欄位的索引。 您可以使用這個函式來分割以空格字元作為分隔符號的輸入。

### <a name="parameters"></a>參數
* `delimiter`︰分割輸入字串時，用為分隔符號的字串。
* `position`：分割輸入字串後，要挑選的權杖以零為基底位置的整數。    

### <a name="example"></a>範例
```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "FirstName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 0 } }
  },
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "LastName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 1 } }
  }]
```

<a name="jsonArrayToStringCollectionFunction"></a>

## <a name="jsonarraytostringcollection"></a>jsonArrayToStringCollection
將格式化為字串 JSON 陣列的字串，轉換為可用來填入索引中 `Collection(Edm.String)` 欄位的字串陣列。

例如，輸入字串是 `["red", "white", "blue"]`，則 `Collection(Edm.String)` 類型的目標欄位會填入 `red`、`white`、`blue` 三個值。 若為無法剖析為 JSON 字串陣列的輸入值，會傳回錯誤。

### <a name="sample-use-case"></a>範例使用案例
Azure SQL 資料庫沒有內建資料類型，不能自然對應至 Azure 搜尋服務的 `Collection(Edm.String)` 欄位。 若要填入字串集合欄位，請將來源資料格式化為 JSON 字串陣列，並使用這個函式。

### <a name="example"></a>範例
```JSON

"fieldMappings" : [
  { "sourceFieldName" : "tags", "mappingFunction" : { "name" : "jsonArrayToStringCollection" } }
]
```


## <a name="help-us-make-azure-search-better"></a>協助我們改進 Azure 搜尋服務
如果您有功能要求或改進的想法，請在我們的 [UserVoice 網站](https://feedback.azure.com/forums/263029-azure-search/)與我們連絡。
