---
title: Contoso 移轉至 Azure 的概觀 | Microsoft Docs
description: 提供 Contoso 將內部部署資料中心移轉至 Azure 時使用的移轉策略和案例概觀。
author: rayne-wiselman
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 1a86f12fddf9b18700f2c4a2aff840512dfe4a6f
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303637"
---
# <a name="contoso-migration-overview"></a>Contoso 移轉：概觀


本文示範虛構的組織 Contoso 如何將其內部部署基礎結構遷移至 [Microsoft Azure](https://azure.microsoft.com/overview/what-is-azure/) 雲端。 

本文件是系列文章中的第一篇，會說明 Contoso 這家虛構公司如何遷移至 Azure。 整個系列文章中包含資訊以及一些案例，用以說明如何設定基礎結構移轉，以及執行不同類型的移轉。 案例會變得越來越複雜，而我們也將不定期增加其他文章。 這些文章將會說明 Contoso 公司如何完成移轉任務，而一般閱讀要點和具體指示則會在下文完整提供。

## <a name="introduction"></a>簡介

Azure 提供全方位雲端服務組合的存取權。 讓開發人員與 IT 專業人員使用這組服務，透過全球資料中心網路在各種工具和架構上建置、部署及管理應用程式。 當貴公司面臨與數位轉換有關的挑戰時，Azure 雲端可協助您了解如何將資源和作業最佳化、與客戶和員工交流，以及轉換您的產品。

不過，Azure 了解，即使雲端可在速度和彈性、降至最低的成本、效能和可靠性等方面提供優勢，但許多組織還是需要在今後的一段時間裡，執行內部部署資料中心。 為了回應雲端採用障礙，Azure 提供混合式雲端策略來銜接內部部署資料中心與 Azure 公用雲端。 例如，使用 Azure 備份等 Azure 雲端資源來保護內部部署資源，或使用 Azure 分析來深入了解內部部署工作負載。 

做為混合式雲端策略的一部分，Azure 會提供不斷增加的解決方案，協助您將內部部署應用程式和工作負載移轉至雲端。 透過簡單的步驟，您即可全面地評估內部部署資源，以了解要如何在 Azure 雲端中執行這些資源。 然後，在有了深入評估的情況下，您可以放心地將資源遷移至 Azure。 當資源在 Azure 中啟動並執行時，您可以將它們最佳化以維持和改善存取、彈性、安全性與可靠性。

## <a name="migration-strategies"></a>移轉策略

移轉至雲端的策略可分成以下四大類別：重新裝載、重構、重新架構或重建。 您採用的策略取決於商業誘因和移轉目標。 您可能會採用多項策略。 例如，您可選擇對 簡單的應用程式或對業務重要性不大的應用程式採取重新裝載 (隨即轉移) 的方式；而較複雜或具業務關鍵功能的應用程式則採用重新架構的方式。 讓我們來看看這些策略：


**策略** | **定義** | **使用時機** 
--- | --- | --- 
**重新裝載** | 重新裝載通常稱為「隨即轉移」移轉。 這個選項不需要變更程式碼，可讓我們將您現有的應用程式迅速移轉至 Azure。 每個應用程式皆依原狀移轉，可直接享有雲端優勢，而無須承擔變更程式碼的相關風險和成本。 | 需要將應用程式快速移轉至雲端時。<br/><br/> 想要在不修改應用程式的情況下進行移轉時。<br/><br/> 應用程式的架構適合在移轉後運用 [Azure IaaS](https://azure.microsoft.com/overview/what-is-iaas/) 延展性時。<br/><br/> 應用程式對您的業務很重要，但還不需要立即變更應用程式功能時。
**重構** | 重構通常稱為「重新封裝」，需要對應用程式進行最小程度的變更，以便您連結至 [Azure PaaS](https://azure.microsoft.com/overview/what-is-paas/) 並使用雲端供應項目。<br/><br/> 例如，您可將現有應用程式移轉至 Azure App Service 或 Azure Kubernetes Service (AKS)。<br/><br/> 或者，您也可以將關聯式和非關聯式資料庫重構為 Azure SQL Database 受控執行個體、適用於 MySQL 的 Azure 資料庫、適用於 PostgreSQL 的 Azure 資料庫和 Azure Cosmos DB 之類的選項。 | 若您的應用程式可輕鬆地重新封裝以在 Azure 中運作。<br/><br/> 若想採用 Azure 提供的創新式 DevOps 做法或針對工作負載使用容器策略會考慮 DevOps 時。<br/><br/> 若要選擇重構，您必須考量現有程式碼基礎的可攜性及可用的開發技能。
**重新架構** | 若採用重新架構進行移轉，則會著重於修改和擴充應用程式功能及程式碼基礎，藉此將應用程式架構最佳化，以達成雲端延展性。<br/><br/> 例如，您可將單一應用程式劃分為數個微服務群組，如此即能輕鬆搭配運作和擴充。<br/><br/> 或者，您可將關聯式和非關聯式資料庫重新架構為完全受控的 DBaaS 解決方案，例如 Azure SQL Database 受控執行個體、適用於 MySQL 的 Azure 資料庫、適用於 PostgreSQL 的 Azure 資料庫和 Azure Cosmos DB。 | 您的應用程式需要主要修訂，以納入新功能或在雲端平台上有效地運作時。<br/><br/> 您想要使用現有應用程式投資、達到延展性要求、採用創新的 Azure DevOps 做法，以及盡可能避免使用虛擬機器時。
**重建** | 重建的做法則更深入，您必須使用 Azure 雲端技術從頭重新打造應用程式。<br/><br/> 例如，使用 Azure Functions、Azure AI、Azure SQL Database 受控執行個體和 Azure Cosmos DB 等雲端原生技術建置全新應用程式。 | 想要快速完成開發，且現有應用程式的功能及使用週期有限時。<br/><br/> 準備加速商務創新 (包括 Azure 提供的 DevOps 做法)、使用雲端原生技術建置全新應用程式，以及利用 AI、區塊鏈和 IoT 領域的先進技術時。

## <a name="migration-articles"></a>移轉文章

下表摘要說明這一系列的文章。  

- 由於每種移轉案例的商務目標都略有不同，因此移轉策略也會有所差異。
- 針對每個部署案例，我們提供的相關資訊包括：商業誘因和目標、提議的架構、執行移轉的步驟，以及清理建議和移轉完成後可採取的後續步驟。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
文章 1：概觀 | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 本文
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用：[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會移轉 SmartHotel360 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用：[文章 7：在 Azure VM 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業。 | 可用
[文章 8：在 Azure VM 和適用於 MySQL 的 Azure 資料庫上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會藉由使用 Site Recovery 將其 Linux osTicket 應用程式遷移至 Azure VM。 它會使用 MySQL Workbench 將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL Database 中重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將其 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體。 | 可用    
[文章 10：在 Azure Web 應用程式和適用於 MySQL 的 Azure 資料庫中重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用
[文章 11：在 Azure DevOps Services 上重構 Team Foundation Server](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 中重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用    
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用 


在本文中，Contoso 會設定為了完成所有移轉案例，所需的所有基礎結構元素。 







### <a name="demo-apps"></a>示範應用程式

這些文章使用兩個示範應用程式：SmartHotel360 和 osTicket。

- **SmartHotel360**：此應用程式由 Microsoft 開發做為測試應用程式，可讓您搭配 Azure 使用。 SmartHotel360 以開放原始碼提供，您可從 [GitHub](https://github.com/Microsoft/SmartHotel360) 下載； 其為一種連線至 SQL Server 資料庫的 ASP.NET 應用程式。 目前該應用程式位於執行 Windows Server 2008 R2 和 SQL Server 2008 R2 的兩個 VMware 虛擬機器中。 應用程式虛擬機器裝載於內部部署，且由 vCenter Server 管理。
- **osTicket**：一種在 Linux 上執行的開放原始碼服務台票證應用程式。 您可以從 [GitHub](https://github.com/osTicket/osTicket) 下載。 目前應用程式位於執行 Ubuntu 16.04 LTS (使用 Apache 2、PHP 7.0 和 MySQL 5.7) 的兩個 VMware 虛擬機器中
    

## <a name="next-steps"></a>後續步驟

[深入了解](contoso-migration-infrastructure.md) Contoso 如何設定內部部署和 Azure 基礎結構以準備移轉。



