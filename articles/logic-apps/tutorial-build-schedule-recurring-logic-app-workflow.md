---
title: 建置以排程為基礎的自動化工作流程 - Azure Logic Apps | Microsoft Docs
description: 教學課程 - 如何使用 Azure Logic Apps 建立以排程為基礎的週期性自動化工作流程
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: tutorial
ms.custom: mvc
ms.date: 01/12/2018
ms.openlocfilehash: 70d7adc9ea8c65a7e6a21165864b7e6d661c6f8c
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48042286"
---
# <a name="check-traffic-on-a-schedule-with-azure-logic-apps"></a>使用 Azure Logic Apps 來檢查排程上的流量

Azure Logic Apps 可協助您自動化定期執行的工作流程。 本教學課程說明如何建置具有排程器觸發程序的[邏輯應用程式](../logic-apps/logic-apps-overview.md)，而該觸發程序會在每個工作天早上執行並檢查兩地間的行進時間 (包括流量)。 如果時間超過特定限制，則邏輯應用程式會傳送包含行進時間和前往目的地所需額外時間的電子郵件。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立空白邏輯應用程式。 
> * 為邏輯應用程式新增作為排程器運作的觸發程序。
> * 新增可取得路線行進時間的動作。
> * 新增可建立變數、將行進時間從秒鐘轉換為分鐘，並將該結果儲存在變數中的動作。
> * 新增可針對指定的限制比較行進時間的條件。
> * 新增可在行進時間超過限制時傳送電子郵件的動作。

當您完成時，邏輯應用程式大致如下列工作流程所示︰

![邏輯應用程式概觀](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-overview.png)

如果您沒有 Azure 訂用帳戶，請在開始前，先<a href="https://azure.microsoft.com/free/" target="_blank">註冊一個免費的 Azure 帳戶</a>。

## <a name="prerequisites"></a>必要條件

* Logic Apps 支援的任何電子郵件提供者 (例如 Office 365 Outlook、Outlook.com 或 Gmail) 所提供的電子郵件帳戶。 對於其他提供者，請[檢閱這裡的連接器清單](https://docs.microsoft.com/connectors/)。 本快速入門使用 Outlook.com 帳戶。 如果您使用不同的電子郵件帳戶，整體步驟將維持不變，但您的 UI 外觀可能會略有不同。

* 若要取得路線的行進時間，您需要 Bing 地圖服務 API 的存取金鑰。 若要取得此金鑰，請遵循<a href="https://msdn.microsoft.com/library/ff428642.aspx" target="_blank">如何取得 Bing 地圖服務金鑰</a>的步驟。 

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

使用您的 Azure 帳戶認證登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。

## <a name="create-your-logic-app"></a>建立邏輯應用程式

1. 從主要 Azure 功能表，選擇 [建立資源] > [企業整合] > [邏輯應用程式]。

   ![建立邏輯應用程式](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-logic-app.png)

2. 在 [建立邏輯應用程式] 下，提供有關於邏輯應用程式的這項資訊，如下所示和描述。 完成時，請選擇 [釘選到儀表板] > [建立]。

   ![提供邏輯應用程式資訊](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-logic-app-settings.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **名稱** | LA-TravelTime | 邏輯應用程式的名稱 | 
   | **訂用帳戶** | <your-Azure-subscription-name> | Azure 訂用帳戶的名稱 | 
   | **資源群組** | LA-TravelTime-RG | 用來組織相關資源之 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)的名稱 | 
   | **位置** | 美國東部 2 | 用來存放邏輯應用程式相關資訊的區域 | 
   | **Log Analytics** | 關閉 | 保留診斷記錄的 [關閉] 設定。 | 
   |||| 

3. 在 Azure 部署您的應用程式之後，Logic Apps 設計工具隨即開啟，並顯示含有簡介影片和常用邏輯應用程式模式範本的頁面。 在 [範本] 底下，選擇 [空白邏輯應用程式]。

   ![選擇空白邏輯應用程式範本](./media/tutorial-build-scheduled-recurring-logic-app-workflow/choose-logic-app-template.png)

接下來，新增週期[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)，該觸發程序會根據指定的排程引發。 每個邏輯應用程式都必須使用觸發程序啟動，而該觸發程序會在特定事件發生或新資料符合特定條件時引發。 如需詳細資訊，請參閱[建立您的第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

## <a name="add-scheduler-trigger"></a>新增排程器觸發程序

1. 在設計工具的搜尋方塊中輸入 [週期]。 選取此觸發程序：**排程 - 重複**

   ![尋找並新增 [排程 - 週期] 觸發程序](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-schedule-recurrence-trigger.png)

2. 在 [週期] 圖形中，選擇**省略符號** (**...**) 按鈕，然後選擇 [重新命名]。 以下列描述為觸發程序重新命名：```Check travel time every weekday morning```

   ![為觸發程序重新命名](./media/tutorial-build-scheduled-recurring-logic-app-workflow/rename-recurrence-schedule-trigger.png)

3. 在觸發程序內部，選擇 [顯示進階選項]。

4. 為您的觸發程序提供排程和週期詳細資料，如下所示和描述：

   ![提供排程和週期詳細資料](./media/tutorial-build-scheduled-recurring-logic-app-workflow/schedule-recurrence-trigger-settings.png)

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **間隔** | 1 | 在檢查之間所要等待的間隔數目 | 
   | **頻率** | 週 | 要用於週期的時間單位 | 
   | **時區** | None | 只有在指定開始時間時套用。 很適合用來指定非當地時區。 | 
   | **開始時間** | None | 將週期延遲至特定日期和時間。 如需詳細資訊，請參閱[排程定期執行的工作和工作流程](../connectors/connectors-native-recurrence.md)。 | 
   | **在這幾天內** | 星期一、星期二、星期三、星期四、星期五 | 僅適用於 [頻率] 設定為 [週] 時。 | 
   | **在這幾小時內** | 7、8、9 | 僅適用於 [頻率] 設定為 [週] 或 [天] 時。 選取要執行此週期的整點。 這個範例會在上午 7、8 和 9 點整點執行。 | 
   | **在這幾分鐘內** | 0、15、30、45 | 僅適用於 [頻率] 設定為 [週] 或 [天] 時。 選取要執行此週期的時刻。 這個範例會每隔 15 分鐘執行一次 (從整點開始算起)。 | 
   ||||

   此觸發程序會在每個工作天、每隔 15 分鐘引發一次 (從上午 7:00 開始到上午 9:45 結束)。 
   [預覽] 方塊會顯示週期排程。 
   如需詳細資訊，請參閱[排程工作和工作流程](../connectors/connectors-native-recurrence.md)及[工作流程動作和觸發程序](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger)。

5. 若要立即隱藏觸發程序的詳細資料，請按一下圖形的標題列內部。

   ![摺疊圖形以隱藏詳細資料](./media/tutorial-build-scheduled-recurring-logic-app-workflow/collapse-trigger-shape.png)

6. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。 

邏輯應用程式目前作用中，但不會執行週期以外的任何其他作業。 因此，新增會在觸發程序引發時回應的動作。

## <a name="get-the-travel-time-for-a-route"></a>取得路線的行進時間

有了觸發程序之後，請新增可取得兩地間行進時間的[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)。 Logic Apps 會提供 Bing 地圖服務 API 的連接器，以便輕鬆地取得此資訊。 在開始這項工作之前，請確定您擁有本教學課程必要條件中所述的 Bing 地圖服務 API 金鑰。

1. 在 Logic Apps 設計工具中，於您的觸發程序之下，選擇 [+新增步驟] > [新增動作]。

2. 搜尋「地圖」，然後選取此動作：**Bing 地圖服務 - 取得路線**

3. 如果您沒有 Bing 地圖服務連線，系統會要求您建立連線。 提供以下連線詳細資料，然後選擇 [建立]。

   ![選取 [Bing 地圖服務 - 取得路線] 動作](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-maps-connection.png)

   | 設定 | 值 | 說明 |
   | ------- | ----- | ----------- |
   | 連線名稱 | BingMapsConnection | 為您的連線提供一個名稱。 | 
   | **API 金鑰** | <*your-Bing-Maps-key*> | 輸入您先前收到的 Bing 地圖服務金鑰。 如果您沒有 Bing 地圖服務金鑰，請了解<a href="https://msdn.microsoft.com/library/ff428642.aspx" target="_blank">如何取得金鑰</a>。 | 
   | | | |  

4. 以下列描述為動作重新命名：```Get route and travel time with traffic```

5. 提供 [取得路線] 動作的詳細資料，如下面範例所示和描述，例如：

   ![提供 [Bing 地圖服務 - 取得路線] 動作的資訊](./media/tutorial-build-scheduled-recurring-logic-app-workflow/get-route-action-settings.png) 

   | 設定 | 值 | 說明 |
   | ------- | ----- | ----------- |
   | **導航點 1** | <*start-location*> | 您的路線起點 | 
   | **導航點 2** | <*end-location*> | 您的路線目的地 | 
   | **避開** | None | 您的路線所要避開的項目，例如高速公路、收費站等等 | 
   | **最佳化** | timeWithTraffic | 可將您的路線最佳化的參數，例如距離、目前流量的行進時間等等。 選取此參數："timeWithTraffic" | 
   | **距離單位** | <*your-preference*> | 路線的距離單位。 本文使用此單位：[英哩]  | 
   | **行進模式** | 開車 | 路線的行進模式。 選取此模式：[開車] | 
   | **運輸日期時間** | None | 僅適用於運輸模式 | 
   | **日期時間類型** | None | 僅適用於運輸模式 | 
   |||| 

   如需這些參數的詳細資訊，請參閱[計算路線](https://msdn.microsoft.com/library/ff701717.aspx)。

6. 儲存您的邏輯應用程式。

接下來，建立一個變數，以便將目前行進時間轉換為分鐘 (而不是秒鐘) 並加以儲存。 如此一來，即可避免重複轉換，並且在後續步驟中更輕鬆使用此值。 

## <a name="create-variable-to-store-travel-time"></a>建立變數來儲存行進時間

有時候，您可以在工作流程中對資料執行作業，並在後續動作中使用其結果。 若要儲存這些結果，以便輕鬆地重複使用或參考，您可以建立一些變數，以在處理後儲存這些結果。 您只能在邏輯應用程式的最上層建立變數。

根據預設，前一個 [取得路線] 動作會透過 [行進持續時間流量] 欄位，傳回包含流量的目前行進時間 (以秒為單位)。 藉由轉換此值並改以分鐘為單位進行儲存，即可讓此值日後更容易重複使用，而不需再次轉換。

1. 在 [取得路線] 動作下方，選擇 [+ 新增步驟] > [新增動作]。

2. 搜尋「變數」，然後選取此動作：**變數 - 初始化變數**

   ![選取 [變數 - 初始化變數] 動作](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-initialize-variable-action.png)

3. 以下列描述為動作重新命名：```Create variable to store travel time```

4. 為您的變數提供如下所述的詳細資料：

   | 設定 | 值 | 說明 | 
   | ------- | ----- | ----------- | 
   | **名稱** | travelTime | 您的變數名稱 | 
   | **類型** | 整數  | 您的變數資料類型 | 
   | **值** | 可將目前行進時間從秒數轉換為分鐘數的運算式 (請參閱此表格底下的步驟)。 | 您的變數初始值 | 
   |||| 

   1. 若要建立 [值] 欄位的運算式，請按一下此欄位內部，動態內容清單隨即出現。 
   必要時，讓您的瀏覽器變寬，直到清單出現為止。 
   在動態內容清單中，選擇 [運算式]。 

      ![提供 [變數 - 初始化變數] 動作的資訊](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings.png)

      當您在特定編輯方塊內按一下時，隨即出現動態內容清單或內嵌參數清單。 這份清單會顯示先前動作中可作為工作流程中輸入的任何參數。 
      動態內容清單有一個運算式編輯器，您可以在其中選取用來執行作業的函式。 
      此運算式編輯器只會顯示在動態內容清單中。

      您的瀏覽器寬度可決定哪一份清單會出現。 
      如果您的瀏覽器很寬，則會出現動態內容清單。 
      如果您的瀏覽器很窄，則參數清單會以內嵌方式顯示在目前具有焦點的編輯方塊底下。

   2. 在運算式編輯器中，輸入此運算式：```div(,60)```

      ![輸入此運算式："div(,60)"](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-2.png)

   3. 將游標放在運算式內部的左括號 (**(**) 與逗號 (**,**) 之間。 
   選擇 [動態內容]。

      ![放置游標，選擇 [動態內容]](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-3.png)

   4. 在動態內容清單中，選取 [行進持續時間流量]。

      ![選取 [行進持續時間流量] 欄位](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-4.png)

   5. 在運算式內解析此欄位之後，請選擇 [確定]。

      ![選擇 [確定]](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-5.png)

      [值] 欄位隨即出現，如下所示：

      ![包含已解析運算式的 [值] 欄位](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-6.png)

5. 儲存您的邏輯應用程式。

接下來，新增可檢查目前行進時間是否大於特定限制的條件。

## <a name="compare-travel-time-with-limit"></a>比較行進時間與限制

1. 在前一個動作之下，選擇 [+ 新增步驟] > [新增條件]。 

2. 以下列描述為條件重新命名：```If travel time exceeds limit```

3. 建立一個條件，以檢查 **travelTime** 是否超過您指定的限制，如下所描述和顯示：

   1. 在此條件內，按一下位於左邊 (寬型瀏覽器檢視) 或頂端 (窄型瀏覽器檢視) 的 [選擇值] 方塊內部。

   2. 從動態內容清單或參數清單中，選取 [變數] 下方的 [travelTime] 欄位。

   3. 在比較方塊中，選取此運算子：**大於**

   4. 在右側 (寬型檢視) 或底部 (窄型檢視) 的 [選擇值] 方塊中，輸入此限制：```15```

   例如，如果您在窄型檢視中作業，以下是您建立此條件的方式：

   ![在窄型檢視中建立條件](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-check-travel-time-narrow.png)

4. 儲存您的邏輯應用程式。

接下來，新增當行進時間超過限制時所要執行的動作。

## <a name="send-email-when-limit-exceeded"></a>在超過限制時傳送電子郵件

現在，新增可在行進時間超過限制時傳送電子郵件給您的動作。 此電子郵件包含目前行進時間和行經指定路線所需的額外時間。 

1. 在條件的 [若為 true] 分支中，選擇 [新增動作]。

2. 搜尋 [傳送電子郵件]，然後選取電子郵件連接器和您要使用的 [傳送電子郵件動作]。

   ![尋找並選取 [傳送電子郵件] 動作](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-action-send-email.png)

   * 對於個人 Microsoft 帳戶，選取 [Outlook.com]。 
   * 對於 Azure 工作或學校帳戶，選取 [Office 365 Outlook]。

3. 如果您還沒有連線，系統會要求您登入您的電子郵件帳戶。

   Logic Apps 會建立與電子郵件帳戶的連線。

4. 以下列描述為動作重新命名：```Send email with travel time```

5. 在 [收件者] 方塊中，輸入收件者的電子郵件地址。 若要進行測試，請使用您的電子郵件地址。

6. 在 [主旨] 方塊中，指定電子郵件的主旨，以及包含 [travelTime] 變數。

   1. 輸入尾端有一個空格的文字 ```Current travel time (minutes): ```。 
   
   2. 從參數清單或動態內容清單中，選取 [變數] 下方的 [travelTime]。 
   
      例如，如果您的瀏覽器以窄型檢視呈現：

      ![輸入主旨文字及傳回行進時間的運算式](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-subject-settings.png)

7. 在 [內文] 方塊中，指定電子郵件內文的內容。 

   1. 輸入尾端有一個空格的文字 ```Add extra travel time (minutes): ```。 
   
   2. 必要時，讓您的瀏覽器變寬，直到動態內容清單出現為止。 
   在動態內容清單中，選擇 [運算式]。

      ![建立電子郵件內文的運算式](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings.png)

   3. 在運算式編輯器中，輸入此運算式，以便計算超過限制的分鐘數：```sub(,15)```

      ![輸入運算式來計算額外的行進時間分鐘數](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-2.png)

   4. 將游標放在運算式內部的左括號 (**(**) 與逗號 (**,**) 之間。 選擇 [動態內容]。

      ![繼續建立運算式來計算額外的行進時間分鐘數](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-3.png)

   5. 在 [變數] 下方，選取 [travelTime]。

      ![選取要在運算式中使用的 [travelTime] 欄位](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-4.png)

   6. 在運算式內解析此欄位之後，請選擇 [確定]。

      ![包含已解析運算式的 [內文] 欄位](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-5.png)

      [內文] 欄位隨即出現，如下所示：

      ![包含已解析運算式的 [內文] 欄位](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-6.png)

8. 儲存您的邏輯應用程式。

接著請測試您的邏輯應用程式，此時它看起來類似下列範例：

![完成的邏輯應用程式](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-finished.png)

## <a name="run-your-logic-app"></a>執行邏輯應用程式

若要手動啟動邏輯應用程式，在設計工具工具列上，選擇 [執行]。 如果目前行進時間保持在您的限制之內，邏輯應用程式並不會執行任何作業，而會等到下一個間隔後再次檢查。
但是，如果目前行進時間超過您的限制，您會收到一封電子郵件，其中包含目前行進時間和超過限制的分鐘數。 以下是邏輯應用程式傳送的範例電子郵件：

![包含行進時間的已傳送電子郵件](./media/tutorial-build-scheduled-recurring-logic-app-workflow/email-notification.png)

若未收到任何電子郵件，請檢查電子郵件的垃圾郵件資料夾。 您的垃圾電子郵件篩選器可能會重新導向這類郵件。 或者，如果您不確定邏輯應用程式是否正確執行，請參閱[針對邏輯應用程式進行疑難排解](../logic-apps/logic-apps-diagnosing-failures.md)。

恭喜，您已經建立並執行以排程為基礎的週期性邏輯應用程式。 

若要建立使用 [排程 - 週期] 觸發程序的其他邏輯應用程式，請簽出在您建立邏輯應用程式之後可用的下列範本：

* 讓每日提醒傳送給您。
* 刪除較舊的 Azure blob。
* 將訊息新增至 Azure 儲存體佇列。

## <a name="clean-up-resources"></a>清除資源

如果不再需要，請刪除包含邏輯應用程式的資源群組和相關資源。 在 Azure 主要功能表上，移至 [資源群組]，然後選取您邏輯應用程式的資源群組。 選擇 [刪除資源群組]。 輸入資源群組名稱作為確認，然後選擇 [刪除]。

![[概觀] > [刪除資源群組]](./media/tutorial-build-scheduled-recurring-logic-app-workflow/delete-resource-group.png)

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立一個邏輯應用程式，該邏輯應用程式可根據指定的排程 (在工作日上午) 檢查流量，並且在行進時間超過指定的限制時採取動作 (傳送電子郵件)。 現在，藉由 Azure 服務、Microsoft 服務和其他 SaaS 應用程式的整合，了解如何建立可傳送郵寄清單要求以供核准的邏輯應用程式。

> [!div class="nextstepaction"]
> [管理郵寄清單要求](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
