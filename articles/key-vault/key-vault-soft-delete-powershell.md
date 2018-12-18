---
ms.assetid: ''
title: Azure Key Vault - 如何以 PowerShell 使用虛刪除
description: 以 PowerShell 程式碼片段進行虛刪除的使用案例範例
services: key-vault
author: lleonard-msft
manager: mbaldwin
ms.service: key-vault
ms.topic: article
ms.workload: identity
ms.date: 08/21/2017
ms.author: alleonar
ms.openlocfilehash: 48569e31e6400e3ec8958e0bceda1fd3b72207ea
ms.sourcegitcommit: 2a70752d0987585d480f374c3e2dba0cd5097880
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27927986"
---
# <a name="how-to-use-key-vault-soft-delete-with-powershell"></a>如何使用 Key Vault 虛刪除與 PowerShell

Azure Key Vault 的虛刪除功能可復原已刪除的保存庫和保存庫物件。 具體而言，虛刪除解決下列案例：

- 可復原的 Key Vault 刪除支援
- 支援可復原的金鑰保存庫物件刪除；金鑰、密碼和憑證

## <a name="prerequisites"></a>先決條件

- Azure PowerShell 4.0.0 或更新版本 - 如果您尚未安裝，請安裝 Azure PowerShell，並將它與 Azure 訂用帳戶建立關聯，請參閱[如何安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)。 

>[!NOTE]
> 載入您的環境的**可能**是過期版本的 Key Vault PowerShell 輸出格式檔案，而不是正確版本。 我們預計 PowerShell 的更新版本會包含輸出格式所需的更正，屆時將更新此主題。 如果您遇到此格式問題，目前的因應措施是：
> - 如果您注意到您看不到此主題中所述的已啟用虛刪除的屬性，請使用下列查詢：`$vault = Get-AzureRmKeyVault -VaultName myvault; $vault.EnableSoftDelete`。


如需 PowerShell 的 Key Vault 特定參考資訊，請參閱 [Azure Key Vault PowerShell 參考](https://docs.microsoft.com/powershell/module/azurerm.keyvault/?view=azurermps-4.2.0)。

## <a name="required-permissions"></a>所需的權限

Key Vault 作業透過角色型存取控制 (RBAC) 權限來分別管理，如下所示：

| 作業 | 說明 | 使用者權限 |
|:--|:--|:--|
|列出|列出已刪除的金鑰保存庫。|Microsoft.KeyVault/deletedVaults/read|
|復原|還原已刪除的金鑰保存庫。|Microsoft.KeyVault/vaults/write|
|清除|永久移除已刪除的金鑰保存庫和其所有內容。|Microsoft.KeyVault/locations/deletedVaults/purge/action|

如需權限和存取控制的詳細資訊，請參閱[保護您的金鑰保存庫](key-vault-secure-your-key-vault.md)。

## <a name="enabling-soft-delete"></a>啟用虛刪除

若要能夠復原已刪除的金鑰保存庫或儲存在金鑰保存庫中的物件，您必須先為該金鑰保存庫啟用虛刪除。

### <a name="existing-key-vault"></a>現有的金鑰保存庫

對於名為 ContosoVault 的現有金鑰保存庫啟用虛刪除，如下所示。 

>[!NOTE]
>目前，您需要使用 Azure Resource Manager 資源操作來直接寫入 Key Vault 資源的 *enableSoftDelete* 屬性。

```powershell
($resource = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -VaultName "ContosoVault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"

Set-AzureRmResource -resourceid $resource.ResourceId -Properties $resource.Properties
```

### <a name="new-key-vault"></a>新的金鑰保存庫

為新的金鑰保存庫啟用虛刪除是在建立時完成，方法是新增虛刪除啟用旗標到您的 create 命令。

```powershell
New-AzureRmKeyVault -VaultName "ContosoVault" -ResourceGroupName "ContosoRG" -Location "westus" -EnableSoftDelete
```

### <a name="verify-soft-delete-enablement"></a>驗證啟用虛刪除

若要驗證金鑰保存庫已啟用虛刪除，請執行 *get* 命令，並尋找「虛刪除已啟用?」 屬性，及其設定 true 或 false。

```powershell
Get-AzureRmKeyVault -VaultName "ContosoVault"
```

## <a name="deleting-a-key-vault-protected-by-soft-delete"></a>刪除虛刪除所保護的金鑰保存庫

刪除 (或移除) 金鑰保存庫的命令維持不變，但它的行為會根據您是否已啟用虛刪除而改變。

```powershell
Remove-AzureRmKeyVault -VaultName 'ContosoVault'
```

> [!IMPORTANT]
>如果您先前針對沒有啟用虛刪除的金鑰保存庫執行命令，您將永久刪除此金鑰保存庫及其所有的內容，並且沒有任何選項可以復原。

### <a name="how-soft-delete-protects-your-key-vaults"></a>虛刪除如何保護您的金鑰保存庫

已啟用虛刪除：

- 刪除金鑰保存庫時，它會從其資源群組中移除，並放置於僅與其建立所在位置建立關聯的保留命名空間。 
- 已刪除的金鑰保存庫中的物件，例如金鑰、密碼和憑證都無法存取，並且只要包含的金鑰保存庫處於已刪除狀態便維持不變。 
- 已刪除狀態中的金鑰保存庫 DNS 名稱仍會被保留，因此無法以相同的名稱建立新的金鑰保存庫。  

您可以檢視與您的訂用帳戶建立關聯的已刪除狀態金鑰保存庫，請使用下列命令：

```powershell
PS C:\> Get-AzureRmKeyVault -InRemovedStateVault 

Name           : ContosoVault
Location             : westus
Id                   : /subscriptions/xxx/providers/Microsoft.KeyVault/locations/westus/deletedVaults/ContosoVault
Resource ID          : /subscriptions/xxx/resourceGroups/ContosoVault/providers/Microsoft.KeyVault/vaults/ContosoVault
Deletion Date        : 5/9/2017 12:14:14 AM
Scheduled Purge Date : 8/7/2017 12:14:14 AM
Tags                 :
```

輸出中的「資源識別碼」是指此保存庫的原始資源識別碼。 因為此金鑰保存庫目前處於已刪除狀態，所以沒有具有該資源識別碼的資源存在。 「識別碼」欄位可以用來在復原或清除時識別資源。 「排定清除日期」欄位指出如果對這個已刪除的保存庫不採取任何動作，何時將永久刪除 (清除) 保存庫。 用來計算「排定清除日期」的預設保留期間為 90 天。

## <a name="recovering-a-key-vault"></a>復原金鑰保存庫

若要復原金鑰保存庫，您需要指定金鑰保存庫名稱、資源群組和位置。 請記下已刪除之金鑰保存庫的位置和資源群組，因為您需要這些才能進行金鑰保存庫復原程序。

```powershell
Undo-AzureRmKeyVaultRemoval -VaultName ContosoVault -ResourceGroupName ContosoRG -Location westus
```

在金鑰保存庫復原之後，結果會是新的資源，並具有金鑰保存庫的原始資源識別碼。 如果金鑰保存庫存在的資源群組已被移除，則必須先建立具有相同名稱的新資源群組，才能復原金鑰保存庫。

## <a name="key-vault-objects-and-soft-delete"></a>金鑰保存庫物件和虛刪除

對於已啟用虛刪除的金鑰保存庫 'ContosoVault'，其中的金鑰 'ContosoFirstKey'，以下為該金鑰的刪除方式。

```powershell
Remove-AzureKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey
```

在金鑰保存庫啟用虛刪除的情況下，已刪除的金鑰仍會看似已刪除，例外情況是當您明確地列出或擷取已刪除的金鑰時。 對於已刪除狀態的金鑰，大部分作業會失敗，只除了列出、復原或清除已刪除的金鑰時例外。 

例如，若要要求列出金鑰保存庫中的已刪除金鑰，請使用下列命令：

```powershell
Get-AzureKeyVaultKey -VaultName ContosoVault -InRemovedState
```

### <a name="transition-state"></a>轉換狀態 

當您刪除金鑰保存庫中的金鑰，並且已啟用虛刪除時，可能需要幾秒鐘的時間讓轉換完成。 在此轉換狀態期間，可能會出現金鑰不在使用中狀態或已刪除狀態。 此命令會列出名為 'ContosoVault' 的金鑰保存庫中，所有已刪除的金鑰。

```powershell
  Get-AzureKeyVaultKey -VaultName ContosoVault -InRemovedState
  Vault Name           : ContosoVault
  Name                 : ContosoFirstKey
  Id                   : https://ContosoVault.vault.azure.net:443/keys/ContosoFirstKey
  Deleted Date         : 2/14/2017 8:20:52 PM
  Scheduled Purge Date : 5/15/2017 8:20:52 PM
  Enabled              : True
  Expires              :
  Not Before           :
  Created              : 2/14/2017 8:16:07 PM
  Updated              : 2/14/2017 8:16:07 PM
  Tags                 :
```

### <a name="using-soft-delete-with-key-vault-objects"></a>對金鑰保存庫物件使用虛刪除

就像金鑰保存庫，已刪除的金鑰、密碼或憑證仍然會維時在已刪除狀態長達 90 天，除非加以復原或清除。 

#### <a name="keys"></a>金鑰

復原已刪除的金鑰：

```powershell
Undo-AzureKeyVaultKeyRemoval -VaultName ContosoVault -Name ContosoFirstKey
```

永久刪除金鑰：

```powershell
Remove-AzureKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey -InRemovedState
```

>[!NOTE]
>清除金鑰會永久刪除它，這表示它將無法復原。

**復原**和**清除**動作在金鑰保存庫存取原則中有自己的相關聯權限。 使用者或服務主體若要能夠執行**復原**或**清除**動作，必須在金鑰保存庫存取原則中有該物件 (金鑰或密碼) 的個別權限。 根據預設，**清除**權限不會在 'all' 捷徑用於授與所有權限給使用者時，新增到金鑰保存庫的存取原則。 您必須明確授與**清除**權限。 例如，下列命令授與 user@contoso.com 對 *ContosoVault* 中的金鑰執行數種作業的權限，這其中包括**清除**。

#### <a name="set-a-key-vault-access-policy"></a>設定金鑰保存庫存取原則

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToKeys get,create,delete,list,update,import,backup,restore,recover,purge
```

>[!NOTE] 
> 如果您有現有的金鑰保存庫，且剛剛啟用虛刪除，您可能沒有**復原**和**清除**權限。

#### <a name="secrets"></a>密碼

就像金鑰，金鑰保存庫中的密碼會有自己的操作指令。 以下是刪除、列出、復原和清除密碼的命令。

- 刪除名為 SQLPassword 的密碼： 
```powershell
Remove-AzureKeyVaultSecret -VaultName ContosoVault -name SQLPassword
```

- 列出金鑰保存庫中的所有已刪除密碼： 
```powershell
Get-AzureKeyVaultSecret -VaultName ContosoVault -InRemovedState
```

- 復原已刪除狀態的密碼： 
```powershell
Undo-AzureKeyVaultSecretRemoval -VaultName ContosoVault -Name SQLPAssword
```

- 清除已刪除狀態的密碼： 
```powershell
Remove-AzureKeyVaultSecret -VaultName ContosoVault -InRemovedState -name SQLPassword
```

>[!NOTE]
>清除密碼會永久刪除它，這表示它將無法復原。

## <a name="purging-and-key-vaults"></a>清除金鑰保存庫

### <a name="key-vault-objects"></a>金鑰保存庫物件

清除金鑰、密碼或憑證會永久刪除它，這表示它將無法復原。 不過，包含已刪除之物件的金鑰保存庫會維持不變，金鑰保存庫中的所有其他物件也是。 

### <a name="key-vaults-as-containers"></a>金鑰保存庫作為容器
當清除金鑰保存庫時，它的所有內容，包括金鑰、密碼和憑證，都會永久刪除。 若要清除金鑰保存庫，請使用 `Remove-AzureRmKeyVault` 命令與 `-InRemovedState` 選項，並使用 `-Location location` 引數指定已刪除之金鑰保存庫的位置。 您可以使用命令 `Get-AzureRmKeyVault -InRemovedState` 找到已刪除之保存庫的位置。

```powershell
Remove-AzureRmKeyVault -VaultName ContosoVault -InRemovedState -Location westus
```

>[!NOTE]
>清除金鑰保存庫會永久刪除它，這表示它將無法復原。

### <a name="purge-permissions-required"></a>所需的清除權限
- 若要清除已刪除的金鑰保存庫，以便保存庫和其所有內容永久移除，使用者需要執行 *Microsoft.KeyVault/locations/deletedVaults/purge/action* 作業的 RBAC 權限。 
- 若要列出已刪除的金鑰保存庫，使用者需要執行 *Microsoft.KeyVault/deletedVaults/read* 作業的 RBAC 權限。 
- 根據預設，只有訂用帳戶管理員擁有這些權限。 

### <a name="scheduled-purge"></a>排定的清除

列出您的已刪除金鑰保存庫物件，會顯示它們排定要由金鑰保存庫清除的時間。 「排定清除日期」欄位指出如果不採取任何動作，何時將永久刪除金鑰保存庫物件。 根據預設，已刪除的金鑰保存庫物件的保留期限為 90 天。

>[!NOTE]
>已清除的保存庫物件，由其「排定清除日期」欄位觸發，會永久刪除。 無法復原。

## <a name="other-resources"></a>其他資源

- 如需 Key Vault 的虛刪除功能概觀，請參閱 [Azure Key Vault 虛刪除概觀](key-vault-ovw-soft-delete.md)。
- 如需 Azure Key Vault 使用的一般概觀，請參閱[開始使用 Azure Key Vault](key-vault-get-started.md)。

