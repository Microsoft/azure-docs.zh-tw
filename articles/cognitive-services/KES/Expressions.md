---
title: 知識探索服務 API 中的結構化查詢運算式 | Microsoft Docs
description: 了解如何在認知服務的知識探索服務 (KES) API 中使用結構化查詢運算式。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: 070ee311a1153bc9fb59870dce68f385a43b15f1
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368062"
---
# <a name="structured-query-expression"></a>結構化查詢運算式
結構化查詢運算式會指定一組要針對資料索引進行評估的作業。  其包含屬性查詢運算式和更高層級的函式。  請使用[評估](evaluateMethod.md)方法來計算符合運算式的物件。  以下範例來自學術刊物領域，會傳回自 2013 年後由 Jaime Teevan 所撰寫的刊物。

`And(Composite(Author.Name=='jaime teevan'),Y>=2013)`

結構化查詢運算式可能取自[解譯](interpretMethod.md)要求，每個解譯的語意輸出都是結構化查詢運算式，會傳回符合輸入自然語言查詢的索引物件。  或者，您也可以使用本節所述的語法，手動撰寫結構化查詢運算式。

## <a name="attribute-query-expression"></a>屬性查詢運算式
屬性查詢運算式會根據對特定屬性所做的比對，識別一組物件。  根據[結構描述](SchemaFormat.md)中所指定的屬性類型和已編制索引的作業，可支援不同的比對作業：

| 類型 | 作業 | 範例 |
|------|-------------|------------|
| 字串 | equals | Title='latent semantic analysis'  (canonical + synonyms) |
| 字串 | equals | Author.Name=='susan t dumais'  (canonical only)|
| 字串 | starts_with | Title='latent s'... |
| Int32/Int64/Double | equals | Year=2000 |
| Int32/Int64/Double | starts_with | Year='20'... (any decimal value starting with "20") |
| Int32/Int64/Double | is_between | Year&lt;2000 <br/> Year&lt;=2000 <br/> Year&gt;2000 <br/> Year&gt;=2000 <br/> Year=[2010,2012) (includes only left boundary value: 2010, 2011) <br/> Year=[2000,2012] (includes both boundary values: 2010, 2011, 2012) |
| 日期 | equals | BirthDate='1984-05-14' |
| 日期 | is_between | BirthDate&lt;='2008/03/14' <br/> PublishDate=['2000-01-01','2009-12-31'] |
| Guid | equals | Id='602DD052-CC47-4B23-A16A-26B52D30C05B' |


例如，運算式 "Title='latent s'..." 會符合其標題開頭為字串 "latent s" 的所有學術刊物。  若要評估此運算式，在用來建置索引的結構描述中，Title 屬性必須指定 "starts_with" 作業。

若屬性有相關聯的同義字，查詢運算式可以使用 "==" 運算子指定其標準值符合指定字串的物件，也可以使用 "=" 運算子指定其任何標準/同義字值相符的物件。  這兩種方式都需要在屬性定義中指定 "equals" 運算子。


## <a name="functions"></a>Functions
有一組內建函式可從基本的屬性查詢建構更複雜的查詢運算式。

### <a name="and-function"></a>And 函式
`And(expr1, expr2)`

傳回兩個輸入查詢運算式的交集。

下列範例會傳回西元 2000 年所發行，有關於資訊擷取的學術刊物：

`And(Year=2000, Keyword=='information retrieval')`

### <a name="or-function"></a>Or 函式
`Or(expr1, expr2)`

傳回兩個輸入查詢運算式的聯集。

下列範例會傳回西元 2000 年所發行，有關於資訊擷取或使用者模型化的學術刊物：

`And(Year=2000, Or(Keyword='information retrieval', Keyword='user modeling'))`

### <a name="composite-function"></a>Composite 函式
`Composite(expr)`

其所傳回的運算式，會封裝一個內部運算式，而此內部運算式是由針對常見複合屬性的子屬性所進行的查詢所組成的。  任何相符資料物件的複合屬性至少要有一個個別滿足內部運算式的值，才會封裝。  請注意，針對複合屬性的子屬性所建立的查詢運算式必須先使用 Composite() 函式加以封裝，才能與其他查詢運算式結合。

例如，下列運算式會傳回 "harry shum" 任職於 "microsoft" 時所撰寫的學術刊物：

```
Composite(And(Author.Name="harry shum", 
              Author.Affiliation="microsoft"))
```

另一方面，下列運算式則會傳回其中一位作者是 "harry shum" 以及其中一個所屬單位是 "microsoft" 的學術刊物：

```
And(Composite(Author.Name="harry shum"), 
    Composite(Author.Affiliation="microsoft"))
```
