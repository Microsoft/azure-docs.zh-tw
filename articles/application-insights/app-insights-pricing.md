---
title: 管理 Azure Application Insights 的價格和資料量 | Microsoft Docs
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
ms.reviewer: Dale.Koetke
ms.date: 08/11/2018
ms.author: mbullwin
ms.openlocfilehash: 221d0d8a78a46cad9f1994e9313b21e2867d9563
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42145539"
---
# <a name="manage-pricing-and-data-volume-in-application-insights"></a>管理 Application Insights 的價格和資料量

> [!NOTE]
> 本文說明如何在 Application Insights 中分析資料使用量。  請參閱下列文章以了解相關資訊。
> - [監視使用量和估計成本](../monitoring-and-diagnostics/monitoring-usage-and-estimated-costs.md)說明如何針對不同的定價模型，檢視多項 Azure 監視功能的使用量和估計成本。 其中也會說明如何變更定價模型。

[Azure Application Insights][start] 的價格是根據每個應用程式的資料量而定。 每項 Application Insights 資源都是個別計費的服務，並且會計入到您的 Azure 訂用帳戶帳單。

Application Insights 有兩種定價方案：「基本」和「企業」。 「基本」定價方案是預設方案。 它包含所有「企業」方案功能，且無須額外付費。 「基本」方案主要是針對內嵌的資料磁碟區計費。 

「企業」方案會針對每個節點計費，而每個節點會收到每日資料額度。 在「企業」定價方案中，當擷取的資料超過內含的額度時，便會向您收取費用。 如果您使用 Operations Management Suite，即應該選擇「企業」方案。 

如果您有關於 Application Insights 定價運作方式的疑問，請在我們的[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=ApplicationInsights)中張貼問題。

## <a name="pricing-plans"></a>定價方案

如需您幣別與區域的目前價格，請參閱 [Application Insights 價格][pricing]。

> [!NOTE]
> 我們在 2018 年 4 月[推出](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/)新的 Azure 監視定價模型。 這個模型在完整的監視服務組合之間採用簡單的「隨用隨付」模型。 深入了解[新的定價模型](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs)、如何根據您的使用模式[評估移到這個模型的影響](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs#assessing-the-impact-of-the-new-pricing-model)，以及[如何加入新的模型](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-usage-and-estimated-costs#moving-to-the-new-pricing-model)。

### <a name="basic-plan"></a>基本方案

「基本」方案是建立新 Application Insights 資源時的預設方案。 「基本」方案是適合所有客戶的最佳方案，但已有 Operations Management Suite 訂用帳戶的客戶除外。

* 在「基本」方案中，會依資料量向您收取費用。 資料量係指 Application Insights 所接收的遙測位元組數。 資料量是以 Application Insights 從應用程式接收的未壓縮 JSON 資料套件大小來測量。 針對[匯入至分析的表格式資料](https://docs.microsoft.com/azure/application-insights/app-insights-analytics-import)，資料量是以傳送至 Application Insights 的檔案解壓縮大小來測量。
* 從 2018 年 4 月之後，您的應用程式資料磁碟區費用現在會在名為**資料擷取**的新計費計量上報告。 新的計量可在監視技術之間共用，例如 Applications Insights 和 Log Analytics，且目前位於服務名稱**應用程式服務**底下 (即將變更至 **Log Analytics**)。 
* [即時計量串流](app-insights-live-stream.md)資料不會計入價格用途。
* 自 2018 年 4 月起，在「基本」方案中無須額外付費，即可使用[連續匯出](app-insights-export-telemetry.md)和 [Azure Log Analytics 連接器](https://go.microsoft.com/fwlink/?LinkId=833039&amp;clcid=0x409)。

### <a name="enterprise-plan-and-operations-management-suite-subscription-entitlements"></a>企業方案和 Operations Management Suite 訂用帳戶的權利

如[先前所宣布](https://blogs.technet.microsoft.com/msoms/2017/05/19/azure-application-insights-enterprise-as-part-of-operations-management-suite-subscription/)，購買 Operations Management Suite E1 和 E2 的客戶可以取得 Application Insights Enterprise 作為額外的元件，無須額外付費。 具體來說，Operations Management Suite E1 和 E2 的每個單位皆包含一個 Application Insights Enterprise 方案節點的權利。 每個 Application Insights 節點包含每天最多 200 MB 擷取的資料 (與 Log Analytics 資料擷取分開計算)，資料可保留 90 天而無須額外付費。 本文稍後會更詳細說明此方案。 

由於此方案僅適用於具有 Operations Management Suite 訂用帳戶的客戶，因此沒有 Operations Management Suite 訂用帳戶的客戶不會看到可選取此方案的選項。

> [!NOTE]
> 為了確保您取得此權利，您的 Application Insights 資源必須在「企業」定價方案中。 此權利只能以節點為單位套用。 「基本」方案中的 Application Insights 資源無法實現任何效益。 在 [使用量和估計成本] 窗格中所顯示的估計成本中看不到此權利。 此外，如果您將訂用帳戶移轉至 2018 年 4 月的新 Azure 監視定價模型，「基本」方案將會是唯一可用的方案。 如果您擁有 Operations Management Suite 訂用帳戶，建議您不要將訂用帳戶移轉至新的 Azure 監視定價模型。

如需有關「企業」方案的詳細資訊，請參閱[企業定價詳細資料](app-insights-pricing-enterprise-details.md)。

### <a name="multi-step-web-tests"></a>多重步驟 Web 測試

[多重步驟 Web 測試](app-insights-monitor-web-app-availability.md#multi-step-web-tests)會產生額外費用。 多重步驟 Web 測試係指執行一系列動作的 Web 測試。

單一頁面的「Ping 測試」不另外收費。 針對來自 Ping 測試和多重步驟測試的遙測，收費方式與來自您應用程式的其他遙測一樣。

## <a name="review-pricing-plans-and-estimate-costs"></a>檢閱定價方案和估計成本

Application Insights 可讓您輕鬆了解可用的定價方案，以及根據最近的使用模式可能產生的成本。 若要開始，請在 Azure 入口網站中，針對 Application Insights 資源，移至 [使用量和估計成本] 窗格：

![選擇價格](./media/app-insights-pricing/pricing-001.png)

A. 檢閱當月的資料量。 這包括從您的伺服器和用戶端應用程式，以及從可用性測試接收並保留的所有資料 (在任何[取樣](app-insights-sampling.md)之後)。  
B. 進行[多步驟 Web 測試](app-insights-monitor-web-app-availability.md#multi-step-web-tests)需另外收費。 (這不包括簡單的可用性測試，其已包含在資料量費用中。)  
C. 檢視上個月的資料量趨勢。  
D. 啟用資料擷取[取樣](app-insights-sampling.md)。   
E. 設定每日資料量上限。  

Application Insights 費用會加到您的 Azure 帳單中。 您可以在 Azure 入口網站的 [帳務] 區段中，或在 [Azure 計費入口網站](https://account.windowsazure.com/Subscriptions)中，查看您的 Azure 帳單詳細資料。 

![在左側功能表中，選取 [帳務]](./media/app-insights-pricing/02-billing.png)

## <a name="data-rate"></a>資料速率
您可以透過三種方式限制傳送的資料量：

* **取樣**：您可以使用取樣來減少從伺服器和用戶端應用程式傳送的遙測量，這對計量的扭曲程度最小。 取樣是您可用來調整所傳送資料量的主要工具。 深入了解[取樣功能](app-insights-sampling.md)。 
* **每日上限**：在 Azure 入口網站中建立 Application Insights 資源時，每日上限會設定為每日 100 GB。 在 Visual Studio 中建立 Application Insights 資源時，預設值很小 (只有每日 32.3 MB)。 每日上限預設值的設定是為了協助測試。 這是預期使用者在將應用程式部署至生產環境之前，會提高每日上限。 

    除非您針對高流量的應用程式要求更高的最大值，否則每日的最高上限是 1,000 GB。 

    設定每日上限時，請務必小心。 您的目的應該是「一律不要達到每日上限」。 如果達到每日上限，就會失去當天其餘時間的資料，而無法監視應用程式。 若要變更每日上限，請使用 [每日用量上限] 選項。 您可以在 [使用量和估計成本] 窗格中存取此選項 (本文稍後會有更詳細的說明)。
    我們已移除某些訂用帳戶類型的信用額度無法用於 Application Insights 的限制。 先前，如果訂用帳戶有消費限制，則 [每日上限] 對話方塊中會有指示，說明如何移除消費限制並讓每日上限提高到每日 32.3 MB 以上。
* **節流**：節流會將資料速率限制在每秒 32,000 個事件 (每個檢測金鑰以 1 分鐘計算平均值)。

如果應用程式超過節流速率會發生什麼事？

* 系統會每分鐘評估應用程式傳送的資料量。 如果每秒速率超過每分鐘平均值，伺服器會拒絕部分要求。 SDK 會緩衝處理資料，然後嘗試重新傳送它。 它會在數分鐘的期間內散佈大量資料。 如果應用程式始終以高於節流速率的方式傳送資料，有些資料會就遭到捨棄。 (ASP.NET、Java 和 JavaScript SDK 會嘗試以此方式重新傳送資料，其他 SDK 則可能直接捨棄節流的資料)。當節流發生時，會有通知警告來警示您已發生這種情況。

如何知道應用程式正在傳送多少資料？

您可以使用下列其中一個選項來查看應用程式傳誦多紹資料：

* 開啟 [使用量和估計成本] 窗格以查看每日資料量圖表。 
* 在 [計量瀏覽器] 中，新增圖表。 針對圖表計量，選取 [資料點量]。 開啟 [群組]，然後依 [資料類型] 分組。

## <a name="reduce-your-data-rate"></a>降低資料速率
以下是您可進行以減少資料量的一些事項︰

* 使用 [取樣](app-insights-sampling.md)。 此技術可在不扭曲計量的情況下降低資料速率。 您不會失去在 [搜尋] 中的相關項目之間瀏覽的能力。 在伺服器應用程式中，取樣會自動運作。
* [限制可回報的 Ajax 呼叫次數](app-insights-javascript.md#detailed-configuration) (在每個頁面檢視中) 或關閉 Ajax 報告功能。
* [編輯 ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md)以關閉您不需要的集合模組。 例如，您可能會決定效能計數器或相依性資料是不必要的。
* 將遙測資料分攤到各個檢測金鑰。 
* 預先彙總度量。 如果您在應用程式中呼叫 TrackMetric，便可以使用接受一批測量之平均及標準差計算的多載來減少流量。 或者，您也可以使用[預先彙總套件](https://www.myget.org/gallery/applicationinsights-sdk-labs)。

## <a name="manage-the-maximum-daily-data-volume"></a>管理每日資料量上限

您可以使用每日用量上限來限制所收集的資料。 不過，如果達到上限，就會失去當天其餘時間從您應用程式傳送的所有遙測資料。 建議您「不要」讓應用程式達到每日上限。 一旦應用程式達到每日上限，您便無法追蹤應用程式的健康情況和效能。

請不要使用每日用量上限，而是改用[取樣](app-insights-sampling.md)將資料量調整到您想要的程度。 然後，只使用每日上限作為應付萬一應用程式開始未預期地傳送大量遙測時的「最後手段」。

若要變更每日上限，請在 Application Insights 資源的 [設定] 區段中，從 [使用量和估計成本] 窗格選取 [每日上限]。

![調整每日遙測資料量上限](./media/app-insights-pricing/pricing-003.png)

## <a name="sampling"></a>取樣
[取樣](app-insights-sampling.md)是一種方法，可降低將遙測傳送到您應用程式時的速率，同時仍保留在診斷搜尋期間尋找相關事件的功能。 此外，也保留正確的事件計數。

取樣可有效減少費用並維持在每月配額內。 取樣演算法會保留相關的遙測項目，因此，例如在使用 [搜尋] 時，您便可以找到與特定例外狀況相關的要求。 此演算法也會保留正確的計數，因此您在 [計量瀏覽器] 中會看到要求率、例外狀況率及其他計數的正確值。

取樣有數種形式。

* [調適型取樣](app-insights-sampling.md)是 ASP.NET SDK 的預設值。 調適型取樣會自動針對應用程式所傳送的遙測量進行調整。 這會自動在 Web 應用程式的 SDK 中運作，以便降低網路上的遙測流量。 
*  是替代方法，它會在遙測從應用程式進入 Application Insights 服務時運作。 擷取取樣不會影響從應用程式傳送的遙測量，但可減少服務所保留的量。 您可以使用擷取取樣來減少被來自瀏覽器及其他 SDK 的遙測用光的配額。

若要設定擷取取樣，請移至 [定價] 窗格：

![在 [配額和定價] 窗格中，選取 [範例] 圖格，然後選取一個取樣分數](./media/app-insights-pricing/pricing-004.png)

> [!WARNING]
> [資料取樣] 窗格只會控制擷取取樣的值。 它不會反映應用程式中 Application Insights SDK 所套用的取樣率。 如果已在 SDK 中對連入遙測進行取樣，就不會套用擷取取樣。
>

若要探索實際的取樣率，而不論其套用位置是哪裡，請使用[分析查詢](app-insights-analytics.md)。 此查詢看起來像這樣：

    requests | where timestamp > ago(1d)
    | summarize 100/avg(itemCount) by bin(timestamp, 1h)
    | render areachart

在每個保留的記錄中，`itemCount` 會指出它所代表的原始記錄數目。 它等於 1 + 先前捨棄的記錄數目。 

## <a name="automation"></a>自動化

您可以使用 Azure Resource Management 來撰寫指令碼以設定價格方案。 [了解作法](app-insights-powershell.md#price)。

## <a name="limits-summary"></a>限制摘要

[!INCLUDE [application-insights-limits](../../includes/application-insights-limits.md)]

## <a name="next-steps"></a>後續步驟

* [取樣](app-insights-sampling.md)

[api]: app-insights-api-custom-events-metrics.md
[apiproperties]: app-insights-api-custom-events-metrics.md#properties
[start]: app-insights-overview.md
[pricing]: http://azure.microsoft.com/pricing/details/application-insights/