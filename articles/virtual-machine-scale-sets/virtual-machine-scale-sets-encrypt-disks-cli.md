---
title: 使用 Azure CLI 加密 Azure 擴展集的磁碟 | Microsoft Docs
description: 了解如何使用 Azure CLI 加密 Linux 虛擬機器擴展集中的 VM 執行個體和與連結的磁碟
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
ms.openlocfilehash: fecf7e121b1c0404f1ec67d46cc88008250b9d68
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981054"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-the-azure-cli-preview"></a>使用 Azure CLI (預覽) 加密虛擬機器擴展集中的 OS 與連結的資料磁碟

為了使用業界標準加密技術來保護與防禦待用資料，虛擬機器擴展集支援 Azure 磁碟加密 (ADE)。 您可以針對 Linux 和 Windows 虛擬機器擴展集啟用加密。 如需詳細資訊，請參閱 [Linux 和 Windows 適用的 Azure 磁碟加密](../security/azure-security-disk-encryption.md)。

> [!NOTE]
>  適用於虛擬機器擴展集的 Azure 磁碟加密目前處於公開預覽，可在所有 Azure 公用區域使用。

Azure 磁碟加密支援：
- 使用受控磁碟建立的擴展集，且不支援原生 (或非受控) 磁碟擴展集。
- Windows 擴展集中的作業系統和資料磁碟區。 停用加密支援 Windows 擴展集的作業系統和資料磁碟區。
- Linux 擴展集中的資料磁碟區。 目前的 Linux 擴展集預覽中「不」支援作業系統磁碟加密。

目前的預覽中不支援擴展集 VM 重新安裝映像和升級作業。 建議您僅在測試環境中使用虛擬機器擴展集預覽的 Azure 磁碟加密。 在預覽中，請勿在生產環境中啟用磁碟加密，因為您在其中可能必須升級加密擴展集中的作業系統映像。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.31 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="register-for-disk-encryption-preview"></a>註冊磁碟加密預覽

適用於虛擬機器擴展集預覽的 Azure 磁碟加密會要求您使用 [az feature register](/cli/azure/feature#az_feature_register) 自行註冊訂用帳戶。 您只需要在第一次使用磁碟加密預覽功能時，執行下列步驟：

```azurecli-interactive
az feature register --name UnifiedDiskEncryption --namespace Microsoft.Compute
```

註冊要求傳播的時間最多可能需要 10 分鐘。 您可以使用 [az feature show](/cli/azure/feature#az_feature_show) 檢查註冊狀態。 當 `State` 回報「已登錄」時，請使用 [az provider register](/cli/azure/provider#az_provider_register) 重新登錄 *Mirosoft.Compute* 提供者：

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```

## <a name="create-a-scale-set"></a>建立擴展集

請先使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組，才可以建立擴展集。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

現在使用 [az vmss create](/cli/azure/vmss#az_vmss_create) 建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，其已設定為在套用變更時自動更新，並在 ~/.ssh/id_rsa 中沒有 SSH 金鑰時產生 SSH 金鑰。 系統會將 32 Gb 資料磁碟連結至每個 VM 執行個體，並使用 Azure [自訂指令碼擴充功能](../virtual-machines/linux/extensions-customscript.md)搭配 [az vmss extension set](/cli/azure/vmss/extension#az_vmss_extension_set) 來準備資料磁碟：

```azurecli-interactive
# Create a scale set with attached data disk
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 32

# Prepare the data disk for use with the Custom Script Extension
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>建立針對磁碟加密啟用的 Azure 金鑰保存庫

Azure 金鑰保存庫儲存可讓您安全地在應用程式和服務中實作的金鑰和密碼 (Secret 或 Password)。 密碼編譯金鑰會使用軟體保護功能儲存在 Azure 金鑰保存庫中，或者您可以在 FIPS 140-2 第 2 級標準認證的硬體安全性模組 (HSM) 中匯入或產生金鑰。 這些密碼編譯金鑰用來加密及解密連接到 VM 的虛擬磁碟。 您可保留這些密碼編譯金鑰的控制權，並可稽核其使用情況。

定義您自己的唯一 *keyvault_name*。 接著，在與擴展集相同的訂用帳戶和區域中，使用 [az keyvault create](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-create) 建立 KeyVault，並設定 *--enabled-for-disk-encryption* 存取原則。

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault create --resource-group myResourceGroup --name $keyvault_name --enabled-for-disk-encryption
```

### <a name="use-an-existing-key-vault"></a>使用現有的 Key Vault

如果您有想要用於磁碟加密的現有 Key Vault，才需要這個步驟。 如果您已在上一節中建立 Key Vault，請略過此步驟。

定義您自己的唯一 *keyvault_name*。 接著，使用 [az keyvault update](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-update) 更新您的 KeyVault，並設定 *--enabled-for-disk-encryption* 存取原則。

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault update --name $keyvault_name --enabled-for-disk-encryption
```

## <a name="enable-encryption"></a>啟用加密

若要加密擴展集中的 VM 執行個體，請先使用 [az keyvault show](/cli/azure/ext/keyvault-preview/keyvault#ext-keyvault-preview-az-keyvault-show) 取得 Key Vault 資源識別碼的部分資訊。 然後使用這些變數搭配 [az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable) 開始執行加密程序：

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

加密程序開始可能需要一兩分鐘的時間。

由於擴展集是在先前步驟設定為「自動」時建立的擴展集上的升級原則，因此 VM 執行個體會自動開始執行加密程序。 在升級原則為手動的擴展集上，使用 [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances) 對 VM 執行個體啟動加密原則。

## <a name="check-encryption-progress"></a>檢查加密程序

若要檢查磁碟加密的狀態，請使用 [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show)：

```azurecli-interactive
az vmss encryption show --resource-group myResourceGroup --name myScaleSet
```

當 VM 執行個體經過加密時，狀態碼會回報 *EncryptionState/encrypted*，如下列範例輸出所示：

```bash
[
  {
    "disks": [
      {
        "encryptionSettings": null,
        "name": "myScaleSet_myScaleSet_0_disk2_3f39c2019b174218b98b3dfae3424e69",
        "statuses": [
          {
            "additionalProperties": {},
            "code": "EncryptionState/encrypted",
            "displayStatus": "Encryption is enabled on disk",
            "level": "Info",
            "message": null,
            "time": null
          }
        ]
      }
    ],
    "id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualMachines/0",
    "resourceGroup": "MYRESOURCEGROUP"
  }
]
```

## <a name="disable-encryption"></a>停用加密

如果您不想再使用加密的 VM 執行個體磁碟，則可以如下所示，使用 [az vmss encryption disable](/cli/azure/vmss/encryption?view=azure-cli-latest#az-vmss-encryption-disable) 來停用加密：

```azurecli-interactive
az vmss encryption disable --resource-group myResourceGroup --name myScaleSet
```

## <a name="next-steps"></a>後續步驟

在此文章中，您使用 Azure CLI 加密虛擬機器擴展集。 您也可以使用 [Azure PowerShell](virtual-machine-scale-sets-encrypt-disks-ps.md) 或適用於 [Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-windows-jumpbox) 或 [Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-linux-jumpbox) 的範本。

您可以在[這裡](https://gist.githubusercontent.com/ejarvi/7766dad1475d5f7078544ffbb449f29b/raw/03e5d990b798f62cf188706221ba6c0c7c2efb3f/enable-linux-vmss.bat)找到 Linux 擴展集資料磁碟加密的端對端批次檔案範例。 這個範例會建立資源群組和 Linux 擴展集、掛接 5 GB 資料磁碟，並加密虛擬機器擴展集。