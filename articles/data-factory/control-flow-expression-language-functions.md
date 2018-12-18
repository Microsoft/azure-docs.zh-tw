---
title: Azure Data Factory 中的運算式和函式 | Microsoft Docs
description: 本文提供可用來建立資料處理站實體之運算式和函式的相關資訊。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
ms.openlocfilehash: 5cdaba2a280221fa5fa9274ebfa6cafa18e7690c
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39055010"
---
# <a name="expressions-and-functions-in-azure-data-factory"></a>Azure Data Factory 中的運算式和函式
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-functions-variables.md)
> * [目前的版本](control-flow-expression-language-functions.md)

本文提供 Azure Data Factory 支援之運算式和函式的詳細資料。 

## <a name="introduction"></a>簡介
定義中的 JSON 值可以是常值，或是在執行階段評估的運算式。 例如︰  
  
```json
"name": "value"
```

 (或)  
  
```json
"name": "@pipeline().parameters.password"
```

## <a name="expressions"></a>運算式  
運算式可以出現在 JSON 字串值中的任何一處，並一律產生另一個 JSON 值。 當 JSON 值為運算式時，可透過移除 \@ 符號來擷取運算式的主體。 如果需要的常值字串開頭為 \@，就必須使用 \@\@ 逸出。 下列範例顯示如何評估運算式。  
  
|JSON 值|結果|  
|----------------|------------|  
|"parameters"|傳回字元 'parameters'。|  
|"parameters[1]"|傳回字元 'parameters[1]'。|  
|"\@\@"|傳回包含 '\@' 的 1 個字元字串。|  
|" \@"|傳回包含 '\@' 的 2 個字元字串。|  
  
 使用稱為「字串插補」的功能，運算式也可以出現在字串內，其中運算式會包含在 `@{ ... }` 內。 例如：`"name" : "First Name: @{pipeline().parameters.firstName} Last Name: @{pipeline().parameters.lastName}"`  
  
 使用字串插補時，結果一律為字串。 假設我將 `myNumber` 定義為 `42`，將 `myString` 定義為 `foo`：  
  
|JSON 值|結果|  
|----------------|------------|  
|"\@pipeline().parameters.myString"| 傳回 `foo` 做為字串。|  
|"\@{pipeline().parameters.myString}"| 傳回 `foo` 做為字串。|  
|"\@pipeline().parameters.myNumber"| 傳回 `42` 做為「編號」。|  
|"\@{pipeline().parameters.myNumber}"| 傳回 `42` 做為「字串」。|  
|"Answer is: @{pipeline().parameters.myNumber}"| 傳回字串 `Answer is: 42`。|  
|"\@concat('Answer is: ', string(pipeline().parameters.myNumber))"| 傳回字串 `Answer is: 42`|  
|"Answer is: \@\@{pipeline().parameters.myNumber}"| 傳回字串 `Answer is: @{pipeline().parameters.myNumber}`。|  
  
### <a name="examples"></a>範例

#### <a name="a-dataset-with-a-parameter"></a>具有參數的資料集
在以下範例中，BlobDataset 會採用一個名為 **path** 的參數。 其值會藉由下列運算式，用來設定 **folderPath** 屬性的值：`@{dataset().path}`。 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "@dataset().path"
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

#### <a name="a-pipeline-with-a-parameter"></a>具有參數的管線
在以下範例中，管線會採用 **inputPath** 和 **outputPath** 參數。 這些參數的值會用來設定參數化 Blob 資料集的 **path**。 這裡使用的語法為：`pipeline().parameters.parametername`. 

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToBlob",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.inputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.outputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                }
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath": {
                "type": "String"
            }
        }
    }
}
```
  
## <a name="functions"></a>Functions  
 您可以在運算式內呼叫函式。 下列各節提供可在運算式中使用之函式的相關資訊。  

## <a name="string-functions"></a>字串函數  
 下列函式只適用於字串。 您也可以在字串上使用許多集合函式。  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|concat|結合任何數目的字串。 例如，如果 parameter1 是 `foo,`，下列運算式會傳回 `somevalue-foo-somevalue`：`concat('somevalue-',pipeline().parameters.parameter1,'-somevalue')`<br /><br /> **參數編號**：1 ... n<br /><br /> **名稱**：字串 n<br /><br /> **描述**︰必要。 要結合至單一字串的字串。|  
|substring|從字串傳回字元的子集。 例如，下列運算式：<br /><br /> `substring('somevalue-foo-somevalue',10,3)`<br /><br /> 傳回：<br /><br /> `foo`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 從中採用子字串的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：開始索引<br /><br /> **描述**︰必要。 子字串在參數 1 中開始的索引。<br /><br /> **參數編號**：3<br /><br /> **名稱**：長度<br /><br /> **描述**︰必要。 子字串的長度。|  
|取代|以指定的字串取代字串。 例如，運算式：<br /><br /> `replace('the old string', 'old', 'new')`<br /><br /> 傳回：<br /><br /> `the new string`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。  若參數 2 在參數 1 中找到時，針對參數 2 搜尋和使用參數 3 更新的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：舊字串<br /><br /> **描述**︰必要。 在參數 1 中找到相符項目時，以參數 3 取代的字串<br /><br /> **參數編號**：3<br /><br /> **名稱**：新字串<br /><br /> **描述**︰必要。 在參數 1 中找到相符項目時，用來取代參數 2 中字串的字串。|  
|GUID| 產生全域唯一字串 (亦稱為 GUID)。 例如，可能會產生下列輸出 `c2ecc88d-88c8-4096-912c-d6f2e2b138ce`：<br /><br /> `guid()`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 單一格式規範，指出[如何格式化這個 Guid 值](https://msdn.microsoft.com/library/97af8hh4%28v=vs.110%29.aspx)。 格式參數可以是 "N"、"D"、"B"、"P" 或 "X"。 如果未提供格式，則會使用 "D"。|  
|toLower|將字串轉換為小寫。 例如，下列範例會傳回 `two by two is four`：`toLower('Two by Two is Four')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要轉換成小寫的字串。 如果字串中的字元沒有對等小寫，它在傳回的字串中會保持不變。|  
|toUpper|將字串轉換為大寫。 例如，下列運算式會傳回 `TWO BY TWO IS FOUR`：`toUpper('Two by Two is Four')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要轉換成大寫的字串。 如果字串中的字元沒有對等大寫，它在傳回的字串中會保持不變。|  
|indexof|在不區分大小寫字串內尋找值的索引。 例如，下列運算式會傳回 `7`：`indexof('hello, world.', 'world')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 可以包含值的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要搜尋索引的值。|  
|lastindexof|在不區分大小寫字串內尋找值的最後一個索引。 例如，下列運算式會傳回 `3`：`lastindexof('foofoo', 'foo')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 可以包含值的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要搜尋索引的值。|  
|startswith|檢查字串是否以不區分大小寫的值開頭。 例如，下列運算式會傳回 `true`：`startswith('hello, world', 'hello')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 可以包含值的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 字串可能開始的值。|  
|endswith|檢查字串是否以不區分大小寫的值結尾。 例如，下列運算式會傳回 `true`：`endswith('hello, world', 'world')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 可以包含值的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 字串可能結尾的值。|  
|split|使用分隔符號分隔字串。 例如，下列運算式會傳回 `["a", "b", "c"]`：`split('a;b;c',';')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 分割的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 分隔符號。|  
  
  
## <a name="collection-functions"></a>集合函式  
 這些函式會在集合上操作，例如在陣列、字串上，有時候會在字典上。  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|contains|如果字典包含索引鍵、清單包含值，或字串包含子字串，則傳回 true。 例如，下列運算式會傳回 `true:``contains('abacaba','aca')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰集合中<br /><br /> **描述**︰必要。 在其中搜尋的集合。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰尋找物件<br /><br /> **描述**︰必要。 要在**集合內**尋找的物件。|  
|length|傳回陣列或字串中的元素數目。 例如，下列運算式會傳回 `3`：`length('abc')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 要用來取得長度的集合。|  
|empty|如果物件、陣列或字串是空的，則傳回 true。 例如，下列運算式會傳回 `true`：<br /><br /> `empty('')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 集合為空白時要檢查的集合。|  
|交集|傳回單一陣列或物件，在陣列或物件之間具有傳入的通用元素。 例如，此函式會傳回 `[1, 2]`：<br /><br /> `intersection([1, 2, 3], [101, 2, 1, 10],[6, 8, 1, 2])`<br /><br /> 函式的參數可以是一組物件或一組陣列 (不是混合)。 如果有兩個物件具有相同名稱，具有該名稱的最後一個物件會出現在最終物件。<br /><br /> **參數編號**：1 ... n<br /><br /> **名稱**：集合 n<br /><br /> **描述**︰必要。 要評估的集合。 物件必須在傳入以出現在結果中的所有集合中。|  
|union|傳回單一陣列或物件，在陣列或物件中具有傳入的所有元素。 例如，此函式會傳回 `[1, 2, 3, 10, 101]:`<br /><br /> :  `union([1, 2, 3], [101, 2, 1, 10])`<br /><br /> 函式的參數可以是一組物件或一組陣列 (不是混合)。 如果最終輸出中有兩個物件具有相同名稱，具有該名稱的最後一個物件會出現在最終物件。<br /><br /> **參數編號**：1 ... n<br /><br /> **名稱**：集合 n<br /><br /> **描述**︰必要。 要評估的集合。 會出現在任何集合的物件，就會出現在結果中。|  
|first|傳回傳入之陣列或字串中的第一個元素。 例如，此函式會傳回 `0`：<br /><br /> `first([0,2,3])`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 要從中採用第一個物件的集合。|  
|last|傳回傳入之陣列或字串中的最後一個元素。 例如，此函式會傳回 `3`：<br /><br /> `last('0123')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 要從中採用最後一個物件的集合。|  
|take|傳回傳入之陣列或字串中的第一個 **Count** 元素，例如，此函式會傳回 `[1, 2]`：`take([1, 2, 3, 4], 2)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 從其中採用第一個 **Count** 物件的集合。<br /><br /> **參數編號**：2<br /><br /> **名稱**：計數<br /><br /> **描述**︰必要。 從**集合**採用的物件數目。 必須是正整數。|  
|skip|傳回從索引**計數**開始之陣列中的元素，例如，此函式會傳回 `[3, 4]`：<br /><br /> `skip([1, 2 ,3 ,4], 2)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合<br /><br /> **描述**︰必要。 略過第一個 **Count** 物件的集合。<br /><br /> **參數編號**：2<br /><br /> **名稱**：計數<br /><br /> **描述**︰必要。 從**集合**前面移除的物件數目。 必須是正整數。|  
  
## <a name="logical-functions"></a>邏輯函式  
 這些函式在條件內相當有用，而且可以用來評估任何類型的邏輯。  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|equals|如果兩個值相等，則傳回 true。 例如，如果 parameter1 是 foo，下列運算式會傳回 `true`：`equals(pipeline().parameters.parameter1), 'foo')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：物件 1<br /><br /> **描述**︰必要。 要與**物件 2** 比較的物件。<br /><br /> **參數編號**：2<br /><br /> **名稱**：物件 2<br /><br /> **描述**︰必要。 要與**物件 1** 比較的物件。|  
|less|如果第一個引數小於第二個引數，則傳回 true。 請注意，值類型只能是整數、浮點數或字串。 例如，下列運算式會傳回 `true`：`less(10,100)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：物件 1<br /><br /> **描述**︰必要。 物件，以檢查其是否小於**物件 2**。<br /><br /> **參數編號**：2<br /><br /> **名稱**：物件 2<br /><br /> **描述**︰必要。 物件，以檢查其是否大於**物件 1**。|  
|lessOrEquals|如果第一個引數小於或等於第二個引數，則傳回 true。 請注意，值類型只能是整數、浮點數或字串。 例如，下列運算式會傳回 `true`：`lessOrEquals(10,10)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：物件 1<br /><br /> **描述**︰必要。 物件，以檢查其是否小於或等於**物件 2**。<br /><br /> **參數編號**：2<br /><br /> **名稱**：物件 2<br /><br /> **描述**︰必要。 物件，以檢查其是否大於或等於**物件 1**。|  
|greater|如果第一個引數大於第二個引數，則傳回 true。 請注意，值類型只能是整數、浮點數或字串。 例如，下列運算式會傳回 `false`：`greater(10,10)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：物件 1<br /><br /> **描述**︰必要。 物件，以檢查其是否大於**物件 2**。<br /><br /> **參數編號**：2<br /><br /> **名稱**：物件 2<br /><br /> **描述**︰必要。 物件，以檢查其是否小於**物件 1**。|  
|greaterOrEquals|如果第一個引數大於或等於第二個引數，則傳回 true。 請注意，值類型只能是整數、浮點數或字串。 例如，下列運算式會傳回 `false`：`greaterOrEquals(10,100)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：物件 1<br /><br /> **描述**︰必要。 物件，以檢查其是否大於或等於**物件 2**。<br /><br /> **參數編號**：2<br /><br /> **名稱**：物件 2<br /><br /> **描述**︰必要。 物件，以檢查其是否小於或等於**物件 1**。|  
|和|如果兩個參數都為 true，則傳回 true。 兩個引數必須是布林值。 下列範例會傳回 `false`：`and(greater(1,10),equals(0,0))`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰布林值 1<br /><br /> **描述**︰必要。 第一個引數必須是 `true`。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰布林值 2<br /><br /> **描述**︰必要。 第二個引數必須是 `true`。|  
|或|如果任一個參數為 true，則傳回 true。 兩個引數必須是布林值。 下列範例會傳回 `true`：`or(greater(1,10),equals(0,0))`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰布林值 1<br /><br /> **描述**︰必要。 第一個引數可能是 `true`。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰布林值 2<br /><br /> **描述**︰必要。 第二個引數可能是 `true`。|  
|否|如果參數為 `false`，則傳回 true。 兩個引數必須是布林值。 下列範例會傳回 `true`：`not(contains('200 Success','Fail'))`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰布林值<br /><br /> **描述**：如果參數為 `false`，則傳回 true。 兩個引數必須是布林值。 下列範例會傳回 `true`：`not(contains('200 Success','Fail'))`|  
|if|根據運算式是否在 `true` 或 `false` 提供結果傳回指定值。  例如，下列範例會傳回 `"yes"`：`if(equals(1, 1), 'yes', 'no')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰運算式<br /><br /> **描述**︰必要。 布林值，決定運算式傳回哪一個值。<br /><br /> **參數編號**：2<br /><br /> **名稱**：True<br /><br /> **描述**︰必要。 如果運算式為 `true` 時要傳回的值。<br /><br /> **參數編號**：3<br /><br /> **名稱**：False<br /><br /> **描述**︰必要。 如果運算式為 `false` 時要傳回的值。|  
  
## <a name="conversion-functions"></a>轉換函式  
 這些函式是用來在每個原生類型語言之間轉換︰  
  
-   字串  
  
-   integer  
  
-   float  
  
-   布林值  
  
-   陣列  
  
-   字典  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|int|將參數轉換成整數。 例如，下列運算式會傳回 100 做為數字，而不是字串︰`int('100')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成整數的值。|  
|字串|將參數轉換成字串。 例如，下列運算式會傳回 `'10'`：`string(10)`您也可以將物件轉換為字串，例如，如果 **foo** 參數是具有 `bar : baz` 屬性的物件，則下列範例會傳回 `{"bar" : "baz"}` `string(pipeline().parameters.foo)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成字串的值。|  
|json|將參數轉換成 JSON 類型值， 並且與 string() 相反。 例如，下列運算式會傳回 `[1,2,3]` 做為數字，而不是字串︰<br /><br /> `json('[1,2,3]')`<br /><br /> 同樣地，您可以將字串轉換成物件。 例如，`json('{"bar" : "baz"}')` 會傳回：<br /><br /> `{ "bar" : "baz" }`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 轉換成原生類型值的字串。<br /><br /> JSON 函式也支援 XML 輸入。 例如，參數值︰<br /><br /> `<?xml version="1.0"?> <root>   <person id='1'>     <name>Alan</name>     <occupation>Engineer</occupation>   </person> </root>`<br /><br /> 轉換為下列 JSON：<br /><br /> `{ "?xml": { "@version": "1.0" },   "root": {     "person": [     {       "@id": "1",       "name": "Alan",       "occupation": "Engineer"     }   ]   } }`|  
|float|將參數引數轉換成浮點數。 例如，下列運算式會傳回 `10.333`：`float('10.333')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成浮點數的值。|  
|布林|將參數轉換成布林值。 例如，下列運算式會傳回 `false`：`bool(0)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成布林值的值。|  
|coalesce|傳回傳入的引數中第一個非 null 的物件。 注意︰空字串不是 null。 例如，如果未定義參數 1 和 2，此範例會傳回 `fallback`：`coalesce(pipeline().parameters.parameter1', pipeline().parameters.parameter2 ,'fallback')`<br /><br /> **參數編號**：1 ... n<br /><br /> **名稱**︰物件 n<br /><br /> **描述**︰必要。 要檢查其是否有 `null` 的物件。|  
|base64|傳回輸入字串的 base64 表示法。 例如，下列運算式會傳回 `c29tZSBzdHJpbmc=`：`base64('some string')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰字串 1<br /><br /> **描述**︰必要。 要編碼為 base64 表示法的字串。|  
|base64ToBinary|傳回 base64 編碼字串的二進位表示法。 例如，下列運算式會傳回某字串的二進位表示法：`base64ToBinary('c29tZSBzdHJpbmc=')`。<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 base64 編碼的字串。|  
|base64ToString|傳回 based64 編碼字串的字串表示法。 例如，下列運算式會傳回某字串：`base64ToString('c29tZSBzdHJpbmc=')`。<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 base64 編碼的字串。|  
|Binary|傳回值的二進位表示法。  例如，下列運算式會傳回某字串的二進位表示法：`binary(‘some string’).`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成二進位的值。|  
|dataUriToBinary|傳回資料 URI 的二進位表示法。 例如，下列運算式會傳回某字串的二進位表示法：`dataUriToBinary('data:;base64,c29tZSBzdHJpbmc=')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要轉換成二進位表示法的資料 URI。|  
|dataUriToString|傳回資料 URI 的字串表示法。 例如，下列運算式會傳回某字串：`dataUriToString('data:;base64,c29tZSBzdHJpbmc=')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br />**描述**︰必要。 要轉換成字串表示法的資料 URI。|  
|dataUri|傳回值的資料 URI。 例如，下列運算式會傳回資料：`text/plain;charset=utf8;base64,c29tZSBzdHJpbmc=: dataUri('some string')`<br /><br /> **參數編號**：1<br /><br />**名稱**︰值<br /><br />**描述**︰必要。 要轉換成資料 URI 的值。|  
|decodeBase64|傳回輸入 based64 字串的字串表示法。 例如，下列運算式會傳回 `some string`：`decodeBase64('c29tZSBzdHJpbmc=')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**：傳回輸入 based64 字串的字串表示法。|  
|encodeUriComponent|URL 逸出傳入的字串。 例如，下列運算式會傳回 `You+Are%3ACool%2FAwesome`：`encodeUriComponent('You Are:Cool/Awesome')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要從中逸出 URL 不安全字元的字串。|  
|decodeUriComponent|非 URL 逸出傳入的字串。 例如，下列運算式會傳回 `You Are:Cool/Awesome`：`encodeUriComponent('You+Are%3ACool%2FAwesome')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要從中解碼 URL 不安全字元的字串。|  
|decodeDataUri|傳回輸入資料 URI 字串的二進位表示法。 例如，下列運算式會傳回 `some string` 的二進位表示法：`decodeDataUri('data:;base64,c29tZSBzdHJpbmc=')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br /> **描述**︰必要。 要解碼為二進位表示法的 dataURI。|  
|uriComponent|傳回值的 URI 編碼表示法。 例如，下列運算式會傳回 `You+Are%3ACool%2FAwesome: uriComponent('You Are:Cool/Awesome ')`<br /><br /> 參數詳細資料：數字：1，名稱：字串，描述：必要。 要進行 URI 編碼的字串。|  
|uriComponentToBinary|傳回 URI 編碼字串的二進位表示法。 例如，下列運算式會傳回 `You Are:Cool/Awesome` 的二進位表示法：`uriComponentToBinary('You+Are%3ACool%2FAwesome')`<br /><br /> **參數編號**：1<br /><br /> **名稱**：字串<br /><br />**描述**︰必要。 URI 編碼的字串。|  
|uriComponentToString|傳回 URI 編碼字串的字串表示法。 例如，下列運算式會傳回 `You Are:Cool/Awesome`：`uriComponentToString('You+Are%3ACool%2FAwesome')`<br /><br /> **參數編號**：1<br /><br />**名稱**：字串<br /><br />**描述**︰必要。 URI 編碼的字串。|  
|xml|傳回值的 XML 表示法。 例如，下列運算式會傳回以 `'\<name>Alan\</name>'` 表示的 XML 內容：`xml('\<name>Alan\</name>')`。 XML 函式也支援 JSON 物件輸入。 例如，參數 `{ "abc": "xyz" }` 轉換成 XML 內容 `\<abc>xyz\</abc>`<br /><br /> **參數編號**：1<br /><br />**名稱**︰值<br /><br />**描述**︰必要。 要轉換成 XML 的值。|  
|xpath|傳回符合值 (xpath 運算式進行評估) 之 xpath 運算式的 XML 節點陣列。<br /><br />  **範例 1**<br /><br /> 假設參數 ‘p1’ 的值是下列 XML 的字串表示法：<br /><br /> `<?xml version="1.0"?> <lab>   <robot>     <parts>5</parts>     <name>R1</name>   </robot>   <robot>     <parts>8</parts>     <name>R2</name>   </robot> </lab>`<br /><br /> 1.此程式碼：`xpath(xml(pipeline().parameters.p1), '/lab/robot/name')`<br /><br /> 會傳回<br /><br /> `[ <name>R1</name>, <name>R2</name> ]`<br /><br /> 而<br /><br /> 2.此程式碼：`xpath(xml(pipeline().parameters.p1, ' sum(/lab/robot/parts)')`<br /><br /> 會傳回<br /><br /> `13`<br /><br /> <br /><br /> **範例 2**<br /><br /> 指定下列 XML 內容：<br /><br /> `<?xml version="1.0"?> <File xmlns="http://foo.com">   <Location>bar</Location> </File>`<br /><br /> 1.此程式碼：`@xpath(xml(body('Http')), '/*[name()=\"File\"]/*[name()=\"Location\"]')`<br /><br /> 或<br /><br /> 2.此程式碼：`@xpath(xml(body('Http')), '/*[local-name()=\"File\" and namespace-uri()=\"http://foo.com\"]/*[local-name()=\"Location\" and namespace-uri()=\"\"]')`<br /><br /> 傳回<br /><br /> `<Location xmlns="http://foo.com">bar</Location>`<br /><br /> 和<br /><br /> 3.此程式碼：`@xpath(xml(body('Http')), 'string(/*[name()=\"File\"]/*[name()=\"Location\"])')`<br /><br /> 傳回<br /><br /> ``bar``<br /><br /> **參數編號**：1<br /><br />**名稱**：Xml<br /><br />**描述**︰必要。 要評估 XPath 運算式的 XML。<br /><br /> **參數編號**：2<br /><br />**名稱**：XPath<br /><br />**描述**︰必要。 要評估的 XPath 運算式。|  
|array|將參數轉換成陣列。  例如，下列運算式會傳回 `["abc"]`：`array('abc')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰值<br /><br /> **描述**︰必要。 轉換成陣列的值。|
|createArray|從參數建立陣列。  例如，下列運算式會傳回 `["a", "c"]`：`createArray('a', 'c')`<br /><br /> **參數編號**：1 ... n<br /><br /> **名稱**︰任何 n<br /><br /> **描述**︰必要。 要結合到陣列的值。|

## <a name="math-functions"></a>數學函式  
 這些函式可用於任一類型的數字︰**整數**和**浮點數**。  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|新增|傳回兩個數字相加的結果。 例如，此函式會傳回 `20.333`：`add(10,10.333)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：被加數 1<br /><br /> **描述**︰必要。 要加上**被加數 2** 的數字。<br /><br /> **參數編號**：2<br /><br /> **名稱**：被加數 2<br /><br /> **描述**︰必要。 要加上**被加數 1** 的數字。|  
|sub|傳回兩個數字相減的結果。 例如，此函式會傳回：`-0.333`：<br /><br /> `sub(10,10.333)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：被減數<br /><br /> **描述**︰必要。 從**減數**移除的數字。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰減數<br /><br /> **描述**︰必要。 從**被減數**移除的數字。|  
|mul|傳回兩個數字相乘的結果。 例如，下列範例會傳回 `103.33`：<br /><br /> `mul(10,10.333)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰被乘數 1<br /><br /> **描述**︰必要。 要與**被乘數 2** 相乘的數字。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰被乘數 2<br /><br /> **描述**︰必要。 要與**被乘數 1** 相乘的數字。|  
|div|傳回兩個數字相除的結果。 例如，下列範例會傳回 `1.0333`：<br /><br /> `div(10.333,10)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰被除數<br /><br /> **描述**︰必要。 要除以**除數**的數字。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰除數<br /><br /> **描述**︰必要。 **被除數**所除的數字。|  
|mod|傳回兩個數字相除後的餘數結果 (模數)。 例如，下列運算式會傳回 `2`：<br /><br /> `mod(10,4)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰被除數<br /><br /> **描述**︰必要。 要除以**除數**的數字。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰除數<br /><br /> **描述**︰必要。 **被除數**所除的數字。 進行除法之後，會採用餘數。|  
|Min|有兩個不同的模式可以呼叫此函式：`min([0,1,2])` 這裡的 min 採用陣列。 此運算式會傳回 `0`。 或者，此函式可以採用以逗號分隔的值清單：`min(0,1,2)` 此函式也會傳回 0。 注意，所有值都必須是數字，如果參數是陣列，則陣列只能有數字。<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合或值<br /><br /> **描述**︰必要。 它可以是要尋找最小值的值陣列，或集合的第一個值。<br /><br /> **參數編號**：2 ... n<br /><br /> **名稱**︰值 n<br /><br /> **描述**︰選擇性。 如果第一個參數是一個值，則您可以傳遞其他值，然後會傳回所有傳遞值的最小值。|  
|max|有兩個不同的模式可以呼叫此函式：`max([0,1,2])`<br /><br /> 這裡的 max 採用陣列。 此運算式會傳回 `2`。 或者，此函式可以採用以逗號分隔的值清單：`max(0,1,2)` 此函式會傳回 2。 注意，所有值都必須是數字，如果參數是陣列，則陣列只能有數字。<br /><br /> **參數編號**：1<br /><br /> **名稱**：集合或值<br /><br /> **描述**︰必要。 它可以是要尋找最大值的值陣列，或集合的第一個值。<br /><br /> **參數編號**：2 ... n<br /><br /> **名稱**︰值 n<br /><br /> **描述**︰選擇性。 如果第一個參數是一個值，則您可以傳遞其他值，然後會傳回所有傳遞值的最大值。|  
|range| 產生從特定數目開始的整數陣列，您會定義傳回之陣列的長度。 例如，此函式會傳回 `[3,4,5,6]`：<br /><br /> `range(3,4)`<br /><br /> **參數編號**：1<br /><br /> **名稱**：開始索引<br /><br /> **描述**︰必要。 它是陣列中的第一個整數。<br /><br /> **參數編號**：2<br /><br /> **名稱**：計數<br /><br /> **描述**︰必要。 陣列中的整數數目。|  
|rand| 在指定範圍內 (含首尾) 產生隨機整數。 例如，這可能會傳回 `42`：<br /><br /> `rand(-1000,1000)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰最小值<br /><br /> **描述**︰必要。 可以傳回的最小整數。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰最大值<br /><br /> **描述**︰必要。 可以傳回的最大整數。|  
  
## <a name="date-functions"></a>日期函式  
  
|函式名稱|說明|  
|-------------------|-----------------|  
|utcnow|傳回目前的時間戳記做為字串。 例如：`2015-03-15T13:27:36Z`：<br /><br /> `utcnow()`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  
|addseconds|將整數秒數新增至傳入的字串時間戳記。 秒數可以是正數或負數。 結果會是 ISO 8601 格式 ("o") 的字串，除非有提供格式規範。 例如：`2015-03-15T13:27:00Z`：<br /><br /> `addseconds('2015-03-15T13:27:36Z', -36)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰時間戳記<br /><br /> **描述**︰必要。 包含時間的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰秒<br /><br /> **描述**︰必要。 新增的秒數。 可以是負數以減去秒。<br /><br /> **參數編號**：3<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  
|addminutes|將整數分鐘數新增至傳入的字串時間戳記。 分鐘數可以是正數或負數。 結果會是 ISO 8601 格式 ("o") 的字串，除非有提供格式規範。 例如，`2015-03-15T14:00:36Z`：<br /><br /> `addminutes('2015-03-15T13:27:36Z', 33)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰時間戳記<br /><br /> **描述**︰必要。 包含時間的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰分鐘<br /><br /> **描述**︰必要。 要新增的分鐘數。 可以是負數以減去分鐘。<br /><br /> **參數編號**：3<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  
|addhours|將整數時數新增至傳入的字串時間戳記。 時數可以是正數或負數。 結果會是 ISO 8601 格式 ("o") 的字串，除非有提供格式規範。 例如：`2015-03-16T01:27:36Z`：<br /><br /> `addhours('2015-03-15T13:27:36Z', 12)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰時間戳記<br /><br /> **描述**︰必要。 包含時間的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰小時<br /><br /> **描述**︰必要。 要新增的時數。 可以是負數以減去小時。<br /><br /> **參數編號**：3<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  
|adddays|將整數天數新增至傳入的字串時間戳記。 天數可以是正數或負數。 結果會是 ISO 8601 格式 ("o") 的字串，除非有提供格式規範。 例如：`2015-02-23T13:27:36Z`：<br /><br /> `adddays('2015-03-15T13:27:36Z', -20)`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰時間戳記<br /><br /> **描述**︰必要。 包含時間的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰天<br /><br /> **描述**︰必要。 要新增的天數。 可以是負數以減去天。<br /><br /> **參數編號**：3<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  
|formatDateTime|以日期格式傳回字串。 結果會是 ISO 8601 格式 ("o") 的字串，除非有提供格式規範。 例如：`2015-02-23T13:27:36Z`：<br /><br /> `formatDateTime('2015-03-15T13:27:36Z', 'o')`<br /><br /> **參數編號**：1<br /><br /> **名稱**︰日期<br /><br /> **描述**︰必要。 包含日期的字串。<br /><br /> **參數編號**：2<br /><br /> **名稱**︰格式<br /><br /> **描述**︰選擇性。 [單一格式規範字元](https://msdn.microsoft.com/library/az4se3k1%28v=vs.110%29.aspx)或[自訂格式模式](https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx)，指出如何格式化這個時間戳記值。 如果未提供格式，則會使用 ISO 8601 格式 ("o")。|  

## <a name="next-steps"></a>後續步驟
如需可在運算式中使用之系統變數的清單，請參閱[系統變數](control-flow-system-variables.md)。
