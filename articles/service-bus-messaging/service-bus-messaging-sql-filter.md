---
title: Azure 服務匯流排訂用帳戶規則 SQL 篩選語法 |Microsoft Docs
description: 本文提供 SQL 篩選文法的詳細資料。 SQL 篩選器支援 SQL-92 標準的子集。
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 60f3cb6e85cef7a166c353f78cfb50405b962bdd
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98633166"
---
# <a name="subscription-rule-sql-filter-syntax"></a>訂用帳戶規則 SQL 篩選語法

*SQL 篩選* 是服務匯流排主題訂用帳戶的其中一個可用篩選器類型。 它是仰賴在 SQL-92 標準子集上的文字運算式。 篩選條件運算式可搭配 `sqlExpression` Azure Resource Manager 範本中服務匯流排的 ' >sqlfilter ' 屬性元素 `Rule` ，或 Azure CLI [](service-bus-resource-manager-namespace-topic-with-rule.md) `az servicebus topic subscription rule create` 命令的 [`--filter-sql-expression`](/cli/azure/servicebus/topic/subscription/rule#az_servicebus_topic_subscription_rule_create) 引數，以及數個允許管理訂用帳戶規則的 SDK 函數。

服務匯流排 Premium 也透過 JMS 2.0 API 支援 [JMS SQL 訊息選取器語法](https://docs.oracle.com/javaee/7/api/javax/jms/Message.html) 。

  
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
  
-   `<scope>` 是表示 `<property_name>` 範圍的選擇性字串。 有效值為 `sys` 或 `user`。 `sys`值表示系統範圍，其中 `<property_name>` 是[BrokeredMessage 類別](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)的公用屬性名稱。 `user` 表示使用者範圍，其中 `<property_name>` 是 [BrokeredMessage 類別](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 字典的索引鍵。 `user` 如果未指定，範圍即為預設範圍 `<scope>` 。  
  
## <a name="remarks"></a>備註

嘗試存取不存在的系統屬性是一項錯誤，但嘗試存取不存在的使用者屬性不是錯誤。 反之，不存在的使用者屬性會內部評估為未知的值。 未知的值在運算子評估期間會特別處理。  
  
## <a name="property_name"></a>property_name  
  
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
  
`<regular_identifier>`不可以是保留關鍵字。  
  
`<delimited_identifier>` 是使用左/右方括弧 ([]) 括住的任何字串。 右方括弧會以兩個右方括弧代表。 以下為 `<delimited_identifier>`的範例：  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
`<quoted_identifier>` 是以雙引號括住的任何字串。 識別項中的雙引號會以兩個雙引號表示。 不建議使用引號識別碼，因為它很容易與字串常數混淆。 盡可能使用分隔的識別碼。 以下是範例 `<quoted_identifier>` ：  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>模式  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>備註
  
`<pattern>` 必須是評估為字串的運算式。 它是用來做為 LIKE 運算子的模式。      它可以包含下列萬用字元︰  
  
-   `%`︰任何零或多個字元的字串。  
  
-   `_`︰任何單一字元。  
  
## <a name="escape_char"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>備註  

`<escape_char>` 必須是評估為字串為 1 的運算式。 它是用來做為 LIKE 運算子的 escape 字元。  
  
 例如，`property LIKE 'ABC\%' ESCAPE '\'` 符合 `ABC%` 而不是以 `ABC`開頭的字串。  
  
## <a name="constant"></a>常數  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>引數  
  
-   `<integer_constant>` 是數位的字串，不會以引號括住，且不含小數點。 值會在內部儲存為 `System.Int64`，並遵循相同的範圍。  
  
     以下是長常數的範例：  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` 是數位的字串，不會以引號括住，且包含小數點。 值會在內部儲存為 `System.Double`，並遵循相同的範圍/精確度。  
  
     在未來的版本中，這個數位可能會儲存在不同的資料類型，以支援精確的數位語義，因此您不應該依賴基礎資料類型的 `System.Double` 事實 `<decimal_constant>` 。  
  
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
  
## <a name="boolean_constant"></a>boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### <a name="remarks"></a>備註  

布林值常數由關鍵字 **TRUE** 或 **FALSE** 代表。 值會儲存為 `System.Boolean`。  
  
## <a name="string_constant"></a>string_constant  
  
```  
<string_constant>  
```  
  
### <a name="remarks"></a>備註  

字串常數會以單引號括住，且包含任何有效的 Unicode 字元。 內嵌在字串常數中的單引號會以兩個單引號表示。  
  
## <a name="function"></a>函數  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### <a name="remarks"></a>備註
  
函數會傳回 `newid()` `System.Guid` 方法所產生的 `System.Guid.NewGuid()` 。  
  
`property(name)` 函式會傳回 `name`所參考的屬性值 。 `name` 可以是任何會傳回字串值的有效運算式。  
  
## <a name="considerations"></a>考量
  
請考慮下列 [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) 語意：  
  
-   屬性名稱不區分大小寫。  
  
-   後接 C# 的運算子儘可能隱含轉換語意。  
  
-   系統屬性為 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 執行個體中公開的公用屬性。  
  
    考量以下 `IS [NOT] NULL` 語意：  
  
    -   如果屬性不存在或屬性的值是 `null`，`property IS NULL` 會評估為 `true`。  
  
### <a name="property-evaluation-semantics"></a>屬性評估語意  
  
- 嘗試評估不存在的系統屬性會擲回 [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) 例外狀況。  
  
- 不存在的屬性會在內部評估為 **未知**。  
  
  算術運算子的未知評估︰  
  
- 針對二元運算子，如果運算元的左邊或右邊評估為 **未知**，則結果為 **unknown**。  
  
- 針對一元運算子，如果運算元評估為 **未知**，則結果為 **未知**。  
  
  二進位比較運算子的未知評估︰  
  
- 如果運算元的左邊或右邊評估為「 **未知**」，則結果為「 **未知**」。  
  
  `[NOT] LIKE` 中的未知評估：  
  
- 如果任何運算元評估為 **未知**，則結果為 **unknown**。  
  
  `[NOT] IN` 中的未知評估：  
  
- 如果左運算元評估為「 **未知**」，則結果為「 **未知**」。  
  
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


## <a name="examples"></a>範例

### <a name="set-rule-action-for-a-sql-filter"></a>設定 SQL 篩選準則的規則動作

```csharp
// instantiate the ManagementClient
this.mgmtClient = new ManagementClient(connectionString);

// create the SQL filter
var sqlFilter = new SqlFilter("source = @stringParam");

// assign value for the parameter
sqlFilter.Parameters.Add("@stringParam", "orders");

// instantiate the Rule = Filter + Action
var filterActionRule = new RuleDescription
{
    Name = "filterActionRule",
    Filter = sqlFilter,
    Action = new SqlRuleAction("SET source='routedOrders'")
};

// create the rule on Service Bus
await this.mgmtClient.CreateRuleAsync(topicName, subscriptionName, filterActionRule);
```

### <a name="sql-filter-on-a-system-property"></a>系統屬性上的 SQL 篩選

```csharp
sys.Label LIKE '%bus%'`
```

### <a name="using-or"></a>使用或 

```csharp
 sys.Label LIKE '%bus%'` OR `user.tag IN ('queue', 'topic', 'subscription')
```

### <a name="using-in-and-not-in"></a>使用 IN 和 NOT IN

```csharp
StoreId IN('Store1', 'Store2', 'Store3')"

sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'

sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')
```

如需 c # 範例，請參閱 [GitHub 上的主題篩選範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Azure.Messaging.ServiceBus/BasicSendReceiveTutorialwithFilters)。


## <a name="next-steps"></a>下一步

- [SQLFilter 類別 (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
- [SQLFilter 類別 (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlfilter)
- [ (JAVA) 的 >sqlfilter 類別 ](/java/api/com.microsoft.azure.servicebus.rules.SqlFilter)
- [SqlRuleFilter (JavaScript) ](/javascript/api/@azure/service-bus/sqlrulefilter)
- [az 進行的主題訂用帳戶規則](/cli/azure/servicebus/topic/subscription/rule)
- [新 AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)