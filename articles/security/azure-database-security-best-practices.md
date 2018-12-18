---
title: Azure 資料庫安全性最佳做法 | Microsoft Docs
description: 此文章提供一組 Azure 資料庫安全性的最佳做法。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/20/2018
ms.author: tomsh
ms.openlocfilehash: 0f738348dd0a000df8b1da299bb7b58ebc5a1165
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040088"
---
# <a name="azure-database-security-best-practices"></a>Azure 資料庫安全性最佳做法
安全性是管理資料庫的最重要考量，而且向來是 [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/) 的優先考量。 您的資料庫可嚴加保護，有助於符合大多數法規或安全性需求，包括 HIPAA、ISO 27001/27002 和 PCI DSS Level 1。 [Microsoft 信任中心網站](http://azure.microsoft.com/support/trust-center/services/)提供目前的安全性合規性認證清單。 您也可以法規要求作為基礎，選擇將資料庫放在特定的 Azure 資料中心。

此文章討論 Azure 資料庫安全性最佳做法的集合。 這些最佳做法衍生自我們的 Azure 資料庫安全性以及如您本身的客戶體驗。

針對每個最佳做法，我們會說明︰

-   最佳作法是什麼
-   您為何想要啟用該最佳作法
-   如果無法啟用最佳作法，結果可能為何
-   如何學習啟用最佳作法

這篇「Azure 資料庫安全性最佳做法」是以共識意見以及 Azure 平台功能和特性集 (因為在撰寫本文時已存在) 作為基礎。 意見和技術會隨著時間改變，這篇文章將會定期進行更新以反映這些變更。

## <a name="use-firewall-rules-to-restrict-database-access"></a>使用防火牆規則來限制資料庫存取
Microsoft Azure SQL Database 為 Azure 和其他網際網路式應用程式提供關聯式資料庫服務。 為了提供存取安全性，SQL Database 使用下列方式來控制存取：

- 依 IP 位址限制連線的防火牆規則。
- 要求使用者證明其身分識別的驗證機制。
- 將使用者侷限於特定動作和資料的授權機制。

防火牆會防止對您資料庫伺服器的所有存取，直到您指定哪些電腦擁有權限。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。

您可在 SQL Database 中設定伺服器防火牆，如下圖所示：

![防火牆規則](./media/azure-database-security-best-practices/azure-database-security-best-practices-Fig1.png)

Azure SQL Database 服務只透過 TCP 連接埠 1433 提供。 若要從您的電腦存取 SQL 資料庫，請務必確認您的用戶端電腦防火牆允許 TCP 連接埠 1433 上的連出 TCP 通訊。 如果其他應用程式不需要這些連線，請使用防火牆規則封鎖 TCP 連接埠 1433 的傳入連線。

連接程序當中，從 Azure 虛擬機器的連接會被重新導向到每個背景工作角色的唯一 IP 位址和連接埠。 連接埠號碼的範圍從 11000 到 11999。 如需 TCP 連接埠的詳細資訊，請參閱[適用於 ADO.NET 4.5 的 1433 以外的連接埠](../sql-database/sql-database-develop-direct-route-ports-adonet-v12.md)。

如需 SQL Database 中防火牆規則的詳細資訊，請參閱 [SQL Database 防火牆規則](../sql-database/sql-database-firewall-configure.md)。

> [!Note]
> 除了 IP 規則，防火牆會管理虛擬網路規則。 虛擬網路規則是以虛擬網路服務端點為基礎。 在某些情況下，相較於 IP 規則，最好使用虛擬網路規則。 若要深入了解，請參閱 [Azure SQL Database 的虛擬網路服務端點和規則](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md)。

## <a name="enable-database-authentication"></a>啟用資料庫驗證
SQL Database 支援兩種類型的驗證，SQL Server 驗證和 Azure AD 驗證。

### <a name="sql-server-authentication"></a>*SQL Server 驗證*

包括以下優點：

- 它可讓 SQL Database 支援具有混合作業系統的環境，其中所有使用者都未獲得 Windows 網域驗證。
- 允許 SQL Database 支援舊版的應用程式，以及需要 SQL Server 驗證的合作夥伴所提供應用程式。
- 可讓使用者從未知或未受信任的網域進行連線。 例如，可供既有客戶與指派之 SQL Server 登入連線，以接收訂單狀態的應用程式。
- 允許 SQL Database 支援 Web 架構應用程式，讓使用者可在其中建立自己的識別。
- 可讓軟體開發人員散發其應用程式，方法是使用以已知預設的 SQL Server 登入作為基礎的複雜權限階層。

> [!NOTE]
> SQL Server 驗證無法使用 Kerberos 安全性通訊協定。
>
>

如果您是使用 SQL Server 驗證，就必須：

- 自行管理強式認證。
- 保護連接字串中的認證。
- (可能) 保護透過網路從 Web 伺服器傳遞至資料庫的認證。 如需詳細資訊，請參閱[如何：使用 ASP.NET 2.0 中的 SQL 驗證連線到 SQL Server](https://msdn.microsoft.com/library/ms998300.aspx)。

### <a name="azure-active-directory-ad-authentication"></a>*Azure Active Directory (AD) 驗證*
Azure AD 驗證是使用 Azure AD 中的身分識別來連線到 Azure SQL Database 和 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)的機制。 您可以使用 Azure AD 驗證，在單一中央位置管理資料庫使用者和其他 Microsoft 服務的身分識別。 中央識別碼管理提供單一位置以管理資料庫使用者並簡化權限管理。

> [!NOTE]
> 建議您優先使用 Azure AD 驗證，而不是使用 SQL Server 驗證。
>
>

包括以下優點：

- 它提供 SQL Server 驗證的替代方案。
- 有助於防止使用者身分識別在資料庫伺服器之間擴散。
- 允許在單一位置變換密碼。
- 客戶可以使用外部 (Azure AD) 群組來管理資料庫權限。
- 它可以透過啟用整合式 Windows 驗證和 Azure Active Directory 支援的其他形式驗證來避免儲存密碼。
- 它會使用自主資料庫使用者，在資料庫層級驗證身分。
- 可針對連線到 SQL Database 的應用程式支援權杖型驗證。
- 支援本機 Azure Active Directory 執行個體的 ADFS (網域同盟) 或原生使用者/密碼驗證，而不需進行網域同步處理。
- Azure AD 支援來自 SQL Server Management Studio 的連線，其中使用包含 Multi-Factor Authentication 的 Active Directory 通用驗證。 多重要素驗證提供增強式驗證功能，其中提供一系列驗證選項，例如電話、簡訊、含有 PIN 的智慧卡或行動應用程式通知。 如需詳細資訊，請參閱[適用於 Azure AD 多重要素驗證與 SQL Database 和 SQL 資料倉儲的 SSMS 支援](../sql-database/sql-database-ssms-mfa-authentication.md)。

設定步驟包括以下設定和使用 Azure AD 驗證的程序：

- 建立和填入 Azure AD。
- 選用：和目前與您 Azure 訂用帳戶相關聯的 Active Directory 執行個體產生關聯或予以變更。
- 建立 Azure SQL Database 或 [Azure SQL 資料倉儲](https://azure.microsoft.com/services/sql-data-warehouse/)的 Azure Active Directory 系統管理員。
- 設定用戶端電腦。
- 在對應至 Azure AD 身分識別的資料庫中建立自主資料庫使用者。
- 使用 Azure AD 身分識別連接到您的資料庫。

您可以在[使用 Azure Active Directory 驗證向 SQL Database、受控執行個體或 SQL 資料倉儲進行驗證](../sql-database/sql-database-aad-authentication.md)中找到詳細資訊。

## <a name="protect-your-data-by-using-encryption"></a>使用加密來保護您的資料
[Azure SQL Database 的透明資料加密](https://msdn.microsoft.com/library/dn948096.aspx)有助於保護磁碟上的資料，並可防止未經授權存取硬體。 它會對資料庫、相關聯的備份和待用的交易記錄檔執行即時加密和解密，而不需變更應用程式。 透明資料加密會使用稱為資料庫加密金鑰的對稱金鑰，來將整個資料庫的儲存體加密。

即使整個儲存體都已加密，也務必要將資料庫本身加密。 這是資料保護的深度防禦方法實作。 如果您是使用 Azure SQL Database，而且需要保護敏感性資料 (例如信用卡或身分證號碼)，可以使用 FIPS 140-2 驗證的 256 位元 AES 加密來加密資料庫。 此加密符合許多產業標準 (例如 HIPAA 與 PCI) 的需求。

當您使用透明資料加密來加密資料庫時，與[緩衝集區延伸 (BPE)](https://docs.microsoft.com/sql/database-engine/configure-windows/buffer-pool-extension)相關的檔案不會加密。 您必須對 BPE 相關檔案使用檔案系統等級的加密工具，像是 [BitLocker](https://technet.microsoft.com/library/cc732774) 或 [加密檔案系統 (EFS)]()。

因為經過授權的使用者 (像是安全性系統管理員或資料庫管理員) 可以存取資料，所以即使已使用透明資料加密將資料庫加密，您也應該遵循下列建議︰

- 啟用資料庫等級的 SQL Server 驗證。
- 使用 [RBAC 角色](../role-based-access-control/overview.md)來使用 Azure AD 驗證。
- 確定使用者和應用程式使用不同的帳戶進行驗證。 如此一來，您可以限制授與使用者和應用程式的權限，並降低惡意活動的風險。
- 使用固定的資料庫角色 (例如 db_datareader 或 db_datawriter) 實作資料庫等級安全性。 或者您可以為應用程式建立自訂角色，以授與明確的權限給選取的資料庫物件。

如需其他的資料加密方式，請考慮：

- [儲存格層級加密](https://msdn.microsoft.com/library/ms179331.aspx) ，可利用不同的加密金鑰來加密特定的資料行或甚至是資料儲存格。
- [Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx) 可讓用戶端在用戶端應用程式中將敏感性資料加密，且永遠不會對資料庫引擎 (SQL Database 或 SQL Server) 顯示加密金鑰。 因此，Always Encrypted 可將資料擁有者 (和可檢視者) 及資料管理者 (但無法存取資料) 分隔開來。
- [資料列層級安全性](https://msdn.microsoft.com/library/dn765131)讓客戶能夠根據執行查詢的使用者特性，以控制對資料庫資料表中的資料列存取權。 (範例特性是群組成員資格和執行內容)。

不使用資料庫層級加密的組織可能更容易受到攻擊，進而危害 SQL Database 中的資料。

若要深入了解 SQL Database 透明資料加密，請閱讀 [Azure SQL Database 的透明資料加密](https://msdn.microsoft.com/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx)一文。

## <a name="enable-database-auditing"></a>啟用資料庫稽核
稽核 SQL Server 資料庫引擎或個別資料庫的執行個體，會牽涉到追蹤和記錄事件。 對於 SQL Server，您可以建立稽核，其中包含伺服器層級事件規格及資料庫層級事件規格的稽核。 稽核的事件可以寫入事件記錄或稽核檔案。

根據管制或安裝的標準需求而定，會有數個 SQL Server 的稽核等級。 SQL Server 稽核提供的工具和程序可用以啟用、儲存及檢視各種伺服器和資料庫物件上的稽核。

[Azure SQL Database 稽核](../sql-database/sql-database-auditing.md)會追蹤資料庫事件並將事件寫入您 Azure 儲存體帳戶中的稽核記錄。

稽核可協助您保持法規合規性、了解資料庫活動，以及發現可能指出商務考量或安全違規的不一致和異常。 稽核可推動遵循法規標準，但不保證符合法規。

若要深入了解資料庫稽核以及如何啟用它，請參閱[開始使用 SQL Database 稽核](../sql-database/sql-database-auditing.md)。

## <a name="enable-database-threat-detection"></a>啟用資料庫威脅偵測
威脅保護不只是偵測而已。 資料庫威脅保護包括：

- 對於最具敏感性的資料進行探索與分類，讓您可以保護資料。
- 在資料庫上實作安全設定，讓您可以保護資料庫。
- 可在潛在威脅發生時隨即偵測出並予以回應，讓您可以快速地回應與補救。

**最佳做法**：探索、分類資料庫中的敏感性資料並加上標籤。   
**詳細資料**：可啟用 Azure SQL Database 中的[資料探索與分類](../sql-database/sql-database-data-discovery-and-classification.md)，將 SQL 資料庫中的資料分類。 您可以監視 Azure 儀表板中敏感性資料的存取或下載報表。

**最佳做法**：追蹤資料庫弱點，讓您可以主動改進資料庫安全性。   
**詳細資料**：使用 Azure SQL Database [漏洞評量](../sql-database/sql-vulnerability-assessment.md)服務，掃描是否有潛在的資料庫弱點。 此服務採用的規則知識庫會對安全性弱點加上旗標，並顯示偏離最佳做法的情況，例如設定錯誤、權限過高以及敏感性資料未受保護。

規則是以 Microsoft 的最佳做法為基礎，並著重於顯示資料庫和其珍貴資料的最大安全性問題風險。 它們涵蓋了資料庫層級的問題以及伺服器層級的安全性問題，例如伺服器防火牆設定和伺服器層級權限。 這些規則也代表管理機關的許多要求以符合其合規性標準。

**最佳做法**：啟用威脅偵測功能。  
**詳細資料**：啟用 Azure SQL Database[威脅偵測](../sql-database/sql-database-threat-detection.md)以取得有關如何調查與降低威脅的安全性警示和建議。 您會收到可疑活動、潛在弱點、SQL 插入式攻擊以及異常資料庫存取和查詢模式的警示。

[進階威脅防護](../sql-database/sql-advanced-threat-protection.md)是進階 SQL 安全性功能的整合套件。 其中包含先前所述的服務：資料探索與分類、漏洞評量及威脅偵測。 此套件可讓您從單一點位置啟用及管理前述功能。

啟用這些功能，可協助您：

- 符合資料隱私標準和法規合規性需求。
- 控制資料庫的存取，並強化其安全性。
- 監視難以追蹤變更的動態資料庫環境。
- 偵測並回應潛在威脅。

此外，威脅偵測整合警示與 Azure 資訊安全中心，可以集中檢視所有 Azure 資源的安全性狀態。

## <a name="next-steps"></a>後續步驟
如需更多安全性最佳做法，請參閱 [Azure 安全性最佳做法與模式](security-best-practices-and-patterns.md)，以便在使用 Azure 設計、部署和管理雲端解決方案時使用。

下列資源可提供更多有關 Azure 安全性和相關 Microsoft 服務的一般資訊：
* [Azure 安全性部落格](https://blogs.msdn.microsoft.com/azuresecurity/) - Azure 安全性的最新資訊
* [Microsoft 安全性回應中心](https://technet.microsoft.com/library/dn440717.aspx) - 可在其中回報 Microsoft 安全性弱點 (包括 Azure 的問題) 或透過電子郵件傳送給 secure@microsoft.com
