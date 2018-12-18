---
title: 如何在 Azure 搜尋服務中模型化複雜資料類型 | Microsoft Docs
description: 在 Azure 搜尋服務索引中，可以使用扁平化資料列集和 Collections 資料類型來模型化巢狀或階層式資料結構。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
tags: complex data types; compound data types; aggregate data types
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/01/2017
ms.openlocfilehash: 81298bedd43a89ea948753dffc5f80248f5429ca
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31799068"
---
# <a name="how-to-model-complex-data-types-in-azure-search"></a>如何在 Azure 搜尋服務中模型化複雜資料類型
用來填入 Azure 搜尋服務索引的外部資料集，有時包含不會整齊細分成表格式資料列集的階層式或巢狀子結構。 這類結構的範例可能包含單一客戶的多個位置和電話號碼、單一 SKU 的多種色彩和大小、單一書籍的多位作者等等。 就模型化而論，您可能會看到這些結構稱之為「複雜資料類型」、「複合資料類型」、「合成資料類型」或「彙總資料類型」等等。

Azure 搜尋服務中原先不支援複雜資料類型，但經過實證的因應措施包括結構扁平化的程序 (包含兩個步驟)，然後使用 **Collection** 資料類型來重新構成內部結構。 遵循本文所述的技巧，以便搜尋、Facet 處理、篩選和排序內容。

## <a name="example-of-a-complex-data-structure"></a>複雜資料結構的範例
一般而言，有問題的資料會以一組 JSON 或 XML 文件的形式存在，或以項目形式存在於 NoSQL 存放區 (例如 Azure Cosmos DB)。 在結構上，挑戰源自於有多個需要搜尋和篩選的子項目。  在說明因應措施時，請首先採用下列 JSON 文件，其中列出一組連絡人做為範例︰

~~~~~
[
  {
    "id": "1",
    "name": "John Smith",
    "company": "Adventureworks",
    "locations": [
      {
        "id": "1",
        "description": "Adventureworks Headquarters"
      },
      {
        "id": "2",
        "description": "Home Office"
      }
    ]
  }, 
  {
    "id": "2",
    "name": "Jen Campbell",
    "company": "Northwind",
    "locations": [
      {
        "id": "3",
        "description": "Northwind Headquarter"
      },
      {
        "id": "4",
        "description": "Home Office"
      }
    ]
}]
~~~~~

雖然名為 ‘id’、‘name’ 和 ‘company’ 的欄位可以輕易地逐一對應成 Azure 搜尋服務索引中的欄位，但 ‘locations’ 欄位包含位置陣列，並有一組位置識別碼以及位置描述。 假設 Azure 搜尋服務沒有支援此方法的資料類型，我們需要在 Azure 搜尋服務中進行模型化的不同方法。 

> [!NOTE]
> Kirk Evans 也會在部落格文章 [Indexing Azure Cosmos DB with Azure Search](https://blogs.msdn.microsoft.com/kaevans/2015/03/09/indexing-documentdb-with-azure-seach/) 中說明這個技巧，文中顯示稱為「資料扁平化」的技巧，因此您會有稱為 `locationsID` 和 `locationsDescription` 的欄位 (兩者皆為 [collections](https://msdn.microsoft.com/library/azure/dn798938.aspx) (或字串陣列))。   
> 
> 

## <a name="part-1-flatten-the-array-into-individual-fields"></a>第 1 部分︰將陣列壓平成為個別的欄位
若要建立可容納此資料集的 Azure 搜尋服務索引，請為巢狀子結構建立個別的欄位︰資料類型為 [collections](https://msdn.microsoft.com/library/azure/dn798938.aspx) (或字串陣列) 的 `locationsID` 和 `locationsDescription`。 在這些欄位中，您會將值 '1' 和 '2' 的索引編製成 John Smith 的 `locationsID` 欄位，將值 '3' 和 '4' 的索引編製成 Jen Campbell 的 `locationsID` 欄位。  

Azure 搜尋服務中的資料如下所示︰ 

![範例資料，2 個資料列](./media/search-howto-complex-data-types/sample-data.png)

## <a name="part-2-add-a-collection-field-in-the-index-definition"></a>第 2 部分︰在索引定義中加入 collection 欄位
在索引結構描述中，欄位定義看起來類似此範例。

~~~~
var index = new Index()
{
    Name = indexName,
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("name", DataType.String) { IsSearchable = true, IsFilterable = false, IsSortable = false, IsFacetable = false },
        new Field("company", DataType.String) { IsSearchable = true, IsFilterable = false, IsSortable = false, IsFacetable = false },
        new Field("locationsId", DataType.Collection(DataType.String)) { IsSearchable = true, IsFilterable = true, IsFacetable = true },
        new Field("locationsDescription", DataType.Collection(DataType.String)) { IsSearchable = true, IsFilterable = true, IsFacetable = true }
    }
};
~~~~

## <a name="validate-search-behaviors-and-optionally-extend-the-index"></a>驗證搜尋行為並選擇性地擴充索引
假設您已建立索引並載入資料，您現在即可測試解決方案，以驗證對資料集執行的搜尋查詢。 每個 **collection** 欄位都應該**可搜尋**、**可篩選**和**可 Facet 處理**。 您應該能夠執行下列查詢︰

* 尋找所有在 ‘Adventureworks Headquarters’ 工作的人員。
* 取得在「家庭辦公室」工作的人員計數。  
* 在「家庭辦公室」工作的人員中，顯示他們在其他哪些辦公室工作以及每個位置的人員計數。  

當您需要進行結合位置識別碼和位置描述的搜尋時，這個技巧就不管用。 例如︰

* 尋找所有具有「家庭辦公室」且位置識別碼為 4 的人員。  

如果您還記得原始內容看起來如下︰

~~~~
   {
        id: '4',
        description: 'Home Office'
   }
~~~~

不過，既然我們已將資料分成個別的欄位，我們便無法得知 Jen Campbell 的家庭辦公室是否與 `locationsID 3` 或 `locationsID 4` 相關。  

若要處理這種情況，請在索引中定義另一個可將所有資料結合成單一集合的欄位。  此範例中，我們將此欄位稱為 `locationsCombined` 並將以 `||` 分隔內容，雖然您可以為您的內容選擇任何您認為是一組獨特字元的分隔符號。 例如︰ 

![範例資料，含分隔符號的 2 個資料列](./media/search-howto-complex-data-types/sample-data-2.png)

使用此 `locationsCombined` 欄位中，我們現在即可容納更多查詢，例如︰

* 顯示在位置識別碼為 '4' 的「家庭辦公室」工作的人員計數。  
* 搜尋在位置識別碼為 '4' 的「家庭辦公室」工作的人員。 

## <a name="limitations"></a>限制
這個技巧可用於許多案例，但並不適用於每個案例。  例如︰

1. 如果您的複雜資料類型中沒有一組靜態欄位，而且沒有辦法將所有可能的類型對應至單一欄位。 
2. 更新巢狀物件時需要一些額外的工作，才能精確地判斷需要在 Azure 搜尋服務索引中更新的內容

## <a name="sample-code"></a>範例程式碼
您可以看到範例說明如何在 Azure 搜尋服務中編製複雜 JSON 資料集的索引，並且在 [GitHub 儲存機制](https://github.com/liamca/AzureSearchComplexTypes)對此資料集執行多項查詢。

## <a name="next-step"></a>後續步驟
[票選複雜資料類型的原生支援](https://feedback.azure.com/forums/263029-azure-search) ，並提供您希望我們對於功能實作考量的任何其他輸入。 您也可以直接在 Twitter 上透過 @liamca 聯繫我。

