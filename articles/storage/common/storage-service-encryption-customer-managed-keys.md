---
title: 使用 Azure Key Vault 中客戶管理的金鑰進行 Azure 儲存體服務加密 | Microsoft Docs
description: 使用「Azure 儲存體服務加密」功能，在儲存資料時於服務端加密 Azure Blob 儲存體和 Azure 檔案服務，然後在擷取資料時使用客戶管理的金鑰將它解密。
services: storage
author: lakasa
ms.service: storage
ms.topic: article
ms.date: 10/11/2018
ms.author: lakasa
ms.component: common
ms.openlocfilehash: 0ed05cab774360c4165e89399ba16f7443debb85
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165152"
---
# <a name="storage-service-encryption-using-customer-managed-keys-in-azure-key-vault"></a>使用 Azure Key Vault 中客戶管理的金鑰進行儲存體服務加密
Microsoft Azure 承諾協助您保護資料安全，以符合組織安全性和合規性承諾。 Azure 儲存體平台保護您資料的其中一種方式，就是透過「儲存體服務加密」(SSE)，它會在將資料寫入至儲存體時加密資料，並在擷取資料時將資料解密。 加密和解密會自動在背景中執行，並且使用 256 位元 [AES 加密](https://wikipedia.org/wiki/Advanced_Encryption_Standard) (可用的最強大區塊編碼器之一)。

您可以搭配 SSE 使用 Microsoft 管理的加密金鑰，或使用自己的加密金鑰。 本文說明如何使用您自己的加密金鑰。 如需有關使用 Microsoft 管理金鑰的詳細資訊，或有關 SSE 的一般資訊，請參閱[待用資料的儲存體服務加密](storage-service-encryption.md)。

適用於 Azure Blob 儲存體和 Azure 檔案服務的 SSE 已與 Azure Key Vault 整合，因此您可以使用金鑰保存庫來管理您的加密金鑰。 您可以建立自己的加密金鑰，然後將其儲存在金鑰保存庫中，或是使用 Azure Key Vault 的 API 來產生加密金鑰。 使用 Azure Key Vault 時，您可以管理和控制您的金鑰，也可以稽核金鑰的使用方式。

> [!Note]  
> 使用客戶管理金鑰的「儲存體服務加密」不適用於 [Azure 受控磁碟](../../virtual-machines/windows/managed-disks-overview.md)。 [Azure 磁碟加密](../../security/azure-security-disk-encryption-overview.md) 會在 Windows 上使用業界標準 [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview)，並在 Linux 上使用 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt)，以提供與 KeyVault 整合的加密解決方案。

為什麼要建立您自己的金鑰？ 自訂金鑰可賦予您更大的彈性，讓您能夠建立、輪替、停用及定義存取控制。 自訂金鑰也可讓您稽核用來保護資料的加密金鑰。

## <a name="get-started-with-customer-managed-keys"></a>開始使用客戶管理的金鑰
若要搭配 SSE 使用客戶管理的金鑰，您可以建立新的金鑰保存庫與金鑰，或是使用現有金鑰保存庫與金鑰。 儲存體帳戶與金鑰保存庫必須位於相同區域，但可位於不同的訂用帳戶中。 

### <a name="step-1-create-a-storage-account"></a>步驟 1：建立儲存體帳戶
如果您還沒有儲存體帳戶，請先建立一個帳戶。 如需詳細資訊，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。

### <a name="step-2-enable-sse-for-blob-and-file-storage"></a>步驟 2：為 Blob 和檔案儲存體啟用 SSE
若要啟用使用客戶管理金鑰的 SSE，必須一併在 Azure Key Vault 中啟用兩個金鑰保護功能，即「虛刪除」和「不要清除」。 這些設定可確保金鑰無法被意外或蓄意刪除。 金鑰的最長保留期是設定為 90 天，可保護使用者不受惡意執行者或勒索軟體攻擊。

如果您想要以程式設計方式針對 SSE 啟用客戶管理的金鑰，您可以使用 [Azure 儲存體資源提供者 REST API](https://docs.microsoft.com/rest/api/storagerp) \(英文\)、[適用於 .NET 的儲存體資源提供者用戶端程式庫](https://docs.microsoft.com/dotnet/api)、[Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) 或 [Azure CLI](https://docs.microsoft.com/azure/storage/storage-azure-cli)。

若是搭配 SSE 使用客戶管理的金鑰，您必須為儲存體帳戶指派一個儲存體帳戶身分識別。 您可以執行下列 PowerShell 命令來設定身分識別：

```powershell
Set-AzureRmStorageAccount -ResourceGroupName \$resourceGroup -Name \$accountName -AssignIdentity
```

您可以執行下列 PowerShell 命令來啟用「虛刪除」和「不要清除」：

```powershell
($resource = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -VaultName
$vaultName).ResourceId).Properties | Add-Member -MemberType NoteProperty -Name
enableSoftDelete -Value 'True'

Set-AzureRmResource -resourceid $resource.ResourceId -Properties
$resource.Properties

($resource = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -VaultName
$vaultName).ResourceId).Properties | Add-Member -MemberType NoteProperty -Name
enablePurgeProtection -Value 'True'

Set-AzureRmResource -resourceid $resource.ResourceId -Properties
$resource.Properties
```

### <a name="step-3-enable-encryption-with-customer-managed-keys"></a>步驟 3：使用客戶管理的金鑰來啟用加密
SSE 預設會使用 Microsoft 管理的金鑰。 您可以使用 [Azure 入口網站](https://portal.azure.com/)來啟用搭配客戶管理金鑰的 SSE。 在儲存體帳戶的 [設定] 刀鋒視窗上，按一下 [加密]。 選取 [使用您自己的金鑰] 選項，如下圖所示。

![入口網站螢幕擷取畫面顯示 [加密] 選項](./media/storage-service-encryption-customer-managed-keys/ssecmk1.png)

### <a name="step-4-select-your-key"></a>步驟 4：選取您的金鑰
您可以採用 URI 形式來指定您的金鑰，或從金鑰保存庫中選取金鑰。

#### <a name="specify-a-key-as-a-uri"></a>以 URI 形式指定金鑰
若要從 URI 指定金鑰，請依照下列步驟進行操作：

1. 選擇 [輸入金鑰 URI] 選項。  
2. 在 [金鑰 URI] 欄位中，指定 URI。

    ![顯示 [透過輸入金鑰 URI 來加密] 選項的入口網站螢幕擷取畫面](./media/storage-service-encryption-customer-managed-keys/ssecmk2.png)


#### <a name="specify-a-key-from-a-key-vault"></a>從金鑰保存庫指定金鑰 
若要從金鑰保存庫指定您的金鑰，請依照下列步驟進行操作：

1. 選擇 [從 Key Vault 選取] 選項。  
2. 選擇包含您想要使用之金鑰的金鑰保存庫。
3. 從金鑰保存庫選擇金鑰。

    ![顯示 [使用您自己的金鑰加密] 選項的入口網站螢幕擷取畫面](./media/storage-service-encryption-customer-managed-keys/ssecmk3.png)

如果儲存體帳戶沒有金鑰保存庫的存取權，您可以執行下圖所示的 Azure PowerShell 命令來授予存取權。

![顯示金鑰保存庫存取遭拒的入口網站螢幕擷取畫面](./media/storage-service-encryption-customer-managed-keys/ssecmk4.png)

您也可以透過 Azure 入口網站來授與存取權，方法是瀏覽至 Azure 入口網站中的 Azure Key Vault，然後將存取權授與儲存體帳戶。


您可以使用下列 PowerShell 命令，讓上述索引鍵與現有儲存體帳戶產生關聯：
```powershell
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName "myresourcegroup" -AccountName "mystorageaccount"
$keyVault = Get-AzureRmKeyVault -VaultName "mykeyvault"
$key = Get-AzureKeyVaultKey -VaultName $keyVault.VaultName -Name "keytoencrypt"
Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVault.VaultName -ObjectId $storageAccount.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
Set-AzureRmStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName -AccountName $storageAccount.StorageAccountName -KeyvaultEncryption -KeyName $key.Name -KeyVersion $key.Version -KeyVaultUri $keyVault.VaultUri
```

### <a name="step-5-copy-data-to-storage-account"></a>步驟 5︰將資料複製到儲存體帳戶
將資料傳送到新的儲存體帳戶，使其加密。 如需詳細資訊，請參閱[儲存體服務加密的常見問題集](storage-service-encryption.md#faq-for-storage-service-encryption)。

### <a name="step-6-query-the-status-of-the-encrypted-data"></a>步驟 6︰查詢加密資料的狀態
查詢加密資料的狀態。

## <a name="faq-for-sse-with-customer-managed-keys"></a>搭配客戶管理金鑰的 SSE 常見問題集
**我目前使用進階儲存體，是否可以搭配 SSE 使用客戶管理的金鑰？**  
是，在標準儲存體和進階儲存體上都支援搭配 Microsoft 管理金鑰及客戶管理金鑰的 SSE。

**是否可以使用 Azure PowerShell 和 Azure CLI 來建立新的儲存體帳戶並搭配客戶管理金鑰啟用 SSE？**  
是。

**如果搭配 SSE 使用客戶管理的金鑰，Azure 儲存體的成本會增加多少？**  
使用 Azure Key Vault 有相關成本。 如需詳細資訊，請瀏覽 [Key Vault 價格](https://azure.microsoft.com/pricing/details/key-vault/)。 使用 SSE 無須額外付費，針對所有儲存體帳戶都會啟用此功能。

**Azure 受控磁碟上是否可使用「儲存體服務加密」？**  
「儲存體服務加密」適用於使用 Microsoft 管理金鑰的 Azure 受控磁碟，但不適用於使用客戶管理金鑰的 Azure 受控磁碟。 若要替代支援 SSE 和客戶管理金鑰的受控磁碟，建議使用 [Azure 磁碟加密](../../security/azure-security-disk-encryption-overview.md)，這會在 Windows 上使用業界標準 [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview)，並在 Linux 上使用 [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt)，提供與 KeyVault 整合的加密功能。

**儲存體服務加密和 Azure 磁碟加密有何不同？**  
Azure 磁碟加密提供 OS 型解決方案 (例如 BitLocker 及 Dm-crypt) 與 Azure KeyVault 之間的整合。 儲存體服務加密可在虛擬機器下的 Azure 儲存體平台層提供原生加密。

**是否可以撤銷加密金鑰的存取權？**
是，您可以隨時撤銷存取權。 有幾種方法可以撤銷對金鑰的存取權。 如需詳細資訊，請參閱 [Azure Key Vault PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/) \(英文\) 與 [Azure Key Vault CLI](https://docs.microsoft.com/cli/azure/keyvault) \(英文\)。 撤銷存取權將有效封鎖對儲存體帳戶中所有 Blob 的存取，因為「Azure 儲存體」無法存取帳戶加密金鑰。

**是否可以在不同的區域建立儲存體帳戶和金鑰？**  
否，儲存體帳戶與 Azure Key Vault 及金鑰必須位於相同區域中。

**建立儲存體帳戶時，是否可以針對 SSE 啟用客戶管理的金鑰？**  
否。 當您第一次建立儲存體帳戶時，只有 Microsoft 管理的金鑰可供 SSE 使用。 若要使用客戶管理的金鑰，您必須更新儲存體帳戶屬性。 您可以使用 REST 或其中一個儲存體用戶端程式庫來以程式設計方式更新儲存體帳戶，或在建立帳戶之後使用 Azure 入口網站更新儲存體帳戶屬性。

**搭配 SSE 使用客戶管理的金鑰時，是否可以停用加密？**  
否，您無法停用加密。 針對 Azure Blob 儲存體、Azure 檔案服務、Azure 佇列及 Azure 資料表儲存體，預設都會啟用加密。 您可以視需要從使用 Microsoft 管理的金鑰切換成使用客戶管理的金鑰，反之亦然。

**我建立新的儲存體帳戶時是否會啟用 SSE？**  
所有儲存體帳戶與 Azure Blob 儲存體、Azure 檔案服務、Azure 佇列儲存體及 Azure 資料表儲存體，都會啟用 SSE。

**我無法在我的儲存體帳戶上啟用使用客戶管理金鑰的 SSE。**  
這是 Azure Resource Manager 儲存體帳戶嗎？ 不支援傳統儲存體帳戶使用客戶管理的金鑰。 搭配客戶管理金鑰的 SSE 只能在 Resource Manager 儲存體帳戶上啟用。

**什麼是「虛刪除」和「不要清除」？是否必須啟用此設定才能使用搭配客戶管理金鑰的 SSE？**  
必須啟用「虛刪除」和「不要清除」，才能使用搭配客戶管理金鑰的 SSE。 這些設定可確保您的金鑰不會被意外或蓄意刪除。 金鑰的最長保留期是設定為 90 天，可保護使用者不受惡意執行者及勒索軟體攻擊。 您無法停用此設定。

**是否只有在特定區域中才允許使用搭配客戶管理金鑰的 SSE？**  
在 Azure Blob 儲存體和 Azure 檔案服務的所有區域中，都可以使用搭配客戶管理金鑰的 SSE。

**如果我有任何問題或想要提供意見反應，要如何連絡相關人員？**  
如果有儲存體服務加密的相關問題，請連絡 [ssediscussions@microsoft.com](mailto:ssediscussions@microsoft.com)。

## <a name="next-steps"></a>後續步驟
- 如需協助開發人員建置安全應用程式之全方位安全功能的詳細資訊，請參閱[儲存體安全性指南](storage-security-guide.md)。
- 如需 Azure Key Vault 的概觀資訊，請參閱[什麼是 Azure Key Vault？](https://docs.microsoft.com/azure/key-vault/key-vault-whatis)
- 若要開始使用 Azure Key Vault，請參閱[開始使用 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-get-started)。
