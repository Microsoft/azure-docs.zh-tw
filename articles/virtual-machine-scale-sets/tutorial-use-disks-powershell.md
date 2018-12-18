---
title: 教學課程 - 使用 Azure PowerShell 建立及使用擴展集所適用的磁碟 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 來建立及使用虛擬機器擴展集所適用的受控磁碟，包括如何新增、準備、列出和中斷連結磁碟。
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 17f20ac2313ed4abd4f86bf10b65848a9870a688
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38606666"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-azure-powershell"></a>教學課程：使用 Azure PowerShell 建立及使用虛擬機器擴展集所適用的磁碟
虛擬機器擴展集會使用磁碟來儲存 VM 執行個體的作業系統、應用程式和資料。 當您建立及管理擴展集時，請務必選擇預期的工作負載所適用的磁碟大小和組態。 本教學課程將說明如何建立及管理 VM 磁碟。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * OS 磁碟和暫存磁碟
> * 資料磁碟
> * 標準和進階磁碟
> * 磁碟效能
> * 連結及準備資料磁碟

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。 


## <a name="default-azure-disks"></a>預設 Azure 磁碟
建立或調整擴展集後，有兩個磁碟會自動連結到各個 VM 執行個體。 

**作業系統磁碟** - 作業系統磁碟可裝載 VM 執行個體的作業系統，其大小可以高達 2 TB。 OS 磁碟預設會標示為 /dev/sda。 OS 磁碟的磁碟快取組態已針對 OS 效能進行最佳化。 因為此組態，OS 磁碟**不得**裝載應用程式或資料。 請對應用程式和資料使用資料磁碟，本文稍後會詳細說明。 

**暫存磁碟** - 暫存磁碟會使用與 VM 執行個體位於相同 Azure 主機的固態磁碟機。 暫存磁碟的效能非常好，可用於暫存資料處理等作業。 不過，如果 VM 執行個體移至新的主機，則會移除儲存在暫存磁碟上的任何資料。 暫存磁碟的大小取決於 VM 執行個體大小。 暫存磁碟會標示為 /dev/sdb，其掛接點為 /mnt。

### <a name="temporary-disk-sizes"></a>暫存磁碟大小
| 類型 | 一般大小 | 暫存磁碟大小上限 (GiB) |
|----|----|----|
| [一般用途](../virtual-machines/windows/sizes-general.md) | A、B 和 D 系列 | 1600 |
| [計算最佳化](../virtual-machines/windows/sizes-compute.md) | F 系列 | 576 |
| [記憶體最佳化](../virtual-machines/windows/sizes-memory.md) | D、E、G 和 M 系列 | 6144 |
| [儲存體最佳化](../virtual-machines/windows/sizes-storage.md) | L 系列 | 5630 |
| [GPU](../virtual-machines/windows/sizes-gpu.md) | N 系列 | 1440 |
| [高效能](../virtual-machines/windows/sizes-hpc.md) | A 和 H 系列 | 2000 |


## <a name="azure-data-disks"></a>Azure 資料磁碟
如果您需要安裝應用程式和儲存資料，您可以新增額外的資料磁碟。 資料磁碟應使用於任何需要持久且有回應之資料儲存體的情況。 每個資料磁碟皆具有 4 TB 的最大容量。 VM 執行個體的大小會決定可連結的資料磁碟數目。 每個 VM vCPU 可以連結兩個資料磁碟。

### <a name="max-data-disks-per-vm"></a>每部 VM 的資料磁碟上限
| 類型 | 一般大小 | 每部 VM 的資料磁碟上限 |
|----|----|----|
| [一般用途](../virtual-machines/windows/sizes-general.md) | A、B 和 D 系列 | 64 |
| [計算最佳化](../virtual-machines/windows/sizes-compute.md) | F 系列 | 64 |
| [記憶體最佳化](../virtual-machines/windows/sizes-memory.md) | D、E、G 和 M 系列 | 64 |
| [儲存體最佳化](../virtual-machines/windows/sizes-storage.md) | L 系列 | 64 |
| [GPU](../virtual-machines/windows/sizes-gpu.md) | N 系列 | 64 |
| [高效能](../virtual-machines/windows/sizes-hpc.md) | A 和 H 系列 | 64 |


## <a name="vm-disk-types"></a>VM 磁碟類型
Azure 提供兩種類型的磁碟。

### <a name="standard-disk"></a>標準磁碟
標準儲存體以 HDD 作為後盾，既可提供符合成本效益的儲存體，又可保有效能。 標準磁碟適合用於具成本效益的開發和測試工作負載。

### <a name="premium-disk"></a>進階磁碟
以 SSD 為基礎的高效能、低延遲磁碟，是進階磁碟的後盾。 建議將這些磁碟用於執行生產工作負載的 VM。 進階儲存體支援 DS 系列、DSv2 系列、GS 系列和 FS 系列 VM。 當您選取磁碟大小時，其值會上調為下一個類型。 例如，如果磁碟大小少於 128 GB，則磁碟類型為 P10。 如果磁碟大小介於 129 GB 與 512 GB 之間，則大小為 P20。 如果超過 512 GB，則大小為 P30。

### <a name="premium-disk-performance"></a>進階磁碟效能
|進階儲存體磁碟類型 | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 磁碟大小 (上調) | 32 GB | 64 GB | 128 GB | 512 GB | 1,024 GB (1 TB) | 2,048 GB (2 TB) | 4,095 GB (4 TB) |
| 每一磁碟的 IOPS 上限 | 120 | 240 | 500 | 2,300 | 5,000 | 7,500 | 7,500 |
每一磁碟的輸送量 | 25 MB/秒 | 50 MB/秒 | 100 MB/秒 | 150 MB/秒 | 200 MB/秒 | 250 MB/秒 | 250 MB/秒 |

雖然上表指出每個磁碟的最大 IOPS，但可藉由分割多個資料磁碟來達到較高等級的效能。 例如，Standard_GS5 VM 最高可達到 80,000 IOPS。 如需每個 VM 的最大 IOPS 的詳細資訊，請參閱 [Windows VM 大小](../virtual-machines/windows/sizes.md)。


## <a name="create-and-attach-disks"></a>建立和連結磁碟
您可以在建立擴展集時建立並連結磁碟，或使用現有的擴展集。

### <a name="attach-disks-at-scale-set-creation"></a>在建立擴展集時連結磁碟
使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 建立虛擬機器擴展集。 出現提示時，請提供 VM 執行個體的使用者名稱和密碼。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。 負載平衡器包含在 TCP 連接埠 80 上分配流量的規則，同時允許 TCP 連接埠 3389 上的遠端桌面流量以及 TCP 連接埠 5985 上的 PowerShell 遠端處理。

系統會使用 `-DataDiskSizeGb` 參數建立兩個磁碟。 第一個磁碟的大小為 *64* GB，第二個磁碟則為 *128* GB。 出現提示時，請為擴展集中的 VM 執行個體提供適當的系統管理認證：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -DataDiskSizeInGb 64,128
```

建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間。

### <a name="attach-a-disk-to-existing-scale-set"></a>將磁碟連結至現有的擴展集
您也可以將磁碟連結至現有的擴展集。 請使用在先前的步驟中建立的擴展集，透過 [Add-AzureRmVmssDataDisk](/powershell/module/azurerm.compute/add-azurermvmssdatadisk) 新增另一個磁碟。 下列範例會將另一個 *128* GB 的磁碟連結至現有的擴展集：

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzureRmVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Attach a 128 GB data disk to LUN 2
Add-AzureRmVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -CreateOption Empty `
  -Lun 2 `
  -DiskSizeGB 128

# Update the scale set to apply the change
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="prepare-the-data-disks"></a>準備資料磁碟
您所建立並連結至擴展集 VM 執行個體的磁碟，是原始磁碟。 磁碟必須完成準備工作，才能用於您的資料與應用程式。 若要準備磁碟，您可以建立分割區、建立檔案系統，並將其掛接。

若要自動執行跨擴展集中多個 VM 執行個體的程序，您可以使用 Azure 自訂指令碼延伸模組。 此延伸模組可在每個 VM 執行個體的本機位置執行指令碼，以準備已連結的資料磁碟 (舉例而言)。 如需詳細資訊，請參閱[自訂指令碼延伸模組概觀](../virtual-machines/windows/extensions-customscript.md)。

下列範例會使用 [Add-AzureRmVmssExtension](/powershell/module/AzureRM.Compute/Add-AzureRmVmssExtension) 在每個 VM 執行個體上執行 GitHub 範例存放庫中的指令碼，以準備所有已連結的原始資料磁碟：

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzureRmVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Define the script for your Custom Script Extension to run
$publicSettings = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File prepare_vm_disks.ps1"
}

# Use Custom Script Extension to prepare the attached data disks
Add-AzureRmVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

若要確認磁碟已正確備妥，請透過 RDP 連線至其中一個 VM 執行個體。 

首先，請使用 [Get-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancer) 取得負載平衡器物件。 接著，使用 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) 檢視輸入 NAT 規則。 NAT 規則會為每個 VM 執行個體列出 RDP 接聽的 *FrontendPort*。 最後，使用 [Get-AzureRmPublicIpAddress](/powershell/module/AzureRM.Network/Get-AzureRmPublicIpAddress) 取得負載平衡器的公用 IP 位址：

```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzureRmLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzureRmLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort

# View the public IP address of the load balancer
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIPAddress | Select IpAddress
```

若要連線至您的 VM，請為必要的 VM 執行個體指定您自己的公用 IP 位址和連接埠號碼，如先前的命令所示。 出現提示時，請輸入您在建立擴展集時所使用的認證。 如果您使用 Azure Cloud Shell，請從本機 PowerShell 提示字元或遠端桌面用戶端執行此步驟。 下列範例會連線至 VM 執行個體 *1*：

```powershell
mstsc /v 52.168.121.216:50001
```

在 VM 執行個體上開啟本機 PowerShell 工作階段，並使用 [Get-Disk](/powershell/module/storage/get-disk) 查看已連線的磁碟：

```powershell
Get-Disk
```

下列範例輸出顯示三個資料磁碟已連結至 VM 執行個體。

```powershell
Number  Friendly Name      HealthStatus  OperationalStatus  Total Size  Partition Style
------  -------------      ------------  -----------------  ----------  ---------------
0       Virtual HD         Healthy       Online                 127 GB  MBR
1       Virtual HD         Healthy       Online                  14 GB  MBR
2       Msft Virtual Disk  Healthy       Online                  64 GB  MBR
3       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
4       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
```

檢查 VM 執行個體的檔案系統和掛接點，如下所示：

```powershell
Get-Partition
```

下列範例輸出顯示已為三個資料磁碟指派磁碟機代號：

```powershell
   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000001

PartitionNumber  DriveLetter  Offset   Size  Type
---------------  -----------  ------   ----  ----
1                F            1048576  64 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000002

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                G            1048576  128 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000003

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                H            1048576  128 GB  IFS
```

您的擴展集中每個 VM 執行個體上的磁碟都會以相同的方式自動準備。 當您的擴展集要相應增加時，必要的資料磁碟會連結至新的 VM 執行個體。 此外也會執行自訂指令碼延伸模組，以自動準備磁碟。

關閉 VM 執行個體的遠端桌面連線工作階段。


## <a name="list-attached-disks"></a>列出已連結的磁碟
若要檢視已連結至擴展集之磁碟的相關資訊，請使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)，如下所示：

```azurepowershell-interactive
Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
```

在 *VirtualMachineProfile.StorageProfile* 屬性下，會顯示 *DataDisks* 的清單。 此時會顯示磁碟大小、儲存層和 LUN (邏輯單元編號) 的相關資訊。 下列範例輸出會詳盡顯示三個連結至擴展集的資料磁碟：

```powershell
DataDisks[0]                            :
  Lun                                   : 0
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 64
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[1]                            :
  Lun                                   : 1
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[2]                            :
  Lun                                   : 2
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
```


## <a name="detach-a-disk"></a>中斷連結磁碟
當您不再需要給定的磁碟時，您可以從擴展集將其中斷連結。 磁碟會從擴展集中的所有 VM 執行個體移除。 若要從擴展集將磁碟中斷連結，請使用 [Remove-AzureRmVmssDataDisk](/powershell/module/azurerm.compute/remove-azurermvmssdatadisk)，並指定磁碟的 LUN。 LUN 會顯示在上一節提及之 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) 的輸出中。 下列範例會從擴展集將 LUN *3* 中斷連結：

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzureRmVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Detach a disk from the scale set
Remove-AzureRmVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -Lun 2

# Update the scale set and detach the disk from the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集與磁碟，請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 刪除資源群組及其所有資源。 `-Force` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。 `-AsJob` 參數不會等待作業完成，就會將控制項傳回給提示字元。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure PowerShell 來建立及使用擴展集所適用的磁碟：

> [!div class="checklist"]
> * OS 磁碟和暫存磁碟
> * 資料磁碟
> * 標準和進階磁碟
> * 磁碟效能
> * 連結及準備資料磁碟

前往下一個教學課程，了解如何將自訂映像用於擴展集 VM 執行個體。

> [!div class="nextstepaction"]
> [將自訂映像用於擴展集 VM 執行個體](tutorial-use-custom-image-powershell.md)
