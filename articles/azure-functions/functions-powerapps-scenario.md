---
title: 從 PowerApps 呼叫函式 | Microsoft Docs
description: 建立自訂連接器，然後使用該連接器來呼叫函式。
services: functions
keywords: 雲端應用程式, 雲端服務, PowerApps, 商務程序, 商務應用程式
author: ggailey777
manager: jeconnoc
ms.assetid: ''
ms.service: azure-functions
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: glenga
ms.reviewer: sunayv
ms.custom: ''
ms.openlocfilehash: 49000d4fd61e7cd061b415d50545a7f5ee56f3fb
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45579118"
---
# <a name="call-a-function-from-powerapps"></a>從 PowerApps 呼叫函式
[PowerApps](https://powerapps.microsoft.com) 平台的設計是為了讓商務專家不需要傳統應用程式程式碼，就能建置應用程式。 專業開發人員可以使用 Azure Functions 來擴充 PowerApps 的功能，同時透過技術詳細資料來防護 PowerApps 應用程式建立器。

您會在本主題中根據風力渦輪機的維護案例建置一個應用程式。 本主題說明如何呼叫您在[為函式建立 OpenAPI 定義](functions-openapi-definition.md)中定義的函式。 此函式可判斷風力渦輪機的緊急修復是否符合成本效益。

![PowerApps 中已完成的應用程式](media/functions-powerapps-scenario/finished-app.png)

如需從 Microsoft Flow 呼叫同一個函式的相關資訊，請參閱[從 Microsoft Flow 呼叫函式](functions-flow-scenario.md)。

在本主題中，您將了解如何：

> [!div class="checklist"]
> * 在 Excel 中準備範例資料。
> * 匯出 API 定義。
> * 新增 API 連線。
> * 建立應用程式並新增資料來源。
> * 新增控制項以檢視應用程式中的資料。
> * 新增控制項以呼叫函式並顯示資料。
> * 執行應用程式以判斷修復是否符合成本效益。

## <a name="prerequisites"></a>必要條件

+ 使用中的 [PowerApps 帳戶](https://docs.microsoft.com/powerapps/maker/signup-for-powerapps)，其登入認證與您的 Azure 帳戶相同。 
+ Excel 以及要作為應用程式資料來源的 [Excel 範例檔案](https://procsi.blob.core.windows.net/docs/turbine-data.xlsx)。
+ 完成[為函式建立 OpenAPI 定義](functions-openapi-definition.md)教學課程。

[!INCLUDE [Export an API definition](../../includes/functions-export-api-definition.md)]

## <a name="add-a-connection-to-the-api"></a>新增 API 連線
自訂 API (也稱為自訂連接器) 適用於 PowerApps，但您必須建立 API 連線，才能在應用程式中使用。

1. 在 [web.powerapps.com](https://web.powerapps.com) 中，按一下 [連線]。

    ![PowerApps 連線](media/functions-powerapps-scenario/powerapps-connections.png)

1. 按一下 [新增連線]，然後向下捲動至 [渦輪機修復] 連接器並按一下。

    ![新增連線](media/functions-powerapps-scenario/new-connection.png)

1. 輸入 API 金鑰，然後按一下 [建立]。

    ![建立連線](media/functions-powerapps-scenario/create-connection.png)

> [!NOTE]
> 如果您與其他人共用應用程式，處理或使用應用程式的所有人也必須輸入 API 金鑰才能連線到 API。 此行為在未來可能會變更，我們將會更新本主題，以反映這一點。

## <a name="create-an-app-and-add-data-sources"></a>建立應用程式並新增資料來源
現在，您已準備好在 PowerApps 中建立應用程式，並新增 Excel 資料和自訂 API 作為應用程式的資料來源。

1. 在 [web.powerapps.com](https://web.powerapps.com) 中，選擇 [從空白開始] > ![手機應用程式圖示](media/functions-powerapps-scenario/icon-phone-app.png) (手機) > [製作此應用程式]。

    ![從空白開始 - 手機應用程式](media/functions-powerapps-scenario/create-phone-app.png)

    此應用程式會在適用於 Web 的 PowerApps Studio 中開啟。 下圖顯示 PowerApps Studio 的不同部分。

    ![PowerApps Studio](media/functions-powerapps-scenario/powerapps-studio.png)

    **(A) 左導覽列**，您會在其中看到每個畫面上所有控制項的階層檢視

    **(B) 中間窗格**，顯示您正在工作的畫面

    **(C) 右窗格**，您可以在其中設定版面配置和資料來源等選項

    **(D) [屬性]** 下拉式清單，您可以在其中選取要套用公式的屬性

    **(E) 公式列**，您可以在其中新增定義應用程式行為的公式 (如 Excel 所示)
    
    **(F) 功能區**，您可以在其中新增控制項並自訂設計項目

1. 新增 Excel 檔案作為資料來源。

    您會匯入的資料如下所示：

    ![要匯入的 Excel 資料](media/functions-powerapps-scenario/excel-table.png)

    1. 在應用程式畫布上，選擇 [連線到資料]。

    1. 在 [資料] 面板中，按一下 [對應用程式新增靜態資料]。

        ![新增資料來源](media/functions-powerapps-scenario/add-static-data.png)

        在一般情況下，您可以從外部來源讀取和寫入資料，但由於這是範例，因此您將新增 Excel 資料作為靜態資料。

    1. 巡覽至您儲存的 Excel 檔案，選取 [渦輪機] 表格，然後按一下 [連線]。

        ![新增資料來源](media/functions-powerapps-scenario/choose-table.png)


1. 新增自訂 API 作為資料來源。

    1. 在 [資料] 面板上，按一下 [新增資料來源]。

    1. 按一下 [渦輪機修復]。

        ![[渦輪機修復] 連接器](media/functions-powerapps-scenario/turbine-connector.png)

## <a name="add-controls-to-view-data-in-the-app"></a>新增控制項以檢視應用程式中的資料
現在，資料來源可供應用程式使用，您要將一個畫面新增至應用程式，以便檢視渦輪機資料。

1. 在 [首頁] 索引標籤上，按一下 [新增畫面] > [清單畫面]。

    ![清單畫面](media/functions-powerapps-scenario/list-screen.png)

    PowerApps 會新增一個畫面，其中包含用以顯示項目的「資源庫」，以及可進行搜尋、排序和篩選的其他控制項。

1. 將標題列變更為 `Turbine Repair`，並調整資源庫大小以挪出空間給更多控制項。

    ![變更標題並調整資源庫大小](media/functions-powerapps-scenario/gallery-title.png)

1. 在已選取資源庫的情況下，於右窗格的 [屬性] 底下，按一下 [CustomGallerySample]。

    ![變更資料來源](media/functions-powerapps-scenario/change-data-source.png)

1. 在 [資料] 面板中，從清單選取 [渦輪機]。

    ![選取資料來源](media/functions-powerapps-scenario/select-data-source.png)

    資料集未包含影像，因此接下來您要變更為更適合資料的版面配置。 

1. 仍在 [資料] 窗格中，將 [版面配置] 變更為 [標題、子標題與本文]。

    ![變更資源庫版面配置](media/functions-powerapps-scenario/change-layout.png)

1. 最後一個步驟是在 [資料] 面板中，變更資源庫中所顯示的欄位。

    ![變更資源庫欄位](media/functions-powerapps-scenario/change-fields.png)
    
    + **本文1** = 上次維修日期
    + **子標題2** = 需要維修
    + **標題2** = 標題 

1. 在選取資源庫時，將 **TemplateFill** 屬性設定為下列公式：`If(ThisItem.IsSelected, Orange, White)`。

    ![範本填滿公式](media/functions-powerapps-scenario/formula-fill.png)

    現在，您可以更輕鬆地看到選取了哪一個資源庫項目。

    ![選取的項目](media/functions-powerapps-scenario/selected-item.png)

1. 您不需要應用程式中的原始畫面。 在左窗格中，將滑鼠停留在 [畫面1] 上，然後依序按一下 [...] 和 [刪除]。

    ![刪除畫面](media/functions-powerapps-scenario/delete-screen.png)

1. 按一下 [檔案]，並為應用程式命名。 按一下左側功能表上的 [儲存]，然後按一下右下角的 [儲存]。

您通常會在生產環境應用程式中執行許多其他格式化工作，但我們將前往此案例的重要部分，那就是呼叫函式。

## <a name="add-controls-to-call-the-function-and-display-data"></a>新增控制項以呼叫函式並顯示資料
您有一個應用程式會顯示每個渦輪機的摘要資料，現在可以新增控制項以呼叫您所建立的函式，並顯示傳回的資料。 您可以根據在 OpenAPI 定義中命名函式的方式來存取函式，在本例中為 `TurbineRepair.CalculateCosts()`。

1. 在功能區的 [插入] 索引標籤上，按一下 [按鈕]。 然後在相同的索引標籤上，按一下 [標籤]

    ![插入按鈕和標籤](media/functions-powerapps-scenario/insert-controls.png)

1. 將按鈕和標籤拖曳至資源庫下方，並調整標籤大小。 

1. 選取按鈕文字，並將它變更為 `Calculate costs`。 應用程式看起來應該如下圖所示。

    ![含有按鈕的應用程式](media/functions-powerapps-scenario/move-button-label.png)

1. 選取按鈕，然後針對按鈕的 **OnSelect** 屬性輸入下列公式。

    ```
    If (BrowseGallery1.Selected.ServiceRequired="Yes", ClearCollect(DetermineRepair, TurbineRepair.CalculateCosts({hours: BrowseGallery1.Selected.EstimatedEffort, capacity: BrowseGallery1.Selected.MaxOutput})))
    ```
    此公式會在按下按鈕時執行，並在所選取資源庫項目的 [需要維修] 值為 `Yes` 時執行下列動作：

    + 清除「集合」`DetermineRepair` 以移除先前呼叫的資料。 集合是表格式變數。

    + 將呼叫函式 `TurbineRepair.CalculateCosts()` 所傳回的資料指派給集合。 
    
        傳遞至函式的值是來自資源庫中所選項目的 [預計投入的時間] 和 [最大輸出] 欄位。 這些欄位不會顯示在資源庫中，但仍可在公式中使用。

1. 選取標籤，然後輸入下列公式作為標籤的 **Text** 屬性。

    ```
    "Repair decision: " & First(DetermineRepair).message & " | Cost: " & First(DetermineRepair).costToFix & " | Revenue: " & First(DetermineRepair).revenueOpportunity
    ```
    此公式會使用 `First()` 函式只存取 `DetermineRepair` 集合的第一列。 接著會顯示函式所傳回的三個值：`message`、`costToFix` 和 `revenueOpportunity`。 第一次執行應用程式之前，這些值為空白。

    完成的應用程式看起來應該如下圖所示。

    ![執行前已完成的應用程式](media/functions-powerapps-scenario/finished-app-before-run.png)


## <a name="run-the-app"></a>執行應用程式
您有一個完整的應用程式！ 現在可以執行並查看函式呼叫的運作狀況。

1. 在 PowerApps Studio 的右上角，按一下 [執行] 按鈕： ![執行應用程式按鈕](media/functions-powerapps-scenario/f5-arrow-sm.png).

1. 選取 [需要維修] 的值為 [`Yes`] 的渦輪機，然後按一下 [計算成本] 按鈕。 您應該會看到如下圖所示的結果。

    ![PowerApps 中已完成的應用程式](media/functions-powerapps-scenario/finished-app.png)

1. 試試看其他渦輪機，以查看函式每次傳回的結果。

## <a name="next-steps"></a>後續步驟
在本主題中，您已了解如何：

> [!div class="checklist"]
> * 在 Excel 中準備範例資料。
> * 匯出 API 定義。
> * 新增 API 連線。
> * 建立應用程式並新增資料來源。
> * 新增控制項以檢視應用程式中的資料。
> * 新增控制項以呼叫函式並顯示資料
> * 執行應用程式以判斷修復是否符合成本效益。

若要深入了解 PowerApps，請參閱 [PowerApps 簡介](https://powerapps.microsoft.com/tutorials/getting-started/)。

若要了解其他使用 Azure Functions 的有趣情節，請參閱[從 Microsoft Flow 呼叫函式](functions-flow-scenario.md)和[建立與 Azure Logic Apps 整合的函式](functions-twitter-email.md)。
