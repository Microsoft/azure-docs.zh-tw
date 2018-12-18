---
title: PowerShell - 移除 TDE 保護裝置 - Azure SQL Database| Microsoft Docs
description: 本操作指南說明 Azure SQL Database 或資料倉儲若是使用具有「攜帶您自己的金鑰」(BYOK) 支援的 TDE，在其 TDE 保護裝置可能遭破壞時，應如何因應。
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
ms.date: 08/07/2017
ms.openlocfilehash: 6675a68222e09be9a092ad21ee318a53a0a39ca5
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49311298"
---
# <a name="remove-a-transparent-data-encryption-tde-protector-using-powershell"></a>使用 PowerShell 移除透明資料加密 (TDE) 保護裝置
## <a name="prerequisites"></a>必要條件
- 您必須具有 Azure 訂用帳戶，並且是該訂用帳戶的系統管理員
- 您必須安裝並執行 Azure PowerShell 4.2.0 版或更新版本。 
- 本操作指南假設您已使用 Azure Key Vault 中的金鑰作為 Azure SQL Database 或資料倉儲的 TDE 保護裝置。 若要深入了解，請參閱[具有 BYOK 支援的透明資料加密](transparent-data-encryption-byok-azure-sql.md)。

## <a name="overview"></a>概觀
本操作指南說明 Azure SQL Database 或資料倉儲若是使用具有「攜帶您自己的金鑰」(BYOK) 支援的 TDE，在其 TDE 保護裝置可能遭破壞時，應如何因應。 若要深入了解 TDE 的 BYOK 支援，請參閱[概觀頁面](transparent-data-encryption-byok-azure-sql.md)。 

只有在極端情況下或測試環境中，才需要執行下列程序。 請仔細檢閱本操作指南，因為從 Azure Key Vault 中刪除正在使用的 TDE 保護裝置，可能會導致**資料遺失**。 

如果金鑰疑似遭到破壞，而使服務或使用者可在未經授權的情況下存取金鑰，則最好將金鑰刪除。

請切記，在 Key Vault 中刪除 TDE 保護裝置後，**系統會封鎖該伺服器下所有與已加密資料庫的連線；這些資料庫會離線，並在 24 小時內遭到捨棄**。 以遭破壞的金鑰加密的舊備份，將不再可供存取。

本操作指南會根據事件回應之後的所需結果討論兩種方法：
- 將 Azure SQL 資料庫/資料倉儲保持為**可供存取**
- 使 Azure SQL 資料庫/資料倉儲變成**無法存取**

## <a name="to-keep-the-encrypted-resources-accessible"></a>將加密資源保持為可供存取
1. 建立 [Key Vault 中的新金鑰](https://docs.microsoft.com/powershell/module/azurerm.keyvault/add-azurekeyvaultkey?view=azurermps-4.1.0)。 請務必將此新金鑰建立在與可能遭破壞的 TDE 保護裝置不同的金鑰保存庫中，因為存取控制佈建於保存庫層級上。 
2. 使用 [Add-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/add-azurermsqlserverkeyvaultkey) 和 [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) Cmdlet 將新的金鑰新增至伺服器，並將其更新為伺服器的新 TDE 保護裝置。

   ```powershell
   # Add the key from Key Vault to the server  
   Add-AzureRmSqlServerKeyVaultKey `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -KeyId <KeyVaultKeyId>
   
   # Set the key as the TDE protector for all resources under the server
   Set-AzureRmSqlServerTransparentDataEncryptionProtector `
   -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> `
   -Type AzureKeyVault -KeyId <KeyVaultKeyId> 
   ```

3. 使用 [Get-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/get-azurermsqlservertransparentdataencryptionprotector) Cmdlet，確定伺服器和任何複本均已更新至新的 TDE 保護裝置。 

   >[!NOTE]
   > 新的 TDE 保護裝置可能需要幾分鐘的時間才能傳播至伺服器下的所有資料庫和次要資料庫。
   >

   ```powershell
   Get-AzureRmSqlServerTransparentDataEncryptionProtector `
   -ServerName <LogicalServerName> `
   -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

4. 在 Key Vault 中建立[新金鑰的備份](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)。

   ```powershell
   <# -OutputFile parameter is optional; 
   if removed, a file name is automatically generated. #>
   Backup-AzureKeyVaultKey `
   -VaultName <KeyVaultName> `
   -Name <KeyVaultKeyName> `
   -OutputFile <DesiredBackupFilePath>
   ```
 
5. 使用 [Remove-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/remove-azurekeyvaultkey) Cmdlet 從 Key Vault 中刪除遭破壞的金鑰。 

   ```powershell
   Remove-AzureKeyVaultKey `
   -VaultName <KeyVaultName> `
   -Name <KeyVaultKeyName>
   ```
 
6. 日後可使用 [Restore-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey) Cmdlet 將金鑰還原至 Key Vault：
   ```powershell
   Restore-AzureKeyVaultKey `
   -VaultName <KeyVaultName> `
   -InputFile <BackupFilePath>
   ```
 
## <a name="to-make-the-encrypted-resources-inaccessible"></a>使加密資源變得無法存取
1. 捨棄以可能遭破壞的金鑰加密的資料庫。
資料庫和記錄檔會自動備份，因此資料庫的時間點還原可以在任何時間點執行 (只要您提供金鑰)。 資料庫必須在作用中的 TDE 保護裝置刪除之前捨棄，否則最多可能會遺失最新交易 10 分鐘內的資料。 
2. 在 Key Vault 中備份 TDE 保護裝置的金鑰內容。
3. 從 Key Vault 中移除可能遭破壞的金鑰

## <a name="next-steps"></a>後續步驟

- 了解如何輪替伺服器的 TDE 保護裝置以符合安全需求：[使用 PowerShell 輪替透明資料加密保護裝置](transparent-data-encryption-byok-azure-sql-key-rotation.md)

- 開始將「攜帶您自己的金鑰支援」用於 TDE：[透過 PowerShell 從 Key Vault 使用您自己的金鑰開啟 TDE](transparent-data-encryption-byok-azure-sql-configure.md)
