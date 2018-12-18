---
title: Azure 自動化中的排程
description: 自動化排程是用來排程讓 Azure 自動化中的 Runbook 自動啟動。 描述如何建立和管理排程，以便以特定時間或循環排程來自動啟動 Runbook。
services: automation
ms.service: automation
ms.component: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 09/18/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 3d8492d2a8982c9c85bfc91867f7eb6c2da04e58
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46294759"
---
# <a name="scheduling-a-runbook-in-azure-automation"></a>在 Azure 自動化中排程 Runbook

若要排程在指定時間啟動 Azure 自動化中的 Runbook，您會將它連結到一個或多個排程。 您可以在 Azure 入口網站中，將 Runbook 排程設定成執行一次，或是每小時或每天重複發生。 您也可以將其排定為在每週、每月、特定的星期幾、月份中的某些日子或月份中的特定日子執行。 Runbook 可以連結至多個排程，而排程可以有多個與其連結的 Runbook。

> [!NOTE]
> 排程目前不支援 Azure Automation DSC 組態。

## <a name="windows-powershell-cmdlets"></a>Windows PowerShell Cmdlet

下表中的 Cmdlet 是用來在 Azure 自動化中透過 Windows PowerShell 建立和管理排程。 它們是隨著 [Azure PowerShell 模組](/powershell/azure/overview)的一部分推出。

| Cmdlet | 說明 |
|:--- |:--- |
| [Get-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/get-azurermautomationschedule) |擷取排程。 |
| [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) |建立新排程。 |
| [Remove-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/remove-azurermautomationschedule) |移除排程。 |
| [Set-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/set-azurermautomationschedule) |設定現有排程的屬性。 |
| [Get-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/get-azurermautomationscheduledrunbook) |擷取排程的 Runbook。 |
| [Register-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/register-azurermautomationscheduledrunbook) |將 Runbook 與排程相關聯。 |
| [Unregister-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/unregister-azurermautomationscheduledrunbook) |從排程分離 Runbook。 |

## <a name="creating-a-schedule"></a>建立排程

您可以在 Azure 入口網站中或使用 Windows PowerShell，為 Runbook 建立新排程。

> [!NOTE]
> 執行新的排程工作時，Azure 自動化會使用自動化帳戶中的最新模組。  為了避免影響 Runbook 和它們所自動化的處理序，您應該先使用專用於測試的自動化帳戶測試任何具有連結排程的 Runbook。  這會驗證您排程的 Runbook 會持續正確運作，否則，您可以進一步進行疑難排解，並套用任何必要變更，再將已更新的 Runbook 版本移轉至生產環境。
> 除非您已選取 [模組] 中的[更新 Azure 模組](automation-update-azure-modules.md)選項進行手動更新，否則自動化帳戶不會自動取得任何新版本的模組。

### <a name="to-create-a-new-schedule-in-the-azure-portal"></a>在 Azure 入口網站中建立新排程

1. 在 Azure 入口網站中，從您的自動化帳戶選取左側 [共用資源] 區段底下的 [排程]。
1. 在分頁的頂端按一下 [加入排程]。
1. 在 [新增排程] 窗格中，為新排程輸入 [名稱] 並選擇性地輸入 [描述]。
1. 透過選取 [一次] 或 [定期]，以決定排程僅執行一次或定期執行。 如果選取 [一次]，請指定 [開始時間]，然後按一下 [建立]。 如果您選取 [定期]，請指定 [開始時間]，並針對 [重複頻率] 選取所需的 Runbook 重複頻率：按 [小時]、[天]、[週] 或 [月] 執行。
    1. 如果您選取 [週]，系統會提供該週的日期清單供您選擇。 選取您需要的天數。 首次執行排程的時間將會是開始時間後所選的第一天。
    2. 如果您選取 [月]，系統會提供您不同選項。 若是 [每月發生次數] 選項，請選取 [日期] 或 [星期]。 如果您選取 [日期]，系統會顯示行事曆以便您選擇所需的天數。 如果您選擇的某個日期 (例如 31 日) 未出現在目前月份，則排程不會執行。 如果您希望在最後一天執行排程，請在 [在每月最後一天執行] 下選取 [是]。 如果您選擇 [星期]，系統會顯示 [重複頻率] 選項。 選擇 [第一週]、[第二週]、[第三週]、[第四週] 或 [最後一週]。 最後，請選擇重複執行的日期。
1. 完成後，按一下 [建立]。

### <a name="to-create-a-new-schedule-with-windows-powershell"></a>使用 Windows PowerShell 建立新排程

使用 [New-AzureRmAutomationSchedule](/powershell/module/azurerm.automation/new-azurermautomationschedule) Cmdlet 來建立排程。 您必須指定排程的開始時間，以及其應該執行的頻率。

下列範例命令顯示如何使用 Azure Resource Manager Cmdlet，建立會在每月 15 號到 30 號執行的排程。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
New-AzureRMAutomationSchedule –AutomationAccountName $automationAccountName –Name `
$scheduleName -StartTime "7/01/2016 15:30:00" -MonthInterval 1 `
-DaysOfMonth Fifteenth,Thirtieth -ResourceGroupName "ResourceGroup01"
```

## <a name="linking-a-schedule-to-a-runbook"></a>將排程連結至 Runbook

Runbook 可以連結至多個排程，而排程可以有多個與其連結的 Runbook。 如果 Runbook 有參數，您可以為它們提供值。 您必須為任何必要參數提供值，並可以提供任何選擇性參數的值。 每次此排程啟動 Runbook 時會使用這些值。 您可以將相同的 Runbook 附加到另一個排程，並指定不同的參數值。

### <a name="to-link-a-schedule-to-a-runbook-with-the-azure-portal"></a>使用 Azure 入口網站將排程連結至 Runbook

1. 在 Azure 入口網站中，從您的自動化帳戶選取左側 [程序自動化] 區段底下的 [Runbook]。
2. 按一下要排程的 Runbook 名稱。
3. 如果 Runbook 目前未連結至排程，則將提供您建立新排程或連結至現有排程的選項。
4. 如果 Runbook 有參數，您可以選取 [修改執行設定 (預設值: Azure)] 選項，隨即便會顯示 [參數] 窗格供您據以輸入資訊。

### <a name="to-link-a-schedule-to-a-runbook-with-windows-powershell"></a>使用 Windows PowerShell 將排程連結至 Runbook

您可以使用 [Register-AzureRmAutomationScheduledRunbook](/powershell/module/azurerm.automation/register-azurermautomationscheduledrunbook) Cmdlet 來連結排程。 您可以使用 Parameters 參數來指定 Runbook 參數的值。 如需如何指定參數值的詳細資訊，請參閱[在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md)。
下列範例命令顯示如何使用 Azure Resource Manager Cmdlet 與參數，將排程連結至 Runbook。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$runbookName = "Test-Runbook"
$scheduleName = "Sample-DailySchedule"
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Register-AzureRmAutomationScheduledRunbook –AutomationAccountName $automationAccountName `
–Name $runbookName –ScheduleName $scheduleName –Parameters $params `
-ResourceGroupName "ResourceGroup01"
```

## <a name="scheduling-runbooks-more-frequently"></a>更頻繁地排程 Runbook

可為 Azure 自動化中的排程設定的最頻繁間隔是一小時。 如果您需要讓排程以比一小時更頻繁地頻率來執行，您有兩個選項：

* 為 Runbook 建立 [Webhook](automation-webhooks.md) 並使用 [Azure 排程器](../scheduler/scheduler-get-started-portal.md)來呼叫此 Webhook。 Azure 排程器在定義排程時，可提供更細微的方式。

* 建立四個排程，讓其每小時執行一次，且彼此全都會在相差 15 分鐘內開始。 此案例可讓 Runbook 使用不同排程每 15 分鐘執行一次。

## <a name="disabling-a-schedule"></a>停用排程

停用排程時，與其連結的任何 Runbook 無法再依該排程執行。 您可以手動停用排程，或在建立排程時為具有頻率的排程設定到期時間。 當到達到期時間時，就會停用排程。

### <a name="to-disable-a-schedule-from-the-azure-portal"></a>從 Azure 入口網站停用排程

1. 在 Azure 入口網站中，從您的自動化帳戶選取左側 [共用資源] 區段底下的 [排程]。
1. 按一下排程的名稱以開啟詳細資料窗格。
1. 將 [已啟用] 變更為 [否]。

### <a name="to-disable-a-schedule-with-windows-powershell"></a>使用 Windows PowerShell 停用排程

您可以使用 [Set-AzureAutomationSchedule](/powershell/module/azurerm.automation/set-azurermautomationschedule) Cmdlet 變更現有排程的屬性。 若要停用排程，請為 **IsEnabled** 參數指定 **false**。

下列範例命令顯示如何使用 Azure Resource Manager Cmdlet 來停用 Runbook 的排程。

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Set-AzureRmAutomationSchedule –AutomationAccountName $automationAccountName `
–Name $scheduleName –IsEnabled $false -ResourceGroupName "ResourceGroup01"
```

## <a name="next-steps"></a>後續步驟

* 若要在 Azure 自動化中開始使用 Runbook，請參閱 [在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md)
