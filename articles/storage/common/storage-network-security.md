---
title: 設定 Azure 儲存體防火牆和虛擬網路 | Microsoft Docs
description: 為儲存體帳戶設定多層式的網路安全性。
services: storage
author: cbrooksmsft
ms.service: storage
ms.topic: article
ms.date: 10/25/2017
ms.author: cbrooks
ms.component: common
ms.openlocfilehash: ff382becb71f187ac38b0ef5d31c1b29c43f3fe7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972550"
---
# <a name="configure-azure-storage-firewalls-and-virtual-networks"></a>設定 Azure 儲存體防火牆和虛擬網路
Azure 儲存體提供多層式的安全性模型，讓您保護特定允許網路集合的儲存體帳戶。  設定網路規則時，只有來自允許網路的應用程式可以存取儲存體帳戶。  從允許的網路呼叫時，應用程式仍然需要適當的權限 (有效的存取金鑰或 SAS 權杖) 才能存取儲存體帳戶。

> [!IMPORTANT]
> 開啟儲存體帳戶的防火牆規則，將封鎖存取資料的傳入要求，包括來自其他 Azure 服務的要求。  這包括使用入口網站、寫入記錄等。針對參與服務，您可以透過下列[例外狀況](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)一節來重新啟用功能。  若要存取入口網站，必須從位於您已設定之信任界限 (IP 或 VNet) 內的電腦來執行此作業。
>

## <a name="scenarios"></a>案例
根據預設，儲存體帳戶可以設定成拒絕存取所有網路流量 (包括網際網路流量)。  可授與存取特定 Azure 虛擬網路的流量，讓您為應用程式建置安全的網路界限。  也可以授與存取公用的網際網路 IP 位址範圍，啟用來自特定網際網路或內部部署用戶端的連線。

所有的 Azure 儲存體網路通訊協定都會執行網路規則，包括 REST 和 SMB。  從 Azure 入口網站、儲存體總管和 AZCopy 等工具存取資料，在執行網路規則時，需要明確的網路規則授與存取權。

網路規則可以套用到現有的儲存體帳戶，或者可以在建立新儲存體帳戶期間套用。

一旦套用網路規則，就會對所有要求執行。  授與特定 IP 位址服務存取權的 SAS 權杖，是用來**限制**權杖持有者的存取權，但不會授與所設定網路規則以外的新存取權。 

虛擬機器磁碟流量 (包括掛接和取消掛接作業以及磁碟 IO) **不**受網路規則影響。  分頁 Blob 的 REST存取受網路規則保護。

傳統的儲存體帳戶**不**支援防火牆與虛擬網路。

透過如本文[例外狀況](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)一節所述的方式建立例外狀況，可支援在套用網路規則的儲存體帳戶中使用非受控磁碟備份與還原虛擬機器。  防火牆例外不適用於受控磁碟，因為它們已受 Azure 管理。

## <a name="change-the-default-network-access-rule"></a>變更預設的網路存取規則
儲存體帳戶預設接受來自任何網路用戶端的連線。  若要限制對所選網路的存取，您必須先變更預設動作。

> [!WARNING]
> 變更網路規則會影響應用程式連接到 Azure 儲存體的能力。  將預設的網路規則設定為**拒絕**以封鎖所有的資料存取，除非套用特定的網路規則*授與*存取。  請務必先將存取權授與任何使用網路規則的允許網路，再變更預設規則以拒絕存取。
>

#### <a name="azure-portal"></a>Azure 入口網站
1. 巡覽至您要保護的儲存體帳戶。  

2. 按一下名為 [防火牆與虛擬網路] 的設定功能表。
3. 若要預設拒絕存取，請選擇允許「所選網路」存取權。  若要允許來自所有網路的流量，請選擇允許「所有網路」存取權。
4. 按一下 [儲存] 套用變更。

#### <a name="powershell"></a>PowerShell
1. 安裝最新的 [Azure PowerShell](/powershell/azure/install-azurerm-ps) 並[登入](/powershell/azure/authenticate-azureps)。

2. 顯示儲存體帳戶的預設規則狀態。
```PowerShell
(Get-AzureRmStorageAccountNetworkRuleSet  -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").DefaultAction
``` 

3. 根據預設，將預設規則設定為拒絕網路存取。  
```PowerShell
Update-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Deny
```    

4. 根據預設，將預設規則設定為允許網路存取。
```PowerShell
Update-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -DefaultAction Allow
```    

#### <a name="cliv2"></a>CLIv2
1. [安裝 Azure CLI](/cli/azure/install-azure-cli) 並[登入](/cli/azure/authenticate-azure-cli)。
2. 顯示儲存體帳戶的預設規則狀態。
```azurecli
az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.defaultAction
```

3. 根據預設，將預設規則設定為拒絕網路存取。  
```azurecli
az storage account update --name "mystorageaccount" --resource-group "myresourcegroup" --default-action Deny
```

4. 根據預設，將預設規則設定為允許網路存取。
```azurecli
az storage account update --name "mystorageaccount" --resource-group "myresourcegroup" --default-action Allow
```

## <a name="grant-access-from-a-virtual-network"></a>授與虛擬網路存取權
儲存體帳戶可以設定為只允許從特定的 Azure 虛擬網路存取。 

藉由在虛擬網路內啟用 Azure 儲存體的[服務端點](/azure/virtual-network/virtual-network-service-endpoints-overview)，可確保 Azure 儲存體服務流量有最佳路由。 虛擬網路和子網路的身分識別也會隨著每項要求傳輸。  系統管理員接著可以設定儲存體帳戶的網路規則，允許接收來自虛擬網路特定子網路的要求。  透過這些網路規則授與存取的用戶端，必須仍要繼續符合儲存體帳戶的授權需求，才能存取資料。

每個儲存體帳戶最多可支援 100 個虛擬網路規則，它們也可以結合 [IP 網路規則](#grant-access-from-an-internet-ip-range)。

### <a name="available-virtual-network-regions"></a>可用的虛擬網路區域
一般情況下，服務端點是在虛擬網路與相同 Azure 區域的服務執行個體之間運作。  當服務端點搭配 Azure 儲存體使用時，此範圍會擴充包含[配對的區域](/azure/best-practices-availability-paired-regions)。  這可在地區性容錯移轉期間維持持續性，而且能讓您順暢存取唯讀的異地備援儲存體 (RA-GRS) 執行個體。  將虛擬網路存取權授與儲存體帳戶的網路規則，也會將存取權授與任何 RA-GRS 執行個體。

規劃地區服務中斷期間的災害復原時，應該事先在配對區域中佈建虛擬網路。 應該啟用 Azure 儲存體的服務端點，而授與這些替代虛擬網路存取權的網路規則應該套用至您的異地備援儲存體帳戶。

> [!NOTE]
> 服務端點不適用於虛擬網路區域外和指定配對區域外的流量。  將虛擬網路存取權授與儲存體帳戶的網路規則，只能套用到儲存體帳戶主要區域或指定配對區域中的虛擬網路。
>

### <a name="required-permissions"></a>所需的權限
為將虛擬網路規則套用至儲存體帳戶，使用者必須具有要新增的子網路的「將服務加入子網路」的權限。  此權限包含在「儲存體帳戶參與者」內建角色中，並可新增以自訂角色定義。

儲存體帳戶和虛擬網路授與的存取權**可能**在不同的訂用帳戶，但這些訂用帳戶必須屬於相同的 Azure Active Directory 租用戶。

### <a name="managing-virtual-network-rules"></a>管理虛擬網路規則
您可以透過 Azure 入口網站、PowerShell 或 CLIv2 管理儲存體帳戶的虛擬網路規則。

#### <a name="azure-portal"></a>Azure 入口網站
1. 巡覽至您要保護的儲存體帳戶。  
2. 按一下名為 [防火牆與虛擬網路] 的設定功能表。
3. 請確定您已提升權限可允許「所選網路」存取權。
4. 若要將存取權授與具有新網路規則的虛擬網路，請按一下 [虛擬網路] 下的 [新增現有的]，選取現有的虛擬網路和子網路，然後按一下 [新增]。  若要建立新的虛擬網路並授與它存取權，請按一下 [新增]，提供建立新虛擬網路的必要資訊，然後按一下 [建立]。

> [!NOTE]
> 如果 Azure 儲存體的服務端點未針對所選虛擬網路和子網路事先設定，可以設定為這項作業的一部分。
>

5. 若要移除虛擬網路或子網路規則，請按一下 […] 開啟虛擬網路或子網路的操作功能表，然後按一下 [移除]。
6. 按一下 [儲存] 套用變更。

#### <a name="powershell"></a>PowerShell
1. 安裝最新的 [Azure PowerShell](/powershell/azure/install-azurerm-ps) 並[登入](/powershell/azure/authenticate-azureps)。
2. 列出虛擬網路規則
```PowerShell
(Get-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").VirtualNetworkRules
```

3. 啟用現有的虛擬網路和子網路上的 Azure 儲存體服務端點
```PowerShell
Get-AzureRmVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzureRmVirtualNetworkSubnetConfig -Name "mysubnet"  -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.Storage" | Set-AzureRmVirtualNetwork
```

4. 為虛擬網路和子網路新增網路規則。  
```PowerShell
$subnet = Get-AzureRmVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzureRmVirtualNetworkSubnetConfig -Name "mysubnet"
Add-AzureRmStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id   
```    

5. 移除虛擬網路和子網路的網路規則。  
```PowerShell
$subnet = Get-AzureRmVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzureRmVirtualNetworkSubnetConfig -Name "mysubnet"
Remove-AzureRmStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -Name "mystorageaccount" -VirtualNetworkResourceId $subnet.Id   
```    

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則網路規則不會生效。
>

#### <a name="cliv2"></a>CLIv2
1. [安裝 Azure CLI](/cli/azure/install-azure-cli) 並[登入](/cli/azure/authenticate-azure-cli)。
2. 列出虛擬網路規則
```azurecli
az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query virtualNetworkRules
```

2. 啟用現有的虛擬網路和子網路上的 Azure 儲存體服務端點
```azurecli
az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.Storage"
```

3. 為虛擬網路和子網路新增網路規則。  
```azurecli
subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
```

4. 移除虛擬網路和子網路的網路規則。 
```azurecli
subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --subnet $subnetid
```

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則網路規則不會生效。
>

## <a name="grant-access-from-an-internet-ip-range"></a>授與網際網路 IP 範圍存取權
儲存體帳戶可以設定為允許特定的公用網際網路 IP 位址範圍存取權。  此設定可在一般的網際網路流量遭到封鎖時，將存取權授與特定的網際網路型服務和內部部署網路。

使用 [CIDR 標記法](https://tools.ietf.org/html/rfc4632)以 *16.17.18.0/24* 的格式，或 *16.17.18.19* 一類的個別 IP 位址，提供允許的網際網路位址範圍。

> [!NOTE]
> 不支援使用 "/31" 或 "/32" 前置詞大小的小型位址範圍。  這些範圍應該使用個別的 IP 位址規則設定。
>

只有**公用網際網路** IP 位址允許使用 IP 網路規則。  IP 規則中不允許保留私人網路的 IP 位址範圍 (如 RFC 1918 中所定義)。  私人網路包括以 *10.\**、*172.16.\** 和 *192.168.\** 開頭的位址。

目前僅支援 IPV4 位址。

每個儲存體帳戶最多可支援 100 個 IP 網路規則，它們也可以結合 [虛擬網路規則](#grant-access-from-a-virtual-network)。

### <a name="configuring-access-from-on-premises-networks"></a>設定內部部署網路存取權
為將內部部署網路存取權授與使用 IP 網路規則的儲存體帳戶，您必須識別網路所使用的網際網路對應 IP 位址。  請連絡網路系統管理員尋求協助。

如果您的網路使用 [ExpressRoute](/azure/expressroute/expressroute-introduction) 連接至 Azure 網路，每個線路在 Microsoft Edge 會設定兩個公用 IP 位址用來連接到 Microsoft 服務，像使用 [Azure 公用對等](/azure/expressroute/expressroute-circuit-peerings#expressroute-routing-domains)的 Azure 儲存體。  若要允許從您的線路與 Azure 儲存體通訊，您必須為電路的公用 IP 位址建立 IP 網路規則。  若要尋找您 ExpressRoute 線路的公用 IP 位址，請透過 Azure 入口網站[開啟具有 ExpressRoute 的支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。


### <a name="managing-ip-network-rules"></a>管理 IP 網路規則
您可以透過 Azure 入口網站、PowerShell 或 CLIv2 管理儲存體帳戶的 IP 網路規則。

#### <a name="azure-portal"></a>Azure 入口網站
1. 巡覽至您要保護的儲存體帳戶。  
2. 按一下名為 [防火牆與虛擬網路] 的設定功能表。
3. 請確定您已提升權限可允許「所選網路」存取權。
4. 若要授與網際網路 IP 範圍存取權，請在 [防火牆] 的 [位址範圍] 之下輸入 IP 位址或位址範圍 (採用 CIDR 格式)。
5. 若要移除 IP 網路規則，請按一下網路規則旁的垃圾桶圖示。
6. 按一下 [儲存] 套用變更。

#### <a name="powershell"></a>PowerShell
1. 安裝最新的 [Azure PowerShell](/powershell/azure/install-azurerm-ps) 並[登入](/powershell/azure/authenticate-azureps)。
2. 列出 IP 網路規則。
```PowerShell
(Get-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount").IPRules
```

3. 新增個別 IP 位址的網路規則。  
```PowerShell
Add-AzureRMStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19" 
``` 

4. 新增 IP 位址範圍的網路規則。  
```PowerShell
Add-AzureRMStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24" 
```    

5. 移除個別 IP 位址的網路規則。 
```PowerShell
Remove-AzureRMStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.19"  
```

6. 移除 IP 位址範圍的網路規則。  
```PowerShell
Remove-AzureRMStorageAccountNetworkRule -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount" -IPAddressOrRange "16.17.18.0/24"  
```    

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則網路規則不會生效。
>

#### <a name="cliv2"></a>CLIv2
1. [安裝 Azure CLI](/cli/azure/install-azure-cli) 並[登入](/cli/azure/authenticate-azure-cli)。
2. 列出 IP 網路規則
```azurecli
az storage account network-rule list --resource-group "myresourcegroup" --account-name "mystorageaccount" --query ipRules
```

3. 新增個別 IP 位址的網路規則。
```azurecli
az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
```

4. 新增 IP 位址範圍的網路規則。  
```azurecli
az storage account network-rule add --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
```

5. 移除個別 IP 位址的網路規則。  
```azurecli
az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.19"
```

6. 移除 IP 位址範圍的網路規則。  
```azurecli
az storage account network-rule remove --resource-group "myresourcegroup" --account-name "mystorageaccount" --ip-address "16.17.18.0/24"
```

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則網路規則不會生效。
>

## <a name="exceptions"></a>例外狀況
雖然網路規則可以啟用大部分情況的安全網路設定，但在某些情況中，必須授與例外狀況權限才能啟用完整功能。  儲存體帳戶中可以設定受信任的 Microsoft 服務與儲存體分析資料存取的例外狀況。

### <a name="trusted-microsoft-services"></a>受信任的 Microsoft 服務
有些與儲存體帳戶互動的 Microsoft 服務是從無法透過網路規則授與存取權的網路運作。 

若要讓這類服務如預期方式運作，您可以允許受信任的 Microsoft 服務集合略過網路規則。 這些服務會使用增強式驗證存取儲存體帳戶。

啟用「信任的 Microsoft 服務」例外狀況時，下列服務 (在您的訂用帳戶中註冊時) 會被授與存取儲存體帳戶：

|服務|資源提供者名稱|目的|
|:------|:---------------------|:------|
|Azure 備份|Microsoft.Backup|在 IAAS 虛擬機器中執行未受控磁碟備份與還原。 (若為受控磁碟則非必要)。 [深入了解](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup)。|
|Azure DevTest Labs|Microsoft.DevTestLab|自訂映像建立和成品安裝。  [深入了解](https://docs.microsoft.com/azure/devtest-lab/devtest-lab-overview)。|
|Azure Event Grid|Microsoft.EventGrid|啟用 Blob 儲存體事件發佈。  [深入了解](https://docs.microsoft.com/azure/event-grid/overview)。|
|Azure 事件中心|Microsoft.EventHub|使用事件中樞擷取封存資料。  [深入了解](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)。|
|Azure 網路|Microsoft.Networking|儲存及分析網路流量記錄檔。  [深入了解](https://docs.microsoft.com/azure/network-watcher/network-watcher-packet-capture-overview)。|
|Azure 監視器|Microsoft.Insights| 允許將監視資料寫入受保護的儲存體帳戶 [深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-roles-permissions-security#monitoring-and-secured-Azure-storage-and-networks)。|
|


### <a name="storage-analytics-data-access"></a>儲存體分析資料存取
在某些情況下，需要來自網路界限外的存取權才能讀取診斷記錄檔和計量。  您可以授與網路規則的例外狀況，允許讀取存取儲存體帳戶記錄檔、計量資料表，或兩者都存取。 [深入了解儲存體分析的使用方式。](/azure/storage/storage-analytics)

### <a name="managing-exceptions"></a>管理例外狀況
您可以透過 Azure 入口網站、PowerShell 或 CLIv2 管理網路規則例外狀況。

#### <a name="azure-portal"></a>Azure 入口網站
1. 巡覽至您要保護的儲存體帳戶。  
2. 按一下名為 [防火牆與虛擬網路] 的設定功能表。
3. 請確定您已提升權限可允許「所選網路」存取權。
4. 在 [例外狀況] 下選取您希望授與權限的例外狀況。
5. 按一下 [儲存] 套用變更。

#### <a name="powershell"></a>PowerShell
1. 安裝最新的 [Azure PowerShell](/powershell/azure/install-azurerm-ps) 並[登入](/powershell/azure/authenticate-azureps)。
2. 顯示儲存體帳戶網路規則的例外狀況。
```PowerShell
(Get-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount").Bypass
```

3. 設定儲存體帳戶網路規則的例外狀況。
```PowerShell
Update-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount"  -Bypass AzureServices,Metrics,Logging
```

4. 移除儲存體帳戶網路規則的例外狀況。
```PowerShell
Update-AzureRmStorageAccountNetworkRuleSet -ResourceGroupName "myresourcegroup" -Name "mystorageaccount"  -Bypass None
```

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則移除例外狀況不會生效。
>

#### <a name="cliv2"></a>CLIv2
1. [安裝 Azure CLI](/cli/azure/install-azure-cli) 並[登入](/cli/azure/authenticate-azure-cli)。
2. 顯示儲存體帳戶網路規則的例外狀況。
```azurecli
az storage account show --resource-group "myresourcegroup" --name "mystorageaccount" --query networkRuleSet.bypass
```

3. 設定儲存體帳戶網路規則的例外狀況。
```azurecli
az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass Logging Metrics AzureServices
```

4. 移除儲存體帳戶網路規則的例外狀況。
```azurecli
az storage account update --resource-group "myresourcegroup" --name "mystorageaccount" --bypass None
```

> [!IMPORTANT]
> 請務必將[預設規則設定](#change-the-default-network-access-rule)為 [拒絕]，否則移除例外狀況不會生效。
>

## <a name="next-steps"></a>後續步驟
深入了解[服務端點](/azure/virtual-network/virtual-network-service-endpoints-overview) 中的 Azure 網路服務端點。

在 [Azure 儲存體安全性指南](storage-security-guide.md)中深入了解 Azure 儲存體安全性。
