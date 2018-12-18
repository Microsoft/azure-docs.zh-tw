---
title: 知識探索服務 API 中的結構描述格式 | Microsoft Docs
description: 深入了解認知服務中的知識探索服務 (KES) API 結構描述格式。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: 3009392a5acb12a8f4df3d30a2cbe5e74f2172fc
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368071"
---
# <a name="schema-format"></a>結構描述格式
您可以在 JSON 檔案中指定結構描述，該檔案會描述在用來建立索引的資料檔案中，所含物件的屬性結構。  結構描述會為每個屬性指定名稱、資料類型、作業 (選擇性) 和同義字清單 (選擇性)。  物件中的每個屬性可能有 0 個以上的值。  以下是學術刊物領域的簡化範例：

``` json
{
  "attributes":[
    {"name":"Title", "type":"String"},
    {"name":"Year", "type":"Int32"},
    {"name":"Author", "type":"Composite"},
    {"name":"Author.Id", "type":"Int64", "operations":["equals"]},
    {"name":"Author.Name", "type":"String"},
    {"name":"Author.Affiliation", "type":"String"},
    {"name":"Keyword", "type":"String", "synonyms":"Keyword.syn"}
  ]
}
```

屬性名稱是區分大小寫的識別碼，以字母開頭，且只能包含字母 (A 到 Z)、數字 (0 到 9) 和底線 (\_)。  「logprob」保留屬性可用來指定物件之間的相對自然對數機率。

## <a name="attribute-type"></a>屬性類型
以下是所支援的屬性資料類型清單：

| 類型 | 說明 | 作業 | 範例 |
|------|-------------|------------|---------|
| 字串 | 字串 (1 到 1024 個字元) | equals、starts_with | "hello world" |
| Int32 | 帶正負號的 32 位元整數 | equals、starts_with、is_between | 2016 |
| Int64 | 帶正負號的 64 位元整數 | equals、starts_with、is_between | 9876543210 |
| 兩倍 | 雙精確度浮點值 | equals、starts_with、is_between | 1.602e-19 |
| 日期 | 日期 (1400-01-01 到 9999-12-31) | equals、is_between | '2016-03-14' |
| Guid | 全域唯一識別碼 | equals | "602DD052-CC47-4B23-A16A-26B52D30C05B" |
| Blob | 在內部壓縮的非索引資料 | *None* | 「讓地球上的每個人和每個組織有能力獲得更大成就」 |
| 複合 | 多個子屬性的複合體| *N/A* | { "Name":"harry shum", "Affiliation":"microsoft" } |

字串屬性可用來表示可能會顯示為使用者查詢一部分的字串值。  這些屬性支援 equals (完全相符) 作業，以及適用於查詢完成案例的 starts_with 作業，例如比對「micros」與「microsoft」。  未來的版本會支援用來處理拼字錯誤的不區分大小寫比對和模糊比對。

Int32/Int64/Double 屬性可用來表示數值。  is_between 作業可在執行階段支援不等比較 (lt、le、gt、ge)。  starts_with 作業可支援查詢完成案例，例如比對「20」與「2016」或比對「3」與「3.14」。 with "3.14".

Date 屬性可用來有效率地對日期值編碼。  is_between 作業可在執行階段支援不等比較 (lt、le、gt、ge)。
  
Guid 屬性可用來有效率地表示 GUID 值，並預設支援 equals 作業。

Blob 屬性可用來有效率地對可能較大的資料 Blob 進行編碼，以供對應物件的執行階段查閱使用，但不支援任何以 Blob 值內容為基礎的索引作業。

### <a name="composite-attributes"></a>複合屬性
複合屬性可用來表示屬性值群組。  每一個子屬性名稱的開頭為複合屬性名稱後接「.」。  複合屬性的值會指定為巢狀屬性值所在的 JSON 物件。  複合屬性可能會有多個物件值。  不過，複合屬性不可能會有本身為複合屬性的子屬性。

在前面的學術刊物範例中，這會讓服務能夠查詢「harry shum」在「microsoft」服務時所發表的論文。  若沒有複合屬性，此服務就只能查詢其中一位作者是「harry shum」以及其中一位作者服務於「microsoft」的論文。  如需詳細資訊，請參閱[複合查詢](SemanticInterpretation.md#composite-function)。

## <a name="attribute-operations"></a>屬性作業
根據預設，每個屬性都會建立索引以支援該屬性資料類型可用的所有作業。  如果不需要特定作業，則可以明確指定一組已建立索引的作業來減少索引大小。  在來自上述結構描述範例的下列程式碼片段中，Author.Id 屬性會建立索引，以便僅支援 equals 作業，而不會另外對 Int32 屬性支援 starts_with 和 is_between 作業。
```json
{"name":"Author.Id", "type":"Int32", "operations":["equals"]}
```

當文法內參考某個屬性時，就必須在結構描述中指定 starts_with 作業，讓服務能夠從部分查詢產生完成查詢。  

## <a name="attribute-synonyms"></a>屬性同義字
我們常常會希望利用同義字來參考特定字串屬性值。  例如，使用者可能會將「Microsoft」指稱為「MSFT」或「MS」。  在這些情況下，屬性定義可以指定和結構描述檔案位於相同目錄中的結構描述檔案名稱。  同義字檔案中的每一行會以下列 JSON 格式表示一個同義字項目：`["<canonical>", "<synonym>"]`。  在結構描述範例中，「AuthorName.syn」是包含 Author.Name 屬性同義字值的 JSON 檔案。

`{"name":"Author.Name", "type":"String", "synonyms":"AuthorName.syn"}`


標準值可能具有多個同義字。  多個標準值可能共用某個通用同義字。  在這種情況下，服務會透過多重解譯來解決一語多義的問題。  以下是與上述結構描述對應的 AuthorName.syn 同義字檔案範例：
```json
["harry shum","heung-yeung shum"]
["harry shum","h shum"]
["henry shum","h shum"]
...
```
