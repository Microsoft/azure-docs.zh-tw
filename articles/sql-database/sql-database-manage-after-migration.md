---
title: 在移轉後進行管理 - Azure SQL Database | Microsoft Docs
description: 了解如何在移轉至 Azure SQL Database 之後管理資料庫。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: joesackmsft
ms.author: josack
ms.reviewer: carlrab
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 6dd4aacddfbce3e06c1ea9a356a559cc8cd8049c
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166481"
---
# <a name="new-dba-in-the-cloud--managing-your-database-in-azure-sql-database"></a>雲端中的新 DBA - 在 Azure SQL Database 中管理您的資料庫

從傳統的自我管理、自我控管的環境轉移至 PaaS 環境，一開始會讓人有點不知所措。 身為應用程式開發人員或 DBA，建議您熟知可協助您讓應用程式持續保持可用、高效能、安全且有彈性的平台核心功能。 本文章的用意就是如此。 本文章簡潔地整理了資源，並提供一些指引，說明如何善用 SQL Database 的重要功能來管理應用程式並讓其有效率地在雲端中執行及達到最佳結果。 本文章的典型對象是： 
- 正在評估將其應用程式移轉到 Azure SQL DB 的使用者 - 將您的應用程式現代化。
- 正處於移轉其應用程式的使用者 - 持續中的移轉案例。
- 最近完成移轉至 Azure SQL DB 的使用者 - 雲端中的新 DBA。

本文章會討論一些 Azure SQL DB 做為平台的核心特性，讓您可以輕易地運用。 這些特性如下所示： 
- 業務持續性和災害復原 (BCDR)
- 安全性與合規性
- 智慧型資料庫監視和維護
- 資料移動

## <a name="business-continuity-and-disaster-recovery-bcdr"></a>業務持續性和災害復原 (BCDR)
業務持續性和災害復原功能可讓您在發生災害時如往常一樣持續您的業務。 災害可以是資料庫層級事件 (例如，某人不小心卸除重要資料表) 或資料中心層級事件 (區域性災難，例如海嘯)。 

### <a name="how-do-i-create-and-manage-backups-on-sql-database"></a>如何建立及管理 SQL Database 上的備份？
您未在 Azure SQL DB 上建立備份，而這是因為您不需要。 SQL Database 會自動為您備份資料庫，因此您不再需要擔心排程、取得及管理備份。 該平台會每週進行完整備份，每幾小時進行差異備份，以及每隔 5 分鐘進行記錄備份，以確保災害復原有效率並且資料遺失最少。 在您建立資料庫時，就會開始進行第一個完整備份。 這些備份可供您使用一段時間 (稱為「保留期限」)，而且可能因您選擇的服務層而異。 SQL Database 使用[時間點復原 (PITR)](sql-database-recovery-using-backups.md#point-in-time-restore)，讓您能夠還原為此保留期間內的任何時間點。

|服務層|保留期間 (天)|
|---|:---:|
|基本|7|
|標準|35|
|進階|35|
|||

此外，[長期保留 (LTR)](sql-database-long-term-retention.md) 功能可讓您保留備份檔案更長期間 (具體而言，最長為 10 年)，並在該期間內從這些備份的任何時間點還原資料。 更重要的是，資料庫備份會保存在異地複寫儲存體，以確保從區域災難的恢復功能。 您也可以在保留期限內於任何時間點還原任何 Azure 區域的這些備份。 請參閱[業務持續性概觀](sql-database-business-continuity.md)。

### <a name="how-do-i-ensure-business-continuity-in-the-event-of-a-datacenter-level-disaster-or-regional-catastrophe"></a>如何確保發生資料中心層級災害或區域性災難時的業務持續性？
因為您的資料庫備份是儲存在異地複寫儲存體子系統，以確保發生地區性災難時，您可以將備份還原到另一個 Azure 區域。 這稱為異地還原。 此動作的 RPO (復原點目標) 通常是 < 1 小時，而 ERT (預估復原時間) 則是幾分鐘到幾小時。

針對關鍵任務資料庫，Azure SQL DB 提供主動式異地複本。 此動作基本上執行的動作是在另一個區域中對原始資料庫建立異地複寫的次要副本。 例如，如果資料庫最初裝載於 Azure 美國西部地區，而且您想要擁有地區性的災害恢復功能。 您會建立美國西部資料庫的主動式異地複本，假設為美國東部。 當美國西部遭遇災難時，您可以容錯移轉至美國東地區。 在自動容錯移轉群組中設定它們則更好，因為這可確保在發生災難時，資料庫會自動容錯移轉至美國東部的次要複本。 此動作的 RPO 為 < 5 秒，而 ERT 則為 < 30 秒。

如果未設定自動容錯移轉群組，您的應用程式則需要主動監視災害，並開始容錯移轉至次要資料庫。 您可以在不同 Azure 區域中建立最多 4 個這類的主動式異地複本。 這麼做更好。 您也可以以唯讀存取權來存取這些次要主動式異地複本。 對於地理分散的應用程式案例，此方式對助於降低延遲。 

### <a name="how-does-my-disaster-recovery-plan-change-from-on-premises-to-sql-database"></a>我的災害復原計劃如何從內部部署變更為 SQL Database？
總而言之，傳統的內部部署 SQL Server 設定要求您使用容錯移轉叢集、資料庫鏡像、交易複寫、記錄傳送等之類功能來主動管理您的可用性，並維護和管理備份以確保業務持續性。 有了 SQL Database，平台會為您管理這些工作，讓您可以專注於開發和最佳化資料庫應用程式，而不必擔心災害管理。 您可以設定備份和災害復原計劃，且在 Azure 入口網站上按幾下滑鼠 (或使用 PowerShell API 上執行幾個命令) 即可運作。 

若要深入了解災害復原，請參閱：[Azure SQL 資料庫災害復原 101](https://azure.microsoft.com/blog/azure-sql-databases-disaster-recovery-101/)

## <a name="security-and-compliance"></a>安全性與合規性
SQL Database 謹慎對待安全性和隱私權。 SQL Database 於資料庫層級與平台層級提供安全性，而分成數個層時最易於了解。 您可以在每個層進行控制，並為應用程式提供最佳的安全性。 這些層為：
- 身分識別與驗證 ([Windows/SQL 驗證和 Azure Active Directory [AAD] 驗證](sql-database-control-access.md))。
- 監視活動 ([稽核](sql-database-auditing.md)和[威脅偵測](sql-database-threat-detection.md))。
- 保護實際資料 ([透明資料加密 [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)和 [Always Encrypted [AE]](/sql/relational-databases/security/encryption/always-encrypted-database-engine))。 
- 控制對於敏感和特殊權限資料 ([資料列層級安全性](/sql/relational-databases/security/row-level-security)和[動態資料遮罩](/sql/relational-databases/security/dynamic-data-masking)) 的存取權。

[Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)提供跨越在 Azure、內部部署和其他雲端中執行之工作負載的集中式安全性管理。 您可以檢視是否已在所有資源上設定必要的 SQL Database 保護 (例如[稽核](sql-database-auditing.md)和[透明資料加密 [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql))，並根據自己的需求建立原則。

### <a name="what-user-authentication-methods-are-offered-in-sql-database"></a>SQL Database 提供哪些使用者驗證方法？
SQL Database 提供[兩個驗證方法](sql-database-control-access.md#authentication)： 
- [Azure Active Directory 驗證](sql-database-aad-authentication.md)
- SQL 驗證。 

不支援傳統 Windows 驗證。 Azure Active Directory (AD) 是集中式身分識別和存取管理服務。 利用此服務，您可以非常輕鬆地為組織中的所有人員提供單一登入存取 (SSO)。 這表示認證會在所有 Azure 服務間共用，以簡化驗證。 AAD 支援 [MFA (多重要素驗證) ](sql-database-ssms-mfa-authentication.md)，並且[按幾下滑鼠](../active-directory/hybrid/how-to-connect-install-express.md)，AAD 即可與 Windows Server Active Directory 整合。 SQL 驗證的運作方式正如同您過去使用它的方式。 您提供使用者名稱/密碼，而您可以向指定邏輯伺服器上的任何資料庫驗證使用者。 這也可讓 SQL Database 和 SQL 資料倉儲提供多重要素驗證和 Azure AD 網域內的來賓使用者帳戶。 如果您已經有內部部署 Active Directory，您可以使目錄與 Azure Active Directory 結成同盟，將您的目錄延伸至 Azure。

|**如果您...**|**SQL Database/SQL 資料倉儲**|
|---|---|
|不想在 Azure 中使用 Azure Active Directory (AD)|使用 [SQL 驗證](sql-database-security-overview.md)|
|已在內部部署 SQL Server 上使用 AD|[使 AD 與 Azure AD 結成同盟](../active-directory/hybrid/whatis-hybrid-identity.md)，並使用 Azure AD 驗證。 如此一來，您即可使用單一登入。|
|需要強制執行多重要素驗證 (MFA)|透過 [Microsoft 條件式存取](sql-database-conditional-access.md)要求 MFA 作為原則，並使用[具有 MFA 支援的 Azure AD 通用驗證](sql-database-ssms-mfa-authentication.md)。|
|具有來自 Microsoft 帳戶 (live.com、outlook.com) 或其他網域 (gmail.com) 的來賓帳戶|在 SQL Database/資料倉儲中使用 [Azure AD 通用驗證](sql-database-ssms-mfa-authentication.md)，它會利用 [Azure AD B2B 共同作業](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md)。|
|已使用 Azure AD 認證從同盟網域登入 Windows|使用 [Azure AD 整合式驗證](sql-database-aad-authentication-configure.md)。|
|已使用認證從不與 Azure 同盟的網域登入 Windows|使用 [Azure AD 整合式驗證](sql-database-aad-authentication-configure.md)。|
|具有必須連線到 SQL Database 或 SQL 資料倉儲的中介層服務|使用 [Azure AD 整合式驗證](sql-database-aad-authentication-configure.md)。|
|||

### <a name="how-do-i-limit-or-control-connectivity-access-to-my-database"></a>如何限制或控制對我的資料庫的連線存取？
有多個技術隨時可供您用來讓應用程式達到最佳連線能力。 
- 防火牆規則
- VNET 服務端點
- 保留的 IP

#### <a name="firewall"></a>防火牆
藉由僅允許特定實體存取您的邏輯伺服器，防火牆可防止外部實體存取您的伺服器。 預設將禁止邏輯伺服器內的所有連線和資料庫，除非連線是來自其他 Azure 服務。 利用防火牆規則，您可以僅對核准的實體 (例如，開發人員電腦) 開放對伺服器的存取，方法是允許該電腦的 IP 位址通過防火牆。 也可讓您指定想要允許存取邏輯伺服器的某個範圍 IP。 比方說，可以在 [防火牆設定] 頁面中指定範圍，一次新增您組織中開發人員電腦的 IP 位址。 

您可以在伺服器層級或資料庫層級建立防火牆規則。 可以透過入口網站或透過 SSMS 建立伺服器層級防火牆規則。 若要深入了解如何設定伺服器和資料庫層級防火牆規則，請參閱：[在 SQL Database 中建立防火牆規則](sql-database-security-tutorial.md#create-a-server-level-firewall-rule-in-the-azure-portal)。

#### <a name="service-endpoints"></a>服務端點
根據預設，SQL 資料庫會設定為 [允許 Azure 服務存取伺服器]；這表示 Azure 中之任何虛擬機器都可以嘗試連線到您的資料庫。 這些嘗試仍需要取得驗證。 不過，如果您不想要資料庫可供任何 Azure IP 存取，則可以停用 [允許 Azure 服務存取伺服器]。 此外，您可以設定 [VNET 服務端點](sql-database-vnet-service-endpoint-rule-overview.md)。

服務端點 (SE) 可讓您僅對自己在 Azure 中的私人虛擬網路公開您的重要 Azure 資源。 如此一來，基本上可排除對您的資源的公用存取。 您的虛擬網路與 Azure 之間的流量會保持在 Azure 骨幹網路上。 若沒有 SE，則會強制執行通道封包路由。 您的虛擬網路會強制對您的組織的網際網路流量與 Azure 服務流量通過相同的路由。 利用服務端點，您可以將此最佳化，因為封包會直接從您的虛擬網路通往 Azure 骨幹網路上的服務。

![VNET 服務端點](./media/sql-database-manage-after-migration/vnet-service-endpoints.png) 

#### <a name="reserved-ips"></a>保留的 IP
另一個選項就是為您的 VM 佈建[保留的 IP](../virtual-network/virtual-networks-reserved-public-ip.md)，並將伺服器防火牆設定中的特定 VM IP 位址列入允許清單。 藉由指派保留的 IP，即可免去必須以不斷變更的 IP 位址更新防火牆規則的麻煩。

### <a name="what-port-do-i-connect-to-sql-database-on"></a>我會透過哪個連接埠連接到 SQL Database？

連接埠 1433。 SQL Database 會透過此連接埠來通訊。 若要從公司網路內連接，您必須在組織的防火牆設定中新增輸出規則。 一般來說，請避免公開超出 Azure 界限的連接埠 1433。 您可以使用 [Azure RemoteApp](https://www.microsoft.com/cloud-platform/azure-remoteapp-client-apps) 在 Azure 中執行 SSMS。 這麼一來，您不需要開放連接埠 1433 的連出連線，IP 是靜態的，所以可以只對 RemoteApp 開放資料庫，並且可支援多重要素驗證 (MFA)。

### <a name="how-can-i-monitor-and-regulate-activity-on-my-server-and-database-in-sql-database"></a>如何監視和管理我的伺服器和 SQL Database 中資料庫上的活動？
#### <a name="sql-database-auditing"></a>SQL Database 稽核
使用 SQL Database，您可以將稽核開啟以追蹤資料庫事件。 [SQL Database 稽核](sql-database-auditing.md)會追記錄料庫事件並將事件寫入您 Azure 儲存體帳戶中的稽核記錄檔。 如果您想要了解潛在的安全性與原則違規、維護法規合規性等等，稽核特別有用。它可讓您定義及設定您認為需要稽核的某些事件類別，並據以取得預先設定的報表和儀表板，以獲得資料庫上所發生事件的概觀。 您可以在資料庫層級或伺服器層級套用這些稽核原則。 如需有關如何為您的伺服器/資料庫開啟稽核的指南，請參閱：[啟用 SQL Database 稽核](sql-database-security-tutorial.md#enable-sql-database-auditing-if-necessary)。

#### <a name="threat-detection"></a>威脅偵測
利用[威脅偵測](sql-database-threat-detection.md)，您就能非常輕鬆地因應稽核所探索到的安全性或原則違規。 您不需要是安全性專家，也能解決您的系統中的潛在威脅或違規。 威脅偵測也有一些內建的功能，例如 SQL 插入式攻擊偵測。 SQL 插入式攻擊是一種變更或危害資料的嘗試，並且通常是攻擊資料庫應用程式的很常見方式。 SQL Database 威脅偵測會執行多組演算法，以偵測潛在弱點和 SQL 插入式攻擊，以及異常資料庫存取模式 (例如從不尋常的位置存取或由不熟悉的主體存取)。 如果在資料庫上偵測到威脅，安全性人員或其他指定的管理員會收到一封電子郵件通知。 每個通知都會提供可疑活動的詳細資料，以及建議如何進一步調查並減輕威脅。 若要了解如何開啟威脅偵測，請參閱：[啟用 SQL Database 威脅偵測](sql-database-security-tutorial.md#enable-sql-database-threat-detection)。 
### <a name="how-do-i-protect-my-data-in-general-on-sql-database"></a>一般而言，我如何在 SQL Database 上保護資料？
加密提供強式機制來保護及保全您的敏感性資料不被入侵者侵害。 若沒有解密金鑰，加密的資料對入侵者是沒有用處的。 因此，它會在 SQL Database 中建立的現有安全性層級之上新增一層額外的保護。 在 SQL Database 中保護資料有兩個層面： 
- 資料和記錄檔中的待用資料 
- 您的資料在傳輸中。 

依預設，在 SQL Database 中，儲存子系統上的資料和記錄檔中的待用資料是完整且一律會透過[透明資料加密 [TDE]](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) 加密。 也會加密您的備份。 利用 TDE，存取此資料的應用程式端不需進行任何變更。 加密與解密會透明地進行，因此以此為名。 為了保護您的執行中和待用敏感性資料，SQL Database 提供稱為 [Always Encrypted (AE)](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 的功能。 AE 是一種用戶端加密形式，會將您資料庫中的敏感性資料行加密 (所以它們會在資料庫管理員和未經授權使用者的加密文字中)。 伺服器會接收加密的資料來開始處理。 Always Encrypted 金鑰也會儲存在用戶端，因此只有獲得授權的用戶端可以將敏感性資料行解密。 伺服器和資料管理員無法查看敏感性資料，因為加密金鑰存放在用戶端上。 AE 會加密端對端資料表中的敏感性資料行，從未經授權的用戶端至實體磁碟。 AE 目前支援相等比較，所以 DBA 可以繼續在其 SQL 命令中查詢加密的資料行。 Always Encrypted 可以搭配各種金鑰存放區選項使用，例如 [Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md)、Windows 憑證存放區和本機硬體安全性模組。

|**特性**|**一律加密**|**透明資料加密**|
|---|---|---|
|**加密範圍**|端對端|待用資料|
|**資料庫伺服器可以存取敏感性資料**|否|是，因為加密用於待用資料|
|**允許的 T-SQL 作業**|相等比較|所有 T-SQL 介面區域可供使用|
|**使用此功能需要進行應用程式變更**|有限|非常有限|
|**加密資料細微度**|資料行層級|資料庫層級|
||||

### <a name="how-can-i-limit-access-to-sensitive-data-in-my-database"></a>如何才能限制存取我資料庫中的敏感性資料？
每個應用程式的資料庫中會有某些敏感性資料需要加以保護，不讓每個人都看見。 組織內的特定人員可能需要檢視此資料，但其他人卻不應該能夠檢視此資料。 其中一個範例是員工薪資。 經理需要存取其直屬員工的薪資資訊，不過，個別團隊成員不應該可存取其同儕的薪資資訊。 另一個案例是可能會在開發階段或測試期間與敏感性資料互動的資料開發人員，例如，客戶的 SSN。 重申，此資訊不需公開給開發人員。 在這種情況下，可能需要將您的敏感性資料遮罩，或完全不公開。 SQL Database 提供兩個這類方法來防止未經授權的使用者檢視敏感性資料：

[動態資料遮罩](sql-database-dynamic-data-masking-get-started.md)是一項資料遮罩功能，藉由將敏感性資料進行遮罩處理，使得應用程式層上不具權限的使用者看不見這些資料來限制資料的公開方式。 您可以定義可以建立遮罩模式的遮罩規則 (例如，只顯示國民身分證 SSN 的最後四位數字並將其大部分標示為 X：XXX-XX-0000)，並識別可以從遮罩規則中排除的使用者。 遮罩會快速發生，並且有各種不同遮罩函數可供各種資料類別使用。 動態資料遮罩可讓您自動偵測資料庫中的敏感性資料，並對其套用遮罩。

[資料列層級安全性](/sql/relational-databases/security/row-level-security)可讓您控制資料列層級的存取。 這表示，根據執行查詢之使用者 (群組成員資格或執行環境)，會隱藏資料庫資料表中的某些資料列。 在應用程式層 (而非資料庫層) 上進行存取限制，可簡化您的應用程式邏輯。 您可以從建立篩選述詞開始，篩選掉不要公開的資料列，接著使用安全性原則來定義可以存取這些資料列的使用者。 最後，終端使用者執行查詢，並且，根據使用者的權限，他們可以檢視這些受限制的資料列或完全無法看到這些資料。

### <a name="how-do-i-manage-encryption-keys-in-the-cloud"></a>如何管理雲端的加密金鑰？

Always Encrypted (用戶端加密) 和透明資料加密 (待用資料加密) 有金鑰管理選項。 建議您定期輪替使用加密金鑰。 輪替頻率應符合您的組織內部法規和合規性需求。

**透明資料加密 (TDE)**：TDE 有雙重金鑰階層 – 每個使用者資料庫中的資料是由對稱 AES-256 資料庫唯一的資料庫加密金鑰 (DEK) 加密，接著由伺服器唯一的非對稱 RSA 2048 主要金鑰加密。 主要金鑰的管理方式可以是以下兩個方式之一：
- 由平台 - SQL Database 自動執行。
- 或由您使用 [Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md) 作為金鑰存放區。

根據預設，為了方便起見，透明資料加密的主要金鑰是由 SQL Database 服務管理。 如果您的組織想要控管主要金鑰，有一個選項可將 Azure Key Vault](sql-database-always-encrypted-azure-key-vault.md) 當作金鑰存放區。 使用 Azure Key Vault，您的組織即可取得金鑰佈建、輪替和權限控制權。 [輪替或切換 TDE 主索引鍵的類型](/sql/relational-databases/security/encryption/transparent-data-encryption-byok-azure-sql-key-rotation)很快速，因為它只會將 DEK 重新加密。 若為區隔安全性與資料管理之間角色的組織，安全性管理員可以為 Azure Key Vault 中的 TDE 主要金鑰佈建金鑰內容，並將 Azure Key Vault 金鑰識別碼提供給資料庫管理員，以便用於在伺服器上進行待用資料加密。 Key Vault 的設計能使得 Microsoft 無法看見或擷取您的加密金鑰。 您也可以因此集中管理組織的金鑰。 

**Always Encrypted**：Always Encrypted 中也有[雙重金鑰階層](/sql/relational-databases/security/encryption/overview-of-key-management-for-always-encrypted) - 敏感性資料的資料行是由 AES 256 資料行加密金鑰 (CEK) 加密，接著由資料行主要金鑰 (CMK) 加密。 針對 Always Encrypted 提供的用戶端驅動程式沒有 CMK 長度限制。 CEK 的加密值會儲存在資料庫中，而 CMK 會儲存在受信任的金鑰存放區中，例如 Windows 憑證存放區、Azure Key Vault 或硬體安全性模組。 
- [CEK 和 CMK](/sql/relational-databases/security/encryption/rotate-always-encrypted-keys-using-powershell)可以輪替。 
- CEK 輪替是資料作業的大小，並視含有加密資料行的資料表大小而定，可能花費大量時間。 因此，最好據此規劃 CEK 輪替。 
- 不過，CMK 輪替不會干擾資料庫效能，可以透過區隔的角色完成。
下圖顯示 Always Encrypted 中資料行主要金鑰的金鑰存放區選項

![Always Encrypted CMK 存放區提供者](./media/sql-database-manage-after-migration/always-encrypted.png)

### <a name="how-can-i-optimize-and-secure-the-traffic-between-my-organization-and-sql-database"></a>如何最佳化及保護我的組織與 SQL Database 之間的流量？
您的組織與 SQL Database 之間的網路流量通常會透過公用網路路由傳送。 不過，如果您選擇將此路徑最佳化，並使其更安全，您可以研究一下 Express Route。 Express Route 基本上可讓您透過私人連線將公司網路延伸至 Azure 平台。 如此一來，您不會經過公用網際網路。 您也會獲得較高的安全性、可靠性和轉譯至較低網路延遲的路由最佳化，以及比您一般透過公用網際網路會遇到更快的速度。 如果您計劃在組織與 Azure 之間傳輸大量資料區塊，使用 Express Route 可以產生成本優勢。 您可以從您的組織連線至 Azure 的三種不同的連線模型中選擇： 
- [雲端 Exchange 共置](../expressroute/expressroute-connectivity-models.md#CloudExchange)
- [任意對任意](../expressroute/expressroute-connectivity-models.md#IPVPN)
- [點對點](../expressroute/expressroute-connectivity-models.md#Ethernet)

Express Route 也可讓您高載至所購買的頻寬限制最多 2 倍，不額外收費。 您也可以使用 Express Route 設定跨區域連線。 若要查看 ER 連線提供者的清單，請參閱：[Express Route 合作夥伴和對等互連位置](../expressroute/expressroute-locations.md)。 下列文章更詳細說明 Express Route：
- [Express Route 簡介](../expressroute/expressroute-introduction.md)
- [先決條件](../expressroute/expressroute-prerequisites.md)
- [工作流程](../expressroute/expressroute-workflows.md)

### <a name="is-sql-database-compliant-with-any-regulatory-requirements-and-how-does-that-help-with-my-own-organizations-compliance"></a>SQL Database 是否符合任何法規需求，以及這對於自己組織的合規性有何幫助？
SQL Database 符合各種法規規範。 若要檢視已符合的最新一組規範，請瀏覽 [Microsoft 信任中心](https://microsoft.com/trustcenter/compliance/complianceofferings)，並向下切入至對您的組織而言很重要的規範，以查看 SQL Database 是否包含在合規的 Azure 服務之下。 請務必注意，雖然 SQL Database 可能被公認為合規的服務，這有助於您組織的服務合規性，但不自動保證這點。

## <a name="intelligent-database-monitoring-and-maintenance-after-migration"></a>移轉之後的智慧型資料庫監視及維護

一旦資料庫移轉至 SQL Database，即可以監視您的資料庫 (例如，檢查資源使用量的情況或是 DBCC 檢查)，並執行定期維護 (例如，重建或重新組織索引、統計資料等)。 幸運的是，SQL Database 在使用歷史趨勢和記錄的度量資訊與統計資料方面的概念上非常有智慧，可主動協助您監視及維護資料庫，使得您的應用程式永遠以最佳方式執行。 在某些情況下，Azure SQL DB 可以根據組態設定自動執行維護工作。 有三個 Facet 可用來來監視 SQL Database 中的資料庫：
- 效能監視和最佳化。
- 安全性最佳化。
- 成本最佳化。

**效能監控與最佳化**：利用查詢效能深入解析，您可以取得針對您的資料庫工作負載量身訂做的建議，讓您的應用程式以最佳等級持續執行。 您也可加以設定，使得這些建議會自動套用，讓您不必擔心執行維護工作。 利用索引建議程式，您可以根據您的工作負載自動實作索引建議 - 這稱為自動調整。 建議會隨著您的應用程式工作負載變更而發展，以為您提供最相關的建議。 您也可以選擇手動檢閱這些建議，並依您的判斷將其套用。  

**安全性最佳化**：SQL Database 提供可採取動作的安全性建議，以幫助您保護您的資料，以及威脅偵測可用來識別及調查可能會對資料庫造成潛在威脅的可疑資料庫活動。 [SQL 漏洞評估](sql-vulnerability-assessment.md)是一項資料庫掃描和報告服務，可讓您大規模監視資料庫的安全性狀態，以及找出安全性風險和您所定義的安全性基準偏離。 每次掃描後，系統會提供自訂的可行步驟和補救指令碼清單，以及可用於協助符合規範的評估報告。

有了 Azure 資訊安全中心，您可以全面識別安全性建議，並且只要按一下即可將其套用。 

**成本最佳化**：SQL Azure 平台會分析伺服器中不同資料庫的使用量歷程記錄，以為您評估並建議成本最佳化選項。 此分析通常需要兩週的時間來分析及建立可採取動作的建議。 彈性集區是這類的一個選項。 建議會在入口網站上以橫幅形式顯示：

![彈性集區建議](./media/sql-database-manage-after-migration/elastic-pool-recommendations.png) 

您也可以在「建議程式」區段底下檢視此分析：

![彈性集區建議 - 建議程式](./media/sql-database-manage-after-migration/advisor-section.png)

### <a name="how-do-i-monitor-the-performance-and-resource-utilization-in-sql-database"></a>如何在 SQL Database 中監視效能和資源使用率？

在 SQL Database 中，您可以利用平台的智慧型觀點來監視效能並據以調整。 您可以在 SQL Database 中使用下列方法監視效能與資源使用率：
- **Azure 入口網站**：選取資料庫，然後按一下 [概觀] 窗格中的圖表，Azure 入口網站即可顯示單一資料庫的使用率。 您可以修改此圖表來顯示多種計量，包括 CPU 百分比、DTU 百分比、資料 IO 百分比、工作階段百分比，以及資料庫大小百分比。

   ![監視圖表](./media/sql-database-manage-after-migration/monitoring-chart.png)

   ![監視圖表 2](./media/sql-database-manage-after-migration/chart.png)

透過這個圖表，您也可以依照資源設定警示。 這些警示可讓您透過電子郵件回應資源條件、寫入至 HTTPS/HTTP 端點或執行動作。 如需詳細資訊，請參閱[建立警示](sql-database-insights-alerts-portal.md)。

- **動態管理檢視**：您可以查詢 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 動態管理檢視，以傳回過去一小時的資源耗用量統計資料歷程記錄，而查詢 [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) 系統目錄檢視，可傳回過去 14 天的歷程記錄。
- **查詢效能深入解析**：[查詢效能深入解析](sql-database-query-performance.md)可讓您針對特定的資料庫，查看前幾名資源耗用查詢和長時間執行查詢的歷程記錄。 您可以依資源使用量、期間和執行頻率快速識別排名最前面的查詢。 您可以追蹤查詢並偵測迴歸。 這項功能需要啟用[查詢存放區](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)，而且對資料庫有作用。

   ![查詢效能深入解析](./media/sql-database-manage-after-migration/query-performance-insight.png)

- **Log Analytics 中的 Azure SQL 分析 (預覽)**：[Azure Log Analytics](../log-analytics/log-analytics-azure-sql.md) 可讓您收集和視覺化主要的 Azure SQL Azure 效能計量，每個工作區最多支援 150,000 個 SQL Database 和 5,000 個彈性集區。 您可以使用它來監視和收到通知。 您可以監視跨多個 Azure 訂用帳戶和彈性集區的 SQL Database 和彈性集區計量，並可用來識別應用程式堆疊的各層問題。

### <a name="i-am-noticing-performance-issues-how-does-my-sql-database-troubleshooting-methodology-differ-from-sql-server"></a>我注意到效能問題：我的 SQL Database 和 SQL Server 疑難排解方法有何不同？
您用來診斷查詢和進行資料庫效能問題的疑難排解技術的主要部分維持不變。 畢竟雲端是使用相同的 SQL Server 引擎技術。 不過，平台 - Azure SQL DB 具有內建的「智慧」。 它可以協助您更輕鬆地進行疑難排解及診斷效能問題。 它也可以代表您執行其中某些修正動作，並且，在某些情況下，自動地、主動地修正。 

您用來對效能問題進行疑難排解的方法可以大幅獲益，藉由使用智慧型功能，例如結合[查詢效能深入解析 (QPI)](sql-database-query-performance.md) 和 [Database Advisor](sql-database-advisor.md)，使得讓方法中的差異在該方面有所不同 - 您不再需要苦於手動找出可能幫助您對手邊的問題進行疑難排解的基本細節。 平台會為您完成困難的工作。 其中的一個範例是 QPI。 使用 QPI，您可以向下切入直到查詢層級，並查看歷史趨勢，以及找出查詢迴歸的確切時間。 Database Advisor 提供的相關建議事項，可幫助您改善整體效能，一般像是遺漏索引、卸除索引、將查詢參數化等。 

對效能進行疑難排解時，請務必找出影響應用程式效能的是否只是應用程式或是後端資料庫。 往往效能問題會出在應用程式層。 可能是架構或資料存取模式。 例如，假設您有對網路延遲敏感的交談應用程式。 在此情況下，因為在應用程式與伺服器之間有許多往返傳送的簡短要求 (「交談」)，而在網路壅塞時，這些來回會快速增加，使得您的應用程式遭遇困境。 若要改善此情況下的效能，您可以使用[批次查詢](sql-database-performance-guidance.md#batch-queries)。 使用批次對您大有幫助，因為現在系統會以批次處理您的要求，因此可協助您減少來回延遲，並改善您的應用程式效能。 

此外，如果您發現資料庫整體效能降低，您可以監視 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 和 [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) 動態管理檢視，以便了解 CPU、IO 和記憶體耗用量。 您的效能可能受到影響，因為您的資料庫極需要資源。 您可能需要根據成長和縮減的工作負載需求來變更計算大小和/或服務層。 

如需一組完整的調整效能問題相關建議，請參閱[微調資料庫](sql-database-performance-guidance.md#tune-your-database)。

### <a name="how-do-i-ensure-i-am-using-the-appropriate-service-tier-and-compute-size"></a>如何確保我使用適當的服務層和計算大小？
SQL Database 提供三個服務層：基本、標準和進階。 在每個服務層，您會獲得與該服務層保證連結的可預測效能。 根據您的工作負載，您的活動可能會激增，此時的資源使用量可能到達目前所在計算大小上限。 在這種情況下，先從評估進行任何調整 (例如，新增或改變索引等) 是否有幫助很有用。 如果您仍然遇到限制問題，請考慮移到較高的服務層或計算大小。 

|**服務層**|**常見的使用案例**|
|---|---|
|**基本**|應用程式具有少數使用者與一個資料庫，沒有高度並行能力、規模和效能需求。 |
|**標準**|應用程式具有相當大的並行能力、規模和效能需求，結合低到中等的 IO 要求。 |
|**高級**|應用程式具有許多並行使用者、高度 CPU/記憶體和高度 IO 要求。 高度並行能力、高輸送量和對延遲敏感的應用程式可以利用高級層級。 |
|||

若要確定您已採用正確的計算大小，您可以透過上述「如何在 SQL Database 中監視效能和資源使用量？」中之其中一個方式來監視您的查詢和資料庫資源耗用量。 如果您發現您的查詢/資料庫持續耗用 CPU/記憶體等，可以考慮向上增加至更高的計算大小。 同樣地，如果您發現即使在尖峰時刻也未使用一樣多的資源，請考慮從目前的計算大小向下縮減。 

如果您有 SaaS 應用程式模式或資料庫彙總情節，為了成本最佳化，請考慮使用彈性集區。 彈性集區是達到資料庫合併和成本最佳化的好方法。 若要閱讀使用彈性集區管理多個資料庫的相關資訊，請參閱：[管理集區和資料庫](sql-database-elastic-pool-manage.md#azure-portal-manage-elastic-pools-and-pooled-databases)。 

### <a name="how-often-do-i-need-to-run-database-integrity-checks-for-my-database"></a>我必須為我的資料庫執行資料庫完整性檢查的頻率為何？
SQL Database 會使用的一些智慧技術可讓它自動處理特定類別的資料損毀，並且不會遺失任何資料。 服務中內建了這些技術，並且可在需求出現時由服務加以利用。 系統會定期將您在服務間的資料庫備份，藉由還原它們並對其執行 DBCC CHECKDB 來進行測試。 如果有問題，SQL Database 會主動解決它們。 運用[自動頁面修復](/sql/sql-server/failover-clusters/automatic-page-repair-availability-groups-database-mirroring)來修正損毀或有資料完整性問題的頁面。 系統一律會使用可驗證頁面完整性的預設總和檢查碼設定驗證資料庫頁面。 SQL Database 會主動監視並檢閱資料庫的資料完整性，並且，如果發生問題，便以最高優先順序來加以解決。 除了這些，您可以選擇視需要選擇性地執行完整性檢查。  如需詳細資訊，請參閱 [SQL Database 中的資料完整性](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/)

## <a name="data-movement-after-migration"></a>移轉後的資料移動

### <a name="how-do-i-export-and-import-data-as-bacpac-files-from-sql-database"></a>如何以 BACPAC 檔案的形式從 SQL Database 匯出和匯入資料？

- **匯出**：您可以從 Azure 入口網站將 Azure SQL Database 匯出為 BACPAC 檔案

   ![資料庫匯出](./media/sql-database-export/database-export.png)

- **匯入**：您也可以使用 Azure 入口網站將資料以 BACPAC 檔案形式匯入到資料庫。

   ![資料庫匯入](./media/sql-database-import/import.png)

### <a name="how-do-i-synchronize-data-between-sql-database-and-sql-server"></a>如何同步處理 SQL Database 與 SQL Server 之間的資料？
您有幾種方式來達到此目的： 
- **[資料同步](sql-database-sync-data.md)** - 此功能可協助您在多個內部部署 SQL Server 資料庫與 SQL Database 之間雙向同步處理資料。 若要與內部部署 SQL Server 資料庫同步，您需要在本機電腦上安裝並設定同步處理代理程式，以及開放輸出 TCP 連接埠 1433。
- **[交易複寫](https://azure.microsoft.com/blog/transactional-replication-to-azure-sql-database-is-now-generally-available/)** - 您可以利用交易複寫，從內部部署將資料同步處理到 Azure SQL DB，以內部部署作為發行者和 Azure SQL DB 作為訂閱者。 目前僅支援此安裝程式。 如需有關如何將資料從內部部署移轉到 Azure SQL 且具有最少停機時間的詳細資訊，請參閱：[使用交易複寫](sql-database-cloud-migrate.md#method-2-use-transactional-replication)

## <a name="next-steps"></a>後續步驟
了解 [SQL Database](sql-database-technical-overview.md)。

