---
title: 適用於 Windows Server 的 Azure Hybrid Benefit
description: 瞭解如何將內部部署授權帶到 Azure，讓您的 Windows 軟體保證獲益發揮最大效益。
author: xujing-ms
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 4/22/2018
ms.author: xujing
ms.openlocfilehash: c13203c076378e1ff8f213971466eb5f63dfc4f4
ms.sourcegitcommit: fc23b4c625f0b26d14a5a6433e8b7b6fb42d868b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2021
ms.locfileid: "98539163"
---
# <a name="azure-hybrid-benefit-for-windows-server"></a>適用於 Windows Server 的 Azure Hybrid Benefit
對於擁有軟體保證的客戶，適用於 Windows Server 的 Azure Hybrid Benefit 讓您能夠以較低的成本來使用內部部署 Windows Server 授權，以及在 Azure 上執行 Windows 虛擬機器。 您可以使用適用於 Windows Server 的 Azure Hybrid Benefit 部署具有 Windows OS 的新虛擬機器。 本文章會詳述使用適用於 Windows Server 的 Azure Hybrid Benefit 來部署新 VM 的步驟，以及您如何更新現有的執行中 VM。 如需有關適用於 Windows Server 之 Azure Hybrid Benefit 的授權和節省成本詳細資訊，請參閱[適用於 Windows Server 的 Azure Hybrid Benefit 授權頁面](https://azure.microsoft.com/pricing/hybrid-use-benefit/)。

每份 2 處理器授權或每組 16 核心授權能獲得兩個最多 8 核心的執行個體，或一個最多 16 核心的執行個體。 Standard 版本授權的 Azure Hybrid Benefit 只能在內部部署或 Azure 中使用一次。 Datacenter 版本的權益則允許同時用在內部部署與 Azure。

在所有區域中，現在支援使用適用於 Windows Server 的 Azure Hybrid Benefit 搭配執行 Windows Server OS 的任何 VM，包括具有其他軟體 (例如 SQL Server 或協力廠商 Marketplace 軟體) 的 VM。 


## <a name="classic-vms"></a>傳統 VM

若為傳統 Vm，則僅支援從內部部署自訂映射部署新的 VM。 若要充分利用本文章所支援的功能，您必須先將傳統 VM 移轉至 Resource Manager 模型。

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]
 

## <a name="ways-to-use-azure-hybrid-benefit-for-windows-server"></a>使用適用於 Windows Server 的 Azure Hybrid Benefit 的方式
有幾種方式可以搭配 Azure Hybrid Benefit 使用 Windows 虛擬機器：

1. 您可以從 Azure Marketplace 上所提供的其中一個 Windows Server 映像部署 VM
2. 您可以上傳自訂 VM，並使用 Resource Manager 範本或 Azure PowerShell 進行部署
3. 您可以在使用 Azure Hybrid Benefit 執行或支付 Windows Server 的隨選成本之間，切換及轉換現有的 VM
4. 您也可以在虛擬機器擴展集上套用適用於 Windows Server 的 Azure Hybrid Benefit


## <a name="create-a-vm-with-azure-hybrid-benefit-for-windows-server"></a>使用適用於 Windows Server 的 Azure Hybrid Benefit 建立 VM
適用於 Windows Server 的 Azure Hybrid Benefit 支援所有 Windows Server OS 型映像。 您可以使用 Azure 平台支援映像，或上傳您自己的自訂 Windows Server 映像。 

### <a name="portal"></a>入口網站
若要使用適用于 Windows Server 的 Azure Hybrid Benefit 來建立 VM，請在建立程式期間，在 [ **基本** ] 索引標籤的底部，選取 [ **授權** ] 核取方塊，以使用現有的 windows Server 授權。 

### <a name="powershell"></a>PowerShell

```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -ImageName "Win2016Datacenter" `
    -LicenseType "Windows_Server"
```

### <a name="cli"></a>CLI
```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --license-type Windows_Server
```

### <a name="template"></a>範本
在 Resource Manager 範本內，必須指定 `licenseType` 的額外參數。 您可以進一步了解如何[製作 Azure Resource Manager 範本](../../azure-resource-manager/templates/template-syntax.md)
```json
"properties": {
    "licenseType": "Windows_Server",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

## <a name="convert-an-existing-vm-using-azure-hybrid-benefit-for-windows-server"></a>轉換使用適用於 Windows Server 的 Azure Hybrid Benefit 的現有 VM
如果您有想要將其轉換為利用適用於 Windows Server 的 Azure Hybrid Benefit 的現有 VM，您可以依照下列指示更新 VM 的授權類型。

> [!NOTE]
> 變更 VM 的授權類型並不會使系統重新開機或導致服務中斷。  這只是對中繼資料旗標的更新。
> 

### <a name="portal"></a>入口網站
您可以從入口網站 VM 刀鋒視窗中，藉由選取 [設定] 選項並且切換 [Azure Hybrid Benefit] 選項，將 VM 更新為使用 Azure Hybrid Benefit

### <a name="powershell"></a>PowerShell
- 將現有的 Windows Server VM 轉換為適用於 Windows Server 的 Azure Hybrid Benefit

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "Windows_Server"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
- 將具有權益的 Windows Server VM 轉換回隨用隨付

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "None"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
### <a name="cli"></a>CLI
- 將現有的 Windows Server VM 轉換為適用於 Windows Server 的 Azure Hybrid Benefit

    ```azurecli
    az vm update --resource-group myResourceGroup --name myVM --set licenseType=Windows_Server
    ```

### <a name="how-to-verify-your-vm-is-utilizing-the-licensing-benefit"></a>如何確認您的 VM 可享受授權權益
透過 PowerShell、Resource Manager 範本或入口網站部署 VM 之後，您可以使用下列方式驗證設定。

### <a name="portal"></a>入口網站
從入口網站 VM 刀鋒視窗中，您可以選取 [設定] 索引標籤以檢視適用於 Windows Server 的 Azure Hybrid Benefit 切換開關。

### <a name="powershell"></a>PowerShell
下列範例顯示單一 VM 的授權類型
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

輸出：
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Server
```

此輸出和下列在沒有適用於 Windows Server 的 Azure Hybrid Benefit 授權下所部署的 VM 之間，有著顯著的差異：
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

### <a name="cli"></a>CLI
```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVM --query "[?licenseType=='Windows_Server']" -o table
```

> [!NOTE]
> 變更 VM 的授權類型並不會使系統重新開機或導致服務中斷。 這只是中繼資料授權旗標。
>

## <a name="list-all-vms-with-azure-hybrid-benefit-for-windows-server-in-a-subscription"></a>列出訂用帳戶中具有適用於 Windows Server 的 Azure Hybrid Benefit 的所有 VM
若要查看和計算利用適用於 Windows Server 的 Azure Hybrid Benefit 部署的所有虛擬機器，您可以從訂用帳戶執行下列命令：

### <a name="portal"></a>入口網站
從虛擬機器或虛擬機器擴展集資源刀鋒視窗中，您可以將資料表資料行設定為包含 "Azure Hybrid Benefit"，以檢視所有 VM 和授權類型的清單。 VM 設定可以處於「已啟用」、「未啟用」或「不支援」狀態。

### <a name="powershell"></a>PowerShell
```powershell
$vms = Get-AzVM
$vms | ?{$_.LicenseType -like "Windows_Server"} | select ResourceGroupName, Name, LicenseType
```

### <a name="cli"></a>CLI
```azurecli
az vm list --query "[?licenseType=='Windows_Server']" -o table
```

## <a name="deploy-a-virtual-machine-scale-set-with-azure-hybrid-benefit-for-windows-server"></a>使用適用於 Windows Server 的 Azure Hybrid Benefit 部署虛擬機器擴展集
在虛擬機器擴展集 Resource Manager 範本內，必須在 VirtualMachineProfile 屬性中指定額外參數 `licenseType`。 您可以透過 ARM 範本、PowerShell、Azure CLI 或 REST，在擴展集的建立或更新期間執行此動作。

下列範例使用 ARM 範本搭配 Windows Server 2016 Datacenter 映像︰
```json
"virtualMachineProfile": {
    "storageProfile": {
        "osDisk": {
            "createOption": "FromImage"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
        }
    },
    "licenseType": "Windows_Server",
    "osProfile": {
            "computerNamePrefix": "[parameters('vmssName')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
    }
```
您也可以深入了解如何[修改虛擬機器擴展集](../../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md)以便取得更多更新擴展集的方式。

## <a name="next-steps"></a>後續步驟
- 深入瞭解 [如何使用 Azure Hybrid Benefit 來節省成本](https://azure.microsoft.com/pricing/hybrid-use-benefit/)
- 深入了解 [Azure Hybrid Benefit 的常見問題集](https://azure.microsoft.com/pricing/hybrid-use-benefit/faq/)
- 深入了解[適用於 Windows Server 的 Azure Hybrid Benefit 授權詳細指導方針](/windows-server/get-started/azure-hybrid-benefit)
- 深入了解[適用於 Windows Server 的 Azure Hybrid Benefit 和 Azure Site Recovery 能使將應用程式移轉至 Azure 更符合成本效益](https://azure.microsoft.com/blog/hybrid-use-benefit-migration-with-asr/) \(英文\)
- 深入了解[在 Azure上使用多租用戶裝載權限的 Windows 10](./windows-desktop-multitenant-hosting-deployment.md) \(英文\)
- 深入瞭解如何 [使用 Resource Manager 範本](../../azure-resource-manager/management/overview.md)
