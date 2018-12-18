---
title: 建立或加入平行分支 - Azure Logic Apps | Microsoft Docs
description: 如何在 Azure Logic Apps 中建立或加入工作流程的平行分支
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.date: 03/05/2018
ms.topic: article
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 2a8dcd82b67ee64e5687d8687415056b0aab39aa
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35298850"
---
# <a name="create-or-join-parallel-branches-for-workflow-actions-in-azure-logic-apps"></a>在 Azure Logic Apps 中建立或加入工作流程動作的平行分支

根據預設，邏輯應用程式中的動作會依序執行。 若要同時執行獨立的動作，您可以建立[平行分支](#parallel-branches)，並在之後[將這些分支加入](#join-branches)您的流程中。 

> [!TIP] 
> 如果您的觸發程序接收到陣列，並想要針對每個陣列項目執行工作流程，您可以使用 [**SplitOn** 觸發屬性](../logic-apps/logic-apps-workflow-actions-triggers.md#split-on-debatch)將該陣列「解除批次」。

## <a name="prerequisites"></a>先決條件

* Azure 訂用帳戶。 如果您沒有訂用帳戶，請[註冊一個免費的 Azure 帳戶](https://azure.microsoft.com/free/)。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

<a name="parallel-branches"></a>

## <a name="add-a-parallel-branch"></a>新增平行分支

若要同時執行獨立的步驟，您可以在現有的步驟旁新增平行分支。 

![平行執行步驟](media/logic-apps-control-flow-branches/parallel.png)

邏輯應用程式會等到所有分支完成後，再繼續進行工作流程。
平行分支會在其 `runAfter` 屬性值符合完成的父步驟狀態時才執行。 例如，`branchAction1` 和 `branchAction2` 皆設定為在 `parentAction` 完成且狀態為 `Succeded` 時，才會執行。

> [!NOTE]
> 開始之前，應用程式邏輯必須已有可新增平行分支的步驟。

1. 在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>的邏輯應用程式設計工具中，開啟邏輯應用程式。

2. 將滑鼠移到要新增平行分支之步驟上方的箭號。

3. 選擇**加號** (**+**)、選擇 [新增平行分支]，然後選取您想要新增的項目。

   ![新增平行分支](media/logic-apps-control-flow-branches/add-parallel-branch.png)

   您選取的項目現在會出現在平行分支中。

4. 針對每個平行分支，新增您想要的步驟。 若要將循序動作新增至平行分支，請將滑鼠移至您要新增循序動作之動作的下方。 選擇**加號** (**+**) 和您想要新增的步驟。

   ![將循序步驟新增至平行分支](media/logic-apps-control-flow-branches/add-sequential-action-parallel-branch.png)

5. 若要將分支合併回去，則[加入您的平行分支](#join-branches)。 

<a name="parallel-json"></a>

## <a name="parallel-branch-definition-json"></a>平行分支定義 (JSON)

如果您是在程式碼檢視中進行作業，可以改為以邏輯應用程式的 JSON 定義來定義平行結構，例如：

``` json
{
  "triggers": {
    "myTrigger": { }
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

<a name="join-branches"></a>

## <a name="join-parallel-branches"></a>加入平行分支

若要將平行分支合併在一起，只需在底部 (所有分支下方) 新增步驟。 此步驟會在所有平行分支完成執行後再執行。

![加入平行分支](media/logic-apps-control-flow-branches/join.png)

1. 在 [Azure 入口網站](https://portal.azure.com)的邏輯應用程式設計工具中，尋找並開啟邏輯應用程式。 

2. 在想要加入的平行分支下，新增您想要執行的步驟。

   ![新增加入平行分支的步驟](media/logic-apps-control-flow-branches/join-steps.png)

   平行分支現在已合併。

<a name="join-json"></a>

## <a name="join-definition-json"></a>加入定義 (JSON)

如果您是在程式碼檢視中進行作業，您可以改為以邏輯應用程式的 JSON 定義來定義加入結構，例如：

``` json
{
  "triggers": {
    "myTrigger": { }
  },
  "actions": {
    "parentAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {}
    },
    "branchAction1": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "branchAction2": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "parentAction": [
          "Succeeded"
        ]
      }
    },
    "joinAction": {
      "type": "<action-type>",
      "inputs": { },
      "runAfter": {
        "branchAction1": [
          "Succeeded"
        ],
        "branchAction2": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能與建議，請造訪 [Azure Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* [根據條件 (條件陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-conditional-statement.md)
* [根據不同的值 (Switch 陳述式) 執行步驟](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [執行並重複步驟 (迴圈)](../logic-apps/logic-apps-control-flow-loops.md)
* [依據群組的動作狀態執行步驟 (範圍)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
