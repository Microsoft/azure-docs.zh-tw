---
title: 建立用於儲存值的變數 - Azure Logic Apps | Microsoft Docs
description: 如何在 Azure Logic Apps 中建立變數來儲存和管理值
services: logic-apps
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.topic: article
ms.date: 05/30/2018
ms.service: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 0efce9fbbbd241f335f08bb258b6ba343982fdb9
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35299183"
---
# <a name="create-variables-for-saving-and-managing-values-in-azure-logic-apps"></a>在 Azure Logic Apps 中建立用於儲存和管理值的變數

本文說明如何藉由建立變數，在整個邏輯應用程式儲存及使用值。 例如，變數可協助您計算迴圈的執行次數。 逐一查看某個陣列或檢查陣列中是否有特定項目時，您可以使用變數來參考每個陣列項目的索引編號。 

您可以為整數、浮點數、布林值、字串、陣列及物件等資料類型建立變數。 建立變數之後，您可以執行其他工作，例如：

* 取得或參考變數的值。
* 以常數值將變數增大或減小，也稱為「遞增」和「遞減」。
* 為變數指派不同的值。
* 在字串或陣列中插入或「附加」變數的值作為最後一個項目。

變數只存在於建立它們的邏輯應用程式執行個體內，並且是全域性。 此外，它們只會在邏輯應用程式執行個體內跨任何迴圈反覆項目存留。 參考變數時，請使用變數的名稱作為語彙基元，而不是使用動作的名稱，動作名稱是參考動作輸出的實用方式。

如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

## <a name="prerequisites"></a>先決條件

若要依照本文進行操作，以下是您需要的項目：

* 您要建立變數的邏輯應用程式 

  如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

* 作為邏輯應用程式中第一個步驟的[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts) 

  您的邏輯應用程式必須以觸發程序作為開頭，您才能新增用於建立和使用變數的動作。

<a name="create-variable"></a>

## <a name="initialize-variable"></a>將變數初始化

您可以建立變數並宣告其資料類型和初始值 - 全都在邏輯應用程式中的一個動作內完成。 您只能在全域層級宣告變數，而無法在範圍、條件及迴圈內宣告變數。 

1. 在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>或 Visual Studio 的「邏輯應用程式設計工具」中，開啟邏輯應用程式。 

   此範例使用 Azure 入口網站和邏輯應用程式搭配現有的觸發程序。

2. 在邏輯應用程式中您想要新增變數的步驟底下，依照下列其中一個步驟進行操作： 

   * 若要在最後一個步驟底下新增動作，請選擇 [新增步驟] > [新增動作]。

     ![新增動作](./media/logic-apps-create-variables-store-values/add-action.png)

   * 若要在步驟之間新增動作，請將滑鼠移至連接箭頭上，如此就會出現加號 (+)。 
   選擇加號，然後選擇 [新增動作]。

3. 在搜尋方塊中，輸入 "variables" 作為篩選條件。 從動作清單中，選取 [變數 - 初始化變數]。

   ![選取動作](./media/logic-apps-create-variables-store-values/select-initialize-variable-action.png)

4. 為您的變數提供下列資訊：

   | 屬性 | 必要 | 值 |  說明 |
   |----------|----------|-------|--------------|
   | Name | yes | <*variable-name*> | 要遞增的變數名稱 | 
   | 類型 | yes | <*variable-type*> | 變數的資料類型 | 
   | 值 | 否 | <*start-value*> | 您的變數初始值 <p><p>**提示**：雖然此值為選擇性，但最佳做法是設定此值，如此您便一律知道變數的開始值。 | 
   ||||| 

   ![將變數初始化](./media/logic-apps-create-variables-store-values/initialize-variable.png)

5. 現在繼續新增您想要的動作。 當您完成時，請在設計工具的工具列上，選擇 [儲存]。

如果您從設計工具切換到程式碼檢視編輯器，以下是 [初始化變數] 動作在您邏輯應用程式定義內顯示的樣子，其中採用「JavaScript 物件標記法」(JSON) 格式：

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   }
},
```

以下是一些其他變數類型的範例：

*字串變數*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myStringVariable",
               "type": "String",
               "value": "lorem ipsum"
          } ]
      },
      "runAfter": {}
   }
},
```

*布林值變數*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myBooleanVariable",
               "type": "Boolean",
               "value": false
          } ]
      },
      "runAfter": {}
   }
},
```

*含整數的陣列*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": [1, 2, 3]
          } ]
      },
      "runAfter": {}
   }
},
```

*含字串的陣列*

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "myArrayVariable",
               "type": "Array",
               "value": ["red", "orange", "yellow"]
          } ]
      },
      "runAfter": {}
   }
},
```

<a name="get-value"></a>

## <a name="get-the-variables-value"></a>取得變數的值

若要擷取或參考變數的內容，您也可以在「邏輯應用程式設計工具」和程式碼檢視編輯器中使用 [variables() 函式](../logic-apps/workflow-definition-language-functions-reference.md#variables)。
參考變數時，請使用變數的名稱作為語彙基元，而不是使用動作的名稱，動作名稱是參考動作輸出的實用方式。 

例如，下列運算式會使用 **variables()** 函式，從[本文中先前建立的](#append-value)陣列變數取得項目。 **string()** 函式會以字串格式傳回變數的內容：`"1, 2, 3, red"`

```json
@{string(variables('myArrayVariable'))}
```

<a name="increment-value"></a>

## <a name="increment-variable"></a>遞增變數 

若要以常數值將變數增大或「遞增」，請將 [變數 - 遞增變數] 動作新增至邏輯應用程式。 此動作僅適用於整數和浮點數變數。

1. 在「邏輯應用程式設計工具」中您想要增大現有變數的步驟底下，選擇 [新增步驟] > [新增動作]。 

   例如，此邏輯應用程式已經有一個已建立變數的觸發程序和動作。 因此，請在這些步驟底下新增動作：

   ![新增動作](./media/logic-apps-create-variables-store-values/add-increment-variable-action.png)

   若要在現有步驟之間新增動作，請將滑鼠移至連接箭頭上，如此就會出現加號 (+)。 選擇加號，然後選擇 [新增動作]。

2. 在搜尋方塊中，輸入 "increment variable" 作為篩選條件。 從動作清單中，選取 [變數 - 遞增變數]。

   ![選取 [遞增變數] 動作](./media/logic-apps-create-variables-store-values/select-increment-variable-action.png)

3. 提供下列資訊來遞增您的變數：

   | 屬性 | 必要 | 值 |  說明 |
   |----------|----------|-------|--------------|
   | Name | yes | <*variable-name*> | 要遞增的變數名稱 | 
   | 值 | 否 | <*increment-value*> | 用來遞增變數的值。 預設值為一。 <p><p>**提示**：雖然此值為選擇性，但最佳做法是設定此值，如此您便一律知道用來遞增變數的特定值。 | 
   |||| 

   例如︰ 
   
   ![遞增值範例](./media/logic-apps-create-variables-store-values/increment-variable-action-information.png)

4. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

如果您從設計工具切換到程式碼檢視編輯器，以下是 [遞增變數] 動作在您邏輯應用程式定義內顯示的樣子，其中採用 JSON 格式：

```json
"actions": {
   "Increment_variable": {
      "type": "IncrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```

## <a name="example-create-loop-counter"></a>範例：建立迴圈計數器

變數通常用來計算迴圈的執行次數。 此範例藉由建立一個會計算電子郵件中附件的迴圈，說明如何建立及使用變數來進行此工作。

1. 在 Azure 入口網站中，建立空白的邏輯應用程式。 新增一個會檢查是否有新電子郵件及任何附件的觸發程序。 

   此範例使用 Office 365 Outlook 的 [收到新的電子郵件時] 觸發程序。 
   您可以將此觸發程序設定成只有當電子郵件含有附件時才觸發。
   不過，您可以使用 Outlook.com 之類的連接器，檢查是否有含附件的新電子郵件。

2. 在觸發程序中，選擇 [顯示進階選項]。 若要讓觸發程序檢查是否有附件，並將這些附件傳遞至邏輯應用程式的工作流程，請針對下列屬性選取 [是]：
   
   * **有附件** 
   * **包含附件** 

   ![檢查是否有附件並包含附件](./media/logic-apps-create-variables-store-values/check-include-attachments.png)

3. 新增 [[初始化變數] 動作](#create-variable)。 建立名為 **Count** 且開始值為零的整數變數。

   ![新增 [初始化變數] 的動作](./media/logic-apps-create-variables-store-values/initialize-variable.png)

4. 若要循環處理每個附件，請選擇 [新增步驟] > **[其他]** > [新增「針對各項」]，以新增 *for each* 迴圈。

   ![新增 [For each] 迴圈](./media/logic-apps-create-variables-store-values/add-loop.png)

5. 在迴圈中，按一下 [選取先前步驟中的輸出] 方塊內部。 當動態內容清單出現時，選取 [附件]。 

   ![選取 [附件]](./media/logic-apps-create-variables-store-values/select-attachments.png)

   [附件] 欄位會將含有電子郵件附件的陣列從觸發程序輸出傳遞至您的迴圈。

6. 在 [For each] 迴圈中，選取 [新增動作]。 

   ![選取 [新增動作]](./media/logic-apps-create-variables-store-values/add-action-2.png)

7. 在搜尋方塊中，輸入 "increment variable" 作為篩選條件。 從動作清單中，選取 [變數 - 遞增變數]。

   > [!NOTE]
   > 請確定 [遞增變數] 動作出現在迴圈內。 如果此動作出現在迴圈外，請將此動作拖曳到迴圈中。

8. 在 [遞增變數] 動作中，從 [名稱] 清單中，選取 [Count] 變數。 

   ![選取 [Count] 變數](./media/logic-apps-create-variables-store-values/add-increment-variable-example.png)

9. 在迴圈底下，新增任何會將附件數目傳送給您的動作。 在您的動作中，包含來自 **Count** 變數的值，例如： 

   ![新增會傳送結果的動作](./media/logic-apps-create-variables-store-values/send-email-results.png)

10. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。 

### <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 如果未啟用您的邏輯應用程式，請在邏輯應用程式功能表上，選擇 [概觀]。 在工具列上，選擇 [啟用]。 

2. 在「邏輯應用程式設計工具」的工具列上，選擇 [執行]。 此步驟會手動啟動您的邏輯應用程式。

3. 將一封含有一或多個附件的電子郵件傳送給您在此範例中使用的電子郵件帳戶。

   此步驟會觸發邏輯應用程式的觸發程序，它會為邏輯應用程式的工作流程建立並執行一個執行個體。
   因此，邏輯應用程式會傳送訊息或電子郵件給您，當中顯示您所傳送電子郵件中的附件數目。

如果您從設計工具切換到程式碼檢視編輯器，以下是 [For each] 迴圈與 [遞增變數] 一起在您邏輯應用程式定義內顯示的樣子，其中採用 JSON 格式。

```json
"actions": {
   "For_each": {
      "type": "Foreach",
      "actions": {
         "Increment_variable": {
           "type": "IncrementVariable",
            "inputs": {
               "name": "Count",
               "value": 1
            },
            "runAfter": {}
         }
      },
      "foreach": "@triggerBody()?['Attachments']",
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="decrement-value"></a>

## <a name="decrement-variable"></a>遞減變數

若要以常數值將變數減小或「遞減」，請依照[將變數增大](#increment-value)的步驟進行操作，只是其中要改為尋找並選取 [變數 - 遞減變數] 動作。 此動作僅適用於整數和浮點數變數。

以下是 [遞減變數] 動作的屬性：

| 屬性 | 必要 | 值 |  說明 |
|----------|----------|-------|--------------|
| Name | yes | <*variable-name*> | 要遞減的變數名稱 | 
| 值 | 否 | <*increment-value*> | 用來遞減變數的值。 預設值為一。 <p><p>**提示**：雖然此值為選擇性，但最佳做法是設定此值，如此您便一律知道用來遞減變數的特定值。 | 
||||| 

如果您從設計工具切換到程式碼檢視編輯器，以下是 [遞減變數] 動作在您邏輯應用程式定義內顯示的樣子，其中採用 JSON 格式。

```json
"actions": {
   "Decrement_variable": {
      "type": "DecrementVariable",
      "inputs": {
         "name": "Count",
         "value": 1
      },
      "runAfter": {}
   }
},
```


<a name="assign-value"></a>

## <a name="set-variable"></a>設定變數

若要為現有變數指派不同的值，請依照[將變數增大](#increment-value)的步驟進行操作，只是其中要： 

1. 改為尋找並選取 [變數 - 設定變數] 動作。 

2. 提供變數名稱及您想要指派的值。 新的值和變數必須具有相同的資料類型。
必須提供值，因為此動作沒有預設值。 

以下是 [設定變數] 動作的屬性：

| 屬性 | 必要 | 值 |  說明 | 
|----------|----------|-------|--------------| 
| Name | yes | <*variable-name*> | 要變更的變數名稱 | 
| 值 | yes | <*new-value*> | 您想要指派給變數的值。 兩者必須具有相同的資料類型。 | 
||||| 

> [!NOTE]
> 除非您是要將變數增大或減小，否則變更迴圈內的變數「可能」會造成非預期的結果，因為迴圈預設會以平行方式或並行方式執行。 針對這些情況，請嘗試將迴圈設定成以循序方式執行。 例如，當您想要參考迴圈內的變數值，並預期該迴圈執行個體的開始值和結束值相同時，請依照下列步驟來變更迴圈的執行方式： 
>
> 1. 在您迴圈的右上角中，選擇省略符號 (...) 按鈕，然後選擇 [設定]。
> 
> 2. 在 [並行控制] 底下，將 [覆寫預設] 設定變更為 [開啟]。
>
> 3. 將 [平行處理原則的程度] 滑桿拖曳至 [1]。

如果您從設計工具切換到程式碼檢視編輯器，以下是 [設定變數] 動作在您邏輯應用程式定義內顯示的樣子，其中採用 JSON 格式。 此範例會將 "Count" 變數目前的值變更為另一個值。 

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
               "name": "Count",
               "type": "Integer",
               "value": 0
          } ]
      },
      "runAfter": {}
   },
   "Set_variable": {
      "type": "SetVariable",
      "inputs": {
         "name": "Count",
         "value": 100
      },
      "runAfter": {
         "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

<a name="append-value"></a>

## <a name="append-to-variable"></a>附加至變數

針對儲存字串或陣列的變數，您可以插入或「附加」變數的值作為這些字串或陣列的最後一個項目。 您可以依照[將變數增大](#increment-value)的步驟進行操作，只是其中要改為採用下列步驟： 

1. 視您的變數是字串還是陣列而定，尋找並選取下列其中一個動作： 

  * **變數 - 附加至字串變數**
  * **變數 - 附加到陣列變數** 

2. 提供要附加作為字串或陣列最後一個項目的值。 需要此值。 

以下是 [附加至...] 動作的屬性：

| 屬性 | 必要 | 值 |  說明 | 
|----------|----------|-------|--------------| 
| Name | yes | <*variable-name*> | 要變更的變數名稱 | 
| 值 | yes | <*append-value*> | 您想要附加的值 (可以具有任何類型) | 
|||||  

如果您從設計工具切換到程式碼檢視編輯器，以下是 [附加到陣列變數] 動作在您邏輯應用程式定義內顯示的樣子，其中採用 JSON 格式。
此範例會建立一個陣列變數，然後將另一個值新增為陣列中的最後一個項目。 結果會是一個含有下列陣列的已更新變數：`[1,2,3,"red"]` 

```json
"actions": {
   "Initialize_variable": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ {
            "name": "myArrayVariable",
            "type": "Array",
            "value": [1, 2, 3]
         } ]
      },
      "runAfter": {}
   },
   "Append_to_array_variable": {
      "type": "AppendToArrayVariable",
      "inputs": {
         "name": "myArrayVariable",
         "value": "red"
      },
      "runAfter": {
        "Initialize_variable": [ "Succeeded" ]
      }
   }
},
```

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解[Logic Apps 連接器](../connectors/apis-list.md)
