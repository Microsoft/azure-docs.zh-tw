---
title: 適用於 Azure Cosmos DB 的 SQL 查詢 | Microsoft Docs
description: 了解 Azure Cosmos DB 的 SQL 語法、資料庫概念及 SQL 查詢。 SQL 可作為 Azure Cosmos DB 中的 JSON 查詢語言。
keywords: sql 語法, sql 查詢, sql 查詢, json 查詢語言, 資料庫概念和 sql 查詢, 彙總函式
services: cosmos-db
author: LalithaMV
manager: kfile
editor: monicar
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: conceptual
ms.date: 08/10/2018
ms.author: laviswa
ms.openlocfilehash: 766a2a9a2b71d9cd013f26b843d413d7603ab1fd
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44051352"
---
# <a name="query-azure-cosmos-db-data-with-sql-queries"></a>使用 SQL 查詢來查詢 Azure Cosmos DB 資料

Microsoft Azure Cosmos DB 支援在 SQL API 帳戶上使用 SQL (結構化查詢語言) 作為 JSON 查詢語言來查詢文件。 為 Azure Cosmos DB 設計查詢語言時，下列兩項是我們的考量目標：

* 我們不是發明新的查詢語言，而是讓 Azure Cosmos DB 支援 SQL，也就是最熟悉且熱門的查詢語言之一。 Azure Cosmos DB SQL 提供一個正式的程式設計模型，可在 JSON 文件上進行豐富的查詢。  

* Azure Cosmos DB 會使用 JavaScript 的程式設計模型作為查詢語言的基礎。 SQL API 是以 JavaScript 的類型系統、運算式評估和函式叫用為基礎。 這除了其他功能之外，還進而提供自然程式設計模型來進行關聯式投射、跨 JSON 文件的階層式導覽、自我聯結、空間查詢，以及叫用完全以 JavaScript 撰寫的使用者定義函式 (UDF)。 

本文會透過簡單的 JSON 文件引導您執行一些 SQL 查詢範例。 若要深入了解 Azure Cosmos DB SQL 語言語法，請參閱 [SQL 語法參考](sql-api-sql-query-reference.md)文章。 

## <a id="GettingStarted"></a>開始使用 SQL 命令
讓我們建立兩個簡單的 JSON 文件，並對該資料進行查詢。 我們考慮使用兩個關於家庭的 JSON 文件，並將這些 JSON 文件插入集合中，然後查詢此資料。 這裡有 Andersen 一家和 Wakefield 一家的簡單 JSON 文件，其中包括父母、孩子 (以及寵物)、地址及登記資訊。 這份文件包含字串、數字、布林值、陣列和巢狀屬性。 

**文件 1**  

```JSON
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "parents": [
     { "firstName": "Thomas" },
     { "firstName": "Mary Kay"}
  ],
  "children": [
     {
         "firstName": "Henriette Thaulow", 
         "gender": "female", 
         "grade": 5,
         "pets": [{ "givenName": "Fluffy" }]
     }
  ],
  "address": { "state": "WA", "county": "King", "city": "seattle" },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

以下是第二份具有些微差異的文件：使用 `givenName` 和 `familyName` 來取代 `firstName` 和 `lastName`。

**文件 2**  

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam", 
        "givenName": "Jesse", 
        "gender": "female", "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller", 
         "givenName": "Lisa", 
         "gender": "female", 
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```

現在，讓我們嘗試對此資料執行一些查詢，以了解 Azure Cosmos DB SQL 查詢語言的一些重要部分。 

**查詢 1**：例如，下列查詢會傳回 id 欄位符合 `AndersenFamily` 的文件。 由於它是一個 `SELECT *`，所以查詢的輸出是完整的 JSON 文件，若要了解此語法，請參閱 [SELECT 陳述式](sql-api-sql-query-reference.md#select-query)：

```sql
    SELECT * 
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

**查詢 2**：現在，請考慮我們需要以不同形式來重新格式化 JSON 輸出的情況。 當地址所在城市的名稱與省/市的名稱相同時，此查詢會投射具有兩個所選欄位 (Name 和 City) 的新 JSON 物件。 在此情況下，"NY, NY" 相符。   

```sql
    SELECT {"Name":f.id, "City":f.address.city} AS Family 
    FROM Families f 
    WHERE f.address.city = f.address.state
```

**結果**

```json
    [{
        "Family": {
            "Name": "WakefieldFamily", 
            "City": "NY"
        }
    }]
```

**查詢 3**：此查詢會傳回家庭中所有小孩的名字，而該家庭的 id 符合依據居住城市所排序的 `WakefieldFamily`。

```sql
    SELECT c.givenName 
    FROM Families f 
    JOIN c IN f.children 
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.address.city ASC
```

**結果**

```json
    [
      { "givenName": "Jesse" }, 
      { "givenName": "Lisa"}
    ]
```

以下是從目前所見範例中得到的幾個 Cosmos DB 查詢語言觀點：  

* 因為 SQL API 的處理對象是 JSON 值，所以它處理的是樹狀形式的實體，而不是資料列和資料行。 因此，此語言可讓您參考樹狀目錄中任意深度的節點 (例如 `Node1.Node2.Node3…..Nodem`)，該節點與參考 `<table>.<column>` 之兩個部分參考的關聯式 SQL 類似。   

* 結構化查詢語言可處理無結構描述資料。 因此，需要動態繫結類型系統。 相同的運算式可能會對不同的文件產生不同的類型。 查詢的結果會是有效的 JSON 值，但不保證會是固定的結構描述。  

* Azure Cosmos DB 只支援嚴謹的 JSON 文件。 這表示類型系統和運算式只能處理 JSON 類型。 如需詳細資料，請參閱 [JSON 規格](http://www.json.org/) \(英文\)。  

* Cosmos DB 集合是 JSON 文件的無結構描述容器。 集合中文件內及跨文件之資料實體中的關係，會由內含項目以隱含方式擷取，而不是由主索引鍵和外部索引鍵關係擷取。 這是本文稍後討論之文件內聯結中值得指出的重要部分。

## <a id="SelectClause"></a> 子句

根據 ANSI-SQL 標準，每個查詢都會包含 SELECT 子句以及選擇性的 FROM 和 WHERE 子句。 針對每個查詢，通常都會列舉 FROM 子句中的來源。 接著，會對來源套用 WHERE 子句中的篩選，以擷取 JSON 文件的子集。 最後，使用 SELECT 子句來投射選取清單中所要求的 JSON 值。 若要深入了解語法，請參閱 [SELECT 語法](sql-api-sql-query-reference.md#bk_select_query)。

下列範例示範一般的 SELECT 查詢。 

**查詢**

```sql
    SELECT f.address
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "address": {
        "state": "WA", 
        "county": "King", 
        "city": "seattle"
      }
    }]
```

### <a name="nested-properties"></a>巢狀屬性
在下列範例中，我們將投射兩個巢狀屬性 `f.address.state` and `f.address.city`。

**查詢**

```sql
    SELECT f.address.state, f.address.city
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "state": "WA", 
      "city": "seattle"
    }]
```

投射也支援 JSON 運算式 (如下列範例所示)：

**查詢**

```sql
    SELECT { "state": f.address.state, "city": f.address.city, "name": f.id }
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "$1": {
        "state": "WA", 
        "city": "seattle", 
        "name": "AndersenFamily"
      }
    }]
```

讓我們在這裡查看 `$1` 角色。 `SELECT` 子句需要建立 JSON 物件，且因為未提供任何索引鍵，所以我們將使用以 `$1` 開頭的隱含引數變數名稱。 例如，此查詢會傳回兩個隱含引數變數，名稱為 `$1` 和 `$2`。

**查詢**

```sql
    SELECT { "state": f.address.state, "city": f.address.city }, 
           { "name": f.id }
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "$1": {
        "state": "WA", 
        "city": "seattle"
      }, 
      "$2": {
        "name": "AndersenFamily"
      }
    }]
```

## <a id="FromClause"></a>FROM 子句

除非稍後會在查詢中篩選或投射來源，否則 FROM <from_specification> 子句為選用子句。 若要深入了解語法，請參閱 [FROM 語法](sql-api-sql-query-reference.md#bk_from_clause)。 類似 `SELECT * FROM Families` 的查詢指出整個 Families 集合是要列舉的來源。 可使用特殊識別碼 ROOT 來代表此集合，而不使用集合名稱。 下列清單包含會針對每個查詢強制執行的規則：

* 集合可以進行別名處理，例如 `SELECT f.id FROM Families AS f` 或只是 `SELECT f.id FROM Families f`。 此處的 `f` 即等於 `Families`。 `AS` 是對識別碼進行別名處理的選用關鍵字。  

* 進行別名處理之後，就無法繫結原始來源。 例如， `SELECT Families.id FROM Families f` 在語句構造上無效，因為無法再解析識別碼 "Families"。  

* 所有需要參照的屬性都必須是完整的。 如果未遵循嚴謹的結構描述，則會強制執行以避免任何模糊的繫結。 因此，`SELECT id FROM Families f` 在語句構造上無效，因為不會繫結屬性 `id`。

### <a name="get-subdocuments-using-from-clause"></a>使用 FROM 子句取得子文件

來源也可以減少為更小的子集。 例如，若只要列舉每份文件中的樹狀子目錄，則子根目錄可能會變成來源 (如下列範例所示)：

**查詢**

```sql
    SELECT * 
    FROM Families.children
```

**結果**  

```json
    [
      [
        {
            "firstName": "Henriette Thaulow",
            "gender": "female",
            "grade": 5,
            "pets": [
              {
                  "givenName": "Fluffy"
              }
            ]
        }
      ],
      [
        {
            "familyName": "Merriam",
            "givenName": "Jesse",
            "gender": "female",
            "grade": 1
        },
        {
            "familyName": "Miller",
            "givenName": "Lisa",
            "gender": "female",
            "grade": 8
        }
      ]
    ]
```

雖然上述範例使用陣列作為來源，但物件也可用來作為來源，即下列範例中顯示的內容：可在來源中找到的任何有效的 JSON 值 (已定義的)，均會被視為查詢結果中的包含項目。 如果有些家族沒有 `address.state` 值，則會在查詢結果中予以排除。

**查詢**

```sql
    SELECT * 
    FROM Families.address.state
```

**結果**

```json
    [
      "WA", 
      "NY"
    ]
```

## <a id="WhereClause"></a>WHERE 子句
WHERE 子句 (**`WHERE <filter_condition>`**) 是選用的。 它會指定條件，而且來源所提供的 JSON 文件必須滿足這些條件才能併入為結果的一部分。 任何 JSON 文件都必須將指定的條件評估為 "true"，才能視為結果。 索引層使用 WHERE 子句，來判斷可為結果一部分的來源文件的絕對最小子集。 若要深入了解語法，請參閱 [WHERE 語法](sql-api-sql-query-reference.md#bk_where_clause)。

下列查詢會要求包含名稱屬性且其值為 `AndersenFamily`的文件。 沒有名稱屬性或值不符合 `AndersenFamily` 的任何其他文件都會予以排除。 

**查詢**

```sql
    SELECT f.address
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "address": {
        "state": "WA", 
        "county": "King", 
        "city": "seattle"
      }
    }]
```

前一個範例已顯示簡單的相等查詢。 SQL API 也支援各種純量運算式。 最常用的是二元和一元運算式。 來源 JSON 物件中的屬性參考也是有效的運算式。 

下列是目前支援的二元運算式，而且可以用於查詢中 (如下列範例所示)：  

|**運算子類型**  |**值**  |
|---------|---------|
|算術    |   +、-、*、/、%   |
|位元  |   |&、^、<<、>>、>>> (向右移位並填滿零)      |
|邏輯   |   AND、OR、NOT      |
|比較   |    =, !=, &lt;, &gt;, &lt;=, &gt;=, <>     |
|字串  |  || (串連)       |

讓我們了解一些使用二元運算子的查詢。

```sql
    SELECT * 
    FROM Families.children[0] c
    WHERE c.grade % 2 = 1     -- matching grades == 5, 1

    SELECT * 
    FROM Families.children[0] c
    WHERE c.grade ^ 4 = 1    -- matching grades == 5

    SELECT *
    FROM Families.children[0] c
    WHERE c.grade >= 5     -- matching grades == 5
```

一元運算子 +、-、~ 及 NOT 也是支援的運算子，可在查詢內使用 (如下列範例所示)：

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE NOT(c.grade = 5)  -- matching grades == 1

    SELECT *
    FROM Families.children[0] c
    WHERE (-c.grade = -5)  -- matching grades == 5
```

除了二元和一元運算子之外，還允許屬性參照。 例如，`SELECT * FROM Families f WHERE f.isRegistered` 所傳回的 JSON 文件包含屬性 `isRegistered` 且屬性值等於 JSON `true` 值。 任何其他值 (false、null、Undefined、`<number>`、`<string>`、`<object>`、`<array>` 等) 則會導致從結果中排除來源文件。 

### <a name="equality-and-comparison-operators"></a>相等和比較運算子
下表顯示 SQL API 中任何兩個 JSON 類型之間的相等比較結果。

<table style = "width:300px">
   <tbody>
      <tr>
         <td valign="top">
            <strong>運算子</strong>
         </td>
         <td valign="top">
            <strong>Undefined</strong>
         </td>
         <td valign="top">
            <strong>Null</strong>
         </td>
         <td valign="top">
            <strong>布林值</strong>
         </td>
         <td valign="top">
            <strong>數字</strong>
         </td>
         <td valign="top">
            <strong>字串</strong>
         </td>
         <td valign="top">
            <strong>物件</strong>
         </td>
         <td valign="top">
            <strong>陣列</strong>
         </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>Undefined<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>Null<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>布林值<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>數字<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>字串<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>物件<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
         <td valign="top">
Undefined </td>
      </tr>
      <tr>
         <td valign="top">
            <strong>陣列<strong>
         </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
Undefined </td>
         <td valign="top">
            <strong>正常</strong>
         </td>
      </tr>
   </tbody>
</table>

其他比較運算子 (例如 >、>=、!=、< 及 <=) 則適用下列規則：   

* 不同類型的比較會導致 Undefined。  
* 兩個物件或兩個陣列之間的比較會導致 Undefined。   

如果篩選中純量運算式的結果是 Undefined，則不會將對應的文件併入結果中，因為 Undefined 邏輯上不會等於 "true"。

## <a name="between-keyword"></a>BETWEEN 關鍵字
您也可以使用 BETWEEN 關鍵字來表示像對 ANSI SQL 中的值範圍執行查詢。 BETWEEN 可用於字串或數字。

例如，此查詢會傳回其長子的成績介於 1-5 (兩者皆含) 之間的所有家族文件。 

```sql
    SELECT *
    FROM Families.children[0] c
    WHERE c.grade BETWEEN 1 AND 5
```

不同於 ANSI SQL，您也可以在 FROM 子句中使用 BETWEEN 子句，如下列範例所示 。

```sql
    SELECT (c.grade BETWEEN 0 AND 10)
    FROM Families.children[0] c
```

在 SQL API 和 ANSI SQL 中使用 BETWEEN 的主要差異在於，您可以針對混合類型的屬性表示範圍查詢，例如，在某些文件中 "grade" 可能是數字 (5)，而在其他文件中可能會是字串 ("grade4")。 在這些情況下 (像在 JavaScript 中)，這兩種不同類型之間的比較會產生 「 未定義 」，並略過文件。

> [!NOTE]
> 如需更快速的查詢執行時間，請記得針對經過 BETWEEN 子句篩選的任何數值屬性/路徑，建立使用範圍索引類型的檢索原則。 

### <a name="logical-and-or-and-not-operators"></a>邏輯 (AND、OR 和 NOT) 運算子
邏輯運算子的運算對象是布林值。 下表顯示這些運算子的邏輯真值表。

**OR 運算子**

| 或 | True | False | Undefined |
| --- | --- | --- | --- |
| True |True |True |True |
| False |True |False |Undefined |
| Undefined |True |Undefined |Undefined |

**AND 運算子**

| AND | True | False | Undefined |
| --- | --- | --- | --- |
| True |True |False |Undefined |
| False |False |False |False |
| Undefined |Undefined |False |Undefined |

**NOT 運算子**

| NOT |  |
| --- | --- |
| True |False |
| False |True |
| Undefined |Undefined |

## <a name="in-keyword"></a>IN 關鍵字

IN 關鍵字可用來檢查指定的值是否符合清單中的任何值。 例如，此查詢會傳回所有的家族文件，其中 id 是 "WakefieldFamily" 或 "AndersenFamily" 其中一個。 

```sql
    SELECT *
    FROM Families 
    WHERE Families.id IN ('AndersenFamily', 'WakefieldFamily')
```

這個範例會傳回其狀態是任何一個指定值的所有文件。

```sql
    SELECT *
    FROM Families 
    WHERE Families.address.state IN ("NY", "WA", "CA", "PA", "OH", "OR", "MI", "WI", "MN", "FL")
```

## <a name="ternary--and-coalesce--operators"></a>三元 (?) 和聯合 (??) 運算子

三元和聯合運算子可用來建立條件運算式，與熱門程式設計語言 (如 C# 和 JavaScript) 類似。 快速建構新的 JSON 屬性時，三元 (?) 運算子可以說非常好用。 比方說，您現在可以撰寫查詢將類別層級分類為一般人可判讀的格式 (例如初級/中級/進階)，如下所示。

```sql
     SELECT (c.grade < 5)? "elementary": "other" AS gradeLevel 
     FROM Families.children[0] c
```

您也可以巢狀運算子的呼叫，如下方查詢所示。

```sql
    SELECT (c.grade < 5)? "elementary": ((c.grade < 9)? "junior": "high")  AS gradeLevel 
    FROM Families.children[0] c
```

和其他查詢運算子一樣，如果任何文件中的條件運算式遺漏了參考屬性，或者要比較的類型不同，則查詢結果會將那些文件排除在外。

聯合 (??) 運算子可用來有效率地檢查屬性是否存在 (也稱為 已定義) 於文件中。 針對半結構化或混合類型的資料執行查詢時，這非常有用。 例如，此查詢會傳回 "lastName" (如果存在) 或 "surname" (如果不存在)。

```sql
    SELECT f.lastName ?? f.surname AS familyName
    FROM Families f
```

## <a id="EscapingReservedKeywords"></a>加上引號的屬性存取子
您也可以使用加上引號的屬性運算子 `[]`存取屬性。 例如， `SELECT c.grade` and `SELECT c["grade"]` 是相等的。 當您需要逸出包含空格、特殊字元的屬性，或剛好要共用和 SQL 關鍵字或保留字相同的名稱時，此語法很有用。

```sql
    SELECT f["lastName"]
    FROM Families f
    WHERE f["id"] = "AndersenFamily"
```

## <a name="aliasing"></a>別名

現在，讓我們使用值的明確別名處理來擴充上面的範例。 AS 是用來加上別名的關鍵字。 它在將第二個值投射為 `NameInfo` 時是選用項目 (如下所示)。 

如果查詢具有兩個同名的屬性，則必須使用別名處理來重新命名一或兩個屬性，使其在投射的結果中變得更為明確。

**查詢**
```sql
    SELECT 
           { "state": f.address.state, "city": f.address.city } AS AddressInfo, 
           { "name": f.id } NameInfo
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
      "AddressInfo": {
        "state": "WA", 
        "city": "seattle"
      }, 
      "NameInfo": {
        "name": "AndersenFamily"
      }
    }]
```

## <a name="scalar-expressions"></a>純量運算式
除了屬性參考之外，SELECT 子句也支援純量運算式 (例如常數、算術運算式、邏輯運算式等)。例如，以下是簡單 "Hello World" 查詢。

**查詢**

```sql
    SELECT "Hello World"
```

**結果**

```json
    [{
      "$1": "Hello World"
    }]
```

以下是使用純量運算式的較複雜範例。

**查詢**

```sql
    SELECT ((2 + 11 % 7)-2)/3    
```

**結果**

```json
    [{
      "$1": 1.33333
    }]
```

在下列範例中，純量運算式結果是布林值。

**查詢**

```sql
    SELECT f.address.city = f.address.state AS AreFromSameCityState
    FROM Families f    
```

**結果**

```json
    [
      {
        "AreFromSameCityState": false
      }, 
      {
        "AreFromSameCityState": true
      }
    ]
```

## <a name="object-and-array-creation"></a>物件和陣列建立
SQL API 的另一個重要功能是建立陣列/物件。 在前一個範例中，請注意，我們已建立新的 JSON 物件。 同樣地，您也可以建構陣列 (如下列範例所示)：

**查詢**

```sql
    SELECT [f.address.city, f.address.state] AS CityState 
    FROM Families f    
```

**結果**  

```json
    [
      {
        "CityState": [
          "seattle", 
          "WA"
        ]
      }, 
      {
        "CityState": [
          "NY", 
          "NY"
        ]
      }
    ]
```

## <a id="ValueKeyword"></a>VALUE 關鍵字
**VALUE** 關鍵字提供可傳回 JSON 值的方法。 例如，下面顯示的查詢會傳回純量 `"Hello World"` 而非 `{$1: "Hello World"}`。

**查詢**

```sql
    SELECT VALUE "Hello World"
```

**結果**

```json
    [
      "Hello World"
    ]
```

下列查詢會在結果中傳回不含 `"address"` 標籤的 JSON 值。

**查詢**

```sql
    SELECT VALUE f.address
    FROM Families f    
```

**結果**  

```json
    [
      {
        "state": "WA", 
        "county": "King", 
        "city": "seattle"
      }, 
      {
        "state": "NY", 
        "county": "Manhattan", 
        "city": "NY"
      }
    ]
```

下列範例會延伸此程式碼，以示範如何傳回 JSON 基本值 (JSON 樹狀目錄的分葉層級)。 

**查詢**

```sql
    SELECT VALUE f.address.state
    FROM Families f    
```

**結果**

```json
    [
      "WA",
      "NY"
    ]
```

## <a name="-operator"></a>* 運算子
支援使用特殊運算子 (*) 來依原樣投射文件。 使用時，它必須是唯一投射的欄位。 `SELECT * FROM Families f` 之類的查詢有效，`SELECT VALUE * FROM Families f ` 和 `SELECT *, f.id FROM Families f ` 則無效。

**查詢**

```sql
    SELECT * 
    FROM Families f 
    WHERE f.id = "AndersenFamily"
```

**結果**

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

## <a id="TopKeyword"></a>TOP 運算子
TOP 關鍵字可以用來限制來自查詢的值數目。 當 TOP 與 ORDER BY 子句一起使用時，結果集會限制於前 N 個已排序的值。否則，它會以未定義的順序傳回前 N 個結果。 最佳做法是在 SELECT 陳述式中，一律搭配 TOP 子句來使用 ORDER BY 子句。 這是唯一能如預期般指出哪些資料列會受到 TOP 影響的方式。 

**查詢**

```sql
    SELECT TOP 1 * 
    FROM Families f 
```

**結果**

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

TOP 可以與常數值 (如上所示) 或使用參數化查詢的變數值搭配使用 。 如需詳細資訊，請參閱下方的參數化查詢。

## <a id="Aggregates"></a>彙總函式
您也可以在 `SELECT` 子句中執行彙總。 彙總函式會執行一組值的計算，然後傳回單一值。 例如，下列查詢會傳回集合內的系列文件計數。

**查詢**

```sql
    SELECT COUNT(1) 
    FROM Families f 
```

**結果**

```json
    [{
        "$1": 2
    }]
```

您也可以使用 `VALUE` 關鍵字來傳回彙總的純量值。 例如，下列查詢會以單一數字傳回值的計數：

**查詢**

```sql
    SELECT VALUE COUNT(1) 
    FROM Families f 
```

**結果**

```json
    [ 2 ]
```

您也可以搭配篩選器執行彙總。 例如，下列查詢會傳回位址在華盛頓州的文件計數。

**查詢**

```sql
    SELECT VALUE COUNT(1) 
    FROM Families f
    WHERE f.address.state = "WA" 
```

**結果**

```json
    [ 1 ]
```

下表顯示 SQL API 中支援的彙總函式清單。 `SUM` 和 `AVG` 是對數值執行，而 `COUNT`、`MIN`和 `MAX` 則可對數字、字串、布林值和 null 執行。 

| 使用量 | 說明 |
|-------|-------------|
| COUNT | 以運算式傳回項目的數目。 |
| SUM   | 以運算式傳回所有值的總和。 |
| 最小值   | 以運算式傳回最小值。 |
| 最大值   | 以運算式傳回最大值。 |
| 平均   | 以運算式傳回值的平均。 |

彙總也可以對陣列反覆運算的結果執行。 如需詳細資訊，請參閱[查詢中的陣列反覆運算](#Iteration)。

> [!NOTE]
> 使用 Azure 入口網站的 [資料總管] 時，請注意彙總查詢可能會對查詢頁面傳回部分彙總的結果。 SDK 會產生所有頁面的單一累計值。 
> 
> 若要使用程式碼執行彙總查詢，您需要 .NET SDK 1.12.0、.NET Core SDK 1.1.0 或 Java SDK 1.9.5 或更高版本。    
>

## <a id="OrderByClause"></a>ORDER BY 子句
像是在 ANSI SQL 中，您可以在查詢時包含選擇性的 Order By 子句。 子句可以包含選擇性 ASC/DESC 引數，利用它來指定擷取結果時必須依循的順序。

例如，以下是依據居住城市名稱的順序擷取家族的查詢。

**查詢**

```sql
    SELECT f.id, f.address.city
    FROM Families f 
    ORDER BY f.address.city
```

**結果**

```json
    [
      {
        "id": "WakefieldFamily",
        "city": "NY"
      },
      {
        "id": "AndersenFamily",
        "city": "Seattle"    
      }
    ]
```

以下是依據建立日期擷取家族的查詢，儲存為代表 epoch 時間的數字，也就是，自 1970 年 1 月 1 日之後經過的時間 (以秒為單位)。

**查詢**

```sql
    SELECT f.id, f.creationDate
    FROM Families f 
    ORDER BY f.creationDate DESC
```

**結果**

```json
    [
      {
        "id": "WakefieldFamily",
        "creationDate": 1431620462
      },
      {
        "id": "AndersenFamily",
        "creationDate": 1431620472    
      }
    ]
```

## <a id="Advanced"></a>進階資料庫概念和 SQL 查詢

### <a id="Iteration"></a>反覆運算
在 SQL API 中已透過 **IN** 關鍵字新增新的建構，可支援反覆運算 JSON 陣列。 FROM 來源支援反覆運算。 讓我們從下列範例開始：

**查詢**

```sql
    SELECT * 
    FROM Families.children
```

**結果**  

```json
    [
      [
        {
          "firstName": "Henriette Thaulow", 
          "gender": "female", 
          "grade": 5, 
          "pets": [{ "givenName": "Fluffy"}]
        }
      ], 
      [
        {
            "familyName": "Merriam", 
            "givenName": "Jesse", 
            "gender": "female", 
            "grade": 1
        }, 
        {
            "familyName": "Miller", 
            "givenName": "Lisa", 
            "gender": "female", 
            "grade": 8
        }
      ]
    ]
```

現在，讓我們查看另一個查詢，以執行反覆運算集合中的子系。 請注意輸出陣列中的差異。 此範例會分割 `children` ，並將結果簡維成單一陣列。  

**查詢**

```sql
    SELECT * 
    FROM c IN Families.children
```

**結果**  

```json
    [
      {
          "firstName": "Henriette Thaulow",
          "gender": "female",
          "grade": 5,
          "pets": [{ "givenName": "Fluffy" }]
      },
      {
          "familyName": "Merriam",
          "givenName": "Jesse",
          "gender": "female",
          "grade": 1
      },
      {
          "familyName": "Miller",
          "givenName": "Lisa",
          "gender": "female",
          "grade": 8
      }
    ]
```

這可以進一步用來篩選陣列的每個個別項目 (如下列範例所示)：

**查詢**

```sql
    SELECT c.givenName
    FROM c IN Families.children
    WHERE c.grade = 8
```

**結果**  

```json
    [{
      "givenName": "Lisa"
    }]
```

您也可以對陣列反覆運算的結果執行彙總。 例如，下列查詢會計算所有系列的子系數目。

**查詢**

```sql
    SELECT COUNT(child) 
    FROM child IN Families.children
```

**結果**  

```json
    [
      { 
        "$1": 3
      }
    ]
```

### <a id="Joins"></a>聯結
在關聯式資料庫中，跨資料表的聯結需求很重要。 就設計正規化結構描述而言，邏輯上需要它。 與此相反的是，SQL API 會處理無結構描述文件的反正規化資料模型。 這在邏輯上等同於「自我聯結」。

語言支援的語法是 <from_source1> JOIN <from_source2> JOIN ...JOIN <from_sourceN>。 整體而言，這會傳回一組 **N**-Tuple (具有 **N** 個值的 Tuple)。 每個 Tuple 所擁有的值，都是將所有集合別名在其個別集合上反覆運算所產生。 換句話說，這是參與聯結之集的完整交叉乘積。

下列範例示範 JOIN 子句的運作方式。 在下列範例中，結果是空的，因為來源中每個文件與空集合的交叉乘積是空的。

**查詢**

```sql
    SELECT f.id
    FROM Families f
    JOIN f.NonExistent
```

**結果**  

```json
    [{
    }]
```

在下列範例中，聯結會介於文件根目錄與 `children` 子根目錄之間。 它是兩個 JSON 物件之間的交叉乘積。 子系是陣列的事實不會影響 JOIN，因為我們是處理為子陣列的單一根目錄。 因此，結果只會包含兩個結果，因為每個含有陣列之文件的交叉乘積都只會產生一個文件。

**查詢**

```sql
    SELECT f.id
    FROM Families f
    JOIN f.children
```

**結果**

```json
    [
      {
        "id": "AndersenFamily"
      }, 
      {
        "id": "WakefieldFamily"
      }
    ]
```

下列範例示範較傳統的聯結：

**查詢**

```sql
    SELECT f.id
    FROM Families f
    JOIN c IN f.children 
```

**結果**

```json
    [
      {
        "id": "AndersenFamily"
      }, 
      {
        "id": "WakefieldFamily"
      }, 
      {
        "id": "WakefieldFamily"
      }
    ]
```

第一件要注意的事是 **JOIN** 子句的 `from_source` 是迭代器。 因此，此案例中的流程如下：  

* 展開陣列中的每個子項目 **c** 。
* 套用文件 **f** 的根目錄與第一個步驟中所簡維之每個子項目 **c** 的交叉乘積。
* 最後，單獨投射根物件 **f** 名稱屬性。 

第一份文件 (`AndersenFamily`) 只包含一個子項目，因此結果集只會包含與此文件相對應的單一物件。 第二份文件 (`WakefieldFamily`) 包含兩個子系。 因此，交叉乘積會產生每個子系的個別物件，進而導致兩個物件 (一個對應至此文件的子系一個)。 這兩份文件中的根欄位相同，就像您在交叉乘積中預期地一樣。

JOIN 的實際作用是透過圖形中很難投射的交叉乘積來形成 Tuple。 此外，如下面範例所示，您可以根據 Tuple 的組合進行篩選，讓使用者選擇全部 Tuple 都符合的條件。

**查詢**

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName 
    FROM Families f 
    JOIN c IN f.children 
    JOIN p IN c.pets
```

**結果**

```json
    [
      {
        "familyName": "AndersenFamily", 
        "childFirstName": "Henriette Thaulow", 
        "petName": "Fluffy"
      }, 
      {
        "familyName": "WakefieldFamily", 
        "childGivenName": "Jesse", 
        "petName": "Goofy"
      }, 
      {
       "familyName": "WakefieldFamily", 
       "childGivenName": "Jesse", 
       "petName": "Shadow"
      }
    ]
```

此範例是上述範例的自然延伸，用來執行雙重聯結。 因此，交叉乘積可被視為下列虛擬程式碼：

```
    for-each(Family f in Families)
    {    
        for-each(Child c in f.children)
        {
            for-each(Pet p in c.pets)
            {
                return (Tuple(f.id AS familyName, 
                  c.givenName AS childGivenName, 
                  c.firstName AS childFirstName,
                  p.givenName AS petName));
            }
        }
    }
```

`AndersenFamily` 有一個小孩養了一隻寵物。 因此，交叉乘積會從此家族產生 1 個資料列 (1\*1\*1)。 不過，WakefieldFamily 有兩個小孩，但只有一個小孩 "Jesse" 養了多隻寵物。 Jesse 有兩隻寵物。 因此，交叉乘積會從此家族產生 1\*1\*2 = 2 個資料列。

在下一個範例中，對 `pet`有一個額外的篩選。 這會排除寵物名稱不是 "Shadow" 的所有 Tuple。 請注意，我們可以從陣列建置 Tuple、根據 Tuple 的任何元素進行篩選，以及投射元素的任何組合。 

**查詢**

```sql
    SELECT 
        f.id AS familyName,
        c.givenName AS childGivenName,
        c.firstName AS childFirstName,
        p.givenName AS petName 
    FROM Families f 
    JOIN c IN f.children 
    JOIN p IN c.pets
    WHERE p.givenName = "Shadow"
```

**結果**

```json
    [
      {
       "familyName": "WakefieldFamily", 
       "childGivenName": "Jesse", 
       "petName": "Shadow"
      }
    ]
```

## <a id="JavaScriptIntegration"></a>JavaScript 整合
Azure Cosmos DB 提供一個程式設計模型，可藉由預存程序和觸發程序，直接對集合執行 JavaScript 型應用程式邏輯。 這允許：

* 藉由直接在資料庫引擎內深入整合 JavaScript 執行階段，對集合中的文件執行高效能交易式 CRUD 操作和查詢。 
* 將控制流程、變數範圍限制、指派以及例外狀況處理基本類型與資料庫交易的整合自然模型化。 如需有關 JavaScript 整合之 Azure Cosmos DB 支援的詳細資料，請參閱 JavaScript 伺服器端程式設計文件。

### <a id="UserDefinedFunctions"></a>使用者定義函式 (UDF)
除了本文中已定義的類型之外，SQL API 還支援使用者定義函式 (UDF)。 特別的是，如果開發人員可以傳入零個或多個引數並傳回單一引數結果，則支援純量 UDF。 系統會檢查所有這些引數是否為合法的 JSON 值。  

SQL 語法已延伸，可支援使用這些使用者定義函式的自訂應用程式邏輯。 您可以向 SQL API 註冊 UDF，然後在 SQL 查詢中參照它。 實際上，UDF 是特別設計來透過查詢進行叫用。 這項選擇的必然結果，就是 UDF 無法存取其他 JavaScript 類型 (預存程序和觸發程序) 擁有的內容物件。 因為查詢是以唯讀形式執行，所以可以在主要或次要複本上執行。 因此，與其他 JavaScript 類型不同，UDF 是設計成在次要複本上執行。

以下是如何在 Cosmos DB 資料庫 (更明確地說是在文件集合下) 註冊 UDF 的範例。

```javascript
       UserDefinedFunction regexMatchUdf = new UserDefinedFunction
       {
           Id = "REGEX_MATCH",
           Body = @"function (input, pattern) { 
                       return input.match(pattern) !== null;
                   };",
       };

       UserDefinedFunction createdUdf = client.CreateUserDefinedFunctionAsync(
           UriFactory.CreateDocumentCollectionUri("testdb", "families"), 
           regexMatchUdf).Result;  
```

上述範例會建立名為 `REGEX_MATCH`的 UDF。 它接受兩個 JSON 字串值 `input` and `pattern` ，並使用 JavaScript 的 string.match() 函式來檢查第一個項目是否符合第二個項目中指定的模式。

我們現在可以在投射的查詢中使用此 UDF。 必須以區分大小寫的前置詞 "udf." 來限定 UDF (當從查詢內呼叫時)。 

> [!NOTE]
> 在 2015 年 3 月 17 日以前，Cosmos DB 支援無 "udf." 前置詞的 UDF 呼叫， 例如 SELECT REGEX_MATCH()。 這種呼叫模式已被取代。  
> 
> 

**查詢**

```sql
    SELECT udf.REGEX_MATCH(Families.address.city, ".*eattle")
    FROM Families
```

**結果**

```json
    [
      {
        "$1": true
      }, 
      {
        "$1": false
      }
    ]
```

UDF 也可以用於篩選內 (如下面範例所示)，同樣是以 "udf." 前置詞來限定：

**查詢**

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE udf.REGEX_MATCH(Families.address.city, ".*eattle")
```

**結果**

```json
    [{
        "id": "AndersenFamily",
        "city": "Seattle"
    }]
```

在本質上，UDF 是有效的純量運算式，而且可以用於投射和篩選。 

為了擴展 UDF 的能力，請查看另一個含有條件式邏輯的範例：

```javascript
       UserDefinedFunction seaLevelUdf = new UserDefinedFunction()
       {
           Id = "SEALEVEL",
           Body = @"function(city) {
                   switch (city) {
                       case 'seattle':
                           return 520;
                       case 'NY':
                           return 410;
                       case 'Chicago':
                           return 673;
                       default:
                           return -1;
                    }"
            };

            UserDefinedFunction createdUdf = await client.CreateUserDefinedFunctionAsync(
                UriFactory.CreateDocumentCollectionUri("testdb", "families"), 
                seaLevelUdf);
```

以下是執行 UDF 的範例。

**查詢**

```sql
    SELECT f.address.city, udf.SEALEVEL(f.address.city) AS seaLevel
    FROM Families f    
```

**結果**

```json
     [
      {
        "city": "seattle", 
        "seaLevel": 520
      }, 
      {
        "city": "NY", 
        "seaLevel": 410
      }
    ]
```

如上述範例所示，UDF 可將 JavaScript 語言的強大功能與 SQL API 整合，提供可程式化的豐富介面，從而在內建 JavaScript 執行階段功能的協助下，執行複雜的程序性、條件式邏輯。

在處理 UDF 的目前階段 (WHERE 子句或 SELECT 子句) 中，SQL API 會針對來源中的每個文件提供 UDF 的引數。 結果會緊密整合至整體執行管線。 如果 JSON 值中沒有 UDF 參數所參照的屬性，則會將參數視為 undefined，因此，而整個略過 UDF 叫用。 同樣地，如果 UDF 的結果為未定義，便不會被納入結果中。 

簡言之，UDF 是在進行查詢時執行複雜商務邏輯的不錯工具。

### <a name="operator-evaluation"></a>運算子評估
Cosmos DB 藉由作為 JSON 資料庫，可將 JavaScript 運算子與其評估語意做比較。 雖然 Cosmos DB 嘗試以 JSON 支援的形式保留 JavaScript 語意，但是在部分執行個體中，作業評估還是會偏離。

不同於傳統 SQL，在 SQL API 中，通常要等到從資料庫擷取值時，才會知道值的類型。 為了有效率地執行查詢，大部分的運算子都有嚴謹的類型需求。 

不同於 JavaScript，SQL API 不會執行隱含轉換。 例如，類似 `SELECT * FROM Person p WHERE p.Age = 21` 的查詢會針對包含 Age 屬性且值為 21 的文件進行比對。 任何其他 Age 屬性符合字串 "21" 或其他可能之無限制變化 (例如 "021"、"21.0"、"0021"、"00021" 等等) 的文件，則不會成為比對的結果。 這與 JavaScript 形成對比，在 JavaScript 中，會將字串值隱含地轉換成數字 (根據運算子，例如：==)。 這項選擇對 SQL API 中有效率的索引比對十分重要。 

## <a name="parameterized-sql-queries"></a>參數化 SQL 查詢
Cosmos DB 支援以含有常見 \@ 標記法表示的參數進行查詢。 參數化 SQL 提供使用者輸入的健全處理和逸出，防止透過 SQL 插入式意外洩露資料。 

比方說，您可以撰寫查詢，將姓氏和地址 (州) 做為參數，然後根據使用者輸入，使用各種不同姓氏和地址 (州) 的值執行查詢。

```sql
    SELECT * 
    FROM Families f
    WHERE f.lastName = @lastName AND f.address.state = @addressState
```

您可以接著將此要求傳送給 Cosmos DB 作為參數化 JSON 查詢，如下所示。

```sql
    {      
        "query": "SELECT * FROM Families f WHERE f.lastName = @lastName AND f.address.state = @addressState",     
        "parameters": [          
            {"name": "@lastName", "value": "Wakefield"},         
            {"name": "@addressState", "value": "NY"},           
        ] 
    }
```

您可以使用參數化查詢來設定 TOP 的引數，如下所示。

```sql
    {      
        "query": "SELECT TOP @n * FROM Families",     
        "parameters": [          
            {"name": "@n", "value": 10},         
        ] 
    }
```

參數值可以是任何有效的 JSON (字串、數字、布林值、null，甚至是陣列或巢狀 JSON)。 此外，由於 Cosmos DB 並無結構描述，因此系統不會對照任何類型來驗證參數。

## <a id="BuiltinFunctions"></a>內建函數
Cosmos DB 也支援一些適用於一般作業的內建函式，這些函式可在像是使用者定義函式 (UDF) 之類的查詢內使用。

| 函式群組          | 作業                                                                                                                                          |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| 數學函數  | ABS、CEILING、EXP、FLOOR、LOG、LOG10、POWER、ROUND、SIGN、SQRT、SQUARE、TRUNC、ACOS、ASIN、ATAN、ATN2、COS、COT、DEGREES、PI、RADIANS、SIN 和 TAN |
| 類型檢查函數 | IS_ARRAY、IS_BOOL、IS_NULL、IS_NUMBER、IS_OBJECT、IS_STRING、IS_DEFINED 和 IS_PRIMITIVE                                                           |
| 字串函數        | CONCAT、CONTAINS、ENDSWITH、INDEX_OF、LEFT、LENGTH、LOWER、LTRIM、REPLACE、REPLICATE、REVERSE、RIGHT、RTRIM、STARTSWITH、SUBSTRING 和 UPPER       |
| 陣列函數         | ARRAY_CONCAT、ARRAY_CONTAINS、ARRAY_LENGTH 和 ARRAY_SLICE                                                                                         |
| 空間函數       | ST_DISTANCE、ST_WITHIN、ST_INTERSECTS、ST_ISVALID 和 ST_ISVALIDDETAILED                                                                           | 

如果您目前使用的使用者定義函式 (UDF) 已提供內建函式，則應該使用相對應的內建函式，因為這樣會加快執行速度且更有效率。 

### <a name="mathematical-functions"></a>數學函數
每個數學函式都會執行計算，通常以提供來作為引數的輸入值為基礎，並傳回數值。 以下是支援的內建數學函數資料表。


| 使用量 | 說明 |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [[ABS (num_expr)](#bk_abs) | 傳回指定之數值運算式的絕對 (正) 值。 |
| [CEILING (num_expr)](#bk_ceiling) | 傳回大於或等於指定之數值運算式的最小整數值。 |
| [FLOOR (num_expr)](#bk_floor) | 傳回小於或等於指定之數值運算式的最大整數。 |
| [EXP (num_expr)](#bk_exp) | 傳回指定之數值運算式的指數。 |
| [LOG (num_expr [,base])](#bk_log) | 傳回指定之數值運算式的自然對數，或使用指定之基底的對數 |
| [LOG10 (num_expr)](#bk_log10) | 傳回指定之數值運算式的以 10 為基底的對數值。 |
| [ROUND (num_expr)](#bk_round) | 傳回數值，四捨五入到最接近的整數值。 |
| [TRUNC (num_expr)](#bk_trunc) | 傳回數值，截斷至最接近的整數值。 |
| [SQRT (num_expr)](#bk_sqrt) | 傳回指定之數值運算式的平方根。 |
| [SQUARE (num_expr)](#bk_square) | 傳回指定之數值運算式的平方。 |
| [POWER (num_expr, num_expr)](#bk_power) | 將指定數值運算式的乘冪傳回給指定的值。 |
| [SIGN (num_expr)](#bk_sign) | 傳回指定之數值運算式的正負號值 (-1、0、1)。 |
| [ACOS (num_expr)](#bk_acos) | 傳回角度，以弧度為單位，它的餘弦是指定的數值運算式；也稱為反餘弦值。 |
| [ASIN (num_expr)](#bk_asin) | 傳回角度，以弧度為單位，其正弦函數是指定的數值運算式。 這也稱為反正弦值。 |
| [ATAN (num_expr)](#bk_atan) | 傳回角度，以弧度為單位，其正切函數是指定的數值運算式。 這也稱為反正切值。 |
| [ATN2 (num_expr)](#bk_atn2) | 傳回角度，以弧度為單位，正 x 軸和從原點 (y、x) 點的切線之間，其中 x 和 y 是兩個指定之浮點運算式的值。 |
| [COS (num_expr)](#bk_cos) | 在指定運算式中傳回指定角度的三角餘弦函數，以弧度為單位。 |
| [COT (num_expr)](#bk_cot) | 在指定的數值運算式中傳回指定角度的三角餘切函數，以弧度為單位。 |
| [DEGREES (num_expr)](#bk_degrees) | 針對以弧度指定的角度，傳回對應的角度 (以度為單位)。 |
| [PI ()](#bk_pi) | 傳回 PI 的常數值。 |
| [RADIANS (num_expr)](#bk_radians) | 當您輸入數值運算式 (以度為單位) 時，傳回弧度。 |
| [SIN (num_expr)](#bk_sin) | 在指定運算式中傳回指定角度的三角正弦函數 (以弧度為單位)。 |
| [TAN (num_expr)](#bk_tan) | 在指定運算式中傳回輸入運算式的正切函數。 |

例如，您現在可以執行下列的查詢：

**查詢**

```sql
    SELECT VALUE ABS(-4)
```

**結果**

```json
    [4]
```
相較於 ANSI SQL，Cosmos DB 函數的主要差異在於，其設計目的是要與無結構描述資料及混合的結構描述資料有效搭配運作。 例如，如果您有一個文件的 [大小] 屬性已遺失，或具有非數值的值 (例如 "unknown")，該文件就會被略過而不會傳回錯誤。

### <a name="type-checking-functions"></a>類型檢查函數
類型檢查函數可讓您檢查 SQL 查詢中的運算式類型。 類型檢查函數可在屬性類型為變數或未知時，用來快速判斷文件中的屬性類型。 以下是支援的內建類型檢查函數資料表。

<table>
<tr>
  <td><strong>使用量</strong></td>
  <td><strong>說明</strong></td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_array">IS_ARRAY (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為陣列。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_bool">IS_BOOL (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為布林值。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_null">IS_NULL (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為 null。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_number">IS_NUMBER (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為數字。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_object">IS_OBJECT (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為 JSON 物件。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_string">IS_STRING (expr)</a></td>
  <td>傳回布林值，表示值的類型是否為字串。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_defined">IS_DEFINED (expr)</a></td>
  <td>傳回布林值，表示屬性是否已經指派值。</td>
</tr>
<tr>
  <td><a href="https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_is_primitive">IS_PRIMITIVE (expr)</a></td>
  <td>傳回布林值，表示值的類型是字串、數字、布林值或 Null。</td>
</tr>

</table>

藉由使用這些函數，您現在可以執行下列查詢：

**查詢**

```sql
    SELECT VALUE IS_NUMBER(-4)
```

**結果**

```json
    [true]
```

### <a name="string-functions"></a>字串函數
下列純量函數會對字串輸入值執行作業，並傳回字串、數值或布林值。 以下是內建字串函數的資料表：

| 使用量 | 說明 |
| --- | --- |
| [LENGTH (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_length) |傳回指定字串運算式的字元數目 |
| [CONCAT (str_expr, str_expr [, str_expr])](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_concat) |傳回字串，該字串是串連兩個或多個字串值的結果。 |
| [SUBSTRING (str_expr, num_expr, num_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_substring) |傳回字串運算式的一部分。 |
| [STARTSWITH (str_expr, str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_startswith) |傳回布林值，表示第一個字串運算式是否以第二個字串運算式開頭 |
| [ENDSWITH (str_expr, str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_endswith) |傳回布林值，表示第一個字串運算式是否以第二個結束字串運算式做為結束 |
| [CONTAINS (str_expr, str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_contains) |傳回布林值，表示第一個字串運算式是否包含第二個字串運算式。 |
| [INDEX_OF (str_expr, str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_index_of) |傳回第一個指定的字串運算式中，第二個字串運算式第一次出現的開始位置，或者如果找不到字串，則為 -1。 |
| [LEFT (str_expr, num_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_left) |傳回具有指定字元數目的字串左側部分。 |
| [RIGHT (str_expr, num_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_right) |傳回具有指定字元數目的字串右側部分。 |
| [LTRIM (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_ltrim) |傳回移除開頭空白之後的字串運算式。 |
| [RTRIM (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_rtrim) |傳回截斷所有結尾空白之後的字串運算式。 |
| [LOWER (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_lower) |傳回將大寫字元資料轉換成小寫之後的字串運算式。 |
| [UPPER (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_upper) |傳回將小寫字元資料轉換成大寫之後的字串運算式。 |
| [REPLACE (str_expr, str_expr, str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_replace) |使用其他字串值取代指定的字串值的所有項目。 |
| [REPLICATE (str_expr, num_expr)](https://docs.microsoft.com/azure/cosmos-db/sql-api-sql-query-reference#bk_replicate) |將字串值重複指定的次數。 |
| [REVERSE (str_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_reverse) |傳回反向順序的字串值。 |

藉由使用這些函數，您現在可以執行下列查詢。 例如，您可以傳回大寫的家族名稱，如下所示：

**查詢**

```sql
    SELECT VALUE UPPER(Families.id)
    FROM Families
```

**結果**

```json
    [
        "WAKEFIELDFAMILY", 
        "ANDERSENFAMILY"
    ]
```

或串連字串，如下列範例所示：

**查詢**

```sql
    SELECT Families.id, CONCAT(Families.address.city, ",", Families.address.state) AS location
    FROM Families
```

**結果**

```json
    [{
      "id": "WakefieldFamily",
      "location": "NY,NY"
    },
    {
      "id": "AndersenFamily",
      "location": "seattle,WA"
    }]
```

字串函數也可用於 WHERE 子句中以篩選結果，例如下列範例：

**查詢**

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE STARTSWITH(Families.id, "Wakefield")
```

**結果**

```json
    [{
      "id": "WakefieldFamily",
      "city": "NY"
    }]
```

### <a name="array-functions"></a>陣列函數
下列純量函數會對陣列輸入值執行作業，並傳回數值、布林值或陣列值。 以下是內建陣列函數的資料表：

| 使用量 | 說明 |
| --- | --- |
| [ARRAY_LENGTH (arr_expr)](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_array_length) |傳回指定陣列運算式的元素數目。 |
| [ARRAY_CONCAT (arr_expr, arr_expr [, arr_expr])](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_array_concat) |傳回串連兩個或多個陣列值之結果的陣列。 |
| [ARRAY_CONTAINS (arr_expr, expr [, bool_expr])](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_array_contains) |傳回布林值，表示陣列是否包含指定值。 可以指定要進行完整或部分比對。 |
| [ARRAY_SLICE (arr_expr, num_expr [, num_expr])](https://msdn.microsoft.com/library/azure/dn782250.aspx#bk_array_slice) |傳回陣列運算式的一部分。 |

陣列函數可用來管理 JSON 中的陣列。 例如，以下是傳回其中一位父母是 "Robin Wakefield" 之所有文件的查詢。 

**查詢**

```sql
    SELECT Families.id 
    FROM Families 
    WHERE ARRAY_CONTAINS(Families.parents, { givenName: "Robin", familyName: "Wakefield" })
```

**結果**

```json
    [{
      "id": "WakefieldFamily"
    }]
```

您可以指定部分片段來比對陣列內的元素。 下列查詢會尋找 `givenName` 為 `Robin` 的所有父母。

**查詢**

```sql
    SELECT Families.id 
    FROM Families 
    WHERE ARRAY_CONTAINS(Families.parents, { givenName: "Robin" }, true)
```

**結果**

```json
    [{
      "id": "WakefieldFamily"
    }]
```

以下是另一個例子，會使用 ARRAY_LENGTH 取得每個家族的子女數目。

**查詢**

```sql
    SELECT Families.id, ARRAY_LENGTH(Families.children) AS numberOfChildren
    FROM Families 
```

**結果**

```json
    [{
      "id": "WakefieldFamily",
      "numberOfChildren": 2
    },
    {
      "id": "AndersenFamily",
      "numberOfChildren": 1
    }]
```

### <a name="spatial-functions"></a>空間函數
Cosmos DB 支援下列「開放地理空間協會」(OGC) 內建地理空間查詢函數。 

<table>
<tr>
  <td><strong>使用量</strong></td>
  <td><strong>說明</strong></td>
</tr>
<tr>
  <td>ST_DISTANCE (point_expr、point_expr)</td>
  <td>傳回兩個 GeoJSON Point、Polygon 或 LineString 運算式之間的距離。</td>
</tr>
<tr>
  <td>ST_WITHIN (point_expr、polygon_expr)</td>
  <td>傳回布林運算式，指出第一個 GeoJSON 物件 (Point、Polygon 或 LineString) 是否位在第二個 GeoJSON 物件 (Point、Polygon 或 LineString) 內。</td>
</tr>
<tr>
  <td>ST_INTERSECTS (spatial_expr, spatial_expr)</td>
  <td>傳回布林運算式，指出兩個指定的 GeoJSON 物件 (Point、Polygon 或 LineString) 是否相交。</td>
</tr>
<tr>
  <td>ST_ISVALID</td>
  <td>傳回布林值，指出指定的 GeoJSON Point、Polygon 或 LineString 運算式是否有效。</td>
</tr>
<tr>
  <td>ST_ISVALIDDETAILED</td>
  <td>如果指定的 GeoJSON Point、Polygon 或 LineString 運算式有效，就傳回包含布林值的 JSON 值；但如果是無效的，就會額外加上做為字串值的原因。</td>
</tr>
</table>

空間函數可以用來對空間資料執行鄰近性查詢。 例如，以下查詢使用 ST_DISTANCE 內建函數傳回所有家族文件，且這些文件在 30 公里指定位置內。 

**查詢**

```sql
    SELECT f.id 
    FROM Families f 
    WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000
```

**結果**

```json
    [{
      "id": "WakefieldFamily"
    }]
```

如需有關 Cosmos DB 中的地理空間支援詳細資料，請參閱[使用 Azure Cosmos DB 中的地理空間資料](geospatial.md)。 其中提供 Cosmos DB 之空間函數及 SQL 語法的概要。 現在讓我們看看 LINQ 查詢如何運作，以及它如何與我們到目前為止所見到的語法互動。

## <a id="Linq"></a>LINQ to SQL API
LINQ 是一種 .NET 程式設計模型，此模型會將運算表示為對物件串流的查詢。 Cosmos DB 提供一個用戶端程式庫，可透過協助 JSON 與 .NET 物件之間的轉換，以及 LINQ 查詢子集與 Cosmos DB 查詢的對應，來與 LINQ 互動。 

下圖顯示使用 Cosmos DB 來支援 LINQ 查詢的架構。  開發人員可以使用 Cosmos DB 用戶端來建立 **IQueryable** 物件，以直接查詢 Cosmos DB 查詢提供者，而此查詢提供者會接著將 LINQ 查詢轉譯成 Cosmos DB 查詢。 然後，將此查詢傳遞給 Cosmos DB 伺服器，以擷取一組以 JSON 格式表示的結果。 傳回的結果會在用戶端進行還原序列化，變成 .NET 物件的串流。

![使用 SQL API 來支援 LINQ 查詢的架構：SQL 語法、JSON 查詢語言、資料庫概念及 SQL 查詢][1]

### <a name="net-and-json-mapping"></a>.NET 和 JSON 對應
.NET 物件與 JSON 文件之間的對應是自然的，亦即每個資料成員欄位都會對應至 JSON 物件，其中欄位名稱對應至物件的「索引鍵」部分，而「值」部分則遞迴地對應至物件的值部分。 請考量下列範例：建立的 Family 物件會對應至 JSON 文件 (如下所示)。 而反之亦然，JSON 文件會對應回 .NET 物件。

**C# 類別**

```csharp
    public class Family
    {
        [JsonProperty(PropertyName="id")]
        public string Id;
        public Parent[] parents;
        public Child[] children;
        public bool isRegistered;
    };

    public struct Parent
    {
        public string familyName;
        public string givenName;
    };

    public class Child
    {
        public string familyName;
        public string givenName;
        public string gender;
        public int grade;
        public List<Pet> pets;
    };

    public class Pet
    {
        public string givenName;
    };

    public class Address
    {
        public string state;
        public string county;
        public string city;
    };

    // Create a Family object.
    Parent mother = new Parent { familyName= "Wakefield", givenName="Robin" };
    Parent father = new Parent { familyName = "Miller", givenName = "Ben" };
    Child child = new Child { familyName="Merriam", givenName="Jesse", gender="female", grade=1 };
    Pet pet = new Pet { givenName = "Fluffy" };
    Address address = new Address { state = "NY", county = "Manhattan", city = "NY" };
    Family family = new Family { Id = "WakefieldFamily", parents = new Parent [] { mother, father}, children = new Child[] { child }, isRegistered = false };
```

**JSON**  

```json
    {
        "id": "WakefieldFamily",
        "parents": [
            { "familyName": "Wakefield", "givenName": "Robin" },
            { "familyName": "Miller", "givenName": "Ben" }
        ],
        "children": [
            {
                "familyName": "Merriam", 
                "givenName": "Jesse", 
                "gender": "female", 
                "grade": 1,
                "pets": [
                    { "givenName": "Goofy" },
                    { "givenName": "Shadow" }
                ]
            },
            { 
              "familyName": "Miller", 
              "givenName": "Lisa", 
              "gender": "female", 
              "grade": 8 
            }
        ],
        "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
        "isRegistered": false
    };
```


### <a name="linq-to-sql-translation"></a>LINQ 至 SQL 轉譯
Cosmos DB 查詢提供者會盡力將 LINQ 查詢對應至 Cosmos DB SQL 查詢。 在下列描述中，我們假設讀者對 LINQ 有基本的認識。

首先，針對類型系統，我們支援所有 JSON 基本類型：數值類型、布林值、字串及 null。 只支援這些 JSON 類型。 下列是支援的純量運算式。

* 常數值：這些包括評估查詢時之基本資料類型的常數值。
* 屬性/陣列索引運算式：這些運算式參照物件或陣列項目的屬性。
  
     family.Id;    family.children[0].familyName;    family.children[0].grade;    family.children[n].grade; //n is an int variable
* 算術運算式：這些包括數值和布林值的一般算術運算式。 如需完整清單，請參閱 SQL 規格。
  
     2 * family.children[0].grade;    x + y;
* 字串比較運算式：這些包括比較字串值與某個常數字串值。  
  
     mother.familyName == "Smith";    child.givenName == s; //s is a string variable
* 物件/陣列建立運算式：這些運算式傳回複合值類型或匿名類型的物件，或是這類物件的陣列。 這些值可以是巢狀值。
  
     new Parent { familyName = "Smith", givenName = "Joe" }; new { first = 1, second = 2 }; //含有兩個欄位的匿名類型              
     new int[] { 3, child.grade, 5 };

### <a id="SupportedLinqOperators"></a>支援的 LINQ 運算子清單
以下是隨附於 SQL .NET SDK 之 LINQ 提供者中支援的 LINQ 運算子清單。

* **Select**：投影會轉譯為包括物件建構的 SQL SELECT
* **Where**：篩選會轉譯為 SQL WHERE，並支援 &&、|| 及 ! 之間的轉譯 到 SQL 運算子之間的轉譯
* **SelectMany**：可讓陣列回溯到 SQL JOIN 子句。 可用來鏈結/巢串運算式來篩選陣列元素
* **OrderBy 和 OrderByDescending**：轉譯為 ORDER BY 遞增/遞減順序
* 彙總的 **Count**、**Sum**、**Min**、**Max** 與 **Average** 運算子，以及其非同步對應項 **CountAsync**、**SumAsync**、**MinAsync**、**MaxAsync** 與 **AverageAsync**。
* **CompareTo**：轉譯為範圍比較。 通常用於字串，因為字串在 .NET 中是無法比較的
* **Take**：轉譯為 SQL TOP，以限制查詢的結果
* **數學函數**：支援從 .NET 的 Abs、Acos、Asin、Atan、Ceiling、Cos、Exp、Floor、Log、Log10、Pow、Round、Sign、Sin、Sqrt、Tan、Truncate 轉譯為對等的 SQL 內建函式。
* **字串函數**：支援從 .NET 的 Concat、Contains、EndsWith、IndexOf、Count、ToLower、TrimStart、Replace、Reverse、TrimEnd、StartsWith、SubString、ToUpper 轉譯為對等的 SQL 內建函式。
* **陣列函數**：支援從 .NET 的 Concat、Contains 和 Count 轉譯為對等的 SQL 內建函式。
* **地理空間擴充函數**：支援從虛設常式方法 Distance、Within、IsValid 及 IsValidDetailed 轉譯為對等的 SQL 內建函式。
* **使用者定義函式的擴充函式**：支援從 Stub 方法 UserDefinedFunctionProvider.Invoke 轉譯為對應的使用者定義函式。
* **其他**：支援聯合和條件式運算子的轉譯。 可根據內容將 Contains 轉譯為字串 CONTAINS、ARRAY_CONTAINS 或 SQL IN。

### <a name="sql-query-operators"></a>SQL 查詢運算子
以下是一些範例，說明如何將一些標準 LINQ 查詢運算子一路轉譯成 Cosmos DB 查詢。

#### <a name="select-operator"></a>Select 運算子
語法為 `input.Select(x => f(x))`，其中 `f` 是純量運算式。

**LINQ Lambda 運算式**

    input.Select(family => family.parents[0].familyName);

**SQL** 

```sql
    SELECT VALUE f.parents[0].familyName
    FROM Families f
```

**LINQ Lambda 運算式**

    input.Select(family => family.children[0].grade + c); // c is an int variable


**SQL** 

```sql
    SELECT VALUE f.children[0].grade + c
    FROM Families f 
```


**LINQ Lambda 運算式**

    input.Select(family => new
    {
        name = family.children[0].familyName,
        grade = family.children[0].grade + 3
    });


**SQL** 

```sql
    SELECT VALUE {"name":f.children[0].familyName, 
                  "grade": f.children[0].grade + 3 }
    FROM Families f
```


#### <a name="selectmany-operator"></a>SelectMany 運算子
語法為 `input.SelectMany(x => f(x))`，其中 `f` 是傳回集合類型的純量運算式。

**LINQ Lambda 運算式**

    input.SelectMany(family => family.children);

**SQL** 

```sql
    SELECT VALUE child
    FROM child IN Families.children
```

#### <a name="where-operator"></a>Where 運算子
語法為 `input.Where(x => f(x))`，其中 `f` 是傳回布林值的純量運算式。

**LINQ Lambda 運算式**

    input.Where(family=> family.parents[0].familyName == "Smith");

**SQL** 

```sql
    SELECT *
    FROM Families f
    WHERE f.parents[0].familyName = "Smith" 
```

**LINQ Lambda 運算式**

    input.Where(
        family => family.parents[0].familyName == "Smith" && 
        family.children[0].grade < 3);

**SQL** 

```sql
    SELECT *
    FROM Families f
    WHERE f.parents[0].familyName = "Smith"
    AND f.children[0].grade < 3
```

### <a name="composite-sql-queries"></a>複合 SQL 查詢
您可以包含上述的運算子，以形成功能更強大的查詢。 由於 Cosmos DB 支援巢狀集合，因此這個組合可以採用串連或巢狀的形式。

#### <a name="concatenation"></a>串連
語法為 `input(.|.SelectMany())(.Select()|.Where())*`。 串連查詢的開頭可以是選用的 `SelectMany` 查詢，其後接著多個 `Select` 或 `Where` 運算子。

**LINQ Lambda 運算式**

    input.Select(family=>family.parents[0])
        .Where(familyName == "Smith");

**SQL**

```sql
    SELECT *
    FROM Families f
    WHERE f.parents[0].familyName = "Smith"
```

**LINQ Lambda 運算式**

    input.Where(family => family.children[0].grade > 3)
        .Select(family => family.parents[0].familyName);

**SQL** 

```sql
    SELECT VALUE f.parents[0].familyName
    FROM Families f
    WHERE f.children[0].grade > 3
```


**LINQ Lambda 運算式**

    input.Select(family => new { grade=family.children[0].grade}).
        Where(anon=> anon.grade < 3);

**SQL** 

```sql
    SELECT *
    FROM Families f
    WHERE ({grade: f.children[0].grade}.grade > 3)
```

**LINQ Lambda 運算式**

    input.SelectMany(family => family.parents)
        .Where(parent => parents.familyName == "Smith");

**SQL** 

```sql
    SELECT *
    FROM p IN Families.parents
    WHERE p.familyName = "Smith"
```


#### <a name="nesting"></a>巢狀
語法為 `input.SelectMany(x=>x.Q())`，其中 Q 是 `Select`、`SelectMany` 或 `Where` 運算子。

在巢狀查詢中，會將內部查詢套用至外部集合的每個項目。 其中一個重要功能是內部查詢可以參照外部集合中項目的欄位 (例如自我聯結)。

**LINQ Lambda 運算式**

    input.SelectMany(family=> 
        family.parents.Select(p => p.familyName));

**SQL** 

```sql
    SELECT VALUE p.familyName
    FROM Families f
    JOIN p IN f.parents
```

**LINQ Lambda 運算式**

    input.SelectMany(family => 
        family.children.Where(child => child.familyName == "Jeff"));

**SQL** 

```sql
    SELECT *
    FROM Families f
    JOIN c IN f.children
    WHERE c.familyName = "Jeff"
```


**LINQ Lambda 運算式**

    input.SelectMany(family => family.children.Where(
        child => child.familyName == family.parents[0].familyName));

**SQL** 

```sql
    SELECT *
    FROM Families f
    JOIN c IN f.children
    WHERE c.familyName = f.parents[0].familyName
```

## <a id="ExecutingSqlQueries"></a>執行 SQL 查詢
Cosmos DB 公開資源的方式是透過 REST API，任何能夠發出 HTTP/HTTPS 要求的語言都可呼叫此 API。 此外，Cosmos DB 還為數種熱門語言 (例如 .NET、Node.js、JavaScript 和 Python) 提供程式設計程式庫。 REST API 和各種程式庫都支援透過 SQL 進行查詢。 .NET SDK 除了 SQL 之外，也支援 LINQ 查詢。

下列範例顯示如何建立查詢，並針對 Cosmos DB 資料庫帳戶提交它。

### <a id="RestAPI"></a>REST API
Cosmos DB 提供透過 HTTP 的開放 RESTful 程式設計模型。 可以使用 Azure 訂用帳戶佈建資料庫帳戶。 Cosmos DB 資源模型包含某個資料庫帳戶下的一組資源，其中每個資源都可透過邏輯和穩定 URI 來定址。 在本文件中，一組資源稱為摘要。 資料庫帳戶包含一組資料庫，而每個資料庫都包含多個集合，且各集合因此會包含文件、UDF 和其他資源類型。

與這些資源的基本互動模型是透過具有其標準解譯的 HTTP 動詞命令 GET、PUT、POST 和 DELETE。 POST 動詞命令可用於建立新的資源、執行預存程序或發出 Cosmos DB 查詢。 查詢一律是唯讀作業，而且沒有任何副作用。

下列範例示範針對集合進行之 SQL API 查詢的 POST，此集合包含我們到目前為止檢閱過的兩個範例文件。 查詢具有 JSON 名稱屬性的簡單篩選。 請注意 `x-ms-documentdb-isquery` 和內容類型的用法：`application/query+json` 標頭表示該作業是一項查詢。

**要求**

    POST https://<REST URI>/docs HTTP/1.1
    ...
    x-ms-documentdb-isquery: True
    Content-Type: application/query+json

    {      
        "query": "SELECT * FROM Families f WHERE f.id = @familyId",     
        "parameters": [          
            {"name": "@familyId", "value": "AndersenFamily"}         
        ] 
    }


**結果**

```
    HTTP/1.1 200 Ok
    x-ms-activity-id: 8b4678fa-a947-47d3-8dd3-549a40da6eed
    x-ms-item-count: 1
    x-ms-request-charge: 0.32

    <indented for readability, results highlighted>

    {  
       "_rid":"u1NXANcKogE=",
       "Documents":[  
          {  
             "id":"AndersenFamily",
             "lastName":"Andersen",
             "parents":[  
                {  
                   "firstName":"Thomas"
                },
                {  
                   "firstName":"Mary Kay"
                }
             ],
             "children":[  
                {  
                   "firstName":"Henriette Thaulow",
                   "gender":"female",
                   "grade":5,
                   "pets":[  
                      {  
                         "givenName":"Fluffy"
                      }
                   ]
                }
             ],
             "address":{  
                "state":"WA",
                "county":"King",
                "city":"seattle"
             },
             "_rid":"u1NXANcKogEcAAAAAAAAAA==",
             "_ts":1407691744,
             "_self":"dbs\/u1NXAA==\/colls\/u1NXANcKogE=\/docs\/u1NXANcKogEcAAAAAAAAAA==\/",
             "_etag":"00002b00-0000-0000-0000-53e7abe00000",
             "_attachments":"_attachments\/"
          }
       ],
       "count":1
    }
```

第二個範例顯示透過聯結傳回多個結果的較複雜查詢。

**要求**

    POST https://<REST URI>/docs HTTP/1.1
    ...
    x-ms-documentdb-isquery: True
    Content-Type: application/query+json

    {      
        "query": "SELECT 
                     f.id AS familyName, 
                     c.givenName AS childGivenName, 
                     c.firstName AS childFirstName, 
                     p.givenName AS petName 
                  FROM Families f 
                  JOIN c IN f.children 
                  JOIN p in c.pets",     
        "parameters": [] 
    }


**結果**

```
    HTTP/1.1 200 Ok
    x-ms-activity-id: 568f34e3-5695-44d3-9b7d-62f8b83e509d
    x-ms-item-count: 1
    x-ms-request-charge: 7.84

    <indented for readability, results highlighted>

    {  
       "_rid":"u1NXANcKogE=",
       "Documents":[  
          {  
             "familyName":"AndersenFamily",
             "childFirstName":"Henriette Thaulow",
             "petName":"Fluffy"
          },
          {  
             "familyName":"WakefieldFamily",
             "childGivenName":"Jesse",
             "petName":"Goofy"
          },
          {  
             "familyName":"WakefieldFamily",
             "childGivenName":"Jesse",
             "petName":"Shadow"
          }
       ],
       "count":3
    }
```

如果查詢的結果無法放入結果的單一頁面內，則 REST API 會透過 `x-ms-continuation-token` 傳回接續 Token。 用戶端可以透過在後續結果中包括標頭，以將結果分頁。 每頁的結果數目也可以透過 `x-ms-max-item-count` 數字標頭來控制。 如果指定的查詢有一個彙總函數 (如 `COUNT`)，則查詢頁面可對結果頁面傳回部分彙總的值。 用戶端必須對這些結果執行第二層彙總以產生最終結果，例如，對個別頁面中傳回的計數執行加總以傳回總計數。

若要管理查詢的資料一致性原則，請使用 `x-ms-consistency-level` 標頭 (例如，所有 REST API 要求)。 針對工作階段一致性，也需要在查詢要求中回應最新的 `x-ms-session-token` Cookie 標頭。 所查詢集合的索引原則也可能影響查詢結果的一致性。 運用預設的索引原則設定，集合的索引一律具有最新文件內容，而且查詢結果會符合針對資料所選擇的一致性。 如果編索引原則放寬為 Lazy，則查詢可能會傳回過時的結果。 如需詳細資訊，請參閱 [Azure Cosmos DB 的一致性層級][consistency-levels]。

如果集合上所設定的索引原則無法支援指定的查詢，Azure Cosmos DB 伺服器就會傳回 400「錯誤的要求」。 針對範圍查詢 (針對雜湊 (相等) 查閱所設定的路徑) 以及明確地排除不進行編製索引的路徑，會傳回此訊息。 `x-ms-documentdb-query-enable-scan` 標頭可以指定成允許查詢在無法使用索引時執行掃描。

您也可以藉由將 `x-ms-documentdb-populatequerymetrics` 標頭設為 `True`，在查詢執行期間取得詳細的計量。 如需詳細資訊，請參閱[適用於 Azure Cosmos DB 的 SQL 查詢計量](sql-api-sql-query-metrics.md)。

### <a id="DotNetSdk"></a>C# (.NET) SDK
.NET SDK 支援 LINQ 和 SQL 查詢。 下列範例顯示如何執行稍早在本文件中介紹的簡單篩選查詢。

    foreach (var family in client.CreateDocumentQuery(collectionLink, 
        "SELECT * FROM Families f WHERE f.id = \"AndersenFamily\""))
    {
        Console.WriteLine("\tRead {0} from SQL", family);
    }

    SqlQuerySpec query = new SqlQuerySpec("SELECT * FROM Families f WHERE f.id = @familyId");
    query.Parameters = new SqlParameterCollection();
    query.Parameters.Add(new SqlParameter("@familyId", "AndersenFamily"));

    foreach (var family in client.CreateDocumentQuery(collectionLink, query))
    {
        Console.WriteLine("\tRead {0} from parameterized SQL", family);
    }

    foreach (var family in (
        from f in client.CreateDocumentQuery(collectionLink)
        where f.Id == "AndersenFamily"
        select f))
    {
        Console.WriteLine("\tRead {0} from LINQ query", family);
    }

    foreach (var family in client.CreateDocumentQuery(collectionLink)
        .Where(f => f.Id == "AndersenFamily")
        .Select(f => f))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", family);
    }


此範例會比較每個文件內的兩個屬性是否相等，以及使用匿名投射。 

    foreach (var family in client.CreateDocumentQuery(collectionLink,
        @"SELECT {""Name"": f.id, ""City"":f.address.city} AS Family 
        FROM Families f 
        WHERE f.address.city = f.address.state"))
    {
        Console.WriteLine("\tRead {0} from SQL", family);
    }

    foreach (var family in (
        from f in client.CreateDocumentQuery<Family>(collectionLink)
        where f.address.city == f.address.state
        select new { Name = f.Id, City = f.address.city }))
    {
        Console.WriteLine("\tRead {0} from LINQ query", family);
    }

    foreach (var family in
        client.CreateDocumentQuery<Family>(collectionLink)
        .Where(f => f.address.city == f.address.state)
        .Select(f => new { Name = f.Id, City = f.address.city }))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", family);
    }


下一個範例顯示聯結 (透過 LINQ SelectMany 表示)。

    foreach (var pet in client.CreateDocumentQuery(collectionLink,
          @"SELECT p
            FROM Families f 
                 JOIN c IN f.children 
                 JOIN p in c.pets 
            WHERE p.givenName = ""Shadow"""))
    {
        Console.WriteLine("\tRead {0} from SQL", pet);
    }

    // Equivalent in Lambda expressions
    foreach (var pet in
        client.CreateDocumentQuery<Family>(collectionLink)
        .SelectMany(f => f.children)
        .SelectMany(c => c.pets)
        .Where(p => p.givenName == "Shadow"))
    {
        Console.WriteLine("\tRead {0} from LINQ lambda", pet);
    }



.NET 用戶端會在 foreach 區塊中自動逐一查看查詢結果的所有頁面 (如上所示)。 .NET SDK 中也提供 REST API 小節所介紹的查詢選項，方法是在 CreateDocumentQuery 方法中使用 `FeedOptions` and `FeedResponse` 類別。 頁數可以透過 `MaxItemCount` 設定來控制。 

您也可以明確地控制分頁，方法是使用 `IQueryable` 物件來建立 `IDocumentQueryable`，然後讀取 ` ResponseContinuationToken` 值，並將它們以 `RequestContinuationToken` 的形式在 `FeedOptions` 中傳回。 `EnableScanInQuery` 可以設定為在設定的索引編製原則不支援查詢時啟用掃描。 對於已分割的集合，您可以使用 `PartitionKey` 來針對單一分割區執行查詢 (雖然 Cosmos DB 可以從查詢文字自動擷取)，以及使用 `EnableCrossPartitionQuery` 以執行可能需要針對多個分割區執行的查詢。 

如需更多包含查詢的範例，請參閱 [Azure Cosmos DB .NET 範例](https://github.com/Azure/azure-documentdb-net)。 

### <a id="JavaScriptServerSideApi"></a>JavaScript 伺服器端 API
Cosmos DB 提供一個程式設計模型，可使用預存程序和觸發程序，直接對集合執行 JavaScript 型應用程式邏輯。 在集合層級註冊的 JavaScript 邏輯接著可以對給定集合之文件的作業發出資料庫作業。 這些作業會包裝在環境 ACID 交易中。

下列範例示範如何在 JavaScript 伺服器 API 中使用 queryDocuments，從預存程序和觸發程序內發出查詢。

    function businessLogic(name, author) {
        var context = getContext();
        var collectionManager = context.getCollection();
        var collectionLink = collectionManager.getSelfLink()

        // create a new document.
        collectionManager.createDocument(collectionLink,
            { name: name, author: author },
            function (err, documentCreated) {
                if (err) throw new Error(err.message);

                // filter documents by author
                var filterQuery = "SELECT * from root r WHERE r.author = 'George R.'";
                collectionManager.queryDocuments(collectionLink,
                    filterQuery,
                    function (err, matchingDocuments) {
                        if (err) throw new Error(err.message);
    context.getResponse().setBody(matchingDocuments.length);

                        // Replace the author name for all documents that satisfied the query.
                        for (var i = 0; i < matchingDocuments.length; i++) {
                            matchingDocuments[i].author = "George R. R. Martin";
                            // we don't need to execute a callback because they are in parallel
                            collectionManager.replaceDocument(matchingDocuments[i]._self,
                                matchingDocuments[i]);
                        }
                    })
            });
    }

## <a id="References"></a>參考
1. [Azure Cosmos DB 簡介][introduction]
2. [Azure Cosmos DB SQL 規格](http://go.microsoft.com/fwlink/p/?LinkID=510612)
3. [Azure Cosmos DB .NET 範例](https://github.com/Azure/azure-documentdb-net)
4. [Azure Cosmos DB 一致性層級][consistency-levels]
5. ANSI SQL 2011 [http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681](http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=53681)
6. JSON [http://json.org/](http://json.org/)
7. Javascript 規格 [http://www.ecma-international.org/publications/standards/Ecma-262.htm](http://www.ecma-international.org/publications/standards/Ecma-262.htm) 
8. LINQ [http://msdn.microsoft.com/library/bb308959.aspx](http://msdn.microsoft.com/library/bb308959.aspx) 
9. 大型資料庫的查詢評估技術 [http://dl.acm.org/citation.cfm?id=152611](http://dl.acm.org/citation.cfm?id=152611)
10. 平行關聯式資料庫系統中的查詢處理 (IEEE Computer Society Press，1994 年)
11. Lu, Ooi, Tan, 平行關聯式資料庫系統中的查詢處理 (IEEE Computer Society Press，1994 年)。
12. Christopher Olston、Benjamin Reed、Utkarsh Srivastava、Ravi Kumar、Andrew Tomkins：Pig Latin：資料處理的 Not-So-Foreign 語言，SIGMOD 2008。
13. G. Graefe。 The Cascades framework for query optimization. IEEE Data Eng. Bull., 18(3): 1995.

[1]: ./media/sql-api-sql-query/sql-query1.png
[introduction]: introduction.md
[consistency-levels]: consistency-levels.md
