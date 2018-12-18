---
title: PowerShell 和 CLI - 啟用 SQL TDE - 您的金鑰 - Azure SQL Database| Microsoft Docs
description: 了解如何設定 Azure SQL Database 和資料倉儲，以透過 PowerShell 或 CLI 開始使用透明資料加密 (TDE) 進行待用資料加密。
services: sql-database
keywords: ''
documentationcenter: ''
author: aliceku
manager: craigg
editor: ''
ms.prod: ''
ms.reviewer: vanto
ms.suite: sql
ms.prod_service: sql-database, sql-data-warehouse
ms.service: sql-database
ms.tgt_pltfrm: ''
ms.devlang: azurecli, powershell
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: aliceku
monikerRange: = azuresqldb-current || = azure-sqldw-latest || = sqlallproducts-allversions
ms.openlocfilehash: 11e190e1a4d0309bdbdcb7a578fccaf84fabb8e3
ms.sourcegitcommit: f983187566d165bc8540fdec5650edcc51a6350a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "45543772"
---
# <a name="powershell-and-cli-enable-transparent-data-encryption-using-your-own-key-from-azure-key-vault"></a>PowerShell 和 CLI：從 Azure Key Vault 使用您自己的金鑰啟用透明資料加密

本文將逐步解說如何將 Azure Key Vault 中的金鑰用於 SQL Database 或資料倉儲上的透明資料加密 (TDE)。 若要深入了解具有「攜帶您自己的金鑰」(BYOK) 支援的 TDE，請瀏覽[將 TDE 攜帶您自己的金鑰用於 Azure SQL](transparent-data-encryption-byok-azure-sql.md)。 

## <a name="prerequisites-for-powershell"></a>PowerShell 的必要條件

- 您必須具有 Azure 訂用帳戶，並且是該訂用帳戶的系統管理員。
- [建議執行的選擇性作業] 備妥硬體安全性模組 (HSM) 或本機金鑰存放區，用來建立 TDE 保護裝置金鑰內容的本機複本。
- 您必須安裝並執行 Azure PowerShell 4.2.0 版或更新版本。 
- 建立要用於 TDE 的 Azure Key Vault 和金鑰。
   - [從 Key Vault 使用 PowerShell 的指示](https://docs.microsoft.com/azure/key-vault/key-vault-get-started)
   - [使用硬體安全性模組 (HSM) 與 Key Vault 的指示](https://docs.microsoft.com/azure/key-vault/key-vault-get-started#a-idhsmaif-you-want-to-use-a-hardware-security-module-hsm)
 - 金鑰保存庫必須具有下列屬性才能用於 TDE：
   - [虛刪除](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete)
   - [如何使用 Key Vault 虛刪除與 PowerShell](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell) 
- 金鑰必須具有下列屬性才能用於 TDE：
   - 沒有到期日
   - 未停用
   - 能夠執行*取得*、*包裝金鑰*、*解除包裝金鑰*作業

## <a name="step-1-assign-an-azure-ad-identity-to-your-server"></a>步驟 1. 將 Azure AD 身分識別指派給您的伺服器 

如果您有現有的伺服器，請使用下列程式碼將 Azure AD 身分識別新增至您的伺服器：

   ```powershell
   $server = Set-AzureRmSqlServer `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -AssignIdentity
   ```

如果您要建立伺服器，請使用 [New-AzureRmSqlServer](/powershell/module/azurerm.sql/new-azurermsqlserver) Cmdlet 和 -Identity 標記，在伺服器建立期間新增 Azure AD 身分識別：

   ```powershell
   $server = New-AzureRmSqlServer `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -Location <RegionName> `
   -ServerName <LogicalServerName> `
   -ServerVersion "12.0" `
   -SqlAdministratorCredentials <PSCredential> `
   -AssignIdentity 
   ```

## <a name="step-2-grant-key-vault-permissions-to-your-server"></a>步驟 2. 為您的伺服器授與 Key Vault 權限

請先使用 [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) Cmdlet 為您的伺服器授與金鑰保存庫的存取權，再將其中的金鑰用於 TDE。

   ```powershell
   Set-AzureRmKeyVaultAccessPolicy  `
   -VaultName <KeyVaultName> `
   -ObjectId $server.Identity.PrincipalId `
   -PermissionsToKeys get, wrapKey, unwrapKey
   ```

## <a name="step-3-add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>步驟 3. 將 Key Vault 金鑰新增至伺服器，並設定 TDE 保護裝置

- 使用 [Add-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/add-azurermsqlserverkeyvaultkey) Cmdlet 將 Key Vault 中的金鑰新增至伺服器。
- 使用 [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) Cmdlet，將金鑰設定為所有伺服器資源的 TDE 保護裝置。
- 使用 [Get-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/get-azurermsqlservertransparentdataencryptionprotector) Cmdlet，確認 TDE 保護裝置已依預期設定。

> [!Note]
> 金鑰保存庫名稱和金鑰名稱的合併長度不可超過 94 個字元。
> 

>[!Tip]
>Key Vault 中的範例 KeyId： https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h
>

   ```powershell
   <# Add the key from Key Vault to the server #>
   Add-AzureRmSqlServerKeyVaultKey `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -KeyId <KeyVaultKeyId>

   <# Set the key as the TDE protector for all resources under the server #>
   Set-AzureRmSqlServerTransparentDataEncryptionProtector `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -Type AzureKeyVault `
   -KeyId <KeyVaultKeyId> 

   <# To confirm that the TDE protector was configured as intended: #>
   Get-AzureRmSqlServerTransparentDataEncryptionProtector `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> 
   ```

## <a name="step-4-turn-on-tde"></a>步驟 4. 開啟 TDE 

使用 [Set-AzureRMSqlDatabaseTransparentDataEncryption](/powershell/module/azurerm.sql/set-azurermsqldatabasetransparentdataencryption) Cmdlet 開啟 TDE。

   ```powershell
   Set-AzureRMSqlDatabaseTransparentDataEncryption `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -DatabaseName <DatabaseName> `
   -State "Enabled"
   ```

現在，資料庫或資料倉儲已使用 Key Vault 中的加密金鑰啟用 TDE。

## <a name="step-5-check-the-encryption-state-and-encryption-activity"></a>步驟 5。 檢查加密狀態和加密活動

使用 [Get-AzureRMSqlDatabaseTransparentDataEncryption](/powershell/module/azurerm.sql/get-azurermsqldatabasetransparentdataencryption) 取得加密狀態，並使用 [Get-AzureRMSqlDatabaseTransparentDataEncryptionActivity](/powershell/module/azurerm.sql/get-azurermsqldatabasetransparentdataencryptionactivity) 檢查資料庫或資料倉儲的加密進度。

   ```powershell
   # Get the encryption state
   Get-AzureRMSqlDatabaseTransparentDataEncryption `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -DatabaseName <DatabaseName> `

   <# Check the encryption progress for a database or data warehouse #>
   Get-AzureRMSqlDatabaseTransparentDataEncryptionActivity `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -DatabaseName <DatabaseName>  
   ```

## <a name="other-useful-powershell-cmdlets"></a>其他實用的 PowerShell Cmdlet

- 使用 [Set-AzureRMSqlDatabaseTransparentDataEncryption](/powershell/module/azurerm.sql/set-azurermsqldatabasetransparentdataencryption) Cmdlet 關閉 TDE。

   ```powershell
   Set-AzureRMSqlDatabaseTransparentDataEncryption `
   -ServerName <LogicalServerName> `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -DatabaseName <DatabaseName> `
   -State "Disabled”
   ```
 
- 使用 [Get-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/get-azurermsqlserverkeyvaultkey) Cmdlet 傳回已新增至伺服器的 Key Vault 金鑰清單。

   ```powershell
   <# KeyId is an optional parameter, to return a specific key version #>
   Get-AzureRmSqlServerKeyVaultKey `
   -ServerName <LogicalServerName> `
   -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```
 
- 使用 [Remove-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/remove-azurermsqlserverkeyvaultkey) 從伺服器中移除 Key Vault 金鑰。

   ```powershell
   <# The key set as the TDE Protector cannot be removed. #>
   Remove-AzureRmSqlServerKeyVaultKey `
   -KeyId <KeyVaultKeyId> `
   -ServerName <LogicalServerName> `
   -ResourceGroupName <SQLDatabaseResourceGroupName>   
   ```
 
## <a name="troubleshooting"></a>疑難排解

如果發生問題，請進行下列檢查：
- 如果找不到金鑰保存庫，請使用 [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription) Cmdlet 確認您位於正確的訂用帳戶中。

   ```powershell
   Get-AzureRmSubscription `
   -SubscriptionId <SubscriptionId>
   ```

- 如果新的金鑰無法新增至伺服器，或新的金鑰無法更新為 TDE 保護裝置，請進行下列檢查：
   - 金鑰不應有到期日
   - 金鑰必須已啟用*取得*、*包裝金鑰*和*解除包裝金鑰*作業。

## <a name="next-steps"></a>後續步驟

- 了解如何輪替伺服器的 TDE 保護裝置以符合安全需求：[使用 PowerShell 輪替透明資料加密保護裝置](transparent-data-encryption-byok-azure-sql-key-rotation.md)。
- 如有安全性風險，請了解如何移除可能遭破壞的 TDE 保護裝置：[移除可能遭破壞的金鑰](transparent-data-encryption-byok-azure-sql-remove-tde-protector.md)。 

## <a name="prerequisites-for-cli"></a>CLI 的必要條件

- 您必須具有 Azure 訂用帳戶，並且是該訂用帳戶的系統管理員。
- [建議執行的選擇性作業] 備妥硬體安全性模組 (HSM) 或本機金鑰存放區，用來建立 TDE 保護裝置金鑰內容的本機複本。
- 命令列介面 2.0 版或更新版本。 若要安裝最新版本，並連線至 Azure 訂用帳戶，請參閱[安裝與設定 Azure 跨平台命令列介面 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。 
- 建立要用於 TDE 的 Azure Key Vault 和金鑰。
   - [使用 CLI 2.0 管理 Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-manage-with-cli2)
   - [使用硬體安全性模組 (HSM) 與 Key Vault 的指示](https://docs.microsoft.com/azure/key-vault/key-vault-get-started#a-idhsmaif-you-want-to-use-a-hardware-security-module-hsm)
 - 金鑰保存庫必須具有下列屬性才能用於 TDE：
   - [虛刪除](https://docs.microsoft.com/azure/key-vault/key-vault-ovw-soft-delete)
   - [如何使用 Key Vault 虛刪除與 CLI](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-cli) 
- 金鑰必須具有下列屬性才能用於 TDE：
   - 沒有到期日
   - 未停用
   - 能夠執行*取得*、*包裝金鑰*、*解除包裝金鑰*作業
   
## <a name="step-1-create-a-server-and-assign-an-azure-ad-identity-to-your-server"></a>步驟 1. 建立伺服器，並將 Azure AD 身分識別指派給您的伺服器
      cli
      # create server (with identity) and database
      az sql server create -n "ServerName" -g "ResourceGroupName" -l "westus" -u "cloudsa" -p "YourFavoritePassWord99@34" -I 
      az sql db create -n "DatabaseName" -g "ResourceGroupName" -s "ServerName" 
      

 
## <a name="step-2-grant-key-vault-permissions-to-your-server"></a>步驟 2. 為您的伺服器授與 Key Vault 權限
      cli
      # create key vault, key and grant permission
      az keyvault create -n "VaultName" -g "ResourceGroupName" 
      az keyvault key create -n myKey -p software --vault-name "VaultName" 
      az keyvault set-policy -n "VaultName" --object-id "ServerIdentityObjectId" -g "ResourceGroupName" --key-permissions wrapKey unwrapKey get list 
      

 
## <a name="step-3-add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>步驟 3. 將 Key Vault 金鑰新增至伺服器，並設定 TDE 保護裝置
  
     cli
     # add server key and update encryption protector
      az sql server key create -g "ResourceGroupName" -s "ServerName" -t "AzureKeyVault" -u "FullVersionedKeyUri 
      az sql server tde-key update -g "ResourceGroupName" -s "ServerName" -t AzureKeyVault -u "FullVersionedKeyUri" 
      
  
  > [!Note]
> 金鑰保存庫名稱和金鑰名稱的合併長度不可超過 94 個字元。
> 

>[!Tip]
>Key Vault 中的範例 KeyId： https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h
>
  
## <a name="step-4-turn-on-tde"></a>步驟 4. 開啟 TDE 
      cli
      # enable encryption
      az sql db tde create -n "DatabaseName" -g "ResourceGroupName" -s "ServerName" --status Enabled 
      

現在，資料庫或資料倉儲已使用 Key Vault 中的加密金鑰啟用 TDE。

## <a name="step-5-check-the-encryption-state-and-encryption-activity"></a>步驟 5。 檢查加密狀態和加密活動

     cli
      # get encryption scan progress
      az sql db tde show-activity -n "DatabaseName" -g "ResourceGroupName" -s "ServerName" 

      # get whether encryption is on or off
      az sql db tde show-configuration -n "DatabaseName" -g "ResourceGroupName" -s "ServerName" 

## <a name="sql-cli-references"></a>SQL CLI 參考

https://docs.microsoft.com/cli/azure/sql?view=azure-cli-latest 

https://docs.microsoft.com/cli/azure/sql/server/key?view=azure-cli-latest 

https://docs.microsoft.com/cli/azure/sql/server/tde-key?view=azure-cli-latest 

https://docs.microsoft.com/cli/azure/sql/db/tde?view=azure-cli-latest 

