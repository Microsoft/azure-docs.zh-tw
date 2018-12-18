---
title: Azure Data Lake Storage Gen1 中的網路安全性 | Microsoft Docs
description: 了解 IP 防火牆和虛擬網路在 Azure Data Lake Storage Gen1 中的整合方式
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 10/09/2018
ms.author: elsung
ms.openlocfilehash: 0da5962bc0b48a387ee82a1db36099682e14bca3
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49168066"
---
# <a name="virtual-network-integration-for-azure-data-lake-storage-gen1---preview"></a>Azure Data Lake Storage Gen1 的虛擬網路整合 - 預覽

Azure Data Lake Storage Gen1 的虛擬網路整合簡介 (預覽階段)。 Vnet 整合可讓您將 Azure Data Lake Storage Gen1 帳戶限用特定的虛擬網路和子網路，以防止未經授權者存取這些帳戶。 現在，您可以設定 ADLS Gen1 帳戶，使其僅接受來自指定虛擬網路和子網路的流量，並封鎖來自任何其他位置的流量。 這有助於保護您的 ADLS 帳戶不受外部威脅入侵。

ADLS Gen1 的 VNet 整合會在虛擬網路與 Azure Active Directory 服務之間使用虛擬網路服務端點安全性，以在存取權杖中產生額外的安全性宣告。 這些宣告隨後將用來對您的 ADLS Gen1 帳戶驗證虛擬網路並允許存取。

> [!NOTE]
> 這是預覽技術，不建議將其用於生產環境中。
>
> 使用這些功能不會產生額外的相關費用。 您的帳戶將依據 ADLS Gen1 的標準費率 ([定價](https://azure.microsoft.com/pricing/details/data-lake-store/?cdn=disable))，針對您所使用的所有 Azure 服務 ([定價](https://azure.microsoft.com/pricing/#product-picker)) 計費。

## <a name="scenarios-for-vnet-integration-for-adls-gen1"></a>ADLS Gen1 的 VNET 整合案例

ADLS Gen1 VNet 整合可讓您限定只能從指定的虛擬網路和子網路存取您的 ADLS Gen1 帳戶。  Azure 中的其他 Vnet/VM 一旦鎖定為指定的 VNet 子網路，即無法存取您的帳戶。  在功能上，ADLS Gen1 VNet 整合可支援與[虛擬網路服務端點](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)的相同案例。  下列各節將詳述其間的一些主要差異。 

![ADLS Gen1 VNet 整合的案例圖](media/data-lake-store-network-security/scenario-diagram.png)

> [!NOTE]
> 在 VNet 規則以外也可使用現有的 IP 防火牆規則，以同時允許來自內部部署網路的存取。 

## <a name="optimal-routing-with-adls-gen1-vnet-integration"></a>ADLS Gen1 VNet 整合的最佳路由

VNet 服務端點的主要優點是您的虛擬網路具有[最佳路由](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview#key-benefits)。  若要對 ADLS Gen1 帳戶執行相同的路由最佳化，請使用下列從您的 VNet 到 ADLS Gen1 帳戶的[使用者定義路由](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview#user-defined)：

- **公用 IP 位址 ADLS** – 為您的目標 ADLS Gen1 帳戶使用公用 IP 位址。  您可以對 ADLS Gen1 帳戶進行 [DNS 名稱解析](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-connectivity-from-vnets#enabling-connectivity-to-azure-data-lake-storage-gen1-from-vms-with-restricted-connectivity)，以識別這些帳戶的 IP 位址。  請為每個位址建立個別的項目。

```azurecli
# Create a Route table for your resource group
az network route-table create --resource-group $RgName --name $RouteTableName

# Create Route Table Rules for ADLS Public IP Addresses
# There will be one rule per ADLS Public IP Addresses 
az network route-table route create --name toADLSregion1 --resource-group $RgName --route-table-name $RouteTableName --address-prefix <ADLS Public IP Address> --next-hop-type Internet

# Update the VNet and apply the newly created Route Table to it
az network vnet subnet update --vnet-name $VnetName --name $SubnetName --resource-group $RgName --route-table $RouteTableName
```

## <a name="data-exfiltration-from-the-customer-vnet"></a>客戶 VNet 的資料外洩

除了為 ADLS 帳戶進行虛擬網路存取的保護，您可能也會想要確定不會有任何資料外洩給未經授權的帳戶。

建議您在 VNet 中使用防火牆解決方案，根據目的地帳戶 URL 篩選輸出流量，並僅允許存取已授權的 ADLS Gen1 帳戶。

可用的選項包括：
- [Azure 防火牆](https://docs.microsoft.com/azure/firewall/overview)：您可以為 Vnet [部署及設定 Azure 防火牆](https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal)，並保護輸出 ADLS 流量，將其限定於已知且經過授權的帳戶 URL。
- [網路虛擬設備](https://azure.microsoft.com/solutions/network-appliances/)防火牆：如果您的系統管理員僅授權使用特定的商業防火牆供應商，您可以使用 Azure Marketplace 提供的 NVA 防火牆解決方案來執行相同的功能。

> [!NOTE]
> 在資料路徑中使用防火牆將會在資料路徑中導入額外的躍點，且端對端資料交換的網路效能可能會受到影響，包括可用的輸送量和連線延遲。 

## <a name="limitations"></a>限制
1.  HDInsight 叢集必須是在新增至預覽版之後新建的。  在啟用 ADLS Gen1 VNet 整合支援之前建立的叢集，必須要重新建立才可支援這項新功能。 
2.  在建立新的 HDInsight 叢集時，若選取已啟用 VNet 整合的 ADLS Gen1 帳戶，將會導致程序失敗。 您必須先停用 VNet 規則，或者您可以透過 ADLS 帳戶的 [防火牆與虛擬網路] 刀鋒視窗，[允許來自所有網路和服務的存取]。  如需詳細資訊，請參閱[例外狀況](##Exceptions)。
3.  ADLS Gen1 VNet 整合預覽不適用於 [Azure 資源的受控識別](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)。  
4.  在您已啟用 VNet 的 ADLS Gen1 帳戶中，檔案/資料夾資料無法從入口網站存取。  這包括從 VNET 內的 VM 存取，以及諸如使用資料總管的活動。  帳戶管理活動會繼續運作。  在您已啟用 VNET 的 ADLS 帳戶中，檔案/資料夾資料可透過所有非入口網站資源來存取：SDK 存取、PowerShell 指令碼、其他 Azure 服務 (若並非來自於入口網站) 等等。 

## <a name="configuration"></a>組態

### <a name="step1-configure-your-vnet-to-use-aad-service-endpoint"></a>步驟 1：設定 VNET 以使用 AAD 服務端點
1.  移至 Azure 入口網站並登入您的帳戶。 
2.  在您的訂用帳戶中[建立新的虛擬網路](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)，或瀏覽至現有的虛擬網路。  VNET 目前必須與 ADLS Gen 1 帳戶位於相同的區域中。 
3.  在 [虛擬網路] 刀鋒視窗中，選擇 [服務端點]。 
4.  按一下 [新增] 以新增服務端點。
![新增 VNet 服務端點](media/data-lake-store-network-security/config-vnet-1.png)
5.  選擇 **Microsoft.AzureActiveDirectory** 作為端點的服務。
![選取 Microsoft.AzureActiveDirectory 服務端點](media/data-lake-store-network-security/config-vnet-2.png)
6.  選擇要允許連線的子網路，然後按一下 [新增]。
![選取子網路](media/data-lake-store-network-security/config-vnet-3.png)
7.  新增服務端點可能需要 15 分鐘的時間。 新增之後，它會顯示在清單中。 請確認端點已顯示，且所有詳細資料皆如預期設定。 
![成功新增的服務端點](media/data-lake-store-network-security/config-vnet-4.png)

### <a name="step-2-set-up-the-allowed-vnetsubnet-for-your-adls-gen1-account"></a>步驟 2：為 ADLS Gen1 帳戶設定允許的 VNET/子網路
1.  設定 VNET 之後，請在您的訂用帳戶中[建立新的 Azure Data Lake Storage Gen1 帳戶](data-lake-store-get-started-portal.md#create-a-data-lake-storage-gen1-account)，或瀏覽至現有的 ADLS Gen1 帳戶。 ADLS Gen1 帳戶目前必須與 VNET 位於相同的區域中。 
2.  選擇 [防火牆與虛擬網路]。

  > [!NOTE]
  > 如果您在設定中未看到 [防火牆與虛擬網路]，請登出入口網站。 關閉瀏覽器。 清除瀏覽器快取。 重新啟動電腦並重試。

  ![將 VNet 規則新增至您的 ADLS 帳戶](media/data-lake-store-network-security/config-adls-1.png)
3.  選擇 [選取的網路]。 
4.  按一下 [新增現有的虛擬網路]。
  ![新增現有的虛擬網路](media/data-lake-store-network-security/config-adls-2.png)
5.  選擇要允許連線的 VNet 和子網路，然後按一下 [新增]。
  ![選擇 VNet 和子網路](media/data-lake-store-network-security/config-adls-3.png)
6.  確定 VNet 和子網路正確顯示於清單中，然後加以**儲存**。
  ![儲存新規則](media/data-lake-store-network-security/config-adls-4.png)

  > [!NOTE]
  > 設定儲存後可能需要 5 分鐘才會生效。

7.  [選擇性] 除了 VNet 和子網路以外，如果您想要允許來自特定 IP 位址的連線，您可以在相同頁面的 [防火牆] 區段中執行該設定。 

## <a name="exceptions"></a>例外狀況
在 [防火牆與虛擬網路] 刀鋒視窗上的 [例外狀況] 區域中，有兩個核取方塊可啟用從 Azure 上的一組服務和虛擬機器進行連線的功能。
![防火牆與虛擬網路例外狀況](media/data-lake-store-network-security/firewall-exceptions.png)
- [允許所有 Azure 服務存取此 Data Lake Storage Gen1 帳戶] 會允許所有的 Azure 服務 (例如 Azure Data Factory、事件中樞、所有 Azure VM 等等)  與您的 ADLS 帳戶進行通訊。

- [允許 Azure Data Lake Analytics 存取此 Data Lake Storage Gen1 帳戶] 會允許 Azure Data Lake Analytics 服務連線到此 ADLS 帳戶。 

建議您將這些例外狀況設定為關閉，且只有在需要從 VNet 以外的其他服務進行連線時，才加以開啟。
