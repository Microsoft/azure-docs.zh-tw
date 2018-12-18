---
title: 適用於 Linux 的 Azure DSC 擴充功能
description: 安裝 OMI 和 DSC 套件，以允許使用「期望的狀態設定」來設定 Azure Linux VM。
services: virtual-machines-linux
documentationcenter: ''
author: bobbytreed
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 06/12/2018
ms.author: robreed
ms.openlocfilehash: 70280676453bd146102ca331daae038b947aab58
ms.sourcegitcommit: ab9514485569ce511f2a93260ef71c56d7633343
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2018
ms.locfileid: "45632852"
---
# <a name="dsc-extension-for-linux-microsoftostcextensionsdscforlinux"></a>適用於 Linux 的 DSC 擴充功能 (Microsoft.OSTCExtensions.DSCForLinux)

## <a name="overview"></a>概觀

期望的狀態設定 (DSC) 是一個管理平台，可讓您以組態作為程式碼來管理 IT 和開發基礎結構。

DSCForLinux 擴充功能由 Microsoft 所發佈及支援。 此擴充功能可在 Azure 虛擬機器上安裝 OMI 和 DSC 代理程式。 DSC 擴充功能也可執行下列動作


- 將 Linux VM 註冊至 Azure 自動化帳戶，以從 Azure 自動化服務中提取組態 (註冊 ExtensionAction)
- 將 MOF 組態推送至 Linux VM (推送 ExtensionAction)
- 將中繼 MOF 組態套用至 Linux VM 以設定提取伺服器，進而提取節點組態 (提取 ExtensionAction)
- 將自訂 DSC 模組安裝至 Linux VM (安裝 ExtensionAction)
- 移除 Linux VM 的自訂 DSC 模組 (移除 ExtensionAction)

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

DSC Linux 擴充功能支援所有[支援擴充功能的作業系統](https://azurewiki.cloudapp.netVMAgentExtension/extensionSupportedOSs)，但下列項目除外：

| 配送映像 | 版本 |
|---|---|
| CentOS Linux | 6.5 及更新版本 |
| Ubuntu| 12.04 LTS、14.04 LTS、16.04 LTS  |
| RHEL| 6.5 及更新版本  |
| openSUSE| 13.1 及更新版本  |
| SUSE Linux Enterprise Server| 11 SP3 及更新版本  |

  
 
### <a name="internet-connectivity"></a>網際網路連線

要使用 DSCForLinux 擴充功能，必須將目標虛擬機器連線至網際網路。 例如，註冊擴充功能需要連線至自動化服務。 其他動作 (例如推送、提取、安裝) 則需要連線至 Azure 儲存體/Github。 這取決於客戶所提供的設定。

## <a name="extension-schema"></a>擴充功能結構描述

### <a name="11-public-configuration"></a>1.1 公用組態

以下是所有受支援的公用組態參數：

* `FileUri`：(選擇性，字串) MOF 檔案/中繼 MOF 檔案/自訂資源 ZIP 檔案的 URI。
* `ResourceName`：(選擇性，字串) 自訂資源模組的名稱
* `ExtensionAction`：(選擇性，字串) 指定擴充功能的作用。 有效值：註冊、推送、提取、安裝、移除。 若未指定，依預設會將其視為推送動作。
* `NodeConfigurationName`：(選擇性，字串) 要套用的節點組態名稱。
* `RefreshFrequencyMins`：(選擇性，int) 指定 DSC 嘗試從提取伺服器取得組態的頻率 (以分鐘為單位)。 
       如果提取伺服器上的組態不同於目標節點上目前的組態，則會將該組態複製到擱置存放區並套用。
* `ConfigurationMode`：(選擇性，字串) 指定 DSC 套用組態的方式。 有效的值為：ApplyOnly、ApplyAndMonitor、ApplyAndAutoCorrect。
* `ConfigurationModeFrequencyMins`：(選擇性，int) 指定 DSC 確認組態處於適當狀態的頻率 (以分鐘為單位)。

> [!NOTE]
> 如果您使用低於 2.3 的版本，則模式參數會與 ExtensionAction 相同。 模式一詞似乎有太多涵義。 因此，為了避免混淆，自 2.3 版起均使用 ExtensionAction。 為了顧及回溯相容性，此擴充功能同時支援模式和 ExtensionAction。 
>

### <a name="12-protected-configuration"></a>1.2 受保護的組態

以下是所有受支援和保護的組態參數：

* `StorageAccountName`：(選擇性，字串)：檔案所屬儲存體帳戶的名稱
* `StorageAccountKey`：(選擇性，字串)：檔案所屬儲存體帳戶的金鑰
* `RegistrationUrl`：(選擇性，字串) Azure 自動化帳戶的 URL
* `RegistrationKey`：(選擇性，字串) Azure 自動化帳戶的存取金鑰


## <a name="scenarios"></a>案例

### <a name="register-to-azure-automation-account"></a>註冊到 Azure 自動化帳戶
protected.json
```json
{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}
```
public.json
```json
{
  "ExtensionAction" : "Register",
  "NodeConfigurationName" : "<node-configuration-name>",
  "RefreshFrequencyMins" : "<value>",
  "ConfigurationMode" : "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins" : "<value>"
}
```

Powershell 格式
```powershell
$privateConfig = '{
  "RegistrationUrl": "<azure-automation-account-url>",
  "RegistrationKey": "<azure-automation-account-key>"
}'

$publicConfig = '{
  "ExtensionAction" : "Register",
  "NodeConfigurationName": "<node-configuration-name>",
  "RefreshFrequencyMins": "<value>",
  "ConfigurationMode": "<ApplyAndMonitor | ApplyAndAutoCorrect | ApplyOnly>",
  "ConfigurationModeFrequencyMins": "<value>"
}'
```

### <a name="apply-a-mof-configuration-file-in-azure-storage-account-to-the-vm"></a>將 MOF 組態檔 (位於 Azure 儲存體帳戶中) 套用至 VM

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}
```

Powershell 格式
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "FileUri": "<mof-file-uri>",
  "ExtensionAction": "Push"
}'
```


### <a name="apply-a-mof-configuration-file-in-public-storage-to-the-vm"></a>將 MOF 組態檔 (位於公用儲存體中) 套用至 VM

public.json
```json
{
  "FileUri": "<mof-file-uri>"
}
```

Powershell 格式
```powershell
$publicConfig = '{
  "FileUri": "<mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-azure-storage-account-to-the-vm"></a>將中繼 MOF 組態檔 (位於 Azure 儲存體帳戶中) 套用至 VM

protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```

public.json
```json
{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}
```

Powershell 格式
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Pull",
  "FileUri": "<meta-mof-file-uri>"
}'
```

### <a name="apply-a-meta-mof-configuration-file-in-public-storage-to-the-vm"></a>將中繼 MOF 組態檔 (位於公用儲存體中) 套用至 VM
public.json
```json
{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}
```
Powershell 格式
```powershell
$publicConfig = '{
  "FileUri": "<meta-mof-file-uri>",
  "ExtensionAction": "Pull"
}'
```

### <a name="install-a-custom-resource-module-zip-file-in-azure-storage-account-to-the-vm"></a>將自訂資源模組 (Azure 儲存體帳戶中的 ZIP 檔案) 安裝到 VM
protected.json
```json
{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}
```
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```

Powershell 格式
```powershell
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'

$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="install-a-custom-resource-module-zip-file-in-public-storage-to-the-vm"></a>將自訂資源模組 (公用儲存體中的 ZIP 檔案) 安裝到 VM
public.json
```json
{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}
```
Powershell 格式
```powershell
$publicConfig = '{
  "ExtensionAction": "Install",
  "FileUri": "<resource-zip-file-uri>"
}'
```

### <a name="remove-a-custom-resource-module-from-the-vm"></a>從 VM 中移除自訂資源模組
public.json
```json
{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}
```
Powershell 格式
```powershell
$publicConfig = '{
  "ResourceName": "<resource-name>",
  "ExtensionAction": "Remove"
}'
```

## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 部署一或多部需要部署後設定的虛擬機器時 (例如上架至 Azure 自動化)，很適合使用範本。 

範例 Resource Manager 範本為 [201-dsc-linux-azure-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-azure-storage-on-ubuntu) 和 [201-dsc-linux-public-storage-on-ubuntu](https://github.com/Azure/azure-quickstart-templates/tree/master/201-dsc-linux-public-storage-on-ubuntu)。

如需關於 Azure Resource Manager 範本的詳細資訊，請參閱[編寫 Azure Resource Manager 範本](../../azure-resource-manager/resource-group-authoring-templates.md)。


## <a name="azure-cli-deployment"></a>Azure CLI 部署

### <a name="21-using-azure-cliazure-cli"></a>2.1. 使用 [**Azure CLI**][azure-cli]
在部署 DSCForLinux 擴充功能之前，您應該先根據第 3 節中的不同案例設定 `public.json` 和 `protected.json`。

#### <a name="211-classic"></a>2.1.1. 傳統
傳統模式也稱為 Azure 服務管理模式。 您可以執行下列命令以切換至該模式：
```
$ azure config mode asm
```

您可以執行下列命令以部署 DSCForLinux 擴充功能：
```
$ azure vm extension set <vm-name> DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```

若要了解最新的可用擴充功能版本，請執行：
```
$ azure vm extension list
```

#### <a name="212-resource-manager"></a>2.1.2. Resource Manager
您可以執行下列命令，以切換至 Azure Resource Manager 模式：
```
$ azure config mode arm
```

您可以執行下列命令以部署 DSCForLinux 擴充功能：
```
$ azure vm extension set <resource-group> <vm-name> \
DSCForLinux Microsoft.OSTCExtensions <version> \
--private-config-path protected.json --public-config-path public.json
```
> [!NOTE]
> 在 Azure Resource Manager 模式中，`azure vm extension list` 目前無法使用。
>

### <a name="22-using-azure-powershellazure-powershell"></a>2.2. 使用 [**Azure PowerShell**][azure-powershell]

#### <a name="221-classic"></a>2.2.1 傳統

您可以執行下列命令，以登入您的 Azure 帳戶 (Azure Service Management 模式)：

```powershell>
Add-AzureAccount
```

此外，執行下列命令以部署 DSCForLinux 擴充功能：

```powershell>
$vmname = '<vm-name>'
$vm = Get-AzureVM -ServiceName $vmname -Name $vmname
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

您必須根據上一節中的不同案例變更 $privateConfig 和 $publicConfig 的內容 
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzureVMExtension -ExtensionName $extensionName -VM $vm -Publisher $publisher `
  -Version $version -PrivateConfiguration $privateConfig `
  -PublicConfiguration $publicConfig | Update-AzureVM
```

#### <a name="222resource-manager"></a>2.2.2. 資源管理員

您可以執行下列命令，以登入您的 Azure 帳戶 (Azure Resource Manager 模式)：

```powershell>
Login-AzureRmAccount
```

按一下[**這裡**](../../azure-resource-manager/powershell-azure-resource-manager.md)，以深入了解如何搭配使用 Azure PowerShell 與 Azure Resource Manager。

您可以執行下列命令以部署 DSCForLinux 擴充功能：

```powershell>
$rgName = '<resource-group-name>'
$vmName = '<vm-name>'
$location = '< location>'
$extensionName = 'DSCForLinux'
$publisher = 'Microsoft.OSTCExtensions'
$version = '< version>'
```

您必須根據上一節中的不同案例變更 $privateConfig 和 $publicConfig 的內容 
```
$privateConfig = '{
  "StorageAccountName": "<storage-account-name>",
  "StorageAccountKey": "<storage-account-key>"
}'
```

```
$publicConfig = '{
  "ExtensionAction": "Push",
  "FileUri": "<mof-file-uri>"
}'
```

```
Set-AzureRmVMExtension -ResourceGroupName $rgName -VMName $vmName -Location $location `
  -Name $extensionName -Publisher $publisher -ExtensionType $extensionName `
  -TypeHandlerVersion $version -SettingString $publicConfig -ProtectedSettingString $privateConfig
```

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure CLI，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請使用 Azure CLI 執行下列命令。

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

擴充功能執行輸出會記錄至下列檔案︰

```
/var/log/azure/<extension-name>/<version>/extension.log file.
```

錯誤碼：51 代表不支援的發行版本或不支援的擴充功能動作。
在某些情況下，如果電腦上已有較高版本的 OMI 存在，DSC Linux 擴充功能會無法安裝 OMI。 [錯誤回應：(000003) 不允許降級]



### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/community/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。

## <a name="next-steps"></a>後續步驟
如需擴充功能的詳細資訊，請參閱[虛擬機器擴充功能和 Linux 功能](features-linux.md)。
