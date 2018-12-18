---
title: Azure 資料庫安全性概觀 | Microsoft Docs
description: 本文提供 Azure 資料庫安全性功能概觀。
services: security
documentationcenter: na
author: UnifyCloud
manager: mbaldwin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/01/2017
ms.author: TomSh
ms.openlocfilehash: 398594c37cb01cd337da983307a5f56753e54fd2
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574754"
---
# <a name="azure-database-security-overview"></a>Azure 資料庫安全性概觀

安全性是管理資料庫的最重要考量，而且向來是 Azure SQL Database 的優先考量。 Azure SQL Database 支援使用防火牆規則和連線加密的連線安全性。 它支援利用使用者名稱和密碼進行的驗證和 Azure Active Directory (Azure AD) 驗證，後者使用由 Azure Active Directory 所管理的身分識別。 授權使用角色型存取控制。

Azure SQL Database 可在不需變更應用程式的情況下，對靜止的資料庫、相關聯的備份和交易記錄檔執行即時加密和解密，藉此支援加密。

Microsoft 提供其他方法來加密企業資料：

-   資料格層級加密適用於利用不同加密金鑰來加密特定的資料行，甚至是資料格。
-   如果您需要硬體安全模組或集中管理您的加密金鑰階層，請考慮在 Azure 虛擬機器 (VM) 中使用 Azure Key Vault 搭配 SQL Server。
-   Always Encrypted (目前為預覽版本) 讓加密對應用程式透明化。 它也可讓用戶端在用戶端應用程式中將敏感性資料加密，而不需將加密金鑰與 SQL Database 共用。

Azure SQL Database 稽核可讓企業將事件記錄到 Azure 儲存體中的稽核記錄。 SQL Database 稽核也整合了 Microsoft Power BI，具備向下鑽研報表和分析的功能。

Azure SQL 資料庫可嚴加保護，以符合大多數法規或安全性需求，包括 HIPAA、ISO 27001/27002 和 PCI DSS Level 1。 [Microsoft Azure 信任中心網站](http://azure.microsoft.com/support/trust-center/services/)提供目前的安全性合規性認證清單。

本文將逐步解說保護 Microsoft Azure SQL Database 中結構化、表格式和關聯式資料的基本概念。 本文尤其著重於協助您開始利用資源來保護資料、控制存取，以及進行主動式監視。

## <a name="protection-of-data"></a>對資料的保護

SQL Database 透過提供加密來協助您保護資料：

- 適用於透過[傳輸層安全性 (TLS)](https://support.microsoft.com/kb/3135244) \(機器翻譯\) 的運行中資料。
- 適用於透過[透明資料加密](http://go.microsoft.com/fwlink/?LinkId=526242)的待用資料。
- 適用於透過 [Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx) 的使用中資料。

如需其他的資料加密方式，請考慮：

-   [儲存格層級加密](https://msdn.microsoft.com/library/ms179331.aspx) ，可利用不同的加密金鑰來加密特定的資料行或甚至是資料儲存格。
-   [在 Azure VM 中使用 Azure Key Vault 搭配 SQL Server](http://blogs.technet.com/b/kv/archive/2015/01/12/using-the-key-vault-for-sql-server-encryption.aspx) \(英文\)，但前提是您需要硬體安全模組或集中管理您的加密金鑰階層。

### <a name="encryption-in-motion"></a>移動中加密

所有用戶端/伺服器應用程式的一個常見問題，就是透過公用和私人網路移動資料時的隱私權需求。 如果透過網路移動的資料並未加密，就有可能遭到未經授權的使用者擷取和竊取。 當您處理資料庫服務時，請確定會在資料庫用戶端和伺服器之間將資料加密。 也請確定會在要彼此通訊且具有中間層應用程式的資料庫伺服器之間將資料加密。

當您管理網路時，其中一個問題就是保護在不受信任網路上應用程式之間傳送的資料。 您可以使用 [TLS/SSL](https://docs.microsoft.com/windows-server/security/tls/transport-layer-security-protocol) \(機器翻譯\) 來驗證伺服器和用戶端，然後使用它來加密已驗證對象之間的訊息。

在驗證過程中，TLS/SSL 用戶端會將訊息傳送到 TLS/SSL 伺服器。 該伺服器會回應伺服器需要驗證自己的資訊。 用戶端和伺服器會執行其他工作階段金鑰交換，並結束 [驗證] 對話方塊。 完成驗證之後，即可透過驗證程序期間所建立的對稱式加密金鑰，在伺服器與用戶端之間開始進行 SSL 安全保護的通訊。

當資料會來回「傳輸」到資料庫時，Azure SQL Database 的所有連線永遠都需要加密 (TLS/SSL)。 SQL Database 會使用 TLS/SSL 來驗證伺服器和用戶端，然後使用它來將已驗證對象之間的訊息加密。 

在應用程式連接字串中，您必須指定參數來將連線加密，而不是信任伺服器憑證 (如果您從 Azure 入口網站複製出連接字串，即會為您完成此動作)。否則，連線將不會驗證伺服器的身分識別，而且將受到「攔截式」攻擊。 例如，對於 ADO.NET 驅動程式，這些連接字串參數為 `Encrypt=True` 和 `TrustServerCertificate=False`。

### <a name="encryption-at-rest"></a>待用加密
您可以採取幾個預防措施來協助保護資料庫。 例如，設計安全的系統、將機密資產加密，以及在資料庫伺服器周圍建置防火牆。 但是，在實體媒體 (例如磁碟機或備份磁帶) 遭竊的案例中，惡意人士可能會直接還原或連結資料庫，然後瀏覽資料。

其中一個解決方案就是將資料庫中的敏感性資料加密，然後使用憑證來保護用來加密資料的金鑰。 這個解決方案可防止所有沒有金鑰的人使用該資料，但這種保護必須予以規劃。

為了解決這個問題，SQL Server 和 SQL Database 支援[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql?view=azuresqldb-current&viewFallbackFrom=sql-server-2017)。 透明資料加密會將 SQL Server 和 SQL Database 資料檔案加密，也稱為待用資料加密。

透明資料加密有助於防範惡意活動的威脅。 它會對資料庫、相關聯的備份和待用的交易記錄檔執行即時加密和解密，而不需變更應用程式。  

透明資料加密會使用稱為資料庫加密金鑰的對稱金鑰，來將整個資料庫的儲存體加密。 在 SQL Database 中，資料庫加密金鑰是由內建伺服器憑證保護。 內建伺服器憑證對每個 SQL Database 伺服器都是唯一的。

如果資料庫處於異地災害復原關聯性中，則會由每部伺服器上的不同金鑰來保護。 如果兩個資料庫連線到相同的伺服器，則會共用同一個內建憑證。 Microsoft 至少每 90 天會自動替換這些憑證。 

如需詳細資訊，請參閱[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-tde)。

### <a name="encryption-in-use-client"></a>使用中加密 (用戶端)

大多數資料入侵情況與重要資料 (例如信用卡號碼或個人識別資訊) 遭竊相關。 資料庫可能富藏機密資訊。 其中可能包含客戶的個人資料 (例如國民身分證號碼)、機密競爭資訊，以及智慧財產權。 資料遺失或遭竊 (特別是客戶資料) 可能會導致品牌受損、喪失競爭優勢及面臨大量罰金，甚至是訴訟。

![利用鎖和鑰匙來說明的 Always Encrypted 功能](./media/azure-databse-security-overview/azure-database-fig1.png)

[Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx) 功能是設計來保護儲存於 Azure SQL Database 或 SQL Server 資料庫中的敏感性資料。 Always Encrypted 可讓用戶端在用戶端應用程式中將敏感性資料加密，且永遠不會對資料庫引擎 (SQL Database 或 SQL Server) 顯示加密金鑰。

Always Encrypted 可將擁有資料 (且可加以檢視) 的人員與管理資料 (但不應該有任何存取權) 的人員分隔開來。 它有助於確保內部部署資料庫管理員、雲端資料庫操作員或其他具有高權限但未經授權的使用者無法存取加密的資料。

此外，Always Encrypted 讓加密對應用程式透明化。 啟用 Always Encrypted 的驅動程式會安裝於用戶端電腦上，以便自動加密和解密用戶端應用程式中的敏感性資料。 驅動程式會先將敏感性資料行中的資料加密，然後將資料傳遞到資料庫引擎。 驅動程式會自動重寫查詢，以便保留應用程式的語意。 同樣地，驅動程式會將儲存在加密資料庫資料行並包含在查詢結果中的資料明確解密。

## <a name="access-control"></a>存取控制
為了提供安全性，SQL Database 使用下列方式來控制存取：

- 依 IP 位址限制連線的防火牆規則。
- 要求使用者證明其身分識別的驗證機制。
- 將使用者侷限於特定動作和資料的授權機制。

### <a name="database-access"></a>資料庫存取

資料保護是從控制資料存取開始。 裝載您資料的資料中心會管理實體存取。 您可以設定防火牆來管理網路層級的安全性。 您也可以藉由設定驗證登入，並定義伺服器和資料庫角色的權限，來控制存取。

#### <a name="firewall-and-firewall-rules"></a>防火牆與防火牆規則

[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) 可為 Azure 和其他網際網路型應用程式提供關聯式資料庫服務。 為了協助保護您的資料，防火牆會防止對您的資料庫伺服器的所有存取，直到您指定哪些電腦擁有權限。 此防火牆會根據每一個要求的來源 IP 位址來授與資料庫存取權。 如需詳細資訊，請參閱 [Azure SQL Database 防火牆規則概觀](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)。

Azure SQL Database 服務只透過 TCP 連接埠 1433 提供。 若要從您的電腦存取 SQL 資料庫，請務必確認您的用戶端電腦防火牆允許 TCP 連接埠 1433 上的連出 TCP 通訊。 如果其他應用程式不需要連入連線，請在 TCP 通訊埠 1433 上封鎖它們。

#### <a name="authentication"></a>驗證

「驗證」是指連線到資料庫時如何證明身分識別。 SQL Database 支援兩種驗證類型：

-   **SQL Server 驗證**：建立名為 SQL Database 訂閱者帳戶的邏輯 SQL 執行個體時，會建立單一登入帳戶。 此帳戶會使用 [SQL Server 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-security-overview) (使用者名稱和密碼) 來連線。 此帳戶是邏輯伺服器執行個體和附加至該執行個體之所有使用者資料庫的系統管理員。 無法限制訂閱者帳戶的權限。 只有其中一個帳戶可以存在。
-   **Azure Active Directory 驗證**：[Azure AD 驗證](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)是使用 Azure AD 中的身分識別連線到 Azure SQL Database 和 Azure SQL 資料倉儲的機制。 您可以使用它來集中管理資料庫使用者的身分識別。

![使用 SQL Database 的 Azure AD 驗證](./media/azure-databse-security-overview/azure-database-fig2.png)

 Azure AD 驗證的優點包括：
  - 它提供 SQL Server 驗證的替代方案。
  - 它有助於防止使用者身分識別在資料庫伺服器之間擴散，並允許在單一位置進行密碼輪換。
  - 您可以使用外部 (Azure AD) 群組來管理資料庫權限。
  - 它可以透過啟用整合式 Windows 驗證和 Azure AD 支援的其他形式驗證來減少密碼儲存需求。

#### <a name="authorization"></a>Authorization
[授權](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins)是指使用者可在 Azure SQL 資料庫內執行的動作。 它是由您使用者帳戶的資料庫[角色成員資格](https://msdn.microsoft.com/library/ms189121)和[物件層級權限](https://msdn.microsoft.com/library/ms191291.aspx)來控制的。 授權是決定主體可存取哪些安全性實體資源，以及可對這些資源執行哪些作業的程序。

### <a name="application-access"></a>應用程式存取

#### <a name="dynamic-data-masking"></a>動態資料遮罩
客服中心的服務代表可能會透過來電者的社會安全號碼或信用卡號碼中的數個號碼來識別他們。 但不應向服務代表完全公開那些資料項目。

您可以定義遮罩規則，針對任何查詢結果集中社會安全號碼或信用卡號碼末四碼以外的所有數字進行遮罩處理。

![對於在 SQL Database 與商務應用程式之間傳送的號碼進行遮罩處理](./media/azure-databse-security-overview/azure-database-fig3.png)

另一個範例是可定義適當的資料遮罩來保護個人識別資訊。 開發人員接著就能基於疑難排解用途來查詢生產環境，而不會違反合規性法規。

[SQL Database 動態資料遮罩](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)可藉由遮罩處理，使不具權限的使用者無法看見敏感性資料。 Azure SQL Database V12 版可支援動態資料遮罩。

[動態資料遮罩](https://docs.microsoft.com/sql/relational-databases/security/dynamic-data-masking)可讓您在應用程式層級受到最小影響的情況下指定要顯示多少敏感性資料，而協助防止未經授權者存取敏感性資料。 它是以原則為基礎的安全性功能，可針對指定的資料庫欄位隱藏查詢結果集中的機密資料，而不變更資料庫中的資料。


> [!Note]
> 動態資料遮罩可由　Azure 資料庫管理員、伺服器管理員或安全性主管人員等角色來設定。

#### <a name="row-level-security"></a>資料列層級安全性
多租用戶資料庫的另一個常見安全性需求是[資料列層級安全性](https://msdn.microsoft.com/library/dn765131.aspx)。 您可以使用此功能，根據執行查詢的使用者特性，以控制對資料庫資料表中的資料列存取權 (範例特性是群組成員資格和執行內容)。

![允許使用者透過用戶端應用程式存取資料表中資料列的資料列層級安全性](./media/azure-databse-security-overview/azure-database-fig4.png)

存取限制邏輯位於資料庫層，而不是遠離另一個應用程式層中的資料。 資料庫系統會在每次嘗試從任何層存取該資料時套用存取限制。 這可藉由縮小安全性系統的接觸區，讓安全性系統更加可靠和健全。

資料列層級安全性引進述詞型存取控制。 它提供彈性的集中式評估，可將系統管理員認定適當的中繼資料或任何其他準則列入考量。 此述詞會當做準則來使用，以根據使用者屬性，來判斷使用者是否具有資料的適當存取權限。 您可以使用述詞型存取控制來實作標籤型存取控制。

## <a name="proactive-monitoring"></a>主動監視
SQL Database 可透過提供「稽核」和「威脅偵測」功能，協助保護您的資料。

### <a name="auditing"></a>稽核
[Azure SQL Database 稽核](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started)可提高您的能力，以取得資料庫內所發生事件和變更的見解。 範例為對資料進行更新和查詢。

SQL Database 稽核會追蹤資料庫事件，並將它們寫入至您 Azure 儲存體帳戶中的稽核記錄。 稽核可協助您保持法規合規性、了解資料庫活動，以及取得可能指出商務考量或疑似安全違規的不一致和異常見解。 稽核會啟用及推動遵循法規標準，但不保證符合法規。

您可以使用 SQL Database 稽核來：

-   **保留** 所選事件的稽核記錄。 您可以定義要稽核的資料庫動作類別。
-   **報告** 資料庫活動。 您可以使用預先設定的報告和儀表板，以便快速開始使用活動和事件報告。
-   **分析** 報告。 您可以尋找可疑事件、異常活動及趨勢。

稽核方法有兩種：

-   **Blob 稽核**：記錄會寫入至 Azure Blob 儲存體。 這是較新的稽核方法。 此方法可提供更高效能、支援細微度更高的物件層級稽核，且更符合成本效益。
-   **資料表稽核**：記錄會寫入至 Azure 資料表儲存體。

### <a name="threat-detection"></a>威脅偵測
[Azure SQL Database 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection)會偵測可疑的活動，指出潛在的安全性威脅。 您可以使用威脅偵測來回應資料庫中所發生的可疑事件，例如 SQL 插入。 它會提供警示，並允許使用 Azure SQL Database 稽核來探索可疑事件。

![包括外部攻擊者與惡意內部人員對於 SQL Database 和 Web 應用程式的威脅偵測](./media/azure-databse-security-overview/azure-database-fig5.jpg)

例如，SQL 插入是常見的 Web 應用程式安全性問題之一。 它會被用來攻擊資料導向的應用程式。 攻擊者利用應用程式弱點將惡意的 SQL 陳述式插入應用程式輸入欄位，破壞或修改資料庫中的資料。

資訊安全人員或其他指定的系統管理員可以在發生可疑的資料庫活動時立即取得通知。 每個通知都會提供可疑活動的詳細資料，以及建議如何進一步調查並減輕威脅。        

## <a name="centralized-security-management"></a>集中式安全性管理

[Azure 資訊安全中心](https://azure.microsoft.com/documentation/services/security-center/)可協助您保護、偵測威脅並採取相應的措施。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理。 它有助於偵測可能會被忽視的威脅，並可搭配廣泛的安全性解決方案生態系統使用。

[資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-sql-database)藉由提供您所有伺服器和資料庫的安全性可見性，協助您保護 SQL Database 中的資料。 使用資訊安全中心，您可以︰

-   定義 SQL Database 加密和稽核原則。
-   監視所有訂用帳戶的 SQL Database 資源安全性。
-   快速找出並修復安全性問題。
-   整合 [Azure SQL Database 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection)所提供的警示。

資訊安全中心支援角色型存取。

## <a name="azure-marketplace"></a>Azure Marketplace

Azure Marketplace 是線上應用程式和服務市集，可讓新興和獨立軟體廠商 (ISV) 提供解決方案給全球的 Azure 客戶。
Azure Marketplace 將 Microsoft Azure 合作夥伴生態系統結合成整合的平台，為客戶與合作夥伴提供更好的服務。 您可以[執行搜尋](https://azuremarketplace.microsoft.com/marketplace/apps?search=Database%20Security&page=1) \(英文\)，以檢視 Azure Marketplace 中所提供的資料庫安全性產品。

## <a name="next-steps"></a>後續步驟

- [保護 Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-security-tutorial)
- [Azure 資訊安全中心和 Azure SQL Database 服務](https://docs.microsoft.com/azure/security-center/security-center-sql-database)
- [SQL Database 威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection)
- [改善 SQL 資料庫效能](https://docs.microsoft.com/azure/sql-database/sql-database-performance-tutorial)
