---
title: Azure SQL Database 受控執行個體概觀 | Microsoft Docs
description: 本主題將說明 Azure SQL Database 受控執行個體和其運作方式，以及其與 Azure SQL Database 中單一資料庫之間的差異。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: carlrab
manager: craigg
ms.date: 09/26/2018
ms.openlocfilehash: 626dd362248027831c78d1505662ca12d2ff334d
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392817"
---
# <a name="what-is-a-managed-instance"></a>什麼是受控執行個體？

Azure SQL Database 受控執行個體是 Azure SQL Database 的新部署模型，幾乎可與最新 SQL Server 內部部署環境 (Enterprise Edition) 資料庫引擎 100% 相容，並提供原生[虛擬網路 (VNet)](../virtual-network/virtual-networks-overview.md) 實作，可解決常見的安全性考量，以及提供有利於內部部署 SQL Server 客戶的[商務模型](https://azure.microsoft.com/pricing/details/sql-database/)。 受控執行個體可讓現有 SQL Server 客戶透過最少的應用程式和資料庫變更，將他們的內部部署應用程式隨即轉移至雲端。 同時，受控執行個體會保留 PaaS 的所有功能 (自動修補和版本的更新、[自訂備份](sql-database-automated-backups.md)、[高可用性](sql-database-high-availability.md))，可以大幅降低管理負擔和 TCO。

> [!IMPORTANT]
> 如需受控執行個體目前可用的區域清單，請參閱[使用 Azure SQL Database 受控執行個體將資料庫遷移到完全受控的服務](https://azure.microsoft.com/blog/migrate-your-databases-to-a-fully-managed-service-with-azure-sql-database-managed-instance/)。
 
下圖概述受控執行個體的主要功能：

![主要功能](./media/sql-database-managed-instance/key-features.png)

Azure SQL Database 受控執行個體專為以下客戶設計：想要盡可能輕鬆地將大量應用程式，從內部部署或 IaaS、自行建置或 ISV 提供的環境遷移至完全受控的 PaaS 雲端環境。 利用 Azure 中完全自動化的[資料移轉服務 (DMS)](../dms/tutorial-sql-server-to-managed-instance.md#create-an-azure-database-migration-service-instance)，客戶可以將內部部署 SQL Server 隨即移轉至受控執行個體，受控執行個體可與 SQL Server 內部部署環境相容，並透過原生 VNET 支援來完全隔離客戶執行個體。  您可以透過軟體保證使用[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md)，以折扣優惠在 SQL Database 受控執行個體上交換執行個體的現有授權。  對於需要高度安全性和程式設計介面豐富的 SQL Server 執行個體而言，SQL Database 受控執行個體是雲端中最佳的移轉目的地。 

在受控執行個體正式運作之前，其目標是透過階段式發行計劃，為最新版內部部署 SQL Server 提供幾乎 100% 的介面區相容性。 

若要決定使用 Azure SQL Database 單一資料庫、Azure SQL Database 受控執行個體或虛擬機器中裝載的 SQL Server IaaS，請參閱[如何在 Azure 雲端選擇正確的 SQL Server 版本](sql-database-paas-vs-sql-server-iaas.md)。

## <a name="key-features-and-capabilities"></a>重要功能 

Azure SQL Database 受控執行個體結合了可在 Azure SQL Database 和 SQL Server 資料庫引擎中取得的最佳功能。

> [!IMPORTANT]
> 受控執行個體能執行 SQL Server 最新版本的所有功能，包括線上作業、自動計劃修正，以及其他企業效能增強功能。 

| **PaaS 支援** | **商務持續性** |
| --- | --- |
|無須硬體採購和管理 <br>沒有管理基礎結構的管理負擔 <br>快速佈建和服務調整 <br>自動修補和版本升級 <br>與其他 PaaS 資料服務整合 |99.99% 的 SLA 運作時間  <br>內建[高可用性](sql-database-high-availability.md) <br>使用[自動備份](sql-database-automated-backups.md)保護資料 <br>客戶可設定備份保留期限 (在公開預覽階段中固定是 7 天) <br>使用者起始的[備份](https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql?view=azuresqldb-mi-current) <br>[資料庫還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)功能 |
|**安全性與合規性** | **管理**|
|隔離的環境 ([VNet 整合](sql-database-managed-instance-vnet-configuration.md)、單一租用戶服務、專用的運算和儲存體) <br>[透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)<br>[Azure AD 驗證](sql-database-aad-authentication.md)、單一登入支援 <br>與 Azure SQL 資料庫遵循相同的合規性標準 <br>[SQL 稽核](sql-database-managed-instance-auditing.md) <br>[威脅偵測](sql-database-managed-instance-threat-detection.md) |用於自動化服務佈建與調整的 Azure Resource Manager API <br>用於手動服務佈建與調整的 Azure 入口網站功能 <br>資料移轉服務 

## <a name="vcore-based-purchasing-model"></a>以虛擬核心為基礎的購買模型

[以虛擬核心為基礎的購買模型](sql-database-service-tiers-vcore.md)提供彈性、可控制、透明及直接的方法，讓您將內部部署工作負載需求平移到雲端。 此模型可讓您根據工作負載需求，進行計算、記憶體和儲存體調整。 虛擬核心模型也能夠透過[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md)，最多節省 30% 的成本。

虛擬核心代表可以選擇使用的邏輯 CPU，可在各硬體世代間進行選擇。
- 「第 4 代」邏輯 CPU 是以 Intel E5-2673 v3 (Haswell) 2.4-GHz 處理器為基礎。
- 「第 5 代」邏輯 CPU 是以 Intel E5-2673 v4 (Broadwell) 2.3-GHz 處理器為基礎。

下表可協助您了解如何為您的計算、記憶體、儲存體和 I/O 資源選取最佳組態。

||Gen 4|Gen 5|
|----|------|-----|
|硬體|Intel E5 2673 v3 (Haswell) 2.4 GHz 處理器，附加 SSD 虛擬核心 = 1 PP (實體核心)|Intel E5 2673 v4 (Broadwell) 2.3 GHz 處理器，快速 eNVM SSD，虛擬核心 = 1 LP (超執行緒)|
|計算大小|8 個、16 個、24 個虛擬核心|8 個、16 個、24 個、32 個、40 個、64 個、80 個虛擬核心|
|記憶體|每個虛擬核心 7GB|每個虛擬核心 5.5GB|
||||

## <a name="managed-instance-service-tiers"></a>受控執行個體服務層

有兩個服務層可使用受控執行個體：
- **一般用途**：專為具有標準效能和 IO 延遲需求的應用程式所設計。
- **商務關鍵 (預覽)**：專為具有低 IO 延遲需求且對工作負載的基礎維護作業影響最小的應用程式所設計。

這兩個服務層均保證 99.99% 的可用性，可讓您單獨選取儲存體大小和計算容量。 如需 Azure SQL Database 高可用性架構的詳細資訊，請參閱[高可用性和 Azure SQL Database](sql-database-high-availability.md)。

> [!IMPORTANT]
> 公開預覽版本不支援將服務層級從一般用途變更為業務關鍵，反之亦然。 如果您想要將資料庫遷移至不同服務層中的執行個體，您可以建立新執行個體，並透過原始執行個體的還原時間點來還原資料庫，然後捨棄原始執行個體 (如果已不再需要)。 

### <a name="general-purpose-service-tier"></a>一般用途服務層

下列清單說明一般用途服務層的主要特色： 

- 專為大多數有標準效能需求的商務應用程式所設計 
- 高效能的 Azure 進階儲存體 (8 TB) 
- 每個執行個體 100 個資料庫 

下列清單概述一般用途服務層的主要特色：

|功能 | 說明|
|---|---|
| 虛擬核心數* | 8 個、16 個、24 個 (第 4 代)<br>8 個、16 個、24 個、32個、40 個、64個、80 個 (第 5 代)|
| SQL Server 版本/組建 | SQL Server 資料庫引擎 (最新穩定版) |
| 儲存體大小下限 | 32 GB |
| 儲存體大小上限 | 8 TB |
| 每個資料庫的儲存體上限 | 取決於每個執行個體的最大儲存體大小 |
| 預期的儲存體 IOPS | 每個資料檔案 500-7500 IOPS (視資料檔案而定)。 請參閱[進階儲存體](../virtual-machines/windows/premium-storage-performance.md#premium-storage-disk-sizes) |
| 每個資料庫的資料檔案 (ROWS) 數目 | 多個 | 
| 每個資料庫的記錄檔 (LOG) 數目 | 1 | 
| 受控自動化備份 | 是 |
| HA | Azure 儲存體和 [Azure Service Fabric](../service-fabric/service-fabric-overview.md) 中儲存的資料 |
| 內建執行個體和資料庫的監視與計量 | 是 |
| 自動軟體修補 | 是 |
| VNet - Azure Resource Manager 部署 | 是 |
| VNet - 傳統部署模型 | 否 |
| 入口網站支援 | 是|
|||

\*虛擬核心代表可以選擇使用的邏輯 CPU，可在各硬體世代間進行選擇。 第四代邏輯 CPU 的基礎為 E5-2673 v3 (Haswell) 2.4 GHz 處理器，第五代邏輯 CPU 的基礎為 Intel E5-2673 v4 (Broadwell) 2.3 GHz 處理器。 

如需詳細資訊，請參閱 Azure SQL Database 中的[標準/一般用途可用性和架構](sql-database-high-availability.md#standardgeneral-purpose-availability)。

### <a name="business-critical-service-tier-preview"></a>商務關鍵服務層 (預覽)

業務關鍵服務層是為具有高 IO 需求的應用程式所建置。 該服務層使用數個分開的 Always On 複本，可在發生失敗時提供最高的復原能力。 

下列清單概述業務關鍵服務層的主要特色： 
-   專為具有極高效能和 HA 需求的商務應用程式所設計 
-   提供超級快速的 SSD 儲存體 (在第 4 代上可達 1 TB，而在第 5 代上可達 4 TB)
-   可針對每個執行個體支援多達 100 個資料庫 
- 可用於報告和其他唯讀工作負載的內建其他唯讀執行個體
- [記憶體內部 OLTP](sql-database-in-memory.md)，可用於具有高效能需求的工作負載  

|功能 | 說明|
|---|---|
| 虛擬核心數* | 8 個、16 個、24 個、32個 (第 4 代)<br>8 個、16 個、24 個、32個、40 個、64個、80 個 (第 5 代)|
| SQL Server 版本/組建 | SQL Server (最新的可用版本) |
| 其他功能 | [記憶體內部 OLTP](sql-database-in-memory.md)<br> 1 個額外的唯讀複本 ([讀取縮放](sql-database-read-scale-out.md))
| 儲存體大小下限 | 32 GB |
| 儲存體大小上限 | 第 4 代：1 TB (適用於所有虛擬核心大小)<br> 第 5 代：<ul><li>1 TB (適用於 8 個、16 個虛擬核心)</li><li>2 TB (適用於 24 個虛擬核心)</li><li>4 TB (適用於 32 個、40 個、64 個、80 個 vCore)</ul>|
| 每個資料庫的儲存體上限 | 取決於每個執行個體的最大儲存體大小 |
| 每個資料庫的資料檔案 (ROWS) 數目 | 多個 | 
| 每個資料庫的記錄檔 (LOG) 數目 | 1 | 
| 受控自動化備份 | 是 |
| HA | 資料儲存於本機 SSD，並使用 [Always On 可用性群組](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server)和 [Azure Service Fabric](../service-fabric/service-fabric-overview.md) |
| 內建執行個體和資料庫的監視與計量 | 是 |
| 自動軟體修補 | 是 |
| VNet - Azure Resource Manager 部署 | 是 |
| VNet - 傳統部署模型 | 否 |
| 入口網站支援 | 是|
|||

如需詳細資訊，請參閱 Azure SQL Database 中的[進階/業務關鍵可用性和架構](sql-database-high-availability.md#premiumbusiness-critical-availability)。

> [!IMPORTANT]
> **商務關鍵**服務層處於預覽階段。

## <a name="advanced-security-and-compliance"></a>進階安全性與合規性 

Azure SQL Database 受控執行個體結合了 Azure 雲端與 SQL Server 資料庫引擎所提供的進階安全性功能。 

### <a name="managed-instance-security-isolation-in-azure-cloud"></a>Azure 雲端中受控執行個體的安全性隔離 

受控執行個體提供額外的安全性隔離，可與 Azure 雲端中的其他租用戶隔離。 安全性隔離包括： 

- [實作原生虛擬網路](sql-database-managed-instance-vnet-configuration.md)和使用 Azure Express Route 或 VPN 閘道與內部部署環境連線 
- SQL 端點只會透過私人 IP 位址公開，並允許來自私人 Azure 或混合式網路的安全連線
- 單一租用戶具有專用的基礎結構 (計算、儲存體)

下圖概述您應用程式的各種連線選項： 

![高可用性](./media/sql-database-managed-instance/application-deployment-topologies.png)  

如需深入了解子網路層級的 VNet 整合和網路原則強制施行，請參閱[設定 Azure SQL Database 受控執行個體的 VNet](sql-database-managed-instance-vnet-configuration.md) 和[將應用程式連線到 Azure SQL Database受控執行個體](sql-database-managed-instance-connect-app.md)。 

> [!IMPORTANT]
> 將多個受控執行個體放在相同子網路中 (如果您的安全性需求允許的話)，因為這會帶來額外的好處。 將執行個體放在相同子網路中，可大幅簡化網路基礎結構的維護工作，並且可減少執行個體的佈建時間，因為長時間的佈建期間與在子網路中部署第一個受控執行個體的成本有關。

### <a name="azure-sql-database-security-features"></a>Azure SQL Database 安全性功能

Azure SQL Database 提供一組可用來保護資料的進階安全性功能。

- [受控執行個體稽核](sql-database-managed-instance-auditing.md)會追蹤資料庫事件並將事件寫入您 Azure 儲存體帳戶中的稽核記錄檔。 稽核有助於保持法規遵循、了解資料庫活動，以及深入了解可指出商務考量或疑似安全違規的不一致和異常。 
- 移動中資料加密 - 受控執行個體會使用傳輸層安全性對移動中的資料加密，藉此保護您的資料。 除了傳輸層安全性，SQL Database 受控執行個體會使用 [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 來保護傳輸中、待用和查詢處理期間的敏感性資料。 Always Encrypted 是業界優先，可提供無與倫比的資料安全性，以對抗涉及重要資料竊取的入侵。 例如，透過 Always Encrypted，信用卡號碼會永遠加密儲存在資料庫中，即使在查詢處理期間，都允許需要處理該資料的已授權人員或應用程式在使用時解密。 
- [威脅偵測](sql-database-managed-instance-threat-detection.md)會提供服務內建的額外安全情報層，此情報層可偵測到不尋常且有危害的資料庫存取或攻擊動作，藉此補充[受控執行個體稽核](sql-database-managed-instance-auditing.md)的不足之處。 系統會警示您有關可疑活動、潛在弱點、SQL 插入式攻擊和異常資料庫存取模式。 您可以從 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)檢視威脅偵測警示，該警示會提供可疑活動的詳細資料，以及如何調查與降低威脅的建議。  
- [動態資料遮罩](/sql/relational-databases/security/dynamic-data-masking)可藉由遮罩處理，使不具權限的使用者無法看見敏感性資料。 動態資料遮罩可讓您在應用程式層級受到最小影響的情況下指定要顯示多少敏感性資料，而協助防止未經授權者存取敏感性資料。 它是以原則為基礎的安全性功能，可針對指定的資料庫欄位隱藏查詢結果集中的機密資料，而不變更資料庫中的資料。 
- [資料列層級安全性](/sql/relational-databases/security/row-level-security)讓您能夠根據執行查詢之使用者的特性 (例如，依群組成員資格或執行內容) 來控制資料庫資料表中的資料列存取。 資料列層級安全性 (RLS) 可簡化應用程式安全性的設計和編碼。 RLS 可讓您實作資料的資料列存取限制。 例如，確保背景工作角色只能存取其部門相關資料列，或將資料存取權限制為僅限相關資料。 
- [透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) 會將 Azure SQL Database 受控執行個體的資料檔案加密，也稱為「待用資料加密」。 TDE 會執行資料和記錄檔的即時 I/O 加密和解密。 加密會使用資料庫加密金鑰 (DEK)，此金鑰會儲存在資料庫開機記錄中，以在復原期間提供可用性。 您可以使用透明資料加密來保護受控執行個體中的所有資料庫。 TDE 是 SQL 經實證的靜態加密技術，許多合規性標準都需要這項奇數才能防禦儲存媒體的竊取。 公開預覽版支援自動金鑰管理模型 (由 PaaS 平台執行)。 

透過 Azure 資料庫移轉服務 (DMS) 或原生還原，可支援將加密的資料庫遷移到 SQL 受控執行個體。 如果您打算使用原生還原來遷移加密的資料庫，必須執行將現有 TDE 憑證從 SQL Server 內部部署或 SQL Server VM 移轉至受控執行個體的步驟。 如需移轉選項的詳細資訊，請參閱[將 SQL Server 執行個體移轉至 Azure SQL Database 受控執行個體](sql-database-managed-instance-migrate.md)。

## <a name="azure-active-directory-integration"></a>Azure Active Directory 整合

Azure SQL Database 受控執行個體支援傳統的 SQL Server 資料庫引擎登入以及與 Azure Active Directory (AAD) 整合的登入。 AAD 登入是您使用於內部部署環境的 Azure 雲端版 Windows 資料庫登入。

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Azure Active Directory 整合和多重要素驗證 

受控執行個體可讓您透過 [Azure Active Directory 整合](sql-database-aad-authentication.md)，集中管理資料庫使用者和其他 Microsoft 服務的身分識別。 這項功能簡化了權限管理並增強安全性。 Azure Active Directory 支援[多重要素驗證](sql-database-ssms-mfa-authentication-configure.md) (MFA)，以提高資料和應用程式安全性，同時支援單一登入程序。 

### <a name="authentication"></a>驗證 
受控執行個體驗證是指使用者連線到資料庫時如何證明他們的身分識別。 SQL Database 支援兩種驗證類型：  

- SQL 驗證，其需要使用者名稱和密碼。
- Azure Active Directory 驗證，它會使用由 Azure Active Directory 管理的身分識別，並支援受控和整合的網域。 

### <a name="authorization"></a>Authorization

授權是指使用者可以在 Azure SQL Database 內執行的動作，這是由使用者帳戶的資料庫角色成員資格和物件層級權限所控制。 受控執行個體與 SQL Server 2017 具有相同的授權功能。 

## <a name="database-migration"></a>資料庫移轉 

受控執行個體鎖定的是將大量資料庫從內部部署或 IaaS 資料庫實作移轉的使用者案例。 受控執行個體支援數個資料庫移轉選項： 

### <a name="backup-and-restore"></a>備份與還原  

移轉方法會利用 Azure Blob 儲存體的 SQL 備份。 透過 [T-SQL RESTORE 命令](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql?view=azuresqldb-mi-current)，儲存在 Azure 儲存體 Blob 的備份可以直接用來還原到受控執行個體。 
  - 如需示範如何還原 Wide World Importers - 標準資料庫備份檔案的快速入門，請參閱[還原備份檔案至受控執行個體](sql-database-managed-instance-get-started-restore.md)。 本快速入門顯示，您必須將備份檔案上傳到 Azure Blog 儲存體，並使用共用存取簽章 (SAS) 金鑰保護其安全。
  - 如需從 URL 還原的資訊，請參閱[從 URL 原生還原](sql-database-managed-instance-migrate.md#native-restore-from-url)。

> [!IMPORTANT]
> 來自受控執行個體的備份只能還原至其他受控執行個體。 它們無法還原至內部部署 SQL Server，或是還原至 Azure SQL Database 邏輯伺服器單一或集區資料庫。

### <a name="data-migration-service"></a>資料移轉服務

Azure 資料庫移轉服務是一個完全受控的服務，能夠從多個資料庫來源無縫移轉到 Azure 資料平台，將停機時間降到最低。 此服務可簡化將現有第三方和 SQL Server 資料庫移動至 Azure 時所需的工作。 部署選項包括 Azure SQL Database、受控執行個體，以及 Azure VM 中處於公開預覽階段的 SQL Server。 請參閱[如何使用 DMS 將您的內部部署資料庫遷移至受控執行個體](https://aka.ms/migratetoMIusingDMS)。

## <a name="sql-features-supported"></a>SQL 功能支援 

受控執行個體的目標是在各階段中，為內部部署 SQL Server 提供幾乎 100% 的介面區相容性，直到服務正式運作為止。 如需功能和比較清單，請參閱 [SQL Database 功能比較](sql-database-features.md)，而如需受控執行個體與 SQL Server 的 T-SQL 差異清單，請參閱[受控執行個體與 SQL Server 的 T-SQL 差異](sql-database-managed-instance-transact-sql-information.md)。
 
受控執行個體支援與 SQL 2008 資料庫的回溯相容性。 支援直接從 SQL 2005 資料庫伺服器進行移轉，移轉後，SQL 2005 資料庫的相容性層級會更新為 SQL 2008。 
 
下圖概述受控執行個體中的介面區相容性：  

![移轉](./media/sql-database-managed-instance/migration.png) 

### <a name="key-differences-between-sql-server-on-premises-and-managed-instance"></a>SQL Server 內部部署和受控執行個體之間的主要差異 

受控執行個體的優勢是其在雲端中一律是最新狀態，這表示內部部署 SQL Server 中的某些功能可能已過時、已停用或已有替代方案。 在某些情況，當工具必須辨識特定功能的運作方式稍有不同，或是服務不在某個環境中執行時，您無法完全控制： 

- 高可用性會內建和預先設定。 Always On 高可用性功能的公開方式不會與 SQL IaaS 實作上的相同 
- 自動備份和時間點還原。 客戶可以起始 `copy-only` 備份，這不會干擾自動備份鏈結。 
- 受控執行個體不允許指定完整實體路徑，因此需要以不同的方式支援所有相對應的情節：RESTORE DB 不支援 WITH MOVE、CREATE DB 不允許的實體路徑、BULK INSERT 只能搭配 Azure Blob 使用等等。 
- 受控執行個體支援以 [Azure AD 驗證](sql-database-aad-authentication.md) 作為 Windows 驗證的雲端替代方案。 
- 受控執行個體都會自動為包含記憶體內部 OLTP 物件的資料庫管理 XTP 檔案群組和檔案
- 受控執行個體支援 SQL Server Integration Services (SSIS)，且可主控儲存 SSIS 封裝的 SSIS 目錄 (SSISDB)，但會在 Azure Data Factory (ADF) 中的受控 Azure-SSIS Integration Runtime (IR) 上執行，請參閱[在 ADF 中建立 Azure-SSIS IR](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime) \(英文\)。 若要比較 SQL Database 與受控執行個體中的 SSIS 功能，請參閱[比較 SQL Database 邏輯伺服器與受控執行個體](../data-factory/create-azure-ssis-integration-runtime.md#compare-sql-database-logical-server-and-sql-database-managed-instance)。

### <a name="managed-instance-administration-features"></a>受控執行個體的管理功能  

受控執行個體讓系統管理員可以專注於與商務最相關的事情。 許多系統管理員/DBA 活動其實非必要或相當簡單。 例如，OS / RDBMS 安裝和修補、動態執行個體的大小調整和組態、備份、[資料庫複寫](replication-with-sql-database-managed-instance.md) (包括系統資料庫)、高可用性組態，以及健康情況和效能監視資料流的組態。 

> [!IMPORTANT]
> 如需可支援、部分支援和不支援的功能清單，請參閱[SQL Database 功能](sql-database-features.md)。 如需受控執行個體與 SQL Server 的 T-SQL 差異清單，請參閱[受控執行個體與 SQL Server 的 T-SQL 差異](sql-database-managed-instance-transact-sql-information.md)

### <a name="how-to-programmatically-identify-a-managed-instance"></a>如何以程式設計方式識別受控執行個體

下表顯示數個透過 Transact SQL 使用的屬性，可用來檢測出應用程式正在使用受控執行個體，並擷取重要的屬性。

|屬性|值|註解|
|---|---|---|
|`@@VERSION`|Microsoft SQL Azure (RTM) - 12.0.2000.8 2018-03-07 Copyright (C) 2018 Microsoft Corporation.|此值與 SQL Database 中的相同。|
|`SERVERPROPERTY ('Edition')`|SQL Azure|此值與 SQL Database 中的相同。|
|`SERVERPROPERTY('EngineEdition')`|8|此值只會識別出受控執行個體。|
|`@@SERVERNAME`、`SERVERPROPERTY ('ServerName')`|下列格式的完整執行個體 DNS 名稱：<instanceName>.<dnsPrefix>.database.windows.net，其中 <instanceName> 是客戶提供的名稱，而 <dnsPrefix> 是自動產生的部分名稱，確保全域 DNS 名稱是唯一的 (例如，"wcus17662feb9ce98")|範例：my-managed-instance.wcus17662feb9ce98.database.windows.net|

## <a name="next-steps"></a>後續步驟

- 若要了解如何建立您的第一個受控執行個體，請參閱[快速入門指南](sql-database-managed-instance-get-started.md)。
- 如需功能與比較清單，請參閱 [SQL 的一般功能](sql-database-features.md)。
- 如需 VPN 組態的詳細資訊，請參閱[受控執行個體 VNet 組態](sql-database-managed-instance-vnet-configuration.md)。
- 如需建立受控執行個體，並從備份檔案還原資料庫的快速入門，請參閱[建立受控執行個體](sql-database-managed-instance-get-started.md)。
- 如需使用 Azure 資料庫移轉服務 (DMS) 進行移轉的教學課程，請參閱[使用 DMS 的受控執行個體移轉](../dms/tutorial-sql-server-to-managed-instance.md)。
- 如需價格資訊，請參閱 [SQL Database 受控執行個體的價格](https://azure.microsoft.com/pricing/details/sql-database/managed/)。
