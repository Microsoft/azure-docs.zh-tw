---
title: Contoso - 設定移轉基礎結構 | Microsoft Docs
description: 了解 Contoso 如何設定移轉至 Azure 的 Azure 基礎結構。
services: azure-migrate
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/26/2018
ms.author: raynew
ms.openlocfilehash: bf2562f4772d20f099faec94794e0e908e6636d5
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227532"
---
# <a name="contoso---deploy-a-migration-infrastructure"></a>Contoso - 部署移轉基礎結構

在本文中，Contoso 會準備其內部部署基礎結構以便進行移轉，並設定 Azure 基礎結構，以準備進行移轉，以及在混合式環境中執行業務。

- 這是 Contoso 專用的範例架構。
- 您是否需要此文章所說明的所有元素，取決於您的移轉策略。 例如，如果您只是要在 Azure 中建置雲端原生應用程式，則可能需要較不複雜的網路結構。

此文章是一系列文章中的一篇，描述 Contoso 這家虛構的公司如何將其內部部署資源移轉到 Microsoft Azure 雲端。 整個系列文章中包含背景資訊和一系列部署案例，說明如何設定移轉基礎結構、評定內部部署資源移轉的適合性，以及執行不同類型的移轉。 案例的複雜性會提高。 隨著時間的推移，此系列會新增其他文章。


**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
文章 2：部署 Azure 基礎結構 | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 此文章
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會使用 Site Recovery 來移轉應用程式 VM，以移轉應用程式，以及使用資料庫移轉服務，將應用程式資料庫移轉至受到 AlwaysOn 可用性群組保護的 SQL Server 叢集。 | 可用
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業。 | 可用
[文章 8：在 Azure VM 和適用於 MySQL 的 Azure 資料庫上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會透過使用 Site Recovery 將其 Linux osTicket 應用程式移轉到 Azure VM。 它會使用 MySQL Workbench 將應用程式資料庫移轉到適用於 MySQL 的 Azure 資料庫。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL Database 中重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將其 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體。 | 可用    
[文章 10：在 Azure Web 應用程式和適用於 MySQL 的 Azure 資料庫中重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會將其 Linux osTicket 應用程式移轉到多個網站上的 Azure Web 應用程式。 Web 應用程式會與 GitHub 整合以提供持續傳遞能力。 其會將應用程式資料庫移轉到適用於 MySQL 的 Azure 資料庫執行個體。 | 可用
[文章 11：在 Azure DevOps Services 上重構 Team Foundation Server](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 中重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的應用程式資料庫。 | 可用    
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會透過使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用 

在此文章中，Contoso 會設定為了完成所有移轉案例，所需的所有基礎結構元素。 


## <a name="overview"></a>概觀

Contoso 必須先將 Azure 基礎結構準備就緒，才能移轉到 Azure。  Contoso 通常有五大領域的事項需要考量：

**步驟 1：Azure 訂用帳戶**：Contoso 會如何購買 Azure，以及與 Azure 平台和服務互動？  
**步驟 2：混合式身分識別**：在移轉之後，其會如何管理及控制對內部部署和 Azure 資源的存取？ Contoso 會如何將身分識別管理擴充或移至雲端？  
**步驟 3：災害復原和恢復**：Contoso 會如何確保其應用程式和基礎結構在中斷和嚴重損壞發生後具有恢復能力？  
**步驟 4：網路**：Contoso 應如何設計網路基礎結構，並建立其內部部署資料中心與 Azure 之間的連線？  
**步驟 5：安全性**：其會如何保護其混合式/Azure 部署？  
**步驟 6：控管**：Contoso 會如何使部署符合安全性和控管需求？

## <a name="before-you-start"></a>開始之前

在我們開始探討基礎結構之前，您可以先閱讀與此文章中討論的 Azure 功能有關的一些背景資訊：

- 有一些選項可用來購買 Azure 存取權，包括預付型方案、Enterprise 合約 (EA)、Microsoft 轉銷商所提供的 Open 授權，或 Microsoft 合作夥伴 (稱為「雲端解決方案提供者」(CSP)) 所提供的 Open 授權。 請了解[購買選項](https://azure.microsoft.com/pricing/purchase-options/)，並閱讀如何[組織 Azure 訂用帳戶](https://azure.microsoft.com/blog/organizing-subscriptions-and-resource-groups-within-the-enterprise/)的相關資訊。
- 取得 Azure [身分識別和存取管理](https://www.microsoft.com/trustcenter/security/identity)的概觀。 特別是，請了解 [Azure AD 以及將內部部署 AD 擴充至雲端](https://docs.microsoft.com/azure/active-directory/identity-fundamentals)的相關資訊。 您可以下載關於[混合式環境中的身分識別和存取管理 (IAM)](https://azure.microsoft.com/resources/hybrid-cloud-identity/) 的實用電子書。
- Azure 透過混合式連線的選項提供穩健的網路基礎結構。 請取得[網路功能和網路存取控制](https://docs.microsoft.com/azure/security/security-network-overview)的概觀。
- 取得 [Azure 安全性](https://docs.microsoft.com/azure/security/azure-security)的簡介，並閱讀如何建立[控管](https://docs.microsoft.com/azure/security/governance-in-azure)計劃的相關資訊。


## <a name="on-premises-architecture"></a>內部部署架構

下圖顯示目前的 Contoso 內部部署基礎結構。

 ![Contoso 架構](./media/contoso-migration-infrastructure/contoso-architecture.png)  

- Contoso 有一個位於美國東部紐約市的主要資料中心。
- 在全美另有三家地區性分公司。
- 主要資料中心透過光纖都會乙太網路連線 (500 mbps) 連到網際網路。
- 每家分公司皆使用企業級連線從本機連到網際網路，並透過 IPSec VPN 通道連回主要資料中心。 這可讓整個網路永久連線，並將網際網路連線最佳化。
- 主要資料中心已透過 VMware 完全虛擬化。 Contoso 有兩部 ESXi 6.5 虛擬化主機，均由 vCenter Server 6.5 管理。
- Contoso 使用 Active Directory 進行身分識別管理，並使用內部網路上的 DNS 伺服器。
- 資料中心的網域控制站會在 VMware VM 上執行。 地區分公司的網域控制站會在實體伺服器上執行。


## <a name="step-1-buy-and-subscribe-to-azure"></a>步驟 1：購買和訂閱 Azure

Contoso 必須了解如何購買 Azure、如何建構訂用帳戶，以及如何為服務和資源授權。

### <a name="buy-azure"></a>購買 Azure

Contoso 將使用 [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)。 Contoso 須向 Azure 提前預付承諾用量，如此即可獲得絕佳權益，包括彈性的計費選項和最優惠的價格。

- Contoso 估算將須支付多少 Azure 年費。 Contoso 在簽署合約時，即已支付第一年的所有費用。
- Contoso 必須在年度結束前用完所有承諾用量，否則將會失去那些金額。
- 如果 Contoso 因故超出其承諾用量和花費，Microsoft 將會就差額對他們開立發票。
- 因前述承諾用量而產生的任何費用，都會採用 Contoso 合約中所載的相同費率。 超量不會有任何罰款。

### <a name="manage-subscriptions"></a>管理訂用帳戶

向 Azure 支付費用後，Contoso 必須了解如何管理 Azure 訂用帳戶。 Contoso 具有 EA，因此在可設定的 Azure 訂用帳戶數目方面不受限制。

- 「Azure Enterprise 註冊」會定義 Azure 服務在公司內的形式和用途，並定義核心治理結構。
- 在第一個步驟中，Contoso 決定 Enterprise 註冊的結構 (稱為企業 Scaffold)。 Contoso 使用了[此文章](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-subscription-governance)來協助其了解和設計 Scaffold。
- 現在，Contoso 已決定使用函式型方法來管理訂用帳戶。
    - 在企業內，其會以單一 IT 部門來控制 Azure 預算。 這將是唯一具有訂用帳戶的群組。
    - Contoso 會在未來擴充此模型，讓其他公司群組也可加入 Enterprise 註冊的部門。
    - 在 IT 部門內，Contoso 建構了兩個訂用帳戶：生產和開發。
    - 如果 Contoso 在未來需要其他訂用帳戶，便需要管理這些訂用帳戶的存取、原則和合規性。 為此，Contoso 必須將 [Azure 管理群組](https://docs.microsoft.com/azure/azure-resource-manager/management-groups-overview)加入作為訂用帳戶上的額外層級。

    ![企業結構](./media/contoso-migration-infrastructure/enterprise-structure.png) 

### <a name="examine-licensing"></a>檢查授權

在設定訂用帳戶後，Contoso 可以查看 Microsoft 授權。 授權策略會取決於 Contoso 要移轉到 Azure 的資源，以及他們選取和部署 Azure VM 與服務的方式。 

#### <a name="azure-hybrid-benefit"></a>Azure Hybrid Benefit

在 Azure 中部署 VM 時，標準映像會包含將依分鐘數針對使用的軟體向 Contoso 收費的授權。 不過，Contoso 一直是 Microsoft 的長期客戶，且已保有 EA 和附有「軟體保證」(SA) 的 Open 授權。 

Azure Hybrid Benefit 會讓 Contoso 能夠轉換或重複使用 Windows Server Datacenter 和「軟體保證」所涵蓋的標準版授權，來節省 Azure VM 和 SQL Server 工作負載成本，藉此為 Contoso 的移轉提供符合成本效益的方法。 這將使 Contoso 能夠根據 VM 和 SQL Server 的計算費率支付較低費用。 [深入了解](https://azure.microsoft.com/pricing/hybrid-benefit/)。


#### <a name="license-mobility"></a>授權流動性

透過 SA 的授權流動性，可讓 Contoso 這樣的 Microsoft 大量授權客戶在 Azure 上靈活地部署具有 SA 效力的伺服器應用程式。 這樣就不需要購買新的授權。 現有的授權不會產生相關行動費用，可輕易部署在 Azure 中。 [深入了解](https://azure.microsoft.com/pricing/license-mobility/)。

#### <a name="reserve-instances-for-predictable-workloads"></a>保留執行個體供可預測的工作負載使用

執行 VM 時一律需要用到的，就是可預測的工作負載。 例如，SAP ERP 系統之類的企業營運應用程式。  另一方面，可能有變化的則屬於無法預測的工作負載。 例如，在高需求期間開啟，並在非尖峰時間關閉的 VM。

![保留的執行個體](./media/contoso-migration-infrastructure/reserved-instance.png) 

使用保留的執行個體作為必須長時間維護的特定 VM 執行個體，Contoso 將可同時享有折扣和高優先順序的容量。 在 [Azure 保留執行個體](https://azure.microsoft.com/pricing/reserved-vm-instances/)與 Azure Hybrid Benefit 的搭配使用下，Contoso 可省下高達 82% 的定期預付型方案定價 (2018 年 4 月)。


## <a name="step-2-manage-hybrid-identity"></a>步驟 2：管理混合式身分識別

透過身分識別和存取管理 (IAM) 提供和控制使用者對 Azure 資源的存取，是整合 Azure 基礎結構的重要步驟。  

- Contoso 公司決定將其內部部署 Active Directory 擴充至雲端，而不是在 Azure 中建置新的個別系統。
- 為此，其會建立以 Azure 為基礎的 Active Directory。
- Contoso 並未安裝 Office 365，因此必須佈建新的 Azure AD。
- Office 365 會使用 Azure AD 進行使用者管理。 如果 Contoso 過去使用 Office 365，即表示已有 Azure AD 租用戶，而可以使用該 AD 作為主要 AD。
- [深入了解](https://support.office.com/article/understanding-office-365-identity-and-azure-active-directory-06a189e7-5ec6-4af2-94bf-a22ea225a7a9) Azure AD for Office 365，並了解[如何新增訂用帳戶](https://docs.microsoft.com/azure/active-directory/active-directory-how-subscriptions-associated-directory)至現有的 Azure AD 租用戶。

### <a name="create-an-azure-ad"></a>建立 Azure AD

Contoso 會使用 Azure 訂用帳戶隨附的 Azure AD Free 版本。 Contoso 管理員會如下所示地設定 AD 目錄：

1. 在 [Azure 入口網站](http://portal.azure.com/)中，其瀏覽至 [建立資源] > [身分識別] > [Azure Active Directory]。
2. 在 [建立目錄] 中，他們指定目錄的名稱、初始網域名稱，以及應建立 Azure AD 目錄的區域。

    ![建立 Azure AD](./media/contoso-migration-infrastructure/azure-ad-create.png) 

    > [!NOTE]
    > 所建立的目錄具有 **domainname.onmicrosoft.com** 格式的初始網域名稱。 此名稱無法變更或刪除。 他們必須將其已註冊的網域名稱新增至 Azure AD。

### <a name="add-the-domain-name"></a>新增網域名稱

若要使用其標準網域名稱，Contoso 管理員必須將它以自訂網域名稱的形式新增至 Azure AD。 此選項可讓他們指派熟悉的使用者名稱。 例如，使用者可以使用電子郵件地址 billg@contoso.com 來登入，而不需要使用 billg@contosomigration.microsoft.com。 

為了設定自訂網域名稱，他們會將其新增至目錄、新增 DNS 項目，然後在 Azure AD 中驗證該名稱。

1. 在 [自訂網域名稱] > [新增自訂網域] 中，他們新增網域。
2. 為了在 Azure 中使用 DNS 項目，他們必須向網域註冊機構註冊。 

    - 在 [自訂網域名稱] 清單中，他們記下名稱的 DNS 資訊。 其會使用 MX 項目。
    - 為此，他們需要存取名稱伺服器。 他們會登入 Contoso.com 網域，並使用記下的詳細資料，為 Azure AD 所提供的 DNS 項目建立新的 MX 記錄。  
1. 在 DNS 記錄傳播之後，他們會在網域的詳細名稱中按一下 [驗證]，以檢查自訂網域名稱。

     ![Azure AD DNS](./media/contoso-migration-infrastructure/azure-ad-dns.png) 

### <a name="set-up-on-premises-and-azure-groups-and-users"></a>設定內部部署和 Azure 群組與使用者

現在，Azure AD 已啟動並執行中，Contoso 管理員必須為將會同步至 Azure AD 的內部部署 AD 群組新增員工。 他們應該使用與 Azure 中的資源群組名稱相符的內部部署群組名稱。 這可讓您更輕鬆地找出同步用途的相符項。

#### <a name="create-resource-groups-in-azure"></a>在 Azure 中建立資源群組

Azure 資源群組會將 Azure 資源收集在一起。 使用資源群組識別碼可讓 Azure 對群組內的資源執行作業。

- Azure 訂用帳戶可以有多個資源群組，但一個資源群組只能存在於單一訂用帳戶內。
- 此外，單一資源群組可以有多個資源，但一個資源只能屬於單一資源群組。

Contoso 管理員設定了 Azure 資源群組，如下表中的摘要所說明。

**資源群組** | **詳細資料**
--- | ---
**ContosoCobRG** | 此群組包含所有與商務 (COB) 持續性相關的資源。  其中包含 Contoso 會用於 Azure Site Recovery 服務和 Azure 備份服務的保存庫。<br/><br/> 此外也會包含用於移轉的資源，包括 Azure 移轉和資料庫移轉服務。
**ContosoDevRG** | 此群組包含開發和測試資源。
**ContosoFailoverRG** | 此群組會供容錯移轉後的資源作為登陸區域。
**ContosoNetworkingRG** | 此群組包含所有網路資源。
**ContosoRG** | 此群組包含生產應用程式和資料庫的相關資源。

他們依照下列方式建立資源群組：

1. 在 Azure 入口網站 > [資源群組] 中，他們新增群組。
2. 他們為每個群組指定名稱、群組所屬的訂用帳戶和區域。
3. 資源群組會出現在 [資源群組] 清單中。

    ![資源群組](./media/contoso-migration-infrastructure/resource-groups.png) 

##### <a name="scaling-resource-groups"></a>調整資源群組

日後，Contoso 會根據需求來新增其他資源群組。 例如，他們可以為每個應用程式或服務定義資源群組，以便能獨立管理和保護。

#### <a name="create-matching-security-groups-on-premises"></a>在內部建立相符的安全性群組

1. 在內部部署 Active Directory 中，Contoso 管理員會使用與 Azure 資源群組的名稱相符的名稱設定安全性群組。
 
    ![內部部署 AD 安全性群組](./media/contoso-migration-infrastructure/on-prem-ad.png) 

2. 為了方便管理，他們額外建立了將會新增至所有其他群組的群組。 此群組將有權使用 Azure 中的所有資源群組。 將會有限定數目的全域系統管理員新增至此群組。

### <a name="synchronize-ad"></a>同步處理 AD

Contoso 想要提供一般身分識別，用來存取內部部署和雲端中的資源。 為此，其會將內部部署 Active Directory 與 Azure AD 整合。 透過此模型：

- 使用者和組織可以使用單一身分識別來存取內部部署應用程式和雲端服務 (例如 Office 365) 或網際網路上數千個其他網站。
- 管理員可利用 AD 中的群組在 Azure 中實作[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)。

為了加速整合，Contoso 使用 [Azure AD Connect 工具](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect)。 當您在網域控制站上安裝並設定此工具時，它會將本機內部部署 AD 身分識別同步至 Azure AD。 

### <a name="download-the-tool"></a>下載工具


1. 在 Azure 入口網站中，Contoso 管理員會移至 [Azure Active Directory] > [Azure AD Connect]，並將最新版本的工具下載至他們目前用於同步處理的伺服器。

    ![下載 AD Connect](./media/contoso-migration-infrastructure/download-ad-connect.png) 

2. 他們會使用 [使用快速設定] 來啟動 **AzureADConnect.msi** 安裝。 這是最常見的安裝，可用於以密碼雜湊同步處理進行驗證的單一樹系拓撲。

    ![AD Connect 精靈](./media/contoso-migration-infrastructure/ad-connect-wiz1.png) 

3. 在 [連線至 Azure AD] 中，他們指定用來連線至 Azure AD 的認證 (其格式為 CONTOSO\admin 或 contoso.com\admin)。

    ![AD Connect 精靈](./media/contoso-migration-infrastructure/ad-connect-wiz2.png) 

4. 在 [連線至 AD DS] 中，他們會指定內部部署 AD 的認證。

     ![AD Connect 精靈](./media/contoso-migration-infrastructure/ad-connect-wiz3.png) 

5. 在 [準備好設定] 頁面中，他們按一下 [在設定完成時開始同步處理程序] 以立即啟動同步。 然後，他們進行安裝。

請注意：
- Contoso 直接連線至 Azure。 如果您的內部部署 AD 位於 Proxy 後方，請閱讀此[文章](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-troubleshoot-connectivity)。
- 第一次同步處理後，內部部署 Azure AD 會顯示在 Azure AD 物件中。

    ![Azure 中的內部部署 AD](./media/contoso-migration-infrastructure/on-prem-ad-groups.png) 

- Contoso IT 小組會根據其角色顯示在每個群組中。

    ![Azure 中的內部部署 AD 成員](./media/contoso-migration-infrastructure/on-prem-ad-group-members.png) 

### <a name="set-up-rbac"></a>設定 RBAC

Azure [角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) 可以對 Azure 進行更細緻的存取權管理。 使用 RBAC，您可以僅授與使用者執行工作所需的存取權。 您可以在特定範圍層級，將適當的 RBAC 角色指派給使用者、群組及應用程式。 角色指派的範圍可以是訂用帳戶、資源群組或單一資源。 

現在，Contoso 管理員會將角色指派給他們從內部部署同步處理的 AD 群組。

1. 在 **ControlCobRG** 資源群組中，他們按一下 [存取控制 (IAM)] > [新增]。
2. 在 [新增權限] > [角色] > [參與者] 中，他們會從清單中選取 [ContosoCobRG] AD 群組。 接著，群組就會出現在 [選取的成員] 清單中。 
3. 他們以相同的權限對其他資源群組重複此步驟 (**ContosoAzureAdmins** 除外)，方法將參與者權限新增至符合資源群組的 AD 帳戶。
4. 針對 **ContosoAzureAdmins** AD 群組，他們指派 [擁有者] 角色。

    ![Azure 中的內部部署 AD 成員](./media/contoso-migration-infrastructure/on-prem-ad-groups.png) 


## <a name="step-3-design-for-resilience-and-disaster"></a>步驟 3：針對復原能力與災害進行設計

### <a name="set-up-regions"></a>設定區域

Azure 資源會部署在區域內。

- 區域會組織為地理區，而資料存放區、主權、合規性及復原需求在地理界限內皆屬有效。
- 區域由一組資料中心組成。 這些資料中心部署在定義有延遲的邊緣網路，並透過區域低延遲網路進行連線。
- 每個 Azure 區域都會與另一個區域配對，以提供復原能力。
- 閱讀 [Azure 區域](https://azure.microsoft.com/global-infrastructure/regions/)的相關資訊，並了解[區域的配對方式](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。


Contoso 已決定採用 [美國東部 2] (位於維吉尼亞州) 作為主要區域，並以 [美國中部] (位於愛荷華州) 作為次要區域。 其原因如下：

- Contoso 的資料中心位於紐約，Contoso 已考量過最近資料中心的延遲。
- 美國東部 2 區域具有 Contoso 需要使用的所有服務和產品。 並非所有 Azure 區域都有相同的產品和服務可供使用。 您可以檢閱[不同區域的 Azure 產品](https://azure.microsoft.com/global-infrastructure/services/)。
- 美國中部是與美國東部 2 配對的 Azure 區域。

Contoso 在考量混合式環境時，必須考量如何將復原能力和災害復原策略建置到區域設計中。 廣泛而言，策略的涵蓋範圍包括需依賴 Azure 平台功能 (例如復原所需的容錯網域和區域配對) 的單一區域部署，乃至於從兩個區域部署雲端服務和資料庫並為使用者提供服務的完整主動-主動模型。

Contoso 決定採取折衷方式。 它會在主要區域中部署應用程式與資源，並在次要區域中保存一份基礎結構的完整複本，以便在發生全面性應用程式災害或區域性失敗時，可立即作為完整備份。

### <a name="set-up-availability-zones"></a>設定可用性區域

可用性區域有助於讓應用程式和資料免於遭受資料中心失敗的影響。

- 每個可用性區域都是 Azure 區域內獨一無二的實體位置。
- 每個區域皆由一或多個配備獨立電力、冷卻系統及網路的資料中心所組成。 
- 所有已啟用的 Azure 區域中都至少有三個不同的可用性區域。
- Azure 區域內可用性區域的實體區隔可保護應用程式和資料不受資料中心故障影響。

當應用程式要求延展性、高可用性和復原功能時，Contoso 便會部署可用性區域。 [深入了解](https://docs.microsoft.com/azure/availability-zones/az-overview)。 


## <a name="step-4-design-a-network-infrastructure"></a>步驟 4：設計網路基礎結構

在區域設計完成後，Contoso 即可考量網路策略。 它需要思考內部部署資料中心與 Azure 如何彼此進行連線和通訊，以及如何在 Azure 中設計網路基礎結構。 具體來說，Contoso 必須：

- **規劃混合式網路連線**：了解其會如何連接內部部署和 Azure 之間的網路。
- **設計 Azure 網路基礎結構**：決定其會如何跨區域部署網路。 相同區域內和跨區域的網路要如何通訊？
- **設計並設定 Azure 網路**：設定 Azure 網路和子網路，並決定哪些項目位於其中。

### <a name="plan-hybrid-network-connectivity"></a>規劃混合式網路連線

Contoso 針對 Azure 與內部部署資料中心之間的混合式網路考量了[多種架構](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/)。 [深入了解](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/considerations)選項的比較。

Contoso 內部部署網路基礎結構目前包含位於紐約的資料中心，以及位於美國東部的地區分公司。  所有位置都具有商務等級的網際網路連線。  每家分公司會再透過網際網路經由 IPSec VPN 通道連線至資料中心。

![Contoso 網路](./media/contoso-migration-infrastructure/contoso-networking.png) 

以下是 Contoso 決定實作混合式連線的方式：

1. 在位於紐約的 Contoso 資料中心與位於美國東部 2 和美國中部的兩個 Azure 區域之間設定新的站對站 VPN 連線。
2. 連至 Azure 虛擬網路的分公司流量會透過主要 Contoso 資料中心進行路由。 
3. 當 Contoso 相應增加 Azure 部署時，其會建立資料中心與 Azure 區域之間的 ExpressRoute 連線。 在這種情況下，Contoso 會基於容錯移轉這個唯一的原因，保留 VPN 站對站連線。
    - [深入了解](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/considerations)如何在 VPN 和 ExpressRoute 混合式解決方案之間做選擇。
    - 確認 [ExpressRoute 位置和支援](https://docs.microsoft.com/azure/expressroute/expressroute-locations-providers)。


**僅限 VPN**

![Contoso VPN](./media/contoso-migration-infrastructure/hybrid-vpn.png) 


**VPN 和 ExpressRoute**

![Contoso VPN/ExpressRoute](./media/contoso-migration-infrastructure/hybrid-vpn-expressroute.png) 


### <a name="design-the-azure-network-infrastructure"></a>設計 Azure 網路基礎結構

Contoso 必須以混合式部署兼具安全性和擴充性的方式，妥善設置網路。 為此，Contoso 採用長期的方法，並設計具有復原能力且符合企業需求的虛擬網路 (VNet)。 [深入了解](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)如何規劃 VNet。

為了連接兩個區域，Contoso 決定實作中樞對中樞網路模型：

- 在每個區域中，Contoso 都將使用中樞和輪輻模型。
- 為了連接網路和中樞，Contoso 將使用 Azure 網路對等互連。

#### <a name="network-peering"></a>網路對等互連

Azure 提供可連接 VNet 與中樞的網路對等互連功能。 全域對等互連可在不同區域的 Vnet/中樞之間建立連線。 區域對等互連可連接相同區域中 VNet。 VNet 對等互連有許多優點：

- 對等互連 VNet 之間的網路流量是私人的。
- VNet 之間的流量會保留在 Microsoft 骨幹網路上。 VNet 之間的通訊不需要公用網際網路、閘道或加密。
- 對等互連可在不同 VNet 中的資源之間提供預設的低延遲、高頻寬連線。

[深入了解](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)網路對等互連。

#### <a name="hub-to-hub-across-regions"></a>跨區域的中樞對中樞

Contoso 會在每個區域中部署中樞。 中樞是 Azure 中的虛擬網路 (VNet)，可當作內部部署網路的連線中心點。 中樞 VNet 會使用全域 VNet 對等互連彼此連接。 全域 VNet 對等互連會連接各 Azure 區域內的 VNet。

- 每個區域中的中樞會分別與其他區域中的夥伴中樞配對。
- 中樞會與其區域中的每個網路配對，而且可以連線至所有網路資源。

    ![全域對等互連](./media/contoso-migration-infrastructure/global-peering.png) 

#### <a name="hub-and-spoke-within-a-region"></a>區域內的中樞和輪輻

在每個區域中，Contoso 將會基於不同的用途部署 Vnet，作為來自區域中樞的輪輻網路。 區域內的 VNet 會使用對等互連功能連線至其中樞，以及彼此相連。

#### <a name="design-the-hub-network"></a>設計中樞網路

在 Contoso 所選擇的中樞和輪輻模型內，其必須考量來自內部部署資料中心和來自網際網路的流量將如何路由傳送。 Contoso 決定採用下列方式來處理美國東部 2 和美國中部中樞的路由：

- Contoso 設計名為 "reverse c" 的網路，因為這是從輸入網路傳至輸出網路的封包所依循的路徑。
- 此網路架構有兩個界限，即不受信任的前端周邊區域和後端受信任的區域。
- 防火牆在每個區域中都會有網路介面卡，用以控制對受信任區域的存取。
- 從網際網路：
    - 網際網路流量會送到周邊網路上的負載平衡公用 IP 位址。
    - 此流量會透過防火牆根據防火牆規則進行路由。
    - 網路存取控制實作之後，流量將會轉送至受信任區域中的適當位置。
    - VNet 的輸出流量會透過使用者定義的路由 (UDR) 路由至網際網路。 系統會強制流量通過防火牆，並根據 Contoso 原則檢查流量。
- 從 Contoso 資料中心：
    - 透過 VPN 站對站 (或 ExpressRoute) 傳入的流量會送達 Azure VPN 閘道的公用 IP 位址。
    - 流量會透過防火牆根據防火牆規則進行路由。
    - 在套用防火牆規則之後，流量會轉送至受信任內部區域子網路上的內部負載平衡器 (標準 SKU)。
    - 透過 VPN 從受信任的子網路輸出至內部部署資料中心的流量，會根據規則透過防火牆進行路由，然後再透過 VPN 站對站連線進行傳送。



### <a name="design-and-set-up-azure-networks"></a>設計和設定 Azure 網路

在網路和路由拓撲設定完成後，Contoso 即可設定 Azure 網路和子網路。

- Contoso 將會在 Azure 中實作類別 A 私人網路 (0.0.0.0 至 127.255.255.255)。 此方法是可行的，因為其目前在內部部署中具有類別 B 的私人位址空間 172.160.0/16，因此 Contoso 可確定位址範圍之間不會有任何重疊。
- 其會在主要和次要區域中部署 VNet。
- Contoso 會使用包含前置詞 **VNET** 與區域縮寫 **EUS2** 或 **CUS** 的命名慣例。 使用此標準時，中樞網路將會命名為 **VNET-HUB-EUS2** (美國東部 2) 和 **VNET-HUB-CUS** (美國中部)。
- Contoso 沒有 [IPAM 解決方案](https://docs.microsoft.com/windows-server/networking/technologies/ipam/ipam-top)，因此必須規劃不具 NAT 的網路路由。


#### <a name="virtual-networks-in-east-us-2"></a>美國東部 2 中的虛擬網路

美國東部 2 是 Contoso 將用來部署資源和服務的主要區域。 Contoso 會透過下列方式建構其中的網路：

- **中樞**：美國東部 2 中的中樞 VNet 是內部部署資料中心主要連線的中心點。
- **VNet**：美國東部 2 中的輪輻 Vnet 可在必要時用來隔離工作負載。 除了中樞 VNet 以外，Contoso 在美國東部 2 還會有兩個輪輻 VNet：
    - **VNET-DEV-EUS2**。 此 VNet 將為開發和測試小組提供功能完整的網路，以用於開發專案。 它會作為生產試驗區域，且其運作將依賴產生基礎結構。
    - **VNET-PROD-EUS2**：Azure IaaS 生產元件將位於此網路中。 
    -  每個 VNet 都會有其本身的唯一位址空間，彼此不會重疊。 Contoso 想要設定不需要 NAT 的路由。
- **子網路**：
    - 每個網路中都會有一個子網路供個別的應用程式層使用
    - 生產環境網路中的每個子網路在開發 VNet 中都會有對應的子網路。
    - 此外，生產網路會有網域控制站的子網路。

下表摘要說明美國東部 2 中的 VNet。

**VNet** | **Range** | **對等**
--- | --- | ---
**VNET-HUB-EUS2** | 10.240.0.0/20 | VNET-HUB-CUS2、VNET-DEV-EUS2、VNET-PROD-EUS2
**VNET-DEV-EUS2** | 10.245.16.0/20 | VNET-HUB-EUS2
**VNET-PROD-EUS2** | 10.245.32.0/20 | VNET-HUB-EUS2、VNET-PROD-CUS

![主要區域中的中樞/輪輻](./media/contoso-migration-infrastructure/primary-hub-peer.png) 


#### <a name="subnets-in-the-east-us-2-hub-network-vnet-hub-eus2"></a>美國東部 2 中樞網路中的子網路 (VNET-HUB-EUS2)

**子網路/區域** | **CIDR** | **可用的 IP 位址
--- | --- | ---
**IB-UntrustZone** | 10.240.0.0/24 | 251
**IB-TrustZone** | 10.240.1.0/24 | 251
**OB-UntrustZone** | 10.240.2.0/24 | 251
**OB-TrustZone** | 10.240.3.0/24 | 251
**GatewaySubnets** | 10.240.10.0/24 | 251


#### <a name="subnets-in-the-east-us-2-dev-network-vnet-dev-eus2"></a>美國東部 2 開發網路中的子網路 (VNET-DEV-EUS2)

開發小組會使用開發 VNet 作為生產環境試驗區域。 它有三子網路。

**子網路** | **CIDR** | **位址** | **在子網路中**
--- | --- | --- | ---
**DEV-FE-EUS2** | 10.245.16.0/22 | 1019 | 前端/網路層 VM
**DEV-APP-EUS2** | 10.245.20.0/22 | 1019 | 應用程式層 VM
**DEV-DB-EUS2** | 10.245.24.0/23 | 507 | 資料庫 VM


#### <a name="subnets-in-the-east-us-2-production-network-vnet-prod-eus2"></a>美國東部 2 生產網路中的子網路 (VNET-PROD-EUS2)

Azure IaaS 元件位於生產網路中。 每個應用程式層都有其本身的子網路。 這些子網路會與開發網路中的子網路相符，但會多出一個網域控制站的子網路。

**子網路** | **CIDR** | **位址** | **在子網路中**
--- | --- | --- | ---
**PROD-FE-EUS2** | 10.245.32.0/22 | 1019 | 前端/網路層 VM
**PROD-APP-EUS2** | 10.245.36.0/22 | 1019 | 應用程式層 VM
**PROD-DB-EUS2** | 10.245.40.0/23 | 507 | 資料庫 VM
**PROD-DC-EUS2** | 10.245.42.0/23 | 251 | 網域控制站 VM


![中樞網路架構](./media/contoso-migration-infrastructure/azure-networks-eus2.png)


#### <a name="virtual-networks-in-central-us-secondary-region"></a>美國中部 (次要區域) 中的虛擬網路

美國中部是 Contoso 的次要區域。 Contoso 會透過下列方式建構其中的網路：

- **中樞**：「美國東部 2」中的中樞 VNet 是與內部部署資料中心之連線的中心點，而「美國東部 2」中的輪輻 VNet 則可在必要時用來隔離工作負載，與其他輪輻分開管理。
- **VNet**：Contoso 在美國中部會有兩個 VNet：
    - VNET-PROD-CUS。 此 VNet 是生產網路，類似於 VNET-PROD_EUS2。 
    - VNET-ASR-CUS。 此 VNet 會作為從內部部署容錯移轉之後用來建立 VM 的位置，或作為從主要區域容錯移轉至次要區域的 Azure VM 所在的位置。 此網路類似於生產網路，但其上不會有任何網域控制站。
    -  區域中的每個 VNet 都會有其本身位址空間，彼此不會重疊。 Contoso 會設定不需要 NAT 的路由。
- **子網路**：子網路將會以美國東部 2 中的子網路所採用的類似方式建構。 差別在於，Contoso 不需要網域控制站的子網路。

下表摘要說明美國中部的 VNet。

**VNet** | **Range** | **對等**
--- | --- | ---
**VNET-HUB-CUS** | 10.250.0.0/20 | VNET-HUB-EUS2、VNET-ASR-CUS、VNET-PROD-CUS
**VNET-ASR-CUS** | 10.255.16.0/20 | VNET-HUB-CUS、VNET-PROD-CUS
**VNET-PROD-CUS** | 10.255.32.0/20 | VNET-HUB-CUS、VNET-ASR-CUS、VNET-PROD-EUS2  


![配對區域中的中樞/輪輻](./media/contoso-migration-infrastructure/paired-hub-peer.png)


#### <a name="subnets-in-the--central-us-hub-network-vnet-hub-cus"></a>美國中部中樞網路的子網路 (VNET-HUB-CUS)

**子網路** | **CIDR** | **可用的 IP 位址**
--- | --- | ---
**IB-UntrustZone** | 10.250.0.0/24 | 251
**IB-TrustZone** | 10.250.1.0/24 | 251
**OB-UntrustZone** | 10.250.2.0/24 | 251
**OB-TrustZone** | 10.250.3.0/24 | 251
**GatewaySubnet** | 10.250.2.0/24 | 251


#### <a name="subnets-in-the-central-us-production-network-vnet-prod-cus"></a>美國中部生產網路中的子網路 (VNET-PROD-CUS)

次要的美國中部區域會有一個與主要美國東部 2 區域中的生產網路相類似的生產網路。 

**子網路** | **CIDR** | **位址** | **在子網路中**
--- | --- | --- | ---
**PROD-FE-CUS** | 10.255.32.0/22 | 1019 | 前端/網路層 VM
**PROD-APP-CUS** | 10.255.36.0/22 | 1019 | 應用程式層 VM
**PROD-DB-CUS** | 10.255.40.0/23 | 507 | 資料庫 VM
**PROD-DC-CUS** | 10.255.42.0/24 | 251 | 網域控制站 VM

#### <a name="subnets-in-the-central-us-failoverrecovery-network-in-central-us-vnet-asr-cus"></a>美國中部容錯移轉/復原網路中的子網路 (VNET-ASR-CUS)

VNET-ASR-CUS 網路可用於在區域之間進行容錯移轉的用途。 Site Recovery 將用來在區域之間複寫和容錯移轉 Azure VM。 它也可供仍在內部部署上但容錯移轉至 Azure 以進行災害復原的受保護工作負載作為 Azure 網路的 Contoso 資料中心。

VNET-ASR-CUS 是與「美國東部 2」中的生產環境 VNet 相同的基本子網路，但不需要網域控制站子網路。

**子網路** | **CIDR** | **位址** | **在子網路中**
--- | --- | --- | ---
**ASR-FE-CUS** | 10.255.16.0/22 | 1019 | 前端/網路層 VM
**ASR-APP-CUS** | 10.255.20.0/22 | 1019 | 應用程式層 VM
**ASR-DB-CUS** | 10.255.24.0/23 | 507 | 資料庫 VM

![中樞網路架構](./media/contoso-migration-infrastructure/azure-networks-cus.png)

#### <a name="configure-peered-connections"></a>設定對等互連連線

每個區域中的中樞都將與其他區域中的中樞，以及中樞區域內的所有 VNet 配對。 這可讓中樞進行通訊，以及檢視區域內的所有 VNet。 請注意：

- 對等互連會建立雙邊連線。 一個來自於第一個 VNet 上的起始對等互連，另一個則來自第二個 VNet。
- 在混合式部署中，在對等節點之間傳送的流量必須從內部部署資料中心與 Azure 之間的 VPN 連線來檢視。 若要啟用此功能，必須在對等連線上進行特定的設定。

對於任何從輪輻 VNet 經由中樞連至內部部署資料中心的連線，Contoso 都必須允許流量進行轉送，並周遊 VPN 閘道。

##### <a name="domain-controller"></a>網域控制站

對於 VNET-PROD-EUS2 網路中的網域控制站，Contoso 想要讓流量在 EUS2 中樞/生產網路之間傳送，以及透過 VPN 連線傳送至內部部署。 為了這麼做，Contoso 管理員必須允許下列各項：

1. 在對等連線上**允許轉送的流量**和**允許閘道傳輸組態**。 在此範例中，這將是 VNET-HUB-EUS2 對 VNET-PROD-EUS2 的連線。

    ![對等互連](./media/contoso-migration-infrastructure/peering1.png)

2. 在 VNET-PROD-EUS2 對 VNET-HUB-EUS2 的連線上，在對等互連的另一端**允許轉送的流量**和**使用遠端閘道**。

    ![對等互連](./media/contoso-migration-infrastructure/peering2.png)

3. 在內部部署中，他們將設定會透過 VPN 通道將本機流量導向至 VNet 的靜態路由。 他們必須確認哪個閘道提供從 Contoso 到 Azure 的 VPN 通道，並在該閘道上完成此設定。 為此，他們會使用 RRAS。

    ![對等互連](./media/contoso-migration-infrastructure/peering3.png)

##### <a name="production-networks"></a>生產網路 

輪輻對等網路無法透過中樞看到另一個區域中的輪輻對等網路。

若要讓 Contoso 在兩個區域中的生產網路能看到彼此，Contoso 管理員必須為 VNET-PROD-EUS2 和 VENT-PROD-CUS 建立直接對等連線。 

![對等互連](./media/contoso-migration-infrastructure/peering4.png)

### <a name="set-up-dns"></a>設定 DNS

當您在虛擬網路中部署資源時，您會有數種網域名稱解析可供選擇。 您可以使用 Azure 所提供的名稱解析，或是提供 DNS 伺服器以進行解析。 您使用的名稱解析類型取決於資源如何彼此通訊。 取得關於 Azure DNS 服務的[詳細資訊](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#azure-provided-name-resolution)。

Contoso 管理員認為 Azure DNS 服務在混合式環境中並非理想的選擇。 因此，他們決定使用內部部署 DNS 伺服器。

- 由於這是混合式網路，因此內部部署和 Azure 中的所有 VM 都必須能夠解析名稱，才可正常運作。 這表示，必須將自訂 DNS 設定套用至所有 VNet。
- Contoso 目前已在 Contoso 資料中心和分公司部署 DC。 主要 DNS 伺服器為 CONTOSODC1(172.16.0.10) 和 CONTOSODC2(172.16.0.1)
- 在部署 VNet 之後，內部部署網域控制站將設定成用來作為網路中的 DNS 伺服器。 
- 若要進行此設定，在 VNet 上使用自訂 DNS 時，必須將 Azure 的遞迴解析程式 IP 位址 (例如 168.63.129.16) 新增至 DNS 清單。  為此，Contoso 會在每個 VNet 上設定 DNS 伺服器設定。 例如，VNET-HUB-EUS2 網路的自訂 DNS 設定會顯示如下：
    
    ![自訂 DNS](./media/contoso-migration-infrastructure/custom-dns.png)

除了內部部署網域控制站以外，Contoso 還會再實作四個網域控制站以支援 Azure 網路，每個區域各兩個。 Contoso 會在 Azure 中進行下列部署。

**區域** | **DC** | **VNet** | **子網路** | **IP 位址**
--- | --- | --- | --- | ---
EUS2 | CONTOSODC3 | VNET-PROD-EUS2 | PROD-DC-EUS2 | 10.245.42.4
EUS2 | CONTOSODC4 | VNET-PROD-EUS2 | PROD-DC-EUS2 | 10.245.42.5
CUS | CONTOSODC5 | VNET-PROD-CUS | PROD-DC-CUS | 10.255.42.4
CUS | CONTOSODC6 | VNET-PROD-CUS | PROD-DC-CUS | 10.255.42.4

在部署內部部署網域控制站之後，Contoso 必須在其中一個區域的網路上更新 DNS 設定，以將新的網域控制站納入 DNS 伺服器清單中。



#### <a name="set-up-domain-controllers-in-azure"></a>在 Azure 中設定網域控制站

在更新網路設定之後，Contoso 管理員即可在 Azure 中建置網域控制站。

1. 在 Azure 入口網站中，他們可以將新的 Windows Server VM 部署至適當的 VNet。
2. 他們可以在每個位置中建立 VM 的可用性設定組。 可用性設定組有下列功用：
    - 確保 Azure 網狀架構會將 VM 分隔至 Azure 區域的不同基礎結構中。 
    -  可讓 Contoso 針對 Azure 中的 VM 達到 99.95% SLA 的標準。  [深入了解](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)。

    ![可用性群組](./media/contoso-migration-infrastructure/availability-group.png) 
3. 部署 VM 之後，他們會對 VM 開放網路介面。 他們會將私人 IP 位址設定為靜態，並指定有效位址。

    ![VM NIC](./media/contoso-migration-infrastructure/vm-nic.png)

4. 此時，他們將新的資料磁碟連結至 VM。 此磁碟包含 Active Directory 資料庫和 sysvol 共用。 
    - 磁碟的大小將會決定它所支援的 IOPS 數目。
    - 經過一段時間後，磁碟大小可能需要隨著環境成長而增加。
    - 磁碟機的主機快取不應設為讀取/寫入。 Active Directory 資料庫不支援此功能。

     ![Active Directory 磁碟](./media/contoso-migration-infrastructure/ad-disk.png)

5. 磁碟新增後，他們透過遠端桌面連線至 VM，並開啟伺服器管理員。
6. 接著，他們在 [檔案和存放服務] 中執行 [新增磁碟區精靈]，以確定該磁碟機已取得代號 F: 或位於本機 VM 上。

     ![新增磁碟區精靈](./media/contoso-migration-infrastructure/volume-wizard.png)

7. 在伺服器管理員中，他們新增 [Active Directory Domain Services] 角色。 接著，他們將 VM 設定為網域控制站。

      ![伺服器角色](./media/contoso-migration-infrastructure/server-role.png)  

9. 在 VM 設定為 DC 並重新啟動之後，他們開啟 DNS 管理員，並將 Azure DNS 解析程式設定為轉送子。  這可讓 DC 轉送它無法在 Azure DNS 中解析的 DNS 查詢。

    ![DNS 轉送子](./media/contoso-migration-infrastructure/dns-forwarder.png)

10. 現在，他們會使用 VNet 區域的適當網域控制站來更新每個 VNet 的自訂 DNS 設定。 他們將內部部署 DC 納入清單中。

### <a name="set-up-active-directory"></a>設定 Active Directory

AD 是網路中的重要服務，必須正確設定。 Contoso 管理員會為 Contoso 資料中心建置 AD 網站，並且為 EUS2 和 CUS 區域建置。  

1. 他們建立兩個新的站台 (AZURE-EUS2 和 AZURE-CUS) 以及資料中心網站 (ContosoDatacenter)。
2. 建立網站之後，他們在網站中建立子網路，以比對 Vnet 與資料中心。

    ![DC 子網路](./media/contoso-migration-infrastructure/dc-subnets.png)

3. 然後，他們建立兩個網站連結來連接所有項目。 網域控制站隨後應移至其位置。

    ![DC 連結](./media/contoso-migration-infrastructure/dc-links.png)

4. 在所有項目皆設定後，Active Directory 複寫拓撲即已備妥。
    
    ![DC 複寫](./media/contoso-migration-infrastructure/ad-resolution.png)

5. 一切都準備就緒後，網域控制站和網站的清單會顯示於內部部署 Active Directory 管理中心。

    ![AD 系統管理中心](./media/contoso-migration-infrastructure/ad-center.png)

## <a name="step-5-plan-for-governance"></a>步驟 5︰為控管做規劃

Azure 提供多種跨服務和 Azure 平台的控管功能。 [深入了解](https://docs.microsoft.com/azure/security/governance-in-azure)選項的基本概念。

Contoso 在設定身分識別和存取控制時，即已開始設立控管和安全性的某些層面。 其大致上有三個領域需要考量：

- **原則**：Azure 中的原則會對您的資源套用並強制執行規則和效果，以確保資源符合公司的需求和 SLA。
- **鎖定**：Azure 可讓您鎖定訂用帳戶、資源群組和其他資源，使其只能由具有相關權限的人員進行修改。
- **標記**：資源可用標記來控制、稽核及管理。 標記會將中繼資料附加至資源，而提供資源或擁有者的相關資訊。

### <a name="set-up-policies"></a>設定原則

「Azure 原則」服務會評估您的資源，掃描出與您所擁有的原則定義不符的資源。 例如，您的原則可能只允許特定類型的 VM，或要求資源必須具有特定標記。 

Azure 原則會指定原則定義，而原則指派則會指定應套用原則的範圍。 從管理群組到資源群組皆可涵蓋於此範圍中。 [了解](https://docs.microsoft.com/azure/azure-policy/create-manage-policy)如何建立及管理原則。

Contoso 想要從幾個原則開始著手：

- 其想要透過一個原則來確保資源只能部署在 EUS2 和 CUS 區域中。
- 其想要將 VM SKU 限定為已核准的 SKU。 其目的是要確保不會使用昂貴的 VM SKU。

#### <a name="limit-resources-to-regions"></a>將資源限定於區域

Contoso 使用內建的原則定義 [允許的位置] 來限制資源區域。

1. 在 Azure 入口網站中，按一下 [所有服務]，然後搜尋 [原則]。
2. 選取 [指派] > [指派原則]。
3. 在原則清單中，選取 [允許的位置]。
4. 將 [範圍] 設定為 Azure 訂用帳戶的名稱，然後在允許清單中選取兩個區域。

    ![原則允許的區域](./media/contoso-migration-infrastructure/policy-region.png)

5. 根據預設，原則會有 [拒絕] 設定，這表示如果有人在並非位於 EUS2 或 CUS 的訂用帳戶中啟動部署，部署將會失敗。 以下說明某人在位於美國西部的 Contoso 訂用帳戶中嘗試設定部署時所發生的情形。

    ![原則失敗](./media/contoso-migration-infrastructure/policy-failed.png)

#### <a name="allow-specific-vm-skus"></a>允許特定 VM SKU

Contoso 會使用內建的原則定義 [允許虛擬機器 SKU] 來限制可在訂用帳戶中建立的 VM 類型。 

![原則 SKU](./media/contoso-migration-infrastructure/policy-sku.png)



#### <a name="check-policy-compliance"></a>檢查原則合規性

原則會立即生效，而 Contoso 可以檢查資源的合規性。 

1. 請在 Azure 入口網站中，按一下 [合規性] 連結。
2. 合規性儀表板隨即出現。 您可以向下切入以取得詳細資料。

    ![原則相容性](./media/contoso-migration-infrastructure/policy-compliance.png)


### <a name="set-up-locks"></a>設定鎖定

Contoso 一直都使用 ITIL 架構來管理其系統。 變更控制是此架構最重要的環節之一，而 Contoso 想要確定 Azure 部署中已實作變更控制。

Contoso 將依照下列方式實作鎖定：

- 任何生產或容錯移轉元件必須位於具有 ReadOnly 鎖定的資源群組中。  這表示，若要修改或刪除生產項目，必須先移除鎖定。 
- 非生產資源群組將具有 CanNotDelete 鎖定。 這表示經過授權的使用者可以讀取或修改資源，但無法將它刪除。

[深入了解](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources)鎖定。

### <a name="set-up-tagging"></a>設定標籤

為了追蹤日漸增加的資源，Contoso 勢必要為資源與適當的部門、客戶和環境建立關聯，因為其重要性也是與日俱增。 

除了提供資源和擁有者的相關資訊以外，標記亦可讓 Contoso 彙總和分組資源，並將該資料用於退款。

Contoso 必須以對企業有意義的方式將 Azure 資產視覺化。 例如，依角色或部門。 請注意，資源不需要位於相同的資源群組即可共用標記。 Contoso 會建立一個簡單的標記分類，讓每個人都使用相同的標記。

**標記名稱** | **值**
--- | ---
CostCenter | 12345：它必須是來自 SAP 的有效成本中心。
BusinessUnit | 業務單位的名稱 (來自 SAP)。 符合 CostCenter。
ApplicationTeam | 擁有應用程式支援之小組的電子郵件別名。
CatalogName | 應用程式或 ShareServices 的名稱，視資源所支援的服務類別而定。
ServiceManager | 資源之 ITIL 服務管理員的電子郵件別名。
COBPriority | BCDR 的企業所設定的優先順序。 1-5 的值。
ENV | 可能的值為 DEV、STG、PROD。 分別代表開發、預備和生產。

例如︰ 

 ![Azure 標記](./media/contoso-migration-infrastructure/azure-tag.png)

在建立標記之後，Contoso 會回過頭建立新的 Azure 原則定義和指派，以強制在整個組織使用必要的標記。


## <a name="step-6-consider-security"></a>步驟 6：考量安全性

安全性在雲端中至關重要，而 Azure 提供了豐富的安全性工具和功能。 這些工具和功能可協助您在安全的 Azure 平台上建立安全的解決方案。 請閱讀[受信任雲端的信賴度](https://azure.microsoft.com/overview/trusted-cloud/)，以深入了解 Azure 安全性。

Contoso 有幾個需要考量的層面：

- **Azure 資訊安全中心**：Azure 資訊安全中心可為混合式雲端工作負載提供統一的安全性管理和進階威脅防護。 使用資訊安全中心，您可以在工作負載中套用安全性原則、限制暴露於威脅的可能性，以及偵測和回應攻擊。  [深入了解](https://docs.microsoft.com/azure/security-center/security-center-intro)。
- **網路安全性群組 (NSG)**：NSG 是一種篩選器 (防火牆)，其中包含安全性規則清單，經套用後可允許或拒絕對已連線至 Azure VNet 的資源傳輸網路流量。 [深入了解](https://docs.microsoft.com/azure/virtual-network/security-overview)。
- **資料加密**：Azure 磁碟加密是協助您加密 Windows 和 Linux IaaS 虛擬機器磁碟的功能。 [深入了解](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest)。

### <a name="work-with-the-azure-security-center"></a>使用 Azure 資訊安全中心

Contoso 想要快速了解其新建混合式雲端的安全性態勢，特別是 Azure 工作負載的安全性。  因此，Contoso 決定從下列功能開始實作 Azure 資訊安全中心： 

- 集中式管理原則
- 持續評估
- 可操作的建議 

#### <a name="centralize-policy-management"></a>集中管理原則

透過集中式原則管理，Contoso 將可集中管理整個環境的安全性原則，以確實符合安全性需求。 其可以輕鬆快速地實作會套用至所有 Azure 資源的原則。

![安全性原則](./media/contoso-migration-infrastructure/security-policy.png)

#### <a name="assess-and-action"></a>評估和動作

Contoso 將利用持續的安全性評估來監視機器、網路、儲存體、資料和應用程式的安全性，以找出潛在的安全性問題。 

- 資訊安全中心會分析 Contoso 的計算、基礎結構和資料資源的安全性狀態，以及 Azure 應用程式和服務的安全性狀態。
- 持續評估可協助 Contoso 作業小組找出潛在的安全性問題，例如遺漏安全性更新或已公開網路連接埠的系統。 
- 特別是，Contoso 想要確定所有 VM 皆受到保護。 資訊安全中心可提供這方面的協助，因為它能夠驗證 VM 健康情況，運用已排定優先順序和可操作的建議，在攻擊者入侵之前修正安全性弱點。

![監視](./media/contoso-migration-infrastructure/monitoring.png)

### <a name="work-with-nsgs"></a>使用 NSG

Contoso 可以使用網路安全性群組，限制對虛擬網路中的資源所傳輸的網路流量。

- 網路安全性群組包含一些安全性規則，可根據來源或目的地 IP 位址、連接埠和通訊協定允許或拒絕輸入或輸出網路流量。
- 套用至子網路時，規則將會套用至子網路中的所有資源。 除了網路介面以外，這包括在子網路中部署的 Azure 服務執行個體。
- 應用程式安全性群組 (ASG) 可讓您將網路安全性設定為應用程式結構的自然擴充功能，讓您將 VM 分組，並定義以這些群組為基礎的網路安全性原則。
    - 應用程式安全性群組意謂著 Contoso 可以大規模重複使用安全性原則，而無須手動維護明確的 IP 位址。 此平台可處理明確 IP 位址和多個規則集的複雜性，讓您專注於商務邏輯。
    - Contoso 可以在指定應用程式安全性群組作為安全性規則中的來源和目的地。 定義安全性原則之後，Contoso 可以建立 VM，並將 VM NIC 指派給群組。 


Contoso 會混合實作 NSG 與 ASG。 Contoso 對 NSG 管理有所顧慮。 其也擔心會過度使用 NSG，並讓作業人員的作業變得更加複雜。 以下是 Contoso 將採取的做法：

- 進出於所有子網路的所有流量 (南-北) 都將受到 NSG 規則的規範，但中樞網路中的 GatewaySubnets 除外。
- 任何防火牆或網域控制站均同時受到 NSG 子網路和 NIC NSG 的保護。
- 所有生產應用程式都將套用 ASG。


Contoso 建置了此機制尋找其應用程式的模型。

![安全性](./media/contoso-migration-infrastructure/asg.png)


與 ASG 相關聯的 NSG 將會以最低權限設定，以確保只有允許的封包可從網路的某個部分流向其目的地。

**Action** | **名稱** | **來源** | **目標** | **連接埠**
--- | --- | --- | --- | --- 
允許 | AllowiInternetToFE | VNET-HUB-EUS1/IB-TrustZone | APP1-FE 80、443
允許 | AllowWebToApp | APP1-FE | APP1-DB | 1433
允許 | AllowAppToDB | APP1-APP | 任意 | 任意
拒絕 | DenyAllInbound | 任意 | 任意 | 任意

### <a name="encrypt-data"></a>加密資料

Azure 磁碟加密可與 Azure Key Vault 整合，協助您控制及管理金鑰保存庫訂用帳戶中的磁碟加密金鑰與密碼。 它可確保 VM 磁碟上的所有資料都會在 Azure 儲存體中進行待用加密。  

- Contoso 確定特定 VM 需要加密。
- Contoso 會對含有客戶、機密或 PPI 資料的 VM 套用加密。


## <a name="conclusion"></a>結論

在此文章中，Contoso 為 Azure 訂用帳戶、混合式身分識別、災害復原、網路、控管和安全性設定了 Azure 基礎結構和原則。 

在移轉至雲端的作業中，並不一定會用到 Contoso 在此處完成的所有步驟。 在此案例中，其想要規劃可用於所有移轉類型的網路基礎結構，且需兼具安全性、復原能力和可擴充性。 

在此基礎結構就緒後，Contoso 即可繼續操作，嘗試進行移轉。

## <a name="next-steps"></a>後續步驟

在第一個移轉案例中，Contoso 要[評估其內部部署 SmartHotel360 兩層式應用程式是否可移轉至 Azure](contoso-migration-assessment.md)。 
