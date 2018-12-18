---
title: 建立、編輯或擴充邏輯應用程式定義的 JSON - Azure Logic Apps | Microsoft Docs
description: 在 Azure Logic Apps 中撰寫及擴充邏輯應用程式定義的 JSON
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, jehollan, LADocs
ms.assetid: d565873c-6b1b-4057-9250-cf81a96180ae
ms.topic: article
ms.date: 01/01/2018
ms.openlocfilehash: 1f2e136810194ad044255f9d129b5c03549221b9
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128655"
---
# <a name="create-edit-or-extend-json-for-logic-app-definitions-in-azure-logic-apps"></a>在 Azure Logic Apps 中建立、編輯或擴充邏輯應用程式定義的 JSON

當您在 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 中使用自動化工作流程建立企業整合解決方案時，基礎邏輯應用程式定義會針對其描述和驗證，使用簡單的宣告式 JavaScript 物件標記法 (JSON) 及[工作流程定義語言 (WDL) 結構描述](../logic-apps/logic-apps-workflow-definition-language.md)。 這些格式讓邏輯應用程式定義更容易閱讀及了解，且無須很多程式碼方面的知識。 如果您想要自動化邏輯應用程式的建立及部署，您可以將邏輯應用程式定義納入 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-overview.md#template-deployment)中的 [Azure 資源](../azure-resource-manager/resource-group-overview.md)。 然後您可以使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.logicapp)、[Azure CLI](../azure-resource-manager/resource-group-template-deploy-cli.md) 或 [Azure Logic Apps REST API](https://docs.microsoft.com/rest/api/logic/) 來建立、管理及部署邏輯應用程式。

若要以 JSON 編寫邏輯應用程式定義，可以在 Azure 入口網站或 Visual Studio 中開啟程式碼檢視編輯器，或將定義複製到任何您想要的編輯器中。 如果您還不熟悉邏輯應用程式，請先檢閱[如何建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

> [!NOTE]
> 有些 Azure Logic Apps 功能只支援 JSON，不支援 Logic Apps 設計工具，例如在邏輯應用程式定義中定義參數和多個觸發程序。 因此針對這些工作，您必須使用程式碼檢視或其他編輯器。

## <a name="edit-json---azure-portal"></a>編輯 JSON - Azure 入口網站

1. 登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。

2. 在左側功能表中，選擇 [所有服務]。 在搜尋方塊中尋找「邏輯應用程式」，然後從結果中選取您的邏輯應用程式。

3. 在邏輯應用程式功能表的 [開發工具] 底下，選取 [邏輯應用程式程式碼檢視]。

   程式碼檢視編輯器隨即開啟，並以 JSON 格式顯示您的邏輯應用程式定義。

## <a name="edit-json---visual-studio"></a>編輯 JSON - Visual Studio

在 Visual Studio 中編寫邏輯應用程式定義之前，請先確定您已經[安裝必要的工具](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites)。 若要使用 Visual Studio 建立邏輯應用程式，請檢閱[快速入門：使用 Azure Logic Apps 自動化工作和程序 - Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)。

在 Visual Studio 中，您可以開啟直接從 Azure 入口網站建立或部署的邏輯應用程式，或開啟 Visual Studio 中以 Azure Resource Manager 專案形式建立或部署的邏輯應用程式。

1. 開啟包含邏輯應用程式的 Visual Studio 解決方案或 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)專案。

2. 尋找並開啟邏輯應用程式的定義，根據預設，該定義在 [Resource Manager 範本](../azure-resource-manager/resource-group-overview.md#template-deployment)中會以 **LogicApp.json** 的名稱顯示。 您可以使用及自訂此範本，以部署至不同的環境。

3. 開啟邏輯應用程式定義和範本的捷徑功能表。 選取 [以邏輯應用程式設計工具開啟]。

   ![開啟 Visual Studio 解決方案中的邏輯應用程式](./media/logic-apps-author-definitions/open-logic-app-designer.png)

4. 在設計工具底部，選擇 [程式碼檢視]。 

   程式碼檢視編輯器隨即開啟，並以 JSON 格式顯示您的邏輯應用程式定義。

5. 若要返回設計工具檢視，請在程式碼檢視編輯器的底部，選擇 [設計]。

## <a name="parameters"></a>參數

參數可讓您在整個邏輯應用程式中重複使用值，且適用於取代您可能經常變更的值。 例如，如果您想要在多個位置中使用同一個電子郵件地址，您應將該電子郵件地址定義為參數。 

當您必須在不同的環境中覆寫參數時，參數也很有用。深入了解[用於部署的參數](#deployment-parameters)和 [Azure Logic Apps 文件的 REST API](https://docs.microsoft.com/rest/api/logic)。

> [!NOTE]
> 參數僅適用於程式碼檢視。

在[第一個範例邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)中，您所建立的工作流程會在網站的 RSS 摘要中出現新文章時傳送電子郵件。 摘要的 URL 為硬式編碼，因此這個範例會示範如何將查詢值取代為參數，進而更輕鬆地變更摘要的 URL。

1. 在程式碼檢視中，尋找 `parameters : {}` 物件，並新增 `currentFeedUrl` 物件：

   ``` json
     "currentFeedUrl" : {
      "type" : "string",
            "defaultValue" : "http://rss.cnn.com/rss/cnn_topstories.rss"
   }
   ```

2. 在 `When_a_feed-item_is_published` 動作中，尋找 `queries` 區段，然後以 `"feedUrl": "#@{parameters('currentFeedUrl')}"` 取代查詢值。 

   **之前**
   ``` json
   }
      "queries": {
          "feedUrl": "https://s.ch9.ms/Feeds/RSS"
       }
   },   
   ```

   **之後**
   ``` json
   }
      "queries": {
          "feedUrl": "#@{parameters('currentFeedUrl')}"
       }
   },   
   ```

   若要加入兩或多個字串，您也可以使用 `concat` 函式。 
   例如，`"@concat('#',parameters('currentFeedUrl'))"` 的運作方式與上述範例相同。

3.  完成之後，請選擇 [儲存]。 

現在您可以透過 `currentFeedURL` 物件傳遞不同的 URL，藉以變更網站的 RSS 摘要。

<a name="deployment-parameters"></a>

## <a name="deployment-parameters-for-different-environments"></a>不同環境的部署參數

通常，部署生命週期具有開發、預備及生產的環境。 例如，您可以在所有這些環境中使用相同的邏輯應用程式定義，但使用不同的資料庫。 同樣地，建議您在不同的區域使用相同的定義，以發揮高可用性，但希望每個邏輯應用程式執行個體使用該區域的資料庫。 

> [!NOTE] 
> 這種情況與在執行階段採用參數不同，您反而應該使用 `trigger()` 函式。

以下是基本定義：

``` json
{
    "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "uri": {
            "type": "string"
        }
    },
    "triggers": {
        "request": {
          "type": "request",
          "kind": "http"
        }
    },
    "actions": {
        "readData": {
            "type": "Http",
            "inputs": {
                "method": "GET",
                "uri": "@parameters('uri')"
            }
        }
    },
    "outputs": {}
}
```
在邏輯應用程式的實際 `PUT` 要求中，您可以提供參數 `uri`。 在每個環境中，您可以提供不同的值給 `connection` 參數。 因為預設值不存在，邏輯應用程式承載需要此參數︰

``` json
{
    "properties": {},
        "definition": {
          /// Use the definition from above here
        },
        "parameters": {
            "connection": {
                "value": "https://my.connection.that.is.per.enviornment"
            }
        }
    },
    "location": "westus"
}
``` 

若要深入了解，請參閱 [Azure Logic Apps 文件的 REST API](https://docs.microsoft.com/rest/api/logic/)。

## <a name="process-strings-with-functions"></a>使用函式處理字串

Logic Apps 具有各種函式可處理字串。 例如，假設您需要將公司名稱從訂單傳遞至另一個系統。 不過，您不確定字元編碼的正確處理方式。 您可以在這個字串上執行 base64 編碼，但若要避免在 URL 中逸出，您可改為取代數個字元。 此外，公司名稱只需要一個子字串，因為不會用到前五個字元。 

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder1",
        "companyName": "NAME=Contoso"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "request": {
      "type": "Request",
      "kind": "Http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "http://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
      }
    }
  },
  "outputs": {}
}
```

下列步驟說明此範例從內部到外部處理這個字串的方式：

``` 
"uri": "http://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
```

1. 取得公司名稱的 [`length()`](../logic-apps/logic-apps-workflow-definition-language.md)，以便取得字元總數。

2. 若要取得較短的字串，請減去 `5`。

3. 現在可取得 [`substring()`](../logic-apps/logic-apps-workflow-definition-language.md)。 從索引 `5` 開始，並移至字串的其餘部分。

4. 將這個子字串轉換成 [`base64()`](../logic-apps/logic-apps-workflow-definition-language.md) 字串。

5. 現在以 `-` 字元 [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) 所有 `+` 字元。

6. 最後，以 `_` 字元 [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) 所有 `/` 字元。

## <a name="map-list-items-to-property-values-then-use-maps-as-parameters"></a>將清單項目對應到屬性值，然後使用對應作為參數

若要根據屬性值取得不同的結果，您可以建立可比對每個屬性值與結果的對應，然後使用該對應作為參數。 

例如，此工作流程會將某些類別定義為參數，以及定義可比對這些類別與特定 URL 的對應。 首先，工作流程會取得文章清單。 接著，工作流程會使用此對應來尋找符合每篇文章之類別的 URL。

*   [`intersection()`](../logic-apps/logic-apps-workflow-definition-language.md) 函式會檢查類別是否符合已定義的知名類別。

*   取得相符的類別之後，此範例會使用方括號來提取對應中的項目：`parameters[...]`

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "specialCategories": {
      "defaultValue": [
        "science",
        "google",
        "microsoft",
        "robots",
        "NSA"
      ],
      "type": "Array"
    },
    "destinationMap": {
      "defaultValue": {
        "science": "http://www.nasa.gov",
        "microsoft": "https://www.microsoft.com/en-us/default.aspx",
        "google": "https://www.google.com",
        "robots": "https://en.wikipedia.org/wiki/Robot",
        "NSA": "https://www.nsa.gov/"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "Request",
      "kind": "http"
    }
  },
  "actions": {
    "getArticles": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://ajax.googleapis.com/ajax/services/feed/load?v=1.0&q=http://feeds.wired.com/wired/index"
      }
    },
    "forEachArticle": {
      "type": "foreach",
      "foreach": "@body('getArticles').responseData.feed.entries",
      "actions": {
        "ifGreater": {
          "type": "if",
          "expression": "@greater(length(intersection(item().categories, parameters('specialCategories'))), 0)",
          "actions": {
            "getSpecialPage": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('destinationMap')[first(intersection(item().categories, parameters('specialCategories')))]"
              }
            }
          }
        }
      },
      "runAfter": {
        "getArticles": [
          "Succeeded"
        ]
      }
    }
  }
}
```

## <a name="get-data-with-date-functions"></a>使用 Date 函式取得資料

若要從原生不支援「觸發程序」的資料來源取得資料，您可改為使用 Date 函式來處理時間和日期。 例如，這個運算式會尋找這個工作流程從內部到外部的步驟需要花費多少時間：

``` json
"expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
```

1. 從 `order` 動作，擷取 `startTime`。 
2. 使用 `utcNow()` 取得目前時間。
3. 減去一秒：

   [`addseconds(..., -1)`](../logic-apps/logic-apps-workflow-definition-language.md) 

   您可以使用其他的時間單位，例如 `minutes` 或 `hours`。 

3. 現在，您可以比較這兩個值。 

   如果第一個值小於第二個值，則自從訂單最初提交以來已超過一秒。

若要將日期格式化，您可以使用字串格式器。 例如，若要取得 RFC1123，請使用 [`utcnow('r')`](../logic-apps/logic-apps-workflow-definition-language.md)。 深入了解[日期格式化](../logic-apps/logic-apps-workflow-definition-language.md)。

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder-id"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "request",
      "kind": "http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "http://www.example.com/?id=@{parameters('order').id}"
      }
    },
    "ifTimingWarning": {
      "type": "If",
      "expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
      "actions": {
        "timingWarning": {
          "type": "Http",
          "inputs": {
            "method": "GET",
            "uri": "http://www.example.com/?recordLongOrderTime=@{parameters('order').id}&currentTime=@{utcNow('r')}"
          }
        }
      },
      "runAfter": {
        "order": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```


## <a name="next-steps"></a>後續步驟

* [根據條件 (條件陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [根據不同的值 (Switch 陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [執行並重複步驟 (迴圈)](../logic-apps/logic-apps-control-flow-loops.md)
* [執行或合併平行步驟 (分支)](../logic-apps/logic-apps-control-flow-branches.md)
* [根據分組的動作狀態執行步驟 (範圍)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* 深入了解 [Azure Logic Apps 的工作流程定義語言結構描述](../logic-apps/logic-apps-workflow-definition-language.md)
* 深入了解 [Azure Logic Apps 的工作流程動作和觸發程序](../logic-apps/logic-apps-workflow-actions-triggers.md)