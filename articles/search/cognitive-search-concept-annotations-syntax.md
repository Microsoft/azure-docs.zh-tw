---
title: 在 Azure 搜尋服務中參考認知搜尋管線中各項輸入和輸出內的註解 | Microsoft Docs
description: 說明註解語法，以及如何在 Azure 搜尋服務中參考認知搜尋管線中某技能集之輸入和輸出內的註解。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 0e074e93ecbe80c3acf3481c0d33917fbe5090c6
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34640901"
---
# <a name="how-to-reference-annotations-in-a-cognitive-search-skillset"></a>如何參考認知搜尋技能集中的註釋

在本文中，我們將使用範例說明各種案例，讓您了解如何參考技能定義中的註解。 當文件內容流經技能集時，將會增添註解而有所擴充。 註解可以作為進一步下游擴充的輸入，或對應至索引中的輸出欄位。 
 
本文中的範例，以 [Azure Blob 索引子](search-howto-indexing-azure-blob-storage.md)在文件萃取階段的執行過程中自動產生的*內容*欄位為基礎。 從 Blob 容器參照文件時，請使用 `"/document/content"` 之類的格式，其中，*內容*欄位是*文件*的一部分。 

## <a name="background-concepts"></a>背景概念

在檢閱語法之前，讓我們先回顧幾個重要的概念，以深入了解本文稍後所提供的範例。

| 詞彙 | 說明 |
|------|-------------|
| 擴充的文件 | 擴充的文件是管線所建立及使用的內部結構，用來保存與文件相關的所有註解。 請將擴充的文件設想為註解的樹狀結構。 一般而言，從上一個註解建立的註解將會成為其子系。<p/>只有在技能集執行期間，才會有擴充的文件。 在內容對應至搜尋索引後，即不再需要擴充的文件。 雖然您無法直接擴充的文件互動，但在建立技能集時，具有文件的心智模型將有所幫助。 |
| 擴充內容 | 執行擴充所在的內容，以擴充何種元素的觀點來表示。 根據預設，擴充內容會在 `"/document"` 層級上，其範圍僅限於個別文件。 當技能執行時，該技能的輸出會成為[已定義內容的屬性](#example-2)。|

<a name="example-1"></a>
## <a name="example-1-simple-annotation-reference"></a>範例 1：簡單註解參考

在 Azure Blob 儲存體中，假設您有多種檔案都參考了您想要使用具名實體辨識來擷取的人員名稱。 在下列技能定義中，`"/document/content"` 是整份文件的文字表示法，而 "people" 則是對識別為人員的實體擷取到的完整名稱。

由於預設內容為 `"/document"`，因此人員清單此時可以參考為 `"/document/people"`。 在這種情況下，`"/document/people"` 是註解，且此時有可能對應至索引中的欄位，或用於相同技能集中的另一項技能。

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
    "categories": [ "Person"],
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      }
    ]
  }
```

<a name="example-2"></a>

## <a name="example-2-reference-an-array-within-a-document"></a>範例 2：參考文件內的陣列

此範例是根據上一個範例而建置的，可說明如何對相同的文件叫用擴充步驟多次。 假設上一個範例產生了一個字串陣列，其中包含來自於單一文件的 10 個人員名稱。 合理的後續步驟可能是從完整名稱中擷取姓氏的二次擴充。 由於有 10 個名稱，因此您會在此文件中呼叫此步驟 10 次，每個人員各一次。 

若要叫用正確數量的反覆項目，請將內容設定為 `"/document/people/*"`，其中，星號 (`"*"`) 代表擴充的文件中所有作為 `"/document/people"` 下階的節點。 雖然這項技能只會在技能陣列中定義一次，但對於文件中的每個成員都會呼叫此技能，直到所有成員皆處理完成。

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the last name from a full name",
    "uri": "http://names.azurewebsites.net/api/GetLastName",
    "context" : "/document/people/*",
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "fullname",
        "source": "/document/people/*"
      }
    ],
    "outputs": [
      {
        "name": "lastname",
        "targetName": "last"
      }
    ]
  }
```

如果附註是字串的陣列或集合，您可以將特定的成員作為目標，而不是整個陣列。 前述範例會內容所代表的每個節點下產生名為 `"last"` 的註解。 如果您想要參照此系列的註解，您可以使用語法 `"/document/people/*/last"`。 如果您想要參照特定註解，則可以使用明確索引 `"/document/people/1/last`" 來參考文件中識別的第一個人員的姓氏。 請注意，在此語法中，陣列為「單一索引」陣列。

<a name="example-3"></a>

## <a name="example-3-reference-members-within-an-array"></a>範例 3：參考陣列內的成員

有時候，您必須將特定類型的所有註解分組在一起，以將其傳遞至特定技能。 請考慮建立假設性的自訂技能，以從擷取自範例 2 的所有姓氏中識別出最常見的姓氏。 若只要將姓氏提供給自訂技能，請將內容指定為 `"/document"`，並將輸入指定為 `"/document/people/*/lastname"`。

請注意，`"/document/people/*/lastname"` 的基數大於文件的基數。 姓氏節點可能有 10 個，而此文件只有一個文件節點。 在此情況下，系統會自動建立一個 `"/document/people/*/lastname"` 陣列，其中包含文件中的所有元素。

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the most common string from an array of strings",
    "uri": "http://names.azurewebsites.net/api/MostCommonString",
    "context" : "/document",
    "inputs": [
      {
        "name": "strings",
        "source": "/document/people/*/lastname"
      }
    ],
    "outputs": [
      {
        "name": "mostcommon",
        "targetName": "common-lastname"
      }
    ]
  }
```



## <a name="see-also"></a>另請參閱
+ [如何將自訂技能整合到擴充管線中](cognitive-search-custom-skill-interface.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md)
+ [建立技能集 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何將擴充的欄位對應至索引](cognitive-search-output-field-mapping.md)
