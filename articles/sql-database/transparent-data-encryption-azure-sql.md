---
title: Azure SQL Database 和資料倉儲的透明資料加密 | Microsoft Docs
description: 概述 SQL Database 和資料倉儲的透明資料加密。 本文件說明其優點和設定選項，其中包括服務管理的透明資料加密和「攜帶您自己的金鑰」。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: becczhang
ms.author: aliceku
ms.reviewer: vanto
manager: craigg
ms.date: 07/09/2018
ms.openlocfilehash: 50b433c65dec1f667f32aaf60148a6e393c67320
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165921"
---
# <a name="transparent-data-encryption-for-sql-database-and-data-warehouse"></a>SQL Database 和資料倉儲的透明資料加密

透明資料加密 (TDE) 可協助 Azure SQL Database 和 Azure 資料倉儲抵禦惡意活動的威脅。 它會對資料庫、相關聯的備份和待用的交易記錄檔執行即時加密和解密，而不需變更應用程式。 根據預設，會為所有新部署的 Azure SQL 資料庫啟用 TDE。 TDE 無法用來加密 SQL Database 中的邏輯**主要**資料庫。  **主要**資料庫包含在使用者資料庫上執行 TDE 作業所需的物件。

對於較舊的資料庫或 Azure SQL 資料倉儲，TDE 必須以手動方式啟用。  

透明資料加密會使用稱為資料庫加密金鑰的對稱金鑰，來將整個資料庫的儲存體加密。 此資料庫加密金鑰受到透明資料加密保護裝置的保護。 此保護裝置可以是服務管理的憑證 (服務管理的透明資料加密) 或儲存在 Azure Key Vault 中的非對稱金鑰 (攜帶您自己的金鑰)。 您會在伺服器層級上設定透明資料加密保護裝置。 

在資料庫啟動時，加密的資料庫加密金鑰會進行解密，然後在 SQL Server Database Engine 程序中用於資料庫檔案的解密和重新加密。 透明資料加密會在頁面層級執行資料的即時 I/O 加密和解密。 每個頁面在讀取到記憶體時都會進行解密，然後再重新加密並寫入至磁碟。 如需透明資料加密的一般描述，請參閱[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption)。

在 Azure 虛擬機器上執行的 SQL Server 也可以使用 Key Vault 中的非對稱金鑰。 其設定步驟與在 SQL Database 中使用非對稱金鑰不同。 如需詳細資訊，請參閱[使用 Azure Key Vault 進行可延伸金鑰管理 (SQL Server)](https://docs.microsoft.com/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server)。

## <a name="service-managed-transparent-data-encryption"></a>服務管理的透明資料加密

在 Azure 中，透明資料加密的預設設定是以內建伺服器憑證保護資料庫加密金鑰。 每個伺服器的內建伺服器憑證都是唯一的。 如果資料庫具有異地複寫關聯性，則主要和異地次要資料庫都會受到主要資料庫的父伺服器金鑰保護。 如果兩個資料庫連線到相同的伺服器，則會共用同一個內建憑證。 Microsoft 至少每 90 天會自動替換這些憑證。

Microsoft 也會視異地複寫和還原的需要順暢地移動和管理金鑰。 

> [!IMPORTANT]
> 依預設會使用服務管理的透明資料加密為所有新建立的 SQL 資料庫加密。 2017 年 5 月以前的現有資料庫以及透過還原、異地複寫和資料庫複本建立的資料庫，依預設不會進行加密。
>

## <a name="bring-your-own-key"></a>攜帶您自己的金鑰

透過「攜帶您自己的金鑰」支援，您將可控制透明資料加密金鑰，以及可存取金鑰的人員與時間。 Key Vault 是 Azure 雲端式外部金鑰管理系統，這是透明資料加密整合至「攜帶您自己的金鑰」支援的第一項金鑰管理服務。 透過「攜帶您自己的金鑰」支援，資料庫加密金鑰將可由儲存在 Key Vault 中的非對稱金鑰提供保護。 非對稱金鑰一律會存放在 Key Vault 中。 當伺服器具有金鑰保存庫的權限後，伺服器即會透過 Key Vault 對該保存庫傳送基本金鑰作業要求。 您可以在伺服器層級設定非對稱金鑰，如此，該伺服器下的所有資料庫會都繼承此設定。

透過「攜帶您自己的金鑰」支援，您現在已可控制金鑰管理工作 (例如金鑰輪替) 和金鑰保存庫權限。 您也可以刪除金鑰，以及啟用所有加密金鑰的稽核/報告功能。 Key Vault 提供集中管理金鑰的機制，並使用嚴密監控的硬體安全性模組。 Key Vault 有助於個別管理金鑰和資料，以利符合法規合規性。 若要深入了解 Key Vault，請參閱 [Key Vault 文件頁面](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)。

若要深入了解如何將具有「攜帶您自己的金鑰」支援的透明資料加密用於 Azure SQL Database 和資料倉儲，請參閱[具有攜帶您自己的金鑰支援的透明資料加密](transparent-data-encryption-byok-azure-sql.md)。

若要開始使用具有「攜帶您自己的金鑰」支援的透明資料加密，請參閱[透過 PowerShell 從 Key Vault 使用您自己的金鑰開啟透明資料加密](transparent-data-encryption-byok-azure-sql-configure.md)操作指南。

## <a name="move-a-transparent-data-encryption-protected-database"></a>移動以透明資料加密保護的資料庫

在 Azure 中執行作業時，您無須解密資料庫。 在目標上會自動繼承來源資料庫或主要資料庫的透明資料加密設定。 其中包含的作業涉及：
- 異地還原。
- 自助式時間點還原。
- 還原已刪除的資料庫。
- 主動式異地複寫。
- 建立資料庫複本。

當您匯出以透明資料加密保護的資料庫時，匯出的資料庫內容並不會加密。 這些匯出的內容會儲存在未加密的 BACPAC 檔案中。 請務必適當保護 BACPAC 檔案，並在新的資料庫匯入完成後啟用透明資料加密。

例如，如果從內部部署 SQL Server 執行個體匯出 BACPAC 檔案，新資料庫的匯入內容並不會自動加密。 同樣地，如果將 BACPAC 檔案匯出至內部部署 SQL Server 執行個體，新的資料庫也不會自動加密。

但當您匯出至 SQL 資料庫或從中匯出時，則屬例外。 新的資料庫中會啟用透明資料加密，但 BACPAC 檔案本身仍不會加密。

## <a name="manage-transparent-data-encryption-in-the-azure-portal"></a>在 Azure 入口網站中管理透明資料加密

若要透過 Azure 入口網站設定透明資料加密，您必須以 Azure 擁有者、參與者或 SQL 安全性管理員的身分連線。 

您可以在資料庫層級上設定透明資料加密。 若要在資料庫上啟用透明資料加密，請前往 [Azure 入口網站](https://portal.azure.com)，並使用您的 Azure 系統管理員或參與者帳戶登入。 在您的使用者資料庫下找出透明資料加密設定。 依預設會使用服務管理的透明資料加密。 系統會自動為包含資料庫的伺服器產生透明資料加密憑證。 

![服務管理的透明資料加密](./media/transparent-data-encryption-azure-sql/service-managed-tde.png)  

您可以在伺服器層級上設定透明資料加密主要金鑰，也就是透明資料加密保護裝置。 若要使用具有「攜帶您自己的金鑰」支援的透明資料加密，並以 Key Vault 中的金鑰保護您的資料庫，請查看伺服器下方的透明資料加密設定。 

![具有「攜帶您自己的金鑰」支援的透明資料加密](./media/transparent-data-encryption-azure-sql/tde-byok-support.png) 

## <a name="manage-transparent-data-encryption-by-using-powershell"></a>使用 PowerShell 管理透明資料加密

若要透過 PowerShell 設定透明資料加密，您必須以 Azure 擁有者、參與者或 SQL 安全性管理員的身分連線。 

| Cmdlet | 說明 |
| --- | --- |
| [Set-AzureRmSqlDatabaseTransparentDataEncryption](/powershell/module/azurerm.sql/set-azurermsqldatabasetransparentdataencryption) |啟用或停用資料庫的透明資料加密|
| [Get-Azure-Rm-Sql-Database-Transparent-Data-Encryption](/powershell/module/azurerm.sql/get-azurermsqldatabasetransparentdataencryption) |取得資料庫的透明資料加密狀態 |
| [Get-Azure-Rm-Sql-Database-Transparent-Data-Encryption-Activity](/powershell/module/azurerm.sql/get-azurermsqldatabasetransparentdataencryptionactivity) |查看資料庫的加密進度 |
| [Add-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/add-azurermsqlserverkeyvaultkey) |將 Key Vault 金鑰新增至 SQL Server 執行個體 |
| [Get-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/get-azurermsqlserverkeyvaultkey) |取得 SQL Server 執行個體的 Key Vault 金鑰  |
| [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) |為 SQL Server 執行個體設定透明資料加密保護裝置 |
| [Get-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/get-azurermsqlservertransparentdataencryptionprotector) |取得透明資料加密保護裝置 |
| [Remove-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/remove-azurermsqlserverkeyvaultkey) |從 SQL Server 執行個體中移除 Key Vault 金鑰 |
|  | |

## <a name="manage-transparent-data-encryption-by-using-transact-sql"></a>使用 Transact-SQL 管理透明資料加密

使用在主要資料庫中作為系統管理員或 **dbmanager** 角色成員的登入，連線至資料庫。

| 命令 | 說明 |
| --- | --- |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) | SET ENCRYPTION ON/OFF 可加密或解密資料庫 |
| [sys.dm_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql) |傳回資料庫及其相關資料庫加密金鑰的加密狀態相關資訊 |
| [sys.dm_pdw_nodes_database_encryption_keys](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql) |傳回每個資料倉儲節點及其相關資料庫加密金鑰的加密狀態相關資訊 | 
|  | |

您無法使用 Transact-SQL 將透明資料加密保護裝置切換為 Key Vault 中的金鑰。 請使用 PowerShell 或 Azure 入口網站。

## <a name="manage-transparent-data-encryption-by-using-the-rest-api"></a>使用 REST API 管理透明資料加密
 
若要透過 REST API 設定透明資料加密，您必須以 Azure 擁有者、參與者或 SQL 安全性管理員的身分連線。 

| 命令 | 說明 |
| --- | --- |
|[建立或更新伺服器](/rest/api/sql/servers/createorupdate)|將 Azure Active Directory 身分識別新增至 SQL Server 執行個體 (用以授與 Key Vault 的存取權)|
|[建立或更新伺服器金鑰](/rest/api/sql/serverkeys/createorupdate)|將 Key Vault 金鑰新增至 SQL Server 執行個體|
|[刪除伺服器金鑰](/rest/api/sql/serverkeys/delete)|從 SQL Server 執行個體中移除 Key Vault 金鑰|
|[取得伺服器金鑰](/rest/api/sql/serverkeys/get)|從 SQL Server 執行個體取得特定的 Key Vault 金鑰|
|[依伺服器列出伺服器金鑰](/rest/api/sql/serverkeys/listbyserver)|取得 SQL Server 執行個體的 Key Vault 金鑰 |
|[建立或更新加密保護裝置](/rest/api/sql/encryptionprotectors/createorupdate)|為 SQL Server 執行個體設定透明資料加密保護裝置|
|[取得加密保護裝置](/rest/api/sql/encryptionprotectors/get)|取得 SQL Server 執行個體的透明資料加密保護裝置|
|[依伺服器列出加密保護裝置](/rest/api/sql/encryptionprotectors/listbyserver)|取得 SQL Server 執行個體的透明資料加密保護裝置 |
|[建立或更新透明資料加密組態](/rest/api/sql/transparentdataencryptions/createorupdate)|啟用或停用資料庫的透明資料加密|
|[取得透明資料加密組態](/rest/api/sql/transparentdataencryptions/get)|取得資料庫的透明資料加密組態|
|[列出透明資料加密組態結果](/rest/api/sql/transparentdataencryptionactivities/ListByConfiguration)|取得資料庫的加密結果|

## <a name="next-steps"></a>後續步驟

- 如需透明資料加密的一般描述，請參閱[透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption)。
- 若要深入了解如何將具有「攜帶您自己的金鑰」支援的透明資料加密用於 Azure SQL Database 和資料倉儲，請參閱[具有攜帶您自己的金鑰支援的透明資料加密](transparent-data-encryption-byok-azure-sql.md)。
- 若要開始使用具有「攜帶您自己的金鑰」支援的透明資料加密，請參閱[透過 PowerShell 從 Key Vault 使用您自己的金鑰開啟透明資料加密](transparent-data-encryption-byok-azure-sql-configure.md)操作指南。
- 如需 Key Vault 的詳細資訊，請參閱 [Key Vault 文件頁面](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)。
