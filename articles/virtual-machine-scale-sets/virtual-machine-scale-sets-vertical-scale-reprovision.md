---
title: 垂直調整 Azure 虛擬機器擴展集 | Microsoft Docs
description: 如何垂直調整虛擬機器大小以回應 Azure 自動化的監視警示
services: virtual-machine-scale-sets
documentationcenter: ''
author: gatneil
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 16b17421-6b8f-483e-8a84-26327c44e9d3
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: article
ms.date: 08/03/2016
ms.author: negat
ms.openlocfilehash: 6e4733e023d1dc27fb099216f9afea07fe07446c
ms.sourcegitcommit: f46cbcff710f590aebe437c6dd459452ddf0af09
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/20/2017
ms.locfileid: "26781814"
---
# <a name="vertical-autoscale-with-virtual-machine-scale-sets"></a>使用虛擬機器擴展集垂直自動調整
這篇文章描述如何使用或不使用重新佈建以垂直調整 Azure [虛擬機器擴充集](https://azure.microsoft.com/services/virtual-machine-scale-sets/) 。 若為垂直調整不在擴展集中的虛擬機器，請參閱[使用 Azure 自動化垂直調整 Azure 虛擬機器](../virtual-machines/windows/vertical-scaling-automation.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

垂直調整也稱為向相應增加和相應減少，意味增加或減少虛擬機器 (VM) 大小以回應工作負載。 將此行為與[水平調整](virtual-machine-scale-sets-autoscale-overview.md) (也稱為相應放大和相應縮小) 做比較，後者會根據工作負載改變虛擬機器數目。

重新佈建表示移除現有的 VM，並以新的 VM 取代它。 當您增加或減少虛擬機器擴展集中的虛擬機器大小時，在某些情況下您想要調整現有虛擬機器的大小並保留資料，而在其他情況下您需要部署具有新大小的新虛擬機器。 本文件涵蓋這兩種情況。

在下列情況下，垂直調整可能十分有用︰

* 內建在虛擬機器上的服務使用量過低 (例如在週末)。 減少 VM 大小可以降低每月成本。
* 增加 VM 大小以應付更大的需求，而不需要建立額外的 VM。

您可以根據虛擬機器擴展集的度量型警示來設定要觸發的垂直調整。 啟動警示時它就會引發 Webhook 來觸發 Runbook，可讓您調整相應增加和相應減少設定。 您可以依照下列步驟來設定垂直調整︰

1. 使用執行身分功能來建立 Azure 自動化帳戶。
2. 將虛擬機器擴展集的 Azure 自動化垂直調整大小 Runbook 匯入訂用帳戶。
3. 將 Webhook 加入您的 Runbook 中。
4. 使用 Webhook 通知將警示加入至您的虛擬機器擴展集。

> [!NOTE]
> 垂直自動調整只能在特定範圍的 VM 大小內進行。 先比較各種大小的規格，然後再決定從一種大小調整成另一種大小 (數字較大並不一定代表 VM 大小較大)。 您可以在以下大小配對之間選擇調整︰
> 
> | 成對的調整 VM 大小 |  |
> | --- | --- |
> | Standard_A0 |Standard_A11 |
> | 標準_D1 |標準_D14 |
> | Standard_DS1 |Standard_DS14 |
> | Standard_D1v2 |Standard_D15v2 |
> | Standard_G1 |Standard_G5 |
> | Standard_GS1 |Standard_GS5 |
> 
> 

## <a name="create-an-azure-automation-account-with-run-as-capability"></a>使用執行身分功能來建立 Azure 自動化帳戶
您需要做的第一件事是建立 Azure 自動化帳戶，以此帳戶裝載用來調整虛擬機器擴展集執行個體的 Runbook。 最近，[Azure 自動化](https://azure.microsoft.com/services/automation/)引進了「執行身分帳戶」功能，此功能可設定服務主體以便代表使用者自動執行 Runbook。 如需詳細資訊，請參閱

* [使用 Azure 執行身分帳戶驗證 Runbook](../automation/automation-sec-configure-azure-runas-account.md)

## <a name="import-azure-automation-vertical-scale-runbooks-into-your-subscription"></a>將 Azure 自動化垂直調整大小 Runbook 匯入訂用帳戶
Azure 自動化 Runbook 資源庫已發佈垂直調整虛擬機器擴展集所需的 Runbook。 若要將其匯入到您的訂用帳戶，請依照這篇文章的步驟︰

* [Azure 自動化的 Runbook 和模組資源庫](../automation/automation-runbook-gallery.md)

從 Runbooks 功能表選擇 [瀏覽資源庫] 選項︰

![要匯入的 Runbook][runbooks]

需要如下所示匯入 Runbook。 根據您要使用重新佈建來垂直調整，以選取 Runbook：

![Runbook 資源庫][gallery]

## <a name="add-a-webhook-to-your-runbook"></a>將 Webhook 加入您的 Runbook 中
匯入 Runbook 之後，請將 Webhook 新增至 Runbook 中，使其可由虛擬機器擴展集所發出的警示加以觸發。 本文說明如何為 Runbook 建立 Webhook 的詳細資訊：

* [Azure 自動化 Webhook](../automation/automation-webhooks.md)

> [!NOTE]
> 關閉 Webhook 對話方塊之前，請務必複製 Webhook URI，因為下一節會需要此位址。
> 
> 

## <a name="add-an-alert-to-your-virtual-machine-scale-set"></a>對虛擬機器擴展集新增警示
下面的 PowerShell 指令碼會顯示如何對虛擬機器擴展集新增警示。 請參閱下列文章，取得度量名稱以引發警示︰[Azure 監視器自動調整的常用度量](../monitoring-and-diagnostics/insights-autoscale-common-metrics.md)。

```
$actionEmail = New-AzureRmAlertRuleEmail -CustomEmail user@contoso.com
$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri <uri-of-the-webhook>
$threshold = <value-of-the-threshold>
$rg = <resource-group-name>
$id = <resource-id-to-add-the-alert-to>
$location = <location-of-the-resource>
$alertName = <name-of-the-resource>
$metricName = <metric-to-fire-the-alert-on>
$timeWindow = <time-window-in-hh:mm:ss-format>
$condition = <condition-for-the-threshold> # Other valid values are LessThanOrEqual, GreaterThan, GreaterThanOrEqual
$description = <description-for-the-alert>

Add-AzureRmMetricAlertRule  -Name  $alertName `
                            -Location  $location `
                            -ResourceGroup $rg `
                            -TargetResourceId $id `
                            -MetricName $metricName `
                            -Operator  $condition `
                            -Threshold $threshold `
                            -WindowSize  $timeWindow `
                            -TimeAggregationOperator Average `
                            -Actions $actionEmail, $actionWebhook `
                            -Description $description
```

> [!NOTE]
> 建議您設定合理的警示時間範圍以避免頻繁觸發垂直調整，及任何相關聯的服務中斷。 請考慮至少 20-30 分鐘以上的時段。 如果需要避免任何中斷，請考慮水平調整。
> 
> 

如需如何建立警示的詳細資訊，請參閱下列文章：

* [Azure 監視器 PowerShell 快速入門範例](../monitoring-and-diagnostics/insights-powershell-samples.md)
* [Azure 監視器跨平台 CLI 快速入門範例](../monitoring-and-diagnostics/insights-cli-samples.md)

## <a name="summary"></a>總結
這篇文章示範簡單的垂直調整範例。 藉助這些建置組塊 (自動化帳戶、Runbook、Webhook、警示)，您可以連接各式各樣的事件與一組自訂的動作。

[runbooks]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks.png
[gallery]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks-gallery.png
