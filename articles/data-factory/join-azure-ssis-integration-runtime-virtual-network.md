---
title: 將 Azure-SSIS 整合執行階段加入虛擬網路 | Microsoft Docs
description: 了解如何將 Azure-SSIS 整合執行階段加入 Azure 虛擬網路。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/27/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: a9a4b7728eff3057b9677d12df51cc8c477744ca
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46953934"
---
# <a name="join-an-azure-ssis-integration-runtime-to-a-virtual-network"></a>將 Azure-SSIS 整合執行階段加入虛擬網路
在下列案例中，將 Azure-SSIS 整合執行階段 (IR) 加入 Azure 虛擬網路： 

- 您想要從 Azure-SSIS 整合執行階段上執行的 SSIS 套件連線至內部部署資料存放區。 

- 您要將 SQL Server Integration Services (SSIS) 目錄資料庫裝載於具有虛擬網路服務端點/受控執行個體的 Azure SQL Database 中。 

 Azure Data Factory 可讓您將 Azure-SSIS 整合執行階段加入透過傳統部署模型或 Azure Resource Manager 部署模型建立的虛擬網路。 

## <a name="access-to-on-premises-data-stores"></a>存取內部部署資料存放區
如果 SSIS 套件只會存取公用雲端資料存放區，則您不需要將 Azure-SSIS IR 加入虛擬網路中。 如果 SSIS 套件會存取內部部署資料存放區，則您必須將 Azure-SSIS IR 加入連線至內部部署網路的虛擬網路中。 

以下是一些需要注意的重要事項： 

- 如果目前沒有虛擬網路連線至您的內部部署網路，請先建立 Azure-SSIS 整合執行階段要加入的 [Azure Resource Manager 虛擬網路](../virtual-network/quick-create-portal.md#create-a-virtual-network)或[傳統虛擬網路](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。 然後，設定從該虛擬網路到內部部署網路的站對站 [VPN 閘道連線](../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md)或 [ExpressRoute](../expressroute/expressroute-howto-linkvnet-classic.md) 連線。 

- 如果在與 Azure-SSIS 整合執行階段相同的位置中有現有的 Azure Resource Manager 或傳統虛擬網路連線至您的內部部署網路，您可以將整合執行階段加入該虛擬網路。 

- 如果在與 Azure-SSIS 整合執行階段不同的位置中有現有的傳統虛擬網路連線至您的內部部署網路，您可以先建立 Azure-SSIS IR 要加入的[傳統虛擬網路](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)。 然後，設定[傳統對傳統虛擬網路](../vpn-gateway/vpn-gateway-howto-vnet-vnet-portal-classic.md)連線。 或者，您可以建立 [Azure Resource Manager 虛擬網路](../virtual-network/quick-create-portal.md#create-a-virtual-network)，讓您的 Azure-SSIS 整合執行階段加入。 然後，設定[傳統對 Azure Resource Manager 虛擬網路](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)連線。 
 
- 如果在與 Azure-SSIS IR 不同的位置中有現有的 Azure Resource Manager 虛擬網路連線至您的內部部署網路，您可以先建立 Azure-SSIS IR 要加入的 [Azure Resource Manager 虛擬網路](../virtual-network/quick-create-portal.md##create-a-virtual-network)。 然後，設定 Azure Resource Manager 對 Azure Resource Manager 虛擬網路連線。 或者，您可以建立[傳統虛擬網路](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)，讓您的 Azure-SSIS IR 加入。 然後，設定[傳統對 Azure Resource Manager 虛擬網路](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)連線。 

## <a name="host-the-ssis-catalog-database-in-azure-sql-database-with-virtual-network-service-endpointsmanaged-instance"></a>將 SSIS 目錄資料庫裝載於具有虛擬網路服務端點/受控執行個體的 Azure SQL Database 中
如果 SSIS 目錄已裝載於具有虛擬網路服務端點或受控執行個體的 Azure SQL Database 中，您可以將 Azure-SSIS IR 加入至： 

- 相同虛擬網路 
- 具有網路對網路連線的虛擬網路，不同於處理具有虛擬網路服務端點/受控執行個體的 Azure SQL Database 時所用的虛擬網路 

如果您將 Azure-SSIS IR 加入至受控執行個體所在的虛擬網路上，請確定 Azure-SSIS IR 和受控執行個體分屬不同的子網路。 如果 Azure-SSIS IR 並不是加入至受控執行個體所在的虛擬網路，我們建議虛擬網路對等互連 (限制為相同的區域) 或虛擬網路對虛擬網路連線。 請參閱[將您的應用程式連線到 Azure SQL Database 受控執行個體](../sql-database/sql-database-managed-instance-connect-app.md)。

虛擬網路可透過傳統部署模型或 Azure Resource Manager 部署模型來部署。

下列各節提供更多詳細資料。 

## <a name="requirements-for-virtual-network-configuration"></a>虛擬網路設定的需求
-   在裝載 Azure-SSIS IR 的虛擬網路子網路訂用帳戶之下，確認 `Microsoft.Batch` 是已註冊的提供者。 若您使用傳統虛擬網路，也請為該虛擬網路將 `MicrosoftAzureBatch` 加入「傳統虛擬機器參與者」角色。 

-   選取要裝載 Azure-SSIS IR 的適當子網路。 請參閱[選取子網路](#subnet)。 

-   若在虛擬網路上使用自己的網域名稱服務 (DNS) 伺服器，請參閱[網域名稱服務伺服器](#dns_server)。 

-   若在子網路上使用「網路安全性群組」(NSG)，請參閱[網路安全性群組](#nsg) 

-   若使用 Azure Express Route 或設定使用者定義的路由 (UDR)，請參閱[使用 Azure ExpressRoute 或使用者定義的路由](#route)。 

-   請確認虛擬網路的資源群組可以建立及刪除特定的 Azure 網路資源。 請參閱[資源群組的需求](#resource-group)。 

### <a name="subnet"></a> 選取子網路
-   請勿選取 GatewaySubnet 來部署 Azure-SSIS Integration Runtime，因為它是虛擬網路閘道所專用。 

-   確定您選取的子網路有足夠的可用位址空間，以供 Azure-SSIS IR 使用。 在可用的 IP 位址中保留至少 2 * IR 節點數目。 Azure 會在每個子網路中保留一些 IP 位址，但這些位址無法使用。 子網路的第一個和最後一個 IP 位址會保留給相容的通訊協定，以及用於 Azure 服務的額外 3 個位址。 如需詳細資訊，請參閱[在這些子網路內使用 IP 位址是否有任何限制？](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets) 

-   請勿使用由其他 Azure 服務 (例如 SQL Database 受控執行個體、App Service 等) 所獨佔的子網路。 

### <a name="dns_server"></a> 網域名稱服務伺服器 
若需要在 Azure-SSIS 整合執行階段所加入的虛擬網路中使用自己的網域名稱服務 (DNS) 伺服器，請確認該伺服器可以解析公用 Azure 主機名稱 (例如，`<your storage account>.blob.core.windows.net` Azure 儲存體 blob 名稱)。 

以下為建議步驟： 

-   設定自訂 DNS 將要求轉送至 Azure DNS。 您可以在自己的 DNS 伺服器上，將無法解析的 DNS 記錄轉送至 Azure 遞迴解析程式的 IP 位址 (168.63.129.16)。 

-   將虛擬網路的主要 DNS 設定為自訂 DNS，並將次要 DNS 設定為 Azure DNS。 將 Azure 遞迴解析程式的 IP 位址 (168.63.129.16) 註冊為次要 DNS 伺服器，以便您自己的 DNS 伺服器無法使用時可備援。 

如需詳細資訊，請參閱[使用自有 DNS 伺服器的名稱解析](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)。 

### <a name="nsg"></a> 網路安全性群組
如果您需要在 Azure-SSIS 整合執行階段所使用的子網路上實作網路安全性群組 (NSG)，請允許通過下列連接埠的輸入/輸出流量： 

| 方向 | 傳輸通訊協定 | 來源 | 來源連接埠範圍 | 目的地 | 目的地連接埠範圍 | 註解 |
|---|---|---|---|---|---|---|
| 輸入 | TCP | Internet | * | VirtualNetwork | 29876、29877 (如果您將 IR 加入 Azure Resource Manager 虛擬網路) <br/><br/>10100、20100、30100 (如果您將 IR 加入傳統虛擬網路)| Data Factory 服務會使用這些連接埠與虛擬網路中的 Azure SSIS 整合執行階段節點進行通訊。 <br/><br/> 無論您是否有建立子網路層級的 NSG，Data Factory 一律會在連結至 Azure-SSIS IR 主控虛擬機器的網路介面卡 (NIC) 的層級上設定 NSG。 該 NIC 層級的 NSG 僅允許來自指定連接埠上 Data Factory IP 位址的輸入流量。 即使您在子網路層級上對網際網路流量開啟這些連接埠，只要流量不是來自 Data Factory 的 IP 位址，就會在 NIC 層級上遭到封鎖。 |
| 輸出 | TCP | VirtualNetwork | * | Internet | 443 | 虛擬網路中的 Azure-SSIS 整合執行階段節點會使用此連接埠來存取 Azure 服務，例如 Azure 儲存體和 Azure 事件中樞。 |
| 輸出 | TCP | VirtualNetwork | * | 網際網路或 Sql | 1433、11000-11999、14000-14999 | 您在虛擬網路中的 Azure-SSIS 整合執行階段節點會使用這些連接埠，來存取 Azure SQL Database 伺服器所裝載的 SSISDB，此用途不適用於受控執行個體所裝載的 SSISDB。 |
||||||||

### <a name="route"></a> 使用 Azure ExpressRoute 或使用者定義的路由
您可以將 [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) 線路連線至虛擬網路基礎結構，以將內部部署網路延伸至 Azure。 

常見的設定是使用強制通道 (通告 BGP 路由，0.0.0.0/0 至虛擬網路)，其可將虛擬網路流程的輸出網際網路流量強制傳送到內部網路設備，以用於檢查和記錄。 此流量會中斷虛擬網路中的 Azure-SSIS IR 與相依 Azure Data Factory 服務之間的連線能力。 解決方法是在包含 Azure-SSIS IR 的子網路上，定義一個 (或多個) [使用者定義路由 (UDR)](../virtual-network/virtual-networks-udr-overview.md)。 UDR 會定義要代替 BGP 路由使用的子網路特定路由。 

或者，您可以定義使用者定義的路由 (UDR)，將來自裝載 Azure-SSIS IR 所在子網路的對外網際網路流量，強制導向裝載虛擬網路設備為防火牆或 DMZ 主機的另一個子網路，以進行檢查與記錄。 

在這兩種情況下，請都在裝載 Azure-SSIS IR 的子網路上，使用下一個躍點類型為 **Internet** (網際網路) 的 0.0.0.0/0 路由，以便 Data Factory 服務與 Azure-SSIS IS IR 之間能夠順利通訊。 

![新增路由](media/join-azure-ssis-integration-runtime-virtual-network/add-route-for-vnet.png)

如果您擔心會遺失檢查來自該子網路之輸出網際網路流量的能力，您也可以在子網路上加入 NSG 規則，來將輸出目的地限制為 [Azure 資料中心 IP 位址](https://www.microsoft.com/download/details.aspx?id=41653) \(英文\)。 

如需範例，請參閱[這個 PowerShell 指令碼](https://gallery.technet.microsoft.com/scriptcenter/Adds-Azure-Datacenter-IP-dbeebe0c) \(英文\)。 您必須每週執行這個指令碼，讓這份 Azure 資料中心 IP 位址清單保持在最新狀態。 

### <a name="resource-group"></a> 資源群組的需求
-   Azure-SSIS IR 需要在與虛擬網路相同的資源群組下，建立特定的網路資源。 這些資源包括下列各項：
    -   Azure 負載平衡器，名稱為 *<Guid>-azurebatch-cloudserviceloadbalancer*。
    -   Azure 公用 IP 位址，名稱為 *<Guid>-azurebatch-cloudservicepublicip*。
    -   網路工作安全性群組，名稱為 *<Guid>-azurebatch-cloudservicenetworksecuritygroup*。 

-   請確認在虛擬網路所屬的資源群組或訂用帳戶上，沒有任何鎖定的資源。 如果您設定了唯讀鎖定或刪除鎖定，則啟動或停止 IR 可能會失敗或停止回應。 

-   請確認沒有任何 Azure 原則會禁止在虛擬網路所屬資源群組或訂用帳戶之下建立下列資源： 
    -   Microsoft.Network/LoadBalancers 
    -   Microsoft.Network/NetworkSecurityGroups 
    -   Microsoft.Network/PublicIPAddresses 

## <a name="azure-portal-data-factory-ui"></a>Azure 入口網站 (Data Factory UI)
本節說明如何使用 Azure 入口網站及 Data Factory UI，將現有的 Azure SSIS 執行階段加入虛擬網路 (傳統或 Azure Resource Manager)。 首先，在將 Azure SSIS IR 加入虛擬網路之前，必須先適當設定虛擬網路。 根據您的虛擬網路類型 (傳統或 Azure Resource Manager)，執行下列兩節的其中一節。 然後，繼續執行第三節，將您的 Azure SSIS IR 加入虛擬網路。 

### <a name="use-the-portal-to-configure-an-azure-resource-manager-virtual-network"></a>使用入口網站設定 Azure Resource Manager 虛擬網路
您必須先設定虛擬網路，才能將 Azure-SSIS IR 加入虛擬網路。 

1. 啟動 Microsoft Edge 或 Google Chrome。 目前只有這些網頁瀏覽器支援 Data Factory UI。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

1. 選取 [更多服務]。 篩選並選取 [虛擬網路]。 

1. 篩選並選取清單中的 [虛擬網路]。 

1. 在 [虛擬網路] 頁面上，選取 [屬性]。 

1. 選取 [資源識別碼] 的 [複製] 按鈕，將虛擬網路的資源識別碼複製到剪貼簿。 將剪貼簿中的識別碼儲存至 OneNote 或檔案中。 

1. 選取左側功能表上的 [子網路]。 確定 [可用的位址] 數目大於 Azure-SSIS 整合執行階段中的節點數。 

1. 確認已在具有虛擬網路的 Azure 訂用帳戶中註冊 Azure Batch 提供者。 或者，註冊 Azure Batch 提供者。 如果您的訂用帳戶中已經有 Azure Batch 帳戶，則會註冊 Azure Batch 的訂用帳戶。 (若在 Data Factory 入口網站中建立 Azure-SSIS IR，便會自動註冊 Azure Batch 提供者)。 

   a. 在 Azure 入口網站中，選取左側功能表上的 [訂用帳戶]。 

   b. 選取您的訂用帳戶。 

   c. 選取左側的 [資源提供者]，並確認 **Microsoft.Batch** 是已註冊的提供者。 

   ![確認「已註冊」狀態](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

   如果您在清單中未看到 **Microsoft.Batch**，若要進行註冊，請在訂用帳戶中[建立空白 Azure Batch 帳戶](../batch/batch-account-create-portal.md)。 您可以稍後刪除它。 

### <a name="use-the-portal-to-configure-a-classic-virtual-network"></a>使用入口網站設定傳統虛擬網路
您必須先設定虛擬網路，才能將 Azure-SSIS IR 加入虛擬網路。 

1. 啟動 Microsoft Edge 或 Google Chrome。 目前只有這些網頁瀏覽器支援 Data Factory UI。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

1. 選取 [更多服務]。 篩選並選取 [虛擬網路 (傳統)]。 

1. 篩選並選取清單中的 [虛擬網路]。 

1. 在 [虛擬網路 (傳統)] 頁面上，選取 [屬性]。 

   ![傳統虛擬網路資源識別碼](media/join-azure-ssis-integration-runtime-virtual-network/classic-vnet-resource-id.png)

1. 選取 [資源識別碼] 的 [複製] 按鈕，將傳統網路的資源識別碼複製到剪貼簿。 將剪貼簿中的識別碼儲存至 OneNote 或檔案中。 

1. 選取左側功能表上的 [子網路]。 確定 [可用的位址] 數目大於 Azure-SSIS 整合執行階段中的節點數。 

   ![虛擬網路中的可用位址數目](media/join-azure-ssis-integration-runtime-virtual-network/number-of-available-addresses.png)

1. 將 **MicrosoftAzureBatch** 加入虛擬網路的**傳統虛擬機器參與者**角色。 

    a. 選取左側功能表上的 [存取控制 (IAM)]，然後選取工具列上的 [新增]。 

    ![[存取控制] 和 [新增] 按鈕](media/join-azure-ssis-integration-runtime-virtual-network/access-control-add.png)

    b. 在 [新增權限] 頁面上，針對 [角色] 選取 [傳統虛擬機器參與者]。 在 [選取] 方塊中複製 **ddbf3205-c6bd-46ae-8127-60eb93363864**，然後從搜尋結果清單中選取 [Microsoft Azure Batch]。 

    ![[新增權限] 頁面上的搜尋結果](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-to-vm-contributor.png)

    c. 選取 [儲存] 以儲存設定並關閉頁面。 

    ![儲存存取設定](media/join-azure-ssis-integration-runtime-virtual-network/save-access-settings.png)

    d. 確認您在參與者清單中看到 **Microsoft Azure Batch**。 

    ![確認 Azure Batch 存取](media/join-azure-ssis-integration-runtime-virtual-network/azure-batch-in-list.png)

1. 確認已在具有虛擬網路的 Azure 訂用帳戶中註冊 Azure Batch 提供者。 或者，註冊 Azure Batch 提供者。 如果您的訂用帳戶中已經有 Azure Batch 帳戶，則會註冊 Azure Batch 的訂用帳戶。 (若在 Data Factory 入口網站中建立 Azure-SSIS IR，便會自動註冊 Azure Batch 提供者)。 

   a. 在 Azure 入口網站中，選取左側功能表上的 [訂用帳戶]。 

   b. 選取您的訂用帳戶。 

   c. 選取左側的 [資源提供者]，並確認 **Microsoft.Batch** 是已註冊的提供者。 

   ![確認「已註冊」狀態](media/join-azure-ssis-integration-runtime-virtual-network/batch-registered-confirmation.png)

   如果您在清單中未看到 **Microsoft.Batch**，若要進行註冊，請在訂用帳戶中[建立空白 Azure Batch 帳戶](../batch/batch-account-create-portal.md)。 您可以稍後刪除它。 

### <a name="join-the-azure-ssis-ir-to-a-virtual-network"></a>將 Azure-SSIS IR 加入虛擬網路
1. 啟動 Microsoft Edge 或 Google Chrome。 目前只有這些網頁瀏覽器支援 Data Factory UI。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取左側功能表中的 [資料處理站]。 如果在功能表上未看到 [資料處理站]，請選取 [更多服務]，然後選取 [智慧 + 分析] 區段中的 [資料處理站]。 

   ![資料處理站清單](media/join-azure-ssis-integration-runtime-virtual-network/data-factories-list.png)

1. 在清單中選取具有 Azure-SSIS 整合執行階段的資料處理站。 您會看到資料處理站的首頁。 選取 [編寫與部署] 圖格。 您會在個別的索引標籤上看到 Data Factory UI。 

   ![Data Factory 首頁](media/join-azure-ssis-integration-runtime-virtual-network/data-factory-home-page.png)

1. 在 Data Factory UI 中，切換至 [編輯] 索引標籤，選取 [連線]，然後切換至 [整合執行階段] 索引標籤。 

   ![[整合執行階段] 索引標籤](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtimes-tab.png)

1. 如果您的 Azure-SSIS 整合執行階段正在執行，在整合執行階段清單中，請在 [動作] 資料行中針對您的 Azure SSIS IR 選取 [停止] 按鈕。 您必須停止整合執行階段才能進行編輯。 

   ![停止 IR](media/join-azure-ssis-integration-runtime-virtual-network/stop-ir-button.png)

1. 在整合執行階段清單中，在 [動作] 資料行中針對您的 Azure-SSIS IR 選取 [編輯] 按鈕。 

   ![編輯整合執行階段](media/join-azure-ssis-integration-runtime-virtual-network/integration-runtime-edit.png)

1. 在 [整合執行階段設定] 視窗的 [一般設定] 頁面上，選取 [下一步]。 

   ![IR 設定的一般設定](media/join-azure-ssis-integration-runtime-virtual-network/ir-setup-general-settings.png)

1. 在 [SQL 設定] 頁面上，輸入系統管理員密碼，然後選取 [下一步]。 

   ![IR 設定的 SQL Server 設定](media/join-azure-ssis-integration-runtime-virtual-network/ir-setup-sql-settings.png)

1. 在 [進階設定] 頁面上，執行下列動作： 

   a. 選取 [選取您的 Azure-SSIS 整合執行階段要加入的 VNet 並允許 Azure 服務設定 VNet 權限/設定] 核取方塊。 

   b. 針對 [類型]，選取虛擬網路是傳統虛擬網路還是 Azure Resource Manager 虛擬網路。 

   c. 針對 [VNet 名稱]，選取您的虛擬網路。 

   d. 針對 [子網路名稱]，選取您在虛擬網路中的子網路。 

   e. 按一下 [VNet 驗證]，如果成功的話，按一下 [更新]。 

   ![IR 設定的進階設定](media/join-azure-ssis-integration-runtime-virtual-network/ir-setup-advanced-settings.png)

1. 現在，您可以針對您的 Azure-SSIS IR 使用 [動作] 資料行中的 [啟動] 按鈕，以啟動 IR。 啟動 Azure-SSIS IR 需要約 20 至 30 分鐘。 

## <a name="azure-powershell"></a>Azure PowerShell

### <a name="configure-a-virtual-network"></a>設定虛擬網路
您必須先設定虛擬網路，才能將 Azure-SSIS IR 加入虛擬網路。 若要自動設定虛擬網路權限/設定，讓您的 Azure-SSIS 整合執行階段加入虛擬網路，請新增下列指令碼：

```powershell
# Make sure to run this script against the subscription to which the virtual network belongs.
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzureRmADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzureRmRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="create-an-azure-ssis-ir-and-join-it-to-a-virtual-network"></a>建立 Azure-SSIS IR 並將其加入虛擬網路
您可以建立 Azure-SSIS IR，同時將其加入虛擬網路。 在完整的指令碼和指示，請參閱[建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md#azure-powershell)。

### <a name="join-an-existing-azure-ssis-ir-to-a-virtual-network"></a>將現有的 Azure-SSIS IR 加入虛擬網路
[建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)一文中的指令碼會說明如何以相同的指令碼建立 Azure-SSIS IR 並將其加入虛擬網路。 如果您有現有的 Azure-SSIS IR，請執行下列步驟將其加入虛擬網路： 
1. 停止 Azure-SSIS IR。 
1. 設定要加入虛擬網路的 Azure-SSIS IR。 
1. 啟動 Azure-SSIS IR。 

### <a name="define-the-variables"></a>定義變數
```powershell
$ResourceGroupName = "<your Azure resource group name>"
$DataFactoryName = "<your Data Factory name>" 
$AzureSSISName = "<your Azure-SSIS IR name>"
# Specify the information about your classic or Azure Resource Manager virtual network.
$VnetId = "<your Azure virtual network resource ID>"
$SubnetName = "<the name of subnet in your virtual network>"
```

### <a name="stop-the-azure-ssis-ir"></a>停止 Azure-SSIS IR
必須先停止 Azure-SSIS 整合執行階段，才能將其加入虛擬網路。 此命令會釋出其所有節點並停止計費：

```powershell
Stop-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Force 
```

### <a name="configure-virtual-network-settings-for-the-azure-ssis-ir-to-join"></a>進行虛擬網路設定讓 Azure-SSIS IR 可以加入
```powershell
# Make sure to run this script against the subscription to which the virtual network belongs.
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzureRmADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
        Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign VM contributor role to Microsoft.Batch
        New-AzureRmRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="configure-the-azure-ssis-ir"></a>設定 Azure-SSIS IR
若要設定要加入虛擬網路的 Azure-SSIS 整合執行階段，請執行 `Set-AzureRmDataFactoryV2IntegrationRuntime` 命令： 

```powershell
Set-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                           -DataFactoryName $DataFactoryName `
                                           -Name $AzureSSISName `
                                           -Type Managed `
                                           -VnetId $VnetId `
                                           -Subnet $SubnetName
```

### <a name="start-the-azure-ssis-ir"></a>啟動 Azure-SSIS IR
若要啟動 Azure-SSIS 整合執行階段，請執行下列命令： 

```powershell
Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force

```

此命令約需要 20 到 30 分鐘才能完成。

## <a name="next-steps"></a>後續步驟
如需 Azure-SSIS 執行階段的詳細資訊，請參閱下列主題： 
- [Azure-SSIS 整合執行階段](concepts-integration-runtime.md#azure-ssis-integration-runtime)。 此文章提供整合執行階段的一般概念性資訊，包括 Azure-SSIS IR。 
- [教學課程：將 SSIS 套件部署至 Azure](tutorial-create-azure-ssis-runtime-portal.md)。 此文章將逐步說明如何建立 Azure-SSIS IR。 它會使用 Azure SQL Database 來裝載 SSIS 目錄。 
- [建立 Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md)。 此文章會詳述教學課程，並提供使用具有虛擬網路服務端點/受控執行個體的 Azure SQL Database 來裝載 SSIS 目錄並將 IR 加入虛擬網路的指示。 
- [監視 Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime). 此文章示範如何在傳回的資訊中擷取 Azure-SSIS IR 的相關資訊和狀態描述。 
- [管理 Azure-SSIS IR](manage-azure-ssis-integration-runtime.md). 此文章示範如何停止、啟動或移除 Azure-SSIS IR。 此外也會說明如何藉由新增節點來相應放大 Azure-SSIS IR。 
