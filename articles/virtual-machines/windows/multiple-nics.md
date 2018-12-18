---
title: 在 Azure 中建立和管理使用多個 NIC 的 Windows VM | Microsoft Docs
description: 了解如何使用 Azure PowerShell 或 Resource Manager 範本，建立和管理連結多個 NIC 的 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: 9bff5b6d-79ac-476b-a68f-6f8754768413
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/26/2017
ms.author: cynthn
ms.openlocfilehash: 47f02c008a0498492af3503d90fda8ff6e2eefa8
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141493"
---
# <a name="create-and-manage-a-windows-virtual-machine-that-has-multiple-nics"></a>建立及管理具有多個 NIC 的 Windows 虛擬機器
Azure 中的虛擬機器 (VM) 可以連結多個虛擬網路介面卡 (NIC)。 常見案例是有不同的子網路可用於前端和後端連線。 您可以將 VM 上的多個 NIC 關聯至多個子網路，但這些子網路必須位於相同虛擬網路 (VNet) 中。 本文詳述如何建立已連結多個 NIC 的 VM。 您也了解如何新增或移除現有 VM 中的 NIC。 不同的 [VM 大小](sizes.md) 支援不同數量的 NIC，因此可據以調整您的 VM。

## <a name="prerequisites"></a>必要條件
確定您已[安裝和設定最新的 Azure PowerShell 版本](/powershell/azure/overview)。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 *myResourceGroup*、*myVnet* 和 *myVM*。


## <a name="create-a-vm-with-multiple-nics"></a>建立具有多個 NIC 的 VM
首先，建立資源群組。 下列範例會在 EastUs 位置建立名為 myResourceGroup 的資源群組：

```powershell
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

### <a name="create-virtual-network-and-subnets"></a>建立虛擬網路和子網路
常見的案例是有兩個或多個子網路的虛擬網路。 一個子網路可能用於前端流量，另一個則用於後端流量。 若要連結至這兩個子網路，您可在 VM 上使用多個 NIC。

1. 使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 定義兩個虛擬網路子網路。 下列範例會定義 mySubnetFrontEnd 和 mySubnetBackEnd 的子網路：

    ```powershell
    $mySubnetFrontEnd = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnetFrontEnd" `
        -AddressPrefix "192.168.1.0/24"
    $mySubnetBackEnd = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnetBackEnd" `
        -AddressPrefix "192.168.2.0/24"
    ```

2. 使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路和子網路。 下列範例會建立名為 myVnet 的虛擬網路：

    ```powershell
    $myVnet = New-AzureRmVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "EastUs" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $mySubnetFrontEnd,$mySubnetBackEnd
    ```


### <a name="create-multiple-nics"></a>建立多個 NIC
使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立兩個 NIC。 將一個 NIC 連結到前端子網路，將另一個 NIC 連結到後端子網路。 下列範例會建立名為 myNic1 和 myNic2 的兩個 NIC：

```powershell
$frontEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetFrontEnd'}
$myNic1 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic1" `
    -Location "EastUs" `
    -SubnetId $frontEnd.Id

$backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}
$myNic2 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic2" `
    -Location "EastUs" `
    -SubnetId $backEnd.Id
```

通常您也會建立[網路安全性群組](../../virtual-network/security-overview.md)來篩選至 VM 的網路流量，和建立[負載平衡器](../../load-balancer/load-balancer-overview.md)以跨多個 VM 分散流量。

### <a name="create-the-virtual-machine"></a>建立虛擬機器
現在開始建置您的 VM 組態。 在每個 VM 大小中，您可以新增至 VM 的 NIC 總數是有限制的。 如需詳細資訊，請參閱 [Windows VM 大小](sizes.md)。

1. 將您的 VM 認證設定為 `$cred` 變數，如下所示︰

    ```powershell
    $cred = Get-Credential
    ```

2. 使用 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) 來定義您的 VM。 下列範例會定義名為 myVM 的 VM，並使用支援兩個以上 NIC 的 VM 大小 (Standard_DS3_v2)：

    ```powershell
    $vmConfig = New-AzureRmVMConfig -VMName "myVM" -VMSize "Standard_DS3_v2"
    ```

3. 使用 [Set-AzureRmVMOperatingSystem](/powershell/module/azurerm.compute/set-azurermvmoperatingsystem) 和 [Set-AzureRmVMSourceImage](/powershell/module/azurerm.compute/set-azurermvmsourceimage) 建立其餘的 VM 組態。 下列範例會建立 Windows Server 2016 VM：

    ```powershell
    $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig `
        -Windows `
        -ComputerName "myVM" `
        -Credential $cred `
        -ProvisionVMAgent `
        -EnableAutoUpdate
    $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig `
        -PublisherName "MicrosoftWindowsServer" `
        -Offer "WindowsServer" `
        -Skus "2016-Datacenter" `
        -Version "latest"
   ```

4. 使用 [Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface) 連結您先前建立的兩個 NIC：

    ```powershell
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $myNic1.Id -Primary
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $myNic2.Id
    ```

5. 使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM：

    ```powershell
    New-AzureRmVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "EastUs"
    ```

6. 完成[針對多個 NIC 設定作業系統](#configure-guest-os-for-multiple-nics)中的步驟，以將次要 NIC 的路由新增至作業系統。

## <a name="add-a-nic-to-an-existing-vm"></a>將 NIC 新增至現有的 VM
若要將虛擬 NIC 新增至現有 VM，請解除配置 VM、新增虛擬 NIC，然後啟動 VM。 不同的 [VM 大小](sizes.md) 支援不同數量的 NIC，因此可據以調整您的 VM。 如有需要，您可以[調整 VM 的大小](resize-vm.md)。

1. 使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) 來解除配置 VM。 下列範例會解除配置 myResourceGroup 中名為 myVM 的 VM：

    ```powershell
    Stop-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. 使用 [Get-AzureRmVm](/powershell/module/azurerm.compute/get-azurermvm) 取得 VM 的現有組態。 下列範例可取得 myResourceGroup 中名為 myVM 之 VM 的資訊：

    ```powershell
    $vm = Get-AzureRmVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. 下列範例會使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立虛擬 NIC，其名稱為 myNic3 並已連結至 mySubnetBackEnd。 接著會使用 [Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface)，將虛擬 NIC 連結至 myResourceGroup 中名為 myVM 的 VM：

    ```powershell
    # Get info for the back end subnet
    $myVnet = Get-AzureRmVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
    $backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}

    # Create a virtual NIC
    $myNic3 = New-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic3" `
        -Location "EastUs" `
        -SubnetId $backEnd.Id

    # Get the ID of the new virtual NIC and add to VM
    $nicId = (Get-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" -Name "MyNic3").Id
    Add-AzureRmVMNetworkInterface -VM $vm -Id $nicId | Update-AzureRmVm -ResourceGroupName "myResourceGroup"
    ```

    ### <a name="primary-virtual-nics"></a>主要虛擬 NIC
    您必須在具有多個 NIC 的 VM 上將其中一個 NIC 設為主要。 如果 VM 上其中一個現有虛擬 NIC 已設定為主要，即可略過此步驟。 下列範例假設有兩個虛擬 NIC 現在出現在 VM 上，而您想要新增第一個 NIC (`[0]`) 作為主要：
        
    ```powershell
    # List existing NICs on the VM and find which one is primary
    $vm.NetworkProfile.NetworkInterfaces
    
    # Set NIC 0 to be primary
    $vm.NetworkProfile.NetworkInterfaces[0].Primary = $true
    $vm.NetworkProfile.NetworkInterfaces[1].Primary = $false
    
    # Update the VM state in Azure
    Update-AzureRmVM -VM $vm -ResourceGroupName "myResourceGroup"
    ```

4. 使用 [Start-AzureRmVm](/powershell/module/azurerm.compute/start-azurermvm) 啟動 VM：

    ```powershell
    Start-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

5. 完成[針對多個 NIC 設定作業系統](#configure-guest-os-for-multiple-nics)中的步驟，以將次要 NIC 的路由新增至作業系統。

## <a name="remove-a-nic-from-an-existing-vm"></a>從現有的 VM 移除 NIC
若要從現有的 VM 移除虛擬 NIC，您可以解除配置 VM，移除虛擬 NIC，然後啟動 VM。

1. 使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) 來解除配置 VM。 下列範例會解除配置 myResourceGroup 中名為 myVM 的 VM：

    ```powershell
    Stop-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. 使用 [Get-AzureRmVm](/powershell/module/azurerm.compute/get-azurermvm) 取得 VM 的現有組態。 下列範例可取得 myResourceGroup 中名為 myVM 之 VM 的資訊：

    ```powershell
    $vm = Get-AzureRmVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. 使用 [Get-AzureRmNetworkInterface](/powershell/module/azurerm.network/get-azurermnetworkinterface) 取得 NIC 移除相關資訊。 下列範例可取得 myNic3 相關資訊：

    ```powershell
    # List existing NICs on the VM if you need to determine NIC name
    $vm.NetworkProfile.NetworkInterfaces

    $nicId = (Get-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" -Name "myNic3").Id   
    ```

4. 使用 [Remove-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/remove-azurermvmnetworkinterface) 移除 NIC，然後使用 [Update-AzureRmVm](/powershell/module/azurerm.compute/update-azurermvm) 更新 VM。 下列範例會移除前一個步驟中 `$nicId` 所取得的 myNic3：

    ```powershell
    Remove-AzureRmVMNetworkInterface -VM $vm -NetworkInterfaceIDs $nicId | `
        Update-AzureRmVm -ResourceGroupName "myResourceGroup"
    ```   

5. 使用 [Start-AzureRmVm](/powershell/module/azurerm.compute/start-azurermvm) 啟動 VM：

    ```powershell
    Start-AzureRmVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```   

## <a name="create-multiple-nics-with-templates"></a>使用範本建立多個 NIC
Azure Resource Manager 範本提供一種方式，可在部署期間建立資源的多個執行個體，例如建立多個 NIC。 Resource Manager 範本會使用宣告式 JSON 檔案來定義您的環境。 如需詳細資訊，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。 您可以使用 *copy* 來指定要建立的執行個體數目：

```json
"copy": {
    "name": "multiplenics",
    "count": "[parameters('count')]"
}
```

如需詳細資訊，請參閱[使用 copy 建立多個執行個體](../../resource-group-create-multiple.md)。 

您也可以使用 `copyIndex()`，在資源名稱後面附加一個數字。 接著可以建立 myNic1、MyNic2 等等。 下列程式碼顯示附加索引值的範例：

```json
"name": "[concat('myNic', copyIndex())]", 
```

您可以閱讀[使用 Resource Manager 範本建立多個 NIC](../../virtual-network/template-samples.md)的完整範例。

完成[針對多個 NIC 設定作業系統](#configure-guest-os-for-multiple-nics)中的步驟，以將次要 NIC 的路由新增至作業系統。

## <a name="configure-guest-os-for-multiple-nics"></a>針對多個 NIC 設定客體作業系統

Azure 將預設閘道指派給連接至虛擬機器的第一個 (主要) 網路介面。 Azure 不會將預設閘道指派給連接至虛擬機器的其他 (次要) 網路介面。 因此依預設，您無法與次要網路介面中子網路之外的資源進行通訊。 不過，次要網路介面可與其子網路進行通訊，但不同的作業系統有不同的通訊啟用步驟。

1. 從 Windows 命令提示字元中，執行 `route print` 命令，針對有兩個連接之網路介面的虛擬機器傳回下列的類似輸出：

    ```
    ===========================================================================
    Interface List
    3...00 0d 3a 10 92 ce ......Microsoft Hyper-V Network Adapter #3
    7...00 0d 3a 10 9b 2a ......Microsoft Hyper-V Network Adapter #4
    ===========================================================================
    ```
 
    在此範例中，**Microsoft Hyper-V Network Adapter #4** (介面 7) 是未被指派預設閘道的次要網路介面。

2. 從命令提示字元中執行 `ipconfig` 命令，查看將哪些 IP 位址指派給次要網路介面。 在此範例中，192.168.2.4 會指派給介面 7。 不會為次要網路介面傳回預設閘道位址。

3. 若要將次要網路介面之子網路的外部位址之所有流量路由至子網路的閘道，請執行下列命令：

    ```
    route add -p 0.0.0.0 MASK 0.0.0.0 192.168.2.1 METRIC 5015 IF 7
    ```

    子網路的閘道位址是為子網路定義的位址範圍 (以.1 結尾) 的第一個 IP 位址。 如果您不想要路由子網路外部的所有流量，您可以改為將個別的路由新增至特定的目的地。 例如，如果您只想將從次要網路介面的流量路由到 192.168.3.0 網路，請輸入命令：

      ```
      route add -p 192.168.3.0 MASK 255.255.255.0 192.168.2.1 METRIC 5015 IF 7
      ```
  
4. 例如，若要確認與 192.168.3.0 網路上的資源成功通訊，請使用介面 7 (192.168.2.4)，輸入下列命令來 ping 192.168.3.4：

    ```
    ping 192.168.3.4 -S 192.168.2.4
    ```

    您可能需要開啟 ICMP，方法是通過正在使用下列命令來 ping 之裝置的 Windows 防火牆：
  
      ```
      netsh advfirewall firewall add rule name=Allow-ping protocol=icmpv4 dir=in action=allow
      ```
  
5. 若要確認已在路由表中新增路由，請輸入 `route print` 命令，它會傳回類似下列文字的輸出：

    ```
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.4     15
              0.0.0.0          0.0.0.0      192.168.2.1      192.168.2.4   5015
    ```

    **閘道**下使用 *192.168.1.1* 列出的路由，是根據主要網路介面預設會在該處的路由。 **閘道**下包含 *192.168.2.1* 的路由是您新增的路由。

## <a name="next-steps"></a>後續步驟
嘗試建立具有多個 NIC 的 VM 時，請檢閱 [Windows VM 大小](sizes.md)。 請注意每個 VM 大小所支援的 NIC 數目上限。 


