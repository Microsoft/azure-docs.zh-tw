---
title: 使用 Azure 虛擬網路服務端點保護 Azure Cosmos DB 帳戶的存取權 | Microsoft Docs
description: 本文件將說明設定 Azure Cosmos DB 虛擬網路服務端點所需的步驟。
services: cosmos-db
author: kanshiG
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: govindk
ms.openlocfilehash: a4758e5597876112fa7a85850786491e22af8c83
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037130"
---
# <a name="secure-access-to-an-azure-cosmos-db-account-by-using-azure-virtual-network-service-endpoint"></a>使用 Azure 虛擬網路服務端點保護 Azure Cosmos DB 帳戶的存取權

Azure CosmosDB 帳戶可以設定為只允許從 Azure 虛擬網路的特定子網路存取。 藉由啟用虛擬網路及其子網路中的 Azure CosmosDB [服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)，可確保流量會透過最佳且安全的路由傳送至 Azure Cosmos DB。  

Azure Cosmos DB 是全域散發的多模型資料庫服務。 您可以將 Azure Cosmos DB 帳戶中的資料複寫到多個區域。 如果對 Azure Cosmos DB 設定虛擬網路服務端點，則每個虛擬網路都會允許屬於特定子網路的 IP 進行存取。 下圖顯示已啟用虛擬網路服務端點的 Azure Cosmos DB 圖例：

![虛擬網路服務端點架構](./media/vnet-service-endpoint/vnet-service-endpoint-architecture.png)

對 Azure Cosmos DB 帳戶設定虛擬網路服務端點後，您只能從指定的子網路存取該帳戶，所有公用/網際網路存取都會遭到移除。 若要深入了解服務端點，請參閱 Azure [虛擬網路服務端點概觀](../virtual-network/virtual-network-service-endpoints-overview.md)一文。

## <a name="configure-service-endpoint-by-using-azure-portal"></a>使用 Azure 入口網站來設定服務端點
### <a name="configure-service-endpoint-for-an-existing-azure-virtual-network-and-subnet"></a>對現有的 Azure 虛擬網路和子網路設定服務端點

1. 在 [所有資源] 刀鋒視窗中，尋找要設定 Azure Cosmos DB 服務端點的虛擬網路。 瀏覽至 [服務端點] 刀鋒視窗，並確定已為 "Azure.CosmosDB" 服務端點啟用虛擬網路的子網路。  

   ![確認已啟用服務端點](./media/vnet-service-endpoint/confirm-service-endpoint-enabled.png)

2. 從 [所有資源] 刀鋒視窗中，尋找要保護的 Azure Cosmos DB 帳戶。  

3. 啟用虛擬網路服務端點之前，請複製與 Azure Cosmos DB 帳戶供相關聯的 IP 防火牆資訊以供未來使用。 服務端點設定好之後，您可以重新啟用 IP 防火牆。  

4. 從設定功能表中選取 [防火牆與虛擬網路]，並選擇 [允許從選取的網路進行存取]。  

3. 若要將存取權授與現有虛擬網路的子網路，請在虛擬網路下選取 [新增現有 Azure 虛擬網路]。  

4. 選取您要從中新增 Azure 虛擬網路的**訂用帳戶**。 選取要取得 Azure Cosmos DB 帳戶存取權的 Azure **虛擬網路**和**子網路**。 接下來選取 [啟用] 以啟用具有 "Microsoft.AzureCosmosDB" 服務端點的選定網路。 完成時，請選取 [新增]。  

   ![選取虛擬網路和子網路](./media/vnet-service-endpoint/choose-subnet-and-vnet.png)

   > [!NOTE]
   > 如果 Azure Cosmos DB 的服務端點未針對所選 Azure 虛擬網路和子網路事先設定，可以設定為這項作業的一部分。 啟用存取權可能需要 15 分鐘的時間才能完成。 在記下防火牆 ACL 的內容之後，請務必停用 IP 防火牆，以便之後予以重新啟用。 

   ![已成功設定虛擬網路和子網路](./media/vnet-service-endpoint/vnet-and-subnet-configured-successfully.png)

現在您的 Azure Cosmos DB 帳戶將只允許來自此選定子網路的流量。 如果您先前都有啟用 IP 防火牆，請使用先前的資訊將其重新啟用。

### <a name="configure-service-endpoint-for-a-new-azure-virtual-network-and-subnet"></a>對新的 Azure 虛擬網路和子網路設定服務端點

1. 從 [所有資源] 刀鋒視窗中，尋找要保護的 Azure Cosmos DB 帳戶。  

> [!NOTE]
> 如果您的 Azure Cosmos DB 帳戶已設定 IP 防火牆，則須注意防火牆組態，請移除 IP 防火牆，然後啟用服務端點。 如果您未停用防火牆就啟用服務端點，來自該 IP 範圍的流量將會遺失虛擬 IP 身分識別而遭到捨棄，並顯示 IP 篩選條件錯誤訊息。 因此，若要避免這個錯誤，請一律停用防火牆規則、複製規則、啟用子網路的服務端點，最後為 Cosmos DB 的子網路設定 ACL。 服務端點設定好並新增 ACL 之後，您可以再次重新啟用 IP 防火牆 (如有需要)。

2. 啟用虛擬網路服務端點之前，請複製與 Azure Cosmos DB 帳戶供相關聯的 IP 防火牆資訊以供未來使用。 服務端點設定好之後，您可以重新啟用 IP 防火牆。  

3. 從設定功能表中選取 [防火牆與 Azure 虛擬網路]，並選擇 [允許從選取的網路進行存取]。  

4. 若要授與新 Azure 虛擬網路的存取權，請在虛擬網路下選取 [新增新的 Azure 虛擬網路]。  

5. 提供建立新虛擬網路所需的詳細資料，然後選取 [建立]。 這會建立已啟用 "Microsoft.AzureCosmosDB" 服務端點的子網路。

   ![選取虛擬網路與新虛擬網路的子網路](./media/vnet-service-endpoint/choose-subnet-and-vnet-new-vnet.png)

## <a name="allow-access-from-azure-portal"></a>允許從 Azure 入口網站存取

當您的 Azure Cosmos DB 資料庫帳戶啟用 Azure 虛擬網路服務端點之後，系統會預設禁止來自入口網站或其他 Azure 服務的存取。 從所設定子網路以外的機器存取 Azure Cosmos DB 資料庫帳戶會遭到封鎖 (包括來自入口網站的存取)。

![允許從入口網站存取](./media/vnet-service-endpoint/allow-access-from-portal.png)

如果您的 Azure Cosmos DB 帳戶會由其他 Azure 服務 (例如 Azure 搜尋服務) 使用，或是會從串流分析或 Power BI 進行存取，則您可以核取 [允許存取 Azure 服務] 來允許存取。

若要確保您可以從入口網站存取 Azure Cosmos DB 計量，您必須啟用 [允許存取 Azure 入口網站] 選項。 若要深入了解這些選項，請參閱[來自 Azure 入口網站的連線](firewall-support.md#connections-from-the-azure-portal)和[來自 Azure PaaS 服務的連線](firewall-support.md#connections-from-global-azure-datacenters-or-azure-paas-services)章節。 選取存取之後，請選取 [儲存] 以儲存設定。

## <a name="remove-a-virtual-network-or-subnet"></a>移除現有虛擬網路或子網路 

1. 從 [所有資源] 刀鋒視窗中，尋找您要對其指派端點的 Azure Cosmos DB 帳戶。  

2. 在設定功能表中選取 [防火牆與虛擬網路]。  

3. 若要移除虛擬網路或子網路規則，請選取該虛擬網路或子網路旁的 […]，然後選取 [移除]。

   ![移除虛擬網路](./media/vnet-service-endpoint/remove-a-vnet.png)

4.  按一下 [儲存] 套用變更。

## <a name="configure-service-endpoint-by-using-azure-powershell"></a>使用 Azure PowerShell 來設定服務端點 

若要使用 Azure PowerShell 對 Azure Cosmos DB 帳戶設定服務端點，請使用下列步驟：  

1. 安裝最新的 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) 並[登入](https://docs.microsoft.com/powershell/azure/authenticate-azureps)。  啟用帳戶的服務端點之前，請確定您已記下 IP 防火牆設定並完全刪除 IP 防火牆。


> [!NOTE]
> 如果您的 Azure Cosmos DB 帳戶已設定 IP 防火牆，則須注意防火牆組態，請移除 IP 防火牆，然後啟用服務端點。 如果您未停用防火牆就啟用服務端點，來自該 IP 範圍的流量將會遺失虛擬 IP 身分識別而遭到捨棄，並顯示 IP 篩選條件錯誤訊息。 因此，若要避免這個錯誤，請一律停用防火牆規則、複製規則、啟用子網路的服務端點，最後為 Cosmos DB 的子網路設定 ACL。 服務端點設定好並新增 ACL 之後，您可以再次重新啟用 IP 防火牆 (如有需要)。

2. 啟用虛擬網路服務端點之前，請複製與 Azure Cosmos DB 帳戶供相關聯的 IP 防火牆資訊以供未來使用。 服務端點設定好之後，您可以重新啟用 IP 防火牆。  

3. 對現有的虛擬網路子網路啟用服務端點。  

   ```powershell
   $rgname= "<Resource group name>"
   $vnName = "<virtual network name>"
   $sname = "<Subnet name>"
   $subnetPrefix = "<Subnet address range>"

   Get-AzureRmVirtualNetwork `
    -ResourceGroupName $rgname `
    -Name $vnName | Set-AzureRmVirtualNetworkSubnetConfig `
    -Name $sname  `
    -AddressPrefix $subnetPrefix `
    -ServiceEndpoint "Microsoft.AzureCosmosDB" | Set-AzureRmVirtualNetwork
   ```

4. 確定虛擬網路與子網路已為 Azure Cosmos DB 啟用服務端點，以便在 CosmosDB 帳戶上啟用 ACL。

   ```powershell
   $vnProp = Get-AzureRmVirtualNetwork `
     -Name $vnName  -ResourceGroupName $rgName
   ```

5. 請執行下列 Cmdlet 以取得 Azure Cosmos DB 帳戶的屬性：  

   ```powershell
   $apiVersion = "2015-04-08"
   $acctName = "<Azure Cosmos DB account name>"

   $cosmosDBConfiguration = Get-AzureRmResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
     -ApiVersion $apiVersion `
     -ResourceGroupName $rgName `
     -Name $acctName
   ```

6. 初始化變數以供稍後使用。 設定現有帳戶定義中的所有變數，如果您有多個位置，請務必將其新增為陣列的一部分。 在此步驟中，您也可以藉由將 "accountVNETFilterEnabled" 變數設定為 "True" 來設定虛擬網路服務端點。 稍後，這個值會指派給 "isVirtualNetworkFilterEnabled" 參數。  

   ```powershell
   $locations = @(@{})

   <# If you have read regions in addition to a write region, use the following code to set the $locations variable instead.

   $locations = @(@{"locationName"="<Write location>"; 
                 "failoverPriority"=0}, 
               @{"locationName"="<Read location>"; 
                  "failoverPriority"=1}) #>

   $consistencyPolicy = @{}
   $cosmosDBProperties = @{}

   $locations[0]['failoverPriority'] = $cosmosDBConfiguration.Properties.failoverPolicies.failoverPriority
   $locations[0]['locationName'] = $cosmosDBConfiguration.Properties.failoverPolicies.locationName

   $consistencyPolicy = $cosmosDBConfiguration.Properties.consistencyPolicy

   $accountVNETFilterEnabled = $True
   $subnetID = $vnProp.Id+"/subnets/" + $sname  
   $virtualNetworkRules = @(@{"id"=$subnetID})
   $databaseAccountOfferType = $cosmosDBConfiguration.Properties.databaseAccountOfferType
   ```

7. 若要使用新組態來更新 Azure Cosmos DB 帳戶屬性，請執行下列 Cmdlet： 

   ```powershell
   $cosmosDBProperties['databaseAccountOfferType'] = $databaseAccountOfferType
   $cosmosDBProperties['locations'] = $locations
   $cosmosDBProperties['consistencyPolicy'] = $consistencyPolicy
   $cosmosDBProperties['virtualNetworkRules'] = $virtualNetworkRules
   $cosmosDBProperties['isVirtualNetworkFilterEnabled'] = $accountVNETFilterEnabled

   Set-AzureRmResource `
     -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
     -ApiVersion $apiVersion `
     -ResourceGroupName $rgName `
     -Name $acctName -Properties $CosmosDBProperties
   ```

8. 若要確認 Azure Cosmos DB 帳戶已透過上一個步驟中設定的虛擬網路服務端點進行更新，請執行下列命令：

   ```powershell
   $UpdatedcosmosDBConfiguration = Get-AzureRmResource `
     -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
     -ApiVersion $apiVersion `
     -ResourceGroupName $rgName `
     -Name $acctName

   $UpdatedcosmosDBConfiguration.Properties
   ```

## <a name="add-virtual-network-service-endpoint-for-an-azure-cosmos-db-account-that-has-ip-firewall-enabled"></a>為已啟用 IP 防火牆的 Azure Cosmos DB 帳戶新增虛擬網路服務端點

1. 先對 Azure Cosmos DB 帳戶停用 IP 防火牆存取。  

2. 使用前面幾節所述的其中一種方法對 Azure Cosmos DB 帳戶啟用虛擬網路端點。  

3. 重新啟用 IP 防火牆存取。 

## <a name="test-the-access"></a>測試存取

若要檢查 Azure Cosmos DB 的服務端點是否已如預期般設定，請使用下列步驟：

* 將虛擬網路中的兩個子網路設定為前端及後端，為後端子網路啟用 Cosmos DB 服務端點。  

* 在 Cosmos DB 帳戶中啟用後端子網路的存取權。  

* 建立兩個虛擬機器 - 一個在後端子網路中，而另一個在前端子網路中。  

* 試著從兩個虛擬機器存取 Azure Cosmos DB 帳戶。 您應該可以從建立於後端子網路中的虛擬機器進行連線，但無法從建立於前端子網路的虛擬機器連線。 當您嘗試從前端子網路進行連線時，此要求將會導致出現錯誤 404，這表示已使用虛擬網路服務端點來保護對 Azure Cosmos DB 的存取。 後端子網路中的機器應可正常運作。

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="what-happens-when-you-access-an-azure-cosmos-db-account-that-has-virtual-network-access-control-list-acl-enabled"></a>存取已啟用虛擬網路存取控制清單 (ACL) 的 Azure Cosmos DB 帳戶時，會發生什麼事？  

系統會傳回 HTTP 404 錯誤。  

### <a name="are-subnets-of-a-virtual-network-created-in-different-regions-allowed-to-access-an-azure-cosmos-db-account-in-another-region-for-example-if-azure-cosmos-db-account-is-in-west-us-or-east-us-and-virtual-networks-are-in-multiple-regions-can-the-virtual-network-access-azure-cosmos-db"></a>在不同區域中建立的虛擬網路可存取另一個區域中的 Azure Cosmos DB 帳戶嗎? 例如，如果 Azure Cosmos DB 帳戶位於美國西部或美國東部，而虛擬網路位於多個區域，那麼虛擬網路可存取 Azure Cosmos DB 嗎？  

可以，在不同區域中建立的虛擬網路可透過新功能進行存取。 

### <a name="can-an-azure-cosmos-db-account-have-both-virtual-network-service-endpoint-and-a-firewall"></a>Azure Cosmos DB 帳戶可同時具備虛擬網路服務端點和防火牆嗎？  

可以，虛擬網路服務端點和防火牆可以共存。 一般來說，您應在設定虛擬網路服務端點之前，確定對入口網站的存取會永遠啟用，以便您檢視與容器相關聯的計量。

### <a name="can-i-allow-access-to-other-azure-services-from-a-given-azure-region-when-service-endpoint-access-is-enabled-for-azure-cosmos-db"></a>對 Azure Cosmos DB 啟用服務端點存取時，我可以「允許從指定的 Azure 區域存取存取其他 Azure 服務」嗎？  

只有當您需要讓其他 Azure 第一方服務 (例如 Azure Data factory、Azure 搜尋服務)，或任何部署在指定 Azure 區域的服務存取 Azure Cosmos DB 帳戶時，此項目才是必要的。

### <a name="how-many-virtual-network-service-endpoints-are-allowed-for-azure-cosmos-db"></a>Azure Cosmos DB 可設定多少個虛擬網路服務端點？  

一個 Azure Cosmos DB 帳戶可設定 64 個虛擬網路服務端點。

### <a name="what-is-the-relationship-between-service-endpoint-and-network-security-group-nsg-rules"></a>服務端點與網路安全性群組 (NSG) 規則的關係為何？  

Azure Cosmos DB 中的 NSG 規則可讓您對 Azure Cosmos DB IP 位址範圍的存取進行限制。 如果您想要允許對特定[區域](https://azure.microsoft.com/global-infrastructure/regions/)中的 Azure Cosmos DB 執行個體進行存取，您可以用下列格式指定區域： 

    AzureCosmosDB.<region name>

若要深入了解 NSG 標籤，請參閱[虛擬網路服務標籤](../virtual-network/security-overview.md#service-tags)一文。 
  
### <a name="what-is-relationship-between-an-ip-firewall-and-virtual-network-service-endpoint-capability"></a>IP 防火牆與虛擬網路服務端點功能之間的關係為何？  

這兩個功能彼此互補，可確保 Azure Cosmos DB 資產的獨立性並加以保護。 使用 IP 防火牆可確保靜態 IP 可以存取 Azure Cosmos DB 帳戶。  

### <a name="can-an-on-premises-devices-ip-address-that-is-connected-through-azure-virtual-network-gatewayvpn-or-express-route-gateway-access-azure-cosmos-db-account"></a>透過 Azure 虛擬網路閘道 (VPN) 或 ExpressRoute 閘道連線的內部部署裝置 IP 位址，可以存取 Azure Cosmos DB 帳戶嗎？  

應將內部部署裝置的 IP 位址或 IP 位址範圍新增至靜態 IP 清單，才能存取 Azure Cosmos DB 帳戶。  

### <a name="what-happens-if-you-delete-a-virtual-network-that-has-service-endpoint-setup-for-azure-cosmos-db"></a>如果刪除已設定 Azure Cosmos DB 服務端點的虛擬網路會發生什麼事？  

一旦刪除虛擬網路後，與 Azure Cosmos DB 相關聯的 ACL 資訊就會遭到刪除，並且一併移除虛擬網路與 Azure Cosmos DB 帳戶之間的互動。 

### <a name="what-happens-if-an-azure-cosmos-db-account-that-has-virtual-network-service-endpoint-enabled-is-deleted"></a>刪除已啟用虛擬網路服務端點的 Azure Cosmos DB 帳戶會發生什麼事？

子網路中與特定 Azure Cosmos DB 帳戶相關聯的中繼資料會遭到刪除。 而刪除使用的子網路和虛擬網路是終端使用者的責任。

### <a name="can-i-use-a-peered-virtual-network-to-create-service-endpoint-for-azure-cosmos-db"></a>我可以使用對等互連的虛擬網路來建立 Azure Cosmos DB 服務端點嗎？  

不可以，只有直接虛擬網路和其子網路可以建立 Azure Cosmos DB 服務端點。

### <a name="what-happens-to-the-source-ip-address-of-resource-like-virtual-machine-in-the-subnet-that-has-azure-cosmos-db-service-endpoint-enabled"></a>在啟用 Azure Cosmos DB 服務端點的子網路中，虛擬機器這類資源的來源 IP 位址會發生什麼事？

當虛擬網路服務端點在您虛擬網路的子網路中啟用時，其中資源的來源 IP 位址會從使用公用 IPV4 位址切換為使用 Azure 虛擬網路的私人位址，以便傳送流量至 Azure Cosmos DB。

### <a name="does-azure-cosmos-db-reside-in-the-azure-virtual-network--provided-by-the-customer"></a>Azure Cosmos DB 是位於客戶提供的 Azure 虛擬網路中嗎？  

Azure Cosmos DB 是使用公用 IP 位址的多租用戶服務。 當您使用服務端點功能對 Azure 虛擬網路的子網路限制存取權時，Azure Cosmos DB 會限制為只能透過指定 Azure 虛擬網路和其子網路進行存取。  Azure DB Cosmos 帳戶不在 Azure 虛擬網路中。 

### <a name="what-if-anything-will-be-logged-in-log-analyticsoms-if-it-is-enabled"></a>如果有任何項目記錄在已啟用的 Log Analytics/OMS 中會怎麼樣呢？  

Azure Cosmos DB 會針對 ACL 封鎖的要求，以狀態 403 推送記錄和 IP 位址 (不含最後一個八位元)。  

## <a name="next-steps"></a>後續步驟
若要設定 Azure Cosmos DB 的防火牆，請參閱[防火牆支援](firewall-support.md)一文。

