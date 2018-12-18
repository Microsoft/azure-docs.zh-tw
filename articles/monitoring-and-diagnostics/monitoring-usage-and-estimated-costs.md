---
title: 在 Azure 監視器中監視使用量和估計成本
description: 使用 Azure 監視器的使用量和估計成本頁面的程序概觀
author: dalekoetke
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 08/11/2018
ms.author: mbullwin
ms.reviewer: Dale.Koetke
ms.component: ''
ms.openlocfilehash: 35e7d36043defd236252f86facf4b9e2ed945d67
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42140062"
---
# <a name="monitoring-usage-and-estimated-costs"></a>監視使用量和估計成本

> [!NOTE]
> 本文說明如何針對不同的定價模型，檢視多項 Azure 監視功能的使用量和估計成本。  請參閱下列文章以了解相關資訊。
> - [藉由控制 Log Analytics 中的資料量與保留期來管理成本](../log-analytics/log-analytics-manage-cost-storage.md)說明如何藉由變更資料保留期限來控制成本。
> - [分析 Log Analytics 中的資料使用量](../log-analytics/log-analytics-usage.md)說明如何分析資料使用量及提出警示。
> - [管理 Application Insights 的價格和資料量](../application-insights/app-insights-pricing.md)說明如何分析 Application Insights 中的資料使用量。

在 Azure 入口網站的監視器中樞中，[使用量和估計成本] 頁面會說明核心監視功能 (例如[警示、計量、通知](https://azure.microsoft.com/pricing/details/monitor/)、[Azure Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) 和 [Azure Application Insights](https://azure.microsoft.com/pricing/details/application-insights/)) 的使用量。 客戶若是採用在 2018 年 4 月前提供的定價方案，則也會提供透過深入解析與分析供應項目購買的 Log Analytics 使用量。

在此頁面上，使用者可以檢視他們在過去 31 天內對這些資源的使用量 (依個別訂用帳戶彙總)。 向下切入功能可顯示這 31 天的使用量趨勢。 系統必須彙整許多資料才能產生此預估值，因此請耐心等候頁面載入。

下列範例顯示監視使用量和估計最終成本的情形：

![使用量和估計成本入口網站螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/001.png)

選取 [每月使用量] 資料行中的連結以開啟一個圖表，顯示過去 31 天的使用量趨勢：

![每個節點包含長條圖的螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/002.png)

以下是另一個類似的使用量和成本摘要。 此範例說明採用 2018 年 4 月 以使用量為基礎的新式定價模式的訂用帳戶。 請留意到，此時並沒有任何以節點為基礎的計費。 Log Analytics 和 Application Insights 的資料擷取和保留都會在新的通用計量上報告。

![使用量和估計成本入口網站螢幕擷取畫面 - 2018 年 4 月定價](./media/monitoring-usage-and-estimated-costs/003.png)

## <a name="new-pricing-model"></a>新的定價模式

我們在 2018 年 4 月[發行了新的監視定價模式](https://azure.microsoft.com/blog/introducing-a-new-way-to-purchase-azure-monitoring-services/)。  此模式非常適合雲端使用，並具有以使用量為基礎的定價功能。 您只需依用量付費，而無需承擔以節點為基礎的承諾。 [警示、計量、通知](https://azure.microsoft.com/pricing/details/monitor/)、[Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) 和 [Application Insights](https://azure.microsoft.com/pricing/details/application-insights/) 都有新定價模式的詳細資料可供參考。 

對於在 2018 年 4 月 2 日之後加入 Log Analytics 或 Application Insights 的客戶而言，新的定價模式將是唯一選項。 已使用這些服務的客戶則可選擇是否移轉至新的定價模式。

## <a name="assessing-the-impact-of-the-new-pricing-model"></a>評估新定價模式的影響
新的定價模式將會根據客戶的監視使用模式，對每個客戶產生不同的影響。 對於在 2018 年 4 月 2 日之前即使用 Log Analytics 或 Application Insights 的客戶，Azure 監視器中的 [使用量和估計成本] 頁面將會估計移轉至新的定價模式時所將產生的成本變更。 此外也提供將訂用帳戶移轉至新模式的途徑。 對大多數客戶而言，新的定價模式將較有利。 但客戶的資料使用量若特別高，或是屬於成本較高的區域，可能就另當別論。

若要對您在 [使用量和估計成本] 頁面上選擇的訂用帳戶查看成本的估計值，請選取頁面頂端附近的藍色橫幅。 建議您一次查看一個訂用帳戶的資料，因為新的定價模式是在此層級上採行的。

![監視新定價模式中的使用量和估計成本的螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/004.png)

新的頁面會顯示與先前的頁面類似、但具有綠色橫幅的內容：

![監視現行定價模式中的使用量和估計成本的螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/005.png)

此頁面也會顯示一組與新的定價模式相對應的不同計量。 以下清單為範例：

- 深入解析與分析\每個節點的超額
- 深入解析與分析\每個節點包含
- Application Insights\基本超額資料
- Application Insights\內含資料

新的定價模式並未以節點為基礎納入資料配置。 因此，這些資料擷取計量會結合為新的通用資料擷取計量，名為**共用服務\資料擷取**。 

擷取至較高成本區域的 Log Analytics 或 Application Insights 中的資料，則有另一項變更。 這些高成本區域的資料將會依新的區域計量顯示。 範例之一是**資料擷取 (美國中西部)**。

> [!NOTE]
> 按訂用帳戶的估計成本不會考量 Operations Management Suite (OMS) 訂用帳戶在帳戶層級上的個別節點權利。 如需深入討論新的定價模式在此情況下的運作方式，請洽詢帳戶代表。

## <a name="new-pricing-model-and-operations-management-suite-subscription-entitlements"></a>新的定價模型和 Operations Management Suite 訂用帳戶的權利

凡購買 Microsoft Operations Management Suite E1 和 E2 的客戶，皆符合資格使用 [Log Analytics](https://www.microsoft.com/cloud-platform/operations-management-suite) 和 [Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-pricing#the-price-plans) 的按節點資料擷取權利。 若要以指定訂用帳戶享用 Log Analytics 工作區或 Application Insights 資源的這些權利： 

- 該訂用帳戶的定價模式必須保持在 2018 年 4 月前的模式。
- Log Analytics 工作區應該使用「每個節點 (OMS)」定價層。
- Application Insights 資源應該使用「企業」定價方案。

根據貴組織所購買套件的節點數目，讓部分訂用帳戶改採新的定價模式仍可能對您有利，但這需要仔細考量。 一般情況下，建議您只要維持如上所述 2018 年 4 月前的模式即可。

> [!WARNING]
> 如果貴組織已購買 Microsoft Operations Management Suite E1 和 E2，訂用帳戶的定價模式通常最好保持在 2018 年 4 月前的模式。 
>

## <a name="changes-when-youre-moving-to-the-new-pricing-model"></a>移轉至新定價模式時的變更

新的定價模式可將 Log Analytics 和 Application Insights 定價選項簡化至只有單一層 (或方案)。 將訂用帳戶移轉至新的定價模式將會：

- 將每個 Log Analytics 的定價層變更成新的「每一 GB」層 (在 Azure Resource Manager 中稱為「pergb2018」)
- 企業方案中的任何 Application Insights 資源會變更為基本方案。

成本估計會反映出這些變更的影響。

> [!WARNING]
> 重要注意事項：如果您使用 Azure Resource Manager 或 PowerShell 來部署訂用帳戶中的 [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-template-workspace-configuration)或 [Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-powershell)，表示您已移至新的定價模式。 如果您為 Log Analytics 指定「pergb2018」以外的定價層/方案，或是為 Application Insights 指定「基本」方案，就不會因為指定了無效的定價層/方案而導致部署失敗；不過，雖然部署成功，**但系統也只會使用有效的定價層/方案** (此方式不適用於會產生無效定價層訊息的 Log Analytics 免費層)。
>

## <a name="moving-to-the-new-pricing-model"></a>移轉至新的定價模式

如果您已決定為訂用帳戶採用新的定價模式，請選取 [使用量和估計成本] 頁面頂端的 [定價模式選取] 選項：

![監視新定價模式中的使用量和估計成本的螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/006.png)

[定價模式選取] 頁面隨即開啟。 此頁面會顯示您在先前頁面上檢視的每個訂用帳戶的清單：

![定價模式選取螢幕擷取畫面](./media/monitoring-usage-and-estimated-costs/007.png)

若要將訂用帳戶移轉至新的定價模式，請先選取方塊，再選取 [儲存] 即可。 您可以用相同方式重新移轉至舊的定價模式。 請注意，必須具備訂用帳戶擁有者或參與者權限，才能變更定價模式。

## <a name="automate-moving-to-the-new-pricing-model"></a>自動移轉至新的定價模式

下列指令碼需要 Azure PowerShell 模組。 若要檢查是否有最新版，請參閱[安裝 Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-6.1.0)。

您有最新版的 Azure PowerShell 後，需要先執行 ``Connect-AzureRmAccount``。

``` PowerShell
# To check if your subscription is eligible to adjust pricing models.
$ResourceID ="/subscriptions/<Subscription-ID-Here>/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action listmigrationdate `
 -Force
```

您可在定價模型之間，移動 isGrandFatherableSubscription 下表示此訂用帳戶定價模型的 True 結果。 optedInDate 下缺少值表示此訂用帳戶目前已設定為舊的定價模型。

```
isGrandFatherableSubscription optedInDate
----------------------------- -----------
                         True            
```

若要將此訂用帳戶移轉至新的定價模型執行：

```PowerShell
$ResourceID ="/subscriptions/<Subscription-ID-Here>/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action migratetonewpricingmodel `
 -Force
```

若要確認變更成功，請重新執行：

```PowerShell
$ResourceID ="/subscriptions/<Subscription-ID-Here>/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action listmigrationdate `
 -Force
```

如果移轉成功，結果應該如下所示：

```
isGrandFatherableSubscription optedInDate                      
----------------------------- -----------                      
                         True 2018-05-31T13:52:43.3592081+00:00
```

optInDate 現在包含此訂用帳戶選擇加入新的定價模型時的時間戳記。

如果您需要還原回舊的定價模型，您可以執行：

```PowerShell
 $ResourceID ="/subscriptions/<Subscription-ID-Here>/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action rollbacktolegacypricingmodel `
 -Force
```

如果您重新執行有 ``-Action listmigrationdate`` 的上述指令碼，您現在應該會看到空白 optedInDate 值，指出您的訂用帳戶已恢復舊版定價模型。

如果您想要移轉裝載在相同租用戶的多個訂用帳戶，可使用下列指令碼的片段建立您自己的變體：

```PowerShell
#Query tenant and create an array comprised of all of your tenants subscription ids
$TenantId = <Your-tenant-id>
$Tenant =Get-AzureRMSubscription -TenantId $TenantId
$Subscriptions = $Tenant.Id
```

若要檢查您租用戶中的所有訂用帳戶是否適合新的定價模型，您可以執行：

```PowerShell
Foreach ($id in $Subscriptions)
{
$ResourceID ="/subscriptions/$id/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action listmigrationdate `
 -Force
}
```

建立產生三個陣列的指令碼可以進一步重新調整指令碼。 一個陣列將包含 ```isGrandFatherableSubscription``` 是設為 True 的所有訂用帳戶識別碼，且 optedInDate 目前沒有值。 目前在新定價模型上的任何訂用帳戶第二陣列。 僅填入租用戶訂用帳戶識別碼 (不適合新的定價模型) 的第三陣列：

```PowerShell
[System.Collections.ArrayList]$Eligible= @{}
[System.Collections.ArrayList]$NewPricingEnabled = @{}
[System.Collections.ArrayList]$NotEligible = @{}

Foreach ($id in $Subscriptions)
{
$ResourceID ="/subscriptions/$id/providers/microsoft.insights"
$Result= Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action listmigrationdate `
 -Force

     if ($Result.isGrandFatherableSubscription -eq $True -and [bool]$Result.optedInDate -eq $False)
     {
     $Eligible.Add($id)
     }

     elseif ($Result.isGrandFatherableSubscription -eq $True -and [bool]$Result.optedInDate -eq $True)
     {
     $NewPricingEnabled.Add($id)
     }

     elseif ($Result.isGrandFatherableSubscription -eq $False)
     {
     $NotEligible.add($id)
     }
}
```

> [!NOTE]
> 端視訂用帳戶數目而定，上述指令碼可能需要一些時間才能執行。 由於使用 .add() 方法，PowerShell 視窗將回應遞增值，因為項目會加入至每個陣列。

由於您將訂用帳戶分為三個陣列，因此您應該仔細檢閱您的結果。 您會想要備份陣列的內容，以便未來有需要時還原變更。 如果您已決定，您會想要將目前在舊的定價模型上的所有合格訂用帳戶轉換為新的定價模型，此工作現在應該會完成：

```PowerShell
Foreach ($id in $Eligible)
{
$ResourceID ="/subscriptions/$id/providers/microsoft.insights"
Invoke-AzureRmResourceAction `
 -ResourceId $ResourceID `
 -ApiVersion "2017-10-01" `
 -Action migratetonewpricingmodel `
 -Force
}

```