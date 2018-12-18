---
title: NVIDIA GPU 驅動程式擴充功能 - Azure Linux VM |Microsoft Docs
description: 用於在執行 Linux 的 N 系列計算虛擬機器上安裝 NVIDIA GPU 驅動程式的 Microsoft Azure 擴充功能。
services: virtual-machines-linux
documentationcenter: ''
author: vermagit
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/20/2018
ms.author: roiyz
ms.openlocfilehash: 307bdb5fa7a5d14a77c71d0ea40634a55d8507b6
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145358"
---
# <a name="nvidia-gpu-driver-extension-for-linux"></a>適用於 Linux 的 NVIDIA GPU 驅動程式擴充功能

## <a name="overview"></a>概觀

這個擴充功能可在 Linux N 系列虛擬機器上安裝 NVIDIA GPU 驅動程式。 視虛擬機器系列而定，擴充功能會安裝 CUDA 或 GRID 驅動程式。 若您使用此擴充功能安裝 NVIDIA 驅動程式，即表示您接受並同意 NVIDIA 使用者授權合約的條款。 在安裝過程中，您的虛擬機器可能會重新開機以便完成驅動程式設定。

也可使用擴充功能在 [Windows N 系列虛擬機器](hpccompute-gpu-windows.md)上安裝 NVIDIA GPU 驅動程式。

NVIDIA 使用者授權合約條款位於此處 - https://go.microsoft.com/fwlink/?linkid=874330

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

此擴充功能支援下列作業系統：

| 配送映像 | 版本 |
|---|---|
| Linux：Ubuntu | 16.04 LTS |
| Linux：Red Hat Enterprise Linux | 7.3、7.4 |
| Linux：CentOS | 7.3、7.4 |

### <a name="internet-connectivity"></a>網際網路連線

適用於 NVIDIA GPU 驅動程式的 Microsoft Azure 擴充功能會要求目標虛擬機器需連線到網際網路並擁有存取權。

## <a name="extension-schema"></a>擴充功能結構描述

下列 JSON 會顯示擴充功能的結構描述。

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="property-values"></a>屬性值

| 名稱 | 值 / 範例 | 資料類型 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | 日期 |
| publisher | Microsoft.HpcCompute | 字串 |
| type | NvidiaGpuDriverLinux | 字串 |
| typeHandlerVersion | 1.1 | int |


## <a name="deployment"></a>部署


### <a name="azure-resource-manager-template"></a>Azure Resource Manager 範本 

也可以使用 Azure Resource Manager 範本部署 Azure VM 擴充功能。 部署一或多部需要部署後設定的虛擬機器時，很適合使用範本。

虛擬機器擴充功能的 JSON 設定可以巢狀方式置於虛擬機器資源內部，或放在 Resource Manager JSON 範本的根目錄或最上層。 JSON 設定的放置會影響資源名稱和類型的值。 如需詳細資訊，請參閱[設定子資源的名稱和類型](../../azure-resource-manager/resource-manager-template-child-resource.md)。 

下列範例假設擴充功能以巢狀方式置於虛擬機器資源內部。 在巢狀處理擴充資源時，JSON 會放在虛擬機器的 `"resources": []` 物件中。

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="powershell"></a>PowerShell

```powershell
Set-AzureRmVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "NvidiaGpuDriverLinux" `
    -ExtensionType "NvidiaGpuDriverLinux" `
    -TypeHandlerVersion 1.1 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set `
  --resource-group myResourceGroup `
  --vm-name myVM `
  --name NvidiaGpuDriverLinux `
  --publisher Microsoft.HpcCompute `
  --version 1.1 `
  --settings '{ `
  }'
```

## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure PowerShell 和 Azure CLI，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請執行下列命令。

```powershell
Get-AzureRmVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

擴充功能執行輸出會記錄至下列檔案︰

```bash
/var/log/azure/nvidia-vmext-status
```

### <a name="exit-codes"></a>結束代碼

| 結束代碼 | 意義 | 可能的動作 |
| :---: | --- | --- |
| 0 | 作業已順利完成 |
| 1 | 擴充功能的使用方式不正確。 | 請以執行輸出記錄連絡支援人員。 |
| 10 | 適用於 Hyper-V 和 Azure 的 Linux Integration Services 無法使用或未安裝。 | 檢查 lspci 輸出。 |
| 11 | 在此虛擬機器大小上找不到 NVIDIA GPU。 | 使用[支援的虛擬機器大小和作業系統](../linux/n-series-driver-setup.md)。 |
| 12 | 不支援的映像供應項目 |
| 13 | 不支援的 VM 大小 | 請使用 N 系列 VM 進行部署 |
| 14 | 作業失敗 | |
| 21 | 無法在 Ubuntu 上更新 | 檢查 "sudo apt-get update" 的輸出 |


### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/community/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。

## <a name="next-steps"></a>後續步驟
如需擴充功能的詳細資訊，請參閱[虛擬機器擴充功能和 Linux 功能](features-linux.md)。

如需 N 系列虛擬機器的詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../linux/sizes-gpu.md)。