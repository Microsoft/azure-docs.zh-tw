---
title: 將含有 Azure Application Insights 資料的自訂報表自動化
description: 將含有 Azure Application Insights 資料的自訂每天/每週/每月報表自動化
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/25/2018
ms.reviewer: sdash
ms.author: mbullwin
ms.openlocfilehash: a3f2021a1a1e0a0ff97c5966e151894fd7b6743b
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574636"
---
# <a name="automate-custom-reports-with-azure-application-insights-data"></a>將含有 Azure Application Insights 資料的自訂報表自動化

定期的報表有助於讓小組隨時了解商務關鍵服務的執行狀況。 開發人員、DevOps/SRE 小組及其主管可藉由能可靠地提供深入解析的自動化報表來提升生產力，而無須要求每個人都登入入口網站。 這類報表也有助於識別逐漸增加的延遲、負載或失敗率，這些項目可能不會觸發任何警示規則。

每個企業都有其獨特的報表需求，例如： 

* 計量的特定百分位數彙總，或自訂報表中的計量。
* 每天、每週和每月的資料彙總有不同報表，適用於不同對象。
* 依自訂的屬性 (例如區域或環境) 分割。 
* 要將某些 AI 資源一起分組在單一報表中，即使它們可能在不同的訂用帳戶或資源群組當中也一樣。
* 將包含敏感性計量的報表分別傳送至特定對象。
* 提供報表給可能沒有入口網站資源存取權限的專案關係人。

> [!NOTE] 
> 每週的 Application Insights 摘要電子郵件不允許任何自訂，且將改為下列自訂選項而中止。 在 2018 年 6 月 11 日將傳送最後的每週摘要電子郵件。 請設定下列其中一個選項，來取的類似的自訂報表 (使用下列建議的查詢)。

## <a name="to-automate-custom-report-emails"></a>將自訂報表電子郵件自動化

您可以[以程式設計方式查詢 Application Insights](https://dev.applicationinsights.io/) 資料，來依排程產生自訂報表。 下列選項可協助您快速入門：

* [透過 Microsoft Flow 進行報表自動化](app-insights-automate-with-flow.md)
* [透過 Logic Apps 進行報表自動化](automate-with-logic-apps.md)
* 在監視案例中使用 "Application Insights 排程摘要" [Azure 函式](https://docs.microsoft.com/azure/azure-functions/functions-create-first-azure-function)範本。 此函式會使用 SendGrid 來傳送電子郵件。 

    ![Azure 函式範本](./media/automate-custom-reports/azure-function-template.png)

## <a name="sample-query-for-a-weekly-digest-email"></a>每週摘要電子郵件的範例查詢
下列查詢顯示跨多個資料集聯結，來取得有如報表般的每週摘要電子郵件。 您可以視需要加以自訂，並搭配上面所列的任何選項使用以將每週報表自動化。   

```AIQL
let period=7d;
requests
| where timestamp > ago(period)
| summarize Row = 1, TotalRequests = sum(itemCount), FailedRequests = sum(toint(success == 'False')),
    RequestsDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
| join (
dependencies
| where timestamp > ago(period)
| summarize Row = 1, TotalDependencies = sum(itemCount), FailedDependencies = sum(success == 'False'),
    DependenciesDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
) on Row | join (
pageViews
| where timestamp > ago(period)
| summarize Row = 1, TotalViews = sum(itemCount)
) on Row | join (
exceptions
| where timestamp > ago(period)
| summarize Row = 1, TotalExceptions = sum(itemCount)
) on Row | join (
availabilityResults
| where timestamp > ago(period)
| summarize Row = 1, OverallAvailability = iff(isnan(avg(toint(success))), '------', tostring(toint(avg(toint(success)) * 10000) / 100.0)),
    AvailabilityDuration = iff(isnan(avg(duration)), '------', tostring(toint(avg(duration) * 100) / 100.0))
) on Row
| project TotalRequests, FailedRequests, RequestsDuration, TotalDependencies, FailedDependencies, DependenciesDuration, TotalViews, TotalExceptions, OverallAvailability, AvailabilityDuration
```

## <a name="application-insights-scheduled-digest-report"></a>Application Insights 排程摘要報表

1. 從 Azure 入口網站中，選取 [建立資源] > [計算] > [函數應用程式]。

   ![「建立 Azure 資源函式應用程式」螢幕擷取畫面](./media/automate-custom-reports/function-app-01.png)

2. 為您的應用程式輸入適當的資訊，然後選取 [建立]。 (只有當您想要以 Application Insights 監視新的「函數應用程式」時，才必須將 Application Insights 切換至 [開啟])

   ![「建立 Azure 資源函式應用程式」的「設定」螢幕擷取畫面](./media/automate-custom-reports/function-app-02.png)

3. 新「函數應用程式」完成部署之後，選取 [前往資源]。

4. 選取 [新增函式]。

   ![「建立新函式」螢幕擷取畫面](./media/automate-custom-reports/function-app-03.png)

5. 選取 [Application Insights 排程摘要範本]。

   ![「新增函式 Application Insights 範本」螢幕擷取畫面](./media/automate-custom-reports/function-app-04.png)

6. 為您的報表輸入適當的收件者電子郵件地址，然後選取 [建立]。

   ![「函式設定」螢幕擷取畫面](./media/automate-custom-reports/function-app-05.png)

7. 選取您的「函數應用程式」 > [平台功能] > [應用程式設定]。

    ![「Azure 函數應用程式」設定螢幕擷取畫面](./media/automate-custom-reports/function-app-07.png)

8. 建立三個具有適當對應值 ``AI_APP_ID``、``AI_APP_KEY`` 及 ``SendGridAPI`` 的新應用程式設定。 選取 [ **儲存**]。

     ![函式整合介面螢幕擷取畫面](./media/automate-custom-reports/function-app-08.png)
    
    (針對要提出報告的「Application Insights 資源」，可以在 [API 存取權] 底下找到 AI_ 值。 如果您沒有「Application Insights API 金鑰」，可以選擇 [建立 API 金鑰])。
    
    * AI_APP_ID = 應用程式識別碼
    * AI_APP_KEY = API 金鑰
    * SendGridAPI = SendGrid API 金鑰

    > [!NOTE]
    > 如果您沒有 SendGrid 帳戶，便可以加以建立。 如需適用於 Azure Functions 的 SendGrid 文件，請參閱[這裡](https://docs.microsoft.com/azure/azure-functions/functions-bindings-sendgrid)。 如果您只需有關如何設定 SendGrid 及產生 API 金鑰的最基本說明，可在本文結尾找到該說明。 

9. 選取 [整合]，然後在 [輸出] 底下，按一下 [SendGrid ($return)]。

     ![「輸出」螢幕擷取畫面](./media/automate-custom-reports/function-app-09.png)

10. 在 [SendGridAPI 金鑰應用程式設定] 底下，針對 [SendGridAPI] 選取新建立的「應用程式設定」。

     ![「執行函數應用程式」螢幕擷取畫面](./media/automate-custom-reports/function-app-010.png)

11. 執行並測試您的「函數應用程式」。

     ![「測試」螢幕擷取畫面](./media/automate-custom-reports/function-app-11.png)

12. 檢查您的電子郵件，以確認是否已成功傳送/接收訊息。

     ![電子郵件主旨列螢幕擷取畫面](./media/automate-custom-reports/function-app-12.png)

## <a name="sendgrid-with-azure"></a>與 Azure 搭配運作的 SendGrid

只有當您尚未設定 SendGrid 帳戶時，才適用這些步驟。

1. 從 Azure 入口網站中選取 [建立資源]，搜尋 **SendGrid Email Delivery** > 按一下 [建立] > 填寫 SendGrid 特定的建立指示。 

     ![建立 SendGrid 資源螢幕擷取畫面](./media/automate-custom-reports/function-app-13.png)

2. 在 [SendGrid 帳戶] 底下建立之後，選取 [管理]。

     ![[Settings] \(設定\) 下的 [API Keys] \(API 金鑰\) 螢幕擷取畫面](./media/automate-custom-reports/function-app-14.png)

3. 這會啟動 SendGrid 的網站。 選取 [Settings] \(設定\) > [API Keys] \(API 金鑰\)。

     ![建立並檢視 API 金鑰應用程式螢幕擷取畫面](./media/automate-custom-reports/function-app-15.png)

4. 建立 API 金鑰 > 選擇 [Create & View] \(建立並檢視\) (請檢閱 SendGrid 有關受限制存取權的文件，以判斷適合您 API 金鑰的權限層級。 這裡選取 [Full Access] \(完整存取權\) 只是為了舉例)。

   ![完整存取權螢幕擷取畫面](./media/automate-custom-reports/function-app-16.png)

5. 複製整個金鑰，您將需要此值來作為「函數應用程式」設定中 SendGridAPI 的值

   ![複製 API 金鑰螢幕擷取畫面](./media/automate-custom-reports/function-app-17.png)

## <a name="next-steps"></a>後續步驟

* 深入了解建立 [Analytics 查詢](app-insights-analytics-using.md)。
* 深入了解[以程式設計方式查詢 Application Insights 資料](https://dev.applicationinsights.io/)
* 深入了解 [Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-what-are-logic-apps)。
* 深入了解 [Microsoft Flow](https://ms.flow.microsoft.com)。
