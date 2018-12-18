---
title: 傳送使用者內容識別碼以啟用 Azure Application Insights 中的使用體驗 | Microsoft Docs
description: 追蹤使用者如何在您的服務移動，方法是為每個使用者指派 Application Insights 中唯一的持續性識別碼字串。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/02/2017
ms.reviewer: abgreg
ms.author: mbullwin
ms.openlocfilehash: 14322162d3f78f0cb90ecaf077d1d85f7cbba581
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47091527"
---
#  <a name="send-user-context-ids-to-enable-usage-experiences-in-azure-application-insights"></a>傳送使用者內容識別碼以啟用 Azure Application Insights 中的使用體驗

## <a name="tracking-users"></a>追蹤使用者

Application Insights 可讓您透過一組產品使用量工具來監控並追蹤使用者： 
* [使用者、工作階段、事件](https://docs.microsoft.com/azure/application-insights/app-insights-usage-segmentation)
* [漏斗圖](https://docs.microsoft.com/azure/application-insights/usage-funnels)
* [保留](https://docs.microsoft.com/azure/application-insights/app-insights-usage-retention)
* 同群使用者
* [活頁簿](https://docs.microsoft.com/azure/application-insights/app-insights-usage-workbooks)

若要追蹤使用者在一段時間內所做的行為，Application Insights 需要每個使用者或工作階段的識別碼。 包括下列每個自訂事件或頁面檢視畫面中的識別碼。
- 使用者、漏斗圖、保留期和同群使用者：包含使用者識別碼。
- 工作階段：包含工作階段識別碼。

如果您的應用程式與 [JavaScript SDK](https://docs.microsoft.com/azure/application-insights/app-insights-javascript#set-up-application-insights-for-your-web-page) 整合，則會自動追蹤使用者識別碼。

## <a name="choosing-user-ids"></a>選擇使用者識別碼

使用者識別碼應該在使用者工作階段期間持續存在，以追蹤使用者在一段時間內的行為。 有多種方法可持續使用識別碼。
- 您的服務中已有使用者定義。
- 如果服務可存取瀏覽器，該服務可利用識別碼將 cookie 傳遞給瀏覽器。 只要 cookie 保留在使用者的瀏覽器中，識別碼就會持續存在。
- 如有必要，您可以每個工作階段都使用新的識別碼，但與使用者相關的結果會受到限制。 例如，您將無法看到使用者的行為如何隨著時間而變化。

識別碼應該是 Guid 或足夠複雜的另一個字串，以專門識別每個使用者。 例如，它可能是一個長的隨機數字。

如果識別碼包含使用者的個人識別資訊，則該值不適合傳送至 Application Insights 做為使用者識別碼。 您可以傳送此類識別碼做為[已驗證的使用者識別碼](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#authenticated-users)，但不符合使用案例的使用者識別碼需求。

## <a name="aspnet-apps-setting-the-user-context-in-an-itelemetryinitializer"></a>ASP.NET 應用程式：在 ITelemetryInitializer 中設定使用者內容

建立遙測初始設定式 (依照[這裡](https://docs.microsoft.com/azure/application-insights/app-insights-api-filtering-sampling#add-properties-itelemetryinitializer)的詳細說明)，並設定 Context.User.Id 和 Context.Session.Id。

此範例會將使用者識別碼設定為在工作階段之後到期的識別碼。 如果可能，請使用工作階段期間持續存在的使用者識別碼。

```csharp

    using System;
    using System.Web;
    using Microsoft.ApplicationInsights.Channel;
    using Microsoft.ApplicationInsights.Extensibility;

    namespace MvcWebRole.Telemetry
    {
      /*
       * Custom TelemetryInitializer that sets the user ID.
       *
       */
      public class MyTelemetryInitializer : ITelemetryInitializer
      {
        public void Initialize(ITelemetry telemetry)
        {
            // For a full experience, track each user across sessions. For an incomplete view of user 
            // behavior within a session, store user ID on the HttpContext Session.
            // Set the user ID if we haven't done so yet.
            if (HttpContext.Current.Session["UserId"] == null)
            {
                HttpContext.Current.Session["UserId"] = Guid.NewGuid();
            }

            // Set the user id on the Application Insights telemetry item.
            telemetry.Context.User.Id = (string)HttpContext.Current.Session["UserId"];

            // Set the session id on the Application Insights telemetry item.
            telemetry.Context.Session.Id = HttpContext.Current.Session.SessionID;
        }
      }
    }
```

## <a name="next-steps"></a>後續步驟
- 若要啟用使用體驗，請開始傳送[「自訂事件」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) 或 [「頁面檢視」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views)。
- 如果您已傳送自訂事件或頁面檢視，請探索「使用量工具」，以了解使用者如何使用您的服務。
    * [使用量概觀](app-insights-usage-overview.md)
    * [使用者、工作階段和事件](app-insights-usage-segmentation.md)
    * [漏斗圖](usage-funnels.md)
    * [保留](app-insights-usage-retention.md)
    * [活頁簿](app-insights-usage-workbooks.md)
