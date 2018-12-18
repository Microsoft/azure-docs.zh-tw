---
title: 根據群組狀態新增執行動作的範圍 - Azure Logic Apps | Microsoft Docs
description: 如何在 Azure Logic Apps 中根據群組動作狀態，建立執行工作流程動作的範圍
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.date: 03/05/2018
ms.topic: article
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 1258175eb3d28d39be8be08498ba8d2e0998aa43
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35298809"
---
# <a name="create-scopes-that-run-workflow-actions-based-on-group-status-in-azure-logic-apps"></a>在 Azure Logic Apps 中根據群組狀態，建立執行工作流程動作的範圍

若要在另一個群組成功或失敗之後才執行步驟，請將那些步驟群組在「範圍」中。 如果您想要將動作組織為邏輯群組、評估該群組的狀態，以及執行以範圍狀態為基礎的動作，此結構將會十分實用。 當範圍中的所有動作都執行完成之後，範圍也會取得自己的狀態。 例如，您可以在想要實作[例外狀況和錯誤處理](../logic-apps/logic-apps-exception-handling.md#scopes)時使用範圍。 

若要檢查範圍的狀態，您可以使用判斷邏輯應用程式執行狀態時所用的相同準則，例如「Succeeded (成功)」、「Failed (失敗)」、「Cancelled (取消)」等。 根據預設，當範圍的所有動作都成功時，範圍的狀態會標示為「Succeeded (成功)」。 但如果範圍中有任何動作失敗或取消，範圍的狀態會標示為「Failed (失敗)」。 如需範圍的限制，請參閱[限制和設定](../logic-apps/logic-apps-limits-and-config.md)。 

例如，以下高階邏輯應用程式會使用範圍來執行特定動作，以及使用條件來檢查範圍的狀態。 如果範圍內的任何動作失敗或非預期地結束，範圍會分別標示為「失敗」或「中止」，而邏輯應用程式會傳送「範圍失敗」的訊息。 如果範圍內的所有動作都成功，則邏輯應用程式會傳送「範圍成功」的訊息。

![設定「排程 - 週期」觸發程序](./media/logic-apps-control-flow-run-steps-group-scopes/scope-high-level.png)

## <a name="prerequisites"></a>先決條件

若要遵循本文中的範例，您需要以下項目：

* Azure 訂用帳戶。 如果您沒有訂用帳戶，請[註冊一個免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 

* 電子郵件帳戶 (來自任何 Logic Apps 所支援的電子郵件提供者)。 此範例使用 Outlook.com。 如果您使用不同的提供者，一般流程維持不變，但您的 UI 會有所不同。

* Bing 地圖金鑰。 若要取得此金鑰，請參閱<a href="https://msdn.microsoft.com/library/ff428642.aspx" target="_blank">取得 Bing 地圖金鑰</a>。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

## <a name="create-sample-logic-app"></a>建立範例邏輯應用程式

首先，建立此範例邏輯應用程式，以便您可以在稍後新增範圍：

![建立範例邏輯應用程式](./media/logic-apps-control-flow-run-steps-group-scopes/finished-sample-app.png)

* **排程 - 週期**觸發程序，以您指定的時間間隔檢查 Bing 地圖服務
* **Bing 地圖 - 取得路線**動作，檢查兩個位置之間的旅行時間
* 條件陳述式，檢查旅遊時間是否超過指定的旅行時間
* 動作，傳送電子郵件給您，告知您目前的旅行時間超過指定的時間

您可以隨時儲存您的邏輯應用程式，所以請經常儲存您的工作。

1. 請先登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a> (如果您尚未登入)。 建立空白邏輯應用程式。

2. 使用下列設定新增**排程 - 週期**觸發程序：**間隔** =「1」，以及**頻率** =「分鐘」

   ![設定「排程 - 週期」觸發程序](./media/logic-apps-control-flow-run-steps-group-scopes/recurrence.png)

   > [!TIP]
   > 若要以視覺化方式來簡化您的檢視，並在設計工具中隱藏每個動作的詳細資料，請在進行這些步驟時摺疊每個動作圖形。

3. 新增 [Bing 地圖 - 取得路線] 動作。 

   1. 如果您尚未與 Bing 地圖連線，系統會要求您建立連線。

      | 設定 | 值 | 說明 |
      | ------- | ----- | ----------- |
      | 連線名稱 | BingMapsConnection | 為您的連線提供一個名稱。 | 
      | **API 金鑰** | <*your-Bing-Maps-key*> | 輸入您先前收到的 Bing 地圖服務金鑰。 | 
      ||||  

   2. 設定 [取得路線] 動作，如下圖中的資料表所示：

      ![設定 [Bing 地圖服務 - 取得路線] 動作](./media/logic-apps-control-flow-run-steps-group-scopes/get-route.png) 

      如需這些參數的詳細資訊，請參閱[計算路線](https://msdn.microsoft.com/library/ff701717.aspx)。

      | 設定 | 值 | 說明 |
      | ------- | ----- | ----------- |
      | **導航點 1** | <起點> | 輸入您的路線起點。 | 
      | **導航點 2** | <終點> | 輸入您路線的目的地。 | 
      | **避開** | None | 輸入路線所要避開的項目，例如高速公路、收費站等等。 如需可能使用的值，請參閱[計算路線](https://msdn.microsoft.com/library/ff701717.aspx)。 | 
      | **最佳化** | timeWithTraffic | 選取可將路線最佳化的參數，例如距離、和使用目前交通資訊的時間等等。 此範例會使用此值："timeWithTraffic" | 
      | **距離單位** | <*your-preference*> | 輸入要計算您路線的距離單位。 此範例會使用此值：「英里」 | 
      | **行進模式** | 開車 | 輸入路線的旅行模式。 此範例會使用此值：「開車」 | 
      | **運輸日期時間** | None | 僅適用於運輸模式。 | 
      | **運輸日期類型** | None | 僅適用於運輸模式。 | 
      ||||  

4. 新增條件以檢查目前交通下的旅行時間是否超過指定時間。 針對此範例，請遵循下圖中的步驟：

   ![建置條件](./media/logic-apps-control-flow-run-steps-group-scopes/build-condition.png)

   1. 以此描述重新命名條件：**如果交通時間超過指定時間**

   2. 從參數清單中，選取 [旅行期間交通] 欄位 (以秒為單位)。 

   3. 針對比較運算子，選取此運算子：**大於**

   4. 針對比較值，輸入 **600**，這會以秒計算，也就是 10 分鐘。

5. 在條件的 [若為 true] 分支中，新增適用於您電子郵件提供者的「傳送電子郵件」動作。 設定此動作的詳細資料，如下圖中的資料表所示：

   ![將「傳送電子郵件」動作新增至「若為 true」分支](./media/logic-apps-control-flow-run-steps-group-scopes/send-email.png)

   1. 針對 [收件者] 欄位，輸入您的電子郵件地址以作為測試用。

   2. 針對 [主旨] 欄位，輸入下列文字：

      ```Time to leave: Traffic more than 10 minutes```

   3. 針對 [內文] 欄位，輸入此文字並加上尾端空格： 

      ```Travel time: ```

      當您的游標出現在 [內文] 欄位時，動態內容清單會保持開啟狀態，因此您可以選取任何目前可用的參數。

   4. 在動態內容清單中，選擇 [運算式]。

   5. 尋找並選取 **div( )** 函式。

   6. 當游標位於函式的括號內時，選擇 [動態內容]，這樣您就可以在下一步新增 [旅行期間交通] 參數。

   7. 在動態參數清單中的 [取得路線] 下，選取 [旅行期間交通] 欄位。

      ![選取 [旅行期間交通] 欄位](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-2.png)

   8. 在欄位解析成 JSON 格式之後，新增**逗號** (```,```) 並在後方加上數字 ```60```，讓**旅行期間交通**從以秒計算轉換為以分鐘計算。 
   
      ```
      div(body('Get_route')?['travelDurationTraffic'],60)
      ```

      您的運算式現在看起來就像下面這個範例︰

      ![完成運算式](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-3.png)  

   9. 完成時，請務必選擇 [確定]。

  10. 解析運算式之後，新增下列文字，並加上前置空格：``` minutes```
  
      您的 [內文] 欄位現在看起來就像下面這個範例︰

      ![完成的 [內文] 欄位](./media/logic-apps-control-flow-run-steps-group-scopes/send-email-4.png)

6. 儲存您的邏輯應用程式。

接下來，新增範圍，讓您可以群組特定動作，並評估其狀態。

## <a name="add-a-scope"></a>新增範圍

1. 如果您還沒開啟邏輯應用程式設計工具中的邏輯應用程式，請將它開啟。 

2. 在要使用的工作流程位置上新增範圍。 例如︰

   * 若要在邏輯應用程式工作流程中的現有步驟之間新增範圍，將指標移動至您要新增範圍的箭頭處。 
   選擇**加號** (**+**) > [新增範圍]。

     ![新增範圍](./media/logic-apps-control-flow-run-steps-group-scopes/add-scope.png)

     如果想要在工作流程結尾處新增範圍，請在邏輯應用程式底部，選擇 [+ 新增步驟] > [...更多] > [新增範圍]。

3. 現在可新增步驟，或拖曳您想要在範圍內執行的現有步驟。 針對此範例，拖曳以下動作至範圍中：
      
   * **取得路線**
   * **如果流量時間超過指定時間**，其中需同時包含 **true** 和 **false** 分支

   您的邏輯應用程式現在看起來就像下面這個範例︰

   ![新增的範圍](./media/logic-apps-control-flow-run-steps-group-scopes/scope-added.png)

4. 在範圍下，新增會檢查範圍狀態的條件。 以下列描述為條件重新命名：**如果範圍失敗**

   ![新增條件來檢查範圍狀態](./media/logic-apps-control-flow-run-steps-group-scopes/add-condition-check-scope-status.png)
  
5. 建立此運算式，檢查範圍狀態是否等於 `Failed` 或 `Aborted`。

   ![新增會檢查範圍狀態的運算式](./media/logic-apps-control-flow-run-steps-group-scopes/build-expression-check-scope-status.png)

   或者，若要以文字形式輸入此運算式，請選擇 [在進階模式中編輯]。

   ```@equals('@result(''Scope'')[0][''status'']', 'Failed, Aborted')```

6. 在 [若為 true] 和 [若為 false] 的分支中，新增您想執行的動作，例如傳送電子郵件或訊息。

   ![新增會檢查範圍狀態的運算式](./media/logic-apps-control-flow-run-steps-group-scopes/handle-true-false-branches.png)

7. 儲存您的邏輯應用程式。

已完成的邏輯應用程式現在看起來就像以下範例 (將所有圖形展開)：

![具有範圍的已完成邏輯應用程式](./media/logic-apps-control-flow-run-steps-group-scopes/scopes-overview.png)

## <a name="test-your-work"></a>儲存您的工作

在設計工具的工具列上，選擇 [執行]。 如果範圍內的所有動作都成功，您會取得「範圍成功」的訊息。 如果範圍內有任何動作失敗，您會取得「範圍失敗」的訊息。 

<a name="scopes-json"></a>

## <a name="json-definition"></a>JSON 定義

如果您是在程式碼檢視中進行作業，您可以改為以邏輯應用程式的 JSON 定義來定義範圍結構。 例如，下列是前面邏輯應用程式中觸發程序和動作的 JSON 定義：

``` json
"triggers": {
  "Recurrence": {
    "type": "Recurrence",
    "recurrence": {
       "frequency": "Minute",
       "interval": 1
    },
  }
}
```

```json
"actions": {
  "If_scope_failed": {
    "type": "If",
    "actions": {
      "Scope_failed": {
        "type": "ApiConnection",
        "inputs": {
          "body": {
            "Body": "Scope failed",
            "Subject": "Scope failed",
            "To": "<your-email@domain.com>"
          },
          "host": {
            "connection": {
              "name": "@parameters('$connections')['outlook']['connectionId']"
            }
          },
          "method": "post",
          "path": "/Mail"
        },
        "runAfter": {}
      }
    },
    "else": {
      "actions": {
        "Scope_succeded": {
          "type": "ApiConnection",
          "inputs": {
            "body": {
              "Body": "None",
              "Subject": "Scope succeeded",
              "To": "<your-email@domain.com>"
            },
            "host": {
              "connection": {
               "name": "@parameters('$connections')['outlook']['connectionId']"
              }
            },
            "method": "post",
            "path": "/Mail"
          },
          "runAfter": {}
        }
      }
    },
    "expression": "@equals('@result(''Scope'')[0][''status'']', 'Failed, Aborted')",
    "runAfter": {
      "Scope": [
        "Succeeded"
      ]
    }
  },
  "Scope": {
    "type": "Scope",
    "actions": {
      "Get_route": {
        "type": "ApiConnection",
        "inputs": {
          "host": {
            "connection": {
              "name": "@parameters('$connections')['bingmaps']['connectionId']"
            }
          },
          "method": "get",
          "path": "/REST/V1/Routes/Driving",
          "queries": {
            "distanceUnit": "Mile",
            "optimize": "timeWithTraffic",
            "travelMode": "Driving",
            "wp.0": "<start>",
            "wp.1": "<end>"
          }
        },
        "runAfter": {}
      },
      "If_traffic_time_more_than_specified_time": {
        "type": "If",
        "actions": {
          "Send_mail_when_traffic_exceeds_10_minutes": {
            "type": "ApiConnection",
            "inputs": {
              "body": {
                 "Body": "Travel time:@{div(body('Get_route')?['travelDurationTraffic'], 60)} minutes",
                 "Subject": "Time to leave: Traffic more than 10 minutes",
                 "To": "<your-email@domain.com>"
              },
              "host": {
                "connection": {
                   "name": "@parameters('$connections')['outlook']['connectionId']"
                }
              },
              "method": "post",
              "path": "/Mail"
            },
            "runAfter": {}
          }
        },
        "expression": "@greater(body('Get_route')?['travelDurationTraffic'], 600)",
        "runAfter": {
          "Get_route": [
            "Succeeded"
          ]
        }
      }
    },
    "runAfter": {}
  }
}
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能與建議，請造訪 [Azure Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* [根據條件 (條件陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [根據不同的值 (Switch 陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [執行並重複步驟 (迴圈)](../logic-apps/logic-apps-control-flow-loops.md)
* [執行或合併平行步驟 (分支)](../logic-apps/logic-apps-control-flow-branches.md)
