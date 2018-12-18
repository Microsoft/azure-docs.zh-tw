---
title: 在 VM 上的 SQL Server 應用程式模式 | Microsoft Docs
description: 這篇文章涵蓋適用於 Azure VM 上的 SQL Server 應用程式模式。 它們可為解決方案架構師和開發人員提供基礎良好的應用程式架構和設計。
services: virtual-machines-windows
documentationcenter: na
author: ninarn
manager: craigg
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 41863c8d-f3a3-4584-ad86-b95094365e05
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 05/31/2017
ms.author: ninarn
ms.openlocfilehash: 18de5b3d71922effa72414a6a52b126a34076cf3
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44722124"
---
# <a name="application-patterns-and-development-strategies-for-sql-server-in-azure-virtual-machines"></a>Azure 虛擬機器中的 SQL Server 應用程式模式和開發策略
[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="summary"></a>摘要：
判斷要針對 Azure 環境中以 SQL Server 為基礎應用程式使用哪種或哪些應用程式模式，是相當重要的設計決策，這需要深入了解 SQL Server 與 Azure 的每個基礎結構元件是如何搭配運作。 您可以透過 Azure 基礎結構服務中的 SQL Server，輕鬆地維護和監視建置於 Windows Server 上的現有 SQL Server 應用程式，或移轉至 Azure 的虛擬機器中。

此文章的目的在於為解決方案架構師和開發人員提供基礎良好的應用程式架構和設計，讓他們將現有應用程式移轉至 Azure，以及在 Azure 中開發新應用程式時可以採用。

每個應用程式模式都有提供內部部署案例、啟用雲端功能的個別解決方案，以及相關的技術建議。 此外，此文章還討論 Azure 特定的開發策略，讓您可以正確地設計應用程式。 由於存在許多可能的應用程式模式，因此建議架構師和開發人員應該針對其應用程式和使用者，選擇最適合的應用程式模式。

**技術參與者：** Luis Carlos Vargas Herring、Madhan Arumugam Ramakrishnan

**技術審稿人員：** Corey Sanders、Drew McDaniel、Narayan Annamalai、Nir Mashkowski、Sanjay Mishra、Silvano Coriani、Stefan Schackow、Tim Hickey、Tim Wieman、Xin Jin

## <a name="introduction"></a>簡介
您可以透過分隔出不同機器上的不同應用程式層元件，以及個別元件，開發各種類型的多層式架構應用程式。 例如，您可以將用戶端應用程式和商務規則元件放在一部機器中，前端 Web 層和資料存取層元件放在另一部機器，然後再將後端資料庫層放在另一部機器。 這種結構有助於分離各個層。 如果您變更資料來源，則您不需要變更用戶端或 Web 應用程式，只要變更資料存取層元件即可。

典型的 *多層式架構* 應用程式包含展示層、商務層和資料層：

| 層 | 說明 |
| --- | --- |
| **展示** |*展示層* (Web 層、前端層) 是使用者與應用程式互動的層。 |
| **商務** |*商務層* (中間層) 是讓展示層和資料層彼此通訊的層，並含有系統的核心功能。 |
| **資料** |*資料層* 基本上是儲存應用程式資料的伺服器 (例如，執行 SQL Server 的伺服器)。 |

應用程式層描述應用程式中功能與元件的邏輯群組；而層則描述個別實體伺服器、電腦、網路或遠端位置上功能與元件的實體分佈。 應用程式層可位於同一部實體電腦 (同一層)，或分散在不同電腦 (多層式架構)，而每個層中的元件可透過定義明確的介面，與其他層中的元件進行通訊。 您可以將「層」這個詞彙視為實體分佈模式，例如兩層、三層和多層式架構。 **2 層應用程式模式** 包含兩個應用程式層：應用程式伺服器和資料庫伺服器。 應用程式伺服器和資料庫伺服器之間會進行直接通訊。 應用程式伺服器包含 Web 層和商務層元件。 在 **3 層應用程式模式**中，有三個應用程式層：Web 伺服器、應用程式伺服器 (其中包含商務邏輯層和/或商務層資料存取元件)，以及資料庫伺服器。 Web 伺服器和資料庫伺服器之間會透過應用程式伺服器進行通訊。 如需應用程式層和各層的詳細資訊，請參閱 [Microsoft 應用程式架構指南](https://msdn.microsoft.com/library/ff650706.aspx)。

開始閱讀此文章之前，您應該先了解 SQL Server 和 Azure 的基本概念。 如需相關資訊，請參閱 [SQL Server 線上叢書](https://msdn.microsoft.com/library/bb545450.aspx)、[Azure 虛擬機器中的 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md) 以及 [Azure.com](https://azure.microsoft.com/)。

此文章會說明一些應用程式模式，它們適用於簡單的應用程式以及複雜度高的企業應用程式。 在詳述每個模式之前，我們建議您應該先熟悉 Azure 中可用的資料儲存體服務，例如 [Azure 儲存體](../../../storage/common/storage-introduction.md)、[Azure SQL Database](../../../sql-database/sql-database-technical-overview.md) 和 [Azure 虛擬機器中的 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md)。 若要為應用程式做出最佳設計決策，您必須清楚地了解何時要使用哪一種資料儲存體服務。

### <a name="choose-sql-server-in-an-azure-virtual-machine-when"></a>若有下列情況，請選擇使用 Azure 虛擬機器中的 SQL Server：
* 您需要控制 SQL Server 和 Windows。 例如，這可能包括 SQL Server 版本、特殊的 Hotfix、效能組態等等。
* 您需要與 SQL Server 內部部署完全相容，且想要將現有應用程式按原狀移至 Azure。
* 您想要利用 Azure 環境的功能，但 Azure SQL Database 不支援您應用程式需要的所有功能。 這可能包括下列部分：
  
  * **資料庫大小**：此文章更新時，SQL Database 支援的資料庫最多可有 1 TB 的資料。 如果應用程式需要的資料超過 1 TB，而您不想要實作自訂分區化解決方案，則建議您使用「Azure 虛擬機器」中的 SQL Server。 如需最新資訊，請參閱[向外擴充 Azure SQL Database](https://msdn.microsoft.com/library/azure/dn495641.aspx)、[以 DTU 為基礎的購買模型](../../../sql-database/sql-database-service-tiers-dtu.md)和[以虛擬核心為基礎的購買模型](../../../sql-database/sql-database-service-tiers-vcore.md) (預覽)。
  * **HIPAA 法規遵循**：醫療保健產業的客戶和獨立軟體廠商 (ISV) 可以選擇使用 [Azure 虛擬機器中 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md) 而不是 [Azure SQL Database](../../../sql-database/sql-database-technical-overview.md)，因為 HIPAA 業務合作協議 (BAA) 已涵蓋 Azure 虛擬機器中的 SQL Server。 如需法規遵循的資訊，請參閱 [Microsoft Azure 信任中心：法規遵循](https://azure.microsoft.com/support/trust-center/compliance/)。
  * **執行個體層級功能**：目前 SQL Database 不支援存在於資料庫外的功能 (例如連結的伺服器、代理程式作業、FileStream、Service Broker 等等)。 如需詳細資訊，請參閱 [Azure SQL Database 方針和限制](https://msdn.microsoft.com/library/azure/ff394102.aspx)。

## <a name="1-tier-simple-single-virtual-machine"></a>1 層 (簡單)：單一虛擬機器
在此應用程式模式中，您會將 SQL Server 應用程式和資料庫部署到 Azure 中的獨立虛擬機器。 同一個虛擬機器會包含您的用戶端/Web 應用程式、商務元件、資料存取層和資料庫伺服器。 展示、商務及資料存取程式碼會以邏輯方式分隔，但實際上位於同一部伺服器機器中。 大部分客戶會從此應用程式模式開始，然後將更多 Web 角色或虛擬機器加入系統，以向外延展。

若有下列情況，此應用程式模式會很適用：

* 您想要執行簡單的移轉到 Azure 平台，以評估該平台是否會回應應用程式的需求。
* 您想要將所有應用程式層裝載在同一 Azure 資料中心的同一個虛擬機器中，以降低各層之間的延遲。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。

下圖說明簡單的內部部署案例，以及您如何能夠在 Azure 的單一虛擬機器內，部署其啟用雲端功能的解決方案。

![1 層應用程式模式](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728008.png)

在與展示層相同的實體層上部署商務層 (商務邏輯和資料存取元件) 可以將應用程式效能最大化，除非您因為延展性或安全性問題，必須使用不同的層。

由於這是很常見的模式，因此以下移轉文章可能會很實用，可讓您將資料移到 SQL Server VM：[將資料庫移轉至 Azure VM 上的 SQL Server](virtual-machines-windows-migrate-sql.md)。

## <a name="3-tier-simple-multiple-virtual-machines"></a>3 層 (簡單)：多個虛擬機器
在此應用程式模式中，您會將每個應用程式層放在不同的虛擬機器中，以在 Azure 中部署 3 層應用程式。 這麼做可提供彈性的環境，以便輕鬆地向上和向外延展案例。 當一個虛擬機器包含您的用戶端/Web 應用程式時，另一個可裝載您的商務元件，而另一個則可裝載資料庫伺服器。

若有下列情況，此應用程式模式會很適用：

* 您想要將複雜的資料庫應用程式移轉至 Azure 虛擬機器。
* 您想要在不同的區域裝載不同的應用程式層。 例如，您可能會為了報告用途，而有部署到多個區域的共用資料庫。
* 您想要將企業應用程式，從內部部署虛擬化平台移動到 Azure 虛擬機器。 如需企業應用程式的詳細討論，請參閱 [企業應用程式是什麼](https://msdn.microsoft.com/library/aa267045.aspx)。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。

下圖說明如何將每個應用程式層放在不同的虛擬機器中，以在 Azure 內設置簡單的 3 層應用程式。

![3 層應用程式模式](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728009.png)

在此應用程式模式中，每一層只有一個虛擬機器 (VM)。 如果您在 Azure 中有多個 VM，我們建議您設定虛擬網路。 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 會建立信任的安全性界限，並讓 VM 透過私人 IP 位址彼此通訊。 此外，請務必確認所有網際網路連線只會進入展示層。 遵循此應用程式模式時，請管理網路安全性群組規則以控制存取。 如需詳細資訊，請參閱 [允許使用 Azure 入口網站從外部存取您的 VM](../nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

在圖表中，網際網路通訊協定可以是 TCP、UDP、HTTP 或 HTTPS。

> [!NOTE]
> 在 Azure 中設定虛擬網路是免費的。 不過，您必須為連接到內部部署的 VPN 閘道付費。 此費用取決於佈建和可用的連接時間量。
> 
> 

## <a name="2-tier-and-3-tier-with-presentation-tier-scale-out"></a>展示層向外延展的 2 層和 3 層
在此應用程式模式中，您會將每個應用程式層放在不同的虛擬機器中，以便將 2 層或 3 層資料庫應用程式部署到 Azure 虛擬機器。 此外，因為傳入的用戶端要求數量增加，所以您必須向外延展展示層。

若有下列情況，此應用程式模式會很適用：

* 您想要將企業應用程式，從內部部署虛擬化平台移動到 Azure 虛擬機器。
* 因為傳入的用戶端要求數量增加，所以想要向外延展展示層。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。
* 您想要擁有可依需求相應增加和降低規模的基礎結構環境。

下圖說明為了因應傳入的用戶端要求數量增加，如何向外延展展示層，以在 Azure 的多個虛擬機器中設置應用程式層。 如圖所示，Azure Load Balancer 會負責將流量分配到多個虛擬機器，也會決定要連接到哪部 Web 伺服器。 在負載平衡器後方有多個 Web 伺服器執行個體，可確保展示層的高可用性。

![應用程式模式 - 展示層向外延展](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728010.png)

### <a name="best-practices-for-2-tier-3-tier-or-n-tier-patterns-that-have-multiple-vms-in-one-tier"></a>一層內有多個 VM 之 2 層、3 層或多層式架構模式的最佳做法
建議您將屬於同一層的虛擬機器放在相同雲端服務中，並使用相同的可用性設定組。 例如，將一組 Web 伺服器放在 **CloudService1** 和 **AvailabilitySet1**，然後將一組資料庫伺服器放在 **CloudService2** 和 **AvailabilitySet2**。 Azure 中的可用性設定組可讓您將高可用性節點放入分隔的容錯網域和升級網域。

若要利用某一層中的多個 VM 執行個體，您必須在應用程式層之間設定 Azure Load Balancer。 若要設定每一層的負載平衡器，請分別在每一層的 VM 上建立負載平衡的端點。 針對特定的某一層，請先在相同雲端服務中建立 VM。 這可確保它們具有相同的公用「虛擬 IP」位址。 接下來，在該層的其中一個虛擬機器上建立端點。 然後將同一個端點指派到該層上的其他虛擬機器，以進行負載平衡。 您可以透過建立負載平衡集，將流量分配到多個虛擬機器，並在後端 VM 節點失敗時，讓負載平衡器決定要連接的節點。 例如，在負載平衡器後方有多個 Web 伺服器執行個體，可確保展示層的高可用性。

就最佳做法而言，請務必使所有網際網路連線先進入展示層。 展示層會存取商務層，而商務層會存取資料層。 如需有關如何允許存取展示層的詳細資訊，請參閱 [允許使用 Azure 入口網站從外部存取您的 VM](../nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

請注意，Azure 中的負載平衡器運作方式類似內部部署環境中的負載平衡器。 如需詳細資訊，請參閱 [Azure 基礎結構服務的負載平衡](../tutorial-load-balancer.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

此外，我們建議您使用 Azure 虛擬網路，為虛擬機器設定私人網路。 這可讓虛擬機器透過私人 IP 位址彼此通訊。 如需詳細資訊，請參閱 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md)。

## <a name="2-tier-and-3-tier-with-business-tier-scale-out"></a>商務層向外延展的 2 層和 3 層
在此應用程式模式中，您會將每個應用程式層放在不同的虛擬機器中，以便將 2 層或 3 層資料庫應用程式部署到 Azure 虛擬機器。 此外，您可能因為應用程式的複雜性，而想要將應用程式伺服器元件分配到多個虛擬機器。

若有下列情況，此應用程式模式會很適用：

* 您想要將企業應用程式，從內部部署虛擬化平台移動到 Azure 虛擬機器。
* 您因為應用程式的複雜性，而想要將應用程式伺服器元件分配到多個虛擬機器。
* 您想要將商務邏輯大型內部部署 LOB (企業營運) 應用程式移動到 Azure 虛擬機器。 LOB 應用程式是一套對企業營運而言十分重要的電腦應用程式，例如會計、人力資源 (HR)、薪資、供應鏈管理和資源規劃應用。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。
* 您想要擁有可依需求相應增加和降低規模的基礎結構環境。

下圖說明某個內部部署案例以及其啟用雲端功能的解決方案。 在此案例中，您會透過向外延展商務層 (其中包含商務邏輯層和資料存取元件)，在 Azure 的多個虛擬機器中放置應用程式層。 如圖所示，Azure Load Balancer 會負責將流量分配到多個虛擬機器，也會決定要連接到哪部 Web 伺服器。 在負載平衡器後方有多個應用程式伺服器執行個體，可確保商務層的高可用性。 如需詳細資訊，請參閱 [一層內有多個虛擬機器之 2 層、3 層或多層式架構模式的最佳做法](#best-practices-for-2-tier-3-tier-or-n-tier-patterns-that-have-multiple-vms-in-one-tier)。

![商務層向外延展的應用程式模式](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728011.png)

## <a name="2-tier-and-3-tier-with-presentation-and-business-tiers-scale-out-and-hadr"></a>展示層和商務層向外延展的 2 層和 3 層以及 HADR
在此應用程式模式中，您會透過將展示層 (Web 伺服器) 和商務層 (應用程式伺服器) 元件分散到多個虛擬機器，在 Azure 虛擬機器中部署 2 層或 3 層資料庫應用程式。 此外，您會針對 Azure 虛擬機器中的資料庫，實作高可用性和災害復原解決方案。

若有下列情況，此應用程式模式會很適用：

* 您想要透過實作 SQL Server 高可用性和災害復原功能，將企業應用程式從虛擬化平台的內部部署移至 Azure。
* 由於傳入的用戶端要求數量增加和應用程式的複雜性，您想要向外延展展示層和商務層。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。
* 您想要擁有可依需求相應增加和降低規模的基礎結構環境。

下圖說明某個內部部署案例以及其啟用雲端功能的解決方案。 在此案例中，您會在 Azure 的多個虛擬機器中，向外延展展示層和商務層元件。 此外，您會針對 Azure 中的 SQL Server 資料庫，實作高可用性和災害復原 (HADR) 技術。

請在不同的 VM 中執行應用程式的多個複本，確認可以負載平衡它們的要求。 當您有多個虛擬機器時，必須確定可以存取所有 VM，且 VM 均能在某個時間點同時執行。 如果設定負載平衡，Azure Load Balancer 就會追蹤 VM 的健全狀況，然後將傳入的呼叫正確引導至運作狀況良好的 VM 節點。 如需如何設定虛擬機器負載平衡的資訊，請參閱 [Azure 基礎結構服務的負載平衡](../tutorial-load-balancer.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 在負載平衡器後方有多個 Web 和應用程式伺服器執行個體，可確保展示層和商務層的高可用性。

![向外延展和高可用性](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728012.png)

### <a name="best-practices-for-application-patterns-requiring-sql-hadr"></a>需要 SQL HADR 之應用程式模式的最佳做法
當您在 Azure 虛擬機器中設定 SQL Server 高可用性和災害復原解決方案時，必須使用 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 為您的虛擬機器設定虛擬網路。  即使在服務停機期間，虛擬網路內的虛擬機器也會有一個穩定的私人 IP 位址，讓您可以避免 DNS 名稱解析所需的更新時間。 此外，虛擬網路可讓您將內部部署網路延伸至 Azure，並建立信任的安全性界限。 例如，如果您的應用程式具有公司網域限制 (如 Windows 驗證、Active Directory)，則必須設定 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 。

大部分在 Azure 上執行實際程式碼的客戶，都會在 Azure 中保留主要和次要複本。

如需高可用性和災害復原技術的完整資訊及教學課程，請參閱 [Azure 虛擬機器中的 SQL Server 高可用性和嚴重損壞修復](virtual-machines-windows-sql-high-availability-dr.md)。

## <a name="2-tier-and-3-tier-using-azure-vms-and-cloud-services"></a>使用 Azure VM 和雲端服務的 2 層和 3 層
在此應用程式模式中，您會使用 [Azure 雲端服務 ](../../../cloud-services/cloud-services-choose-me.md)(Web 和背景工作角色 - 平台即服務 (PaaS)) 和 [Azure 虛擬機器](../overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (基礎結構即服務 (IaaS))，將 2 層或 3 層應用程式部署至 Azure。 針對展示層/商務層使用 [Azure 雲端服務](https://azure.microsoft.com/documentation/services/cloud-services/)，以及針對資料層使用 [Azure 虛擬機器](../overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中的 SQL Server，對於大多數在 Azure 上執行的應用程式會較有幫助。 原因是在雲端服務上執行的計算執行個體，可讓您輕鬆管理、部署、監視和向外延展。

有了雲端服務，Azure 就能為您維護基礎結構、執行例行性維護、修補作業系統，以及嘗試從服務和硬體故障中復原。 當您的應用程式需要向外延展時，有自動和手動向外延展選項可供您的雲端服務專案使用，方法是增加或減少應用程式所使用的執行個體或虛擬機器數目。 此外，您還可以使用內部部署 Visual Studio，將應用程式部署至 Azure 中的雲端服務專案。

簡而言之，如果您不想要產生大量的展示層/商業層管理工作，且應用程式不需要對軟體或作業系統做任何複雜的設定，請使用 Azure 雲端服務。 如果 Azure SQL Database 不支援您需要的所有功能，請針對資料層使用 Azure 虛擬機器中的 SQL Server。 在 Azure 雲端服務上執行應用程式，並將資料儲存在 Azure 虛擬機器中，可結合這兩個服務的優點。 如需詳細的比較，請參閱此主題的小節：[比較 Azure 中的開發策略](#comparing-web-development-strategies-in-azure)。

在此應用程式模式中，展示層包含 Web 角色，這是在 Azure 執行環境中執行的雲端服務元件，且是針對 Web 應用程式的程式設計而自訂，並受 IIS 和 ASP.NET 的支援。 商務層或後端層包含背景工作角色，這是在 Azure 執行環境中執行的雲端服務元件，對於通用開發而言很實用，此外也可針對 Web 角色執行背景處理。 資料庫層位於 Azure 中的 SQL Server 虛擬機器內。 展示層和資料庫層之間會透過商務層 (背景工作角色元件)，直接進行通訊。

若有下列情況，此應用程式模式會很適用：

* 您想要透過實作 SQL Server 高可用性和災害復原功能，將企業應用程式從虛擬化平台的內部部署移至 Azure。
* 您想要擁有可依需求相應增加和降低規模的基礎結構環境。
* Azure SQL Database 不支援您應用程式或資料庫所需的所有功能。
* 您想要針對各種工作負載層級執行壓力測試，但同時又不想一直擁有及維護許多實體機器。

下圖說明某個內部部署案例以及其啟用雲端功能的解決方案。 在此案例中，您會在 Web 角色中放置展示層、在背景工作角色中放置商務層，但在 Azure 虛擬機器中放置資料層。 在不同的 Web 角色中執行多個展示層的複本，可確保能負載平衡它們的要求。 當您將「Azure 雲端服務」與「Azure 虛擬機器」結合時，建議您一併設定 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 。 有了 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md)，就能在雲端內的相同雲端服務中擁有穩定且持續不變的私人 IP 位址。 當您定義虛擬機器和雲端服務的虛擬網路後，它們就可以開始透過私人 IP 位址彼此通訊。 此外，讓虛擬機器和 Azure Web 角色/背景工作角色位於同一個 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 中，可提供低延遲且更安全的連線。 如需詳細資訊，請參閱 [什麼是雲端服務](../../../cloud-services/cloud-services-choose-me.md)。

如圖所示，Azure Load Balancer 會將流量分配到多個虛擬機器，也會決定要連接到哪部 Web 伺服器或應用程式伺服器。 在負載平衡器後方有多個 Web 和應用程式伺服器執行個體，可確保展示層和商務層的高可用性。 如需詳細資訊，請參閱 [需要 SQL HADR 之應用程式模式的最佳做法](#best-practices-for-application-patterns-requiring-sql-hadr)。

![應用程式模式搭配雲端服務](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728013.png)

實作此應用程式模式的另一種方法是使用合併的 Web 角色 (包含展示層和商務層元件)，如下圖所示。 如果應用程式需要可設定狀態的設計，此應用程式模式就很實用。 由於 Azure 在 Web 和背景工作角色上提供無狀態的計算節點，因此建議您實作邏輯，以使用下列其中一種技術來儲存工作階段狀態：[Azure 快取](https://azure.microsoft.com/documentation/services/redis-cache/)、[Azure 資料表儲存體](../../../cosmos-db/table-storage-how-to-use-dotnet.md)或 [Azure SQL Database](../../../sql-database/sql-database-technical-overview.md)。

![應用程式模式搭配雲端服務](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728014.png)

## <a name="pattern-with-azure-vms-azure-sql-database-and-azure-app-service-web-apps"></a>Azure VM、Azure SQL Database 和 Azure App Service (Web Apps) 的模式
此應用程式模式的主要目標是示範如何在解決方案中，將 Azure 基礎結構即服務 (IaaS) 元件與 Azure 平台即服務元件 (PaaS) 結合。 此模式著重於關聯式資料存放區的 Azure SQL Database。 這不包括 Azure 虛擬機器中的 SQL Server，因為其已包含在 Azure 基礎結構即服務的供應項目中。

在此應用程式模式中，您會將展示層和商務層放在同一個虛擬機器中，並存取 Azure SQL Database (SQL Database) 伺服器中的資料庫，藉此將資料庫應用程式部署到 Azure。 您可以使用傳統的 IIS 型 Web 解決方案來實作展示層。 或者，您可以使用 [Azure Web Apps](https://azure.microsoft.com/documentation/services/app-service/web/)，實作結合的展示層和商業層。

若有下列情況，此應用程式模式會很適用：

* 您在 Azure 中已設定了現有的 SQL Database 伺服器，且想要快速測試應用程式。
* 您想要測試 Azure 環境的功能。
* 您想要在短時間內快速佈建開發和測試環境。
* 您的商務邏輯和資料存取元件可以獨立在 Web 應用程式內。

下圖說明某個內部部署案例以及其啟用雲端功能的解決方案。 在此案例中，您會將應用程式層放在 Azure 的單一虛擬機器內，並存取 Azure SQL Database 中的資料。

![混合應用程式模式](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728015.png)

如果您選擇使用 Azure Web 應用程式實作結合的 Web 和應用程式層，則建議您在 Web Apps 的內容中，將中間層或應用程式層保留為動態連結程式庫 (DLL)。

此外，請查看此文章結尾 [比較 Azure 中的 Web 開發策略](#comparing-web-development-strategies-in-azure) 一節中提供的建議，以深入了解程式設計技術。

## <a name="n-tier-hybrid-application-pattern"></a>多層式架構混合式應用程式模式
在多層式架構混合式應用程式模式中，您會在分散於內部部署和 Azure 間的多個層中，實作應用程式。 因此，您會建立有彈性且可重複使用的混合式系統，藉此修改或新增特定的層，而不必變更其他層。 若要將公司網路延伸至雲端，請使用 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md) 服務。

若有下列情況，此混合式應用程式模式會很適用：

* 您想要建置部分在雲端中執行，而部分在內部部署中執行的應用程式。
* 您想要將現有內部部署應用程式的部分或所有項目移轉至雲端。
* 您想要將企業應用程式，從內部部署虛擬化平台移動到 Azure。
* 您想要擁有可依需求相應增加和降低規模的基礎結構環境。
* 您想要在短時間內快速佈建開發和測試環境。
* 您想要以符合成本效益的方式，取得企業資料庫應用程式的備份。

下圖示範橫跨內部部署和 Azure 的多層式架構混合式應用程式模式。 如圖所示，內部部署基礎結構包含 [Active Directory 網域服務](https://technet.microsoft.com/library/hh831484.aspx) 網域控制站，以支援使用者驗證和授權。 請注意圖示說明的案例，其中資料層的某些部分位於內部部署資料中心，而某些部分則位於 Azure。 您可以根據應用程式的需求，實作數個其他混合式案例。 例如，您可能需要在內部部署環境中保留展示層和商務層，但在 Azure 中保留資料層。

![多層式架構的應用程式模式](./media/virtual-machines-windows-sql-server-app-patterns-dev-strategies/IC728016.png)

在 Azure 中，您可以使用 Active Directory 做為貴組織的獨立雲端目錄，或者您也可以將現有的內部部署 Active Directory 與 [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)整合。 如圖所示，商務層元件可以存取多個資料來源，例如透過私人內部 IP 位址存取 [Azure 中的 SQL Server](virtual-machines-windows-sql-server-iaas-overview.md)，或透過 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md)存取內部部署 SQL Server，或是使用 .NET Framework 資料提供者技術存取 [SQL Database](../../../sql-database/sql-database-technical-overview.md)。 在此圖中，Azure SQL Database 是選用的資料儲存體服務。

在多層式架構混合式應用程式模式中，您可以依指定的順序，實作下列工作流程：

1. 使用 [Microsoft Assessment and Planning (MAP) Toolkit](http://microsoft.com/map)，識別需要移至雲端的企業資料庫應用程式。 MAP Toolkit 會從您考慮虛擬化的電腦上，收集詳細目錄和效能資料，並提供容量和評估規劃的建議。
2. Azure 平台中需要規劃資源和組態，例如儲存體帳戶和虛擬機器。
3. 設定公司網路內部部署和 [Azure 虛擬網路](../../../virtual-network/virtual-networks-overview.md)之間的網路連線。 若要設定公司網路內部部署與 Azure 虛擬機器之間的連線，請使用下列兩種方法之一：
   
   1. 透過 Azure 虛擬機器上的公用端點，建立內部部署與 Azure 之間的連線。 此方法提供簡單的設定，並可讓您在虛擬機器中使用 SQL Server 驗證。 此外，請設定您的網路安全性群組規則以控制連到 VM 的公用流量。 如需詳細資訊，請參閱 [允許使用 Azure 入口網站從外部存取您的 VM](../nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
   2. 透過 Azure 虛擬私人網路 (VPN) 通道，在內部部署與 Azure 之間建立連線。 此方法可讓您將網域原則延伸至 Azure 中的虛擬機器。 此外，您可以設定防火牆規則，並在虛擬機器中使用 Windows 驗證。 Azure 目前支援安全的網站間 VPN 和點對站 VPN 連線：
      
      * 您可以透過安全的網站間連線，在內部部署網路與 Azure 虛擬網路之間建立網路連線。 建議您將內部部署資料中心環境連接到 Azure。
      * 您可以透過安全的點對站連線，在 Azure 虛擬網路與在任何一處執行的電腦之間，建立網路連線。 此方法多半建議用於開發和測試。
      
      如需如何連接至 Azure 中 SQL Server 的資訊，請參閱 [連接 Azure 上的 SQL Server 虛擬機器](virtual-machines-windows-sql-connect.md)。
4. 設定排定的工作和警示，以在 Azure 的虛擬機器磁碟中備份內部部署資料。 如需詳細資訊，請參閱 [SQL Server 備份及還原與 Azure Blob 儲存體服務](https://msdn.microsoft.com/library/jj919148.aspx)和 [Azure 虛擬機器中的 SQL Server 備份和還原](virtual-machines-windows-sql-backup-recovery.md)。
5. 您可以依據應用程式的需求，實作下列三種常見案例之一：
   
   1. 您可以在 Azure 中保留 Web 伺服器、應用程式伺服器及資料庫伺服器內的非機密資料，然後將機密資料保留在內部部署。
   2. 您可以將 Web 伺服器和應用程式伺服器保留在內部部署，而在 Azure 虛擬機器中保留資料庫伺服器。
   3. 您可以將資料庫伺服器、Web 伺服器和應用程式伺服器保留在內部部署，而在 Azure 虛擬機器中保留資料庫複本。 此設定可讓內部部署 Web 伺服器或報告應用程式存取 Azure 中的資料庫複本。 因此，您可以在內部部署資料庫中達到較低的工作負載。 我們建議您針對讀取密集的工作負載和開發用途，實作此案例。 如需在 Azure 中建立資料庫複本的資訊，請參閱 [Azure 虛擬機器中的 SQL Server 高可用性和災害復原](virtual-machines-windows-sql-high-availability-dr.md)中的「AlwaysOn 可用性群組」。

## <a name="comparing-web-development-strategies-in-azure"></a>比較 Azure 中的 Web 開發策略
若要在 Azure 中實作和部署以多層式 SQL Server 為基礎的應用程式，您可以使用下列兩種程式設計方法其中之一：

* 在 Azure 中設定傳統 Web 伺服器 (IIS - 網際網路資訊服務)，並存取位於 Azure 虛擬機器中 SQL Server 內的資料庫。
* 實作雲端服務並將其部署至 Azure。 然後請確定此雲端服務可以存取 Azure 虛擬機器中 SQL Server 內的資料庫。 雲端服務可以包含多個 Web 角色和背景工作角色。

關於 Azure 虛擬機器中的 SQL Server，下表提供傳統 Web 開發與 Azure 雲端服務和 Azure Web Apps 的比較。 表格中包含 Azure Web Apps，因為它可以透過其公用虛擬 IP 位址或 DNS 名稱，使用 Azure VM 中的 SQL Server 當做 Azure Web Apps 的資料來源。

|  | Azure 虛擬機器中的傳統 Web 開發 | Azure 中的雲端服務 | 虛擬主機與 Azure Web Apps |
| --- | --- | --- | --- |
| **從內部部署進行應用程式移轉** |現有的應用程式維持不變。 |應用程式需要 Web 角色和背景工作角色。 |現有的應用程式維持不變，但仍適用於需要快速擴充的獨立式 Web 應用程式和 Web 服務。 |
| **開發和部署** |Visual Studio、WebMatrix、Visual Web Developer、WebDeploy、FTP、TFS、IIS Manager，PowerShell。 |Visual Studio、Azure SDK、TFS，PowerShell。 每一個雲端服務都有兩個環境，可供您部署服務套件和組態：預備環境和生產環境。 在將雲端服務升級到生產環境之前，您可以先在預備環境中測試雲端服務。 |Visual Studio、WebMatrix、Visual Web Developer、FTP、GIT、BitBucket、CodePlex、DropBox、GitHub、Mercurial、TFS、Web Deploy、PowerShell。 |
| **系統管理和設定** |您必須負責應用程式、資料、防火牆規則、虛擬網路，和作業系統上的系統管理工作。 |您必須負責應用程式、資料、防火牆規則，和虛擬網路上的系統管理工作。 |您只需負責應用程式和資料的系統管理工作。 |
| **高可用性和災害復原 (HADR)** |建議您將虛擬機器放置於相同的可用性設定組和相同的雲端服務中。 將 VM 置於相同的可用性設定組中，可讓 Azure 將高可用性節點放入分隔的容錯網域和升級網域。 同樣地，將 VM 放在相同的雲端服務可啟用負載平衡，且 VM 可以透過 Azure 資料中心內的本機網路直接與彼此進行通訊。<br/><br/>您必須負責在 Azure 虛擬機器中實作 SQL Server 的高可用性和災害復原解決方案，以避免任何停機。 如需支援的 HADR 技術，請參閱 [Azure 虛擬機器中的 SQL Server 高可用性和災害復原](virtual-machines-windows-sql-high-availability-dr.md)。<br/><br/>您需負責備份自己的資料和應用程式。<br/><br/>如果資料中心的主機電腦因硬體問題而失敗，Azure 可以移動您的虛擬機器。 此外，當主機電腦因安全性或軟體更新而有更新時，VM 可能會按計劃停機。 因此，建議您在每個應用程式層中至少維護兩個 VM，以確保持續可用性。 Azure 沒有為單一虛擬機器提供 SLA。 |Azure 會管理從基礎硬體或作業系統軟體導致的失敗。 我們建議您實作多個 Web 角色或背景工作角色執行個體，以確保應用程式的高可用性。 如需資訊，請參閱[雲端服務、虛擬機器和虛擬網路服務等級協定](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)。<br/><br/>您需負責備份自己的資料和應用程式。<br/><br/>針對在 Azure VM 中位於 SQL Server 資料庫的資料庫，您必須負責實作高可用性和災害復原解決方案，以避免任何停機。 如需支援的 HDAR 技術，請參閱 Azure 虛擬機器中的 SQL Server 高可用性和災害復原。<br/><br/>**SQL Server 資料庫鏡像**：與 Azure 雲端服務 (web/背景工作角色) 搭配使用。 SQL Server VM 和雲端服務專案可位於相同的 Azure 虛擬網路。 如果 SQL Server VM 不在相同的虛擬網路中，您便必須建立 SQL Server 別名，以將通訊路由傳送至 SQL Server 執行個體。 此外，別名必須與 SQL Server 名稱相符。 |高可用性是繼承自 Azure 背景工作角色、Azure Blob 儲存體和 Azure SQL Database。 例如，Azure 儲存體會維護所有 Blob、資料表和佇列資料的三個複本。 不論何時，Azure SQL Database 都會保留三個執行中的資料複本—一個主要複本和兩個次要複本。 如需詳細資訊，請參閱 [Azure 儲存體](https://azure.microsoft.com/documentation/services/storage/) 和 [Azure SQL Database](../../../sql-database/sql-database-technical-overview.md)。<br/><br/>使用 Azure VM 中的 SQL Server 當做 Azure Web Apps 的資料來源時，請記住，Azure Web Apps 不支援 Azure 虛擬網路。 換言之，從 Azure Web Apps 到 Azure 中 SQL Server VM 的所有連線，都必須透過虛擬機器的公用端點。 這可能會對高可用性和災害復原案例造成一些限制。 例如，在 Azure Web Apps 上使用資料庫鏡像連接到 SQL Server VM 的用戶端應用程式，會無法連接至新的主要伺服器，因為您必須在 Azure 中的 SQL Server 主機 VM 之間設定 Azure 虛擬網路才能使用資料庫鏡像。 因此，目前不支援 **SQL Server 資料庫鏡像**與 Azure Web 應用程式搭配使用。<br/><br/>**SQL Server AlwaysOn 可用性群組**︰當使用 Azure 中的 SQL Server VM 搭配 Azure Web 應用程式時，您可以設定 AlwaysOn 可用性群組。 但您必須設定 AlwaysOn 可用性群組接聽程式，將通訊透過公用負載平衡的端點路由至主要複本。 |
| **跨單位連線** |您可以使用 Azure 虛擬網路連線到內部部署。 |您可以使用 Azure 虛擬網路連線到內部部署。 |支援 Azure 虛擬網路。 如需詳細資訊，請參閱 [Web Apps Virtual Network Integration (Web Apps 虛擬網路整合)](https://azure.microsoft.com/blog/2014/09/15/azure-websites-virtual-network-integration/)。 |
| **延展性** |可透過增加虛擬機器大小或新增更多硬碟來相應增加。 如需關於虛擬機器大小的詳細資訊，請參閱 [Azure 的虛擬機器大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。<br/><br/>**資料庫伺服器**︰向外延展可透過資料庫資料分割技術和 SQL Server AlwaysOn 可用性群組提供。<br/><br/>對於大量讀取工作負載，您可以在多個次要節點以及 SQL Server 複寫上使用 [AlwaysOn 可用性群組](https://msdn.microsoft.com/library/hh510230.aspx)。<br/><br/>對於大量寫入工作負載，您可以在多部實體伺服器實作水平資料分割，提供應用程式向外延展。<br/><br/>此外，您可以使用 [SQL Server 與資料相依路由](https://technet.microsoft.com/library/cc966448.aspx)來實作向外延展。 使用資料依存路由 (DDR) 時，您必須在用戶端應用程式 (通常是在商務層) 中，實作資料分割機制，以將資料庫要求路由傳送至多個 SQL Server 節點。 商務層中包含如何分割資料，以及哪個節點包含資料的對應。<br/><br/>您可以為執行虛擬機器的應用程式調整規模。 如需詳細資訊，請參閱[如何調整應用程式](../../../cloud-services/cloud-services-how-to-scale-portal.md)。<br/><br/>**重要注意事項**：Azure 中的「自動調整規模」功能可讓您自動增加或減少應用程式所使用的虛擬機器數量。 此功能可保證使用者的體驗不會在尖峰期間受到影響，且 VM 不會在需求降低時關機。 如果雲端服務含有 SQL Server VM，建議您不要為雲端服務設定「自動調整規模」選項。 原因是「自動調整規模」功能會讓 Azure 在該 VM 中的 CPU 使用率高於某個臨界值時，開啟虛擬機器；以及在 CPU 使用率低於該臨界值時，關閉虛擬機器。 「自動調整規模」功能對於無狀態的應用很實用，例如 Web 伺服器，其中所有 VM 均可管理工作負載，且不用參考任何先前的狀態。 但「自動調整規模」功能不適用於可設定狀態的應用程式，例如 SQL Server，其中只有一個執行個體允許寫入資料庫。 |透過使用多個 Web 角色和背景工作角色來相應增加。 如需 Web 角色和背景工作角色之虛擬機器大小的詳細資訊，請參閱[設定雲端服務的大小](../../../cloud-services/cloud-services-sizes-specs.md)。<br/><br/>當使用**雲端服務**時，您可以定義多個角色來分散處理，並達到彈性調整應用程式。 每個雲端服務包含一或多個 Web 角色和/或背景工作角色，且各有自己的應用程式檔案和組態。 您可以增加針對某個角色而部署的角色執行個體 (虛擬機器) 數目，藉此調升雲端服務的規模；或是減少角色執行個體數目，以調降雲端服務的規模。 如需詳細資訊，請參閱 [Azure 執行模型](../../../cloud-services/cloud-services-choose-me.md)。<br/><br/>向外延展可透過[雲端服務、虛擬機器和虛擬網路服務等級協定](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/)和負載平衡器，透過內建的 Azure 高可用性支援提供使用。<br/><br/>針對多層式應用程式，我們建議您透過 Azure 虛擬網路，將 web/背景工作角色應用程式連接至資料庫伺服器 VM。 此外，Azure 會為相同雲端服務中的 VM 提供負載平衡，以平均分配使用者要求。 以這種方式連接的虛擬機器能夠透過本機網路在 Azure 資料中心內直接相互通訊。<br/><br/>您可以在 Azure 入口網站上設定**自動調整**以及排程時間。 如需詳細資訊，請參閱[如何在入口網站中設定雲端服務的自動調整](../../../cloud-services/cloud-services-how-to-scale-portal.md)。 |**相應增加和減少**︰您可以增加/減少保留給您網站的執行個體 (VM) 大小。<br/><br/>相應放大︰您可以為您的網站新增更多保留的執行個體 (VM)。<br/><br/>您可以在入口網站上設定**自動調整**以及排程的時間。 如需詳細資訊，請參閱[如何調整 Web Apps](../../../app-service/web-sites-scale.md)。 |

如需有關如何在這些程式設計方法之間做選擇的詳細資訊，請參閱 [Azure Web Apps、雲端服務和 VM：每一個的使用時機](../../../app-service/choose-web-site-cloud-service-vm.md)。

## <a name="next-steps"></a>後續步驟
如需在 Azure 虛擬機器中執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)。

