---
title: Azure 中的 SQLRuleAction 語法參考 | Microsoft Docs
description: SQLRuleAction 文法的詳細資料。
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
ms.date: 09/05/2018
ms.author: spelluru
ms.openlocfilehash: cd5bc1fb959872cfb98236feb5c32517ac7472ac
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394177"
---
# <a name="sqlruleaction-syntax"></a>SQLRuleAction 語法

*SqlRuleAction* 是 [SqlRuleAction](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction) 類別的執行個體，代表對 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)執行的以 SQL 語言為基礎之語法所撰寫的動作集。   
  
本文章列出 SQL 規則動作文法的詳細資料。  
  
```  
<statements> ::=
    <statement> [, ...n]  
  
```  
  
```  
<statement> ::=
    <action> [;]
    Remarks
    -------
    Semicolon is optional.  
  
```  
  
```  
<action> ::=
    SET <property> = <expression>
    REMOVE <property>  
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
  
### <a name="remarks"></a>備註  

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
  
 這表示任何以字母為開頭且後面跟著一或多個底線/字母/數字的字串。  
  
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
  
     以下為長常數的範例：  
  
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
  
布林值常數由關鍵字 `TRUE` 或 `FALSE` 代表。 值會儲存為 `System.Boolean`。  
  
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

- SET 是用來建立新的屬性或更新現有屬性的值。
- REMOVE 是用來移除屬性。
- 當運算式類型和現有的屬性型別不同時，SET 會盡可能執行隱含轉換。
- 如果參考不存在的系統屬性，動作就會失敗。
- 如果參考不存在的使用者屬性，動作就不會失敗。
- 不存在的使用者屬性會內部評估為「不明」，評估運算子時遵循與 [SQLFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) 相同的語意。

## <a name="next-steps"></a>後續步驟

- [SQLRuleAction 類別](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction)
- [SQLFilter 類別](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
