---
title: 將 Azure 中 Windows VM 上的磁碟加密 | Microsoft Docs
description: 如何使用 Azure PowerShell 將 Windows VM 上的虛擬磁碟加密以增強安全性
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/07/2018
ms.author: cynthn
ms.openlocfilehash: 20d3568fa3f583c190f087de861d857fe3e793a9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985418"
---
# <a name="how-to-encrypt-virtual-disks-on-a-windows-vm"></a>如何將 Windows VM 上的虛擬磁碟加密
如需強化虛擬機器 (VM) 安全性與法規遵循，可以將 Azure 中的虛擬磁碟加密。 磁碟會使用 Azure 金鑰保存庫中受保護的密碼編譯金鑰進行加密。 您可控制這些密碼編譯金鑰，並可稽核其使用情況。 本文詳細說明如何使用 Azure PowerShell 將 Windows VM 上的虛擬磁碟加密。 您也可以[使用 Azure CLI 將 Linux VM 加密](../linux/encrypt-disks.md)。

## <a name="overview-of-disk-encryption"></a>磁碟加密概觀
加密 Windows VM 上的虛擬磁碟時，是在待用時使用 Bitlocker 進行加密。 將 Azure 中的虛擬磁碟加密完全免費。 密碼編譯金鑰會使用軟體保護功能儲存在 Azure 金鑰保存庫中，或者您可以在 FIPS 140-2 第 2 級標準認證的硬體安全性模組 (HSM) 中匯入或產生金鑰。 這些密碼編譯金鑰用來加密及解密連接到 VM 的虛擬磁碟。 您可保留這些密碼編譯金鑰的控制權，並可稽核其使用情況。 Azure Active Directory 服務主體會提供一個安全機制，以便在 VM 開機或關機時發出這些密碼編譯金鑰。

將 VM 加密的程序如下所示：

1. 在 Azure 金鑰保存庫中建立密碼編譯金鑰。
2. 將密碼編譯金鑰設定為可用於磁碟加密。
3. 若要讀取 Azure Key Vault 中的密碼編譯金鑰，請使用適當的權限建立 Azure Active Directory 服務主體。
4. 發出命令以將您的虛擬磁碟加密，並指定要使用的 Azure Active Directory 服務主體和適當密碼編譯金鑰。
5. Azure Active Directory 服務主體會向 Azure Key Vault 要求所需的密碼編譯金鑰。
6. 虛擬磁碟會使用所提供的密碼編譯金鑰進行加密。

## <a name="encryption-process"></a>加密程序
磁碟加密依賴下列其他元件︰

* **Azure 金鑰保存庫** - 用來保護磁碟加密/解密程序所使用的密碼編譯金鑰和密碼。 
  * 如果有的話，您可以使用現有的 Azure 金鑰保存庫。 您沒有專門用來加密磁碟的金鑰保存庫。
  * 若要區分系統管理界限和金鑰可視性，您可以建立專用的金鑰保存庫。
* **Azure Active Directory** - 處理必要密碼編譯金鑰的安全交換以及所要求動作的驗證。 
  * 您通常使用現有的 Azure Active Directory 執行個體來裝載您的應用程式。
  * 服務主體會提供一個安全機制來要求並取得適當的密碼編譯金鑰。 您並未開發與 Azure Active Directory 整合的實際應用程式。

## <a name="requirements-and-limitations"></a>需求和限制
磁碟加密支援的案例和需求：

* 從 Azure Marketplace 映像或自訂 VHD 映像在新的 Windows VM 上啟用加密。
* 在 Azure 中現有的 Windows VM 上啟用加密。
* 在使用「儲存空間」來設定的 Windows VM 上啟用加密。
* 在 Windows VM 的 OS 和資料磁碟機上停用加密。
* 所有資源 (例如金鑰保存庫、儲存體帳戶、VM) 必須位於相同的 Azure 區域和訂用帳戶。
* 標準層 VM，例如 A、D、DS、G 及 GS 系列 VM。

下列案例目前不支援磁碟加密︰

* 基本層 VM。
* 使用傳統部署模型建立的 VM。
* 在已經加密的 VM 上更新密碼編譯金鑰。
* 與內部部署「金鑰管理服務」整合。

## <a name="create-azure-key-vault-and-keys"></a>建立 Azure Key Vault 和金鑰
開始之前，請確定已安裝最新版的 Azure PowerShell 模組。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。 在整個命令範例中，以您自己的名稱、位置和金鑰值取代所有的範例參數。 下列範例使用 *myResourceGroup*、*myKeyVault*、*myVM* 等慣例。

建立 Azure 金鑰保存庫的第一個步驟是儲存您的密碼編譯金鑰。 Azure 金鑰保存庫儲存可讓您安全地在應用程式和服務中實作的金鑰和密碼 (Secret 或 Password)。 針對虛擬磁碟加密，您需建立 Key Vault 來儲存用來加密或解密虛擬磁碟的密碼編譯金鑰。 

請使用 [Register-AzureRmResourceProvider](/powershell/module/azurerm.resources/register-azurermresourceprovider) 來啟用您 Azure 訂用帳戶內的 Azure Key Vault 提供者，然後使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 來建立資源群組。 下列範例會在 *East US* 位置建立名為 *myResourceGroup* 的資源群組：

```powershell
$rgName = "myResourceGroup"
$location = "East US"

Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.KeyVault"
New-AzureRmResourceGroup -Location $location -Name $rgName
```

包含密碼編譯金鑰和相關聯計算資源 (例如儲存體和 VM) 的 Azure 金鑰保存庫本身必須位於相同的區域中。 請使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/new-azurermkeyvault) 來建立 Azure Key Vault，然後啟用 Key Vault 以搭配磁碟加密使用。 針對 *keyVaultName* 指定一個唯一的 Key Vault 名稱，如下所示︰

```powershell
$keyVaultName = "myUniqueKeyVaultName"
New-AzureRmKeyVault -Location $location `
    -ResourceGroupName $rgName `
    -VaultName $keyVaultName `
    -EnabledForDiskEncryption
```

您可以使用軟體或硬體安全性模型 (HSM) 保護功能來儲存密碼編譯金鑰。 使用 HSM 時需要進階金鑰保存庫。 建立進階金鑰保存庫 (而非用來儲存軟體保護金鑰的標準金鑰保存庫) 會有額外的成本。 若要建立進階 Key Vault，請在上一個步驟中新增 *-Sku "Premium"* 參數。 下列範例會使用軟體保護的金鑰，因為我們建立了標準金鑰保存庫。 

在兩種保護模型中，Azure 平台都必須獲得存取權，才能在 VM 開機時要求密碼編譯金鑰來將虛擬磁碟解密。 請使用 [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) 在您的 Key Vault 中建立密碼編譯金鑰。 下列範例會建立一個名為 *myKey* 的金鑰：

```powershell
Add-AzureKeyVaultKey -VaultName $keyVaultName `
    -Name "myKey" `
    -Destination "Software"
```


## <a name="create-the-azure-active-directory-service-principal"></a>建立 Azure Active Directory 服務主體
將虛擬磁碟加密或解密時，您可以指定帳戶以處理 Key Vault 中密碼編譯金鑰的驗證和交換。 此帳戶 (Azure Active Directory 服務主體) 可讓 Azure 平台代表 VM 要求適當的密碼編譯金鑰。 雖然許多組織都有專用的 Azure Active Directory 目錄，但您的訂用帳戶中會有預設 Azure Active Directory 執行個體。

請使用 [New-AzureRmADServicePrincipal](/powershell/module/azurerm.resources/new-azurermadserviceprincipal) 在 Azure Active Directory 中建立服務主體。 若要指定安全的密碼，請依照 [Azure Active Directory 中的密碼原則和限制](../../active-directory/authentication/concept-sspr-policy.md)所述進行操作：

```powershell
$appName = "My App"
$securePassword = ConvertTo-SecureString -String "P@ssw0rd!" -AsPlainText -Force
$app = New-AzureRmADApplication -DisplayName $appName `
    -HomePage "https://myapp.contoso.com" `
    -IdentifierUris "https://contoso.com/myapp" `
    -Password $securePassword
New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
```

若要成功加密或解密虛擬磁碟，Key Vault 中儲存之密碼編譯金鑰的權限必須設定為允許 Azure Active Directory 服務主體讀取金鑰。 請使用 [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) 來設定 Key Vault 的權限：

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName $keyvaultName `
    -ServicePrincipalName $app.ApplicationId `
    -PermissionsToKeys "WrapKey" `
    -PermissionsToSecrets "Set"
```


## <a name="create-virtual-machine"></a>建立虛擬機器
為了測試加密程序，讓我們使用 [New-AzureRmVm](/powershell/module/azurerm.compute/new-azurermvm) 建立一個 VM。 下列範例會使用 *Windows Server 2016 Datacenter* 映像來建立一個名為 *myVM* 的 VM。 出現提供認證的提示時，請輸入要用於 VM 的使用者名稱和密碼：

```powershell
$cred = Get-Credential

New-AzureRmVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```


## <a name="encrypt-virtual-machine"></a>將虛擬機器加密
若要將虛擬磁碟加密，您可結合所有先前的元件︰

1. 指定 Azure Active Directory 服務主體和密碼。
2. 指定金鑰保存庫以儲存已加密磁碟的中繼資料。
3. 指定要用於實際加密和解密的密碼編譯金鑰。
4. 指定是否要將作業系統磁碟、資料磁碟或所有磁碟加密。

請使用 [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) 搭配 Azure Key Vault 金鑰和 Azure Active Directory 服務主體認證，來加密您的 VM。 下列範例會擷取所有金鑰資訊，然後加密名為 *myVM* 的 VM：

```powershell
$keyVault = Get-AzureRmKeyVault -VaultName $keyVaultName -ResourceGroupName $rgName;
$diskEncryptionKeyVaultUrl = $keyVault.VaultUri;
$keyVaultResourceId = $keyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $keyVaultName -Name myKey).Key.kid;

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgName `
    -VMName "myVM" `
    -AadClientID $app.ApplicationId `
    -AadClientSecret (New-Object PSCredential "user",$securePassword).GetNetworkCredential().Password `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl `
    -DiskEncryptionKeyVaultId $keyVaultResourceId `
    -KeyEncryptionKeyUrl $keyEncryptionKeyUrl `
    -KeyEncryptionKeyVaultId $keyVaultResourceId
```

接受提示以繼續進行 VM 加密。 VM 會在此程序中重新啟動。 加密程序完成且 VM 重新啟動之後，請使用 [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) 來檢閱加密狀態：

```powershell
Get-AzureRmVmDiskEncryptionStatus  -ResourceGroupName $rgName -VMName "myVM"
```

輸出類似於下列範例：

```powershell
OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : Encrypted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OsVolume: Encrypted, DataVolumes: Encrypted
```

## <a name="next-steps"></a>後續步驟
* 如需有關管理 Azure Key Vault 的詳細資訊，請參閱[為虛擬機器設定 Key Vault](key-vault-setup.md)。
* 如需有關磁碟加密 (例如準備加密的自訂 VM 以上傳至 Azure) 的詳細資訊，請參閱 [Azure 磁碟加密](../../security/azure-security-disk-encryption.md)。
