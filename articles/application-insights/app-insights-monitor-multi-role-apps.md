---
title: Azure Application Insights 支援多個元件、微服務和容器 | Microsoft Docs
description: 監視由多個元件或角色組成之應用程式的效能和使用方式。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 09/17/2018
ms.author: mbullwin
ms.openlocfilehash: bf247748415822d5ba1a0e652fdeff384d8e8db1
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982046"
---
# <a name="monitor-multi-component-applications-with-application-insights-preview"></a>使用 Application Insights (預覽) 監視多元件應用程式

您可以使用 [Azure Application Insights](app-insights-overview.md)，監視由多個伺服器元件、角色或服務所組成的應用程式。 元件的健康情況和其間的關聯性會顯示在單一應用程式對應上。 您可以使用自動 HTTP 相互關聯，透過多個元件追蹤個別作業。 容器診斷可以與應用程式遙測整合並相互關聯。 將單一 Application Insights 資源用於您應用程式的所有元件。 

![多元件應用程式對應](./media/app-insights-monitor-multi-role-apps/application-map-001.png)

我們在此使用「元件」來表示大型應用程式之任何運作中的組件。 例如，一般商務應用程式可能是由用戶端程式碼所組成，這些程式碼會在網頁瀏覽器中執行、與一或多個 Web 應用程式服務通訊，進而使用後端服務。 伺服器元件可能裝載於內部部署或雲端中，可能是 Azure 的 Web 和背景工作角色，也可能在 Docker 或 Service Fabric 等容器中執行。 

### <a name="sharing-a-single-application-insights-resource"></a>共用單一 Application Insights 資源 

此處的重要技術是將您應用程式中每個元件的遙測傳送至相同的 Application Insights 資源，但必要時會使用 `cloud_RoleName` 屬性來區別元件。 Application Insights SDK 將 `cloud_RoleName` 屬性新增至遙測元件發出。 例如，SDK 會將網站名稱或服務角色名稱新增至 `cloud_RoleName` 屬性。 您可以使用 telemetryinitializer 覆寫這個值。 應用程式對應會使用 `cloud_RoleName` 屬性以識別地圖上的元件。

如需有關如何覆寫 `cloud_RoleName` 屬性的詳細資訊，請參閱[新增屬性：ITelemetryInitializer](app-insights-api-filtering-sampling.md#add-properties-itelemetryinitializer)。  

在某些情況下，這可能不適當，而且您可能寧願將不同的資源用於不同的元件群組。 例如，您可能需要將不同的資源用於管理或計費目的。

請特別注意，我們假設在本文件的其餘部分，您想要將資料從多個元件傳送至一個 Application Insights 資源。

## <a name="configure-multi-component-applications"></a>設定多元件應用程式

若要取得多元件應用程式對應，您必須達成下列目標：

* 在應用程式的每個元件中**安裝最新的發行前版本** Application Insights 套件。 
* 對您應用程式的所有元件**共用單一 Application Insights 資源**。
* 在 [預覽] 刀鋒視窗中**啟用複合應用程式對應**。

使用適合其類型的方法，設定您應用程式的每個元件 ([ASP.NET](app-insights-asp-net.md)、[Java](app-insights-java-get-started.md)、[Node.js](app-insights-nodejs.md)、[JavaScript](app-insights-javascript.md))。

### <a name="1-install-the-latest-pre-release-package"></a>1.安裝最新的發行前版本套件

在每個伺服器元件的專案中更新或安裝 Appication Insights 套件。 如果您使用 Visual Studio：

1. 以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]。 
2. 選取 [包括發行前版本]。
3. 如果 Application Insights 套件顯示在 [更新] 中，請加以選取。 

    否則，請瀏覽並安裝適當的套件：
    
    * Microsoft.ApplicationInsights.WindowsServer
    * Microsoft.ApplicationInsights.ServiceFabric - 適用於當做客體可執行檔執行的元件和在 Service Fabric 應用程式中執行的 Docker 容器
    * Microsoft.ApplicationInsights.ServiceFabric.Native - 適用於 ServiceFabric 應用程式中的 Reliable Services
    * Microsoft.ApplicationInsights.Kubernetes - 適用於在 Kubernetes 上的 Docker 中執行的元件

### <a name="2-share-a-single-application-insights-resource"></a>2.共用單一 Application Insights 資源

* 在 Visual Studio 中，以滑鼠右鍵按一下專案，然後選取 [設定 Application Insights]，或 [Application Insights] > [設定]。 對於第一個專案，使用精靈來建立 Application Insights 資源。 對於後續專案，選取相同的資源。
* 如果沒有 Application Insights 功能表上，請手動進行設定︰

   1. 在 [Azure 入口網站](https://portal,azure.com)中，開啟您已經為另一個元件建立的 Application Insights 資源。
   2. 在 [概觀] 刀鋒視窗中，開啟 [基本資訊] 下拉式索引標籤，然後複製 [檢測金鑰]。
   3. 在您的專案中，開啟 ApplicationInsights.config 並插入︰`<InstrumentationKey>your copied key</InstrumentationKey>`

![將檢測金鑰複製到 .config 檔案](./media/app-insights-monitor-multi-role-apps/copy-instrumentation-key.png)


### <a name="3-enable-composite-application-map"></a>3.啟用複合應用程式對應

在 Azure 入口網站中，開啟您應用程式的資源。 在「設定」子標題下方，按一下 [預覽] 以開啟 [預覽] 刀鋒視窗。 在 [預覽] 刀鋒視窗中，啟用「複合應用程式對應」。

### <a name="4-enable-docker-metrics-optional"></a>4.啟用 Docker 計量 (選擇性) 

如果元件在 Azure Windows VM 上裝載的 Docker 中執行，您可以從容器收集其他計量。 在您的 [Azure 診斷](../monitoring-and-diagnostics/azure-diagnostics.md)組態檔中插入此程式碼︰

```
"DiagnosticMonitorConfiguration": {
        ...
        "sinks": "applicationInsights",
        "DockerSources": {
                "Stats": {
                    "enabled": true,
                    "sampleRate": "PT1M"
                }
            },
        ...
    }
    ...   
    "SinksConfig": {
        "Sink": [{
            "name": "applicationInsights",
            "ApplicationInsights": "<your instrumentation key here>"
        }]
    }
    ...
}

```

## <a name="use-cloudrolename-to-separate-components"></a>使用 cloud_RoleName 來區分元件

`cloud_RoleName` 屬性會附加至所有遙測。 它會識別源自遙測的元件 (角色或服務)。 (這與 cloud_RoleInstance 不同，分隔在多個伺服器處理序或電腦上平行執行的相同角色)。

在入口網站中，您可以使用此屬性來篩選或區隔您的遙測。 在此範例中，[失敗] 刀鋒視窗會經過篩選，只顯示來自前端 Web 服務的資訊，並篩選出來自 CRM API 後端的失敗︰

![依雲端角色名稱區隔的計量圖表](./media/app-insights-monitor-multi-role-apps/cloud-role-name.png)

## <a name="trace-operations-between-components"></a>追蹤元件之間的作業

您可以追蹤在處理個別作業時元件之間的呼叫。


![顯示作業的遙測](./media/app-insights-monitor-multi-role-apps/show-telemetry-for-operation.png)

跨前端 Web 伺服器和後端 API，逐一點選此作業的相互關聯遙測清單︰

![跨元件搜尋](./media/app-insights-monitor-multi-role-apps/search-across-components.png)


## <a name="next-steps"></a>後續步驟

* [區分開發、測試及生產環境的遙測](app-insights-separate-resources.md)
