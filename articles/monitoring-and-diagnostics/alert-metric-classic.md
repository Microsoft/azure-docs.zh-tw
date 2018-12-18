---
title: 使用 Azure 監視器建立、檢視及管理傳統計量警示
description: 了解如何使用 Azure 入口網站、CLI 或 Powershell 來建立、檢視及管理傳統計量警示規則。
author: snehithm
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: snmuvva
ms.component: alerts
ms.openlocfilehash: e0c9cc342bb4b06053f53abc891ee1c415327998
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409995"
---
# <a name="create-view-and-manage-classic-metric-alerts-using-azure-monitor"></a>使用 Azure 監視器建立、檢視及管理傳統計量警示

想在其中一個計量超過閾值時收到通知，可以利用 Azure 監視器中的傳統計量警示。 傳統計量警示是較舊的功能，允許只針對無維度計量發出警示。 現有的較新功能稱為計量警示，和傳統計量警示相比，功能更強。 您可以在[計量警示概觀](alert-metric-overview.md)深入了解新的計量警示功能。 在此文章中，我們將會說明如何透過 Azure 入口網站、Azure CLI 與 Powershell 建立、檢視及管理傳統計量警示規則。

## <a name="with-azure-portal"></a>使用 Azure 入口網站

1. 在[入口網站](https://portal.azure.com/)中，找出您想要監視的資源，然後選取它。

2. 選取 [監視] 區段底下的 [警示 (傳統)]。 不同資源的文字和圖示會有些許不同。 如果在此處找不到 [警示 (傳統)]，可能會位在 [警示] 或 [警示規則] 中。

    ![監視](./media/alerts-metric-classic/AlertRulesButton.png)

3. 選取 [新增計量警示 (傳統)] 命令，然後填寫各欄位。

    ![新增警示](./media/alerts-metric-classic/AddAlertOnlyParamsPage.png)

4. **命名**警示規則。 然後選擇也會顯示在電子郵件通知中的 [描述]。

5. 選取您想要監視的**計量**。 然後選擇計量的**條件**和**閾值**。 同時選擇警示觸發程序之前，計量規則必須滿足的 [期間]。 例如，如果您使用「超過最後 5 分鐘」期間，且您的警示會尋找高於 80% 的 CPU，當 CPU 已持續 5 分鐘高於 80%，警示就會觸發。 發生第一次觸發之後，它會在 CPU 持續 5 分鐘低於 80 % 時再次觸發。 系統每分鐘會測量一次 CPU 計量。

6. 如果您想要讓系統管理員和共同管理員在警示引發時收到電子郵件通知，請選取 [電子郵件的擁有者...] 。

7. 如果您想要在警示引發時傳送通知給其他電子郵件地址，請在 [其他管理員的電子郵件] 欄位中新增。 使用下列格式，以分號分隔多個電子郵件：email@contoso.com; email2@contoso.com

8. 如果您想在警示引發時呼叫 Webhook，在 [Webhook]  欄位中放入有效的 URI。

9. 如果您使用 Azure 自動化，可以選取警示引發時執行的 Runbook。

10. 選取 [確定] 可建立警示。

在幾分鐘之內，警示會開始作用，且先前所述觸發。

建立警示之後，您可以選取此警示，然後進行下列其中一個工作：

* 檢視圖表，其中顯示計量閾值與前一天的實際值。
* 編輯或刪除警示。
* 如果您想要暫時停止或恢復接收警示的通知，可以**停用**或**啟用**警示。

## <a name="with-azure-cli"></a>透過 Azure CLI

先前幾節說明如何使用 Azure 入口網站建立、檢視及管理計量警示規則。 此節說明如何使用跨平台 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) 執行相同的作業。 開始使用 Azure CLI 最快的方式就是透過 [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview?view=azure-cli-latest)。

### <a name="get-all-classic-metric-alert-rules-in-a-resource-group"></a>取得資源群組中的所有傳統警示規則

```azurecli
az monitor alert list --resource-group <group name>
```

### <a name="see-details-of-a-particular-classic-metric-alert-rule"></a>查看特定傳統計量警示規則的詳細資訊

```azurecli
az monitor alert show --resource-group <group name> --name <alert name>
```

### <a name="create-a-classic-metric-alert-rule"></a>建立傳統計量警示規則

```azurecli
az monitor alert create --name <alert name> --resource-group <group name> \
    --action email <email1 email2 ...> \
    --action webhook <URI> \
    --target <target object ID> \
    --condition "<METRIC> {>,>=,<,<=} <THRESHOLD> {avg,min,max,total,last} ##h##m##s"
```

### <a name="delete-a-classic-metric-alert-rule"></a>刪除傳統計量警示規則

```azurecli
az monitor alert delete --name <alert name> --resource-group <group name>
```

## <a name="with-powershell"></a>透過 PowerShell

此節示範如何使用 PowerShell 命令建立、檢視及管理傳統計量警示。文章中的範例說明針對傳統計量警示使用 Azure 監視器 Cmdlet 的方式。

1. 設定要在電腦上執行的 PowerShell (如果您還未設定)。 如需詳細資訊，請參閱[如何安裝及設定 PowerShell](/powershell/azure/overview)。 您也可以在 [Azure 監視器 Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.insights) 檢閱整個 Azure 監視器 PowerShell Cmdlet 清單。

2. 首先，登入您的 Azure 訂用帳戶。

    ```PowerShell
    Connect-AzureRmAccount
    ```

3. 您會看到登入畫面。 一旦登入之後，就會顯示您的帳戶、TenantID 和預設的訂用帳戶識別碼。 所有 Azure Cmdlet 都會在您的預設訂用帳戶內容中運作。 若要檢視您具有存取權的訂用帳戶的清單，請使用下列命令：

    ```PowerShell
    Get-AzureRmSubscription
    ```

4. 若要將使用中的內容變更為不同的訂用帳戶，請使用下列命令：

    ```PowerShell
    Set-AzureRmContext -SubscriptionId <subscriptionid>
    ```

5. 您可以擷取資源群組中的所有傳統計量警示規則：

    ```PowerShell
    Get-AzureRmAlertRule -ResourceGroup montest
    ```

6. 您可以檢視傳統計量警示規則的詳細資料

    ```PowerShell
    Get-AzureRmAlertRule -Name simpletestCPU -ResourceGroup montest -DetailedOutput
    ```

7. 您可以擷取針對目標資源設定的所有警示規則。 例如，在 VM 上設定的所有警示規則。

    ```PowerShell
    Get-AzureRmAlertRule -ResourceGroup montest -TargetResourceId /subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig

8. You can use the `Add-AlertRule` cmdlet to create, update, or disable an alert rule. You can create email and webhook properties using  `New-AzureRmAlertRuleEmail` and `New-AzureRmAlertRuleWebhook`, respectively. In the Alert rule cmdlet, assign these properties as actions to the **Actions** property of the Alert Rule. The following table describes the parameters and values used to create an alert using a metric.

    | parameter | value |
    | --- | --- |
    | Name |simpletestdiskwrite |
    | Location of this alert rule |East US |
    | ResourceGroup |montest |
    | TargetResourceId |/subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig |
    | MetricName of the alert that is created |\PhysicalDisk(_Total)\Disk Writes/sec. See the `Get-MetricDefinitions` cmdlet about how to retrieve the exact metric names |
    | operator |GreaterThan |
    | Threshold value (count/sec in for this metric) |1 |
    | WindowSize (hh:mm:ss format) |00:05:00 |
    | aggregator (statistic of the metric, which uses Average count, in this case) |Average |
    | custom emails (string array) |'foo@example.com','bar@example.com' |
    | send email to owners, contributors and readers |-SendToServiceOwners |

9. Create an Email action

    ```PowerShell
    $actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com
    ```

10. 建立 Webhook 動作

    ```PowerShell
    $actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://example.com?token=mytoken
    ```

## <a name="next-steps"></a>後續步驟

- [使用 Resource Manager 範本建立傳統計量警示](monitoring-enable-alerts-using-template.md)。
- [設定傳統計量警示使用 Webhook 通知非 Azure 系統](insights-webhooks-alerts.md)。

