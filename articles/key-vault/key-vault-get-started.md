---
title: 開始使用 Azure 金鑰保存庫 | Microsoft Docs
description: 使用本教學課程可協助您開始使用 Azure 金鑰保存庫，進而在 Azure 中建立強化的容器，以儲存及管理 Azure 中的密碼編譯金鑰和密碼。
services: key-vault
documentationcenter: ''
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 36721e1d-38b8-4a15-ba6f-14ed5be4de79
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 05/10/2018
ms.author: barclayn
ms.openlocfilehash: 3f3adb1230d6ca6b3a7e616a0beed15d66895124
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43282992"
---
# <a name="get-started-with-azure-key-vault"></a>開始使用 Azure 金鑰保存庫
本文可協助您使用 PowerShell 來開始使用 Azure Key Vault，並引導您完成下列活動：
- 如何在 Azure 中建立強行寫入的容器 (保存庫)。
- 如何使用 KeyVault 來儲存和管理密碼編譯金鑰和 Azure 中的祕密。
- 應用程式可以如何使用該金鑰或密碼。

大部分地區均提供 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [金鑰保存庫價格頁面](https://azure.microsoft.com/pricing/details/key-vault/)。

如需跨平台命令列介面的指示，請參閱 [這個對等的教學課程](key-vault-manage-with-cli2.md)。

## <a name="requirements"></a>需求
在繼續進行文章之前，請先確認您有：

- **Azure 訂用帳戶**。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/en-us/free/)。
- **Azure PowerShell**，**最低版本為 1.1.0**。 若要安裝 Azure PowerShell，並將它與 Azure 訂用帳戶建立關聯，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 如果您已安裝 Azure PowerShell 但不知道版本，請在 Azure PowerShell 主控台中輸入 `(Get-Module azure -ListAvailable).Version`。 如果您已安裝 Azure PowerShell 版本 0.9.1 至 0.9.8，您仍可使用本教學課程並稍作變更。 例如，您必須使用 `Switch-AzureMode AzureResourceManager` 命令，而有些 Azure 金鑰保存庫命令已變更。 如需版本 0.9.1 至 0.9.8 的金鑰保存庫 Cmdlet 清單，請參閱 [Azure 金鑰保存庫 Cmdlet](/powershell/module/azurerm.keyvault/#key_vault)。
- **可以設定為使用 Key Vault 的應用程式**。 您可以在 [Microsoft 下載中心](http://www.microsoft.com/download/details.aspx?id=45343)找到範例應用程式。 如需相關指示，請參閱隨附的**讀我**檔案。

>[!NOTE]
本文假設您對 PowerShell 和 Azure 有基本認識。 如需 PowerShell 的詳細資訊，請參閱[開始使用 Windows PowerShell](https://technet.microsoft.com/library/hh857337.aspx)。

若要取得您在本教學課程中任何所見 Cmdlet 的詳細說明，請使用 **Get-Help** Cmdlet。

```powershell-interactive
Get-Help <cmdlet-name> -Detailed
```
    
例如，如需取得 **Connect-AzureRmAccount** Cmdlet 的說明，請輸入：

```PowerShell
Get-Help Connect-AzureRmAccount -Detailed
```

您也可以閱讀下列文章，以熟悉 Azure PowerShell 中的 Azure Resource Manager 部署模型：

* [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)
* [Azure PowerShell 搭配資源管理員使用](../powershell-azure-resource-manager.md)

## <a id="connect"></a>連線到您的訂用帳戶
開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：  

```PowerShell
Connect-AzureRmAccount
```

>[!NOTE]
 如果您是使用 Azure 的特定執行個體，請使用 -Environment 參數。 例如︰ 
 ```powershell
 Connect-AzureRmAccount –Environment (Get-AzureRmEnvironment –Name AzureUSGovernment)
 ```

在快顯瀏覽器視窗中，輸入您的 Azure 帳戶使用者名稱與密碼。 Azure PowerShell 會取得與此帳戶相關聯的所有訂用帳戶，並依預設使用第一個訂用帳戶。

如果您有多個訂用帳戶，並想要指定其中一個訂用帳戶供 Azure 金鑰保存庫使用，請輸入下列內容以查看您帳戶的訂用帳戶：

```powershell
Get-AzureRmSubscription
```

接著，若要指定要使用的訂用帳戶，請輸入：

```powershell
Set-AzureRmContext -SubscriptionId <subscription ID>
```

如需設定 Azure PowerShell 的詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a id="resource"></a>建立新的資源群組
使用 Azure 資源管理員時，會在資源群組中建立所有相關資源。 在本教學課程中，我們將建立名為 **ContosoResourceGroup** 的新資源群組：

```powershell
New-AzureRmResourceGroup –Name 'ContosoResourceGroup' –Location 'East US'
```

## <a id="vault"></a>建立金鑰保存庫
使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/new-azurermkeyvault) Cmdlet 來建立金鑰保存庫。 這個 Cmdlet 包含三個必要參數：**資源群組名稱**、**金鑰保存庫名稱**和**地理位置**。

例如，如果您使用：
- 保存庫名稱 **ContosoKeyVault**。
- 資源群組名稱 **ContosoResourceGroup**。
- **美國東部**位置。

您要輸入：

```powershell
New-AzureRmKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East US'
```
![Key Vault 建立命令完成後的輸出](./media/key-vault-get-started/output-after-creating-keyvault.png)

此 Cmdlet 的輸出會顯示您所建立的 Key Vault 屬性。 兩個最重要屬性是：

* **保存庫名稱**：在此範例中是 **ContosoKeyVault**。 您將在其他金鑰保存庫 Cmdlet 中使用此名稱。
* **保存庫 URI**：在此範例中是 https://contosokeyvault.vault.azure.net/。 透過其 REST API 使用保存庫的應用程式必須使用此 URI。

您的 Azure 帳戶現已取得在此金鑰保存庫上執行任何作業的授權。 而且，沒有其他人有此授權。

> [!NOTE]
> 當您嘗試建立新的 Key Vault 時，可能會看到**訂用帳戶未登錄要使用命名空間 'Microsoft.KeyVault'** 錯誤。 如果顯示該訊息，請執行 `Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.KeyVault"`。 註冊成功完成後，您可以重新執行 New-AzureRmKeyVault 命令。 如需詳細資訊，請參閱 [Register-AzureRmResourceProvider](/powershell/module/azurerm.resources/register-azurermresourceprovider)。
>
>

## <a id="add"></a>新增金鑰或密碼至金鑰保存庫
您可能需要幾種不同的方式來與 Key Vault 和金鑰或祕密進行互動。

### <a name="azure-key-vault-generates-a-software-protected-key"></a>Azure Key Vault 會產生軟體受保護的金鑰

如果您要 Azure Key Vault 為您建立一個軟體防護金鑰，請使用 [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) Cmdlet，並輸入：

```powershell
$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -Destination 'Software'
```
以檢視此金鑰類型的 URI：
```powershell
$key.id
```

您可以參照您所建立或上傳至 Azure Key Vault 的金鑰 (藉由使用其 URI)。 若要取得目前的版本，您可以使用 **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey** 和使用 **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87** 來取得此特定版本。  

### <a name="importing-an-existing-pfx-file-into-azure-key-vault"></a>將現有的 PFX 檔案匯入 Azure Key Vault 中

如果現有金鑰是儲存在您需要上傳至 Azure Key Vault 的 pfx 檔案中，步驟就會不同。 例如︰
- 如果您在 .PFX 檔案中有現有的軟體受保護金鑰
- pfx 檔案就會命名為 softkey.pfx 
- 檔案會儲存在 C 磁碟機中。

您可以輸入：

```powershell
$securepfxpwd = ConvertTo-SecureString –String '123' –AsPlainText –Force  // This stores the password 123 in the variable $securepfxpwd
```

接著，輸入下列內容從 .PFX 檔案匯入金鑰，如此一來便會使用金鑰保存庫服務中的軟體來保護金鑰：

```powershell
$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoImportedPFX' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd
```

若要顯示此金鑰的 URI，請輸入：

```powershell
$Key.id
```
若要檢視您的金鑰，請輸入： 

```powershell
Get-AzureKeyVaultKey –VaultName 'ContosoKeyVault'
```
如果您需要在入口網站上檢視 PFX 檔案的內容，就會看到類似如下所示映像的內容。

![憑證在入口網站中的外觀](./media/key-vault-get-started/imported-pfx.png)
### <a name="to-add-a-secret-to-azure-key-vault"></a>若要將祕密新增至 Azure Key Vault

若要將名為 SQLPassword 且其 Azure Key Vault 的值為 Pa$$w0rd 的密碼新增至保存庫，首先您必須輸入下列內容來將 Pa$$w0rd 值轉換成安全字串：

```powershell    
$secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force
```

然後輸入：

```powershell
$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword' -SecretValue $secretvalue
```


透過使用其 URI，您現在可以參照您新增至 Azure 金鑰保存庫的密碼。 使用 **https://ContosoVault.vault.azure.net/secrets/SQLPassword** 可一律取得目前的版本，使用 **https://ContosoVault.vault.azure.net/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** 則可取得此特定版本。

若要顯示此密碼的 URI，請輸入：

```powershell
$secret.Id
```
若要檢視您的祕密，請輸入：`Get-AzureKeyVaultSecret –VaultName 'ContosoKeyVault'` 或者您可以在入口網站上檢視祕密。

![secret](./media/key-vault-get-started/secret-value.png)

若要以純文字檢視包含在祕密中的值：
```powershell
(get-azurekeyvaultsecret -vaultName "Contosokeyvault" -name "SQLPassword").SecretValueText
```
現在，您可以在應用程式中使用金鑰保存庫和金鑰或密碼。 您必須授權應用程式才能使用他們。  

## <a id="register"></a>向 Azure Active Directory 註冊應用程式
這步驟通常會由開發人員在個別電腦上完成。 並不特定於 Azure Key Vault。 如需使用 Azure Active Directory 註冊應用程式的詳細步驟，您應檢閱標題為[整合應用程式與 Azure Active Directory](../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md)或[使用入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)

> [!IMPORTANT]
> 若要完成本教學課程，您的帳戶、保存庫及將在本步驟中註冊的應用程式全都必須位於相同的 Azure 目錄中。


使用金鑰保存庫的應用程式必須使用 Azure Active Directory 的權杖進行驗證。 若要達到此目的，應用程式擁有者首先必須在其 Azure Active Directory 中註冊該應用程式。 註冊結束時，應用程式擁有者會取得下列值：

- **應用程式識別碼** 
- **驗證金鑰** (也稱為共用密碼)。 

應用程式必須向 Azure Active Directory 出示這兩個值才能取得權杖。 如何設定應用程式執行此作業會取決於應用程式。 在 [Key Vault 範例應用程式](https://www.microsoft.com/download/details.aspx?id=45343)中，應用程式擁有者會在 app.config 檔案中設定這些值。


在 Azure Active Directory 中註冊應用程式：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側按一下 [應用程式註冊]。 如果您沒有看到應用程式註冊，請按一下 [更多服務] 並在該處找到它。  
>[!NOTE]
您必須選取您用來建立金鑰保存庫的 Azure 訂用帳戶所在的同一個目錄。 
3. 按一下 [新增應用程式註冊]。
4. 在 [建立] 刀鋒視窗上提供您的應用程式名稱，然後選取 [WEB 應用程式和/或 WEB API] (預設值)，並指定 Web 應用程式的 [登入 URL]。 如果您目前沒有這項資訊，可在此步驟中假設一個 (例如，您可以指定 http://test1.contoso.com)。 這些網站是否存在並不重要。 

    ![新增應用程式註冊](./media/key-vault-get-started/new-application-registration.png)
    >[!WARNING]
    請確定您已選擇 [Web 應用程式和/或 WEB API]，否則將不會看到設定下的 [金鑰] 選項。

5. 按一下 [ **建立** ] 按鈕。
6. 完成應用程式註冊時，可以看到已註冊應用程式的清單。 尋找您剛註冊的應用程式，並按一下它。
7. 按一下 [註冊的應用程式] 刀鋒視窗並複製 [應用程式識別碼]
8. 按一下 [所有設定]
9. 在 [設定] 刀鋒視窗上，按一下 [金鑰]
9. 在 [金鑰描述] 方塊中輸入描述並選取持續時間，然後按一下 [儲存]。 頁面會重新整理，並顯示金鑰值。 
10. 您將在下一個步驟中使用 [應用程式識別碼] 和 [金鑰] 資訊來設定您保存庫的權限。

## <a id="authorize"></a>授權應用程式使用金鑰或密碼
有兩種方式可授權應用程式存取保存庫中的金鑰或密碼。

### <a name="using-powershell"></a>使用 PowerShell
若要使用 PowerShell，請使用 [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) Cmdlet。

例如，如果您的保存庫名稱是 **ContosoKeyVault** ，且您要授權的應用程式具有 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed 的用戶端識別碼，您想要授權應用程式使用保存庫中的金鑰來進行解密並簽署，則請執行下列作業：

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign
```

如果您想要授權該相同的應用程式讀取您保存庫中的機密資料，請執行以下命令：

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToSecrets Get
```
### <a name="using-the-azure-portal"></a>使用 Azure 入口網站
若要將應用程式的授權變更為使用金鑰或密碼：
1. 從 [金鑰保存庫] 資源刀鋒視窗中選取 [存取原則]
2. 按一下刀鋒視窗頂端的 [+ 新增] 按鈕
3. 按一下 [選取主體] 以選取您稍早建立的應用程式
4. 從 [金鑰權限] 下拉式清單中，選取 [解密] 和 [簽署] 來授權應用程式使用保存庫中的金鑰解密和簽署
5. 從 [祕密權限] 下拉式清單中，選取 [取得] 以允許應用程式讀取保存庫中的祕密

## <a id="HSM"></a>使用硬體安全模組 (HSM)
為了加強保證，您可以在硬體安全模組 (HSM) 中匯入或產生無需離開 HSM 界限的金鑰。 HSM 已通過 FIPS 140-2 Level 2 驗證。 如果此需求對您不適用，請略過本節並移至 [刪除金鑰保存庫及相關聯的金鑰和密碼](#delete)。

若要建立這些受 HSM 保護的金鑰，您必須使用 [Azure 金鑰保存庫高階服務層以支援受 HSM 保護的金鑰](https://azure.microsoft.com/pricing/details/key-vault/)。 此外，請注意此功能不適用於 Azure China。

建立金鑰保存庫時，請新增 **-SKU** 參數：

```powershell
New-AzureRmKeyVault -VaultName 'ContosoKeyVaultHSM' -ResourceGroupName 'ContosoResourceGroup' -Location 'East US' -SKU 'Premium'
```


您可以將軟體防護金鑰 (如稍早所示) 和受 HSM 保護的金鑰新增至此金鑰保存庫。 若要建立受 HSM 保護的金鑰，請將 **-Destination** 參數設為 'HSM'：

```powershell
$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -Destination 'HSM'
```

您可以使用下列命令，來從電腦上的 .PFX 檔案匯入金鑰。 此命令會將金鑰匯入金鑰保存庫服務中的 HSM：

```powershell
$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd -Destination 'HSM'
```

下一個命令會匯入「自備金鑰」(BYOK) 封包。 此案例可讓您在您的本機 HSM 中產生金鑰，且在金鑰無需離開 HSM 界限的情況下，即可將它傳輸到金鑰保存庫服務中的 HSM：

```powershell
$key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMKey' -KeyFilePath 'c:\ITByok.byok' -Destination 'HSM'
```

如需有關如何產生此 BYOK 封裝的詳細指示，請參閱 [如何傳輸 Azure 金鑰保存庫的受 HSM 保護金鑰](key-vault-hsm-protected-keys.md)。

## <a id="delete"></a>刪除金鑰保存庫及相關聯的金鑰和密碼
如果您不再需要金鑰保存庫及其所包含的金鑰或密碼，您可以使用 [Remove-AzureRmKeyVault](/powershell/module/azurerm.keyvault/remove-azurermkeyvault) Cmdlet 來刪除金鑰保存庫：

```powershell
Remove-AzureRmKeyVault -VaultName 'ContosoKeyVault'
```

或者，您可以刪除整個 Azure 資源群組，其中包括金鑰保存庫和您加入該群組的任何其他資源：

```powershell
Remove-AzureRmResourceGroup -ResourceGroupName 'ContosoResourceGroup'
```

## <a id="other"></a>其他 Azure PowerShell Cmdlet
可能有助於管理 Azure 金鑰保存庫的其他命令：

- `$Keys = Get-AzureKeyVaultKey -VaultName 'ContosoKeyVault'`此命令會取得以表格形式顯示的所有金鑰和所選屬性。
- `$Keys[0]`此命令會顯示特定金鑰的完整屬性清單。
- `Get-AzureKeyVaultSecret`此命令會列出以表格形式顯示的所有密碼名稱和所選屬性。
- `Remove-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey'`：如何移除特定金鑰範例。
- `Remove-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword'`：如何移除特定密碼範例。

## <a name="next-steps"></a>後續步驟

- 如需 Azure 金鑰保存庫的概觀資訊，請參閱 [什麼是 Azure 金鑰保存庫？](key-vault-whatis.md)
- 若要查看金鑰保存庫的使用情形，請參閱 [Azure 金鑰保存庫記錄](key-vault-logging.md)。
- 如需在 Web 應用程式中使用 Azure 金鑰保存庫的後續教學課程，請參閱 [從 Web 應用程式使用 Azure 金鑰保存庫](key-vault-use-from-web-application.md)。
- 如需程式設計參考，請參閱 [Azure 金鑰保存庫開發人員指南](key-vault-developers-guide.md)。
- 如需最新 Azure 金鑰保存庫的 Azure PowerShell Cmdlet 清單，請參閱 [Azure 金鑰保存庫 Cmdlet](/powershell/module/azurerm.keyvault/#key_vault)。
