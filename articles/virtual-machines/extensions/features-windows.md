---
title: 適用於 Windows 的 Azure VM 擴充功能和功能 | Microsoft Docs
description: 了解哪些擴充功能適用於 Azure 虛擬機器，並依它們提供或改善的內容來分組。
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 999d63ee-890e-432e-9391-25b3fc6cde28
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: roiyz
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: d7b183a5b87d5777609deac02e4424d9451e9643
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452491"
---
# <a name="virtual-machine-extensions-and-features-for-windows"></a>適用於 Windows 的虛擬機器擴充功能和功能

Azure 虛擬機器 (VM) 擴充功能是小型的應用程式，可在 Azure 虛擬機器上提供部署後設定及自動化工作。 例如，如果虛擬機器要求安裝軟體、防毒保護，或要求執行其中的指令碼，便可使用虛擬機器擴充功能。 您可以使用 Azure CLI、PowerShell、Azure Resource Manager 範本及 Azure 入口網站來執行 Azure 虛擬機器擴充功能。 擴充功能可以與新的虛擬機器部署搭配，或是在任何現有的系統上執行。

本文提供虛擬機器擴充功能概觀、使用 Azure 虛擬機器擴充功能的必要條件，以及如何偵測、管理和移除虛擬機器擴充功能的指引。 因為有許多可用的虛擬機器擴充功能，且每項功能的設定可能都盡不相同，因此本文所提供的是通用資訊。 可以在每份個別擴充功能專用的文件中找到擴充功能特定的詳細資料。

## <a name="use-cases-and-samples"></a>使用案例和範例

有數個不同的 Azure VM 擴充功能可供使用，各有特定使用案例。 部分範例包括：

- 使用適用於 Windows 的 DSC 擴充功能將 PowerShell 預期狀態設定套用至 VM。 如需詳細資訊，請參閱 [Azure 期望狀態組態擴充功能簡介](dsc-overview.md)。
- 使用 Microsoft 監視代理程式虛擬機器擴充功能來設定虛擬機器的監視。 如需詳細資訊，請參閱[將 Azure VM 連線至 Log Analytics](../../log-analytics/log-analytics-azure-vm-extension.md)。
- 使用 Chef 設定 Azure VM。 如需詳細資訊，請參閱[使用 Chef 自動化 Azure VM 部署](../windows/chef-automation.md)。
- 使用 Datadog 副檔名設定 Azure 基礎結構的監視。 如需詳細資訊，請參閱 [Datadog 部落格](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/)。


除了處理序特定擴充功能，自訂指令碼延伸模組適用於 Windows 和 Linux 虛擬機器。 適用於 Windows 的自訂指令碼擴充功能允許在 VM 上執行任何 PowerShell 指令碼。 自訂指令碼對於設計需要超過原生 Azure 工具可提供之設定的 Azure 部署很有用。 如需詳細資訊，請參閱 [Windows VM 自訂指令碼擴充功能](custom-script-windows.md)。

## <a name="prerequisites"></a>必要條件

若要處理虛擬機器上的擴充功能，您必須安裝 Azure Windows 代理程式。 某些個別的擴充會有先決條件，例如可存取資源或相依性。

### <a name="azure-vm-agent"></a>Azure VM 代理程式

Azure 虛擬機器代理程式可管理 Azure 虛擬機器與 Azure 網狀架構控制器之間的互動。 虛擬機器代理程式負責部署和管理 Azure 虛擬機器的許多功能層面，包括執行虛擬機器擴充功能。 Azure VM 代理程式會預先安裝在「Azure Marketplace 映像」上，並可手動安裝在支援的作業系統上。 適用於 Windows 的 Azure VM 代理程式一般稱為 Windows 客體代理程式。

如需有關支援的作業系統和安裝指示，請參閱 [Azure 虛擬機器代理程式](agent-windows.md)。

#### <a name="supported-agent-versions"></a>支援的代理程式版本

為了提供最佳使用體驗，代理程式的版本數量盡可能精簡。 如需詳細資訊，請參閱 [本篇文章](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)。

#### <a name="supported-oses"></a>支援的作業系統

Windows 客體代理程式可在多種 OS 上執行，但具有擴充功能的 OS 在擴充功能方面有所限制。 如需詳細資訊，請參閱 [本篇文章] (https://support.microsoft.com/en-us/help/4078134/azure-extension-supported-operating-systems)。

某些擴充功能並非所有作業系統都可支援，可能會發出錯誤碼 51：「不支援的 OS」。 請查看個別的擴充功能文件以了解支援度。

#### <a name="network-access"></a>網路存取

擴充功能套件可從 Azure 儲存體擴充功能存放庫下載，且擴充功能狀態上傳會發佈至 Azure 儲存體。 如果您使用[支援](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)版本的代理程式，就不需要允許存取虛擬機器區域中的 Azure 儲存體，因為可以使用代理程式將通訊重新導向至代理程式通訊的 Azure 網狀架構控制器。 如果您使用不受支援的代理程式版本，則必須允許從虛擬機器對該區域中的 Azure 儲存體進行存取。

> [!IMPORTANT]
> 如果您已使用客體防火牆封鎖對 168.63.129.16 的存取，則無論前述條件為何，擴充功能都會故障。

代理程式只能用來下載擴充功能套件和報告狀態。 例如，如果需要從 GitHub 下載指令碼 (自訂指令碼)，或需要存取 Azure 儲存體 (Azure 備份) 才能安裝擴充功能，則必須開啟其他防火牆/網路安全性群組連接埠。 不同的擴充功能有不同需求，因為它們是自成一格的應用程式。 對於需要存取 Azure 儲存體的擴充功能，您可以允許使用[儲存體](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)適用的 Azure NSG 服務標記進行存取。

Windows 客體代理程式並沒有 Proxy 伺服器支援可讓您重新導向代理程式傳輸要求。

## <a name="discover-vm-extensions"></a>探索 VM 擴充功能

有許多不同的虛擬機器擴充功能可與 Azure 虛擬機器搭配使用。 若要查看完整清單，請使用 [Get-AzureRmVMExtensionImage](/powershell/module/azurerm.compute/get-azurermvmextensionimage)。 下列範例列出 *WestUS* 位置中所有可用的擴充功能：

```powershell
Get-AzureRmVmImagePublisher -Location "WestUS" | `
Get-AzureRmVMExtensionImageType | `
Get-AzureRmVMExtensionImage | Select Type, Version
```

## <a name="run-vm-extensions"></a>執行 VM 延伸模組

Azure 虛擬機器擴充功能可以在現有的虛擬機器上執行，這在需要進行組態變更或復原已部署虛擬機器上的連線時很有用。 VM 擴充功能也可以搭配 Azure Resource Manager 範本部署。 使用具有 Resource Manager 範本的擴充功能，就可以部署及設定 Azure 虛擬機器而不需要在部署後介入。

下列方法可用來針對現有的虛擬機器執行擴充功能。

### <a name="powershell"></a>PowerShell

有數個 PowerShell 命令可用來執行個別的擴充功能。 若要查看清單，請使用 [Get-Command](/powershell/module/microsoft.powershell.core/get-command) 並對*擴充功能*進行篩選：

```powershell
Get-Command Set-AzureRM*Extension* -Module AzureRM.Compute
```

這會提供類似下面的輸出︰

```powershell
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Set-AzureRmVMAccessExtension                       4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMADDomainExtension                     4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMAEMExtension                          4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMBackupExtension                       4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMBginfoExtension                       4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMChefExtension                         4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMCustomScriptExtension                 4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMDiagnosticsExtension                  4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMDiskEncryptionExtension               4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMDscExtension                          4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMExtension                             4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVMSqlServerExtension                    4.5.0      AzureRM.Compute
Cmdlet          Set-AzureRmVmssDiskEncryptionExtension             4.5.0      AzureRM.Compute
```

下列範例會使用自訂指令碼擴充功能，從 GitHub 存放庫下載指令碼到目標虛擬機器上，然後執行該指令碼。 如需有關自訂指令碼擴充功能的詳細資訊，請參閱[自訂指令碼擴充功能概觀](custom-script-windows.md)。

```powershell
Set-AzureRmVMCustomScriptExtension -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" -Name "myCustomScript" `
    -FileUri "https://raw.githubusercontent.com/neilpeterson/nepeters-azure-templates/master/windows-custom-script-simple/support-scripts/Create-File.ps1" `
    -Run "Create-File.ps1" -Location "West US"
```

在此範例中，VM 存取擴充功能會用來重設 Windows 虛擬機器的系統管理密碼。 如需有關 VM 存取擴充功能的詳細資訊，請參閱[重設 Windows VM 中的遠端桌面服務](../windows/reset-rdp.md)。 在您執行此作業後，應在第一次登入時重設密碼：

```powershell
$cred=Get-Credential

Set-AzureRmVMAccessExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myVMAccess" `
    -Location WestUS -UserName $cred.GetNetworkCredential().Username `
    -Password $cred.GetNetworkCredential().Password -typeHandlerVersion "2.0"
```

`Set-AzureRmVMExtension` 命令可用來啟動任何 VM 擴充功能。 如需詳細資訊，請參閱 [Set-AzureRmVMExtension 參考資料](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmextension)。


### <a name="azure-portal"></a>Azure 入口網站

虛擬機器擴充功能可以透過 Azure 入口網站套用至現有的虛擬機器。 請在入口網站選取虛擬機器，選擇 [擴充功能]，然後選取 [新增]。 從可用擴充功能清單選擇您想要的擴充功能，並遵循精靈中的指示。

下列範例顯示從 Azure 入口網站安裝 Microsoft 反惡意程式碼擴充功能：

![安裝反惡意程式碼延伸模組](./media/features-windows/installantimalwareextension.png)

### <a name="azure-resource-manager-templates"></a>Azure 資源管理員範本

VM 擴充功能可以新增至 Azure Resource Manager 範本，並使用範本的部署執行。 當您使用範本部署擴充功能時，可以建立完全設定的 Azure 部署。 例如，下列 JSON 是取自部署一組負載平衡 VM 和 Azure SQL Database 的 Resource Manager 範本，並在每個 VM 上安裝 .NET 核心應用程式。 VM 擴充功能會處理軟體安裝。

如需詳細資訊，請參閱[完整的 Resource Manager 範本](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows)。

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

如需關於建立 Resource Manager 範本的詳細資訊，請參閱[使用 Windows VM 擴充功能編寫 Azure Resource Manager 範本](../windows/template-description.md#extensions)。

## <a name="secure-vm-extension-data"></a>安全的 VM 擴充功能資料

在執行虛擬機器擴充功能時，可能需要包含機密資訊，例如認證、儲存體帳戶名稱和儲存體帳戶存取金鑰。 許多虛擬機器擴充功能包含受保護的組態，其會加密資料並只在目標虛擬機器內加以解密。 每個擴充功能有特定受保護的組態結構描述，並每個都會在擴充功能特定文件中詳細說明。

下列範例會顯示適用於 Windows 的自訂指令碼擴充功能執行個體。 要執行的命令包含一組認證。 在此範例中，要執行的命令不會加密：

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ],
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

將**要執行的命令**屬性移至**受保護的**組態可保護執行字串，如下列範例所示：

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
    "[variables('musicstoresqlName')]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 -user ',parameters('adminUsername'),' -password ',parameters('adminPassword'),' -sqlserver ',variables('musicstoresqlName'),'.database.windows.net')]"
    }
    }
}
```

### <a name="how-do-agents-and-extensions-get-updated"></a>如何更新代理程式和擴充功能？

代理程式和擴充功能共用相同的更新機制。 某些更新不需要額外的防火牆規則。

有可用的更新時，只有在擴充功能有所變更和其他虛擬機器模型已變更時，才會在虛擬機器上安裝更新：

- 資料磁碟
- 擴充功能
- 開機診斷容器
- 客體作業系統祕密
- VM 大小
- 網路設定檔

發行者可能會在不同的時間將更新發行至不同區域，因此您有可能在不同的區域使用不同版本的 VM。

#### <a name="listing-extensions-deployed-to-a-vm"></a>列出部署到 VM 的擴充功能

```powershell
$vm = Get-AzureRmVM -ResourceGroupName "myResourceGroup" -VMName "myVM"
$vm.Extensions | select Publisher, VirtualMachineExtensionType, TypeHandlerVersion
```

```powershell
Publisher             VirtualMachineExtensionType          TypeHandlerVersion
---------             ---------------------------          ------------------
Microsoft.Compute     CustomScriptExtension                1.9
```

#### <a name="agent-updates"></a>代理程式更新

Windows 客體代理程式僅包含*擴充功能處理程式碼*，*Windows 佈建程式碼*須另行取得。 您可以解除安裝 Windows 客體代理程式。 您無法停用 Windows 客體代理程式的自動更新。

*擴充功能處理程式碼*會負責與 Azure 網狀架構通訊，並處理 VM 擴充功能作業，例如安裝、報告狀態、更新個別的擴充功能和加以移除。 更新項目包含*擴充功能處理程式碼*的安全性修正程式、Bug 修正程式和增強功能。

若要確認您正在執行的版本，請參閱[偵測已安裝的 Windows 客體代理程式](agent-windows.md#detect-the-vm-agent)。

#### <a name="extension-updates"></a>擴充功能更新

有可用的擴充功能更新時，Windows 客體代理程式即會下載並升級擴充功能。 自動擴充功能自動更新分為*次要*和 *Hotfix* 兩種。 您可以在佈建擴充更新時，選擇加入或退出擴充功能「次要」更新。 下列範例說明如何透過 *autoUpgradeMinorVersion": true,'* 在 Resource Manager 範本中自動升級次要版本：

```json
    "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.9",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1"
        ]
    },
```

若要取得最新的次要版本 Bug 修正程式，強烈建議您在擴充功能部署中一律選取自動更新。 您無法選擇退出含有安全性或重大 Bug 修正程式的 Hotfix 更新。

### <a name="how-to-identify-extension-updates"></a>如何識別擴充功能更新

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>識別是否已在虛擬機器上使用 autoUpgradeMinorVersion 設定擴充功能

如果已使用 'autoUpgradeMinorVersion' 佈建擴充功能，您應可從 VM 模型看出。 若要確認，請使用 [Get-AzureRmVm](/powershell/module/azurerm.compute/get-azurermvm)，並提供資源群組和 VM 名稱，如下所示：

```powerShell
 $vm = Get-AzureRmVm -ResourceGroupName "myResourceGroup" -VMName "myVM"
 $vm.Extensions
```

下列範例輸出顯示 *autoUpgradeMinorVersion* 設定為 *true*：

```powershell
ForceUpdateTag              :
Publisher                   : Microsoft.Compute
VirtualMachineExtensionType : CustomScriptExtension
TypeHandlerVersion          : 1.9
AutoUpgradeMinorVersion     : True
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>識別 autoUpgradeMinorVersion 於何時執行

若要查看擴充功能的更新是何時執行的，請在 *C:\WindowsAzure\Logs\WaAppAgent.log* 上檢閱 VM 的代理程式記錄

在下列範例中，VM 已安裝 *Microsoft.Compute.CustomScriptExtension 1.8*。 *1.9* 版有可用的 Hotfix：

```powershell
[INFO]  Getting plugin locations for plugin 'Microsoft.Compute.CustomScriptExtension'. Current Version: '1.8', Requested Version: '1.9'
[INFO]  Auto-Upgrade mode. Highest public version for plugin 'Microsoft.Compute.CustomScriptExtension' with requested version: '1.9', is: '1.9'
```

## <a name="agent-permissions"></a>代理程式權限

若要執行其工作，代理程式必須以*本機系統*的形式執行。

## <a name="troubleshoot-vm-extensions"></a>針對 VM 擴充功能進行疑難排解

每個 VM 擴充功能可能會有擴充功能特定的疑難排解步驟。 例如，當您使用「自訂指令碼」擴充功能時，指令碼執行詳細資料可以在執行擴充功能的虛擬機器本機上找到。 所有擴充功能特定疑難排解步驟都會在擴充功能特定文件中詳述。

下列疑難排解步驟適用於所有虛擬機器擴充功能。

1. 若要查看 Windows 客體代理程式記錄，請在 *C:\WindowsAzure\Logs\WaAppAgent.txt* 中檢視您佈建擴充功能時所執行的活動

2. 若要查看實際的擴充功能記錄，請在 *C:\WindowsAzure\Logs\Plugins\<extensionName>* 中取得詳細資訊。

3. 在擴充功能專用文件的疑難排解章節中，查看錯誤碼、已知問題等。

4. 查看系統記錄。 檢查是否有其他作業對擴充功能造成干擾，例如，長時間執行且需要以獨佔方式存取套件管理員的其他應用程式安裝。

### <a name="common-reasons-for-extension-failures"></a>擴充功能失敗的常見原因

1. 擴充功能可執行 20 分鐘 (除了 CustomScript 擴充功能、Chef 與 DSC 有 90 分鐘可執行以外)。 如果您的部署超過此時間，則會標示為逾時。 其成因可能是由於虛擬機器資源不足，而在擴充功能嘗試進行佈建時，有其他虛擬機器組態/啟動工作耗用了大量資源。

2. 不符合先決條件。 某些擴充功能具有對虛擬機器 SKU 的相依性，例如 HPC 映像。 擴充功能可能會有特定的網路存取需求，例如，須與 Azure 儲存體或公用服務進行通訊。 其他範例可能包括對套件存放庫的存取、磁碟空間不足或安全性限制等。

3. 獨佔套件管理員存取。 在某些情況下，可能會出現長時間執行的虛擬機器組態和擴充功能安裝相衝突的情形，因為兩者都需要以獨佔方式存取套件管理員。

### <a name="view-extension-status"></a>檢視擴充功能狀態

對 VM 執行 VM 擴充功能後，請使用 [Get-AzureRmVM ](/powershell/module/azurerm.compute/get-azurermvm) 傳回擴充功能狀態。 *Substatuses[0]* 顯示擴充功能佈建成功，這表示擴充功能已成功部署至 VM，但在 VM 內部執行失敗，而顯示 *Substatuses[1]*。

```powershell
Get-AzureRmVM -ResourceGroupName "myResourceGroup" -VMName "myVM" -Status
```

輸出會類似下列範例：

```powershell
Extensions[0]           :
  Name                  : CustomScriptExtension
  Type                  : Microsoft.Compute.CustomScriptExtension
  TypeHandlerVersion    : 1.9
  Substatuses[0]        :
    Code                : ComponentStatus/StdOut/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : Windows PowerShell \nCopyright (C) Microsoft Corporation. All rights reserved.\n
  Substatuses[1]        :
    Code                : ComponentStatus/StdErr/succeeded
    Level               : Info
    DisplayStatus       : Provisioning succeeded
    Message             : The argument 'cseTest%20Scriptparam1.ps1' to the -File parameter does not exist. Provide the path to an existing '.ps1' file as an argument to the

-File parameter.
  Statuses[0]           :
    Code                : ProvisioningState/failed/-196608
    Level               : Error
    DisplayStatus       : Provisioning failed
    Message             : Finished executing command
```

也可以在 Azure 入口網站中找到擴充功能的執行狀態。 若要檢視擴充功能的狀態，請選取虛擬機器，選擇 [擴充功能]，然後選取所需的擴充功能。

### <a name="rerun-vm-extensions"></a>執行 VM 擴充功能

可能會發生必須重新執行虛擬機器擴充功能的情況。 您可以藉由移除延伸模組，然後使用您所選的執行方法重新執行延伸模組，來重新執行延伸模組。 若要移除擴充功能，請使用 [Remove-AzureRmVMExtension](/powershell/module/AzureRM.Compute/Remove-AzureRmVMExtension)，如下所示：

```powershell
Remove-AzureRmVMExtension -ResourceGroupName "myResourceGroup" -VMName "myVM" -Name "myExtensionName"
```

您也可以在 Azure 入口網站中移除擴充功能，如下所示：

1. 選取 VM。
2. 選擇**擴充功能**。
3. 選取所需的擴充功能。
4. 選擇**解除安裝**。

## <a name="common-vm-extensions-reference"></a>常見的 VM 擴充功能參考
| 擴充功能名稱 | 說明 | 詳細資訊 |
| --- | --- | --- |
| Windows 的自訂指令碼延伸模組 |對「Azure 虛擬機器」執行指令碼 |[Windows 的自訂指令碼延伸模組](custom-script-windows.md) |
| Windows 的 DSC 延伸模組 |PowerShell DSC (預期狀態設定) 擴充功能 |[適用於 Windows 的 DSC 擴充功能](dsc-overview.md) |
| Azure 診斷擴充功能 |管理「Azure 診斷」 |[Azure 診斷擴充功能](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Azure VM 存取擴充功能 |管理使用者和認證 |[適用於 Linux 的 VM 存取擴充功能](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>後續步驟

如需關於虛擬機器擴充功能的詳細資訊，請參閱 [Azure 虛擬機器擴充功能和功能概觀](overview.md)。
