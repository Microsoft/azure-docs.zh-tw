---
title: Azure Cosmos DB：SQL 語法查詢參考 | Microsoft Docs
description: Azure Cosmos DB SQL 查詢語言的參考文件。
services: cosmos-db
author: LalithaMV
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: reference
ms.date: 08/19/2018
ms.author: laviswa
ms.openlocfilehash: 33614628926e53354db14886530d7ca44da61f0a
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145354"
---
# <a name="azure-cosmos-db-sql-syntax-reference"></a>Azure Cosmos DB SQL 語法參考

Azure Cosmos DB 支援在階層式 JSON 文件上使用諸如文法等熟悉的 SQL (結構式查詢語言) 查詢文件，無需明確的結構描述，也不用建立次要索引。 本文提供 SQL 查詢語言的參考/語法文件，其與 SQL API 帳戶相容。 如需 SQL 查詢的逐步解說與範例資料，請參閱[查詢 Azure Cosmos DB 資料](sql-api-sql-query.md)。  
  
請造訪 [Query Playground](http://www.documentdb.com/sql/demo)，您可以在此試用 Azure Cosmos DB 並針對我們的資料集執行 SQL 查詢。  
  
## <a name="select-query"></a>SELECT 查詢  
根據 ANSI-SQL 標準，每個查詢都會包含 SELECT 子句以及選擇性的 FROM 和 WHERE 子句。 針對每個查詢，通常都會列舉 FROM 子句中的來源。 接著，會對來源套用 WHERE 子句中的篩選，以擷取 JSON 文件的子集。 最後，使用 SELECT 子句來投射選取清單中所要求的 JSON 值。 用來描述 SELECT 陳述式的慣例會在「語法」慣例一節中製成資料表。 如需範例，請參閱 [SELECT 查詢範例](sql-api-sql-query.md#SelectClause)
  
**語法**  
  
```sql
<select_query> ::=  
SELECT <select_specification>   
    [ FROM <from_specification>]   
    [ WHERE <filter_condition> ]  
    [ ORDER BY <sort_specification> ]  
```  
  
 **備註**  
  
 如需每個子句的詳細資料，請參閱下列各節：  
  
-   [SELECT 子句](#bk_select_query)    
-   [FROM 子句](#bk_from_clause)    
-   [WHERE 子句](#bk_where_clause)    
-   [ORDER BY 子句](#bk_orderby_clause)  
  
SELECT 陳述式中的子句的順序必須如上所述。 您可以省略任一選用子句。 但是，若使用了選用子句，則這些子句必須以正確的順序出現。  
  
### <a name="logical-processing-order-of-the-select-statement"></a>SELECT 陳述式的邏輯處理順序  
  
子句處理的順序為：  

1.  [FROM 子句](#bk_from_clause)  
2.  [WHERE 子句](#bk_where_clause)  
3.  [ORDER BY 子句](#bk_orderby_clause)  
4.  [SELECT 子句](#bk_select_query)  

請注意，這裡的順序與語法中子句出現的順序不同。 您可以看到由已處理的子句所導入的所有新符號之順序，也能用於稍後要處理的子句。 例如，您可以在 WHERE 和 SELECT 子句中存取 FROM 子句中宣告的別名。  

### <a name="whitespace-characters-and-comments"></a>空白字元和註解  

所有不屬於括號中的字串或引號識別項的空白字元都不是語言文法的一部分，而且會在剖析時忽略。  

查詢語言支援 T-SQL 樣式的註解，例如  

-   陳述式`-- comment text [newline]`  

若空白字元和註解在文法中不具備任何重要性，則必須用來分隔權杖。 例如：`-1e5` 是單一數字的權杖，而 `: – 1 e5` 則是加上減號的權杖，後面接著數字 1 和識別碼 e5。  

##  <a name="bk_select_query"></a> SELECT 子句  
SELECT 陳述式中的子句的順序必須如上所述。 您可以省略任一選用子句。 但是，若使用了選用子句，則這些子句必須以正確的順序出現。 如需範例，請參閱 [SELECT 查詢範例](sql-api-sql-query.md#SelectClause)

**語法**  

```sql
SELECT <select_specification>  

<select_specification> ::=   
      '*'   
      | <object_property_list>   
      | VALUE <scalar_expression> [[ AS ] value_alias]  
  
<object_property_list> ::=   
{ <scalar_expression> [ [ AS ] property_alias ] } [ ,...n ]  
  
```  
  
 **引數**  
  
- `<select_specification>`  

  要針對此結果集選取的屬性或值。  
  
- `'*'`  

  指定應該在不做出任何變更的情況下指定該值。 特別是若已處理的值是物件，則會擷取所有屬性。  
  
- `<object_property_list>`  
  
  指定要擷取的屬性清單。 每個傳回的值都會是具備指定屬性的物件。  
  
- `VALUE`  

  指定應該擷取 JSON 值，而非擷取完整的 JSON 物件。 這與 `<property_list>` 不同，不會在物件中包裝預估的值。  
  
- `<scalar_expression>`  

  表示要計算之值的運算式。 請參閱[純量運算式](#bk_scalar_expressions)一節以取得詳細資料。  
  
**備註**  
  
`SELECT *` 語法只有在 FROM 子句已明顯宣告一個別名時才會有效。 `SELECT *` 提供一個身分識別投影，在無需投影的情況下會非常實用。 SELECT * 只有在已指定 FROM 子句且只導入單一輸入來源時才有效。  
  
請注意，`SELECT <select_list>` 和 `SELECT *` 為「語法捷徑」，可以使用如下所示的簡單 SELECT 陳述式另外表示。  
  
1. `SELECT * FROM ... AS from_alias ...`  
  
   相當於：  
  
   `SELECT from_alias FROM ... AS from_alias ...`  
  
2. `SELECT <expr1> AS p1, <expr2> AS p2,..., <exprN> AS pN [other clauses...]`  
  
   相當於：  
  
   `SELECT VALUE { p1: <expr1>, p2: <expr2>, ..., pN: <exprN> }[other clauses...]`  
  
**另請參閱**  
  
[純量運算式](#bk_scalar_expressions)  
[SELECT 子句](#bk_select_query)  
  
##  <a name="bk_from_clause"></a>FROM 子句  
指定來源或聯結的來源。 除非稍後會在查詢中篩選或投射來源，否則 FROM 子句為選用子句。 此子句的目的是指定查詢必須操作的資料來源。 整個集合通常是來源，但是您可以改為指定集合的子集。 若未指定此子句，系統仍然會在如同 FROM 子句提供了單一文件的情況下執行其他子句。 如需範例，請參閱 [FROM 子句範例](sql-api-sql-query.md#FromClause)
  
**語法**  
  
```sql  
FROM <from_specification>  
  
<from_specification> ::=   
        <from_source> {[ JOIN <from_source>][,...n]}  
  
<from_source> ::=   
          <collection_expression> [[AS] input_alias]  
        | input_alias IN <collection_expression>  
  
<collection_expression> ::=   
        ROOT   
     | collection_name  
     | input_alias  
     | <collection_expression> '.' property_name  
     | <collection_expression> '[' "property_name" | array_index ']'  
```  
  
**引數**  
  
- `<from_source>`  
  
  指定包含或不包含別名的資料來源。 若未指定別名，則會使用下列規則從 `<collection_expression>` 加以推斷：  
  
  -  如果運算式為 collection_name，則會使用 collection_name 作為別名。  
  
  -  如果運算式為 `<collection_expression>`，則會使用 property_name、 then property_name 作為別名。 如果運算式為 collection_name，則會使用 collection_name 作為別名。  
  
- AS `input_alias`  
  
  指定 `input_alias` 為一組由基礎集合運算式傳回的值。  
 
- `input_alias` IN  
  
  指定 `input_alias` 應該代表一組透過反覆計算所有陣列元素所取得的值，其中會依基礎集合運算式傳回每個陣列。 會忽略任何由基礎集合運算式傳回的任何非陣列的值。  
  
- `<collection_expression>`  
  
  指定要用來擷取文件的集合運算式。  
  
- `ROOT`  
  
  指定應該從預設、目前的連線集合中擷取文件。  
  
- `collection_name`  
  
  指定應該從提供的連線集合中擷取文件。 集合名稱必須符合目前連線的集合名稱。  
  
- `input_alias`  
  
  指定應該從由提供的別名定義的其他來源擷取文件。  
  
- `<collection_expression> '.' property_`  
  
  指定文件應該透過存取 `property_name` 屬性或 array_index 陣列元素加以擷取，其中所文件均依指定的集合運算式擷取。  
  
- `<collection_expression> '[' "property_name" | array_index ']'`  
  
  指定文件應該透過存取 `property_name` 屬性或 array_index 陣列元素加以擷取，其中所文件均依指定的集合運算式擷取。  
  
**備註**  
  
`<from_source>(`中提供的所有別名或推斷) 必須是唯一的。 語法 `<collection_expression>.`property_name 與 `<collection_expression>' ['"property_name"']'` 相同。 不過，若屬性名稱包含非識別碼字元，則可以使用後者的語法。  
  
### <a name="handling-missing-properties-missing-array-elements-and-undefined-values"></a>處理遺漏的屬性、遺漏的陣列元素及未定義的值
  
若集合運算式存取屬性或陣列元素，且該值不存在，則會忽略該值且不會進一步處理。  
  
### <a name="collection-expression-context-scoping"></a>集合運算式內容範圍  
  
集合運算方式可以是集合範圍或文件範圍：  
  
-   若集合運算式的基礎來源為 ROOT 或 `collection_name`，則運算式為集合範圍。 這類運算式代表一組直接從集合中擷取的文件，並不會相依於其他集合運算式的處理。  
  
-   若集合運算式的基礎來源為之前在查詢中導入的 `input_alias`，則運算式為文件範圍。 這類運算式代表一組文件，這組文件是透過評估屬於與別名集合相關聯之集的每個文件範圍的集合運算式而取得。  結果集會是透過評估基礎集中每個文件的集合運算式所獲得的聯集。  
  
### <a name="joins"></a>聯結 
  
在目前版本中，Azure Cosmos DB 支援內部聯結。 即將推出其他聯結功能。 

內部聯結是參與聯結之集的完整交叉乘積。 N 方聯結的結果為一組 N 元素 Tuple，其中 Tuple 中的每個值都與參與聯結的別名集相關聯，而且可以透過參考其他子句中的別名加以存取。 如需範例，請參閱 [JOIN 關鍵字範例](sql-api-sql-query.md#Joins)
  
聯結的評估依參與集的內容範圍而定：  
  
-  集合組 A 和集合範圍組 B 之間的聯結會產生集合 A 和 B 中所有元素的交叉乘積。
  
-   集合組 A 和文件範圍組 B 之間的聯結會產生所有集合的聯集，是依評估集合 A 的每個文件的文件範圍集合 B 來取得。  
  
 在目前的版本中，查詢處理器支援集合範圍運算式的上限。  
  
### <a name="examples-of-joins"></a>聯結範例  
  
我們來看看下面的 FROM 子句：`<from_source1> JOIN <from_source2> JOIN ... JOIN <from_sourceN>`  
  
 讓每個來源定義 `input_alias1, input_alias2, …, input_aliasN`。 這個 FROM 子句會傳回一組 N-Tuple (具有 N 個值的 Tuple)。 每個 Tuple 所擁有的值，都是將所有集合別名在其個別集合上反覆運算所產生。  
  
**範例 1** - 2 個來源  
  
- 讓 `<from_source1>` 為集合範圍並代表集 {A, B, C}。  
  
- 讓 `<from_source2>` 為參照 input_alias1 的文件範圍，並代表以下集：  
  
    {1, 2} 為 `input_alias1 = A,`  
  
    {3} 為 `input_alias1 = B,`  
  
    {4, 5} 為 `input_alias1 = C,`  
  
- FROM 子句 `<from_source1> JOIN <from_source2>` 會產生下列 Tuple：  
  
    (`input_alias1, input_alias2`):  
  
    `(A, 1), (A, 2), (B, 3), (C, 4), (C, 5)`  
  
**範例 2** - 3 個來源  
  
- 讓 `<from_source1>` 為集合範圍並代表集 {A, B, C}。  
  
- 讓 `<from_source2>` 為參照 `input_alias1` 的文件範圍，並代表以下集：  
  
    {1, 2} 為 `input_alias1 = A,`  
  
    {3} 為 `input_alias1 = B,`  
  
    {4, 5} 為 `input_alias1 = C,`  
  
- 讓 `<from_source3>` 為參照 `input_alias2` 的文件範圍，並代表以下集：  
  
    {100, 200} 為 `input_alias2 = 1,`  
  
    {300} 為 `input_alias2 = 3,`  
  
- FROM 子句 `<from_source1> JOIN <from_source2> JOIN <from_source3>` 會產生下列 Tuple：  
  
    (input_alias1, input_alias2, input_alias3):  
  
    (A, 1, 100), (A, 1, 200), (B, 3, 300)  
  
  > [!NOTE]
  > 缺少 `input_alias1`、`input_alias2` 的其他值 Tuple，其中 `<from_source3>` 並未傳回任何值。  
  
**範例 3** - 3 個來源  
  
- 讓 <from_source1> 為集合範圍並代表集 {A, B, C}。  
  
- 讓 `<from_source1>` 為集合範圍並代表集 {A, B, C}。  
  
- 讓 <from_source2> 為參照 input_alias1 的文件範圍，並代表以下集：  
  
    {1, 2} 為 `input_alias1 = A,`  
  
    {3} 為 `input_alias1 = B,`  
  
    {4, 5} 為 `input_alias1 = C,`  
  
- 讓 `<from_source3>` 將範圍設定為 `input_alias1` 並代表集：  
  
    {100, 200} 為 `input_alias2 = A,`  
  
    {300} 為 `input_alias2 = C,`  
  
- FROM 子句 `<from_source1> JOIN <from_source2> JOIN <from_source3>` 會產生下列 Tuple：  
  
    (`input_alias1, input_alias2, input_alias3`):  
  
    (A, 1, 100), (A, 1, 200), (A, 2, 100), (A, 2, 200),  (C, 4, 300) ,  (C, 5, 300)  
  
  > [!NOTE]
  > 這會在 `<from_source2>` 和 `<from_source3>` 之間產生交叉乘積，因為兩者都只限於相同的 `<from_source1>` 範圍。  這會讓 4 (2x2) Tuple 具備值 A，0 Tuple 具備值 B (1x0) 而 2 (2x1) Tuple 具備值 C。  
  
**另請參閱**  
  
 [SELECT 子句](#bk_select_query)  
  
##  <a name="bk_where_clause"></a>WHERE 子句  
 指定查詢所傳回的文件之搜尋條件。 如需範例，請參閱 [WHERE 子句範例](sql-api-sql-query.md#WhereClause)
  
 **語法**  
  
```sql  
WHERE <filter_condition>  
<filter_condition> ::= <scalar_expression>  
  
```  
  
 **引數**  
  
-   `<filter_condition>`  
  
     指定要傳回的文件必須滿足的條件。  
  
-   `<scalar_expression>`  
  
     表示要計算之值的運算式。 請參閱[純量運算式](#bk_scalar_expressions)一節以取得詳細資料。  
  
 **備註**  
  
 為了傳回文件，指定為篩選條件的運算式必須評估為 True。 只有在布林值為 True 的情況下才會滿足條件，任何其他值：未定義、Null、False、數字、陣列或物件都不會滿足條件。  
  
##  <a name="bk_orderby_clause"></a>ORDER BY 子句  
 指定由查詢傳回之結果的排列順序。 如需範例，請參閱 [ORDER BY 子句範例](sql-api-sql-query.md#OrderByClause)
  
 **語法**  
  
```sql  
ORDER BY <sort_specification>  
<sort_specification> ::= <sort_expression> [, <sort_expression>]  
<sort_expression> ::= <scalar_expression> [ASC | DESC]  
  
```  
  
 **引數**  
  
-   `<sort_specification>`  
  
     指定要排列查詢結果集的屬性或運算式。 排序資料行可指定為名稱或資料行別名。  
  
     可以指定多個排序資料行。 資料行必須是唯一的。 ORDER BY 子句中的排序資料行順序會定義已排序結果集的組織。 也就是說，結果集是依第一個屬性排序，而該排序清單會依次要屬性進行排序，以此類推。  
  
     ORDER BY 子句中參照的資料行名稱必須明確對應至選取清單中的資料行，或在 FROM 子句中指定之資料表中定義的資料行。  
  
-   `<sort_expression>`  
  
     指定要排列查詢結果集的單一屬性或運算式。  
  
-   `<scalar_expression>`  
  
     請參閱[純量運算式](#bk_scalar_expressions)一節以取得詳細資料。  
  
-   `ASC | DESC`  
  
     指定特定之資料行中的值，應該以遞增或遞減順序排序。 ASC 從最低值到最高值排序。 DESC 從最高值到最低值排序。 ASC 為預設排序順序。 NULL 值會被視為最低的可能值。  
  
 **備註**  
  
 即使查詢文法支援多個屬性順序，但 Azure Cosmos DB 查詢執行階段僅會支援單一屬性及屬性名稱的排序，例如不支援已計算屬性的排序。 排序也需要索引原則，包括適用於屬性及指定類型的範圍索引，搭配最大有效位數。 如需詳細資訊，請參閱索引原則文件。  
  
##  <a name="bk_scalar_expressions"></a>純量運算式  
 純量運算式結合了符號及運算子，可以加以評估以取得單一值。 簡單運算式可以是常數、屬性參考、陣列元素參考、別名參考或函式呼叫。 簡單運算式可以透過使用運算子，與複雜運算式結合。 如需範例，請參閱[純量運算式範例](sql-api-sql-query.md#scalar-expressions)
  
 如需純量運算式具備哪些值的詳細資料，請參閱[常數](#bk_constants)一節。  
  
 **語法**  
  
```sql  
<scalar_expression> ::=  
       <constant>   
     | input_alias   
     | parameter_name  
     | <scalar_expression>.property_name  
     | <scalar_expression>'['"property_name"|array_index']'  
     | unary_operator <scalar_expression>  
     | <scalar_expression> binary_operator <scalar_expression>    
     | <scalar_expression> ? <scalar_expression> : <scalar_expression>  
     | <scalar_function_expression>  
     | <create_object_expression>   
     | <create_array_expression>  
     | (<scalar_expression>)   
  
<scalar_function_expression> ::=  
        'udf.' Udf_scalar_function([<scalar_expression>][,…n])  
        | builtin_scalar_function([<scalar_expression>][,…n])  
  
<create_object_expression> ::=  
   '{' [{property_name | "property_name"} : <scalar_expression>][,…n] '}'  
  
<create_array_expression> ::=  
   '[' [<scalar_expression>][,…n] ']'  
  
```  
  
 **引數**  
  
-   `<constant>`  
  
     代表常數值。 如需詳細資料，請參閱[常數](#bk_constants)一節。  
  
-   `input_alias`  
  
     代表在 `FROM` 子句中導入，且由 `input_alias` 定義的值。  
    此值不保證為**未定義** – 輸入中的**未定義值**會略過。  
  
-   `<scalar_expression>.property_name`  
  
     代表物件屬性值。 若屬性不存在或已在非物件的值中參考，則運算式會評估為**未定義的**值。  
  
-   `<scalar_expression>'['"property_name"|array_index']'`  
  
     代表具有名稱 `property_name` 的屬性值，或具有物件/陣列索引 `array_index` 的陣列元素。 若屬性不存在或屬性/陣列索引已在非物件/陣列的值中參考，則運算式會評估為未定義的值。  
  
-   `unary_operator <scalar_expression>`  
  
     代表已套用至單一值的運算子。 如需詳細資料，請參閱[運算子](#bk_operators)一節。  
  
-   `<scalar_expression> binary_operator <scalar_expression>`  
  
     代表已套用至兩個值的運算子。 如需詳細資料，請參閱[運算子](#bk_operators)一節。  
  
-   `<scalar_function_expression>`  
  
     代表由函式呼叫結果定義的值。  
  
-   `udf_scalar_function`  
  
     使用者定義純量值函式的名稱。  
  
-   `builtin_scalar_function`  
  
     內建純量值函式的名稱。  
  
-   `<create_object_expression>`  
  
     代表搭配指定屬性及其值所建立之新物件取得的值。  
  
-   `<create_array_expression>`  
  
     代表搭配指定值作為元素所建立之新物件取得的值  
  
-   `parameter_name`  
  
     代表指定參數名字的值。 參數名稱必須帶有單一 \@ 作為第一個字元。  
  
 **備註**  
  
 呼叫內建或使用者定義純量值函式時，必須定義所有引數。 若有任何未定義的引數，則不會呼叫函式，同時會產生未定義的結果。  
  
 建立物件時，會略過任何指派未定義值的屬性，且不會納入已建立的物件中。  
  
 建立陣列時，會略過任何指派**未定義**值的元素值，且不會納入已建立的物件中。 這會讓下一個定義的元素以相同方式取代其位置，而建立的陣列將不會具備已略過的索引。  
  
##  <a name="bk_operators"></a> 運算子  
 本節說明支援的運算子。 每個運算子只能指派給一個類別。  
  
 請參閱以下的**運算子類別**表格，如需處理**未定義**值的詳細資料，請輸入輸入值的需求，以及透過不相符的值處理值的需求。  
  
 **運算子類別：**  
  
|**類別**|**詳細資料**|  
|-|-|  
|**arithmetic**|運算子預期的輸入為數字。 輸出也是數字。 若有任何**未定義的**輸入，或輸入了數字以外的類型，則會產生**未定義**的結果。|  
|**bitwise**|運算子預期的輸入為 32 位元的帶正負號整數。 輸出也為 32 位元的帶正負號整數。<br /><br /> 任何非整數值均會進行四捨五入。 正值會無條件捨去，負值則會進位。<br /><br /> 除了 32 位元整數範圍的任何值，都會以去掉其兩個元件標記法的最後 32 位元進行轉換。<br /><br /> 若有任何**未定義的**輸入，或輸入了數字以外的類型，則會產生**未定義**的結果。<br /><br /> **注意：** 以上行為會與 JavaScript 位元運算子行為相容。|  
|**logical**|運算子預期的輸入為布林值。 輸出也是布林值。<br />若有任何**未定義的**輸入，或輸入了布林值以外的類型，則會產生**未定義**的結果。|  
|**comparison**|運算子預期的輸入具有相同的未定義類型。 輸出也是布林值。<br /><br /> 若有任何**未定義的**輸入，或是不同類型的輸入，則會產生**未定義**的結果。<br /><br /> 請參閱**比較值順序**表格以取得值順序的詳細資料。|  
|**字串**|運算子預期的輸入為字串。 輸出也是字串。<br />若有任何**未定義的**輸入，或輸入了字串以外的類型，則會產生**未定義**的結果。|  
  
 **一元運算子**  
  
|**名稱**|**運算子**|**詳細資料**|  
|-|-|-|  
|**arithmetic**|+<br /><br /> -|傳回數值。<br /><br /> 位元否定。 傳回否定的數值。|  
|**bitwise**|~|1 的補數。 傳回數值的補數。|  
|**Logical**|**NOT**|否定。 傳回否定布林值。|  
  
 **二元運算子：**  
  
|**名稱**|**運算子**|**詳細資料**|  
|-|-|-|  
|**arithmetic**|+<br /><br /> -<br /><br /> *<br /><br /> /<br /><br /> %|加法。<br /><br /> 減法。<br /><br /> 乘法。<br /><br /> 除法。<br /><br /> 調節。|  
|**bitwise**|&#124;<br /><br /> &<br /><br /> ^<br /><br /> <<<br /><br /> >><br /><br /> >>>|位元 OR。<br /><br /> 位元 AND。<br /><br /> 位元 XOR。<br /><br /> 向左移位。<br /><br /> 向右移位。<br /><br /> 向右移位並填滿零。|  
|**logical**|**AND**<br /><br /> **或**|邏輯結合。 若兩個引數都為 **True**，則傳回 **True**，否則會傳回 **False**。<br /><br /> 邏輯結合。 若兩個引數都為 **True**，則傳回 **True**，否則會傳回 **False**。|  
|**comparison**|**=**<br /><br /> **!=, <>**<br /><br /> **>**<br /><br /> **>=**<br /><br /> **<**<br /><br /> **<=**<br /><br /> **??**|等於。 若引數相等，則傳回 **True**，否則會傳回 **False**。<br /><br /> 不等於。 若引數不相等，則傳回 **True**，否則會傳回 **False**。<br /><br /> 大於。 如果第一個引數大於第二個引數，則傳回 **True**，否則傳回 **False**。<br /><br /> 大於或等於。 如果第一個引數大於或等於第二個引數，則傳回 **True**，否則傳回 **False**。<br /><br /> 小於。 如果第一個引數小於第二個引數，則傳回 **True**，否則傳回 **False**。<br /><br /> 小於或等於。 如果第一個引數小於或等於第二個引數，則傳回 **True**，否則傳回 **False**。<br /><br /> 聯合。 若第一個引數為**未定義的**值，則會傳回第二個引數。|  
|**String**|**&#124;&#124;**|串連。 傳回兩個引數的串連。|  
  
 **三元運算子：**   
  
|三元運算子|?|若第一個引數評估為 **True**；則會傳回第二個引數，否則會傳回第三個引數。|  
|-|-|-|  
  
 **比較值順序**  
  
|**類型**|**值順序**|  
|-|-|  
|**未定義**|無法比較。|  
|**Null**|單一值：**Null**|  
|**Number**|自然實數。<br /><br /> 負的無限值小於其他任何數值。<br /><br /> 正無限值大於其他任何數值。**NaN** 值無法比較。 與 **NaN** 比較會產生**未定義的**值。|  
|**String**|詞彙編篡順序。|  
|**Array**|沒有順序，但合理。|  
|**Object**|沒有順序，但合理。|  
  
 **備註**  
  
 在 Azure Cosmos DB 中，通常不會知道值的類型，除非實際從資料庫中擷取。 為了支援有效率的查詢執行，大部分的運算子都有嚴謹的類型需求。 此外，運算子本身並不會執行隱含轉換。  
  
 這表示例如 SELECT * FROM ROOT r WHERE r.Age = 21 的查詢僅會傳回具有適當年齡為數字 21 的文件。 具有適當年齡為字串 "21" 或字串 "0021" 的文件並不相符，因為運算式 "21" = 21 評估為未定義。 由於查詢特定值 (例如數字 21) 比搜尋不限數量的可能相符項目更加快速 (例如數字 21 或字串 "21"、"021"、"21.0" 等)，因此可以利用索引進行。 這點不同於 JavaScript 評估不同類型運算子值的方式。  
  
 **陣列和物件的相等和比較**  
  
 使用範圍運算子 (>, >=, <, <=) 比較陣列或物件值會產生未定義的結果，因為物件或陣列值未定義順序。 不過，支援使用等號/不等運算子 (=, !=, <>) 且會以結構化的方式進行比較。  
  
 若兩個陣列具有相同數目的元素，且位於相符位置的元素也相等，則陣列也會相等。 若比較任何元素配對對會產生未定義的結果，則陣列比較的結果為未定義。  
  
 若兩個物件均定義了相同的屬性，而且相符的屬性也相等，則物件會相等。 若比較任何屬性值配對會產生未定義的結果，則物件比較的結果為未定義。  
  
##  <a name="bk_constants"></a> 常數  
 常數也稱為常值或純量值，是代表特定資料值的符號。 常數的格式會依其代表的值資料類型而定。  
  
 **支援的純量資料類型：**  
  
|**類型**|**值順序**|  
|-|-|  
|**未定義**|單一值： **未定義**|  
|**Null**|單一值：**Null**|  
|**布林值**|值：**False**，**True**。|  
|**Number**|雙精確度浮點數，符合 IEEE 754 標準。|  
|**String**|零或更多 Unicode 字元的序列。 字串必須以單引號或雙引號括住。|  
|**Array**|零或更多元素的序列。 除了未定義的類型，每個元素都可以是任何純量資料類型的值。|  
|**Object**|未排序的零或更多名稱/值組。 名稱為 Unicode 字串；除了**未定義**的類型，值可以是任何純量資料類型。|  
  
 **語法**  
  
```sql  
<constant> ::=  
   <undefined_constant>  
     | <null_constant>   
     | <boolean_constant>   
     | <number_constant>   
     | <string_constant>   
     | <array_constant>   
     | <object_constant>   
  
<undefined_constant> ::= undefined  
  
<null_constant> ::= null  
  
<boolean_constant> ::= false | true  
  
<number_constant> ::= decimal_literal | hexadecimal_literal  
  
<string_constant> ::= string_literal  
  
<array_constant> ::=  
    '[' [<constant>][,...n] ']'  
  
<object_constant> ::=   
   '{' [{property_name | "property_name"} : <constant>][,...n] '}'  
  
```  
  
 **引數**  
  
* `<undefined_constant>; undefined`  
  
  代表未定義類型的未定義值。  
  
* `<null_constant>; null`  
  
  代表 **Null** 類型的 **Null** 值。  
  
* `<boolean_constant>`  
  
  代表布林值類型的常數。  
  
* `false`  
  
  代表布林值類型的 **False** 值。  
  
* `true`  
  
  代表布林值類型的 **True** 值。  
  
* `<number_constant>`  
  
  代表常數值。  
  
* `decimal_literal`  
  
  十進位常值是使用十進位表示法或科學記號標記法表示的數字。  
  
* `hexadecimal_literal`  
  
  十六進位常值是使用前置詞 '0x' 後面接著一或多個十六進位數字表示的數字。  
  
* `<string_constant>`  
  
  代表字串類型的常數。  
  
* `string _literal`  
  
  字串常值是由零個或多個 Unicode 字元序列或逸出序列所表示的 Unicode 字串。 字串常值會以單引號 (所有格符號：') 或雙引號 (引號：") 括起來。  
  
 允許下列逸出序列：  
  
|**逸出序列**|**說明**|**Unicode 字元**|  
|-|-|-|  
|\\'|apostrophe (')|U+0027|  
|\\"|引號 (")|U+0022|  
|\\\|反向斜線 (\\)|U+005C|  
|\\/|斜線 (/)|U+002F|  
|\b|退格鍵|U+0008|  
|\f|換頁字元|U+000C|  
|\n|換行字元|U+000A|  
|\r|歸位字元|U+000D|  
|\t|Tab 鍵|U+0009|  
|\uXXXX|由 4 個十六進位數字所定義的 Unicode 字元。|U+XXXX|  
  
##  <a name="bk_query_perf_guidelines"></a> 查詢效能指導方針  
 為了讓查詢在大型集合中有效率的執行，應該使用可透過一或多個索引提供的篩選條件。  
  
 下列的篩選器會被視為索引查詢：  
  
-   使用等號比較運算子 ( = ) 搭配文件路徑運算式和常數。  
  
-   使用範圍比較運算子 (<, \<=, >, >=) 搭配文件路徑運算式和數字常數。  
  
-   文件路徑運算式代表可識別受參考資料庫集合的文件中，常數路徑的任何運算式。  
  
 **文件路徑運算式**  
  
 文件路徑運算式是一種運算式，會採取來自資料庫集合文件之文件上的屬性或陣列索引子的路徑。 這個路徑可以用來識別值的位置，而這些值會由篩選條件直接在資料庫集合中的文件參考。  
  
 對於被視為文件路徑運算式的運算式，它應該：  
  
1.  直接參考集合根。  
  
2.  參考某些文件路徑運算式的屬性或常數陣列索引子  
  
3.  參考代表某文件路徑運算式的別名。  
  
     **語法慣例**  
  
     下表描述的慣例可用來說明下列 SQL 參考中的語法。  
  
    |**慣例**|**用於**|  
    |-|-|    
    |大寫|關鍵字不區分大小寫。|  
    |小寫|關鍵字區分大小寫。|  
    |\<nonterminal>|非終端項，分別定義。|  
    |\<nonterminal> ::=|非終端項的語法定義。|  
    |other_terminal|終端項 (權杖)，以文字詳細說明。|  
    |識別碼|識別碼。 僅允許下列字元：a-z A-Z 0-9 _第一個字元不能為數字。|  
    |"字串"|括號中的字串。 允許任何有效字串。 請參閱 string_literal 的描述。|  
    |'符號'|常值符號，為語法的一部分。|  
    |&#124; (分隔號)|語法項目的替代項目。 您僅可使用其中一個指定項目。|  
    |[ ] /(方括號)|方括號會括住一或多個選用項目。|  
    |[ ,...n ]|指出先前項目可以重複 n 次。 以逗號分隔項目。|  
    |[ ...n ]|指出先前項目可以重複 n 次。 以空格分隔項目。|  
  
##  <a name="bk_built_in_functions"></a>內建函式  
 Azure Cosmos DB 提供許多內建 SQL 函式。 內建函式的分類如下所示。  
  
|函式|說明|  
|--------------|-----------------|  
|[數學函式](#bk_mathematical_functions)|每個數學函數都會執行計算，通常以提供做為引數的輸入值為基礎，並且會傳回數值。|  
|[類型檢查函式](#bk_type_checking_functions)|類型檢查函數可讓您檢查 SQL 查詢中的運算式類型。|  
|[字串函式](#bk_string_functions)|下列字串函式會對字串輸入值執行作業，並傳回字串、數值或布林值。|  
|[陣列函式](#bk_array_functions)|下列陣列函式會對陣列輸入值執行作業，並傳回數值、布林值或陣列值。|  
|[空間函式](#bk_spatial_functions)|下列空間函數會對空間物件輸入值執行作業，並傳回數值或布林值。|  
  
###  <a name="bk_mathematical_functions"></a>數學函式  
 下列函式都會執行計算，通常以提供作為引數的輸入值為基礎，並且會傳回數值。  
  
||||  
|-|-|-|  
|[ABS](#bk_abs)|[ACOS](#bk_acos)|[ASIN](#bk_asin)|  
|[ATAN](#bk_atan)|[ATN2](#bk_atn2)|[CEILING](#bk_ceiling)|  
|[COS](#bk_cos)|[COT](#bk_cot)|[DEGREES](#bk_degrees)|  
|[EXP](#bk_exp)|[FLOOR](#bk_floor)|[LOG](#bk_log)|  
|[LOG10](#bk_log10)|[PI](#bk_pi)|[POWER](#bk_power)|  
|[RADIANS](#bk_radians)|[ROUND](#bk_round)|[SIN](#bk_sin)|  
|[SQRT](#bk_sqrt)|[SQUARE](#bk_square)|[SIGN](#bk_sign)|  
|[TAN](#bk_tan)|[TRUNC](#bk_trunc)||  
  
####  <a name="bk_abs"></a> ABS  
 傳回指定之數值運算式的絕對 (正) 值。  
  
 **語法**  
  
```  
ABS (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例顯示在三個不同的數字上使用 ABS 函式的結果。  
  
```  
SELECT ABS(-1), ABS(0), ABS(1)  
```  
  
 以下為結果集。  
  
```  
[{$1: 1, $2: 0, $3: 1}]  
```  
  
####  <a name="bk_acos"></a> ACOS  
 傳回角度，以弧度為單位，它的餘弦是指定的數值運算式；也稱為反餘弦值。  
  
 **語法**  
  
```  
ACOS(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回 -1 的 ACOS。  
  
```  
SELECT ACOS(-1)  
```  
  
 以下為結果集。  
  
```  
[{"$1": 3.1415926535897931}]  
```  
  
####  <a name="bk_asin"></a> ASIN  
 傳回角度，以弧度為單位，其正弦函數是指定的數值運算式。 這也稱為反正弦值。  
  
 **語法**  
  
```  
ASIN(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回 -1 的 ASIN。  
  
```  
SELECT ASIN(-1)  
```  
  
 以下為結果集。  
  
```  
[{"$1": -1.5707963267948966}]  
```  
  
####  <a name="bk_atan"></a> ATAN  
 傳回角度，以弧度為單位，其正切函數是指定的數值運算式。 這也稱為反正切值。  
  
 **語法**  
  
```  
ATAN(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回指定值的 ATAN。  
  
```  
SELECT ATAN(-45.01)  
```  
  
 以下為結果集。  
  
```  
[{"$1": -1.5485826962062663}]  
```  
  
####  <a name="bk_atn2"></a> ATN2  
 傳回 y/x 有向徑正切函數的主體值，以弧度表示。  
  
 **語法**  
  
```  
ATN2(<numeric_expression>, <numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會計算指定 X 和 Y 元件的 ATN2 值。  
  
```  
SELECT ATN2(35.175643, 129.44)  
```  
  
 以下為結果集。  
  
```  
[{"$1": 1.3054517947300646}]  
```  
  
####  <a name="bk_ceiling"></a> CEILING  
 傳回大於或等於指定之數值運算式的最小整數值。  
  
 **語法**  
  
```  
CEILING (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例顯示搭配 CEILING 函式的正數、負數和零值。  
  
```  
SELECT CEILING(123.45), CEILING(-123.45), CEILING(0.0)  
```  
  
 以下為結果集。  
  
```  
[{$1: 124, $2: -123, $3: 0}]  
```  
  
####  <a name="bk_cos"></a> COS  
 在指定運算式中傳回指定角度的三角餘弦函數，以弧度為單位。  
  
 **語法**  
  
```  
COS(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會計算指定角度的 COS 值。  
  
```  
SELECT COS(14.78)  
```  
  
 以下為結果集。  
  
```  
[{"$1": -0.59946542619465426}]  
```  
  
####  <a name="bk_cot"></a> COT  
 在指定的數值運算式中傳回指定角度的三角餘切函數，以弧度為單位。  
  
 **語法**  
  
```  
COT(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會計算指定角度的 COS 值。  
  
```  
SELECT COT(124.1332)  
```  
  
 以下為結果集。  
  
```  
[{"$1": -0.040311998371148884}]  
```  
  
####  <a name="bk_degrees"></a> DEGREES  
 針對以弧度指定的角度，傳回對應的角度 (以度為單位)。  
  
 **語法**  
  
```  
DEGREES (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回 PI/2 弧度的角度度數。  
  
```  
SELECT DEGREES(PI()/2)  
```  
  
 以下為結果集。  
  
```  
[{"$1": 90}]  
```  
  
####  <a name="bk_floor"></a> FLOOR  
 傳回小於或等於指定之數值運算式的最大整數。  
  
 **語法**  
  
```  
FLOOR (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例顯示搭配 FLOOR 函式的正數、負數和零值。  
  
```  
SELECT FLOOR(123.45), FLOOR(-123.45), FLOOR(0.0)  
```  
  
 以下為結果集。  
  
```  
[{$1: 123, $2: -124, $3: 0}]  
```  
  
####  <a name="bk_exp"></a> EXP  
 傳回指定之數值運算式的指數值。  
  
 **語法**  
  
```  
EXP (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **備註**  
  
 常數 **e** (2.718281…)，為自然對數的基礎。  
  
 數字的指數是常數**e**的數字乘冪。 例如 EXP(1.0) = e^1.0 = 2.71828182845905 和 EXP(10) = e^10 = 22026.4657948067。  
  
 數字的自然對數之指數為數字本身：EXP (LOG (n)) = n。 而數字指數的自然對數則是該數值本身：LOG (EXP (n)) = n。  
  
 **範例**  
  
 下列範例會宣告一個變數，並傳回指定變數 (10) 的指數值。  
  
```  
SELECT EXP(10)  
```  
  
 以下為結果集。  
  
```  
[{$1: 22026.465794806718}]  
```  
  
 下列範例會傳回自然對數的指數值 20，以及指數的自然對數 20。 因為這些函式是彼此的反向函數，傳回值以浮點數學計算，四捨五入之後在這兩種情況均為 20。  
  
```  
SELECT EXP(LOG(20)), LOG(EXP(20))  
```  
  
 以下為結果集。  
  
```  
[{$1: 19.999999999999996, $2: 20}]  
```  
  
####  <a name="bk_log"></a> LOG  
 傳回指定數值運算式的自然對數。  
  
 **語法**  
  
```  
LOG (<numeric_expression> [, <base>])  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
-   `base`  
  
     選用數值引數會設定對數基數。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **備註**  
  
 依預設，LOG() 會傳回自然對數。 您可以使用選用基數參數，將對數基數變更為其他值。  
  
 自然對數為基數 **e** 的對數，其中 **e** 為無理常數，大約等於 2.718281828。  
  
 數字指數的自然對數則是該數值本身：LOG( EXP( n ) ) = n。 而數字的自然對數之指數為數字本身：EXP( LOG( n ) ) = n。  
  
 **範例**  
  
 下列範例會宣告一個變數，並傳回指定變數 (10) 的對數值。  
  
```  
SELECT LOG(10)  
```  
  
 以下為結果集。  
  
```  
[{$1: 2.3025850929940459}]  
```  
  
 下列範例會計算數字指數的 LOG。  
  
```  
SELECT EXP(LOG(10))  
```  
  
 以下為結果集。  
  
```  
[{$1: 10.000000000000002}]  
```  
  
####  <a name="bk_log10"></a> LOG10  
 傳回指定之數值運算式的以 10 為基底的對數。  
  
 **語法**  
  
```  
LOG10 (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **備註**  
  
 LOG10 和 POWER 函式彼此成反比相關。 例如，10 ^ LOG10(n) = n。  
  
 **範例**  
  
 下列範例會宣告一個變數，並傳回指定變數 (100) 的 LOG10 值。  
  
```  
SELECT LOG10(100)  
```  
  
 以下為結果集。  
  
```  
[{$1: 2}]  
```  
  
####  <a name="bk_pi"></a> PI  
 傳回 PI 的常數值。  
  
 **語法**  
  
```  
PI ()  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回 PI 值。  
  
```  
SELECT PI()  
```  
  
 以下為結果集。  
  
```  
[{"$1": 3.1415926535897931}]  
```  
  
####  <a name="bk_power"></a> POWER  
 將指定之運算式的值傳回給指定的乘冪。  
  
 **語法**  
  
```  
POWER (<numeric_expression>, <y>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
-   `y`  
  
     為要提高至 `numeric_expression` 的乘冪。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例示範如何將乘冪數字提高到 3 (數字立方體)。  
  
```  
SELECT POWER(2, 3), POWER(2.5, 3)  
```  
  
 以下為結果集。  
  
```  
[{$1: 8, $2: 15.625}]  
```  
  
####  <a name="bk_radians"></a> RADIANS  
 當您輸入數值運算式 (以度為單位) 時，傳回弧度。  
  
 **語法**  
  
```  
RADIANS (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會使用幾個角度作為輸入，並傳回其對應的弧度值。  
  
```  
SELECT RADIANS(-45.01), RADIANS(-181.01), RADIANS(0), RADIANS(0.1472738), RADIANS(197.1099392)  
```  
  
 以下為結果集。  
  
```  
[{  
       "$1": -0.7855726963226477,  
       "$2": -3.1592204790349356,  
       "$3": 0,  
       "$4": 0.0025704127119236249,  
       "$5": 3.4402174274458375  
   }]  
```  
  
####  <a name="bk_round"></a> ROUND  
 傳回數值，四捨五入到最接近的整數值。  
  
 **語法**  
  
```  
ROUND(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會將以下正數和負數數字四捨五入至最接近的整數。  
  
```  
SELECT ROUND(2.4), ROUND(2.6), ROUND(2.5), ROUND(-2.4), ROUND(-2.6)  
```  
  
 以下為結果集。  
  
```  
[{$1: 2, $2: 3, $3: 3, $4: -2, $5: -3}]  
```  
  
####  <a name="bk_sign"></a> SIGN  
 傳回指定之數值運算式的正數 (+1)、零 (0) 或負數 (-1) 符號。  
  
 **語法**  
  
```  
SIGN(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回從-2 到 2 的 SIGN 值數字。  
  
```  
SELECT SIGN(-2), SIGN(-1), SIGN(0), SIGN(1), SIGN(2)  
```  
  
 以下為結果集。  
  
```  
[{$1: -1, $2: -1, $3: 0, $4: 1, $5: 1}]  
```  
  
####  <a name="bk_sin"></a> SIN  
 在指定運算式中傳回指定角度的三角正弦函數 (以弧度為單位)。  
  
 **語法**  
  
```  
SIN(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會計算指定角度的 SIN 值。  
  
```  
SELECT SIN(45.175643)  
```  
  
 以下為結果集。  
  
```  
[{"$1": 0.929607286611012}]  
```  
  
####  <a name="bk_sqrt"></a> SQRT  
 傳回指定之數值的平方根。  
  
 **語法**  
  
```  
SQRT(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回數字 1 到 3 的平方根值。  
  
```  
SELECT SQRT(1), SQRT(2.0), SQRT(3)  
```  
  
 以下為結果集。  
  
```  
[{$1: 1, $2: 1.4142135623730952, $3: 1.7320508075688772}]  
```  
  
####  <a name="bk_square"></a> SQUARE  
 傳回指定之數值的平方。  
  
 **語法**  
  
```  
SQUARE(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回數字 1 到 3 的平方值。  
  
```  
SELECT SQUARE(1), SQUARE(2.0), SQUARE(3)  
```  
  
 以下為結果集。  
  
```  
[{$1: 1, $2: 4, $3: 9}]  
```  
  
####  <a name="bk_tan"></a> TAN  
 在指定運算式中傳回指定角度的正切函式 (以弧度為單位)。  
  
 **語法**  
  
```  
TAN (<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會計算 PI()/2 的正切函數。  
  
```  
SELECT TAN(PI()/2);  
```  
  
 以下為結果集。  
  
```  
[{"$1": 16331239353195370 }]  
```  
  
####  <a name="bk_trunc"></a> TRUNC  
 傳回數值，截斷至最接近的整數值。  
  
 **語法**  
  
```  
TRUNC(<numeric_expression>)  
```  
  
 **引數**  
  
-   `numeric_expression`  
  
     為數值運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會將以下正數和負數數字截斷為最接近的整數。  
  
```  
SELECT TRUNC(2.4), TRUNC(2.6), TRUNC(2.5), TRUNC(-2.4), TRUNC(-2.6)  
```  
  
 以下為結果集。  
  
```  
[{$1: 2, $2: 2, $3: 2, $4: -2, $5: -2}]  
```  
  
###  <a name="bk_type_checking_functions"></a>類型檢查函式  
 下列函式支援針對輸入值檢查類型，並且會傳回布林值。  
  
||||  
|-|-|-|  
|[IS_ARRAY](#bk_is_array)|[IS_BOOL](#bk_is_bool)|[IS_DEFINED](#bk_is_defined)|  
|[IS_NULL](#bk_is_null)|[IS_NUMBER](#bk_is_number)|[IS_OBJECT](#bk_is_object)|  
|[IS_PRIMITIVE](#bk_is_primitive)|[IS_STRING](#bk_is_string)||  
  
####  <a name="bk_is_array"></a> IS_ARRAY  
 傳回布林值，表示指定之運算式的類型為陣列。  
  
 **語法**  
  
```  
IS_ARRAY(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_ARRAY 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
 IS_ARRAY(true),   
 IS_ARRAY(1),  
 IS_ARRAY("value"),  
 IS_ARRAY(null),  
 IS_ARRAY({prop: "value"}),   
 IS_ARRAY([1, 2, 3]),  
 IS_ARRAY({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: false, $2: false, $3: false, $4: false, $5: false, $6: true}]  
```  
  
####  <a name="bk_is_bool"></a> IS_BOOL  
 傳回布林值，表示指定之運算式的類型為布林值。  
  
 **語法**  
  
```  
IS_BOOL(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_BOOL 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
    IS_BOOL(true),   
    IS_BOOL(1),  
    IS_BOOL("value"),   
    IS_BOOL(null),  
    IS_BOOL({prop: "value"}),   
    IS_BOOL([1, 2, 3]),  
    IS_BOOL({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: true, $2: false, $3: false, $4: false, $5: false, $6: false}]  
```  
  
####  <a name="bk_is_defined"></a> IS_DEFINED  
 傳回布林值，表示屬性是否已經指派值。  
  
 **語法**  
  
```  
IS_DEFINED(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會檢查指定的 JSON 文件內的屬性是否存在。 第一次會傳回 True，因為出現了 "a"，但第二次就會傳回 False 因為 "b" 不存在。  
  
```  
SELECT IS_DEFINED({ "a" : 5 }.a), IS_DEFINED({ "a" : 5 }.b)  
```  
  
 以下為結果集。  
  
```  
[{  
       "$1": true,    
       "$2": false   
   }]  
```  
  
####  <a name="bk_is_null"></a> IS_NULL  
 傳回布林值，表示指定之運算式的類型為 null。  
  
 **語法**  
  
```  
IS_NULL(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_NULL 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
    IS_NULL(true),   
    IS_NULL(1),  
    IS_NULL("value"),   
    IS_NULL(null),  
    IS_NULL({prop: "value"}),   
    IS_NULL([1, 2, 3]),  
    IS_NULL({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: false, $2: false, $3: false, $4: true, $5: false, $6: false}]  
```  
  
####  <a name="bk_is_number"></a> IS_NUMBER  
 傳回布林值，表示指定之運算式的類型為數字。  
  
 **語法**  
  
```  
IS_NUMBER(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_NULL 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
    IS_NUMBER(true),   
    IS_NUMBER(1),  
    IS_NUMBER("value"),   
    IS_NUMBER(null),  
    IS_NUMBER({prop: "value"}),   
    IS_NUMBER([1, 2, 3]),  
    IS_NUMBER({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: false, $2: true, $3: false, $4: false, $5: false, $6: false}]  
```  
  
####  <a name="bk_is_object"></a> IS_OBJECT  
 傳回布林值，表示指定之運算式的類型為 JSON 物件。  
  
 **語法**  
  
```  
IS_OBJECT(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_OBJECT 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
    IS_OBJECT(true),   
    IS_OBJECT(1),  
    IS_OBJECT("value"),   
    IS_OBJECT(null),  
    IS_OBJECT({prop: "value"}),   
    IS_OBJECT([1, 2, 3]),  
    IS_OBJECT({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: false, $2: false, $3: false, $4: false, $5: true, $6: false}]  
```  
  
####  <a name="bk_is_primitive"></a> IS_PRIMITIVE  
 傳回布林值，表示指定之運算式的類型為基本類型 (字串、布林值、數值或 Null)。  
  
 **語法**  
  
```  
IS_PRIMITIVE(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_PRIMITIVE 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
           IS_PRIMITIVE(true),   
           IS_PRIMITIVE(1),  
           IS_PRIMITIVE("value"),   
           IS_PRIMITIVE(null),  
           IS_PRIMITIVE({prop: "value"}),   
           IS_PRIMITIVE([1, 2, 3]),  
           IS_PRIMITIVE({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{"$1": true, "$2": true, "$3": true, "$4": true, "$5": false, "$6": false, "$7": false}]  
```  
  
####  <a name="bk_is_string"></a> IS_STRING  
 傳回布林值，表示指定之運算式的類型為字串。  
  
 **語法**  
  
```  
IS_STRING(<expression>)  
```  
  
 **引數**  
  
-   `expression`  
  
     為任何有效運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會使用 IS_STRING 函式，檢查 JSON 布林值的物件、數字、字串、Null、物件、陣列以及未定義的類型。  
  
```  
SELECT   
       IS_STRING(true),   
       IS_STRING(1),  
       IS_STRING("value"),   
       IS_STRING(null),  
       IS_STRING({prop: "value"}),   
       IS_STRING([1, 2, 3]),  
       IS_STRING({prop: "value"}.prop2)  
```  
  
 以下為結果集。  
  
```  
[{$1: false, $2: false, $3: true, $4: false, $5: false, $6: false}]  
```  
  
###  <a name="bk_string_functions"></a> 字串函式  
 下列純量函數會對字串輸入值執行作業，並傳回字串、數值或布林值。  
  
||||  
|-|-|-|  
|[CONCAT](#bk_concat)|[CONTAINS](#bk_contains)|[ENDSWITH](#bk_endswith)|  
|[INDEX_OF](#bk_index_of)|[LEFT](#bk_left)|[LENGTH](#bk_length)|  
|[LOWER](#bk_lower)|[LTRIM](#bk_ltrim)|[REPLACE](#bk_replace)|  
|[REPLICATE](#bk_replicate)|[REVERSE](#bk_reverse)|[RIGHT](#bk_right)|  
|[RTRIM](#bk_rtrim)|[STARTSWITH](#bk_startswith)|[SUBSTRING](#bk_substring)|  
|[ToString](#bk_tostring)|[TRIM](#bk_trim)|[UPPER](#bk_upper)||| 
  
####  <a name="bk_concat"></a> CONCAT  
 傳回字串，該字串是串連兩個或多個字串值的結果。  
  
 **語法**  
  
```  
CONCAT(<str_expr>, <str_expr> [, <str_expr>])  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例會傳回指定值的串連字串。  
  
```  
SELECT CONCAT("abc", "def")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "abcdef"}  
```  
  
####  <a name="bk_contains"></a> CONTAINS  
 傳回布林值，表示第一個字串運算式是否包含第二個字串運算式。  
  
 **語法**  
  
```  
CONTAINS(<str_expr>, <str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會檢查 "abc" 是否包含 "ab" 及 "d"。  
  
```  
SELECT CONTAINS("abc", "ab"), CONTAINS("abc", "d")  
```  
  
 以下為結果集。  
  
```  
[{"$1": true, "$2": false}]  
```  
  
####  <a name="bk_endswith"></a> ENDSWITH  
 傳回布林值，表示第一個字串運算式是否以第二個字串運算式結尾。  
  
 **語法**  
  
```  
ENDSWITH(<str_expr>, <str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會傳回以 "b" 或 "bc" 結尾的 "abc"。  
  
```  
SELECT ENDSWITH("abc", "b"), ENDSWITH("abc", "bc")  
```  
  
 以下為結果集。  
  
```  
[{"$1": false, "$2": true}]  
```  
  
####  <a name="bk_index_of"></a> INDEX_OF  
 傳回第一個指定的字串運算式中，第二個字串運算式第一次出現的開始位置，或者如果找不到字串，則為 -1。  
  
 **語法**  
  
```  
INDEX_OF(<str_expr>, <str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例會傳回 "abc" 內各種子字串的索引。  
  
```  
SELECT INDEX_OF("abc", "ab"), INDEX_OF("abc", "b"), INDEX_OF("abc", "c")  
```  
  
 以下為結果集。  
  
```  
[{"$1": 0, "$2": 1, "$3": -1}]  
```  
  
####  <a name="bk_left"></a> LEFT  
 傳回具有指定字元數目的字串左側部分。  
  
 **語法**  
  
```  
LEFT(<str_expr>, <num_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
-   `num_expr`  
  
     為任何有效的數值運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例會傳回 "abc"左側部分的各種長度值。  
  
```  
SELECT LEFT("abc", 1), LEFT("abc", 2)  
```  
  
 以下為結果集。  
  
```  
[{"$1": "a", "$2": "ab"}]  
```  
  
####  <a name="bk_length"></a> LENGTH  
 傳回指定字串運算式的字元數目。  
  
 **語法**  
  
```  
LENGTH(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例會傳回字串的長度。  
  
```  
SELECT LENGTH("abc")  
```  
  
 以下為結果集。  
  
```  
[{"$1": 3}]  
```  
  
####  <a name="bk_lower"></a> LOWER  
 傳回將大寫字元資料轉換成小寫之後的字串運算式。  
  
 **語法**  
  
```  
LOWER(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 LOWER。  
  
```  
SELECT LOWER("Abc")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "abc"}]  
  
```  
  
####  <a name="bk_ltrim"></a> LTRIM  
 傳回移除開頭空白之後的字串運算式。  
  
 **語法**  
  
```  
LTRIM(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 LTRIM。  
  
```  
SELECT LTRIM("  abc"), LTRIM("abc"), LTRIM("abc   ")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "abc", "$2": "abc", "$3": "abc   "}]  
```  
  
####  <a name="bk_replace"></a> REPLACE  
 使用其他字串值取代指定的字串值的所有項目。  
  
 **語法**  
  
```  
REPLACE(<str_expr>, <str_expr>, <str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 REPLACE。  
  
```  
SELECT REPLACE("This is a Test", "Test", "desk")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "This is a desk"}]  
```  
  
####  <a name="bk_replicate"></a> REPLICATE  
 將字串值重複指定的次數。  
  
 **語法**  
  
```  
REPLICATE(<str_expr>, <num_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
-   `num_expr`  
  
     為任何有效的數值運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 REPLICATE。  
  
```  
SELECT REPLICATE("a", 3)  
```  
  
 以下為結果集。  
  
```  
[{"$1": "aaa"}]  
```  
  
####  <a name="bk_reverse"></a> REVERSE  
 傳回反向順序的字串值。  
  
 **語法**  
  
```  
REVERSE(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 REVERSE。  
  
```  
SELECT REVERSE("Abc")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "cbA"}]  
```  
  
####  <a name="bk_right"></a> RIGHT  
 傳回具有指定字元數目的字串右側部分。  
  
 **語法**  
  
```  
RIGHT(<str_expr>, <num_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
-   `num_expr`  
  
     為任何有效的數值運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例會傳回 "abc"右側部分的各種長度值。  
  
```  
SELECT RIGHT("abc", 1), RIGHT("abc", 2)  
```  
  
 以下為結果集。  
  
```  
[{"$1": "c", "$2": "bc"}]  
```  
  
####  <a name="bk_rtrim"></a> RTRIM  
 傳回移除結尾空白之後的字串運算式。  
  
 **語法**  
  
```  
RTRIM(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 RTRIM。  
  
```  
SELECT RTRIM("  abc"), RTRIM("abc"), RTRIM("abc   ")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "   abc", "$2": "abc", "$3": "abc"}]  
```  
  
####  <a name="bk_startswith"></a> STARTSWITH  
 傳回布林值，表示第一個字串運算式是否以第二個字串運算式開頭。  
  
 **語法**  
  
```  
STARTSWITH(<str_expr>, <str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例會檢查字串 "abc" 是否以 "b" 和 "a" 開頭。  
  
```  
SELECT STARTSWITH("abc", "b"), STARTSWITH("abc", "a")  
```  
  
 以下為結果集。  
  
```  
[{"$1": false, "$2": true}]  
```  
  
####  <a name="bk_substring"></a> SUBSTRING  
 傳回字串運算式的部分，從指定字元以零為起始的位置開始，直到指定的長度，或直到字串的結尾。  
  
 **語法**  
  
```  
SUBSTRING(<str_expr>, <num_expr> [, <num_expr>])  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
-   `num_expr`  
  
     為任何有效的數值運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例會傳回從 1 開始，且長度為 1 個字元的子字串 "abc"。  
  
```  
SELECT SUBSTRING("abc", 1, 1)  
```  
  
 以下為結果集。  
  
```  
[{"$1": "b"}]  
```  
####  <a name="bk_tostring"></a> ToString  
 傳回純量運算式的字串表示法。 
  
 **語法**  
  
```  
ToString(<expr>)
```  
  
 **引數**  
  
-   `expr`  
  
     為任何有效的純量運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例顯示 ToString 在不同類型之間的行為方式。   
  
```  
SELECT ToString(1.0000), ToString("Hello World"), ToString(NaN), ToString(Infinity),
ToString(IS_STRING(ToString(undefined))), IS_STRING(ToString(0.1234), ToString(false), ToString(undefined))
```  
  
 以下為結果集。  
  
```  
[{"$1": "1", "$2": "Hello World", "$3": "NaN", "$4": "Infinity", "$5": "false", "$6": true, "$7": "false"}]  
```  
 指定下列輸入︰
```  
{"Products":[{"ProductID":1,"Weight":4,"WeightUnits":"lb"},{"ProductID":2,"Weight":32,"WeightUnits":"kg"},{"ProductID":3,"Weight":400,"WeightUnits":"g"},{"ProductID":4,"Weight":8999,"WeightUnits":"mg"}]}
```    
 下列範例顯示 ToString 可以如何與其他字串函式 (例如 CONCAT) 搭配使用。   
 
```  
SELECT 
CONCAT(ToString(p.Weight), p.WeightUnits) 
FROM p in c.Products 
```  

 以下為結果集。  
  
```  
[{"$1":"4lb" },
 {"$1":"32kg"},
 {"$1":"400g" },
 {"$1":"8999mg" }]

```  
指定下列輸入。
```
{"id":"08259","description":"Cereals ready-to-eat, KELLOGG, KELLOGG'S CRISPIX","nutrients":[{"id":"305","description":"Caffeine","units":"mg"},{"id":"306","description":"Cholesterol, HDL","nutritionValue":30,"units":"mg"},{"id":"307","description":"Sodium, NA","nutritionValue":612,"units":"mg"},{"id":"308","description":"Protein, ABP","nutritionValue":60,"units":"mg"},{"id":"309","description":"Zinc, ZN","nutritionValue":null,"units":"mg"}]}
```
 下列範例顯示 ToString 可以如何與其他字串函式 (例如 REPLACE) 搭配使用。   
```
SELECT 
    n.id AS nutrientID,
    REPLACE(ToString(n.nutritionValue), "6", "9") AS nutritionVal
FROM food 
JOIN n IN food.nutrients
```
 以下為結果集。  
 ```
[{"nutrientID":"305"},
{"nutrientID":"306","nutritionVal":"30"},
{"nutrientID":"307","nutritionVal":"912"},
{"nutrientID":"308","nutritionVal":"90"},
{"nutrientID":"309","nutritionVal":"null"}]
 ``` 
 
####  <a name="bk_trim"></a> TRIM  
 傳回移除開頭和尾端空白之後的字串運算式。  
  
 **語法**  
  
```  
TRIM(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 TRIM。  
  
```  
SELECT TRIM("   abc"), TRIM("   abc   "), TRIM("abc   "), TRIM("abc")   
```  
  
 以下為結果集。  
  
```  
[{"$1": "abc", "$2": "abc", "$3": "abc", "$4": "abc"}]  
``` 
####  <a name="bk_upper"></a> UPPER  
 傳回將小寫字元資料轉換成大寫之後的字串運算式。  
  
 **語法**  
  
```  
UPPER(<str_expr>)  
```  
  
 **引數**  
  
-   `str_expr`  
  
     為任何有效的字運算式。  
  
 **傳回類型**  
  
 傳回字串運算式。  
  
 **範例**  
  
 下列範例示範如何在查詢中使用 UPPER  
  
```  
SELECT UPPER("Abc")  
```  
  
 以下為結果集。  
  
```  
[{"$1": "ABC"}]  
```  
  
###  <a name="bk_array_functions"></a> 陣列函陣列函式  
 下列純量值函式會對陣列輸入值執行作業，並傳回數值、布林值或陣列值  
  
||||  
|-|-|-|  
|[ARRAY_CONCAT](#bk_array_concat)|[ARRAY_CONTAINS](#bk_array_contains)|[ARRAY_LENGTH](#bk_array_length)|  
|[ARRAY_SLICE](#bk_array_slice)|||  
  
####  <a name="bk_array_concat"></a> ARRAY_CONCAT  
 傳回串連兩個或多個陣列值之結果的陣列。  
  
 **語法**  
  
```  
ARRAY_CONCAT (<arr_expr>, <arr_expr> [, <arr_expr>])  
```  
  
 **引數**  
  
-   `arr_expr`  
  
     為任何有效陣列運算式。  
  
 **傳回類型**  
  
 傳回陣列運算式。  
  
 **範例**  
  
 下列範例顯示如何串連兩個陣列。  
  
```  
SELECT ARRAY_CONCAT(["apples", "strawberries"], ["bananas"])  
```  
  
 以下為結果集。  
  
```  
[{"$1": ["apples", "strawberries", "bananas"]}]  
```  
  
####  <a name="bk_array_contains"></a> ARRAY_CONTAINS  
傳回布林值，表示陣列是否包含指定值。 可以指定要進行完整或部分比對。 

 **語法**  
  
```  
ARRAY_CONTAINS (<arr_expr>, <expr> [, bool_expr])  
```  
  
 **引數**  
  
-   `arr_expr`  
  
     為任何有效陣列運算式。  
  
-   `expr`  
  
     為任何有效運算式。  

-   `bool_expr`  
  
     為任何布林運算式。       
  
 **傳回類型**  
  
 傳回布林值。  
  
 **範例**  
  
 下列範例說明如何使用 ARRAY_CONTAINS 檢查陣列中的成員資格。  
  
```  
SELECT   
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "apples"),  
           ARRAY_CONTAINS(["apples", "strawberries", "bananas"], "mangoes")  
```  
  
 以下為結果集。  
  
```  
[{"$1": true, "$2": false}]  
```  

 下列範例說明如何使用 ARRAY_CONTAINS 檢查陣列中的 JSON 部份相符。  
  
```  
SELECT  
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}, true), 
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "apples"}),
    ARRAY_CONTAINS([{"name": "apples", "fresh": true}, {"name": "strawberries", "fresh": true}], {"name": "mangoes"}, true) 
```  
  
 以下為結果集。  
  
```  
[{
  "$1": true,
  "$2": false,
  "$3": false
}] 
```  
  
####  <a name="bk_array_length"></a> ARRAY_LENGTH  
 傳回指定陣列運算式的元素數目。  
  
 **語法**  
  
```  
ARRAY_LENGTH(<arr_expr>)  
```  
  
 **引數**  
  
-   `arr_expr`  
  
     為任何有效陣列運算式。  
  
 **傳回類型**  
  
 傳回數值運算式。  
  
 **範例**  
  
 下列範例說明如何使用 ARRAY_LENGTH 取得陣列的長度。  
  
```  
SELECT ARRAY_LENGTH(["apples", "strawberries", "bananas"])  
```  
  
 以下為結果集。  
  
```  
[{"$1": 3}]  
```  
  
####  <a name="bk_array_slice"></a> ARRAY_SLICE  
 傳回陣列運算式的一部分。
  
 **語法**  
  
```  
ARRAY_SLICE (<arr_expr>, <num_expr> [, <num_expr>])  
```  
  
 **引數**  
  
-   `arr_expr`  
  
     為任何有效陣列運算式。  
  
-   `num_expr`  
  
     為任何有效的數值運算式。  
  
 **傳回類型**  
  
 傳回布林值。  
  
 **範例**  
  
 下列範例說明如何使用 ARRAY_SLICE 取得陣列的一部分。  
  
```  
SELECT   
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1),  
           ARRAY_SLICE(["apples", "strawberries", "bananas"], 1, 1)  
```  
  
 以下為結果集。  
  
```  
[{  
           "$1": ["strawberries", "bananas"],   
           "$2": ["strawberries"]  
       }]  
```  
  
###  <a name="bk_spatial_functions"></a> 空間函式  
 下列純量值函式會對空間物件輸入值執行作業，並傳回數值或布林值。  
  
||||  
|-|-|-|  
|[ST_DISTANCE](#bk_st_distance)|[ST_WITHIN](#bk_st_within)|[ST_INTERSECTS](#bk_st_intersects)|[ST_ISVALID](#bk_st_isvalid)|  
|[ST_ISVALIDDETAILED](#bk_st_isvaliddetailed)|||  
  
####  <a name="bk_st_distance"></a> ST_DISTANCE  
 傳回兩個 GeoJSON Point、Polygon 或 LineString 運算式之間的距離。  
  
 **語法**  
  
```  
ST_DISTANCE (<spatial_expr>, <spatial_expr>)  
```  
  
 **引數**  
  
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 物件運算式。  
  
 **傳回類型**  
  
 傳回數值運算式，其中包含距離。 這在預設參照系統中以公尺表示。  
  
 **範例**  
  
 以下範例顯示如何使用ST_DISTANCE 內建函式傳回所有家族文件，且這些文件在 30 公里的指定位置內。 .  
  
```  
SELECT f.id   
FROM Families f   
WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000  
```  
  
 以下為結果集。  
  
```  
[{  
  "id": "WakefieldFamily"  
}]  
```  
  
####  <a name="bk_st_within"></a> ST_WITHIN  
 傳回布林運算式，指出第一個引數中指定的 GeoJSON 物件 (Point、Polygon 或 LineString) 是否位在第二個引數中的 GeoJSON (Point、Polygon 或 LineString) 內。  
  
 **語法**  
  
```  
ST_WITHIN (<spatial_expr>, <spatial_expr>)  
```  
  
 **引數**  
  
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 物件運算式。  
 
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 物件運算式。  
  
 **傳回類型**  
  
 傳回布林值。  
  
 **範例**  
  
 下列範例說明如何使用 ST_WITHIN 尋找多邊形內的所有家族文件。  
  
```  
SELECT f.id   
FROM Families f   
WHERE ST_WITHIN(f.location, {  
    'type':'Polygon',   
    'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]  
})  
```  
  
 以下為結果集。  
  
```  
[{ "id": "WakefieldFamily" }]  
```  

####  <a name="bk_st_intersects"></a> ST_INTERSECTS  
 傳回布林運算式，指出第一個引數交集中指定的 GeoJSON 物件 (Point、Polygon 或 LineString) 是否位在第二個引數中的 GeoJSON (Point、Polygon 或 LineString) 內。  
  
 **語法**  
  
```  
ST_INTERSECTS (<spatial_expr>, <spatial_expr>)  
```  
  
 **引數**  
  
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 物件運算式。  
 
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 物件運算式。  
  
 **傳回類型**  
  
 傳回布林值。  
  
 **範例**  
  
 下列範例說明如何尋找與給定的多邊形交集的所有區域。  
  
```  
SELECT a.id   
FROM Areas a   
WHERE ST_INTERSECTS(a.location, {  
    'type':'Polygon',   
    'coordinates': [[[31.8, -5], [32, -5], [32, -4.7], [31.8, -4.7], [31.8, -5]]]  
})  
```  
  
 以下為結果集。  
  
```  
[{ "id": "IntersectingPolygon" }]  
```  
  
####  <a name="bk_st_isvalid"></a> ST_ISVALID  
 傳回布林值，指出指定的 GeoJSON Point、Polygon 或 LineString 運算式是否有效。  
  
 **語法**  
  
```  
ST_ISVALID(<spatial_expr>)  
```  
  
 **引數**  
  
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點、Polygon 或 LineString 運算式。  
  
 **傳回類型**  
  
 傳回布林運算式。  
  
 **範例**  
  
 下列範例說明如何使用 ST_VALID 檢查點是否有效。  
  
 例如，此點的維度值不在數值 [-90, 90] 的有效範圍內，因此查詢會傳回 False。  
  
 對於多邊形，GeoJSON 規格需要此資料；若要建立一個封閉的形狀，最後一個座標組應該與第一個座標組相同。 多邊形內的點必須以逆時針順序指定。 以順時針順序指定多邊形，代表區域內的反轉。  
  
```  
SELECT ST_ISVALID({ "type": "Point", "coordinates": [31.9, -132.8] })  
```  
  
 以下為結果集。  
  
```  
[{ "$1": false }]  
```  
  
####  <a name="bk_st_isvaliddetailed"></a> ST_ISVALIDDETAILED  
 如果指定的 GeoJSON Point、Polygon 或 LineString 運算式有效，就傳回包含布林值的 JSON 值；但如果是無效的，就會額外加上做為字串值的原因。  
  
 **語法**  
  
```  
ST_ISVALID(<spatial_expr>)  
```  
  
 **引數**  
  
-   `spatial_expr`  
  
     為任何有效的 GeoJSON 點或多邊形運算式。  
  
 **傳回類型**  
  
 如果指定的 GeoJSON 點或多邊形運算式是有效的，就會傳回包含布林值的 JSON 值；但如果是無效的，就會額外加上做為字串值的原因。  
  
 **範例**  
  
 下列範例說明如何使用 ST_ISVALIDDETAILED 檢查有效性 (包含詳細資料)。  
  
```  
SELECT ST_ISVALIDDETAILED({   
  "type": "Polygon",   
  "coordinates": [[ [ 31.8, -5 ], [ 31.8, -4.7 ], [ 32, -4.7 ], [ 32, -5 ] ]]  
})  
```  
  
 以下為結果集。  
  
```  
[{  
  "$1": {   
    "valid": false,   
    "reason": "The Polygon input is not valid because the start and end points of the ring number 1 are not the same. Each ring of a polygon must have the same start and end points."   
  }  
}]  
```  
  
## <a name="next-steps"></a>後續步驟  
 [適用於Azure Cosmos DB 的 SQL 語法和 SQL 查詢](sql-api-sql-query.md)   
 [Azure Cosmos DB 文件](https://docs.microsoft.com/azure/cosmos-db/)  
  
  
