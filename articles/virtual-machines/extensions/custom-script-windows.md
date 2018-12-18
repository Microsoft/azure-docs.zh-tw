---
title: Windows 的 Azure 自訂指令碼延伸模組 | Microsoft Docs
description: 使用「自訂指令碼擴充功能」來自動執行 Windows VM 組態工作
services: virtual-machines-windows
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: f4181fee-7a9d-4a1c-b517-52956f5b7fa1
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/24/2018
ms.author: roiyz
ms.openlocfilehash: 1201e7ec232a5bd45351072949dc0b9c19af434f
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452168"
---
# <a name="custom-script-extension-for-windows"></a>Windows 的自訂指令碼延伸模組

「自訂指令碼擴充功能」會在 Azure 虛擬機器上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。 「自訂指令碼擴充功能」會與 Azure Resource Manager 範本整合，您也可以使用 Azure CLI、PowerShell、Azure 入口網站或「Azure 虛擬機器 REST API」來執行它。

本文件詳細說明如何透過 Azure PowerShell 模組、Azure Resource Manager 範本使用自訂指令碼擴充功能，同時也詳細說明 Windows 系統上的疑難排解步驟。

## <a name="prerequisites"></a>必要條件

> [!NOTE]  
> 請勿使用自訂指令碼擴充功能以相同的 VM 執行 Update-AzureRmVM 作為其參數，因為它會等候其本身。  
>   
> 

### <a name="operating-system"></a>作業系統

Linux 的自訂指令碼擴充功能將在擴充功能支援的擴充功能 OS 上執行。如需詳細資訊，請參閱這篇[文章](https://support.microsoft.com/en-us/help/4078134/azure-extension-supported-operating-systems)。

### <a name="script-location"></a>指令碼位置

您可以利用擴充功能來使用 Azure Blob 儲存體認證，以存取 Azure Blob 儲存體。 或者，指令碼可以位於任何位置，前提是 VM 可以路由傳送至該端點，例如 GitHub、內部檔案伺服器等。


### <a name="internet-connectivity"></a>網際網路連線
如果您需要在外部下載指令碼 (例如 GitHub 或 Azure 儲存體)，則必須開放額外的防火牆/網路安全性群組連接埠。 例如，如果您的指令碼位於 Azure 儲存體，則可以允許使用[儲存體](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)適用的 Azure NSG 服務標記進行存取。

如果您的指令碼是在本機伺服器上，則仍然可能需要開放額外的防火牆/網路安全性群組連接埠。

### <a name="tips-and-tricks"></a>秘訣與技巧
* 這個擴充功能的最常見失敗原因是指令碼語法錯誤。請測試指令碼執行時沒有錯誤，而且也在指令碼中放入額外記錄，以方便找出失敗之處。
* 撰寫具有等冪性的指令碼，因此即使意外執行多次，也不會造成系統變更。
* 請確定在指令碼執行時，不需要使用者輸入。
* 指令碼可執行的時間為 90 分鐘。若超過這個時間，將會導致擴充功能佈建失敗。
* 請不要在指令碼內放置重新開機指令，這會造成正在安裝的其他擴充功能發生問題。也不要放置後續重新開機指令，因為擴充功能在重新啟動後不會繼續執行。 
* 如果您的指令碼將會造成重新開機，則請安裝應用程式，然後執行指令碼等等。您應該使用 Windows 排程工作，或使用 DSC、Chef 或 Puppet 擴充功能之類的工具，來排程重新開機。
* 擴充功能只會執行指令碼一次。如果您想要在每次開機時執行指令碼，則必須使用擴充功能建立 Windows 排程工作。
* 如果您想要排程指令碼的執行時間，則應該使用擴充功能來建立 Windows 排程工作。 
* 當指令碼正在執行時，只能從 Azure 入口網站或 CLI 看到「正在轉換」擴充功能狀態。 如果您需要執行中指令碼更頻繁的狀態更新，便必須建立自己的解決方案。
* 自訂指令碼擴充功能未原生支援 Proxy 伺服器，但是您可以在指令碼中使用支援 Proxy 伺服器的檔案傳輸工具，例如 *Curl* 
* 請留意指令碼或命令所依賴的非預設目錄位置是否具備處理此情形的邏輯。


## <a name="extension-schema"></a>擴充功能結構描述

「自訂指令碼擴充功能」組態會指定指令碼位置和要執行命令等項目。 您可將此組態儲存在組態檔中、在命令列中指定組態，或在 Azure Resource Manager 範本中指定組態。 

您可將敏感性資料儲存在受保護的組態中，此組態會經過加密，並且只會在虛擬機器內解密。 當執行命令包含機密資料 (例如密碼) 時，受保護的組態會相當有用。

這些項目應被視為敏感性資料，並在擴充功能保護的設定組態中指定。 Azure VM 擴充功能保護的設定資料會經過加密，只會在目標虛擬機器上解密。

```json
{
    "apiVersion": "2015-06-15",
    "type": "Microsoft.Compute/virtualMachines/extensions",
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
                "script location"
            ]
        },
        "protectedSettings": {
            "commandToExecute": "myExecutionCommand",
            "storageAccountName": "myStorageAccountName",
            "storageAccountKey": "myStorageAccountKey"
        }
    }
}
```
**請注意** - 一次只能在 VM 上安裝一個版本的擴充功能，若為相同的 VM 指定相同 Resource Manager 範本中的自訂指令碼兩次，作業將會失敗。 

### <a name="property-values"></a>屬性值

| 名稱 | 值 / 範例 | 資料類型 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | 日期 |
| publisher | Microsoft.Compute | 字串 |
| type | CustomScriptExtension | 字串 |
| typeHandlerVersion | 1.9 | int |
| fileUris (例如) | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1 | array |
| commandToExecute (例如) | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 | 字串 |
| storageAccountName (例如) | examplestorageacct | 字串 |
| storageAccountKey (例如) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | 字串 |

>[!NOTE]
>這些屬性名稱會區分大小寫。 為了避免發生部署問題，請使用如下所示的名稱。

#### <a name="property-value-details"></a>屬性值詳細資料
 * `commandToExecute`：(**必要**，字串) 要執行的進入點指令碼。 如果您的命令包含機密資料 (例如密碼)，或您的 fileUris 是敏感資料，請改用此欄位。
* `fileUris`：(選擇性，字串陣列) 要下載之檔案的 URL。
* `storageAccountName`：(選用，字串) 儲存體帳戶的名稱。 如果您指定儲存體證明資料，則所有 `fileUris` 都必須是 Azure Blob 的 URL。
* `storageAccountKey`：(選用，字串) 儲存體帳戶的存取金鑰

下列值可以在公開或受保護的設定中設定，擴充功能將會拒絕任何同時在公開和受保護的設定中設定下列值的組態。
* `commandToExecute`

使用公開設定可能有助於偵錯，但強烈建議您使用受保護的設定。

公開設定會以純文字的格式，傳送到執行指令碼所在的 VM。  受保護的設定會使用只有 Azure 和 VM 知道的金鑰來加密。 這些設定會在傳送時儲存到 VM 中。亦即，如果這些設定經過加密，也會在 VM 上以加密的形式儲存。 用來解密加密值的憑證會儲存在 VM 上，並在執行階段用於解密設定 (如有必要)。

## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 上一節詳述的 JSON 結構描述可以用於 Azure Resource Manager 範本，以在 Azure Resource Manager 範本部署期間執行自訂指令碼擴充功能。 在 [GitHub](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows) 可以找到包含自訂指令碼擴充功能的範例範本。

## <a name="powershell-deployment"></a>PowerShell 部署

`Set-AzureRmVMCustomScriptExtension` 命令可以用來將自訂指令碼擴充功能新增至現有的虛擬機器。 如需詳細資訊，請參閱 [Set-AzureRmVMCustomScriptExtension](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmcustomscriptextension)。

```powershell
Set-AzureRmVMCustomScriptExtension -ResourceGroupName myResourceGroup `
    -VMName myVM `
    -Location myLocation `
    -FileUri myURL `
    -Run 'myScript.ps1' `
    -Name DemoScriptExtension
```
## <a name="further-examples"></a>深入範例

### <a name="using-multiple-script"></a>使用多個指令碼
在此範例中，您有三個用來建置伺服器的指令碼，'commandToExecute' 會呼叫第一個指令碼，接著，您將可選擇呼叫其他指令碼的方式，例如，您可以使用能夠進行正確的錯誤處理、記錄和狀態管理的主要指令碼來控制執行方式。

```powershell

$fileUri = @("https://xxxxxxx.blob.core.windows.net/buildServer1/1_Add_Tools.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/2_Add_Features.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/3_CompleteInstall.ps1")

$Settings = @{"fileUris" = $fileUri};

$storageaccname = "xxxxxxx"
$storagekey = "1234ABCD"
$ProtectedSettings = @{"storageAccountName" = $storageaccname; "storageAccountKey" = $storagekey; "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File 1_Add_Tools.ps1"};

#run command
Set-AzureRmVMExtension -ResourceGroupName myRG `
    -Location myLocation ` 
    -VMName myVM ` 
    -Name "buildserver1" ` 
    -Publisher "Microsoft.Compute" ` 
    -ExtensionType "CustomScriptExtension" ` 
    -TypeHandlerVersion "1.9" ` 
    -Settings $Settings ` 
    -ProtectedSettings $ProtectedSettings `
```

### <a name="running-scripts-from-a-local-share"></a>從本機共用執行指令碼
在此範例中，您可以使用本機 SMB 伺服器作為指令碼位置。請注意，除了 *commandToExecute* 以外，您不需要傳入其他設定。

```powershell
$ProtectedSettings = @{"commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File \\filesvr\build\serverUpdate1.ps1"};
 
Set-AzureRmVMExtension -ResourceGroupName myRG 
    -Location myLocation ` 
    -VMName myVM ` 
    -Name "serverUpdate" 
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" ` 
    -TypeHandlerVersion "1.9" `
    -ProtectedSettings $ProtectedSettings

```

### <a name="how-to-run-custom-script-more-than-once-with-cli"></a>如何使用 CLI 多次執行自訂指令碼
如果您想要執行自訂指令碼延伸模組多次，您只有在下列情況下才能這麼做：
1. 擴充功能 'Name' 參數等同於先前的擴充功能部署。
2. 您必須已更新組態，否則命令將無法重新執行，例如，您可以將動態屬性新增至命令中，例如時間戳記。 

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure PowerShell 模組，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請執行下列命令：

```powershell
Get-AzureRmVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

擴充功能執行輸出會記錄至目標虛擬機器上下列目錄中的檔案。
```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

指定檔案會下載至目標虛擬機器上的下列目錄之中。
```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>
```
其中 `<n>` 是十進位整數，可能會在擴充功能的執行之間變更。  `1.*` 值符合擴充功能目前實際的 `typeHandlerVersion` 值。  例如，實際的目錄可能是 `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2`。  

執行 `commandToExecute` 命令時，擴充功能會將此目錄 (例如 `...\Downloads\2`) 設定為目前的工作目錄。 這可讓您使用相對路徑找出透過 `fileURIs` 屬性下載的檔案位置。 如需範例，請參閱下表。

由於絕對下載路徑會隨時間而改變，最好盡可能在 `commandToExecute` 字串中選擇相對的指令碼/檔案路徑。 例如︰
```json
    "commandToExecute": "powershell.exe . . . -File \"./scripts/myscript.ps1\""
```

第一個 URI 區段後的路徑資訊會為透過 `fileUris` 屬性清單下載的檔案而保留。  如下表所示，下載的檔案會對應到下載的子目錄，以反映 `fileUris` 值結構。  

#### <a name="examples-of-downloaded-files"></a>下載檔案的範例

| fileUris 中的 URI | 相對下載位置 | 絕對下載位置 * |
| ---- | ------- |:--- |
| `https://someAcct.blob.core.windows.net/aContainer/scripts/myscript.ps1` | `./scripts/myscript.ps1` |`C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\scripts\myscript.ps1`  |
| `https://someAcct.blob.core.windows.net/aContainer/topLevel.ps1` | `./topLevel.ps1` | `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\topLevel.ps1` |

\* 如同上述，絕對目錄路徑會隨 VM 生命週期而變更，但不會在 CustomScript 擴充的單次執行期間。

### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
