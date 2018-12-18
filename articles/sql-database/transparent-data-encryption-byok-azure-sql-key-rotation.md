---
title: PowerShell - 輪替 TDE 保護裝置 - Azure SQL Database| Microsoft Docs
description: 了解如何輪替 Azure SQL Server 的透明資料加密 (TDE) 保護裝置。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: aliceku
ms.author: aliceku
ms.reviewer: vanto
manager: jhubbard
ms.date: 08/07/2017
ms.openlocfilehash: fc8660b6fa6d9778a4eac9ce939181aa24d5097d
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49309633"
---
# <a name="rotate-the-transparent-data-encryption-tde-protector-using-powershell"></a>使用 PowerShell 輪替透明資料加密 (TDE) 保護裝置 

本文說明如何使用 TDE 保護裝置從 Azure Key Vault 進行 Azure SQL Server 的金鑰輪替。 輪替 Azure SQL Server 的 TDE 保護裝置，其實就是切換為新的非對稱金鑰來保護伺服器上的資料庫。 金鑰輪替是一項線上作業，應可在幾秒內完成，因為此作業只會解密和重新加密資料庫的資料加密金鑰，而不是整個資料庫。

本指南將討論用來在伺服器上輪替 TDE 保護裝置的兩個選項。

> [!NOTE]
> 進行金鑰輪替之前，必須先恢復暫停的 SQL 資料倉儲。
>

> [!IMPORTANT]
> 在完成變換後，**請勿刪除**舊版的金鑰。  在金鑰變換後，部分資料仍會以先前的金鑰加密，例如較舊的資料庫備份。 
>

## <a name="prerequisites"></a>必要條件

- 本操作指南假設您已使用 Azure Key Vault 中的金鑰作為 Azure SQL Database 或資料倉儲的 TDE 保護裝置。 請參閱[具有 BYOK 支援的透明資料加密](transparent-data-encryption-byok-azure-sql.md)。
- 您必須安裝並執行 Azure PowerShell 3.7.0 版或更新版本。 
- [建議執行的選擇性作業] 先在硬體安全性模組 (HSM) 或本機金鑰存放區中建立 TDE 保護裝置的金鑰內容，然後將金鑰內容匯入至 Azure Key Vault。 請依照[使用硬體安全性模組 (HSM) 與 Key Vault 的指示](https://docs.microsoft.com/azure/key-vault/key-vault-get-started)操作，以深入了解。

## <a name="option-1-auto-rotation"></a>選項 1：自動輪替

以相同的金鑰名稱和金鑰保存庫，在 Key Vault 中產生現有 TDE 保護裝置金鑰的新版本。 Azure SQL 服務會在 24 小時內開始使用這個新版本。 

若要使用 [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) Cmdlet 建立新版的 TDE 保護裝置：

   ```powershell
   Add-AzureKeyVaultKey `
   -VaultName <KeyVaultName> `
   -Name <KeyVaultKeyName> `
   -Destination <HardwareOrSoftware>
   ```

## <a name="option-2-manual-rotation"></a>選項 2：手動輪替

此選項會使用 [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey)、[Add-AzureRmSqlServerKeyVaultKey](/powershell/module/azurerm.sql/add-azurermsqlserverkeyvaultkey) 和 [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) Cmdlet 新增全新的金鑰，而此金鑰可能會以新的金鑰名稱甚或另一個金鑰保存庫建立。 

>[!NOTE]
>金鑰保存庫名稱和金鑰名稱的合併長度不可超過 94 個字元。
>

   ```powershell
   # Add a new key to Key Vault
   Add-AzureKeyVaultKey `
   -VaultName <KeyVaultName> `
   -Name <KeyVaultKeyName> `
   -Destination <HardwareOrSoftware>

   # Add the new key from Key Vault to the server
   Add-AzureRmSqlServerKeyVaultKey `
   -KeyId <KeyVaultKeyId> `
   -ServerName <LogicalServerName> `
   -ResourceGroup <SQLDatabaseResourceGroupName>   
  
   <# Set the key as the TDE protector for all resources 
   under the server #>
   Set-AzureRmSqlServerTransparentDataEncryptionProtector `
   -Type AzureKeyVault `
   -KeyId <KeyVaultKeyId> `
   -ServerName <LogicalServerName> `
   -ResourceGroup <SQLDatabaseResourceGroupName>
   ```
  
## <a name="other-useful-powershell-cmdlets"></a>其他實用的 PowerShell Cmdlet

- 若要將 TDE 保護裝置從 Microsoft 管理模式切換為 BYOK 模式，請使用 [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) Cmdlet。

   ```powershell
   Set-AzureRmSqlServerTransparentDataEncryptionProtector `
   -Type AzureKeyVault `
   -KeyId <KeyVaultKeyId> `
   -ServerName <LogicalServerName> `
   -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

- 若要將 TDE 保護裝置從 BYOK 模式切換為 Microsoft 管理模式，請使用 [Set-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/azurerm.sql/set-azurermsqlservertransparentdataencryptionprotector) Cmdlet。

   ```powershell
   Set-AzureRmSqlServerTransparentDataEncryptionProtector `
   -Type ServiceManaged `
   -ServerName <LogicalServerName> `
   -ResourceGroup <SQLDatabaseResourceGroupName> 
   ``` 

## <a name="next-steps"></a>後續步驟

- 如有安全性風險，請了解如何移除可能遭破壞的 TDE 保護裝置：[移除可能遭破壞的金鑰](transparent-data-encryption-byok-azure-sql-remove-tde-protector.md) 

- 開始將「攜帶您自己的金鑰支援」用於 TDE：[透過 PowerShell 從 Key Vault 使用您自己的金鑰開啟 TDE](transparent-data-encryption-byok-azure-sql-configure.md)
