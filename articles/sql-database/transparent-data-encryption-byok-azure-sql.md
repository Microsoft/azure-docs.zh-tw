---
title: TDE - 攜帶您自己的金鑰 (BYOK) - Azure SQL Database| Microsoft Docs
description: 透過 Azure Key Vault 將透明資料加密 (TDE) 的「攜帶您自己的金鑰」(BYOK) 支援用於 SQL Database 和資料倉儲。 搭配使用 TDE 與 BYOK 的概觀、優點、運作方式、考量和建議。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: aliceku
ms.author: aliceku
ms.reviewer: vanto
manager: craigg
ms.date: 08/30/2018
ms.openlocfilehash: 5a19afcd713af5b7ea6d8509c7d9645d796bbfad
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162877"
---
# <a name="transparent-data-encryption-with-bring-your-own-key-support-for-azure-sql-database-and-data-warehouse"></a>Azure SQL Database 和資料倉儲具有「攜帶您自己的金鑰」支援的的透明資料加密

[透明資料加密 (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption) 的「攜帶您自己的金鑰」(BYOK) 支援可讓您使用名為 TDE 保護裝置的非對稱金鑰來加密資料庫加密金鑰 (DEK)。  TDE 保護裝置可在您的控制下儲存於 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault) 中，這是 Azure 的雲端式外部金鑰管理系統。 Azure Key Vault 是第一個將 BYOK 的支援整合至 TDE 的金鑰管理服務。 TDE DEK 儲存於資料庫的啟動頁面上，可由 TDE 保護裝置加密和解密。 TDE 保護裝置會儲存在 Azure Key Vault 中，且一律不會移至金鑰保存庫以外。 如果撤銷了伺服器對金鑰保存庫的存取權，資料庫即無法解密並讀取到記憶體中。  TDE 保護裝置可設定於邏輯伺服器層級上，讓所有與該伺服器的相關聯資料庫繼承。 

透過 BYOK 支援，使用者現在可以使用 Azure Key Vault 功能來控制金鑰管理工作，包括金鑰輪替、金鑰保存庫權限、刪除金鑰，以及啟用所有 TDE 保護裝置的稽核/報告功能。 Key Vault 可提供集中管理金鑰的機制、使用嚴密監控的硬體安全性模組 (HSM)，並可區分管理金鑰和資料的職責，以利符合法規合規性。  


搭配使用 TDE 與 BYOK 有下列效益：
- 能夠自我管理 TDE 保護裝置，而提高透明度和精細控制能力   
- 可將 TDE 保護裝置裝載於 Key Vault 中 (連同其他 Azure 服務中使用的其他金鑰和祕密)，而加以集中管理
- 區分組織內部管理金鑰和資料的責任，以利進行職責劃分
- 加深來自您本身用戶端的信任，因為依據 Key Vault 的設計，Microsoft 無法看見或擷取您的加密金鑰。 
- 支援金鑰輪替

> [!IMPORTANT]
> 對於使用服務管理的 TDE 而想要開始使用 Key Vault 的使用者，在切換至 Key Vault 中的 TDE 保護裝置期間，TDE 仍會保持啟用狀態。 既不會有停機時間，資料庫檔案也不會重新加密。 從服務管理的金鑰切換至 Key Vault 金鑰時，只需要重新加密資料庫加密金鑰 (DEK) 即可，而這是快速的線上作業。 
>

## <a name="how-does-tde-with-byok-support-work"></a>具有 BYOK 支援的 TDE 如何運作？
 
![伺服器對 Key Vault 的驗證](./media/transparent-data-encryption-byok-azure-sql/tde-byok-server-authentication-flow.PNG)

當 TDE 第一次設定為使用 Key Vault 中的 TDE 保護裝置時，伺服器會將每個已啟用 TDE 的資料庫 DEK 傳送至 Key Vault，以提出包裝金鑰要求。 Key Vault 會傳回加密的資料庫加密金鑰，並儲存在使用者資料庫中。  

>[!IMPORTANT]
>請務必留意，**一旦 TDE 保護裝置儲存在 Azure Key Vault 中，即不會移至金鑰保存庫以外**。 邏輯伺服器只能對 Key Vault 內的 TDE 保護裝置金鑰內容傳送金鑰作業要求，且**絕不會存取或快取 TDE 保護裝置**。 Key Vault 系統管理員有權隨時撤銷伺服器的 Key Vault 權限，屆時伺服器的所有連線都會中斷。 
>


## <a name="guidelines-for-configuring-tde-with-byok"></a>將 TDE 與 BYOK 搭配運作的設定準則

### <a name="general-guidelines"></a>一般準則
- 確定 Azure Key Vault 和 Azure SQL Database 將會在相同的租用戶中。  目前**不支援**跨租用戶的金鑰保存庫與伺服器互動。
- 決定所需的資源要使用哪些訂用帳戶 – 後續若要在訂用帳戶間移動伺服器，將必須重新設定具有 BYOK 的 TDE。 深入了解如何[移動資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- 設定具有 BYOK 的 TDE 時，請務必考量重複的包裝/解除包裝作業對金鑰保存庫造成的負載。 例如，由於與邏輯伺服器相關聯的所有資料庫都使用相同的 TDE 保護裝置，當該伺服器容錯移轉時將會對保存庫觸發金鑰作業，且其數量會和伺服器中有資料庫時一樣多。 根據我們的經驗和文件中的[金鑰保存庫服務限制](https://docs.microsoft.com/azure/key-vault/key-vault-service-limits)，我們建議您在單一訂用帳戶中將不超過 500 個標準/一般用途或 200 個進階/業務關鍵資料庫與一個 Azure 金鑰保存庫產生關聯，以確保在存取保存庫中的 TDE 保護裝置時可持續保有高可用性。 
- 建議事項：在內部部署中保存 TDE 保護裝置的複本。  為此，您必須以 HSM 裝置在本機建立 TDE 保護裝置，並以金鑰委付系統儲存 TDE 保護裝置的本機複本。  了解[如何將金鑰從本機 HSM 傳輸至 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-hsm-protected-keys)。


### <a name="guidelines-for-configuring-azure-key-vault"></a>設定 Azure Key Vault 的準則

- 建立已啟用[虛刪除](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete)的金鑰保存庫，以防止在意外刪除金鑰 (或金鑰保存庫) 時遺失資料。  您必須對金鑰保存庫使用 [PowerShell 啟用「虛刪除」屬性](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell) (使用 AKV 入口網站時尚不適用此選項 – 但 SQL 則需要此選項)：  
  - 虛刪除的資源預設會保留 90 天的時間，除非您加以復原或清除。
  - **復原**和**清除**動作在金鑰保存庫存取原則中有自己的相關聯權限。 
- 對金鑰保存庫設定資源鎖定，以控制可刪除這項重要資源的人員，並協助防止意外或未經授權的刪除。  [深入了解資源鎖定](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources)

- 使用邏輯伺服器的 Azure Active Directory (Azure AD) 身分識別為其授與金鑰保存庫的存取權。  使用入口網站 UI 時會自動建立 Azure AD 身分識別，並將金鑰保存庫的存取權授與伺服器。  使用 PowerShell 設定具有 BYOK 的 TDE 時，則必須建立 Azure AD 身分識別，並確認作業是否完成。 請參閱[設定具有 BYOK 的 TDE](transparent-data-encryption-byok-azure-sql-configure.md)，以取得使用 PowerShell 時的詳細逐步指示。

  >[!NOTE]
  >如果 Azure AD 身分識別**意外刪除，或使用金鑰保存庫的存取原則撤銷了伺服器的權限**，伺服器即會失去對金鑰保存庫的存取權，且 TDE 加密的資料庫將在 24 小時內遭到捨棄。
  >

- 設定沒有 VNET 或防火牆的 Azure Key Vault。  如果 SQL 失去對金鑰保存庫的存取權，TDE 加密的資料庫將在 24 小時內遭到捨棄。
- 啟用所有加密金鑰的稽核和報告功能：Key Vault 提供可輕易在其他安全性資訊和事件管理 (SIEM) 工具中插入的記錄。 例如，Operations Management Suite (OMS) [Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-azure-key-vault) 即是已整合的服務之一。
- 為確保加密資料庫的高可用性，請使用位於不同區域的兩個 Azure Key Vault 來設定每個邏輯伺服器。


### <a name="guidelines-for-configuring-the-tde-protector-asymmetric-key-stored-in-azure-key-vault"></a>設定會儲存在 Azure Key Vault 中的 TDE 保護裝置 (非對稱金鑰) 的準則

- 在本機 HSM 裝置上建立加密金鑰。 請確定這是對稱的 RSA 2048 金鑰，這樣才能儲存在 Azure Key Vault 中。
- 將金鑰委付到金鑰委付系統中。  
- 將加密金鑰檔案 (.pfx、.byok 或 .backup) 匯入至 Azure Key Vault。 
    

>[!NOTE] 
    >基於測試目的，您可以使用 Azure Key Vault 建立金鑰，但此金鑰無法委付，因為私密金鑰絕不可移至金鑰保存庫以外。  請務必備份並委付用來加密生產資料的金鑰，因為萬一遺失金鑰 (在金鑰保存庫中意外刪除、已到期等)，將導致資料永久遺失。
    >
    
- 使用沒有到期日的金鑰 – ，且對於已在使用中的金鑰一律不設定到期日：**金鑰一旦過期，加密的資料庫即會失去對其 TDE 保護裝置的存取權，且會在 24 小時內遭到捨棄**。
- 確認金鑰已啟用，且具有執行*取得*、*包裝金鑰*和*解除包裝金鑰*作業的權限。
- 在第一次使用 Azure Key Vault 中的金鑰之前，先建立 Azure Key Vault 金鑰備份。 深入了解 [Backup-AzureKeyVaultKey](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-5.1.1) 命令。
- 只要對金鑰進行任何變更 (例如，新增 ACL、新增標記、新增金鑰屬性)，即建立新的備份。
- 在輪替金鑰時**將舊版保留**在金鑰保存庫中，以便在還原較舊的資料庫備份時使用。 資料庫的 TDE 保護裝置有所變更時，資料庫的舊備份**不會更新**為使用最新的 TDE 保護裝置。  在還原時，每個備份都必須使用它在建立時所使用的 TDE 保護裝置。 您可以依照[使用 PowerShell 輪替透明資料加密保護裝置](transparent-data-encryption-byok-azure-sql-key-rotation.md)中的指示，來執行金鑰輪替。
- 變更回服務管理的金鑰之後，請將所有先前使用過的金鑰保存在 Azure Key Vault 中。  如此，即可確保能夠使用儲存在 Azure Key Vault 中的 TDE 保護裝置來還原資料庫備份。  在使用服務管理的金鑰建立所有已儲存的備份之前，必須妥善維護以 Azure Key Vault 建立的 TDE 保護裝置。  
- 使用 [Backup-AzureKeyVaultKey](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-5.1.1) 建立這些金鑰的可復原備份複本。
- 若要在安全性事件發生時移除可能遭破壞的金鑰，且不致產生資料遺失風險，請依照[移除可能遭破壞的金鑰](transparent-data-encryption-byok-azure-sql-remove-tde-protector.md)中的步驟操作。


## <a name="high-availability-geo-replication-and-backup--restore"></a>高可用性、異地複寫和備份/還原

### <a name="high-availability-and-disaster-recovery"></a>高可用性和災害復原

使用 Azure Key Vault 設定高可用性的方式，取決於您的資料庫和邏輯伺服器的組態，以下將分別就兩個不同的案例提供建議的組態。  第一個案例是未設定異地備援的獨立資料庫或邏輯伺服器。  第二個案例是設定了容錯移轉群組或異地備援的資料庫或邏輯伺服器；在此案例中，必須確定每個異地備援複本在容錯移轉群組內都有本機的 Azure Key Vault，以確保異地容錯移轉能夠運作。 在第一個案例中，如果您未設定異地備援的資料庫和邏輯伺服器需要高可用性，強烈建議您將伺服器設定為使用位於不同區域、但具有相同金鑰內容的兩個不同金鑰保存庫。  若要這麼做，請使用與邏輯伺服器共置於相同區域的主要 Key Vault 建立 TDE 保護裝置，然後將金鑰複製到不同 Azure 區域中的金鑰保存庫，如此，萬一主要金鑰保存庫在資料庫已啟動並執行時發生中斷的狀況，伺服器將可存取第二個金鑰保存庫。 您可以使用 Backup-AzureKeyVaultKey Cmdlet 從主要金鑰保存庫擷取加密格式的金鑰，然後使用 Restore-AzureKeyVaultKey Cmdlet，並在第二個區域中指定金鑰保存庫。


![單一伺服器 HA，無異地災害復原](./media/transparent-data-encryption-byok-azure-sql/SingleServer_HA_Config.PNG)

## <a name="how-to-configure-geo-dr-with-azure-key-vault"></a>如何使用 Azure Key Vault 設定異地災害復原

若要讓加密資料庫的 TDE 保護裝置維持高可用性，必須要根據現有或所需的 SQL Database 容錯移轉群組或作用中的異地複寫執行個體設定備援 Azure Key Vault。  每個異地複寫的伺服器都需要個別的金鑰保存庫，且保存庫必須與伺服器共置於相同的 Azure 區域中。 當主要資料庫因某個區域停止運作而無法存取，因而觸發了容錯移轉時，次要資料庫將可使用次要金鑰保存庫加以接管。 
 
異地複寫的 Azure SQL 資料庫必須具有下列 Azure Key Vault 組態：
- 在一個區域中設定具有金鑰保存庫的主要資料庫，並在另一個區域中設定具有金鑰保存庫的次要資料庫。 
- 至少要有一個次要資料庫，最多可支援四個。 
- 不支援次要的次要資料庫 (鏈結)。

下一節將詳細說明安裝和設定步驟。 

### <a name="azure-key-vault-configuration-steps"></a>Azure Key Vault 設定步驟

- 安裝 [PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.6.0) 
- 對金鑰保存庫使用 [PowerShell 啟用「虛刪除」屬性](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell)，在兩個不同的區域中建立兩個 Azure Key Vault (使用 AKV 入口網站時尚不適用此選項 – 但 SQL 則需要此選項)。
- 這兩個 Azure Key Vault 必須位於屬於相同 Azure 區域的兩個區域中，如此才能執行金鑰的備份與還原。  如果您的兩個金鑰保存庫必須位於不同地理區域，以符合 SQL 異地災害復原需求，請依照 [BYOK 程序](https://docs.microsoft.com/azure/key-vault/key-vault-hsm-protected-keys)操作，讓金鑰從內部部署 HSM 匯入。
- 在第一個金鑰保存庫中建立新的金鑰：  
  - RSA/RSA-HSA 2048 金鑰 
  - 沒有到期日 
  - 金鑰已啟用，且具有執行取得、包裝金鑰和解除包裝金鑰作業的權限 
- 備份主要金鑰，並將金鑰還原至第二個金鑰保存庫。  請參閱 [BackupAzureKeyVaultKey](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-5.1.1) 和 [Restore-AzureKeyVaultKey](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-5.5.0)。 

### <a name="azure-sql-database-configuration-steps"></a>Azure SQL Database 設定步驟

下列設定步驟會依據是從新的 SQL 部署開始作業，還是使用現有的 SQL 異地災害復原部署，而有所不同。  我們會先說明新部署的設定步驟，再說明如何將儲存在 Azure Key Vault 中的 TDE 保護裝置指派給已建立異地災害復原連結的現有部署。 

新部署的步驟：
- 在與先前建立的金鑰保存庫相同的兩個區域中，建立兩個邏輯 SQL 伺服器。 
- 選取邏輯伺服器 TDE 窗格，並為每個邏輯 SQL 伺服器：  
   - 選取相同區域中的 AKV 
   - 選取要作為 TDE 保護裝置的金鑰 – 每個伺服器都會使用 TDE 保護裝置的本機複本。 
   - 在入口網站中執行此作業，將會建立邏輯 SQL Server 的 [AppID](https://docs.microsoft.com/azure/active-directory/managed-service-identity/overview)，此 ID 可用來指派邏輯 SQL Server 存取金鑰保存庫的權限 – 請勿刪除此身分識別。 改為針對邏輯 SQL Server 移除 Azure Key Vault 中的權限 (用來指派邏輯 SQL Server 存取金鑰保存庫的權限)，可撤銷存取權。
- 建立主要資料庫。 
- 依照[作用中的異地複寫指引](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)完成案例，此步驟會建立次要資料庫。

![容錯移轉群組和異地災害復原](./media/transparent-data-encryption-byok-azure-sql/Geo_DR_Config.PNG)

>[!NOTE]
>請務必先確定相同的 TDE 保護裝置存在於這兩個金鑰保存庫中，再繼續建立資料庫之間的異地連結。
>

使用現有 SQL DB 和異地災害復原進行部署的步驟：

因為邏輯 SQL 伺服器已存在，且主要和次要資料庫皆已指派，因此設定 Azure Key Vault 的步驟必須依照下列順序執行： 
- 首先設定裝載次要資料庫的邏輯 SQL Server： 
   - 指派位於相同區域中的金鑰保存庫 
   - 指派 TDE 保護裝置 
- 現在，移至裝載主要資料庫的邏輯 SQL Server： 
   - 選取次要資料庫所使用的相同 TDE 保護裝置
   
![容錯移轉群組和異地災害復原](./media/transparent-data-encryption-byok-azure-sql/geo_DR_ex_config.PNG)

>[!NOTE]
>將金鑰保存庫指派給伺服器時，請務必先從次要伺服器開始。  在第二個步驟中，將金鑰保存庫指派給主要伺服器，並更新 TDE 保護裝置，異地災害復原連結將繼續運作，因為在此時，複寫的資料庫所使用 TDE 保護裝置已可供這兩部伺服器使用。
>

在針對 SQL Database 異地災害復原案例使用客戶在 Azure Key Vault 中管理的金鑰啟用 TDE 之前，請務必在相同區域中建立並維護兩個具有相同內容的 Azure Key Vault，以用於 SQL Database 異地複寫。  明確而言，「相同內容」表示兩個金鑰保存庫必須包含相同 TDE 保護裝置的複本，讓兩個伺服器都能存取所有資料庫使用的 TDE 保護裝置。  接下來，必須將兩個金鑰保存庫保持同步，這表示兩者在金鑰輪替後必須包含 TDE 保護裝置的相同複本，保有用於記錄檔或備份的舊版金鑰、TDE 保護裝置必須保有相同的金鑰屬性，且金鑰保存庫必須為 SQL 保有相同的存取權限。  
 
請依照[作用中的異地複寫概觀](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)中的步驟測試並觸發容錯移轉，此作業應定期執行，以確認 SQL 對這兩個金鑰保存庫的存取權限維持不變。 


### <a name="backup-and-restore"></a>備份與還原

使用 Key Vault 中的金鑰以 TDE 加密資料庫後，所產生的任何備份也會使用相同的 TDE 保護裝置進行加密。

若要從 Key Vault 還原以 TDE 保護裝置加密的備份，請確定金鑰內容仍在原始的保存庫中，且金鑰名稱保持不變。 資料庫的 TDE 保護裝置有所變更時，資料庫的舊備份**不會更新**為使用最新的 TDE 保護裝置。 因此，建議您將所有舊版的 TDE 保護裝置保存在 Key Vault 中，以便在還原資料庫備份時使用。 

如果還原備份時可能需要的金鑰已不在原始金鑰保存庫中，則會傳回下列錯誤訊息：「目標伺服器 <Servername> 無法存取在 <Timestamp #1> 與 <Timestamp #2> 之間建立的所有 AKV URI。 請先還原所有的 AKV URI，再重試作業。」

若要緩解此狀況，請執行 [Get-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/get-azurermsqlserverkeyvaultkey) Cmdlet，以從 Key Vault 傳回已新增至伺服器的金鑰清單 (除非使用者已刪除金鑰)。 為確保所有備份均可還原，請確定備份的目標伺服器有權存取前述所有的金鑰。

   ```powershell
   Get-AzureRmSqlServerKeyVaultKey `
   -ServerName <LogicalServerName> `
   -ResourceGroup <SQLDatabaseResourceGroupName>
   ```
若要深入了解 SQL Database 的備份復原，請參閱[復原 Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups)。 若要深入了解 SQL 資料倉儲的備份復原，請參閱[復原 Azure SQL 資料倉儲](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-restore-database-overview)。


已備份記錄檔的其他注意事項：備份的記錄檔仍會以原始 TDE 加密程式加密，即使 TDE 保護裝置已輪替，且資料庫目前使用新的 TDE 保護裝置，仍是如此。  在還原時，必須要有這兩個金鑰才能還原資料庫。  如果記錄檔使用儲存在 Azure Key Vault 中的 TDE 保護裝置，則在還原時將會需要此金鑰，即使資料庫已變更為使用服務管理的 TDE，仍是如此。


