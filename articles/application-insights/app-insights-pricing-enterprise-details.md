---
title: 適用於 Azure Application Insights 的舊版企業定價方案詳細資料 | Microsoft Docs
description: 在 Application Insights 中管理遙測量和監視成本。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: ebd0d843-4780-4ff3-bc68-932aa44185f6
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/21/2018
ms.author: mbullwin
ms.openlocfilehash: 65307eab0bf1b5f502f11c14c369826cd12e0966
ms.sourcegitcommit: 638599eb548e41f341c54e14b29480ab02655db1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36311082"
---
# <a name="enterprise-plan-details"></a>企業方案詳細資料

Azure Application Insights 有兩種定價方案：「基本」和「企業」。 [基本](app-insights-pricing.md)定價方案是預設方案。 它包含所有企業方案功能，不需要額外費用。 「基本」方案主要是針對內嵌的資料磁碟區計費。 

「企業」方案會針對每個節點計費，而每個節點會收到每日資料額度。 在「企業」定價方案中，您要對超過內含資料額度的內嵌資料付費。 如果您使用 Operations Management Suite，您應該選擇「企業」方案。 

如需屬於您貨幣與區域的目前定價，請參閱 [Application Insights 定價](http://azure.microsoft.com/pricing/details/application-insights/)。

> [!NOTE]
> 我們在 2018 年 4 月[推出](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/)新的 Azure 監視定價模型。 這個模型在完整的監視服務組合之間採用簡單的「隨用隨付」模型。 深入了解[新的定價模型](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs)、如何根據您的使用模式[評估移到這個模型的影響](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs#assessing-the-impact-of-the-new-pricing-model)，以及[如何加入新的模型](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs#moving-to-the-new-pricing-model)

## <a name="how-the-enterprise-plan-works"></a>「企業」方案的運作方式

* 在「企業」方案中，是依照傳送所有應用程式遙測的每一節點付費。
 * 「節點」是裝載應用程式的實體或虛擬伺服器機器，或是平台即服務 (PaaS) 角色執行個體。
 * 開發用機器、用戶端瀏覽器及行動裝置不算節點。
 * 如果您的應用程式有數個會傳送遙測的元件，例如 Web 服務和後端背景工作角色，則會分開計算元件。
 * [即時計量串流](app-insights-live-stream.md)資料不會計入價格用途。 訂用帳戶的收費是依照每一節點，而非每一應用程式。 如果您有五個節點傳送 12 個應用程式的遙測，則是以五個節點計算收費。
* 雖然費用是按月報價，您的收費僅適用於一個節點從一個應用程式傳送遙測的任何小時。 每小時的費用是月費報價除以 744 (一個月 31 天的小時數)。
* 每個節點 (資料粒度為小時) 偵測每日 200 MB 的資料量配置。 未使用的資料配置不會累計到隔天。
 * 如果您選擇「企業」定價方案，每個訂用帳戶會根據傳送遙測資料至該訂用帳戶中 Application Insights 資源的節點數，得到每日資料額度。 因此，如果您有 5 個節點全天候傳送資料，您會有合起來 1GB 的資料額度，套用至該訂用帳戶中的所有 Application Insights 資源。 由於內含資料會在所有節點之間共用，因此特定節點所傳送的資料是否超過其他節點並無妨。 如果某一天，Application Insights 資源收到超過此訂用帳戶的每日資料配置，則會按 GB 收取資料超量費用。 
 * 每日資料額度的計算方式，是每個節點在當天傳送遙測資料的時數 (使用 UTC) 除以 24 再乘以 200 MB。 因此，如果您有四個節點在一天 24 個小時的 15 個小時內傳送遙測資料，則當天內含資料量會是 ((4 &#215; 15) / 24) &#215; 200 MB = 500 MB。 以資料超量每一 GB 的價格是 2.30 美元來說，如果節點那一天會傳送 1 GB 的資料，費用就是 1.15 美元。
 * 「企業」方案的每日額度並不與已選擇為「基本」方案的應用程式共用。 未使用額度不會逐日累計。 

## <a name="examples-of-how-to-determine-distinct-node-count"></a>如何判斷相異節點計數的範例

| 案例                               | 每日節點總數 |
|:---------------------------------------|:----------------:|
| 1 個應用程式使用 3 個 Azure App Service 執行個體和 1 個虛擬伺服器 | 4 |
| 3 個應用程式在 2 個虛擬機器上執行，而這些應用程式的 Application Insights 資源位於相同的訂用帳戶中，且採用「企業」方案 | 2 | 
| 4 個應用程式的 Application Insights 資源位於相同訂用帳戶中；每個應用程式在離峰的 16 小時期間執行 2 個執行個體，在尖峰的 8 小時期間執行 4 個執行個體 | 13.33 | 
| 雲端服務有 1 個背景工作角色和 1 個 Web 角色，各執行 2 個執行個體 | 4 | 
| 5 個節點的 Azure Service Fabric 叢集執行 50 個微服務，每個微服務執行 3 個執行個體 | 5|

* 節點的精確計算取決於您的應用程式使用哪個 Application Insights SDK。 
  * 在 SDK 2.2 版和更新版本中，Application Insights [Core SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) 和 [Web SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) 都會將每個應用程式主機報告為節點。 範例包括實體伺服器和虛擬機器主機的電腦名稱，或雲端服務的執行個體名稱。  唯一的例外狀況是僅使用 [.NET Core](https://dotnet.github.io/) 和 Application Insights Core SDK 的應用程式。 在此情況下，因為主機名稱無法使用，所以只會針對所有主機報告一個節點。 
  * 對於較早版本的 SDK，[Web SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) 的行為就和較新版的 SDK 一樣，不過不論應用程式主機的數目是多少，[Core SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) 都只會報告一個節點。 
  * 如果您的應用程式使用 SDK 來將 **roleInstance** 設為自訂值，依預設將使用那個相同的值來判斷節點計數。 
  * 如果您使用新的 SDK 版本，搭配從用戶端機器或行動裝置執行的應用程式，則節點計數可能會傳回很大的數字 (因為有大量的用戶端機器或行動裝置)。 
