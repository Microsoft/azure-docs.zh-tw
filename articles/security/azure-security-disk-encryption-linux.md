---
title: Linux IaaS VM 適用的 Azure 磁碟加密 | Microsoft Docs
description: 本文提供啟用 Linux IaaS VM 適用的 Microsoft Azure 磁碟加密的指示。
services: security
documentationcenter: na
author: mestew
manager: MBaldwin
ms.assetid: 071ee0c3-2e6c-4ea9-bfc7-908865629144
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/24/2018
ms.author: mstewart
ms.openlocfilehash: 5a885ed7c7bf451e0ee087fd4bda0a0e69e7ccad
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42886761"
---
# <a name="enable-azure-disk-encryption-for-linux-iaas-vms"></a>啟用 Linux IaaS VM 適用的 Azure 磁碟加密 

您可以啟用許多磁碟加密案例，步驟可能會因案例而有所不同。 下列各節涵蓋更詳細的 Linux IaaS VM 案例。 您必須先完成 [Azure 磁碟加密必要條件](azure-security-disk-encryption-prerequisites.md)，而且應該檢閱 [Linux IaaS VM 的其他必要條件](azure-security-disk-encryption-prerequisites.md#bkmk_LinuxPrereq)一節，才能您使用磁碟加密。

在磁碟加密前製作[快照集](../virtual-machines/windows/snapshot-copy-managed-disk.md)及/或進行備份。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 具有受控磁碟的 VM 需要有備份，才能進行加密。 在建立備份後，您可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 如需如何備份和還原已加密 VM 的詳細資訊，請參閱 [Azure 備份](../backup/backup-azure-vms-encryption.md)一文。 

>[!WARNING]
 >Azure 磁碟加密需要讓 Key Vault 和 VM 共置於相同區域中。 請在和所要加密 VM 相同的區域中建立並使用 Key Vault。</br></br>

> 在加密 Linux OS 磁碟區時，此程序可能需要幾小時的時間。 相較於資料磁碟區，加密 Linux OS 磁碟區所需的時間通常比較長。 

>只有資料磁碟區支援在 Linux VM 上停用加密。 如果 OS 磁碟區已加密，則不支援在資料或 OS 磁碟區上停用加密。  


## <a name="bkmk_RunningLinux"></a> 在現有或執行中的 IaaS Linux VM 上啟用加密
在這個案例中，您可以使用 Resource Manager 範本、PowerShell Cmdlet 或 CLI 命令啟用加密。 

>[!IMPORTANT]
 >您必須在啟用 Azure 磁碟加密之前，先在 Azure 磁碟加密之外對以受控磁碟為基礎的 VM 執行個體建立快照集和/或備份。 您可以從入口網站建立受控磁碟的快照集，也可以使用 [Azure 備份](../backup/backup-azure-vms-encryption.md)來建立。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 在建立備份後，您就可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 在建立好備份並指定了這個參數之前，對以受控磁碟為基礎的 VM 執行 Set-AzureRmVMDiskEncryptionExtension 命令都將會失敗。 
>
>加密或停用加密功能，可能會導致 VM 重新啟動。 
>

### <a name="bkmk_RunningLinuxCLI"> </a>使用 Azure CLI 在現有或執行中的 Linux VM 上啟用加密 
您可以安裝並使用 [Azure CLI 2.0](/cli/azure) 命令列工具，在加密的 VHD 上啟用磁碟加密。 您可以在瀏覽器中將它與 [Azure Cloud Shell](../cloud-shell/overview.md) 搭配使用，或可將它安裝在本機電腦上，並在任何 PowerShell 工作階段中使用它。 若要在 Azure 中現有或執行中的 IaaS Linux VM 上啟用加密，請使用下列 CLI 命令：

在 Azure 中使用 [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) 命令以在執行中的 IaaS 虛擬機器上啟用加密。

-  **加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **使用 KEK 加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

    >[!NOTE]
    > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> </br>
key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **確認磁碟已加密：** 若要檢查 IaaS VM 的加密狀態，請使用 [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) 命令。 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
     ```

- **停用加密：** 若要停用加密，請使用 [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) 命令。 只能在 Linux VM 的資料磁碟區上停用加密。

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type DATA
     ```

### <a name="bkmk_RunningLinuxPSH"> </a>使用 PowerShell 在現有或執行中的 Linux VM 上啟用加密
在 Azure 中使用 [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) cmdlet 以在執行中的 IaaS 虛擬機器上啟用加密。 

-  **加密執行中的 VM：** 下面指令碼會初始化您的變數並執行 Set-AzureRmVMDiskEncryptionExtension cmdlet。 資源群組、VM 和金鑰保存庫應該已經建立為必要條件。 使用您的值取代 MySecureRg、MySecureVM 和 MySecureVault。 如果您要加密資料磁碟而非作業系統磁碟，則可能需要新增 -VolumeType 參數。 

     ```azurepowershell-interactive
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **使用 KEK 加密執行中的 VM：** 如果您要加密資料磁碟而非作業系統磁碟，則可能需要新增 -VolumeType 參數。 

     ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```

    >[!NOTE]
    > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 
    
- **確認磁碟已加密：** 若要檢查 IaaS VM 的加密狀態，請使用 [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) cmdlet。 
    
     ```azurepowershell-interactive 
     Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName MySecureRg -VMName MySecureVM
     ```
    
- **停用磁碟加密：** 若要停用加密，請使用 [Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet。 只能在 Linux VM 的資料磁碟區上停用加密。
     
     ```azurepowershell-interactive 
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```


### <a name="bkmk_RunningLinux"></a>透過範本在現有或執行中的 IaaS Linux VM 上啟用加密

您可以使用 [Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad)，在 Azure 中現有或執行中的 IaaS Linux VM 上啟用磁碟加密。

1. 按一下 Azure 快速入門範本上的 [部署至 Azure]。

2. 選取訂用帳戶、資源群組、資源群組位置、參數、法律條款及合約。 按一下 [建立]，以在現有或執行中的 IaaS VM 上啟用加密。

下表列出現有或執行中 VM 的 Resource Manager 範本參數︰

| 參數 | 說明 |
| --- | --- |
| vmName | 要執行加密作業的 VM 名稱。 |
| keyVaultName | 應上傳 BitLocker 金鑰的金鑰保存庫名稱。 您可以使用 cmdlet `(Get-AzureRmKeyVault -ResourceGroupName <MyResourceGroupName>). Vaultname` 或 Azure CLI 命令 `az keyvault list --resource-group "MySecureGroup" |Convertfrom-JSON` 來取得它|
| keyVaultResourceGroup | 包含金鑰保存庫的資源群組名稱|
|  keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰 URL。 如果您在 UseExistingKek 下拉式清單中選取 [nokek]，此參數是選擇性的。 如果您在 UseExistingKek 下拉式清單中選取 [kek]，您必須輸入 _keyEncryptionKeyURL_ 值。 |
| volumeType | 執行加密作業所在磁碟區的類型。 有效值為 _OS_、_Data_ 和 _All_。 
| forceUpdateTag | 每次需要強制執行作業時傳入唯一的值，例如 GUID。 |
| resizeOSDisk | 是否應該先將 OS 分割區調整大小以佔用完整的 OS VHD，然後才分割系統磁碟區。 |
| location | 所有資源的位置。 |



## <a name="encrypt-virtual-machine-scale-sets"></a>加密虛擬機器擴展集
[Azure 虛擬機器擴展集](../virtual-machine-scale-sets/overview.md)可讓您建立和管理一組負載平衡的相同 VM。 VM 執行個體的數目可以自動增加或減少，以因應需求或已定義的排程。 請使用 CLI 或 Azure PowerShell 來加密虛擬機器擴展集。

您可以在[這裡](https://github.com/Azure-Samples/azure-cli-samples/tree/master/disk-encryption/vmss)找到 Linux 擴展集資料磁碟加密的批次檔案範例。 這個範例會建立資源群組和 Linux 擴展集、掛接 5 GB 資料磁碟，並加密虛擬機器擴展集。

###  <a name="encrypt-virtual-machine-scale-sets-with-azure-cli"></a>使用 Azure CLI 加密虛擬機器擴展集
使用 [az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable) 在 Windows 虛擬機器擴展集上啟用加密。 如果您將擴展集上的升級原則設定為手動，請使用 [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances) 啟動加密。 

-  **加密執行中的虛擬機器擴展集**
    ```azurecli-interactive
     az vmss encryption enable --resource-group "MySecureRG" --name "MySecureVmss" --disk-encryption-keyvault "MySecureVault" 
    ```

-  **使用 KEK 包裝金鑰來加密執行中的虛擬機器擴展集**
    ```azurecli-interactive
     az vmss encryption enable --resource-group "MySecureRG" --name "MySecureVmss" --disk-encryption-keyvault "MySecureVault" --key-encryption-key "MyKEK" --key-encryption-keyvault "MySecureVault" 

     ```
- **取得虛擬機器擴展集的加密狀態：** 使用 [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show)

    ```azurecli-interactive
     az vmss encryption show --resource-group "MySecureRG" --name "MySecureVmss"
    ```

- **在虛擬機器擴展集上停用加密：** 使用 [az vmss encryption disable](/cli/azure/vmss/encryption#az-vmss-encryption-disable)
    ```azurecli-interactive
     az vmss encryption disable --resource-group "MySecureRG" --name "MySecureVmss"
    ```

###  <a name="encrypt-virtual-machine-scale-sets-with-azure-powershell"></a>使用 Azure PowerShell 加密虛擬機器擴展集
使用 [Set-AzureRmVmssDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmssdiskencryptionextension) Cmdlet 在 Windows 虛擬機器擴展集上啟用加密。

-  **加密執行中的虛擬機器擴展集**：
    ```powershell
     $rgName= "MySecureRg";
     $VmssName = "MySecureVmss";
     $KeyVaultName= "MySecureVault";
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgName;
     $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     Set-AzureRmVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;


-  **Encrypt a running virtual machine scale set using KEK to wrap the key**:
    ```powershell
     $rgName= "MySecureRg";
     $VmssName = "MySecureVmss";
     $KeyVaultName= "MySecureVault";
     $keyEncryptionKeyName = "MyKeyEncryptionKey";
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgName;
     $DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     Set-AzureRmVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
    ```

- **取得虛擬機器擴展集的加密狀態：** 使用 [Get-AzureRmVmssVMDiskEncryption](/powershell/module/azurerm.compute/get-azurermvmssvmdiskencryption) Cmdlet。
    
    ```powershell
    get-AzureRmVmssVMDiskEncryption -ResourceGroupName "MySecureRG" -VMScaleSetName "MySecureVmss"
    ```

- **在虛擬機器擴展集上停用加密：** 使用 [Disable-AzureRmVmssDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmssdiskencryption) Cmdlet。 

    ```powershell
    Disable-AzureRmVmssDiskEncryption -ResourceGroupName "MySecureRG" -VMScaleSetName "MySecureVmss"
    ```



## <a name="bkmk_EFA"> </a>將 EncryptFormatAll 功能使用於 Linux IaaS VM 上的資料磁碟
**EncryptFormatAll** 參數會減少加密 Linux 資料磁碟的時間。 符合特定準則的磁碟分割 (與其目前的檔案系統) 將會格式化。 接著會在命令執行前重新掛接回去。 如果想要排除符合準則的資料磁碟，您可以在執行命令前將它取消掛接。

 執行此命令之後，任何先前掛接的磁碟機都會格式化。 接著，加密層級將會在目前空的磁碟機上啟動。 選取此選項後，也會加密連結至 VM 的暫時資源磁碟。 如果重設暫時磁碟機，則 Azure 磁碟加密解決方案會為 VM 將它重新格式化並重新加密。

>[!WARNING]
> VM 的資料磁碟區有所需的資料時，不應該使用 EncryptFormatAll。 您可將磁碟取消掛接，將它們排除在加密之外。 您應該先在測試 VM 上試用 EncryptFormatAll，了解功能參數和其含意，然後在生產 VM 上試用。 EncryptFormatAll 選項會將資料磁碟格式化，而其上的所有資料都將遺失。 繼續之前，確認您想要排除的磁碟已正確卸載。 </br></br>
 >如果您在更新加密設定時設定此參數，則可能導致在實際加密前重新開機。 在此情況下，您也可以從 fstab 檔案中移除您不想格式化的磁碟。 同樣地，您應該先將想要加密格式化的磁碟分割新增至 fstab 檔案，在起始加密作業。 

### <a name="bkmk_EFACriteria"> </a> EncryptFormatAll 準則
此參數會通過所有磁碟分割，只要它們符合下列**所有**準則，就會予以加密： 
- 不是根/OS/開機磁碟分割
- 尚未加密
- 不是 BEK 磁碟區
- 已掛接

### <a name="bkmk_EFAPSH"> </a> 使用 EncryptFormatAll 參數搭配 Azure CLI
在 Azure 中使用 [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) 命令以在執行中的 IaaS 虛擬機器上啟用加密。

-  **使用 EncryptFormatAll 來加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --encrypt-format-all
     ```

### <a name="bkmk_EFAPSH"> </a> 使用 EncryptFormatAll 參數搭配 PowerShell cmdlet
使用 [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) cmdlet 搭配 [EncryptFormatAll 參數](https://www.powershellgallery.com/packages/AzureRM/5.0.0)。 

**使用 EncryptFormatAll 來加密執行中的 VM：** 例如，下面指令碼會初始化您的變數並執行 Set-AzureRmVMDiskEncryptionExtension cmdlet 搭配 EncryptFormatAll 參數。 資源群組、VM 和金鑰保存庫應該已經建立為必要條件。 使用您的值取代 MySecureRg、MySecureVM 和 MySecureVault。
  
   ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $vmName = 'MySecureVM';
     $KeyVaultName = 'MySecureVault';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
      
     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -EncryptFormatAll
   ```


### <a name="bkmk_EFALVM"> </a> 使用 EncryptFormatAll 參數搭配邏輯磁碟區管理員 (LVM) 
我們建議進行 LVM-on-crypt 設定。 在下列所有範例中，以適合您的使用案例的項目取代裝置路徑和掛接點。 以下步驟可以完成此設定：

- 新增將構成 VM 的資料磁碟。
- 格式化、掛接這些磁碟，並將其新增至 fstab 檔案。

    1. 格式化剛新增的磁碟。 我們會 Azure 在此產生的符號連結。 使用符號連結，可避免裝置名稱變更的相關問題。 如需詳細資訊，請參閱[針對裝置名稱問題進行疑難排解](../virtual-machines/linux/troubleshoot-device-names-problems.md)一文。
    
         `mkfs -t ext4 /dev/disk/azure/scsi1/lun0`
    
    2. 掛接磁碟。
         
         `mount /dev/disk/azure/scsi1/lun0 /mnt/mountpoint`
    
    3. 新增至 fstab。
         
        `echo "/dev/disk/azure/scsi1/lun0 /mnt/mountpoint ext4 defaults,nofail 1 2" >> /etc/fstab`
    
    4. 執行 Set-AzureRmVMDiskEncryptionExtension PowerShell cmdlet 搭配 -EncryptFormatAll to 來加密這些磁碟。
         ```azurepowershell-interactive
         Set-AzureRmVMDiskEncryptionExtension -ResouceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl "https://mykeyvault.vault.azure.net/" -EncryptFormatAll
         ```
    5. 在這些新的磁碟上設定 LVM。 請注意，VM 完成開機之後，加密磁碟機就會解除鎖定。 因此，LVM 掛接後續也必須延遲。


## <a name="bkmk_VHDpre"> </a>透過客戶加密的 VHD 和加密金鑰建立的新 IaaS VM
在這個案例中，您可以使用 PowerShell Cmdlet 或 CLI 命令啟用加密。 

使用附錄中的指示來準備可用於 Azure 的預先加密映像。 映像建立之後，您可以使用下一節的步驟來建立加密的 Azure VM。

* [準備已預先加密的 Windows VHD](azure-security-disk-encryption-appendix.md#bkmk_preWin)
* [準備已預先加密的 Linux VHD](azure-security-disk-encryption-appendix.md#bkmk_preLinux)

>[!IMPORTANT]
 >您必須在啟用 Azure 磁碟加密之前，先在 Azure 磁碟加密之外對以受控磁碟為基礎的 VM 執行個體建立快照集和/或備份。 您可以從入口網站建立受控磁碟的快照集，也可以使用 [Azure 備份](../backup/backup-azure-vms-encryption.md)來建立。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 在建立備份後，您就可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 在建立好備份並指定了這個參數之前，對以受控磁碟為基礎的 VM 執行 Set-AzureRmVMDiskEncryptionExtension 命令都將會失敗。 
>
>加密或停用加密功能，可能會導致 VM 重新啟動。 



### <a name="bkmk_VHDprePSH"> </a> 使用 Azure PowerShell 來加密具有預先加密 VHD 的 IaaS VM 
您可以使用 PowerShell Cmdlet [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk#examples) 在加密的 VHD 上啟用磁碟加密。 下列範例會提供一些常見的參數。 

```azurepowershell-interactive
$VirtualMachine = New-AzureRmVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzureRmVM -VM $VirtualMachine -ResouceGroupName "MySecureRG"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>在新增的資料磁碟上啟用加密
您可以使用 [az vm disk attach](../virtual-machines/linux/add-disk.md) 或[透過 Azure 入口網站](../virtual-machines/linux/attach-disk-portal.md)，新增磁碟新增。 您必須先掛接新連結的資料磁碟，才可以加密。 您必須要求資料磁碟機加密，因為在進行加密時，將無法使用該磁碟機。 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>透過 Azure CLI 在新增的磁碟上啟用加密
 當您執行命令來啟用加密時，Azure CLI 命令會自動為您提供新的序列版本。 
-  **加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **使用 KEK 加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>透過 Azure PowerShell 在新增的磁碟上啟用加密
 使用 Powershell 來加密 Linux 的新磁碟時，必須指定新的序列版本。 序列版本必須是唯一的。 下列指令碼會產生序列版本的 GUID。 
 

-  **加密執行中的 VM：** 下面指令碼會初始化您的變數並執行 Set-AzureRmVMDiskEncryptionExtension cmdlet。 資源群組、VM 和金鑰保存庫應該已經建立為必要條件。 使用您的值取代 MySecureRg、MySecureVM 和 MySecureVault。 -VolumeType 參數會設定為資料磁碟，而非作業系統磁碟。 

     ```azurepowershell-interactive
      $sequenceVersion = [Guid]::NewGuid();
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
    ```
- **使用 KEK 加密執行中的 VM：** 如果您要加密資料磁碟而非作業系統磁碟，則可能需要新增 -VolumeType 參數。 

     ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```

    >[!NOTE]
    > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 


## <a name="disable-encryption-for-linux-vms"></a>停用 Linux VM 的加密
您可以使用 Azure PowerShell、Azure CLI 或 Resource Manager 範本來停用加密。 

>[!IMPORTANT]
>只有資料磁碟區支援在 Linux VM 上透過 Azure 磁碟加密停用加密。 如果 OS 磁碟區已加密，則不支援在資料或 OS 磁碟區上停用加密。  

- **透過 Azure PowerShell 停用磁碟加密：** 若要停用加密，請使用 [Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet。 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM' [--volume-type {ALL, DATA, OS}]
     ```

- **透過 Azure CLI 停用加密：** 若要停用加密，請使用 [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) 命令。 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
- **透過 Resource Manager 範本停用加密：** 使用[在執行中的 Linux VM 上停用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad)範本來停用加密。
     1. 按一下 [ **部署至 Azure**]。
     2. 選取訂用帳戶、資源群組、位置、VM、法律條款及合約。
     3.  按一下 [購買] 以在執行中的 Windows VM 上停用磁碟加密。 


## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [啟用適用於 Windows 的 Azure 磁碟加密](azure-security-disk-encryption-windows.md)
