---
title: 使用 Azure 自動化將伺服器設定為所需狀態並管理漂移
description: 教學課程 - 使用 Azure Automation State Configuration 管理伺服器組態
services: automation
ms.service: automation
ms.component: dsc
author: bobbytreed
ms.author: robreed
manager: carmonm
ms.topic: conceptual
ms.date: 08/08/2018
ms.openlocfilehash: 000875e4e591bcfe94ec99f8b16c8ec40bf52cf4
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45629877"
---
# <a name="configure-servers-to-a-desired-state-and-manage-drift"></a>將伺服器設定為預期狀態並管理漂移

Azure Automation State Configuration 可讓您指定伺服器的組態，並且確定這些伺服器處於指定狀態一段時間。

> [!div class="checklist"]
> - 上架 VM 讓 Azure 自動化 DSC 管理
> - 將設定上傳至 Azure 自動化
> - 將設定編譯成節點設定
> - 將節點設定指派給受控節點
> - 檢查受控節點的合規性狀態

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：

- Azure 自動化帳戶。 如需建立 Azure 自動化執行身分帳戶的指示，請參閱 [Azure 執行身分帳戶](automation-sec-configure-azure-runas-account.md)。
- 執行 Windows Server 2008 R2 或更新版本的 Azure Resource Manager VM (不是傳統)。 如需建立 VM 的指示，請參閱 [在 Azure 入口網站中建立第一個 Windows 虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md)
- Azure PowerShell 模組 3.6 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。
- 熟悉 Desired State Configuration (DSC)。 如需 DSC 的詳細資訊，請參閱 [Windows PowerShell 預期狀態設定概觀](https://docs.microsoft.com/powershell/dsc/overview)

## <a name="log-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶並遵循畫面上的指示。

```powershell
Connect-AzureRmAccount
```

## <a name="create-and-upload-a-configuration-to-azure-automation"></a>建立設定並將其上傳至 Azure 自動化

針對此教學課程，我們將會使用簡單的 DSC 設定，以確保在 VM 上安裝 IIS。

如需 DSC 設定的詳細資訊，請參閱 [DSC 設定](/powershell/dsc/configurations)。

在文字編輯器中輸入下列項目，並將其本機儲存為 `TestConfig.ps1`。

```powershell
configuration TestConfig {
   Node WebServer {
      WindowsFeature IIS {
         Ensure               = 'Present'
         Name                 = 'Web-Server'
         IncludeAllSubFeature = $true
      }
   }
}
```

呼叫 `Import-AzureRmAutomationDscConfiguration` Cmdlet 以將設定上傳至您的自動化帳戶：

```powershell
 Import-AzureRmAutomationDscConfiguration -SourcePath 'C:\DscConfigs\TestConfig.ps1' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Published
```

## <a name="compile-a-configuration-into-a-node-configuration"></a>將設定編譯成節點設定

DSC 設定必須編譯成節點設定，才可以指派至節點。

如需編譯設定的詳細資訊，請參閱 [DSC 設定](/powershell/dsc/configurations)。

呼叫 `Start-AzureRmAutomationDscCompilationJob` Cmdlet 以將 `TestConfig` 設定編譯成節點設定：

```powershell
Start-AzureRmAutomationDscCompilationJob -ConfigurationName 'TestConfig' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount'
```

這樣會在您的自動化帳戶中建立名為 `TestConfig.WebServer` 的節點設定。

## <a name="register-a-vm-to-be-managed-by-state-configuration"></a>註冊 VM 以供 State Configuration 管理

您可以使用 Azure Automation State Configuration 來管理 Azure VM (傳統和 Resource Manager)、內部部署 VM、Linux 機器、AWS VM 和內部部署實體機器。 在本主題中，我們將討論如何只註冊 Azure Resource Manager VM。 如需將其他類型的機器註冊的詳細資訊，請參閱 [將機器上架以供 Azure Automation State Configuration 管理](automation-dsc-onboarding.md)。

呼叫 `Register-AzureRmAutomationDscNode` Cmdlet 以向 Azure Automation State Configuration 註冊您的 VM。

```powershell
Register-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm'
```

這樣會將指定的 VM 註冊為 State Configuration 中的受控節點。

### <a name="specify-configuration-mode-settings"></a>指定設定模式設定

當您將 VM 註冊為受控節點時，您也可以指定設定的屬性。 例如，您可以指定機器的狀態僅套用一次 (DSC 不會在初始檢查之後嘗試套用設定)，方法是指定 `ApplyOnly` 作為 **ConfigurationMode** 屬性的值：

```powershell
Register-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationMode 'ApplyOnly'
```

您也可以指定 DSC 檢查設定狀態的頻率，方法是使用 **ConfigurationModeFrequencyMins** 屬性：

```powershell
# Run a DSC check every 60 minutes
Register-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationModeFrequencyMins 60
```

如需有關設定受控節點之設定屬性的詳細資訊，請參閱 [Register-AzureRmAutomationDscNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode)。

如需有關 DSC 設定的詳細資訊，請參閱[設定本機設定管理員](/powershell/dsc/metaconfig)。

## <a name="assign-a-node-configuration-to-a-managed-node"></a>將節點設定指派給受控節點

現在我們可以將已編譯的節點設定指派給我們想要設定的 VM。

```powershell
# Get the ID of the DSC node
$node = Get-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Assign the node configuration to the DSC node
Set-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeConfigurationName 'TestConfig.WebServer' -Id $node.Id
```

這樣會將名為 `TestConfig.WebServer` 的節點設定指派至名為 `DscVm` 的已註冊 DSC 節點。
根據預設，DSC 節點會每隔 30 分鐘檢查節點設定的合規性。
如需如何變更合規性檢查間隔的詳細資訊，請參閱[設定本機組態管理員](/PowerShell/DSC/metaConfig)。

## <a name="check-the-compliance-status-of-a-managed-node"></a>檢查受控節點的合規性狀態

您可以藉由呼叫 `Get-AzureRmAutomationDscNodeReport` Cmdlet，取得受控節點合規性狀態的報告：

```powershell
# Get the ID of the DSC node
$node = Get-AzureRmAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Get an array of status reports for the DSC node
$reports = Get-AzureRmAutomationDscNodeReport -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Id $node.Id

# Display the most recent report
$reports[0]
```

## <a name="next-steps"></a>後續步驟

- 若要開始使用，請參閱[開始使用 Azure 自動化狀態設定](automation-dsc-getting-started.md)
- 若要深入了解如何將節點上架，請參閱[將機器上架交由 Azure Automation State Configuration 管理](automation-dsc-onboarding.md)
- 若要了解如何編譯 DSC 組態，以將它們指派給目標節點，請參閱[編譯 Azure Automation State Configuration 中的組態](automation-dsc-compile.md)
- 如需 PowerShell Cmdlet 參考，請參閱 [Azure 自動化狀態設定 Cmdlet](/powershell/module/azurerm.automation/#automation)
- 如需定價資訊，請參閱 [Azure 自動化狀態設定的定價](https://azure.microsoft.com/pricing/details/automation/)
- 若要查看在持續部署管線中使用 Azure 自動化狀態設定的範例，請參閱[使用 Azure 自動化狀態設定和 Chocolatey 的持續部署](automation-dsc-cd-chocolatey.md)