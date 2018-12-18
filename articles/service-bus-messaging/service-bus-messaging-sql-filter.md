---
title: Azure 服務匯流排 SQLFilter 語法參考 | Microsoft Docs
description: SQLFilter 文法的詳細資料。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/05/2018
ms.author: spelluru
ms.openlocfilehash: c94ffed753ebf8fddbd553977c5d733f2306971d
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698327"
---
# <a name="sqlfilter-syntax"></a>SQLFilter 語法

*SqlFilter* 物件為 [SqlFilter Class](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) 的執行個體，代表對 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 進行評估之以 SQL 語言為基礎的篩選條件運算式。 SqlFilter 支援 SQL-92 標準的子集。  
  
 本主題列出 SqlFilter 文法的詳細資料。  
  
```  
<predicate ::=  
      { NOT <predicate> }  
      | <predicate> AND <predicate>  
      | <predicate> OR <predicate>  
      | <expression> { = | <> | != | > | >= | < | <= } <expression>  
      | <property> IS [NOT] NULL  
      | <expression> [NOT] IN ( <expression> [, ...n] )  
      | <expression> [NOT] LIKE <pattern> [ESCAPE <escape_char>]  
      | EXISTS ( <property> )  
      | ( <predicate> )  
  
```  
  
```  
<expression> ::=  
      <constant>   
      | <function>  
      | <property>  
      | <expression> { + | - | * | / | % } <expression>  
      | { + | - } <expression>  
      | ( <expression> )  
  
```  
  
```  
<property> :=   
       [<scope> .] <property_name>  
  
```  
  
## <a name="arguments"></a>引數  
  
-   `<scope>` 是表示 `<property_name>` 範圍的選擇性字串。 有效值為 `sys` 或 `user`。 `sys` 值表示系統範圍，當中 `<property_name>` 為 [BrokeredMessage 類別](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)的公用屬性名稱。 `user` 表示使用者範圍，當中 `<property_name>` 為 [BrokeredMessage 類別](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)字典的索引鍵。 如果 `<scope>` 未指定，則 `user` 範圍是預設範圍。  
  
## <a name="remarks"></a>備註

嘗試存取不存在的系統屬性時會發生錯誤，而嘗試存取不存在的使用者屬性時不會發生錯誤。 反之，不存在的使用者屬性會內部評估為未知的值。 未知的值在運算子評估期間會特別處理。  
  
## <a name="propertyname"></a>property_name  
  
```  
<property_name> ::=  
     <identifier>  
     | <delimited_identifier>  
  
<identifier> ::=  
     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
```  
  
### <a name="arguments"></a>引數  

 `<regular_identifier>` 是由下列規則運算式所表示的字串︰  
  
```  
[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
```  
  
此文法表示任何以字母為開頭且後面跟著一或多個底線/字母/數字的字串。  
  
`[:IsLetter:]` 表示分類為 Unicode 字母的任何 Unicode 字元。 如果 `c` 為 Unicode 字母，`System.Char.IsLetter(c)` 會傳回 `true`。  
  
`[:IsDigit:]` 表示分類為十進位數字的任何 Unicode 字元。 如果 `c` 為 Unicode 數字，`System.Char.IsDigit(c)` 會傳回 `true`。  
  
`<regular_identifier>` 不能是保留的關鍵字。  
  
`<delimited_identifier>` 是使用左/右方括弧 ([]) 括住的任何字串。 右方括弧會以兩個右方括弧代表。 以下為 `<delimited_identifier>`的範例：  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
`<quoted_identifier>` 是以雙引號括住的任何字串。 識別項中的雙引號會以兩個雙引號表示。 不建議使用引號識別項，因為它容易與字串常數造成混淆。 盡可能使用分隔的識別碼。 以下是 `<quoted_identifier>` 的範例：  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>模式  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>備註
  
`<pattern>` 必須是評估為字串的運算式。 它會用來做為 LIKE 運算子的模式。      它可以包含下列萬用字元︰  
  
-   `%`︰任何零或多個字元的字串。  
  
-   `_`︰任何單一字元。  
  
## <a name="escapechar"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>備註  

`<escape_char>` 必須是評估為字串為 1 的運算式。 它會用來做為 LIKE 運算子的逸出字元。  
  
 例如，`property LIKE 'ABC\%' ESCAPE '\'` 符合 `ABC%` 而不是以 `ABC`開頭的字串。  
  
## <a name="constant"></a>常數  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>引數  
  
-   `<integer_constant>` 是數字的字串，不會以引號括住且不包含小數點。 值會在內部儲存為 `System.Int64`，並遵循相同的範圍。  
  
     這些是長常數的範例：  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` 是數字的字串，不會以引號括住，且包含小數點。 值會在內部儲存為 `System.Double`，並遵循相同的範圍/精確度。  
  
     在未來版本中，這個數字可能會以不同的資料類型儲存，以支援實際數字的語意，因此您不應依賴 `<decimal_constant>` 的基本資料型別是 `System.Double`。  
  
     以下是十進位常數的範例：  
  
    ```  
    1894.1204  
    2.0  
    ```  
  
-   `<approximate_number_constant>` 是以科學記號標記法撰寫的數字。 值會在內部儲存為 `System.Double`，並遵循相同的範圍/精確度。 以下是大約數字常數的範例：  
  
    ```  
    101.5E5  
    0.5E-2  
    ```  
  
## <a name="booleanconstant"></a>boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### <a name="remarks"></a>備註  

布林值常數由關鍵字 **TRUE** 或 **FALSE** 代表。 值會儲存為 `System.Boolean`。  
  
## <a name="stringconstant"></a>string_constant  
  
```  
<string_constant>  
```  
  
### <a name="remarks"></a>備註  

字串常數會以單引號括住，且包含任何有效的 Unicode 字元。 內嵌在字串常數中的單引號會以兩個單引號表示。  
  
## <a name="function"></a>函式  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### <a name="remarks"></a>備註
  
`newid()` 函式會傳回由 `System.Guid.NewGuid()` 方法所產生的 **System.Guid**。  
  
`property(name)` 函式會傳回 `name`所參考的屬性值 。 `name` 可以是任何會傳回字串值的有效運算式。  
  
## <a name="considerations"></a>考量
  
請考慮下列 [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) 語意：  
  
-   屬性名稱不區分大小寫。  
  
-   後接 C# 的運算子儘可能隱含轉換語意。  
  
-   系統屬性為 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 執行個體中公開的公用屬性。  
  
    考量以下 `IS [NOT] NULL` 語意：  
  
    -   如果屬性不存在或屬性的值是 `null`，`property IS NULL` 會評估為 `true`。  
  
### <a name="property-evaluation-semantics"></a>屬性評估語意  
  
-   嘗試評估不存在的系統屬性會擲回 [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) 例外狀況。  
  
-   不存在的屬性會在內部評估為**未知**。  
  
 算術運算子的未知評估︰  
  
-   針對二進位運算子，如果左和/或右邊的運算元評估為**未知**，則結果為**未知**。  
  
-   針對一元運算子，如果運算元評估為**未知**，則結果為**未知**。  
  
 二進位比較運算子的未知評估︰  
  
-   如果左和/或右邊的運算元評估為**未知**，則結果為**未知**。  
  
 `[NOT] LIKE` 中的未知評估：  
  
-   如果任何運算元評估為**未知**，則結果為**未知**。  
  
 `[NOT] IN` 中的未知評估：  
  
-   如果左運算元評估為**未知**，則結果為**未知**。  
  
 **AND** 運算子的未知評估︰  
  
```  
+---+---+---+---+  
|AND| T | F | U |  
+---+---+---+---+  
| T | T | F | U |  
+---+---+---+---+  
| F | F | F | F |  
+---+---+---+---+  
| U | U | F | U |  
+---+---+---+---+  
```  
  
 **OR** 運算子的未知評估︰  
  
```  
+---+---+---+---+  
|OR | T | F | U |  
+---+---+---+---+  
| T | T | T | T |  
+---+---+---+---+  
| F | T | F | U |  
+---+---+---+---+  
| U | T | U | U |  
+---+---+---+---+  
```  
  
### <a name="operator-binding-semantics"></a>運算子繫結語意
  
-   `>`、`>=`、`<`、`<=`、`!=` 和 `=` 等比較運算子會遵循與資料類型升級和隱含轉換中的 C# 運算子繫結相同的語意。  
  
-   `+`、`-`、`*`、`/` 和 `%` 等算術運算子會遵循與資料類型升級和隱含轉換中的 C# 運算子繫結相同的語意。

## <a name="next-steps"></a>後續步驟

- [SQLFilter 類別 (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
- [SQLFilter 類別 (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlfilter)
- [SQLRuleAction 類別](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction)
