---
title: 使用 Azure 虛擬機器擴展集自動調整的概觀 | Microsoft Docs
description: 深入了解您可以根據效能或依固定的排程自動調整 Azure 虛擬機器擴展集的不同方式
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: d29a3385-179e-4331-a315-daa7ea5701df
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/29/2018
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7e6a897ed0606d8844f66e318b93449d7790e2be
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963420"
---
# <a name="overview-of-autoscale-with-azure-virtual-machine-scale-sets"></a>使用 Azure 虛擬機器擴展集自動調整的概觀
Azure 虛擬機器擴展集可以自動增加或減少執行您的應用程式的 VM 執行個體數目。 這個自動化和彈性的行為會降低監視和最佳化應用程式效能的管理額外負荷。 您可以建立規則，用以定義對於正面客戶體驗可接受的效能水準。 符合那些定義的閾值時，自動調整規則就會採取動作來調整擴展集的容量。 您也可以排定事件，以在固定時間自動增加或減少擴展集的容量。 本文提供可用效能計量以及可執行的自動調整動作的概觀。


## <a name="benefits-of-autoscale"></a>自動調整的優點
如果您的應用程式需求增加，您擴展集內 VM 執行個體上的負載也會跟著增加。 如果這樣的負載增加會持續而非只是短暫的需求，您就可以設定自動調整規則來增加擴展集中的 VM 執行個體數目。

建立這些 VM 執行個體並部署應用程式後，擴展集就會開始透過負載平衡器將流量分散給它們。 您可以控制要監視哪些計量 (例如 CPU 或記憶體)、應用程式負載必須符合給定的閾值多久，以及要將多少個 VM 執行個體新增至擴展集。

在夜晚或週末，您的應用程式需求可能會減少。 若這樣的負載減少會持續一段時間，您就可以設定自動調整規則來減少擴展集中的 VM 執行個體數目。 這個相應縮小的動作可減少執行擴展集的成本，因為只會執行符合目前需求所需的執行個體數目。


## <a name="use-host-based-metrics"></a>使用主機型計量
您可以建立內建於主機計量、可從您的 VM 執行個體取得的自動調整規則。 主機計量可讓您掌握擴展集中 VM 執行個體的效能，而不需要安裝或設定其他代理程式和資料收集。 使用這些計量的自動調整規模規則可以相應放大或相應縮小 VM 執行個體的數目，以回應 CPU 使用量、記憶體需求或磁碟存取。

可以使用下列其中一個工具來建立使用主機型計量的自動調整規則：

- [Azure 入口網站](virtual-machine-scale-sets-autoscale-portal.md)
- [Azure PowerShell](tutorial-autoscale-powershell.md)
- [Azure CLI](tutorial-autoscale-cli.md)
- [Azure 範本](tutorial-autoscale-template.md)

若要建立使用更詳細效能計量的自動調整規則，您可以在 VM 執行個體上[安裝及設定 Azure 診斷擴充功能](#in-guest-vm-metrics-with-the-azure-diagnostics-extension)，或[設定您的應用程式使用 App Insights](#application-level-metrics-with-app-insights)。

使用主機型計量的自動調整規模規則、具有 Azure 診斷擴充功能的客體 VM 內計量和 App Insights 可以使用下列組態設定。

### <a name="metric-sources"></a>計量來源
自動調整規模規則可以使用來自下列來源之一的計量：

| 計量來源        | 使用案例                                                                                                                     |
|----------------------|------------------------------------------------------------------------------------------------------------------------------|
| 目前的擴展集    | 針對不需要安裝或設定其他代理程式的主機型計量。                                  |
| 儲存體帳戶      | Azure 診斷擴充功能會將效能計量寫入 Azure 儲存體，然後用以觸發自動調整規則。 |
| 服務匯流排佇列    | 您的應用程式或其他元件可以在 Azure 服務匯流排佇列上傳輸訊息以觸發規則。                   |
| Application Insights | 安裝在您的應用程式中的檢測套件，會直接從應用程式串流處理計量。                         |


### <a name="autoscale-rule-criteria"></a>自動調整規則條件
建立自動調整規則時，以下主機型計量可供使用。 如果您使用 Azure 診斷擴充功能或 App Insights，您可以定義要使用自動調整規則監視和使用的計量。

| 度量名稱               |
|---------------------------|
| Percentage CPU            |
| Network In                |
| Network Out               |
| Disk Read Bytes           |
| Disk Write Bytes          |
| Disk Read Operations/Sec  |
| Disk Write Operations/Sec |
| 剩餘 CPU 信用額度     |
| CPU Credits Consumed      |

建立自動調整規則來監視指定的計量時，規則會查看下列其中一個計量彙總動作：

| 彙總類型 |
|------------------|
| 平均值          |
| 最小值          |
| 最大值          |
| 總計            |
| 最後一頁             |
| Count            |

使用下列其中一個運算子將計量與您定義的閾值相比較時，則會觸發自動調整規則：

| 運算子                 |
|--------------------------|
| 大於             |
| 大於或等於 |
| 小於                |
| 小於或等於    |
| 等於                 |
| 不等於             |


### <a name="actions-when-rules-trigger"></a>規則觸發時的動作
觸發自動調整規則時，可以利用下列其中一個方式自動調整您的擴展集：

| 調整規模作業     | 使用案例                                                                                                                               |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| 將計數增加   | 要建立的固定 VM 執行個體數目。 在具有較少數量 VM 的擴展集中很有用。                                           |
| 將百分比增加 | VM 執行個體以百分比為基礎的增加。 適用於固定增加可能無法大幅改善效能的較大擴展集。 |
| 將計數增加至   | 需要建立多個 VM 執行個體，才能達到所需的最大數量。                                                            |
| 將計數減少   | 要移除的固定 VM 執行個體數目。 在具有較少數量 VM 的擴展集中很有用。                                           |
| 將百分比減少 | VM 執行個體以百分比為基礎的減少。 適用於固定增加可能無法大幅減少資源耗用量和費用的較大擴展集。 |
| 將計數減少至   | 需要移除多個 VM 執行個體，才能達到所需的最小數量。                                                            |


## <a name="in-guest-vm-metrics-with-the-azure-diagnostics-extension"></a>具有 Azure 診斷擴充功能的客體 VM 內計量
Azure 診斷擴充功能是在 VM 執行個體內部執行的代理程式。 代理程式會監視並將效能計量儲存至 Azure 儲存體。 這些效能計量資訊包含 VM 狀態更詳細的資訊，例如磁碟的 *AverageReadTime* 或 CPU 的 *PercentIdleTime*。 您可以根據對 VM 效能更詳細的認知來建立自動調整規則，而不只是根據 CPU 使用量或記憶體耗用量的百分比。

若要使用 Azure 診斷擴充功能，您必須為您的 VM 執行個體建立 Azure 儲存體帳戶、安裝 Azure 診斷代理程式，然後設定 VM 將特定效能計數器串流至儲存體帳戶。

如需詳細資訊，請參閱如何在 [Linux VM](../virtual-machines/extensions/diagnostics-linux.md) 或 [Windows VM](../virtual-machines/extensions/diagnostics-windows.md) 上啟用 Azure 診斷擴充功能的文章。


## <a name="application-level-metrics-with-app-insights"></a>使用 App Insights 的應用程式層級計量
若要取得應用程式效能的更多可見性，您可以使用 Application Insights。 您可以在應用程式中安裝可監視應用程式並將遙測傳送至 Azure 的小型檢測套件。 您可以監視計量，例如應用程式的回應時間、頁面載入效能和工作階段計數。 這些應用程式計量可以用來以細微和內嵌層級建立自動調整規則，因為您會以可能影響客戶體驗的可採取動作深入資訊來觸發規則。

如需 App Insights 的詳細資訊，請參閱[什麼是 Application Insights](../application-insights/app-insights-overview.md)。


## <a name="scheduled-autoscale"></a>排定的自動調整
您也可以以排程為基礎建立自動調整規則。 這些以排程為基礎的規則可讓您在固定時間自動調整 VM 執行個體的數量。 利用以效能為基礎的規則，在觸發自動調整規則並且佈建新的 VM 執行個體之前，對應用程式可能有效能影響。 如果您可以預期這類需求，即會佈建其他 VM 執行個體，並且可供其他客戶用途與應用程式需求使用。

下列範例是可能因使用以排程為基礎的自動調整規則獲益的情節：

- 在客戶需求增加時的工作日開始時自動相應放大 VM 執行個體數目。 在工作日結束時，自動調整 VM 中的執行個體數目，在應用程式使用較低的夜間降低資源成本。
- 如果部門在月份或會計週期的某些時候會使用大量應用程式，則自動調整 VM 執行個體數目以配合部門的額外需求。
- 有行銷事件、促銷或節日銷售時，您可以在預期的客戶需求來臨前自動調整 VM 執行個體數目。 


## <a name="next-steps"></a>後續步驟
您可以使用下列其中一個工具來建立使用主機型計量的自動調整規則：

- [Azure PowerShell](tutorial-autoscale-powershell.md)
- [Azure CLI](tutorial-autoscale-cli.md)
- [Azure 範本](tutorial-autoscale-template.md)

本概觀詳細說明如何使用自動調整規則進行水平縮放，以及增加或減少擴展集中 VM 執行個體的*數目*。 您也可以進行垂直調整以增加或減少 VM 執行個體的*大小*。 如需詳細資訊，請參閱[使用虛擬機器擴展集垂直自動調整](virtual-machine-scale-sets-vertical-scale-reprovision.md)。

如需有關如何管理 VM 執行個體的資訊，請參閱[使用 Azure PowerShell 管理虛擬機器擴展集](virtual-machine-scale-sets-windows-manage.md)。

若要了解如何在自動調整規則觸發時產生警示，請參閱[使用自動調整動作在 Azure 監視器中傳送電子郵件和 Webhook 警示通知](../monitoring-and-diagnostics/insights-autoscale-to-webhook-email.md)。 您也可以[使用稽核記錄，在 Azure 監視器中傳送電子郵件和 Webhook 警示通知](../monitoring-and-diagnostics/insights-auditlog-to-webhook-email.md)。
