---
title: 限制對 PaaS 資源的網路存取 - Azure PowerShell | Microsoft Docs
description: 在本文中，您會了解如何透過使用 Azure PowerShell 和虛擬網路服務端點來限制對 Azure 資源 (例如 Azure 儲存體和 Azure SQL Database) 的網路存取。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: b3977e045751165947243c67291e81b998b5fcb5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38606108"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-powershell"></a>透過使用 PowerShell 和虛擬網路服務端點來限制對 PaaS 資源的網路存取

虛擬網路服務端點可讓您限制某些 Azure 服務資源對虛擬網路子網路的網路存取。 您也可以移除對資源的網際網路存取。 服務端點提供從您虛擬網路到受支援 Azure 服務的直接連線，讓您可以使用虛擬網路的私人位址空間來存取 Azure 服務。 透過服務端點預定流向 Azure 資源的流量，一律會留在 Microsoft Azure 骨幹網路中。 在本文中，您將了解：

* 建立具有一個子網路的虛擬網路
* 新增子網路，並啟用服務端點
* 建立 Azure 資源，並僅允許從子網路對其進行網路存取
* 將虛擬機器 (VM) 部署到每個子網路
* 確認從子網路對資源的存取
* 確認從子網路和網際網路對資源的存取遭到拒絕

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，本文會要求使用 Azure PowerShell 模組版本 5.4.1 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-virtual-network"></a>建立虛擬網路

建立虛擬網路之前，您必須為虛擬網路以及在本文中建立的所有其他資源，建立資源群組。 使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup)建立資源群組。 下列範例會建立名為 myResourceGroup 的資源群組： 

```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

使用 [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) 建立虛擬網路。 下列範例會建立名為 myVirtualNetwork 的虛擬網路，位址首碼為 10.0.0.0/16。

```azurepowershell-interactive
$virtualNetwork = New-AzureRmVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

請使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) 來建立子網路設定。 下列範例會建立名為 Public 之子網路的子網路組態：

```azurepowershell-interactive
$subnetConfigPublic = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name Public `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork
```

使用 [Set-AzureRmVirtualNetwork](/powershell/module/azurerm.network/Set-AzureRmVirtualNetwork) 將子網路組態寫入至虛擬網路，以在虛擬網路中建立子網路：

```azurepowershell-interactive
$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="enable-a-service-endpoint"></a>啟用服務端點 

您只能針對支援服務端點的服務啟用服務端點。 使用 [Get-AzureRmVirtualNetworkAvailableEndpointService](/powershell/module/azurerm.network/get-azurermvirtualnetworkavailableendpointservice) 來檢視 Azure 位置中所提供且已啟用服務端點的服務。 下列範例會傳回 eastus 區域中所提供且已啟用服務端點的服務。 隨著更多 Azure 服務啟用服務端點，所傳回的服務清單會隨著時間成長。

```azurepowershell-interactive
Get-AzureRmVirtualNetworkAvailableEndpointService -Location eastus | Select Name
``` 

在虛擬網路中建立其他子網路。 在此範例中，系統會建立名為「Private (私人)」的子網路，當中包含適用於 Microsoft.Storage 的服務端點： 

```azurepowershell-interactive
$subnetConfigPrivate = Add-AzureRmVirtualNetworkSubnetConfig `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -VirtualNetwork $virtualNetwork `
  -ServiceEndpoint Microsoft.Storage

$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="restrict-network-access-for-a-subnet"></a>限制子網路的網路存取

使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) 建立網路安全性群組安全性規則。 下列規則允許對指派給 Azure 儲存體服務之公用 IP 位址的輸出存取： 

```azurepowershell-interactive
$rule1 = New-AzureRmNetworkSecurityRuleConfig `
  -Name Allow-Storage-All `
  -Access Allow `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 100 `
  -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

下列規則會拒絕對所有公用 IP 位址的存取。 上一個規則會因其具有較高優先順序而覆寫這項規則，從而允許對 Azure 儲存體之公用 IP 位址的存取。

```azurepowershell-interactive
$rule2 = New-AzureRmNetworkSecurityRuleConfig `
  -Name Deny-Internet-All `
  -Access Deny `
  -DestinationAddressPrefix Internet `
  -DestinationPortRange * `
  -Direction Outbound `
  -Priority 110 `
  -Protocol * `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange *
```

下列規則允許從任何地方輸入子網路的遠端桌面通訊協定 (RDP) 流量。 允許遠端桌面對子網路的連線，因此您可以在稍後的步驟中確認對資源的網路存取。

```azurepowershell-interactive
$rule3 = New-AzureRmNetworkSecurityRuleConfig `
  -Name Allow-RDP-All `
  -Access Allow `
  -DestinationAddressPrefix VirtualNetwork `
  -DestinationPortRange 3389 `
  -Direction Inbound `
  -Priority 120 `
  -Protocol * `
  -SourceAddressPrefix * `
  -SourcePortRange *
```

使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) 建立網路安全性群組。 下列範例會建立名為 myNsgPrivate 的網路安全性群組。

```azurepowershell-interactive
$nsg = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myNsgPrivate `
  -SecurityRules $rule1,$rule2,$rule3
```

使用 [Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig) 將網路安全性群組與「私人」子網路關聯建立，然後將子網路組態寫入虛擬網路中。 下列範例會將 myNsgPrivate 網路安全性群組與「私人」子網路建立關聯：

```azurepowershell-interactive
Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $VirtualNetwork `
  -Name Private `
  -AddressPrefix 10.0.1.0/24 `
  -ServiceEndpoint Microsoft.Storage `
  -NetworkSecurityGroup $nsg

$virtualNetwork | Set-AzureRmVirtualNetwork
```

## <a name="restrict-network-access-to-a-resource"></a>限制對資源的網路存取

如果資源是透過已針對服務端點啟用的 Azure 服務建立，則限制其網路存取的必要步驟會因為服務不同而有所差異。 請參閱個別服務的文件，以了解每個服務的特定步驟。 本文的其餘部分包含限制 Azure 儲存體帳戶網路存取的步驟 (以範例形式說明)。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) 建立 Azure 儲存體帳戶。 請將 `<replace-with-your-unique-storage-account-name>` 取代為在所有 Azure 位置間具有唯一性、長度介於 3-24 個字元，且僅使用數字和小寫字母的名稱。

```azurepowershell-interactive
$storageAcctName = '<replace-with-your-unique-storage-account-name>'

New-AzureRmStorageAccount `
  -Location EastUS `
  -Name $storageAcctName `
  -ResourceGroupName myResourceGroup `
  -SkuName Standard_LRS `
  -Kind StorageV2
```

建立儲存體帳戶之後，使用 [Get-AzureRmStorageAccountKey](/powershell/module/azurerm.storage/get-azurermstorageaccountkey) 將儲存體帳戶的金鑰取出至變數中：

```azurepowershell-interactive
$storageAcctKey = (Get-AzureRmStorageAccountKey `
  -ResourceGroupName myResourceGroup `
  -AccountName $storageAcctName).Value[0]
```

在稍後步驟中，會使用金鑰來建立檔案共用。 輸入 `$storageAcctKey` 並將值記下，因為當您在稍後步驟中將檔案共用對應至 VM 中的磁碟機時，也需要以手動方式將它輸入。

### <a name="create-a-file-share-in-the-storage-account"></a>在儲存體帳戶中建立檔案共用

使用 [New-AzureStorageContext](/powershell/module/azure.storage/new-azurestoragecontext) 建立儲存體帳戶和金鑰的內容。 內容包含儲存體帳戶名稱和帳戶金鑰：

```azurepowershell-interactive
$storageContext = New-AzureStorageContext $storageAcctName $storageAcctKey
```

使用 [New-AzureStorageShare](/powershell/module/azure.storage/new-azurestorageshare) 建立檔案共用：

$share = New-AzureStorageShare my-file-share -Context $storageContext

### <a name="deny-all-network-access-to-a-storage-account"></a>拒絕所有對儲存體帳戶的網路存取

根據預設，儲存體帳戶會接受來自任何網路用戶端的網路連線。 若要限制對選取網路的存取，請使用 [Update-AzureRmStorageAccountNetworkRuleSet](/powershell/module/azurerm.storage/update-azurermstorageaccountnetworkruleset) 將預設動作變更為「拒絕」。 一旦網路存取遭到拒絕後，就無法從任何網路存取儲存體帳戶。

```azurepowershell-interactive
Update-AzureRmStorageAccountNetworkRuleSet  `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -DefaultAction Deny
```

### <a name="enable-network-access-from-a-subnet"></a>啟用子網路的網路存取

使用 [Get-AzureRmVirtualNetwork](/powershell/module/azurerm.network/get-azurermvirtualnetwork) 取出已建立的虛擬網路，然後使用 [Get-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/get-azurermvirtualnetworksubnetconfig) 取出私人子網路物件至變數中：

```azurepowershell-interactive
$privateSubnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName "myResourceGroup" `
  -Name "myVirtualNetwork" `
  | Get-AzureRmVirtualNetworkSubnetConfig `
  -Name "Private"
```

使用 [Add-AzureRmStorageAccountNetworkRule](/powershell/module/azurerm.network/add-azurermnetworksecurityruleconfig) 允許從「私人」子網路對儲存體帳戶的網路存取。

```azurepowershell-interactive
Add-AzureRmStorageAccountNetworkRule `
  -ResourceGroupName "myresourcegroup" `
  -Name $storageAcctName `
  -VirtualNetworkResourceId $privateSubnet.Id
```

## <a name="create-virtual-machines"></a>建立虛擬機器

若要測試對儲存體帳戶的網路存取，請將 VM 部署至每個子網路。

### <a name="create-the-first-virtual-machine"></a>建立第一部虛擬機器

使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 在「公用」子網路中建立虛擬機器。 執行接下來的命令時，系統會提示您輸入認證。 您輸入的值會設定為 VM 的使用者名稱和密碼。 `-AsJob` 選項會在背景建立 VM，以便您繼續進行下一步。

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Public" `
    -Name "myVmPublic" `
    -AsJob
```

系統將傳回與下列範例輸出類似的輸出：

```powershell
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command                  
--     ----            -------------   -----         -----------     --------             -------                  
1      Long Running... AzureLongRun... Running       True            localhost            New-AzureRmVM     
```

### <a name="create-the-second-virtual-machine"></a>建立第二部虛擬機器

在「私人」子網路中建立虛擬機器：

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroup" `
    -Location "East US" `
    -VirtualNetworkName "myVirtualNetwork" `
    -SubnetName "Private" `
    -Name "myVmPrivate"
```

Azure 建立 VM 需要幾分鐘的時間。 在 Azure 建立 VM 並傳回 PowerShell 的輸出之前，請勿繼續執行後續步驟。 

## <a name="confirm-access-to-storage-account"></a>確認對儲存體帳戶的存取

請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 來傳回 VM 的公用 IP 位址。 以下範例會傳回 myVmPrivate 虛擬機器的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVmPrivate `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

使用前一個命令傳回的公用 IP 位址來取代下列命令中的 `<publicIpAddress>`，然後輸入下列命令： 

```powershell
mstsc /v:<publicIpAddress>
```

將會建立一個「遠端桌面通訊協定」(.rdp) 檔案，並下載至您的電腦。 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 選取 [確定] 。 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。

在 myVmPrivate VM 上，使用 PowerShell 將 Azure 檔案共用對應至磁碟機 Z。 在執行後續命令之前，請將 `<storage-account-key>` 和 `<storage-account-name>` 取代為您在[建立儲存體帳戶](#create-a-storage-account)中提供或取出的值。

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```
PowerShell 會傳回類似以下範例輸出的輸出：

```powershell
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
```

Azure 檔案共用已成功對應至 Z 磁碟機。

確認 VM 沒有任何以其他公用 IP 位址為目標的輸出連線：

```powershell
ping bing.com
```

您不會收到回應，因為與「私人」子網路相關聯的網路安全性群組只允許對指派給 Azure 儲存體服務的位址進行輸出存取，此位址以外的公用 IP 位址一律不允許。

關閉 myVmPrivate 虛擬機器的遠端桌面工作階段。

## <a name="confirm-access-is-denied-to-storage-account"></a>確認對儲存體帳戶的存取遭到拒絕

取得 myVmPublic VM 的公用 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress `
  -Name myVmPublic `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

使用前一個命令傳回的公用 IP 位址來取代下列命令中的 `<publicIpAddress>`，然後輸入下列命令： 

```powershell
mstsc /v:<publicIpAddress>
```

在 myVmPublic VM 上，嘗試將 Azure 檔案共用對應磁碟機 Z。在執行後續命令之前，請將 `<storage-account-key>` 和 `<storage-account-name>` 取代為您在[建立儲存體帳戶](#create-a-storage-account)中提供或取出的值。

```powershell
$acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
```

存取共用遭到拒絕，且您收到 `New-PSDrive : Access is denied` 錯誤。 存取遭到拒絕，因為 myVmPublic VM 是部署在「公用」子網路中。 「公用」子網路沒有已啟用的服務端點可供 Azure 儲存體使用，且儲存體帳戶只允許「私人」子網路而不允許「公用」子網路的網路存取。

關閉 myVmPublic VM 的遠端桌面工作階段。

嘗試從您的電腦使用下列命令來檢視儲存體帳戶中的檔案共用：

```powershell-interactive
Get-AzureStorageFile `
  -ShareName my-file-share `
  -Context $storageContext
```

存取遭到拒絕，且您收到「Get-AzureStorageFile：遠端伺服器傳回一個錯誤：(403) 禁止。HTTP 狀態碼：403 - HTTP錯誤訊息：此要求未獲授權執行此作業」錯誤，因為您的電腦不在 MyVirtualNetwork 虛擬網路的「私人」子網路中。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurepowershell-interactive 
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本文中，您已啟用虛擬網路子網路的服務端點。 您已了解可以針對使用多個 Azure 服務部署的資源啟用服務端點。 您已建立 Azure 儲存體帳戶，並將儲存體帳戶的網路存取限制為只能存取虛擬網路子網路內的資源。 若要深入了解服務端點，請參閱[服務端點概觀](virtual-network-service-endpoints-overview.md)和[管理子網路](virtual-network-manage-subnet.md)。

如果您的帳戶中有多個虛擬網路，您可以同時連線兩個虛擬網路，讓每個虛擬網路內的資源都可互相進行通訊。 若要了解作法，請參閱[連線虛擬網路](tutorial-connect-virtual-networks-powershell.md)。
