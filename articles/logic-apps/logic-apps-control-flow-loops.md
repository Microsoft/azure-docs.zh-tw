---
title: 新增會重複動作或處理陣列的迴圈 - Azure Logic Apps | Microsoft Docs
description: 如何在 Azure Logic Apps 中建立會重複工作流程動作或處理陣列的迴圈
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.date: 03/05/2018
ms.topic: article
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 87595eeb0330a2d8210258c097c29b205b628cf4
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35298180"
---
# <a name="create-loops-that-repeat-workflow-actions-or-process-arrays-in-azure-logic-apps"></a>在 Azure Logic Apps 中建立會重複工作流程動作或處理陣列的迴圈

若要逐一查看邏輯應用程式中的陣列，您可以使用  [Foreach](#foreach-loop) 迴圈 或循序  [Foreach 迴圈](#sequential-foreach-loop)。 標準  "Foreach" 迴圈中的反覆項目會平行執行，而循序  "Foreach" 迴圈中的反覆項目為每次執行一個。 如需 "Foreach" 迴圈在單一邏輯應用程式中可處理的陣列項目數上限，請參閱[限制和設定](../logic-apps/logic-apps-limits-and-config.md)。 

> [!TIP] 
> 如果您的觸發程序接收到陣列，並想要針對每個陣列項目執行工作流程，您可以使用 [**SplitOn** 觸發屬性](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch)將該陣列「解除批次」。 
  
若要重複動作直到條件符合或某些狀態改變，請使用 ["Until" 迴圈](#until-loop)。 邏輯應用程式會先執行迴圈內所有動作，然後將檢查條件當作最後一個步驟。 如果符合條件，則迴圈會停止。 否則，迴圈會重複。 如需單一邏輯應用程式中可執行的 "Until" 迴圈數上限，請參閱[限制和設定](../logic-apps/logic-apps-limits-and-config.md)。 

## <a name="prerequisites"></a>先決條件

* Azure 訂用帳戶。 如果您沒有訂用帳戶，請[註冊一個免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

<a name="foreach-loop"></a>

## <a name="foreach-loop"></a>"Foreach" 迴圈

若要針對陣列中每個項目重複動作，可在您的邏輯應用程式工作流程中使用 "Foreach" 迴圈。 您可以在 "Foreach" 迴圈中包含多個動作，並且可以使 "Foreach" 迴圈在彼此之中巢狀化。 根據預設，標準 "Foreach" 迴圈中的循環會平行執行。 如需 "Foreach" 迴圈可平行執行的循環數上限，請參閱[限制和設定](../logic-apps/logic-apps-limits-and-config.md)。

> [!NOTE] 
> "Foreach" 迴圈只適用於陣列，而且迴圈中的動作會使用 `@item()` 參考來處理陣列中的每個項目。 如果您指定的資料不是在陣列中，則邏輯應用程式工作流程將會失敗。 

例如，此邏輯應用程式會從網站的 RSS 摘要傳送每日摘要給您。 應用程式會使用 "Foreach" 迴圈，針對每個找到的新項目傳送電子郵件。

1. 使用 Outlook.com 或 Office 365 Outlook 帳戶來[建立此範例邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

2. 在 RSS 觸發程序和傳送電子郵件動作之間，新增 "Foreach" 迴圈。 

   若要在步驟之間新增迴圈，請將指標移至您要新增迴圈的箭號上。 
   選擇顯示的 **加號** \(**+** \)，然後選擇 [新增 for each]。

   ![在兩個步驟之間新增 "Foreach" 迴圈](media/logic-apps-control-flow-loops/add-for-each-loop.png)

3. 現在建置迴圈。 當**新增動態內容**清單出現後，在 [從上一個步驟中選取輸出] 下，選取 [摘要連結] 陣列，這是來自 RSS 觸發程序的輸出。 

   ![從動態內容清單選取](media/logic-apps-control-flow-loops/for-each-loop-dynamic-content-list.png)

   > [!NOTE] 
   > 您「只」可以選取上一個步驟中的陣列輸出。

   選取的陣列現在會出現在這裡：

   ![選取陣列](media/logic-apps-control-flow-loops/for-each-loop-select-array.png)

4. 若要在每個陣列項目上執行動作，請將 [傳送電子郵件] 動作拖曳至 **For each** 迴圈。 

   您的邏輯應用程式看起來可能就像下面這個範例︰

   ![將步驟新增至 "Foreach" 迴圈](media/logic-apps-control-flow-loops/for-each-loop-with-step.png)

5. 儲存您的邏輯應用程式。 若要手動測試邏輯應用程式，在設計工具的工具列上，選擇 [執行]。

<a name="for-each-json"></a>

## <a name="foreach-loop-definition-json"></a>"Foreach" 迴圈定義 (JSON)

如果您是在邏輯應用程式的程式碼檢視中進行作業，您可以改為以邏輯應用程式的 JSON 定義來定義 `Foreach` 迴圈，例如：

``` json
"actions": {
    "myForEachLoopName": {
        "type": "Foreach",
        "actions": {
            "Send_an_email": {
                "type": "ApiConnection",
                "inputs": {
                    "body": {
                        "Body": "@{item()}",
                        "Subject": "New CNN post @{triggerBody()?['publishDate']}",
                        "To": "me@contoso.com"
                    },
                    "host": {
                        "api": {
                            "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/office365"
                        },
                        "connection": {
                            "name": "@parameters('$connections')['office365']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/Mail"
                },
                "runAfter": {}
            }
        },
        "foreach": "@triggerBody()?['links']",
        "runAfter": {},
    }
},
```

<a name="sequential-foreach-loop"></a>

## <a name="foreach-loop-sequential"></a>"Foreach" 迴圈：循序

根據預設，"Foreach" 迴圈中的每個循環會針對每個陣列項目平行執行。 若要以循序方式執行每個循環，可在 "Foreach"迴圈中設定 [循序] 選項。

1. 在迴圈的右上角，選擇**省略符號** (**...**) > [設定]。

   ![在 "Foreach" 迴圈上，選擇 [...] > [設定]](media/logic-apps-control-flow-loops/for-each-loop-settings.png)

2. 開啟 [循序] 設定，然後選擇 [完成]。

   ![開啟 "Foreach" 迴圈的循序設定](media/logic-apps-control-flow-loops/for-each-loop-sequential-setting.png)

您也可以在邏輯應用程式的 JSON 定義中，將 **operationOptions** 參數設定為 `Sequential`。 例如︰

``` json
"actions": {
    "myForEachLoopName": {
        "type": "Foreach",
        "actions": {
            "Send_an_email": {               
            }
        },
        "foreach": "@triggerBody()?['links']",
        "runAfter": {},
        "operationOptions": "Sequential"
    }
},
```

<a name="until-loop"></a>

## <a name="until-loop"></a>"Until" 迴圈
  
若要重複動作直到條件符合或某些狀態改變，請在邏輯應用程式工作流程中使用 "Until" 迴圈。 以下是可以使用 "Until" 迴圈的一些常見使用案例：

* 呼叫端點，直到您獲得您想要的回應。
* 在資料庫中建立一筆記錄，等到該記錄中的特定欄位獲得核准，然後繼續處理。 

例如，每天上午 8:00，此邏輯應用程式會遞增變數，直到變數的值等於 10。 接著，邏輯應用程式傳送用來確認目前值的電子郵件。 雖然此範例使用 Office 365 Outlook，但您可以使用 Logic Apps 支援的任何電子郵件提供者 ([在此檢閱連接器清單](https://docs.microsoft.com/connectors/))。 如果您使用另一個電子郵件帳戶，則整體步驟相同，但您的 UI 可能稍有不同。 

1. 建立空白邏輯應用程式。 在邏輯應用程式設計工具中，搜尋「週期」，並選取此觸發程序：**排程 - 週期** 

   ![新增「排程 - 週期」觸發程序](./media/logic-apps-control-flow-loops/do-until-loop-add-trigger.png)

2. 透過設定時間間隔、頻率和一天的小時來指定觸發程序引發的時間。 若要設定小時，請選擇 [顯示進階選項]。

   ![新增「排程 - 週期」觸發程序](./media/logic-apps-control-flow-loops/do-until-loop-set-trigger-properties.png)

   | 屬性 | 值 |
   | -------- | ----- |
   | **間隔** | 1 | 
   | **頻率** | 天 |
   | **在這幾小時內** | 8 |
   ||| 

3. 在觸發程序底下，選擇 **新增步驟** > **新增動作**。 搜尋「變數」，然後選取此動作：**變數 - 初始化變數**

   ![新增 [變數 - 初始化變數] 動作](./media/logic-apps-control-flow-loops/do-until-loop-add-variable.png)

4. 使用下列值來設定您的變數：

   ![設定變數屬性](./media/logic-apps-control-flow-loops/do-until-loop-set-variable-properties.png)

   | 屬性 | 值 | 說明 |
   | -------- | ----- | ----------- |
   | **名稱** | 限制 | 變數的名稱 | 
   | **類型** | 整數  | 變數的資料類型 | 
   | **值** | 0 | 變數的起始值 | 
   |||| 

5. 在 [初始化變數] 動作下，選擇 [新增步驟] > [更多]。 選取此迴圈：**新增 do until**

   ![新增 "do until" 迴圈](./media/logic-apps-control-flow-loops/do-until-loop-add-until-loop.png)

6. 選取**限制**變數和**等於**運算子，可建置迴圈的結束條件。 輸入 **10** 作為比較值。

   ![建置停止迴圈的結束條件](./media/logic-apps-control-flow-loops/do-until-loop-settings.png)

7. 在迴圈中，選擇 [新增動作]。 搜尋「變數」，然後新增此動作：[變數 - 遞增變數]

   ![新增遞增變數的動作](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable.png)

8. 針對 [名稱]，選取**限制**變數。 針對 [值]，輸入 "1"。 

   ![以 1 遞增 [限制]](./media/logic-apps-control-flow-loops/do-until-loop-increment-variable-settings.png)

9. 在迴圈之下 (但不是在迴圈內)，新增傳送電子郵件的動作。 如果出現提示，請登入您的電子郵件帳戶。

   ![新增傳送電子郵件的動作](media/logic-apps-control-flow-loops/do-until-loop-send-email.png)

10. 設定電子郵件的內容。 在主旨上新增**限制**變數。 這樣一來，您可以確認變數的目前值符合您指定的條件，例如：

    ![設定電子郵件內容](./media/logic-apps-control-flow-loops/do-until-loop-send-email-settings.png)

    | 屬性 | 值 | 說明 |
    | -------- | ----- | ----------- | 
    | **To** | *<email-address@domain>* | 收件者的電子郵件地址。 若要進行測試，請使用自己的電子郵件地址。 | 
    | **主旨** | [限制] 目前的值是**限制** | 指定電子郵件主旨。 在此範例中，請確定您已包含**限制**變數。 | 
    | **內文** | <*email-content*> | 指定您想要傳送的電子郵件訊息內容。 針對此範例，輸入任何您喜歡的文字。 | 
    |||| 

11. 儲存您的邏輯應用程式。 若要手動測試邏輯應用程式，在設計工具的工具列上，選擇 [執行]。

    當您的邏輯開始執行之後，您會收到電子郵件及您指定的內容：

    ![收到的電子郵件](./media/logic-apps-control-flow-loops/do-until-loop-sent-email.png)

## <a name="prevent-endless-loops"></a>避免無止盡的迴圈

"Until" 迴圈具有預設的限制，就是達到下列任何條件時，便會停止執行：

| 屬性 | 預設值 | 說明 | 
| -------- | ------------- | ----------- | 
| **Count** | 60 | 迴圈結束前可執行的迴圈數上限。 預設為 60 個循環。 | 
| **逾時** | PT1H | 迴圈結束前可執行迴圈的時間上限。 預設值是一小時，並以 ISO 8601 格式指定。 <p>逾時值的評估會以每個迴圈循環為基礎。 如果迴圈中有任何動作所花費的時間超過逾時限制，目前的循環並不會停止，但不會啟動下一個循環，因為不符合限制條件。 | 
|||| 

若要變更這些預設限制，請選擇迴圈動作圖形中的 [顯示進階選項]。

<a name="until-json"></a>

## <a name="until-definition-json"></a>"Until" 定義 (JSON)

如果您是在邏輯應用程式的程式碼檢視中進行作業，您可以改為以邏輯應用程式的 JSON 定義來定義 `Until` 迴圈，例如：

``` json
"actions": {
    "Initialize_variable": {
        // Definition for initialize variable action
    },
    "Send_an_email": {
        // Definition for send email action
    },
    "Until": {
        "type": "Until",
        "actions": {
            "Increment_variable": {
                "type": "IncrementVariable",
                "inputs": {
                    "name": "Limit",
                    "value": 1
                },
                "runAfter": {}
            }
        },
        "expression": "@equals(variables('Limit'), 10)",
        // To prevent endless loops, an "Until" loop 
        // includes these default limits that stop the loop. 
        "limit": { 
            "count": 60,
            "timeout": "PT1H"
        },
        "runAfter": {
            "Initialize_variable": [
                "Succeeded"
            ]
        },
    }
},
```

在另一個範例中，此 "Until" 迴圈會呼叫建立資源的 HTTP 端點，並在 HTTP 回應主體傳回「完成」狀態時停止執行。 若要避免無止盡迴圈，迴圈也會在達到下列任一條件時停止：

* 迴圈已依循 `count` 屬性所指定的內容執行了 10 次。 預設值為 60 次。 
* 迴圈已依循 `timeout` 屬性所指定的內容 (ISO 8601 格式) 嘗試執行兩個小時。 預設值是一小時。
  
``` json
"actions": {
    "myUntilLoopName": {
        "type": "Until",
        "actions": {
            "Create_new_resource": {
                "type": "Http",
                "inputs": {
                    "body": {
                        "resourceId": "@triggerBody()"
                    },
                    "url": "https://domain.com/provisionResource/create-resource",
                    "body": {
                        "resourceId": "@triggerBody()"
                    }
                },
                "runAfter": {},
                "type": "ApiConnection"
            }
        },
        "expression": "@equals(triggerBody(), 'Completed')",
        "limit": {
            "count": 10,
            "timeout": "PT2H"
        },
        "runAfter": {}
    }
},
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能與建議，請造訪 [Azure Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* [根據條件 (條件陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [根據不同值 (Switch 陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [執行或合併平行步驟 (分支)](../logic-apps/logic-apps-control-flow-branches.md)
* [依據群組的動作狀態執行步驟 (範圍)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
