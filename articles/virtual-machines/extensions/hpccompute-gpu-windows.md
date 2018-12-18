---
title: NVIDIA GPU 驅動程式擴充功能 - Azure Windows VM |Microsoft Docs
description: 在執行 Windows 的 N 系列計算 VM 上用來安裝 NVIDIA GPU 驅動程式的 Microsoft Azure 擴充功能。
services: virtual-machines-windows
documentationcenter: ''
author: vermagit
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/20/2018
ms.author: roiyz
ms.openlocfilehash: f7c7877768e2dc06e73f8c91016edd521151a11c
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145557"
---
# <a name="nvidia-gpu-driver-extension-for-windows"></a>適用於 Windows 的 NVIDIA GPU 驅動程式擴充功能

## <a name="overview"></a>概觀

此擴充功能可在 Windows N 系列虛擬機器上安裝 NVIDIA GPU 驅動程式。 視虛擬機器系列而定，擴充功能會安裝 CUDA 或 GRID 驅動程式。 若您使用此擴充功能安裝 NVIDIA 驅動程式，即表示您接受並同意 NVIDIA 使用者授權合約的條款。 在安裝過程中，您的虛擬機器可能會重新開機以便完成驅動程式設定。

在 [Linux N 系列 VM](hpccompute-gpu-linux.md) 上也有擴充功能可用來安裝 NVIDIA GPU 驅動程式。

NVIDIA 使用者授權合約條款位於此處 - https://go.microsoft.com/fwlink/?linkid=874330

## <a name="prerequisites"></a>必要條件

### <a name="operating-system"></a>作業系統

此擴充功能支援下列作業系統：

| 配送映像 | 版本 |
|---|---|
| Windows 10 | 核心 |
| Windows Server 2016 | 核心 |
| Windows Server 2012R2 | 核心 |

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
    "type": "NvidiaGpuDriverWindows",
    "typeHandlerVersion": "1.2",
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
| type | NvidiaGpuDriverWindows | 字串 |
| typeHandlerVersion | 1.2 | int |


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
    "type": "NvidiaGpuDriverWindows",
    "typeHandlerVersion": "1.2",
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
    -ExtensionName "NvidiaGpuDriverWindows" `
    -ExtensionType "NvidiaGpuDriverWindows" `
    -TypeHandlerVersion 1.2 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set `
  --resource-group myResourceGroup `
  --vm-name myVM `
  --name NvidiaGpuDriverWindows `
  --publisher Microsoft.HpcCompute `
  --version 1.2 `
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

擴充功能執行輸出會記錄至下列目錄︰

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.HpcCompute.NvidiaGpuDriverMicrosoft\
```

### <a name="error-codes"></a>錯誤碼

| 錯誤碼 | 意義 | 可能的動作 |
| :---: | --- | --- |
| 0 | 作業已順利完成 |
| 1 | 作業成功。 需要重新開機。 |
| 100 | 不支援或無法完成作業。 | 可能原因：PowerShell 版本不受支援、VM 大小不是 N 系列 VM、資料下載失敗。 請檢查記錄檔，以判斷錯誤的成因。 |
| 240、840 | 作業逾時。 | 重試作業。 |
| -1 | 發生例外狀況。 | 請檢查記錄檔，以判斷例外狀況的成因。 |
| -5x | 作業因重新開機擱置而中斷。 | 重新啟動 VM。 重新開機後將會繼續安裝。 解除安裝應以手動方式叫用。 |


### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/community/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。

## <a name="next-steps"></a>後續步驟
如需擴充功能的詳細資訊，請參閱[虛擬機器擴充功能和 Windows 功能](features-windows.md)。

如需 N 系列虛擬機器的詳細資訊，請參閱 [GPU 最佳化的虛擬機器大小](../windows/sizes-gpu.md)。