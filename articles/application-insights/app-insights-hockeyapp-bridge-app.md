---
title: 在 Azure Application Insights 中探索 HockeyApp 資料 | Microsoft Docs
description: 使用 Application Insights 分析 Azure 應用程式的使用量和效能。
services: application-insights
documentationcenter: windows
author: mrbullwinkle
manager: carmonm
ms.assetid: 97783cc6-67d6-465f-9926-cb9821f4176e
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/30/2017
ms.author: mbullwin
ms.openlocfilehash: 7586dbc4d7a0b7dbc7756eabbb4a8d5e0e60a731
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35632861"
---
# <a name="exploring-hockeyapp-data-in-application-insights"></a>在 Application Insights 中探索 HockeyApp 資料

> [!NOTE]
> HockeyApp 不再適用於新的應用程式。 現有的 HockeyApp 部署將會繼續運作。 Visual Studio App Center 現在是 Microsoft 中建議用來監視最新行動應用程式的服務。 [了解如何使用 App Center 和 Application Insights 設定您的應用程式](app-insights-mobile-center-quickstart.md)。

[HockeyApp](https://azure.microsoft.com/services/hockeyapp/) 是監視即時桌面和行動應用程式的服務。 您可以從 HockeyApp 傳送自訂和追蹤遙測，以便監視使用情況和協助診斷 (除了取得損毀資料以外)。 使用 [Azure Application Insights](app-insights-overview.md) 的強大[分析](app-insights-analytics.md)功能，即可查詢此遙測資料流。 此外，您可以 [匯出自訂和追蹤遙測](app-insights-export-telemetry.md)。 若要啟用這些功能，您可設定橋接器，以將 HockeyApp 自訂資料轉送至 Application Insights。

## <a name="the-hockeyapp-bridge-app"></a>HockeyApp 橋接器應用程式
HockeyApp 橋接器應用程式是一項核心功能，可讓您透過分析和連續匯出功能來存取 Application Insights 中的 HockeyApp 自訂和追蹤遙測。 經由上述這些功能，可以存取 HockeyApp 在 HockeyApp 橋接器應用程式建立後所收集的自訂和追蹤事件。 我們一起看看如何設定其中一個橋接器應用程式。

在 HockeyApp 中，開啟 [帳戶設定]、 [API 權杖](https://rink.hockeyapp.net/manage/auth_tokens)。 建立新的權杖，或重複使用現有的權杖。 所需的最小權限是「唯讀」。 取得 API 權杖的複本。

![取得 HockeyApp API 權杖](./media/app-insights-hockeyapp-bridge-app/01.png)

開啟 Microsoft Azure 入口網站並 [建立 Application Insights 資源](app-insights-create-new-resource.md)。 將應用程式類型設定為「HockeyApp 橋接器應用程式」：

![新增 Application Insights 資源](./media/app-insights-hockeyapp-bridge-app/02.png)

您不需要設定名稱 - 將會自動從 HockeyApp 名稱設定。

HockeyApp 橋接器欄位隨即出現。 

![輸入橋接器欄位](./media/app-insights-hockeyapp-bridge-app/03.png)

輸入您先前記下的 HockeyApp 權杖。 此動作會以您所有的 HockeyApp 應用程式填入 [HockeyApp 應用程式] 下拉式功能表。 選取您想要使用的應用程式，並完成其餘的欄位。 

開啟新的資源。 

請注意，資料需要一些時間才會開始流動。

![等候資料的 Application Insights 資源](./media/app-insights-hockeyapp-bridge-app/04.png)

這樣就大功告成了！ 從現在開始在 HockeyApp 檢測的應用程式中收集的自訂和追蹤資料，也可供您在 Application Insights 的分析和連續匯出功能中使用。

讓我們簡短地回顧一下您現在可以使用的功能。

## <a name="analytics"></a>分析
「分析」是強大的資料特定查詢工具，可讓您診斷和分析您的遙測，並快速找出根本原因和模式。

![分析](./media/app-insights-hockeyapp-bridge-app/05.png)

* [深入了解分析](app-insights-analytics-tour.md)

## <a name="continuous-export"></a>連續匯出
連續匯出可讓您將資料匯出至 Azure Blob 儲存體容器。 如果您的資料所需的保留時間超過 Application Insights 目前提供的保留期間，這就非常有用。 您可以將資料保留在 Blob 儲存體中，將它處理到 SQL Database 或您慣用的資料倉儲解決方案中。

[深入了解連續匯出](app-insights-export-telemetry.md)

## <a name="next-steps"></a>後續步驟
* [將分析套用到資料](app-insights-analytics-tour.md)

