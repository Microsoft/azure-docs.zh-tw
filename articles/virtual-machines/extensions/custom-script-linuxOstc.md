---
title: 在 Azure 中的 Linux VM 上執行自訂指令碼 | Microsoft Docs
description: 使用自訂指令碼擴充功能 v1，將 Linux VM 設定工作自動化
services: virtual-machines-linux
documentationcenter: ''
author: danielsollondon
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: danis
ms.openlocfilehash: b88d850b708a10d0e0fdff2f54b68cb9b39988f5
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42140260"
---
# <a name="use-the-azure-custom-script-extension-version-1-with-linux-virtual-machines"></a>搭配 Linux 虛擬機器使用 Azure 自訂指令碼擴充功能第 1 版

[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]

自訂指令碼擴充功能第 1 版會在 Azure 虛擬機器上下載並執行指令碼。 此擴充功能適用於部署後設定、軟體安裝或其他任何設定/管理工作。 您可以從 Azure 儲存體或其他可存取的網際網路位置下載指令碼，或是將指令碼提供給擴充功能執行階段。

自訂指令碼功能可以與 Azure Resource Manager 範本整合。 您也可以使用 Azure CLI、PowerShell、Azure 入口網站或 Azure 虛擬機器 REST API 來執行它。

本文將詳細說明如何從 Azure CLI 使用自訂指令碼擴充功能，以及如何使用 Azure Resource Manager 範本來執行擴充功能。 本文也提供適用於 Linux 系統的疑難排解步驟。

Linux 自訂指令碼擴充功能有兩個：

* 第 1 版 - Microsoft.OSTCExtensions.CustomScriptForLinux

* 第 2 版 - Microsoft.Azure.Extensions.CustomScript

請切換新的和現有的部署，以改用新版本 ([Microsoft.Azure.Extensions.CustomScript](custom-script-linux.md))。 新版本可供簡易替換 (drop-in replacement)。 因此，移轉就像變更名稱和版本一樣簡單，而不需要變更您的擴充功能設定。

### <a name="operating-system"></a>作業系統

支援的 Linux 散發套件：

* CentOS 6.5 及更新版本
* Debian 8 及更新版本
  * Debian 8.7 並未與 Python2 一起隨附在最新的映像中，如此會中止 CustomScriptForLinux。
* FreeBSD
* OpenSUSE 13.1 及更新版本
* Oracle Linux 6.4 及更新版本
* SUSE Linux Enterprise Server 11 SP3 及更新版本
* Ubuntu 12.04 及更新版本

### <a name="script-location"></a>指令碼位置

您可以利用擴充功能來使用 Azure Blob 儲存體認證，以存取 Azure Blob 儲存體。 或者，指令碼可以位於任何位置，前提是 VM 可以路由傳送至該端點，例如 GitHub、內部檔案伺服器等。

### <a name="internet-connectivity"></a>網際網路連線

如果您需要在外部下載指令碼 (例如 GitHub 或 Azure 儲存體)，則必須開放額外的防火牆/網路安全性群組連接埠。 例如，如果您的指令碼位於 Azure 儲存體，則可以允許使用[儲存體](../../virtual-network/security-overview.md#service-tags)適用的 Azure NSG 服務標記進行存取。

如果您的指令碼是在本機伺服器上，則仍然可能需要開放額外的防火牆/網路安全性群組連接埠。

### <a name="tips-and-tricks"></a>秘訣與技巧

* 這個擴充功能的最常見失敗原因是指令碼語法錯誤。請測試指令碼執行時沒有錯誤，而且也在指令碼中放入額外記錄，以方便找出失敗之處。
* 撰寫具有等冪性的指令碼，因此即使意外執行多次，也不會造成系統變更。
* 請確定在指令碼執行時，不需要使用者輸入。
* 指令碼可執行的時間為 90 分鐘。若超過這個時間，將會導致擴充功能佈建失敗。
* 請不要在指令碼內放置重新開機指令，這會造成正在安裝的其他擴充功能發生問題。也不要放置後續重新開機指令，因為擴充功能在重新啟動後不會繼續執行。 
* 如果您的指令碼將會造成重新開機，則請安裝應用程式，然後執行指令碼等等。您應該使用 Cron 作業，或使用 DSC、Chef 或 Puppet 擴充功能之類的工具，以排程重新開機。
* 擴充功能只會執行指令碼一次。如果您想要在每次開機時執行指令碼，則可以使用 [cloud-init image](../linux/using-cloud-init.md)，並使用 [Scripts Per Boot](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#scripts-per-boot) 模組。 或者，您可以使用指令碼來建立 Systemd 服務單位。
* 如果您想要排程指令碼的執行時間，則應該使用擴充功能來建立 Cron 作業。
* 當指令碼正在執行時，只能從 Azure 入口網站或 CLI 看到「正在轉換」擴充功能狀態。 如果您需要執行中指令碼更頻繁的狀態更新，便必須建立自己的解決方案。
* 自訂指令碼擴充功能未原生支援 Proxy 伺服器，但是您可以在指令碼中使用支援 Proxy 伺服器的檔案傳輸工具，例如 *Curl*。
* 請留意指令碼或命令所依賴的非預設目錄位置是否具備處理此情形的邏輯。

## <a name="extension-schema"></a>擴充功能結構描述

「自訂指令碼擴充功能」組態會指定指令碼位置和要執行命令等項目。 您可將此組態儲存在組態檔中、在命令列中指定組態，或在 Azure Resource Manager 範本中指定組態。 

您可將敏感性資料儲存在受保護的組態中，此組態會經過加密，並且只會在虛擬機器內解密。 當執行命令包含機密資料 (例如密碼) 時，受保護的組態會相當有用。

這些項目應被視為敏感性資料，並在擴充功能保護的設定組態中指定。 Azure VM 擴充功能保護的設定資料會經過加密，只會在目標虛擬機器上解密。

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "<url>"
      ],
      "enableInternalDNSCheck": true
    },
    "protectedSettings": {
      "storageAccountName": "<storage-account-name>",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "<command>"
    }
  }
}
```

### <a name="property-values"></a>屬性值

| 名稱 | 值 / 範例 | 資料類型 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | 日期 |
| publisher | Microsoft.OSTCExtensions | 字串 |
| type | CustomScriptForLinux | 字串 |
| typeHandlerVersion | 1.5 | int |
| fileUris (例如) | https://github.com/MyProject/Archive/MyPythonScript.py | array |
| commandToExecute (例如) | python MyPythonScript.py \<my-param1\> | 字串 |
| enableInternalDNSCheck | true | 布林值 |
| storageAccountName (例如) | examplestorageacct | 字串 |
| storageAccountKey (例如) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | 字串 |

### <a name="property-value-details"></a>屬性值詳細資料

* `fileUris`：(選用，字串陣列) 指令碼的 URI 清單
* `enableInternalDNSCheck`：(選用，布林值) 預設為 True，設定為 False 時可停用 DNS 檢查。
* `commandToExecute`：(選用，字串) 要執行的進入點指令碼
* `storageAccountName`：(選用，字串) 儲存體帳戶的名稱
* `storageAccountKey`：(選用，字串) 儲存體帳戶的存取金鑰

下列值可以在公開或受保護設定兩者之一中設定，但不得同時在公開和受保護設定中設定下列這些值。

* `commandToExecute`

使用公開設定可能有助於偵錯，但強烈建議您使用受保護的設定。

公開設定會以純文字的格式，傳送到執行指令碼所在的 VM。  受保護的設定會使用只有 Azure 和 VM 知道的金鑰來加密。 這些設定會在傳送時儲存到 VM 中。亦即，如果這些設定經過加密，也會在 VM 上以加密的形式儲存。 用來解密加密值的憑證會儲存在 VM 上，並在執行階段用於解密設定 (如有必要)。

## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 上一節詳述的 JSON 結構描述可以用於 Azure Resource Manager 範本，以在 Azure Resource Manager 範本部署期間執行自訂指令碼擴充功能。

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": ["http://MyAccount.blob.core.windows.net/vhds/MyShellScript.sh"]
    },
    "protectedSettings": {
      "storageAccountName": "MyAccount",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "sh MyShellScript.sh"
    }
  }
}
```

>[!NOTE]
>這些屬性名稱會區分大小寫。 為了避免發生部署問題，請使用如下所示的名稱。

## <a name="azure-cli"></a>Azure CLI

當您要使用 Azure CLI 來執行自訂指令碼擴充功能時，請建立組態檔。 您至少必須擁有 'commandToExecute'。

```azurecli
az vm extension set -n VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.5 \
  --vm-name MyVm --resource-group MyResourceGroup \
  --protected-settings '{"commandToExecute": "echo hello"}'
```

您可以選擇在命令中以 JSON 格式化字串的形式指定設定。 這可讓您在執行期間指定組態，而不需使用個別的組態檔。

```azurecli
az vm extension set \
  --resource-group exttest \
  --vm-name exttest \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings '{"fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],"commandToExecute": "./config-music.sh"}'
```

### <a name="azure-cli-examples"></a>Azure CLI 範例

#### <a name="public-configuration-with-no-script-file"></a>不含指令檔的公用組態

```json
{
  "commandToExecute": "apt-get -y update && apt-get install -y apache2"
}
```

Azure CLI 命令：

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json
```

#### <a name="public-and-protected-configuration-files"></a>公用且受保護的組態檔

您可以使用公用組態檔來指定指令碼檔案 URI。 您可以使用受保護的組態檔來指定要執行的命令。

公用組態檔：

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"]
}
```

受保護的組態檔：  

```json
{
  "commandToExecute": "./config-music.sh <param1>"
}
```

Azure CLI 命令：

```azurecli
az vm extension set
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json \
  --protected-settings ./protected-config.json
```

## <a name="troubleshooting"></a>疑難排解

當「自訂指令碼擴充功能」執行時，會建立指令碼，或將指令碼下載到類似下列範例的目錄。 命令輸出也會儲存到這個目錄的 `stdout` 和 `stderr` 檔案中。

```bash
/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-<version>/download/1
```

若要進行疑難排解，請先檢查 Linux 代理程式記錄檔，確認擴充功能已執行，然後檢查：

```bash
/var/log/waagent.log
```

您應該尋找如下所示的擴充功能執行：

```text
2018/04/26 15:29:44.835067 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Target handler state: enabled
2018/04/26 15:29:44.867625 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] [Enable] current handler state is: notinstalled
2018/04/26 15:29:44.959605 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Download, message=Download succeeded, duration=59
2018/04/26 15:29:44.993269 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Initialize extension directory
2018/04/26 15:29:45.022972 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Update settings file: 0.settings
2018/04/26 15:29:45.051763 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Install extension [customscript.py -install]
2018/04/26 15:29:45 CustomScriptForLinux started to handle.
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] cwd is /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Change log file to /var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log
2018/04/26 15:29:46.088212 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Install, message=Launch command succeeded: customscript.py -install, duration=1005
2018/04/26 15:29:46.133367 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable extension [customscript.py -enable]
2018/04/26 15:29:46 CustomScriptForLinux started to handle.
..
2018/04/26 15:29:47.178163 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Enable, message=Launch command succeeded: customscript.py -enable, duration=1012
```

注意事項：

1. 命令開始執行時啟用。
1. 下載與從 Azure 下載 CustomScript 擴充功能套件有關，與在 fileUris 中指定的指令碼檔案無關。
1. 您也可以查看它寫出哪個記錄檔 `/var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log
`

下一步是要檢查記錄檔，以下是格式：

```bash
/var/log/azure/<extension-name>/<version>/extension.log file.
```

您應該尋找如下所示的個別執行：

```text
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable,transitioning,0,Launching the script...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] sequence number is 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] setting file path is/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/config/0.settings
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] JSON config: {"runtimeSettings": [{"handlerSettings": {"protectedSettings": "MIIB0AYJKoZIhvcNAQcDoIIBwTCCAb0CAQAxggF+hnEXRtFKTTuKiFC8gTfHKupUSs7qI0zFYRya", "publicSettings": {"fileUris": ["https://dannytesting.blob.core.windows.net/demo/myBash.sh"]}, "protectedSettingsCertThumbprint": "4385AB21617C2452FF6998C0A37F71A0A01C8368"}}]}
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Config decoded correctly.
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Will try to download files, number of retries = 10, wait SECONDS between retrievals = 20s
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Downloading,transitioning,0,Downloading files...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] No azure storage account and key specified in protected settings. Downloading scripts from external links...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Converting /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh from DOS to Unix formats: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Removing BOM of /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Succeeded to download files, retry count = 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Internal DNS is ready, retry count = 0
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Daemon,success,0,Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
```

您可以在此發現：

* 啟動「啟用」命令的是這個記錄檔
* 傳遞至擴充功能的設定
* 擴充功能下載檔案及其結果。
* 正在執行的命令和結果。

您也可以使用 Azure CLI 來擷取「自訂指令碼擴充功能」的執行狀態：

```azurecli
az vm extension list -g myResourceGroup --vm-name myVM
```

輸出看起來會像下列文字：

```azurecli
Name                  ProvisioningState    Publisher                   Version  AutoUpgradeMinorVersion
--------------------  -------------------  ------------------------  ---------  -------------------------
CustomScriptForLinux  Succeeded            Microsoft.OSTCExtensions        1.5  True
```

## <a name="next-steps"></a>後續步驟

若要查看程式碼、目前問題和版本，請參閱 [CustomScript 擴充功能存放庫](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript)。
