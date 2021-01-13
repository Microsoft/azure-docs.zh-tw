---
title: 管理 Azure 自動化中的變數
description: 本文說明如何在 Runbook 和 DSC 組態中使用變數。
services: automation
ms.subservice: shared-capabilities
ms.date: 12/01/2020
ms.topic: conceptual
ms.openlocfilehash: 6db0c82c034aab97deee1be4aa8bdc54368521bc
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98131520"
---
# <a name="manage-variables-in-azure-automation"></a>管理 Azure 自動化中的變數

變數資產是可用於自動化帳戶中所有 Runbook 和 DSC 組態的值。 您可以從 Azure 入口網站、從 PowerShell、在 PowerShell 內，或在 DSC 組態中加以管理。

自動化變數對下列案例很實用：

- 在多個 Runbook 或 DSC 組態之間共用值。

- 在相同 Runbook 或 DSC 組態的多個作業之間共用值。

- 從入口網站或 Windows PowerShell 命令列管理 Runbook 或 DSC 組態所使用的值。 例如，一組一般設定項目，例如特定的 VM 名稱清單、特定的資源群組、AD 網域名稱等。  

Azure 自動化會保存變數，並使其可供使用，即使 Runbook 或 DSC 組態失敗也一樣。 此行為可讓一個 Runbook 或 DSC 組態設定另一個 Runbook 所使用的值，或下一次其執行時的相同 Runbook 或 DSC 組態。

Azure 自動化會安全地儲存每個加密變數。 當您建立變數時，您可以 Azure 自動化為安全資產來指定其加密和儲存。 建立變數之後，您就無法在未重新建立變數的情況下變更其加密狀態。 如果您有自動化帳戶變數儲存了尚未加密的敏感資料，您必須加以刪除，並將其重新建立為加密變數。 Azure 資訊安全中心的建議是將所有 Azure 自動化變數加密，如[應加密自動化帳戶變數](../../security-center/recommendations-reference.md#recs-compute)所說明。 如果您有要從此安全性建議中排除的未加密變數，請參閱[從建議與安全分數中豁免資源](../../security-center/exempt-resource.md)，以建立豁免規則。

>[!NOTE]
>Azure 自動化中的安全資產包括認證、憑證、連接和加密的變數。 這些資產都會經過加密，並使用為每個自動化帳戶產生的唯一金鑰，儲存在 Azure 自動化中。 Azure 自動化會將金鑰儲存在系統管理的 Key Vault 中。 在儲存安全資產之前，自動化會從 Key Vault 載入金鑰，然後將其用來加密資產。

## <a name="variable-types"></a>變數型別

使用 Azure 入口網站建立變數時，您必須從下拉式清單中指定資料類型，使入口網站可以對輸入變數的值顯示適當的控制項。 以下是可在 Azure 自動化中使用的變數類型：

* String
* 整數
* Datetime
* Boolean
* Null

變數不會受限於指定的資料類型。 如果您想要指定不同類型的值，必須使用 Windows PowerShell 來設定變數。 如果您指出 `Not defined`，變數的值會設定為 Null。 您必須使用 [Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable) Cmdlet 或內部 `Set-AutomationVariable` Cmdlet 來設定此值。

您無法使用 Azure 入口網站來建立或變更複雜變數類型的值。 不過，您可以使用 Windows PowerShell 提供任何類型的值。 複雜類型會以Newtonsoft.Js的形式抓取 [ 。](https://www.newtonsoft.com/json/help/html/N_Newtonsoft_Json_Linq.htm) 複雜物件類型的 JProperty，而不是 PSObject 類型 [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject)。

您可以藉由建立陣列或雜湊表，並將它儲存到變數中，來儲存多個值到單一變數。

>[!NOTE]
>VM 名稱變數最多可有 80 個字元。 資源群組變數最多可以有 90 個字元。 請參閱 [Azure 資源的命名規則和限制](../../azure-resource-manager/management/resource-name-rules.md)。

## <a name="powershell-cmdlets-to-access-variables"></a>存取變數的 PowerShell Cmdlet

下表中的 Cmdlet 會使用 PowerShell 來建立及管理自動化變數。 其會隨附於 [Az 模組](modules.md#az-modules)中。

| Cmdlet | 描述 |
|:---|:---|
|[Get-AzAutomationVariable](/powershell/module/az.automation/get-azautomationvariable) | 擷取現有變數的值。 如果值為簡單類型，則會擷取該相同的類型。 如果其為複雜類型，則會擷取 `PSCustomObject` 類型。 <br>**注意：** 您無法使用這個 Cmdlet 來取出加密變數的值。 若要這麼做，唯一的方法是在 Runbook 或 DSC 組態中使用內部 `Get-AutomationVariable` Cmdlet。 請參閱[存取變數的內部 Cmdlet](#internal-cmdlets-to-access-variables)。 |
|[New-AzAutomationVariable](/powershell/module/az.automation/new-azautomationvariable) | 建立新的變數並設定其值。|
|[Remove-AzAutomationVariable](/powershell/module/az.automation/remove-azautomationvariable)| 移除現有的變數。|
|[Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable)| 設定現有的變數的值。 |

## <a name="internal-cmdlets-to-access-variables"></a>存取變數的內部 Cmdlet

下表中的內部 Cmdlet 是用來存取 Runbook 和 DSC 組態中的變數。 這些 Cmdlet 隨附於全域模組 `Orchestrator.AssetManagement.Cmdlets`。 如需詳細資訊，請參閱[內部 Cmdlet](modules.md#internal-cmdlets)。

| 內部 Cmdlet | 描述 |
|:---|:---|
|`Get-AutomationVariable`|擷取現有變數的值。|
|`Set-AutomationVariable`|設定現有的變數的值。|

> [!NOTE]
> 避免在 Runbook 或 DSC 組態中，`Get-AutomationVariable` 的 `Name` 參數中使用變數。 使用變數可能會在設計階段將 Runbook 和自動化變數之間的相依性探索複雜化。

`Get-AutomationVariable` 無法在 PowerShell 中運作，但只能在 Runbook 或 DSC 組態中使用。 例如，若要查看加密變數的值，您可以建立 Runbook 來取得該變數，然後將其寫入輸出資料流：

```powershell
$mytestencryptvar = Get-AutomationVariable -Name TestVariable
Write-output "The encrypted value of the variable is: $mytestencryptvar"
```

## <a name="python-functions-to-access-variables"></a>用來存取變數的 Python 函式

下表中的函式可用來存取 Python 2 和 3 runbook 中的變數。 Python 3 runbook 目前為預覽狀態。

|Python 函式|描述|
|:---|:---|
|`automationassets.get_automation_variable`|擷取現有變數的值。 |
|`automationassets.set_automation_variable`|設定現有的變數的值。 |

> [!NOTE]
> 您必須在 Python Runbook 的頂端匯入 `automationassets` 模組，才能存取資產函式。

## <a name="create-and-get-a-variable"></a>建立並取得變數

>[!NOTE]
>如果您想要移除變數的加密，必須刪除變數，並將其重新建立為未加密。

### <a name="create-and-get-a-variable-using-the-azure-portal"></a>使用 Azure 入口網站建立並取得變數

1. 從您的自動化帳戶，在左窗格中選取 [**共用資源**] 底下的 [**變數**]。
2. 在 [ **變數** ] 頁面上，選取 [ **加入變數**]。
3. 完成 [ **新增變數** ] 頁面上的選項，然後選取 [ **建立** ] 以儲存新變數。

> [!NOTE]
> 儲存加密變數之後，就無法在入口網站中看到它。 只能將其更新。

### <a name="create-and-get-a-variable-in-windows-powershell"></a>在 Windows PowerShell 中建立及取得變數

您的 Runbook 或 DSC 組態會使用 `New-AzAutomationVariable` Cmdlet 來建立新的變數，並設定其初始值。 如果變數已加密，則呼叫應該使用 `Encrypted` 參數。 您的指令碼可以使用 `Get-AzAutomationVariable` 來擷取變數的值。

>[!NOTE]
>PowerShell 指令碼無法擷取已加密的值。 若要這麼做，唯一的方法是使用內部 `Get-AutomationVariable` Cmdlet。

下列範例示範如何建立字串變數，然後傳回其值。

```powershell
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" 
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' `
–Encrypted $false –Value 'My String'
$string = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value
```

下列範例示範如何建立具有複雜類型的變數，然後擷取其屬性。 在此情況下，會使用 [new-azvm](/powershell/module/Az.Compute/Get-AzVM) 中的虛擬機器物件來指定其屬性的子集。

```powershell
$vm = Get-AzVM -ResourceGroupName "ResourceGroup01" –Name "VM01" | Select Name, Location, Extensions
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm

$vmValue = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable"

$vmName = $vmValue.Name
$vmExtensions = $vmValue.Extensions
```

## <a name="textual-runbook-examples"></a>文字式 Runbook 範例

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

下列範例說明示範設定及擷取文字式 Runbook 中的變數。 這個範例假設建立名為 `NumberOfIterations` 和 `NumberOfRunnings` 的整數變數，以及名為 `SampleMessage` 的字串變數。

```powershell
$NumberOfIterations = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfIterations'
$NumberOfRunnings = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfRunnings'
$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'

Write-Output "Runbook has been run $NumberOfRunnings times."

for ($i = 1; $i -le $NumberOfIterations; $i++) {
    Write-Output "$i`: $SampleMessage"
}
Set-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name NumberOfRunnings –Value ($NumberOfRunnings += 1)
```

# <a name="python-2"></a>[Python 2](#tab/python2)

以下範例示範如何在 Python 2 Runbook 中取得變數、設定變數，以及處理不存在變數的例外狀況。

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print "variable not found"
```

# <a name="python-3"></a>[Python 3](#tab/python3) \(英文\)

下列範例示範如何取得變數、設定變數，以及處理 Python 3 runbook (preview) 中不存在之變數的例外狀況。

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print ("variable not found")
```

---

## <a name="graphical-runbook-examples"></a>圖形化 Runbook 範例

在圖形化 Runbook 中，您可以新增內部 Cmdlet `Get-AutomationVariable` 或 `Set-AutomationVariable` 的活動。 只要以滑鼠右鍵按一下圖形化編輯器 [程式庫] 窗格中的每個變數，然後選取您想要的活動。

![加入變數至畫布](../media/variables/runbook-variable-add-canvas.png)

下圖示範的範例活動會在圖形化 Runbook 中使用簡單值來更新變數。 在此範例中，`Get-AzVM` 的活動會擷取單一 Azure 虛擬機器，並將電腦名稱稱儲存至現有的自動化字串變數。 [連結是管線或順序](../automation-graphical-authoring-intro.md#use-links-for-workflow)都沒關係，因為程式碼在輸出中只預期單一物件。

![設定簡單變數](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>後續步驟

* 若要深入了解用來存取變數的 Cmdlet，請參閱[管理 Azure 自動化中的模組](modules.md)。
* 如需 Runbook 的一般資訊，請參閱 [Azure 自動化中的 Runbook 執行](../automation-runbook-execution.md)。
* 如需 DSC 設定的詳細資料，請參閱 [Azure 自動化狀態設定概觀](../automation-dsc-overview.md)。
