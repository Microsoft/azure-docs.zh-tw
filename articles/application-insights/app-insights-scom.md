---
title: SCOM 與 Application Insights 的整合 | Microsoft Docs
description: 如果您是 SCOM 使用者，請使用 Application Insights 來監視效能和診斷問題。 完整的儀表板、智慧警示、功能強大的診斷工具和分析查詢。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 606e9d03-c0e6-4a77-80e8-61b75efacde0
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: mbullwin
ms.openlocfilehash: 8880fbeaad85bc2615292820527c6a9e87000d66
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40246418"
---
# <a name="application-performance-monitoring-using-application-insights-for-scom"></a>使用 Application Insights 的 SCOM 應用程式效能監視
如果您使用 System Center Operations Manager (SCOM) 來管理您的伺服器，您便可以透過 [Azure Application Insights](app-insights-asp-net.md) 的協助來監視效能及診斷效能問題。 Application Insights 會監視 Web 應用程式的連入要求、REST 和 SQL 傳出呼叫、例外狀況，以及記錄追蹤。 它提供的儀表板具有度量圖表和智慧警示，以及透過此遙測來實現的強大診斷搜尋和分析查詢功能。 

您可以使用 SCOM 管理組件來開啟 Application Insights 監視。

> [!IMPORTANT]
> 這個 System Center Operations Manager 管理組件現在**已被取代**。 它不支援最新的 Application Insights SDK，且不再建議使用。

## <a name="before-you-start"></a>開始之前
我們假設︰

* 您熟悉 SCOM，而且您使用 SCOM 2012 R2 或 2016 來管理您的 IIS Web 伺服器。
* 您已在伺服器上安裝想要使用 Application Insights 監視的 Web 應用程式。
* 應用程式架構版本是 .NET 4.5 或更新版本。
* 您可以存取 [Microsoft Azure](https://azure.com) 中的訂用帳戶，而且可以登入 [Azure 入口網站](https://portal.azure.com)。 您的組織可能擁有訂用帳戶，並且可以在其中新增您的 Microsoft 帳戶。

(開發小組可以將 [Application Insights SDK](app-insights-asp-net.md) 建置到 Web 應用程式中。 此建置階段檢測可讓他們獲得更大的彈性來撰寫自訂遙測。 不過，這並不重要︰您可以依照此處所述的步驟來進行，不論有沒有內建 SDK 都可以。)

## <a name="one-time-install-application-insights-management-pack"></a>(一次) 安裝 Application Insights 管理組件
在執行 Operations Manager 的電腦上：

1. 解除安裝任何舊版管理組件︰
   1. 在 Operations Manager 中，開啟 [系統管理] > [管理組件]。 
   2. 刪除舊版本。
2. 從目錄下載並安裝管理組件。
3. 重新啟動 Operations Manager。

## <a name="create-a-management-pack"></a>建立管理組件
1. 在 Operations Manager 中，開啟 [撰寫中] > [.NET...(使用 Application Insights)] > [加入監視精靈]，然後再次選擇 [.NET...(使用 Application Insights)]。
   
    ![](./media/app-insights-scom/020.png)
2. 以您的應用程式命名組態  (您必須一次檢測一個應用程式)。
   
    ![](./media/app-insights-scom/030.png)
3. 在相同的精靈頁面上，建立新的管理組件或選取您稍早為 Application Insights 建立的組件。
   
     (Application Insights [管理組件](https://technet.microsoft.com/library/cc974491.aspx)是可供您用來建立執行個體的範本。 稍後您可以重複使用相同的執行個體。)

    ![在 [一般屬性] 索引標籤中，輸入應用程式的名稱。 按一下 [新增]，然後輸入管理組件的名稱。 按一下 [確定]，然後按 [下一步]。](./media/app-insights-scom/040.png)

1. 選擇一個您想要監視的應用程式。 搜尋功能會搜尋伺服器上所安裝的應用程式。
   
    ![在 [要監視的項目] 索引標籤上按一下 [新增]、輸入部分應用程式名稱、按一下 [搜尋]、選擇應用程式，然後按一下 [新增] 和 [確定]。](./media/app-insights-scom/050.png)
   
    如果您不想要在所有伺服器監視應用程式，選用的 [監視範圍] 欄位可用來指定伺服器的子集。
2. 在下一個精靈頁面上，您必須先提供用來登入 Microsoft Azure 的認證。
   
    在這個頁面上，您可以選擇想要在其中分析和顯示遙測資料的 Application Insights 資源。 
   
   * 如果在開發期間已針對 Application Insights 設定了應用程式，請選取其現有資源。
   * 否則，請建立以應用程式命名的新資源。 如果有其他應用程式是同一個系統的元件，請將它們放在相同的資源群組，以便能更輕鬆地管理遙測的存取權。
     
     您稍後可以變更這些設定。
     
     ![在 [Application Insights 設定] 索引標籤上按一下 [登入]，並提供適用於 Azure 的 Microsoft 帳戶認證。 然後選擇訂用帳戶、資源群組和資源。](./media/app-insights-scom/060.png)
3. 完成精靈。
   
    ![按一下 [建立專案]。](./media/app-insights-scom/070.png)

針對您想要監視的每個應用程式重複此程序。

如果您稍後需要變更設定，請從 [撰寫] 視窗重新開啟監視屬性。

![在 [撰寫] 中，選取 [使用 Application Insights 的 .NET 應用程式監視效能]、選取監視器，然後按一下 [屬性]。](./media/app-insights-scom/080.png)

## <a name="verify-monitoring"></a>驗證監視
您已安裝的監視器會在每一部伺服器上搜尋您的應用程式。 它會在找到應用程式的地方設定 Application Insights 狀態監視器來監視應用程式。 如有必要，它會先在伺服器上安裝狀態監視器。

您可以驗證它已找到之應用程式的執行個體︰

![在 [監視] 中開啟 Application Insights](./media/app-insights-scom/100.png)

## <a name="view-telemetry-in-application-insights"></a>在 Application Insights 中檢視遙測
在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至應用程式的資源。 您會[看到顯示應用程式之遙測的圖表](app-insights-dashboards.md)。 (如果它尚未顯示在主頁面上，請按一下 [即時度量串流])。

## <a name="next-steps"></a>後續步驟
* [設定儀表板](app-insights-dashboards.md) 以將監視此應用程式和其他應用程式的最重要圖表結合在一起。
* [深入了解度量](app-insights-metrics-explorer.md)
* [設定警示](app-insights-alerts.md)
* [診斷效能問題](app-insights-detect-triage-diagnose.md)
* [功能強大的分析查詢](app-insights-analytics.md)
* [可用性 Web 測試](app-insights-monitor-web-app-availability.md)

