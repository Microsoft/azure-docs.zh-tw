---
title: 從 Microsoft Flow 呼叫 Azure 函式 | Microsoft Docs
description: 建立自訂連接器，然後使用該連接器來呼叫函式。
services: functions
keywords: 雲端應用程式, 雲端服務, Microsoft Flow, 商務程序, 商務應用程式
documentationcenter: ''
author: ggailey777
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: functions
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/14/2017
ms.author: glenga
ms.reviewer: sunayv
ms.custom: ''
ms.openlocfilehash: 57d80ad836a16b8821ba0cce42c822728c654dfd
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38467739"
---
# <a name="call-a-function-from-microsoft-flow"></a>從 Microsoft Flow 呼叫函式

[Microsoft Flow](https://flow.microsoft.com/) 可讓您輕易將最愛的應用程式與服務之間的工作流程和商業程序自動化。 專業開發人員可以使用 Azure Functions 來擴充 Microsoft Flow 的功能，同時透過技術詳細資料來防護流程建立器。

您會在本主題中根據風力渦輪機的維護案例建置一個流程。 本主題說明如何呼叫您在[為函式建立 OpenAPI 定義](functions-openapi-definition.md)中定義的函式。 此函式可判斷風力渦輪機的緊急修復是否符合成本效益。 如果符合成本效益，流程就會傳送電子郵件來建議修復。

如需從 PowerApps 呼叫同一個函式的相關資訊，請參閱[從 PowerApps 呼叫函式](functions-powerapps-scenario.md)。

在本主題中，您將了解如何：

> [!div class="checklist"]
> * 在 SharePoint 中建立清單。
> * 匯出 API 定義。
> * 新增 API 連線。
> * 如果修復符合成本效益，就建立流程來傳送電子郵件。
> * 執行流程。

## <a name="prerequisites"></a>先決條件

+ 使用中的 [Microsoft Flow 帳戶](https://flow.microsoft.com/documentation/sign-up-sign-in/)，其登入認證與您的 Azure 帳戶相同。 
+ SharePoint，您可用來作為此流程的資料來源。 如果您還沒有 SharePoint，請註冊 [Office 365 試用版](https://signup.microsoft.com/Signup?OfferId=467eab54-127b-42d3-b046-3844b860bebf&dl=O365_BUSINESS_PREMIUM&ali=1)。
+ 完成[為函式建立 OpenAPI 定義](functions-openapi-definition.md)教學課程。

## <a name="create-a-sharepoint-list"></a>建立 SharePoint 清單
首先，建立一個您用來作為流程資料來源的清單。 清單具有下列資料行。

| 清單資料行     | 資料類型           | 注意                                    |
|-----------------|---------------------|------------------------------------------|
| **標題**           | 單行文字 | 渦輪機的名稱                      |
| **LastServiceDate** | 日期                |                                          |
| **MaxOutput**       | 數字              | 渦輪機的輸出，以 KwH 為單位            |
| **ServiceRequired** | 是/否              |                                          |
| **EstimatedEffort** | 數字              | 修復的預估時間，以小時為單位 |

1. 在 SharePoint 網站中，依序按一下或點選 [新增] 和 [清單]。

    ![建立新的 SharePoint 清單](./media/functions-flow-scenario/new-list.png)

2. 輸入名稱 `Turbines`，然後按一下或點選 [建立]。

    ![指定新清單的名稱](./media/functions-flow-scenario/create-list.png)

    即會建立 [Turbines] \(渦輪機\) 清單，其中含有預設的 [標題] 欄位。

    ![渦輪機清單](./media/functions-flow-scenario/initial-list.png)

3. 依序按一下或點選 ![新項目圖示](./media/functions-flow-scenario/icon-new.png) 和 [日期]。

    ![新增單行文字欄位](./media/functions-flow-scenario/add-column.png)

4. 輸入名稱 `LastServiceDate`，然後按一下或點選 [建立]。

    ![建立 LastServiceDate 資料行](./media/functions-flow-scenario/date-column.png)

5. 針對清單中的其他三個資料行，重複執行最後兩個步驟：

    1. **Number** > "MaxOutput"

    2. **Yes/No** > "ServiceRequired"

    3. **Number** > "EstimatedEffort"

就是現在，您應該具備看起來如下圖的空白清單。 建立流程之後，您可以將資料新增至清單中。

![空白清單](media/functions-flow-scenario/empty-list.png)

[!INCLUDE [Export an API definition](../../includes/functions-export-api-definition.md)]

## <a name="add-a-connection-to-the-api"></a>新增 API 連線
自訂 API (也稱為自訂連接器) 適用於 Microsoft Flow，但您必須建立 API 連線，才能在流程中使用。

1. 在 [flow.microsoft.com](https://flow.microsoft.com) 中，按一下齒輪圖示 (右上角)，然後按一下 [連線]。

    ![流程連線](media/functions-flow-scenario/flow-connections.png)

1. 按一下 [建立連線]、向下捲動至 [Turbine Repair] \(渦輪機修復\) 連接器並按一下。

    ![建立連線](media/functions-flow-scenario/create-connection.png)

1. 輸入 API 金鑰，然後按一下 [建立連線]。

    ![輸入 API 金鑰並建立](media/functions-flow-scenario/api-key.png)

> [!NOTE]
> 如果您與其他人共用流程，處理或使用該流程的所有人也必須輸入 API 金鑰才能連接到 API。 此行為在未來可能會變更，我們將會更新本主題，以反映這一點。


## <a name="create-a-flow"></a>建立流程

現在您已準備好使用自訂連接器和您建立的 SharePoint 清單來建立流程。

### <a name="add-a-trigger-and-specify-a-condition"></a>新增觸發程序並指定條件

首先，從空白 (不含範本) 建立流程，然後新增要在 SharePoint 清單中建立項目時引發的「觸發程序」。 然後新增「條件」來決定接下來的情況。

1. 在 [flow.microsoft.com](https://flow.microsoft.com) 中，按一下 [我的流程]，然後按一下 [從空白建立]。

    ![從空白建立](media/functions-flow-scenario/create-from-blank.png)

2. 按一下 SharePoint 觸發程序 [建立項目時]。

    ![選擇觸發程序](media/functions-flow-scenario/choose-trigger.png)

    如果您尚未登入 SharePoint，系統會提示您登入。

3. 針對 [網站位址]，輸入您的 SharePoint 網站名稱，然後針對 [清單名稱]，輸入包含渦輪機資料的清單。

    ![選擇觸發程序](media/functions-flow-scenario/site-list.png)

4. 依序按一下 [新增步驟] 和 [新增條件]。

    ![新增條件](media/functions-flow-scenario/add-condition.png)

    Microsoft Flow 會在流程中新增兩個分支：**如果是**和**如果否**。 當您定義要比對的條件之後，可將步驟新增至其中一個分支或這兩個分支。

    ![條件分支](media/functions-flow-scenario/condition-branches.png)

5. 在 [條件] 卡片上，按一下第一個方塊，然後從 [動態內容] 對話方塊中選取 [ServiceRequired]。

    ![針對條件選取欄位](media/functions-flow-scenario/condition1-field.png)

6. 針對條件輸入 `True` 值。

    ![針對條件輸入 true](media/functions-flow-scenario/condition1-yes.png)

    值會在 SharePoint 清單中顯示為 `Yes` 或 `No`，但它會儲存為「布林值」(`True` 或 `False`)。 

7. 按一下頁面頂端的 [建立流程]。 請務必定期按一下 [更新流程]。

針對清單中建立的任何項目，流程會檢查是否已將 [ServiceRequired] 欄位設為 `Yes`，接著適當地移至 [如果是] 分支或 [如果否] 分支。 為了節省時間，在本主題中，您只能針對 [如果是] 分支指定動作。

### <a name="add-the-custom-connector"></a>新增自訂連接器

您現在要新增自訂連接器，在 Azure 中呼叫函式。 您可以將自訂連接器新增至流程，就像標準的連接器。 

1. 在 [如果是] 分支中，按一下 [新增動作]。

    ![新增動作](media/functions-flow-scenario/condition1-yes-add-action.png)

2. 在 [選擇動作] 對話方塊中，搜尋 `Turbine Repair`，然後選取動作 [Turbine Repair - Calculates costs] \(渦輪機修復 - 計算成本\)。

    ![選擇動作](media/functions-flow-scenario/choose-turbine-repair.png)

    下圖顯示已新增至流程的卡片。 欄位和描述均來自此連接器的 OpenAPI 定義。

    ![計算成本預設值](media/functions-flow-scenario/calculates-costs-default.png)

3. 在 [計算成本] 卡片上，使用 [動態內容] 對話方塊來選取函式的輸入。 Microsoft Flow 會顯示數值欄位，而不是日期欄位，因為 OpenAPI 定義會指定 [小時] 和 [產能] 均為數值。

    針對 [小時] 選取 [EstimatedEffort]，並針對 [產能] 選取 [MaxOutput]。

    ![選擇動作](media/functions-flow-scenario/calculates-costs-fields.png)

     現在您可以根據函式的輸出新增另一個條件。

4. 在 [如果是] 分支底部，按一下 [更多]，然後按一下 [新增條件]。

    ![新增條件](media/functions-flow-scenario/condition2-add.png)

5. 在 [條件 2] 卡片上，按一下第一個方塊，然後從 [動態內容] 對話方塊中選取 [Message]。

    ![針對條件選取欄位](media/functions-flow-scenario/condition2-field.png)

6. 輸入 `Yes` 的值。 流程會根據函式所傳回的訊息為 Yes (進行修復) 或 No (不要修復)，移至下一個 [如果是] 分支或 [如果否] 分支。 

    ![針對條件輸入 Yes](media/functions-flow-scenario/condition2-yes.png)

流程現在看起來應該類似下圖。

![針對條件輸入 Yes](media/functions-flow-scenario/flow-checkpoint1.png)

### <a name="send-email-based-on-function-results"></a>根據函式結果傳送電子郵件

此時在流程中，函式已從函式傳回 `Yes` 或 `No` 的 **Message** 值，以及其他關於成本和潛在收益的資訊。 在第二個條件的 [如果是] 分支中，您將會傳送電子郵件，但您可以執行任意數目的動作，例如，回寫至 SharePoint 清單或啟動[核准程序](https://flow.microsoft.com/documentation/modern-approvals/)。

1. 在第二個條件的 [如果是] 分支中，按一下 [新增動作]。

    ![新增動作](media/functions-flow-scenario/condition2-yes-add-action.png)

2. 在 [選擇動作] 對話方塊中，搜尋 `email`，然後根據您使用的電子郵件系統 (在此案例中為 Outlook) 傳送電子郵件動作。

    ![Outlook 傳送電子郵件](media/functions-flow-scenario/outlook-send-email.png)

3. 在 [傳送電子郵件] 卡片上，撰寫一封電子郵件。 針對 [收件者] 欄位輸入您組織中的有效名稱。 在下圖中，您可以看到其他欄位為來自 [動態內容] 對話方塊之文字與語彙基元的組合。 

    ![電子郵件欄位](media/functions-flow-scenario/email-fields.png)

    [標題] 語彙基元來自 SharePoint 清單，而 **CostToFix** 和 **RevenueOpportunity** 均由函式所傳回。

    完成的流程看起來應類似下圖 (我們省略了第一個 [如果否] 分支以節省空間)。

    ![完成流程](media/functions-flow-scenario/complete-flow.png)

4. 按一下頁面頂端的 [更新流程]，然後按一下 [完成]。

## <a name="run-the-flow"></a>執行流程

既然流程已完成，您可在 SharePoint 清單中新增一個資料列，並查看流程的回應方式。

1. 前往 SharePoint 清單，然後按一下 [快速編輯]。

    ![快速編輯](media/functions-flow-scenario/quick-edit.png)

2. 在編輯方格中輸入下列值。

    | 清單資料行     | 值           |
    |-----------------|---------------------|
    | **標題**           | 渦輪機 60 |
    | **LastServiceDate** | 08/04/2017 |
    | **MaxOutput**       | 2500 |
    | **ServiceRequired** | yes |
    | **EstimatedEffort** | 10 |

3. 按一下 [完成] 。

    ![快速編輯完成](media/functions-flow-scenario/quick-edit-done.png)

    當您新增項目時，它會觸發您接下來要查看的流程。

4. 在 [flow.microsoft.com](https://flow.microsoft.com) 中，按一下 [我的流程]，然後按一下您建立的流程。

    ![我的流程](media/functions-flow-scenario/my-flows.png)

5. 在 [執行歷程記錄] 下方，按一下流程執行。

    ![執行記錄](media/functions-flow-scenario/run-history.png)

    如果執行成功，您可以在下一個頁面上檢閱流程作業。 如果執行基於任何因素而失敗，下一個頁面就會提供疑難排解資訊。

6. 展開卡片以查看流程期間所發生的事項。 例如，展開 [計算成本] 卡片，以查看函式的輸入和輸出。 

    ![計算成本輸入和輸出](media/functions-flow-scenario/calculates-costs-outputs.png)

7. 檢查您在 [傳送電子郵件] 卡片的 [收件者] 欄位中指定之人員的電子郵件帳戶。 從流程傳送的電子郵件看起來應該類似下圖。

    ![流程電子郵件](media/functions-flow-scenario/flow-email.png)

    您可以查看如何使用來自 SharePoint 清單和函式的正確值來取代語彙基元。

## <a name="next-steps"></a>後續步驟
在本主題中，您已了解如何：

> [!div class="checklist"]
> * 在 SharePoint 中建立清單。
> * 匯出 API 定義。
> * 新增 API 連線。
> * 如果修復符合成本效益，就建立流程來傳送電子郵件。
> * 執行流程。

若要深入了解 Microsoft Flow，請參閱[開始使用 Microsoft Flow](https://flow.microsoft.com/documentation/getting-started/)。

若要了解其他使用 Azure Functions 的有趣案例，請參閱[從 PowerApps 呼叫函式](functions-powerapps-scenario.md)和[建立與 Azure Logic Apps 整合的函式](functions-twitter-email.md)。