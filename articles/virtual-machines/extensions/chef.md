---
title: 適用於 Azure VM 的 Chef 擴充功能 | Microsoft Docs
description: 使用 Chef VM Extension，將 Chef Client 部署至虛擬機器。
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 09/21/2018
ms.author: roiyz
ms.openlocfilehash: 159ce1b565068e2cfdb3cb1cb2e5b5f72ff6848f
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451352"
---
# <a name="chef-vm-extension-for-linux-and-windows"></a>適用於 Linux 和 Windows 的 Chef VM Extension

Chef 軟體是適用於 Linux 和 Windows 的 DevOps 自動化平台，而 Windows 同時啟用實體和虛擬伺服器組態的管理。 Chef VM Extension 是可以在虛擬機器上啟用 Chef 的擴充功能。

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

Chef VM Extension 在 Azure 中所有[擴充功能支援的作業系統](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems)上，都獲得支援。

### <a name="internet-connectivity"></a>網際網路連線

Chef VM Extension 需要目標虛擬機器連線至網際網路，才能夠從內容傳遞網路 (CDN) 擷取 Chef Client 承載。  

## <a name="extension-schema"></a>擴充功能結構描述

下列 JSON 會顯示 Chef VM Extension 的結構描述。 擴充功能至少需要 Chef Server URL、驗證用戶端名稱和 Chef Server 的驗證金鑰；您可以在 starter-kit.zip (安裝 [Chef Automate](https://azuremarketplace.microsoft.com/marketplace/apps/chef-software.chef-automate) 時下載的檔案) 中的 `knife.rb` 檔案或獨立 [Chef Server](https://downloads.chef.io/chef-server) 中找到這些值。 因為驗證金鑰應該視為機密資料，所以應該在 **protectedSettings** 元素底下設定驗證金鑰，這表示它只能在目標虛擬機器上解密。

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "[concat(variables('vmName'),'/', parameters('chef_vm_extension_type'))]",
  "apiVersion": "2017-12-01",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
  ],
  "properties": {
    "publisher": "Chef.Bootstrap.WindowsAzure",
    "type": "[parameters('chef_vm_extension_type')]",
    "typeHandlerVersion": "1210.12",
    "settings": {
      "bootstrap_options": {
        "chef_server_url": "[parameters('chef_server_url')]",
        "validation_client_name": "[parameters('chef_validation_client_name')]"
      },
      "runlist": "[parameters('chef_runlist')]"
    },
    "protectedSettings": {
      "validation_key": "[parameters('chef_validation_key')]"
    }
  }
}  
```

### <a name="core-property-values"></a>核心屬性值

| 名稱 | 值 / 範例 | 資料類型
| ---- | ---- | ---- | ----
| apiVersion | `2017-12-01` | 字串 (日期) |
| publisher | `Chef.Bootstrap.WindowsAzure` | 字串 |
| type | `LinuxChefClient` (Linux)，`ChefClient` (Windows) | 字串 |
| typeHandlerVersion | `1210.12` | 字串 (雙精確度) |

### <a name="settings"></a>設定

| 名稱 | 值 / 範例 | 資料類型 | 必要？
| ---- | ---- | ---- | ----
| settings/bootstrap_options/chef_server_url | `https://api.chef.io/organizations/myorg` | 字串 (url) | Y |
| settings/bootstrap_options/validation_client_name | `myorg-validator` | 字串 | Y |
| settings/runlist | `recipe[mycookbook::default]` | 字串 | Y |

### <a name="protected-settings"></a>受保護的設定

| 名稱 | 範例 | 資料類型 | 必要？
| ---- | ---- | ---- | ---- |
| protectedSettings/validation_key | `-----BEGIN RSA PRIVATE KEY-----\nKEYDATA\n-----END RSA PRIVATE KEY-----` | 字串 | Y |

<!--
### Linux-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |

### Windows-specific settings

| Name | Value / Example | Data Type |
| ---- | ---- | ---- |
-->

## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 可以使用範本來部署一或多部虛擬機器、安裝 Chef Client、連線至 Chef Server 以及在伺服器上執行初始設定，如同[執行清單](https://docs.chef.io/run_lists.html)所定義

在 [Azure 快速啟動資源庫](https://github.com/Azure/azure-quickstart-templates/tree/master/chef-json-parameters-linux-vm)上可找到包含 Chef VM Extension 的 Resource Manager 範本範例。

虛擬機器擴充功能的 JSON 設定可以巢狀方式置於虛擬機器資源內部，或放在 Resource Manager JSON 範本的根目錄或最上層。 JSON 設定的放置會影響資源名稱和類型的值。 如需詳細資訊，請參閱[設定子資源的名稱和類型](../../azure-resource-manager/resource-manager-template-child-resource.md)。

## <a name="azure-cli-deployment"></a>Azure CLI 部署

Azure CLI 可以用來將 Chef VM Extension 部署到現有的 VM。 使用您驗證金鑰的內容 (此檔案的副檔名是 `.pem`) 來取代 **validation_key**。  使用入門套件中 `knife.rb` 檔案的值，來取代 **validation_client_name**、**chef_server_url** 和 **run_list**。

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myExistingVM \
  --name LinuxChefClient \
  --publisher Chef.Bootstrap.WindowsAzure \
  --version 1210.12 --protected-settings '{"validation_key": "<validation_key>"}' \
  --settings '{ "bootstrap_options": { "chef_server_url": "<chef_server_url>", "validation_client_name": "<validation_client_name>" }, "runlist": "<run_list>" }'
```

## <a name="troubleshooting-and-support"></a>疑難排解和支援

使用 Azure CLI，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請使用 Azure CLI 執行下列命令。

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myExistingVM -o table
```

擴充功能執行輸出會記錄至下列檔案︰

### <a name="linux"></a>Linux

```bash
/var/lib/waagent/Chef.Bootstrap.WindowsAzure.LinuxChefClient
```

### <a name="windows"></a>Windows

```powershell
C:\Packages\Plugins\Chef.Bootstrap.WindowsAzure.ChefClient\
```

### <a name="error-codes-and-their-meanings"></a>錯誤碼及其意義

| 錯誤碼 | 意義 | 可能的動作 |
| :---: | --- | --- |
| 51 | VM 的作業系統上不支援此擴充功能 | |

其他疑難排解資訊可以在 [Chef VM Extension 讀我檔案](https://github.com/chef-partners/azure-chef-extension)中找到。

## <a name="next-steps"></a>後續步驟

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
