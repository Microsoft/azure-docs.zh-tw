---
title: 教學課程 - 建立及管理 Azure 虛擬機器擴展集 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 建立虛擬機器擴展集，以及一些常見的管理工作，例如如何啟動和停止執行個體，或變更擴展集容量。
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
ms.date: 05/18/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 6e3262ffbf31c18611a82c8293c63e13e572e30c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38531557"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-azure-powershell"></a>教學課程 - 使用 Azure PowerShell 建立及管理虛擬機器擴展集
虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 在虛擬機器擴展集生命週期期間，您可能需要執行一或多個管理工作。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立及連線至虛擬機器擴展集
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 執行個體大小
> * 手動調整擴展集
> * 執行常見的擴展集管理工作

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。 


## <a name="create-a-resource-group"></a>建立資源群組
Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 資源群組必須在虛擬機器擴展集之前建立。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令建立資源群組。 在此範例中，會在 EastUS 區域中建立名為 myResourceGroup 的資源群組。 

```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName "myResourceGroup" -Location "EastUS"
```
在本教學課程中，當您建立或修改擴展集時，都會指定資源群組名稱。


## <a name="create-a-scale-set"></a>建立擴展集
首先，使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 設定虛擬機器執行個體的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 建立虛擬機器擴展集。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。 負載平衡器包含在 TCP 連接埠 80 上散發流量的規則，同時允許 TCP 連接埠 3389 上的遠端桌面流量以及 TCP 連接埠 5985 上的 PowerShell 遠端流量：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -Credential $cred
```

建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間。


## <a name="view-the-vm-instances-in-a-scale-set"></a>檢視擴展集中的 VM 執行個體
若要檢視擴展集中的 VM 執行個體清單，請使用 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm)，如下所示：

```azurepowershell-interactive
Get-AzureRmVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

下列範例輸出顯示擴展集中的兩個 VM 執行個體：

```powershell
ResourceGroupName         Name Location             Sku InstanceID ProvisioningState
-----------------         ---- --------             --- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUP   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

若要檢視特定 VM 執行個體的其他資訊，請將 `-InstanceId` 參數新增至 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm)。 下列範例顯示 VM 執行個體 *1* 的相關資訊：

```azurepowershell-interactive
Get-AzureRmVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="list-connection-information"></a>列出連線資訊
一個公用 IP 位址指派給將流量路由至個別 VM 執行個體的負載平衡器。 根據預設，網路位址轉譯 (NAT) 規則會新增至 Azure 負載平衡器，以將遠端連線流量轉送至每個 VM 上指定的連接埠。 若要連線至擴展集中的 VM 執行個體，您可以對指派的公用 IP 位址和連接埠號碼建立遠端連線。

若要列出要連線至擴展集中之 VM 執行個體的 NAT 連接埠，請先透過 [Get-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancer) 取得負載平衡器物件。 接著，使用 [Get-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/AzureRM.Network/Get-AzureRmLoadBalancerInboundNatRuleConfig) 檢視輸入 NAT 規則：

```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzureRmLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzureRmLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

下列範例輸出顯示執行個體名稱、負載平衡器的公用 IP 位址，以及 NAT 規則將流量轉送到的連接埠號碼：

```powershell
Name             Protocol FrontendPort BackendPort
----             -------- ------------ -----------
myScaleSet3389.0 Tcp             50001        3389
myScaleSet5985.0 Tcp             51001        5985
myScaleSet3389.1 Tcp             50002        3389
myScaleSet5985.1 Tcp             51002        5985
```

規則的*名稱*會與 VM 執行個體的名稱相對應，如先前的 [Get-AzureRmVmssVM](/powershell/module/azurerm.compute/get-azurermvmssvm) 命令所示。 例如，若要連線至 VM 執行個體 *0*，您會使用 *myScaleSet3389.0* 並連線至連接埠 *50001*。 若要連線至 VM 執行個體 *1*，請使用 *myScaleSet3389.1* 中的值並連線至連接埠 *50002*。 若要使用 PowerShell 遠端處理，您會連線至 *TCP* 連接埠 *5985* 的適當 VM 執行個體規則。

使用 [Get-AzureRmPublicIpAddress](/powershell/module/AzureRM.Network/Get-AzureRmPublicIpAddress) 檢視負載平衡器的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Name "myPublicIPAddress" | Select IpAddress
```

範例輸出︰

```powershell
IpAddress
---------
52.168.121.216
```

建立您第一個 VM 執行個體的遠端連線。 請指定必要 VM 執行個體的公用 IP 位址和連接埠號碼，如先前的命令所示。 出現提示時，請輸入您在建立擴展集時所使用的認證 (在範例命令中預設為 *azureuser* 和 *P@ssw0rd!*)。 如果您使用 Azure Cloud Shell，請從本機 PowerShell 提示字元或遠端桌面用戶端執行此步驟。 下列範例會連線至 VM 執行個體 *1*：

```powershell
mstsc /v 52.168.121.216:50001
```

登入該 VM 執行個體後，您可以視需要執行一些手動組態變更。 目前請先關閉遠端連線。


## <a name="understand-vm-instance-images"></a>了解 VM 執行個體映像
Azure Marketplace 包含許多可用來建立 VM 執行個體的映像。 若要檢視可用發行者清單，請使用 [Get-AzureRmVMImagePublisher](/powershell/module/azurerm.compute/get-azurermvmimagepublisher) 命令。

```azurepowershell-interactive
Get-AzureRmVMImagePublisher -Location "EastUS"
```

若要檢視給定發行者的映像清單，請使用 [Get-AzureRmVMImageSku](/powershell/module/azurerm.compute/get-azurermvmimagesku)。 您也可以依 `-PublisherName` 或 `–Offer` 來篩選此映像清單。 在下列範例中，會篩選出發行者名稱為 *MicrosoftWindowsServer*、且供應項目符合 *WindowsServer* 的所有映像的清單：

```azurepowershell-interactive
Get-AzureRmVMImageSku -Location "EastUS" -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer"
```

下列範例輸出顯示所有可用的 Windows Server 映像：

```powershell
Skus                                  Offer         PublisherName          Location
----                                  -----         -------------          --------
2008-R2-SP1                           WindowsServer MicrosoftWindowsServer eastus
2008-R2-SP1-smalldisk                 WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter                    WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter-smalldisk          WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core           WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core-smalldisk WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-Containers       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-RDSH             WindowsServer MicrosoftWindowsServer eastus
2016-Nano-Server                      WindowsServer MicrosoftWindowsServer eastus
```

當您在教學課程一開始建立擴展集時，您為 VM 執行個體提供了預設的 VM 映像 *Windows Server 2016 DataCenter*。 您可以根據 [Get-AzureRmVMImageSku](/powershell/module/azurerm.compute/get-azurermvmimagesku) 的輸出指定不同的 VM 映像。 下列範例會使用 `-ImageName` 參數指定 VM 映像 *MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest*，以建立擴展集。 建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間，而您並不需要部署下列擴展集：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup2" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet2" `
  -VirtualNetworkName "myVnet2" `
  -SubnetName "mySubnet2" `
  -PublicIpAddressName "myPublicIPAddress2" `
  -LoadBalancerName "myLoadBalancer2" `
  -UpgradePolicyMode "Automatic" `
  -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
  -Credential $cred
```


## <a name="understand-vm-instance-sizes"></a>了解 VM 執行個體大小
VM 執行個體大小 (或 *SKU*) 會決定可供 VM 執行個體使用的計算資源 (例如 CPU、GPU 和記憶體) 數量。 擴展集中的 VM 執行個體必須針對預期的工作負載適當設定大小。

### <a name="vm-instance-sizes"></a>VM 執行個體大小
下表會將一般 VM 大小分類成各種使用案例。

| 類型                     | 一般大小           |    說明       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [一般用途](../virtual-machines/windows/sizes-general.md)         |Dsv3、Dv3、DSv2、Dv2、DS、D、Av2、A0-7| 平衡的 CPU 對記憶體。 適用於開發/測試及小型到中型應用程式和資料解決方案。  |
| [計算最佳化](../virtual-machines/windows/sizes-compute.md)   | Fs、F             | CPU 與記憶體的比例高。 適用於中流量應用程式、網路設備，以及批次處理。        |
| [記憶體最佳化](../virtual-machines/windows/sizes-memory.md)    | Esv3、Ev3、M、GS、G、DSv2、DS、Dv2、D   | 記憶體與核心的比例高。 適用於關聯式資料庫、中型到大型快取，以及記憶體內分析。                 |
| [儲存體最佳化](../virtual-machines/windows/sizes-storage.md)      | Ls                | 高磁碟輸送量及 IO。 適用於巨量資料、SQL 及 NoSQL 資料庫。                                                         |
| [GPU](../virtual-machines/windows/sizes-gpu.md)          | NV、NC            | 以大量圖形轉譯和視訊編輯為目標的特製化 VM。       |
| [高效能](../virtual-machines/windows/sizes-hpc.md) | H、A8-11          | 我們的最強大 CPU VM，可搭配選用的高輸送量網路介面 (RDMA)。 

### <a name="find-available-vm-instance-sizes"></a>尋找可用的 VM 執行個體大小
若要查看特定區域中可用的 VM 執行個體大小清單，請使用 [Get-AzureRmVMSize](/powershell/module/azurerm.compute/get-azurermvmsize) 命令。 

```azurepowershell-interactive
Get-AzureRmVMSize -Location "EastUS"
```

其輸出類似於下列扼要的範例，顯示指派給每個 VM 大小的資源：

```powershell
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_DS1_v2                    1       3584                4        1047552                 7168
Standard_DS2_v2                    2       7168                8        1047552                14336
[...]
Standard_A0                        1        768                1        1047552                20480
Standard_A1                        1       1792                2        1047552                71680
[...]
Standard_F1                        1       2048                4        1047552                16384
Standard_F2                        2       4096                8        1047552                32768
[...]
Standard_NV6                       6      57344               24        1047552               389120
Standard_NV12                     12     114688               48        1047552               696320
```

當您在教學課程一開始建立擴展集時，您為 VM 執行個體提供了預設的 VM SKU Standard_DS1_v2。 您可以根據 [Get-AzureRmVMSize](/powershell/module/azurerm.compute/get-azurermvmsize) 的輸出指定不同的 VM 執行個體大小。 下列範例會使用 `-VmSize` 參數指定 VM 執行個體大小 *Standard_F1*，以建立擴展集。 建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間，而您並不需要部署下列擴展集：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup3" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet3" `
  -VirtualNetworkName "myVnet3" `
  -SubnetName "mySubnet3" `
  -PublicIpAddressName "myPublicIPAddress3" `
  -LoadBalancerName "myLoadBalancer3" `
  -UpgradePolicyMode "Automatic" `
  -VmSize "Standard_F1" `
  -Credential $cred
```


## <a name="change-the-capacity-of-a-scale-set"></a>變更擴展集的容量
當您建立擴展集時，您要求了兩個 VM 執行個體。 若要增加或減少擴展集中的 VM 執行個體數目，您可以手動變更容量。 擴展集會建立或移除所需的 VM 執行個體數目，然後設定用來分散流量的負載平衡器。

首先，使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) 建立擴展集物件，然後指定 `sku.capacity` 的新值。 若要套用容量變更，請使用 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss)。 下列範例會將擴展集中的 VM 執行個體數目設為 *3*：

```azurepowershell-interactive
# Get current scale set
$vmss = Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 3
Update-AzureRmVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss 
```

如果需要幾分鐘的時間才能更新您擴展集的容量。 若要查看擴展集中目前包含的執行個體數目，請使用 [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss)：

```azurepowershell-interactive
Get-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

下列範例輸出顯示目前的擴展集容量為 *3*：

```powershell
Sku        :
  Name     : Standard_DS2
  Tier     : Standard
  Capacity : 3
```


## <a name="common-management-tasks"></a>一般管理工作
現在，您可以建立擴展集、列出連線資訊，以及連線至 VM 執行個體。 您已了解如何對 VM 執行個體使用不同的作業系統映像、選取不同的 VM 大小，或手動調整執行個體數目。 在日常管理工作中，您可能需要停止、啟動或重新啟動擴展集中的 VM 執行個體。

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>停止及解除配置擴展集中的 VM 執行個體
若要停止擴展集中的一或多個 VM，請使用 [Stop-AzureRmVmss](/powershell/module/azurerm.compute/stop-azurermvmss)。 `-InstanceId` 參數可讓您指定停止一或多個 VM。 如果您未指定執行個體識別碼，則會停止擴展集中的所有 VM。 下列範例會停止執行個體 *1*：

```azurepowershell-interactive
Stop-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

預設會解除配置已停止的 VM，而且不會產生計算費用。 如果您想要在停止時保留處於佈建狀態的 VM，請將 `-StayProvisioned` 參數新增至先前的命令。 保持佈建的已停止 VM 會產生一般計算費用。

### <a name="start-vm-instances-in-a-scale-set"></a>啟動擴展集中的 VM 執行個體
若要啟動擴展集中的一或多個 VM，請使用 [Start-AzureRmVmss](/powershell/module/azurerm.compute/start-azurermvmss)。 `-InstanceId` 參數可讓您指定啟動一或多個 VM。 如果您未指定執行個體識別碼，則會啟動擴展集中的所有 VM。 下列範例會啟動執行個體 *1*：

```azurepowershell-interactive
Start-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

### <a name="restart-vm-instances-in-a-scale-set"></a>重新啟動擴展集中的 VM 執行個體
若要重新啟動擴展集中的一或多個 VM，請使用 [Retart-AzureRmVmss](/powershell/module/azurerm.compute/restart-azurermvmss)。 `-InstanceId` 參數可讓您指定重新啟動一或多個 VM。 如果您未指定執行個體識別碼，則會重新啟動擴展集中的所有 VM。 下列範例會重新啟動執行個體 *1*：

```azurepowershell-interactive
Restart-AzureRmVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="clean-up-resources"></a>清除資源
刪除資源群組時，將會同時刪除其內含的所有資源，例如 VM 執行個體、虛擬網路和磁碟。 `-Force` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。 `-AsJob` 參數不會等待作業完成，就會將控制項傳回給提示字元。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure PowerShell 執行一些基本的擴展集建立和管理工作：

> [!div class="checklist"]
> * 建立及連線至虛擬機器擴展集
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 大小
> * 手動調整擴展集
> * 執行常見的擴展集管理工作

請前往下一個教學課程，了解擴展集磁碟。

> [!div class="nextstepaction"]
> [使用資料磁碟搭配擴展集](tutorial-use-disks-powershell.md)
