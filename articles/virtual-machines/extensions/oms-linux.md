---
title: 適用於 Linux 的 Azure Log Analytics 虛擬機器擴充功能 | Microsoft Docs
description: 使用虛擬機器擴充功能在 Linux 虛擬機器上部署 Log Analytics 代理程式。
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: c7bbf210-7d71-4a37-ba47-9c74567a9ea6
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: roiyz
ms.openlocfilehash: b8a946588d09eb05e1609344318c91f76c7ee106
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452117"
---
# <a name="log-analytics-virtual-machine-extension-for-linux"></a>適用於 Linux 的 Log Analytics 虛擬機器擴充功能

## <a name="overview"></a>概觀

Log Analytics 可提供雲端和內部部署資產的監視、警示和警示補救功能。 Microsoft 已發佈和支援適用於 Linux 的 Log Analytics 代理程式虛擬機器擴充功能。 擴充功能會在 Azure 虛擬機器上安裝 Log Analytics 代理程式，並且在現有的 Log Analytics 工作區中註冊虛擬機器。 本文件詳述適用於 Linux 的 Log Analytics 虛擬機器擴充功能所支援的平台、組態和部署選項。

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

可以針對這類 Linux 散發套件執行 Log Analytics 代理程式擴充功能。

| 配送映像 | 版本 |
|---|---|
| CentOS Linux | 6 和 7 (x86/x64) |
| Amazon Linux | 2017.09 | 
| Oracle Linux | 6 和 7 (x86/x64) |
| Red Hat Enterprise Linux Server | 6 和 7 (x86/x64) |
| Debian GNU/Linux | 8 和 9 (x86/x64) |
| Ubuntu | 14.04 LTS、16.04 LTS 和 18.04 LTS (x86/x64) |
| SUSE Linux Enterprise Server | 12 (x86/x64) |

>[!NOTE]
>任何平台皆不支援低於 1.x 版的 OpenSSL，且 1.10 版只受到 x86_64 平台 (64 位元) 的支援。  
>

### <a name="agent-and-vm-extension-version"></a>代理程式和 VM 擴充功能版本
下表針對每個版本提供 Log Analytics VM 擴充功能和 Log Analytics 代理程式套件組合的版本對應。 隨附 Log Analytics 代理程式套件組合版本的版本資訊連結。 版本資訊包含錯誤修正和適用於指定代理程式版本的新功能詳細資料。  

| Log Analytics Linux VM 擴充功能版本 | Log Analytics 代理程式套件組合版本 | 
|--------------------------------|--------------------------|
| 1.7.9 | [1.6.1-3](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.6.1.3)| 
| 1.6.42.0 | [1.6.0-42](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_v1.6.0-42)| 
| 1.4.60.2 | [1.4.4-210](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.4-210)| 
| 1.4.59.1 | [1.4.3-174](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.3-174)|
| 1.4.58.7 | [14.2-125](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.2-125)|
| 1.4.56.5 | [1.4.2-124](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.2-124)|
| 1.4.55.4 | [1.4.1-123](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.1-123)|
| 1.4.45.3 | [1.4.1-45](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.1-45)|
| 1.4.45.2 | [1.4.0-45](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent_GA_v1.4.0-45) \(英文\)|
| 1.3.127.5 | [1.3.5-127](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201705-v1.3.5-127) \(英文\)| 
| 1.3.127.7 | [1.3.5-127](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201705-v1.3.5-127) \(英文\)|
| 1.3.18.7 | [1.3.4-15](https://github.com/Microsoft/OMS-Agent-for-Linux/releases/tag/OMSAgent-201704-v1.3.4-15) \(英文\)|  

### <a name="azure-security-center"></a>Azure 資訊安全中心

Azure 資訊安全中心會自動佈建 Log Analytics 代理程式，並且將它連線至 Azure 訂用帳戶中 ASC 建立的預設 Log Analytics 工作區。 如果您使用的是 Azure 資訊安全中心，請不要執行此文件中的步驟。 這樣做會覆寫已設定的工作區，並中斷與 Azure 資訊安全中心的連線。

### <a name="internet-connectivity"></a>網際網路連線

適用於 Linux 的 Log Analytics 代理程式擴充功能會要求目標虛擬機器連線到網際網路。 

## <a name="extension-schema"></a>擴充功能結構描述

下列 JSON 顯示 Log Analytics 代理程式擴充功能的結構描述。 此擴充功能需要來自目標 Log Analytics 工作區的工作區 ID 和工作區金鑰，這些值可於 Azure 入口網站中[在 Log Analytics 工作區尋找](../../log-analytics/log-analytics-quick-collect-linux-computer.md#obtain-workspace-id-and-key)。 由於工作區金鑰應視為敏感性資料，因此應儲存在受保護的設定組態中。 Azure VM 擴充功能保護的設定資料會經過加密，只會在目標虛擬機器上解密。 請注意，**workspaceId** 和 **workspaceKey** 區分大小寫。

```json
{
  "type": "extensions",
  "name": "OMSExtension",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.7",
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

### <a name="property-values"></a>屬性值

| 名稱 | 值 / 範例 |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.EnterpriseCloud.Monitoring |
| type | OmsAgentForLinux |
| typeHandlerVersion | 1.7 |
| workspaceId (例如) | 6f680a37-00c6-41c7-a93f-1437e3462574 |
| workspaceKey (例如) | z4bU3p1/GrnWpQkky4gdabWXAhbWSTz70hm4m2Xt92XI+rSRgE8qVvRhsGo9TXffbrTahyrwv35W0pOqQAU7uQ== |


## <a name="template-deployment"></a>範本部署

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 部署一或多部需要部署後設定的虛擬機器時 (例如在 Log Analytics 上架)，很適合使用範本。 在 [Azure 快速啟動資源庫](https://github.com/Azure/azure-quickstart-templates/tree/master/201-oms-extension-ubuntu-vm)上可找到包含 Log Analytics 代理程式 VM 擴充功能的範例 Resource Manager 範本。 

虛擬機器擴充功能的 JSON 設定可以巢狀方式置於虛擬機器資源內部，或放在 Resource Manager JSON 範本的根目錄或最上層。 JSON 設定的放置會影響資源名稱和類型的值。 如需詳細資訊，請參閱[設定子資源的名稱和類型](../../azure-resource-manager/resource-manager-templates-resources.md#child-resources)。 

下列範例假設 VM 擴充功能以巢狀方式置於虛擬機器資源內部。 在巢狀處理擴充資源時，JSON 會放在虛擬機器的 `"resources": []` 物件中。

```json
{
  "type": "extensions",
  "name": "OMSExtension",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.7",
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

將擴充 JSON 置於範本的根目錄時，資源名稱包含父系虛擬機器的參考，而類型可反映巢狀的組態。  

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "<parentVmResource>/OMSExtension",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <vm-name>)]"
  ],
  "properties": {
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "type": "OmsAgentForLinux",
    "typeHandlerVersion": "1.7",
    "settings": {
      "workspaceId": "myWorkspaceId"
    },
    "protectedSettings": {
      "workspaceKey": "myWorkSpaceKey"
    }
  }
}
```

## <a name="azure-cli-deployment"></a>Azure CLI 部署

Azure CLI 可以用來將 Log Analytics 代理程式 VM 擴充功能部署到現有的虛擬機器。 將 workspaceId 和 workspaceKey 取代為您的 Log Analytics 工作區的值。 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name OmsAgentForLinux \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --version 1.7 --protected-settings '{"workspaceKey": "omskey"}' \
  --settings '{"workspaceId": "omsid"}'
```

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure CLI，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請使用 Azure CLI 執行下列命令。

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

擴充功能執行輸出會記錄至下列檔案︰

```
/opt/microsoft/omsagent/bin/stdout
```

### <a name="error-codes-and-their-meanings"></a>錯誤碼及其意義

| 錯誤碼 | 意義 | 可能的動作 |
| :---: | --- | --- |
| 9 | 啟用提前呼叫 | [將 Azure Linux 代理程式更新](https://docs.microsoft.com/azure/virtual-machines/linux/update-agent)為最新的可用版本。 |
| 10 | VM 已經連線到 Log Analytics 工作區 | 若要將 VM 連線到擴充功能結構描述中所指定的工作區，請在公用設定中將 stopOnMultipleConnections 設定為 false，或是移除此屬性。 針對此 VM 所連線的每個工作區都會向此 VM 計費一次。 |
| 11 | 提供給擴充功能的組態無效 | 依照上述範例來設定部署所需的所有屬性值。 |
| 17 | OMS 套件安裝失敗 | 
| 19 | OMI 套件安裝失敗 | 
| 20 | SCX 套件安裝失敗 |
| 51 | VM 的作業系統上不支援此擴充功能 | |
| 55 | 無法連線至 OMS 服務，或缺少必要套件，或 dpkg 套件管理員遭到鎖定| 請確認系統是否有網際網路存取權，或已提供有效的 HTTP Proxy。 此外，請檢查工作區識別碼的正確性，並確認已安裝 curl 和 tar 公用程式。 |

如需其他疑難排解資訊，請參閱 [OMS-Agent-for-Linux 疑難排解指南](../../log-analytics/log-analytics-azure-vmext-troubleshoot.md)。

### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
