---
title: 使用 CLI 管理 Azure Key Vault | Microsoft Docs
description: 使用此文章運用 Azure CLI 將 Key Vault 中的常見工作自動化
services: key-vault
documentationcenter: ''
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: ''
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/28/2018
ms.author: barclayn
ms.openlocfilehash: ba39ba2d4206fa86ed32bddf0538a2e997a79339
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220562"
---
# <a name="manage-key-vault-using-the-azure-cli"></a>使用 Azure CLI 來管理 Key Vault 

此文章涵蓋如何運用 Azure CLI 來開始使用 Azure Key Vault。  請查看下列資訊：

- 如何在 Azure 中建立強行寫入的容器 (保存庫)
- 將金鑰、祕密或憑證新增至金鑰保存庫
- 向 Azure Active Directory 註冊應用程式
- 授權應用程式使用金鑰或祕密
- 設定金鑰保存庫進階存取原則
- 使用硬體安全模組 (HSM)
- 刪除金鑰保存庫及相關聯的金鑰和密碼
- 其他 Azure 跨平台命令列介面命令


大部分地區均提供 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [金鑰保存庫價格頁面](https://azure.microsoft.com/pricing/details/key-vault/)。

> [!NOTE]
> 此文章並未說明如何撰寫其中一個步驟所包含的 Azure 應用程式，但會示範如何授權應用程式使用金鑰保存庫中的金鑰或密碼。
>

如需 Azure Key Vault 的概觀，請參閱[什麼是 Azure Key Vault？](key-vault-whatis.md)
如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

若要使用此文章中的 Azure CLI 命名，您必須具有下列項目：

* Microsoft Azure 訂用帳戶。 如果您沒有帳戶，您可以註冊[免費試用](https://azure.microsoft.com/pricing/free-trial)。
* Azure 命令列介面 2.0 版或更新版本。 若要安裝最新版本，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。
* 可將應用程式設定為使用您在此文章中所建立的金鑰或密碼。 您可以在 [Microsoft 下載中心](http://www.microsoft.com/download/details.aspx?id=45343)找到範例應用程式。 如需相關指示，請參閱內含的讀我檔案。

### <a name="getting-help-with-azure-cross-platform-command-line-interface"></a>取得使用 Azure 跨平台命令列介面的說明

此文章假設您熟悉命令列介面 (Bash、終端機、命令提示字元)。

--help 或 -h 參數可用於檢視特定命令的說明。 或者，也可以使用 Azure help [command] [options] 格式。 不確定命令所需的參數時，請查閱說明。 例如，以下命令全部都會傳回相同的資訊：

```azurecli
az account set --help
az account set -h
```

您也可以閱讀下列文章，以熟悉 Azure 跨平台命令列介面中的 Azure 資源管理員：

* [安裝 Azure CLI](/cli/azure/install-azure-cli)
* [開始使用 Azure CLI](/cli/azure/get-started-with-azure-cli)

## <a name="how-to-create-a-hardened-container-a-vault-in-azure"></a>如何在 Azure 中建立強行寫入的容器 (保存庫)

保存庫由硬體安全性模組 (HSM) 支援的安全容器。 保存庫透過集中儲存應用程式祕密，協助減少意外遺失安全性資訊的機會。 Key Vault 也會控制和記錄其中所儲存項目的存取權。 Azure Key Vault 可以處理要求及更新傳輸層安全性 (TLS) 憑證，並提供穩健憑證生命週期管理解決方案所需的功能。 在後續步驟中，您將建立保存庫。

### <a name="connect-to-your-subscriptions"></a>連線到您的訂閱

若要以互動方式登入，請使用下列命令：

```azurecli
az login
```
若要使用組織帳戶進行登入，您可以傳入使用者名稱和密碼。

```azurecli
az login -u username@domain.com -p password
```

如果您有多個訂用帳戶，並且需要指定要使用哪一個，請輸入下列內容以查看您帳戶的訂用帳戶：

```azurecli
az account list
```

使用訂用帳戶參數指定訂用帳戶。

```azurecli
az account set --subscription <subscription name or ID>
```

如需設定 Azure 跨平台命令列介面的詳細資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

### <a name="create-a-new-resource-group"></a>建立新的資源群組

使用 Azure 資源管理員時，會在資源群組內建立所有相關資源。 您可以在現有的資源群組中建立金鑰保存庫。 如果您想使用新的資源群組，可以建立一個新的。

```azurecli
az group create -n 'ContosoResourceGroup' -l 'East Asia'
```

第一個參數是資源群組名稱，而第二個參數是位置。 若要取得所有可能位置類型的清單：

```azurecli
az account list-locations
``` 

### <a name="register-the-key-vault-resource-provider"></a>註冊金鑰保存庫資源提供者

 當您嘗試建立新的金鑰保存庫時，可能會看到「訂用帳戶未登錄要使用命名空間 'Microsoft.KeyVault'」錯誤。 如果出現該訊息，請確定是否已在訂用帳戶中註冊 Key Vault 資源提供者。 每個訂用帳戶只需執行一次此作業。

```azurecli
az provider register -n Microsoft.KeyVault
```

### <a name="create-a-key-vault"></a>建立金鑰保存庫

使用 `az keyvault create` 命令來建立金鑰保存庫。 這個指令碼包含三個必要參數：資源群組名稱、金鑰保存庫名稱和地理位置。

若要在位於 [東亞] 位置的 **ContosoResourceGroup** 資源群組中，建立名稱為 **ContosoKeyVault** 的新保存庫，請輸入： 

```azurecli
az keyvault create --name 'ContosoKeyVault' --resource-group 'ContosoResourceGroup' --location 'East Asia'
```

此命令的輸出會顯示您所建立的金鑰保存庫屬性。 兩個最重要屬性是：

* **name**：在此範例中，名稱是 ContosoKeyVault。 您將在其他 Key Vault 命令中使用此名稱。
* **vaultUri**：在此範例中，URI 是 https://contosokeyvault.vault.azure.net。 透過其 REST API 使用保存庫的應用程式必須使用此 URI。

您的 Azure 帳戶現已取得在此金鑰保存庫上執行任何作業的授權。 而且，沒有其他人已獲授權。

## <a name="adding-a-key-secret-or-certificate-to-the-key-vault"></a>將金鑰、祕密或憑證新增至金鑰保存庫

如果您想讓 Azure Key Vault 為您建立一個軟體防護金鑰，請使用 `az key create` 命令。

```azurecli
az keyvault key create --vault-name 'ContosoKeyVault' --name 'ContosoFirstKey' --protection software
```

如果您在 .pem 檔案中有現有的金鑰，您可以將它上傳至 Azure Key Vault。 您可以選擇使用軟體或 HSM 保護金鑰。 使用下列步驟從 .pem 檔案匯入金鑰，並使用軟體保護它：

```azurecli
az keyvault key import --vault-name 'ContosoKeyVault' --name 'ContosoFirstKey' --pem-file './softkey.pem' --pem-password 'Pa$$w0rd' --protection software
```

您現在可以參照您所建立或上傳至 Azure 金鑰保存庫的金鑰 (透過使用其 URI)。 使用 **https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey** 一律可取得最新版本。 使用 https://[keyvault-name].vault.azure.net/keys/[keyname]/[key-unique-id] 可取得此特定版本。 例如：**https://ContosoKeyVault.vault.azure.net/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87**。 

將名為 SQLPassword 且其 Azure Key Vault 的值為 Pa$$w0rd 的密碼新增至保存庫。 

```azurecli
az keyvault secret set --vault-name 'ContosoKeyVault' --name 'SQLPassword' --value 'Pa$$w0rd'
```

使用此密碼的 URI 參考此密碼。 使用 **https://ContosoVault.vault.azure.net/secrets/SQLPassword** 一律可取得目前的版本，而使用 https://[keyvault-name].vault.azure.net/secret/[secret-name]/[secret-unique-id] 可取得特定版本。 例如：**https://ContosoVault.vault.azure.net/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d**。

使用 .pem 或 .pfx 將憑證匯入至保存庫。

```azurecli
az keyvault certificate import --vault-name 'ContosoKeyVault' --file 'c:\cert\cert.pfx' --name 'ContosoCert' --password 'Pa$$w0rd'
```

讓我們檢視您所建立的金鑰、祕密或憑證：

* 若要檢視您的金鑰，請輸入： 

```azurecli
az keyvault key list --vault-name 'ContosoKeyVault'
```

* 若要檢視您的祕密，請輸入： 

```azurecli
az keyvault secret list --vault-name 'ContosoKeyVault'
```

* 若要檢視您的憑證，請輸入： 

```azurecli
az keyvault certificate list --vault-name 'ContosoKeyVault'
```

## <a name="registering-an-application-with-azure-active-directory"></a>向 Azure Active Directory 註冊應用程式

這步驟通常會由開發人員在個別電腦上完成。 這並非 Azure Key Vault 的特有狀況，在此列出是為了提醒注意。 若要完成應用程式註冊，您的帳戶、保存庫及應用程式都必須位於相同的 Azure 目錄中。

使用金鑰保存庫的應用程式必須使用 Azure Active Directory 的權杖進行驗證。  應用程式擁有者必須先在 Azure Active Directory 中註冊該應用程式。 註冊結束時，應用程式擁有者會取得下列值：

- **應用程式識別碼** (也稱為 AAD 用戶端識別碼或 appID)
- **驗證金鑰** (也稱為共用密碼)。 

應用程式必須向 Azure Active Directory 出示這兩個值才能取得權杖。 設定應用程式以取得權杖的方式取決於應用程式。 在 [Key Vault 範例應用程式](https://www.microsoft.com/download/details.aspx?id=45343)中，應用程式擁有者會在 app.config 檔案中設定這些值。

如需有關使用 Azure Active Directory 來註冊應用程式的詳細步驟，您應檢閱標題為[整合應用程式與 Azure Active Directory](../active-directory/develop/active-directory-integrating-applications.md)、[使用入口網站來建立可存取資源的 Azure Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)及[使用 Azure CLI 來建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)的文章。

若要在 Azure Active Directory 中註冊應用程式：

```azurecli
az ad sp create-for-rbac -n "MyApp" --password 'Pa$$w0rd' --skip-assignment
# If you don't specify a password, one will be created for you.
```

## <a name="authorizing-an-application-to-use-a-key-or-secret"></a>授權應用程式使用金鑰或祕密

若要授權應用程式存取保存庫中的金鑰或密碼，請使用 `az keyvault set-policy` 命令。

例如，如果您的保存庫名稱為 ContosoKeyVault，應用程式的 appID 為 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed，而且您想要授權應用程式使用保存庫中的金鑰來進行解密並簽署，請使用下列命令：

```azurecli
az keyvault set-policy --name 'ContosoKeyVault' --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --key-permissions decrypt sign
```

若要授權相同的應用程式讀取您保存庫中的機密資料，請輸入以下命令：

```azurecli
az keyvault set-policy --name 'ContosoKeyVault' --spn 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed --secret-permissions get
```

## <a name="bkmk_KVperCLI"></a> 設定金鑰保存庫進階存取原則

使用 [az keyvault update](/cli/azure/keyvault#az-keyvault-update) 啟用金鑰保存庫的進階的原則。 

 啟用 Key Vault 以供部署：允許虛擬機器從保存庫擷取儲存為祕密的憑證。
 ```azurecli
 az keyvault update --name 'ContosoKeyVault' --resource-group 'ContosoResourceGroup' --enabled-for-deployment 'true'
 ``` 

啟用 Key Vault 以供磁碟加密：使用保存庫進行 Azure 磁碟加密時所需。

 ```azurecli
 az keyvault update --name 'ContosoKeyVault' --resource-group 'ContosoResourceGroup' --enabled-for-disk-encryption 'true'
 ```  

啟用 Key Vault 以供範本部署：允許 Resource Manager 從保存庫擷取祕密。
 ```azurecli 
 az keyvault update --name 'ContosoKeyVault' --resource-group 'ContosoResourceGroup' --enabled-for-template-deployment 'true'
 ```

## <a name="working-with-hardware-security-modules-hsms"></a>使用硬體安全模組 (HSM)

為了加強保證，您可以從硬體安全模組 (HSM) 匯入或產生無需離開 HSM 界限的金鑰。 HSM 已通過 FIPS 140-2 Level 2 驗證。 如果此需求對您不適用，請略過本節並移至 [刪除金鑰保存庫及相關聯的金鑰和密碼](#delete-the-key-vault-and-associated-keys-and-secrets)。

若要建立這些受 HSM 保護的金鑰，您必須具備支援受 HSM 保護之金鑰的保存庫訂閱。

建立金鑰保存庫時，請新增 'sku' 參數：

```azurecli
az keyvault create --name 'ContosoKeyVaultHSM' --resource-group 'ContosoResourceGroup' --location 'East Asia' --sku 'Premium'
```

您可以將軟體防護金鑰 (如稍早所示) 和受 HSM 保護的金鑰新增至此保存庫。 若要建立受 HSM 保護的金鑰，請將 [目的地] 參數設為 'HSM'：

```azurecli
az keyvault key create --vault-name 'ContosoKeyVaultHSM' --name 'ContosoFirstHSMKey' --protection 'hsm'
```

您可以使用下列命令，從電腦上的 .pem 檔案匯入金鑰。 此命令會將金鑰匯入金鑰保存庫服務中的 HSM：

```azurecli
az keyvault key import --vault-name 'ContosoKeyVaultHSM' --name 'ContosoFirstHSMKey' --pem-file '/.softkey.pem' --protection 'hsm' --pem-password 'PaSSWORD'
```

下一個命令會匯入「自備金鑰」(BYOK) 封包。 這可讓您在您的本機 HSM 中產生金鑰，且在金鑰無需離開 HSM 界限的情況下，即可將它傳輸到金鑰保存庫服務中的 HSM：

```azurecli
az keyvault key import --vault-name 'ContosoKeyVaultHSM' --name 'ContosoFirstHSMKey' --byok-file './ITByok.byok' --protection 'hsm'
```

如需有關如何產生此 BYOK 封包的詳細指示，請參閱 [如何使用 Azure 金鑰保存庫中受 HSM 保護的金鑰](key-vault-hsm-protected-keys.md)。

## <a name="deleting-the-key-vault-and-associated-keys-and-secrets"></a>刪除金鑰保存庫及相關聯的金鑰和密碼

如果您不再需要金鑰保存庫及其所金鑰或祕密，可以使用 `az keyvault delete` 命令來刪除金鑰保存庫：

```azurecli
az keyvault delete --name 'ContosoKeyVault'
```

或者，您可以刪除整個 Azure 資源群組，其中包括金鑰保存庫和您加入該群組的任何其他資源：

```azurecli
az group delete --name 'ContosoResourceGroup'
```

## <a name="miscellaneous-azure-cross-platform-command-line-interface-commands"></a>其他 Azure 跨平台命令列介面命令

可能有助於管理 Azure Key Vault 的其他命令。

此命令會列出以表格形式顯示的所有金鑰和所選屬性：

```azurecli
az keyvault key list --vault-name 'ContosoKeyVault'
```

此命令會顯示指定金鑰的完整屬性清單：

```azurecli
az keyvault key show --vault-name 'ContosoKeyVault' --name 'ContosoFirstKey'
```

此命令會列出以表格形式顯示的所有密碼名稱和所選屬性：

```azurecli
az keyvault secret list --vault-name 'ContosoKeyVault'
```

以下是如何移除特定金鑰的範例：

```azurecli
az keyvault key delete --vault-name 'ContosoKeyVault' --name 'ContosoFirstKey'
```

以下是如何移除特定密碼的範例：

```azurecli
az keyvault secret delete --vault-name 'ContosoKeyVault' --name 'SQLPassword'
```

## <a name="next-steps"></a>後續步驟

- 如需金鑰保存庫命令的完整 Azure CLI 參考，請參閱 [Key Vault CLI 參考](/cli/azure/keyvault)。

- 如需程式設計參考，請參閱 [Azure 金鑰保存庫開發人員指南](key-vault-developers-guide.md)。

- 如需 Azure 金鑰保存庫和 HSM 的相關資訊，請參閱[如何使用 Azure 金鑰保存庫中受 HSM 保護的金鑰](key-vault-hsm-protected-keys.md)。
