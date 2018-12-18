---
title: 使用 Logic Apps 自動執行 Azure Application Insights 程序。
description: 了解如何透過將 Application Insights Connector 新增到您的邏輯應用程式，快速自動執行重複程序。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/29/2017
ms.author: mbullwin
ms.openlocfilehash: 91b5c2c23445e5cd3445d1d5b640cb3ecb8e5e7a
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301186"
---
# <a name="automate-application-insights-processes-by-using-logic-apps"></a>使用 Logic Apps 自動執行 Application Insights 程序

您是否發現自己在遙測資料上不斷重複地執行相同查詢，以檢查服務是否正常運作？ 想要將此類查詢自動化以尋找趨勢和異常，然後針對它們建立您自己的工作流程嗎？ 適用於 Logic Apps 的 Application Insights Connector (預覽) 正是符合此用途的工具。

有了此整合，您就能自動執行許多流程，而不需撰寫任何一行程式碼。 您可以使用 Application Insights Connector 建立邏輯應用程式，以快速自動執行任何 Application Insights 程序。 

您也可以新增額外的動作。 Azure App Service 的 Logic Apps 功能可讓您提供數百個動作。 例如，藉由使用邏輯應用程式，您可以自動傳送電子郵件通知，或在 Azure DevOps 中建立 Bug。 您也可以使用其中一個可用的[範本](https://docs.microsoft.com/azure/logic-apps/logic-apps-use-logic-app-templates)，加快建立邏輯應用程式的程序。 

## <a name="create-a-logic-app-for-application-insights"></a>建立 Application Insights 的邏輯應用程式

在本教學課程中，您會學習如何建立一個邏輯應用程式，該程式會使用 Analytics 自動叢集演算法將 Web 應用程式資料中的屬性分類。 此流程會以電子郵件自動傳送結果，這只是如何將 Application Insights Analytics 與 Logic Apps 一起使用的一個範例。 

### <a name="step-1-create-a-logic-app"></a>步驟 1：建立邏輯應用程式
1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 按一下 [建立資源]，選取 [Web + 行動]，然後選取 [邏輯應用程式]。

    ![新增邏輯應用程式視窗](./media/automate-with-logic-apps/logicapp1.png)

### <a name="step-2-create-a-trigger-for-your-logic-app"></a>步驟 2：建立邏輯應用程式的觸發程序
1. 在 [邏輯應用程式設計工具] 視窗的 [以一般觸發程序開始] 之下，選取 [循環]。

    ![邏輯應用程式設計工具視窗](./media/automate-with-logic-apps/logicapp2.png)

1. 在 [頻率] 方塊中選取 [天]，然後在 [間隔] 方塊中輸入 **1**。

    ![邏輯應用程式設計工具 [參考] 視窗](./media/automate-with-logic-apps/step2b.png)

### <a name="step-3-add-an-application-insights-action"></a>步驟 3：新增 Application Insights 動作
1. 按一下 [新增步驟]，然後按一下 [新增動作]。

1. 在 [選擇動作] 搜尋方塊中，輸入 **Azure Application Insights**。

1. 在 [動作] 之下，按一下 [Azure Application Insights – 視覺化 Analytics 查詢預覽]。

    ![邏輯應用程式設計工具的「選擇動作」視窗](./media/automate-with-logic-apps/flow2.png)

### <a name="step-4-connect-to-an-application-insights-resource"></a>步驟 4：連線到 Application Insights 資源

若要完成此步驟，您需要資源的應用程式識別碼和 API 金鑰。 您可以從 Azure 入口網站擷取這些資訊，如下圖所示：

![Azure 入口網站中的應用程式識別碼](./media/automate-with-logic-apps/appid.png) 

提供您的連線名稱、應用程式識別碼和 API 金鑰。

![邏輯應用程式設計工具連線視窗](./media/automate-with-logic-apps/flow3.png)

### <a name="step-5-specify-the-analytics-query-and-chart-type"></a>步驟 5：指定 Analytics 查詢和圖表類型
在下列範例中，查詢會選取最後一天內的失敗要求，並將它們與作業發生的例外狀況相互關聯。 Analytics 會根據 operation_Id 識別碼，讓失敗的要求相互關聯。 查詢接著會使用自動叢集演算法將結果分段。 

建立自己的查詢時，先確認它們可在 Analytics 中正常運作，再將其新增到您的流程中。

1. 在 [查詢] 方塊中，新增下列 Analytics 查詢： 

    ```
    requests
    | where timestamp > ago(1d)
    | where success == "False"
    | project name, operation_Id
    | join ( exceptions
        | project problemId, outerMessage, operation_Id
    ) on operation_Id
    | evaluate autocluster()
    ```

1. 在 [圖表類型] 方塊中，選取 [Html 表格]。

    ![Analytics 查詢設定畫面](./media/automate-with-logic-apps/flow4.png)

### <a name="step-6-configure-the-logic-app-to-send-email"></a>步驟 6：設定邏輯應用程式以傳送電子郵件

1. 按一下 [新增步驟]，然後選取 [新增動作]。

1. 在搜尋方塊中，輸入**Office 365 Outlook**。

1. 按一下 [Office 365 Outlook – 傳送電子郵件]。

    ![Office 365 Outlook 選項](./media/automate-with-logic-apps/flow2b.png)

1. 在 [傳送電子郵件]  視窗中，執行下列動作：

   a. 輸入收件者的電子郵件地址。

   b. 輸入電子郵件的主旨。

   c. 按一下 [內文] 方塊中的任意處，然後在右方開啟的動態內容功能表上，選取 [內文]。

   d. 按一下 [顯示進階選項]。

      ![Office 365 Outlook 設定](./media/automate-with-logic-apps/flow5.png)

1. 在動態內容功能表上執行下列動作：

    a. 選取 [附件名稱]。

    b. 選取 [附件內容]。
    
    c. 在 [為 HTML] 方塊中選取 [是]。

      ![Office 365 電子郵件設定畫面](./media/automate-with-logic-apps/flow7.png)

### <a name="step-7-save-and-test-your-logic-app"></a>步驟 7：儲存並測試邏輯應用程式
* 按一下 [確定] 儲存變更。

您可以等待觸發程式執行邏輯應用程式，也可以選取 [執行] 來立即執行邏輯應用程式。

![邏輯應用程式建立畫面](./media/automate-with-logic-apps/step7.png)

當您的邏輯應用程式執行時，您在電子郵件清單中指定的收件者將收到電子郵件，如下所示：

![邏輯應用程式電子郵件訊息](./media/automate-with-logic-apps/flow9.png)

## <a name="next-steps"></a>後續步驟

- 深入了解建立 [Analytics 查詢](app-insights-analytics-using.md)。
- 深入了解 [Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)。



<!--Link references-->





