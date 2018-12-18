---
title: Stackify Retrace Azure Linux 代理程式擴充功能 | Microsoft Docs
description: 將 Stackify Retrace Linux 代理程式部署在 Linux 虛擬機器上。
services: virtual-machines-linux
documentationcenter: ''
author: darinhoward
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 04/12/2018
ms.author: roiyz
ms.openlocfilehash: b286ebc2e50166e8491b45346a81b161227f8d21
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39415961"
---
# <a name="stackify-retrace-linux-agent-extension"></a>Stackify Retrace Linux 代理程式擴充功能

## <a name="overview"></a>概觀
Stackify 提供可追蹤您應用程式詳細資料的產品，幫助您快速找到問題並修正。 對於開發人員小組而言，Retrace 是適用於多重環境、完全整合且效能強大的應用程式， 結合了每個開發小組都需要的多種工具。

Retrace 是唯一能以單一平台中在所有環境中提供下列功能的工具。

* 應用程式效能管理 (APM)
* 應用程式和伺服器記錄
* 錯誤追蹤和監視
* 伺服器、應用程式及自訂計量

**關於 Stackify Linux 代理程式擴充功能**

此擴充功能提供 Linux Agent for Retrace 的安裝路徑。 

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統 
可以針對這些 Linux 散發套件執行 Retrace 代理程式

| 配送映像 | 版本 |
|---|---|
| Ubuntu | 16.04 LTS、14.04 LTS、16.10 及 17.04 |
| Debian | 7.9+ 和 8.2+、9 |
| Red Hat | 6.7+、7.1+ |
| CentOS | 6.3+、7.0+ |

### <a name="internet-connectivity"></a>網際網路連線
適用於 Linux 的 Stackify 代理程式擴充功能會要求目標虛擬機器連接到網際網路。 

您可能需要將網路設定調整為允許連線至 Stackify，請參閱 https://support.stackify.com/hc/en-us/articles/207891903-Adding-Exceptions-to-a-Firewall (英文)。 


## <a name="extension-schema"></a>擴充功能結構描述
---

下列 JSON 顯示 Stackify Retrace 代理程式擴充功能的結構描述。 擴充功能需要 `environment` 和 `activationKey`。

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

## <a name="template-deployment"></a>範本部署 

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 上一節中詳述的 JSON 結構描述可在部署 Azure Resource Manager 範本時，在 Azure Resource Manager 範本中用來執行 Stackify Retrace Linux 代理程式擴充功能。  

虛擬機器擴充功能的 JSON 可以巢狀方式置於虛擬機器資源內部，或放在 Resource Manager JSON 範本的根目錄或最上層。 JSON 的放置會影響資源名稱和類型的值。 如需詳細資訊，請參閱「設定子資源的名稱和類型」。

下列範例假設 Stackify Retrace Linux 擴充功能以巢狀方式置於虛擬機器資源內部。 在巢狀處理擴充資源時，JSON 會放在虛擬機器的 "resources": [] 物件中。

擴充功能需要 `environment` 和 `activationKey`。

```json
    {
      "type": "extensions",
      "name": "StackifyExtension",
      "apiVersion": "[variables('apiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachines',variables('vmName'))]"
      ],
      "properties": {
        "publisher": "Stackify.LinuxAgent.Extension",
        "type": "StackifyLinuxAgentExtension",
        "typeHandlerVersion": "1.0",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "environment": "myEnvironment"
        },
        "protectedSettings": {
          "activationKey": "myActivationKey"
        }
      }
    }      
```

將擴充 JSON 置於範本的根目錄時，資源名稱包含父系虛擬機器的參考，而類型可反映巢狀的組態。

```json
    {
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "<parentVmResource>/StackifyExtension",
        "apiVersion": "[variables('apiVersion')]",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "properties": {
            "publisher": "Stackify.LinuxAgent.Extension",
            "type": "StackifyLinuxAgentExtension",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "environment": "myEnvironment"
            },
            "protectedSettings": {
              "activationKey": "myActivationKey"
            }
        }
    }
```


## <a name="powershell-deployment"></a>PowerShell 部署

`Set-AzureRmVMExtension` 命令可以用來將 Stackify Retrace Linux 代理程式虛擬機器擴充功能部署到現有的虛擬機器。 執行命令之前，必須將公用和私人組態儲存在 PowerShell 雜湊表中。

擴充功能需要 `environment` 和 `activationKey`。

```
$PublicSettings = @{"environment" = "myEnvironment"}
$ProtectedSettings = @{"activationKey" = "myActivationKey"}

Set-AzureRmVMExtension -ExtensionName "Stackify.LinuxAgent.Extension" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Stackify.LinuxAgent.Extension" `
    -ExtensionType "StackifyLinuxAgentExtension" `
    -TypeHandlerVersion 1.0 `
    -Settings $PublicSettings `
    -ProtectedSettings $ProtectedSettings `
    -Location WestUS `
```

## <a name="azure-cli-deployment"></a>Azure CLI 部署 

Azure CLI 工具可以用來將 Stackify Retrace Linux 代理程式虛擬機器擴充功能部署到現有的虛擬機器。  

擴充功能需要 `environment` 和 `activationKey`。

```azurecli
az vm extension set --publisher 'Stackify.LinuxAgent.Extension' --version 1.0 --name 'StackifyLinuxAgentExtension' --protected-settings '{"activationKey":"myActivationKey"}' --settings '{"environment":"myEnvironment"}'  --resource-group 'myResourceGroup' --vm-name 'myVmName'
```

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="error-codes"></a>錯誤碼

| 錯誤碼 | 意義 | 可能的動作 |
| :---: | --- | --- |
| 10 | 安裝錯誤 | wget 為必要項目 |
| 20 | 安裝錯誤 | python 為必要項目 |
| 30 | 安裝錯誤 | sudo 為必要項目 |
| 40 | 安裝錯誤 | activationKey 為必要項目 |
| 51 | 安裝錯誤 | 不支援 OS distro |
| 60 | 安裝錯誤 | environment 為必要項目 |
| 70 | 安裝錯誤 | 不明 |
| 80 | 啟用錯誤 | 服務安裝失敗 |
| 90 | 啟用錯誤 | 服務啟動失敗 |
| 100 | 停用錯誤 | 服務停止失敗 |
| 110 | 停用錯誤 | 服務移除失敗 |
| 120 | 解除安裝發生錯誤 | 服務停止失敗 |

如果您需要更多協助，請連絡 Stackify 支援部門：https://support.stackify.com。