---
title: 使用 Azure PowerShell 加密 Azure 擴展集的磁碟 | Microsoft Docs
description: 了解如何使用 Azure PowerShell 加密 Windows 虛擬機器擴展集中的 VM 執行個體和已連結的磁碟
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/30/2018
ms.author: cynthn
ms.openlocfilehash: c39df127e07601dbc5125d246f25da22ce4a0e40
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981758"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-azure-powershell-preview"></a>使用 Azure PowerShell (預覽) 加密虛擬機器擴展集中的 OS 和連結的資料磁碟

為了使用業界標準加密技術來保護與防禦待用資料，虛擬機器擴展集支援 Azure 磁碟加密 (ADE)。 您可以針對 Windows 和 Linux 虛擬機器擴展集啟用加密。 如需詳細資訊，請參閱 [Windows 和 Linux 適用的 Azure 磁碟加密](../security/azure-security-disk-encryption.md)。

> [!NOTE]
>  適用於虛擬機器擴展集的 Azure 磁碟加密目前處於公開預覽，可在所有 Azure 公用區域使用。

Azure 磁碟加密支援：
- 使用受控磁碟建立的擴展集，且不支援原生 (或非受控) 磁碟擴展集。
- Windows 擴展集中的作業系統和資料磁碟區。 停用加密支援 Windows 擴展集的作業系統和資料磁碟區。
- Linux 擴展集中的資料磁碟區。 目前的 Linux 擴展集預覽中「不」支援作業系統磁碟加密。

目前的預覽中不支援擴展集 VM 重新安裝映像和升級作業。 建議您僅在測試環境中使用虛擬機器擴展集預覽的 Azure 磁碟加密。 在預覽中，請勿在生產環境中啟用磁碟加密，因為您在其中可能必須升級加密擴展集中的作業系統映像。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="register-for-disk-encryption-preview"></a>註冊磁碟加密預覽

適用於虛擬機器擴展集預覽的 Azure 磁碟加密會要求您使用 [Register-AzureRmProviderFeature](/powershell/module/azurerm.resources/register-azurermproviderfeature) 自行註冊訂用帳戶。 您只需要在第一次使用磁碟加密預覽功能時，執行下列步驟：

```azurepowershell-interactive
Register-AzureRmProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName "UnifiedDiskEncryption"
```

註冊要求傳播的時間最多可能需要 10 分鐘。 您可以使用 [Get-AzureRmProviderFeature](/powershell/module/AzureRM.Resources/Get-AzureRmProviderFeature) 檢查註冊狀態。 當 `RegistrationState` 回報「已登錄」時，請使用 [Register-AzureRmResourceProvider](/powershell/module/AzureRM.Resources/Register-AzureRmResourceProvider) 重新登錄 *Mirosoft.Compute* 提供者：

```azurepowershell-interactive
Get-AzureRmProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute
```

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>建立針對磁碟加密啟用的 Azure Key Vault

Azure 金鑰保存庫儲存可讓您安全地在應用程式和服務中實作的金鑰和密碼 (Secret 或 Password)。 密碼編譯金鑰會使用軟體保護功能儲存在 Azure 金鑰保存庫中，或者您可以在 FIPS 140-2 第 2 級標準認證的硬體安全性模組 (HSM) 中匯入或產生金鑰。 這些密碼編譯金鑰用來加密及解密連接到 VM 的虛擬磁碟。 您可保留這些密碼編譯金鑰的控制權，並可稽核其使用情況。

使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/new-azurermkeyvault) 建立 Key Vault。 若要允許 Key Vault 用於磁碟加密，請設定 EnabledForDiskEncryption 參數。 下列範例也會定義資源群組名稱、Key Vault 名稱和位置的變數。 提供您自己唯一的 Key Vault 名稱：

```azurepowershell-interactive
$rgName="myResourceGroup"
$vaultName="myuniquekeyvault"
$location = "EastUS"

New-AzureRmResourceGroup -Name $rgName -Location $location
New-AzureRmKeyVault -VaultName $vaultName -ResourceGroupName $rgName -Location $location -EnabledForDiskEncryption
```

### <a name="use-an-existing-key-vault"></a>使用現有的 Key Vault

如果您有想要用於磁碟加密的現有 Key Vault，才需要這個步驟。 如果您已在上一節中建立 Key Vault，請略過此步驟。

您可以使用 [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/AzureRM.KeyVault/Set-AzureRmKeyVaultAccessPolicy)，在與磁碟加密的擴展集相同之訂用帳戶與區域中，啟用現有的 Key Vault。 在 $vaultName 變數中定義現有 Key Vault 的名稱，如下所示：

```azurepowershell-interactive
$vaultName="myexistingkeyvault"
Set-AzureRmKeyVaultAccessPolicy -VaultName $vaultName -EnabledForDiskEncryption
```

## <a name="create-a-scale-set"></a>建立擴展集

首先，使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 設定虛擬機器執行個體的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 建立虛擬機器擴展集。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。 負載平衡器包含在 TCP 連接埠 80 上散發流量的規則，同時允許 TCP 連接埠 3389 上的遠端桌面流量以及 TCP 連接埠 5985 上的 PowerShell 遠端流量：

```azurepowershell-interactive
$vmssName="myScaleSet"

New-AzureRmVmss `
    -ResourceGroupName $rgName `
    -VMScaleSetName $vmssName `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -PublicIpAddressName "myPublicIPAddress" `
    -LoadBalancerName "myLoadBalancer" `
    -UpgradePolicy "Automatic" `
    -Credential $cred
```

## <a name="enable-encryption"></a>啟用加密

若要加密擴展集中的虛擬機器執行個體，首先請使用 [Get-AzureRmKeyVault](/powershell/module/AzureRM.KeyVault/Get-AzureRmKeyVault) 取得 Key Vault URI 和資源識別碼的部分資訊。 然後使用這些變數，以 [Set-AzureRmVmssDiskEncryptionExtension](/powershell/module/AzureRM.Compute/Set-AzureRmVmssDiskEncryptionExtension) 啟動加密程序：

```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzureRmKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzureRmKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId

Set-AzureRmVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId –VolumeType "All"
```

出現提示時，輸入 y 以繼續擴展集虛擬機器執行個體上的磁碟加密程序。

## <a name="check-encryption-progress"></a>檢查加密程序

若要檢查磁碟加密的狀態，請使用 [Get-AzureRmVmssDiskEncryption](/powershell/module/AzureRM.Compute/Get-AzureRmVmssDiskEncryption)：

```azurepowershell-interactive
Get-AzureRmVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

當虛擬機器執行個體已加密時，EncryptionSummary 程式碼會報告 ProvisioningState/succeeded，如下列範例輸出所示：

```powershell
ResourceGroupName            : myResourceGroup
VmScaleSetName               : myScaleSet
EncryptionSettings           :
  KeyVaultURL                : https://myuniquekeyvault.vault.azure.net/
  KeyEncryptionKeyURL        :
  KeyVaultResourceId         : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myuniquekeyvault
  KekVaultResourceId         :
  KeyEncryptionAlgorithm     :
  VolumeType                 : All
  EncryptionOperation        : EnableEncryption
EncryptionSummary[0]         :
  Code                       : ProvisioningState/succeeded
  Count                      : 2
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="disable-encryption"></a>停用加密

如果您不想再使用加密的虛擬機器執行個體磁碟，您可以使用 [Disable-AzureRmVmssDiskEncryption](/powershell/module/AzureRM.Compute/Disable-AzureRmVmssDiskEncryption) 停用加密，如下所示：

```azurepowershell-interactive
Disable-AzureRmVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

## <a name="next-steps"></a>後續步驟

在本文中，您使用 Azure PowerShell 來加密虛擬機器擴展集。 您也可以使用 [Azure CLI](virtual-machine-scale-sets-encrypt-disks-cli.md) 或適用於 [Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-windows-jumpbox) 或 [Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-linux-jumpbox) 的範本。
