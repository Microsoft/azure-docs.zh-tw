---
title: 以適用於 Windows IaaS VM 的 Azure AD App (舊版) 啟用 Azure 磁碟加密 | Microsoft Docs
description: 本文提供啟用 Windows IaaS VM 適用的 Microsoft Azure 磁碟加密的指示。
services: security
documentationcenter: na
author: mestew
manager: MBaldwin
ms.assetid: 30b1151b-3314-4526-ba4e-4f6b23b1b280
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/24/2018
ms.author: mstewart
ms.openlocfilehash: 2ecc177f810c9a3fe2d096b71aed4886cf89d7b4
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42889154"
---
#  <a name="enable-azure-disk-encryption-for-windows-iaas-vms-previous-release"></a>啟用適用於 Windows IaaS VM 的 Azure 磁碟加密 (舊版)

**新版的 Azure 磁碟加密不需要提供 Azure AD 應用程式參數，即可啟用 VM 磁碟加密。若使用新版本，您就不再需要在啟用加密步驟期間提供 Azure AD 認證。若使用新版本，所有新的 VM 必須經過加密，而不需要 Azure AD 應用程式參數。若要檢視使用新版本來啟用 VM 磁碟加密的指示，請參閱[適用於 Windows VM 的 Azure 磁碟加密](azure-security-disk-encryption-windows.md)。已經使用 Azure AD 應用程式參數進行加密的 VM 仍然受支援，應該繼續使用 AAD 語法進行維護。**


您可以啟用許多磁碟加密案例，步驟可能會因案例而有所不同。 下列各節涵蓋更詳細的 Windows IaaS VM 案例。 您必須先完成 [Azure 磁碟加密必要條件](../security/azure-security-disk-encryption-prerequisites-aad.md)，才能您使用磁碟加密。 

在磁碟加密前製作[快照集](../virtual-machines/windows/snapshot-copy-managed-disk.md)及/或進行備份。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 具有受控磁碟的 VM 需要有備份，才能進行加密。 在建立備份後，您可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 如需如何備份和還原已加密 VM 的詳細資訊，請參閱 [Azure 備份](../backup/backup-azure-vms-encryption.md)一文。 

>[!WARNING]
>為了確保加密祕密不會跨出區域界限，Azure 磁碟加密需要讓 Key Vault 和 VM 共置於相同區域中。 請在和所要加密 VM 相同的區域中建立並使用 Key Vault。 


## <a name="enable-encryption-on-new-iaas-vms-created-from-the-marketplace"></a>在透過 Marketplace 建立的新 IaaS VM 上啟用加密
您可以使用 Resource Manager 範本，透過 Azure 中的 Marketplace 在新 IaaS Windows VM 上啟用磁碟加密。 此範本會使用 Windows Server 2012 的資源庫映像來建立新的加密 Windows VM。

1. 在 [Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image)上，按一下 [部署至 Azure]。

2. 選取訂用帳戶、資源群組、資源群組位置、參數、法律條款及協議。 按一下 [採購] 來部署已啟用加密的新 IaaS VM。

3. 您在部署範本之後，請使用您慣用的方法來確認 VM 加密狀態：
     - 使用 [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) 命令，透過 Azure CLI 驗證。 

         ```azurecli-interactive 
         az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
         ```

     - 使用 [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) cmdlet，透過 Azure PowerShell 驗證。 

         ```azurepowershell-interactive
         Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName 'MySecureRg' -VMName 'MySecureVM'
         ```

     -  選取 VM，然後按一下 [設定] 標題之下的 [磁碟]，以在入口網站中驗證加密狀態。 在圖中的 [加密] 之下，您會看到它是否已啟用。 
           ![Azure 入口網站 - 已啟用磁碟加密](./media/azure-security-disk-encryption/disk-encryption-fig2.png) 針對透過 Marketplace 案例使用 Azure AD 用戶端識別碼的新 VM，以下資料表列出其 Resource Manager 範本參數︰

| 參數 | 說明 | 
| --- | --- |
| adminUserName | 虛擬機器的系統管理使用者名稱。 |
| adminPassword | 虛擬機器的系統管理使用者密碼。 |
| newStorageAccountName | 要儲存作業系統和資料 VHD 的儲存體帳戶名稱。 |
| vmSize | VM 的大小。 目前僅支援標準的 A、D 和 G 系列。 |
| virtualNetworkName | VM NIC 應該隸屬的 VNet 名稱。 |
| subnetName | VM NIC 應該隸屬的 VNet 中子網路的名稱。 |
| AADClientID | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端識別碼。 |
| AADClientSecret | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端密碼。 |
| keyVaultURL | 應上傳 BitLocker 金鑰的金鑰保存庫 URL。 您可以使用 Cmdlet `(Get-AzureRmKeyVault -VaultName,-ResourceGroupName).VaultURI` 或 Azure CLI `az keyvault show --name "MySecureVault" --query properties.vaultUri` 來取得它 |
| keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰 URL (選擇性)。 </br> </br>KeyEncryptionKeyURL 是選擇性參數。 您可以使用自己的 KEK，在金鑰保存庫中進一步保護資料加密金鑰 (複雜密碼)。 |
| keyVaultResourceGroup | 金鑰保存庫的資源群組。 |
| vmName | 要執行加密作業所在 VM 的名稱。 |


## <a name="bkmk_RunningWinVM"></a> 在現有或執行中的 IaaS Windows VM 上啟用加密
在這個案例中，您可以使用範本、PowerShell Cmdlet 或 CLI 命令啟用加密。 下列各節將更加詳細地說明如何啟用 Azure 磁碟加密。 

>[!IMPORTANT]
 >您必須在啟用 Azure 磁碟加密之前，先在 Azure 磁碟加密之外對以受控磁碟為基礎的 VM 執行個體建立快照集和/或備份。 您可以從入口網站建立受控磁碟的快照集，也可以使用 [Azure 備份](../backup/backup-azure-vms-encryption.md)來建立。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 在建立備份後，您就可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 在建立好備份並指定了這個參數之前，對以受控磁碟為基礎的 VM 執行 Set-AzureRmVMDiskEncryptionExtension 命令都將會失敗。 
>
>加密或停用加密功能，可能會導致 VM 重新啟動。 
>

### <a name="bkmk_RunningWinVMPSH"></a>透過 Azure PowerShell 在現有或執行中的 VM 上啟用加密 
在 Azure 中使用 [Set-AzureRmVMDiskEncryptionExtension](/powershell/module/azurerm.compute/set-azurermvmdiskencryptionextension) cmdlet 以在執行中的 IaaS 虛擬機器上啟用加密。 如需使用 PowerShell Cmdlet 以 Azure 磁碟加密來啟用加密的相關資訊，請參閱部落格文章[探索使用 Azure PowerShell 的 Azure 磁碟加密 - 第 1 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/17/explore-azure-disk-encryption-with-azure-powershell.aspx)和[探索使用 Azure PowerShell 的 Azure 磁碟加密 - 第 2 部分](http://blogs.msdn.com/b/azuresecurity/archive/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2.aspx)。

-  **使用用戶端祕密來加密執行中的 VM：** 下面指令碼會初始化您的變數並執行 Set-AzureRmVMDiskEncryptionExtension cmdlet。 資源群組、VM、金鑰保存庫、AAD 應用程式和用戶端祕密應該已經建立為必要條件。 使用您的值取代 MySecureRg、MySecureVM、MySecureVault、My-AAD-client-ID 和 My-AAD-client-secret。
     ```azurepowershell-interactive
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **使用 KEK 來加密執行中的 VM 以包裝用戶端祕密：** Azure 磁碟加密可讓您指定您金鑰保存庫中的現有金鑰，以包裝在啟用加密時所產生的磁碟加密祕密。 若指定了金鑰加密金鑰，Azure 磁碟加密會先使用該金鑰包裝加密祕密，再寫入 Key Vault。 

     ```azurepowershell-interactive
     $rgName = 'MySecureRg';
     $vmName = ‘MyExtraSecureVM’;
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **確認磁碟已加密：** 若要檢查 IaaS VM 的加密狀態，請使用 [Get-AzureRmVmDiskEncryptionStatus](/powershell/module/azurerm.compute/get-azurermvmdiskencryptionstatus) cmdlet。 
     ```azurepowershell-interactive
     Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName 'MySecureRg' -VMName 'MySecureVM'
     ```
    
- **停用磁碟加密：** 若要停用加密，請使用 [Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet。 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```

### <a name="bkmk_RunningWinVMCLI"></a>透過 Azure CLI 在現有或執行中的 VM 上啟用加密
在 Azure 中使用 [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable) 命令以在執行中的 IaaS 虛擬機器上啟用加密。

-  **使用用戶端祕密來加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **使用 KEK 來加密執行中的 VM 以包裝用戶端祕密：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] </br> key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

- **確認磁碟已加密：** 若要檢查 IaaS VM 的加密狀態，請使用 [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show) 命令。 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
     ```

- **停用加密：** 若要停用加密，請使用 [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) 命令。 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
 
 > [!NOTE]
 >您必須在啟用 Azure 磁碟加密之前，先在 Azure 磁碟加密之外對以受控磁碟為基礎的 VM 執行個體建立快照集和/或備份。 您可以從入口網站建立受控磁碟的快照集，也可以使用 [Azure 備份](../backup/backup-azure-vms-encryption.md)來建立。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 在建立備份後，您就可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 在建立好備份並指定了這個參數之前，對以受控磁碟為基礎的 VM 執行這個命令都將會失敗。 
>
>加密或停用加密功能，可能會導致 VM 重新啟動。 

### <a name="bkmk_RunningWinVMwRM"></a>使用 Resource Manager 範本
您可以使用 [Resource Manager 範本來加密執行中的 Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)，以在 Azure 中現有或執行中的 IaaS Windows VM 上啟用磁碟加密。


1. 在 Azure 快速入門範本中，按一下 [部署至 Azure]。

2. 選取訂用帳戶、資源群組、資源群組位置、參數、法律條款及協議。 按一下 [購買]，以在現有或執行中的 IaaS VM 上啟用加密。

針對使用 Azure AD 用戶端識別碼的現有或執行中 VM，以下資料表列出其 Resource Manager 範本參數︰

| 參數 | 說明 |
| --- | --- |
| AADClientID | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端識別碼。 |
| AADClientSecret | 具有權限可將密碼寫入金鑰保存庫之 Azure AD 應用程式的用戶端密碼。 |
| keyVaultName | 應上傳 BitLocker 金鑰的金鑰保存庫名稱。 您可以使用 cmdlet `(Get-AzureRmKeyVault -ResourceGroupName <MyResourceGroupName>). Vaultname` 或 Azure CLI 命令 `az keyvault list --resource-group "MySecureGroup" |Convertfrom-JSON` 來取得它|
|  keyEncryptionKeyURL | 用來加密所產生 BitLocker 金鑰的金鑰加密金鑰 URL。 如果您在 UseExistingKek 下拉式清單中選取 [nokek]，此參數是選擇性的。 如果您在 UseExistingKek 下拉式清單中選取 [kek]，您必須輸入 _keyEncryptionKeyURL_ 值。 |
| volumeType | 執行加密作業所在磁碟區的類型。 有效值為 _OS_、_Data_ 和 _All_。 |
| sequenceVersion | BitLocker 作業的順序版本。 每當在相同的 VM 上執行磁碟加密作業時便遞增此版本號碼。 |
| vmName | 要執行加密作業所在 VM 的名稱。 |


## <a name="bkmk_VHDpre"> </a>透過客戶加密的 VHD 和加密金鑰建立的新 IaaS VM
在這個案例中，您可以使用 Resource Manager 範本、PowerShell Cmdlet 或 CLI 命令啟用加密。 下列各節將更加詳細地說明 Resource Manager 範本和 CLI 命令。 

使用附錄中的指示來準備可用於 Azure 的預先加密映像。 映像建立之後，您可以使用下一節的步驟來建立加密的 Azure VM。

* [準備已預先加密的 Windows VHD](azure-security-disk-encryption-appendix.md#bkmk_preWin)
* [準備已預先加密的 Linux VHD](azure-security-disk-encryption-appendix.md#bkmk_preLinux)

>[!IMPORTANT]
 >您必須在啟用 Azure 磁碟加密之前，先在 Azure 磁碟加密之外對以受控磁碟為基礎的 VM 執行個體建立快照集和/或備份。 您可以從入口網站建立受控磁碟的快照集，也可以使用 [Azure 備份](../backup/backup-azure-vms-encryption.md)來建立。 擁有備份可確保在加密期間發生任何非預期的失敗時，能有復原選項可供選擇。 在建立備份後，您就可以使用 Set-AzureRmVMDiskEncryptionExtension Cmdlet 並指定 -skipVmBackup 參數來加密受控磁碟。 在建立好備份並指定了這個參數之前，對以受控磁碟為基礎的 VM 執行 Set-AzureRmVMDiskEncryptionExtension 命令都將會失敗。 
>
>加密或停用加密功能，可能會導致 VM 重新啟動。 



### <a name="bkmk_VHDprePSH"> </a> 透過 Azure PowerShell 加密包含預先加密 VHD 的 VM
您可以使用 PowerShell Cmdlet [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk#examples) 在加密的 VHD 上啟用磁碟加密。 下列範例會提供一些常見的參數。 

```powershell
$VirtualMachine = New-AzureRmVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzureRmVM -VM $VirtualMachine -ResouceGroupName "MySecureRG"
```


### <a name="bkmk_VHDpreRM"> </a>透過 Resource Manager 範本將具有預先加密 VHD 的 IaaS VM 加密 
您可以使用 [Resource Manager 範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-pre-encrypted-vm)在加密的 VHD 上啟用磁碟加密。

1. 在 Azure 快速入門範本中，按一下 [部署至 Azure]。

2. 選取訂用帳戶、資源群組、資源群組位置、參數、法律條款及合約。 按一下 [建立] 以在新 IaaS VM 上啟用加密。

以下資料表列出加密 VHD 的 Resource Manager 範本參數︰

| 參數 | 說明 |
| --- | --- |
| newStorageAccountName | 要儲存加密的作業系統 VHD 的儲存體帳戶名稱。 此儲存體帳戶應該已建立在與 VM 相同的資源群組和相同的位置中。 |
| osVhdUri | 來自儲存體帳戶作業系統 VHD 的 URI。 |
| osType | 作業系統產品類型 (Windows/Linux)。 |
| virtualNetworkName | VM NIC 應該隸屬的 VNet 名稱。 名稱應該已建立在與 VM 相同的資源群組和相同的位置中。 |
| subnetName | VM NIC 應該隸屬的 VNet 上子網路的名稱。 |
| vmSize | VM 的大小。 目前僅支援標準的 A、D 和 G 系列。 |
| keyVaultResourceID | 在 Azure Resource Manager 中識別金鑰保存庫資源的 ResourceID。 您可以使用 PowerShell Cmdlet `(Get-AzureRmKeyVault -VaultName &lt;MyKeyVaultName&gt; -ResourceGroupName &lt;MyResourceGroupName&gt;).ResourceId` 或使用 Azure CLI 命令 `az keyvault show --name "MySecureVault" --query id` 來取得它|
| keyVaultSecretUrl | 在金鑰保存庫中所設定的磁碟加密金鑰的 URL。 |
| keyVaultKekUrl | 用來加密所產生磁碟加密金鑰的金鑰加密金鑰的 URL。 |
| vmName | IaaS VM 的名稱。 |

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>在新增的資料磁碟上啟用加密
您可以使用 PowerShell 或[透過 Azure 入口網站](../virtual-machines/windows/attach-managed-disk-portal.md)，[將新的磁碟新增至 Windows VM](../virtual-machines/windows/attach-disk-ps.md)。 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>透過 Azure PowerShell 在新增的磁碟上啟用加密
 使用 Powershell 來加密 Windows VM 的新磁碟時，應指定新的序列版本。 序列版本必須是唯一的。 下列指令碼會產生序列版本的 GUID。 在某些情況下，Azure 磁碟加密擴充功能可能會自動加密新增的資料磁碟。 如果發生這種情況，建議使用新的序列版本再次執行 Set-AzureRmVmDiskEncryptionExtension cmdlet。
 

-  **使用用戶端祕密來加密執行中的 VM：** 下面指令碼會初始化您的變數並執行 Set-AzureRmVMDiskEncryptionExtension cmdlet。 資源群組、VM、金鑰保存庫、AAD 應用程式和用戶端祕密應該已經建立為必要條件。 使用您的值取代 MySecureRg、MySecureVM、MySecureVault、My-AAD-client-ID 和 My-AAD-client-secret。 -VolumeType 參數會設定為資料磁碟，而非作業系統磁碟。 

     ```azurepowershell-interactive
      $sequenceVersion = [Guid]::NewGuid();
      $rgName = 'MySecureRg';
      $vmName = 'MySecureVM';
      $aadClientID = 'My-AAD-client-ID';
      $aadClientSecret = 'My-AAD-client-secret';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId  –SequenceVersion $sequenceVersion;
    ```
- **使用 KEK 來加密執行中的 VM 以包裝用戶端祕密：** Azure 磁碟加密可讓您指定您金鑰保存庫中的現有金鑰，以包裝在啟用加密時所產生的磁碟加密祕密。 若指定了金鑰加密金鑰，Azure 磁碟加密會先使用該金鑰包裝加密祕密，再寫入 Key Vault。 如果您要加密資料磁碟而非作業系統磁碟，則可能需要新增 -VolumeType 參數。 

     ```azurepowershell-interactive
     $sequenceVersion = [Guid]::NewGuid();
     $rgName = 'MySecureRg';
     $vmName = 'MyExtraSecureVM';
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > disk-encryption-keyvault 參數值的語法為完整識別碼字串：/subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]</br> key-encryption-key 參數值的語法為 KEK 的完整 URI： https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id] 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>透過 Azure CLI 在新增的磁碟上啟用加密
 當您執行命令來啟用加密時，Azure CLI 命令會自動為您提供新的序列版本。 
-  **使用用戶端祕密來加密執行中的 VM：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **使用 KEK 來加密執行中的 VM 以包裝用戶端祕密：**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault"--key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```


## <a name="enable-encryption-using-azure-ad-client-certificate-based-authentication"></a>使用 Azure AD 用戶端憑證式驗證來啟用加密。
您可以使用用戶端憑證驗證 (不管是否搭配 KEK)。 指令碼需要完成 [Azure 磁碟加密必要條件](azure-security-disk-encryption-prerequisites-aad.md)。 使用 PowerShell 指令碼之前，您應該已經有上傳至金鑰保存庫並部署到 VM 的憑證。 如果您也使用 KEK，KEK 應已存在。 如需詳細資訊，請參閱先決條件一文的 [Azure AD 的憑證式驗證](azure-security-disk-encryption-prerequisites-aad.md#bkmk_Cert)一節。


### <a name="enable-encryption-using-certificate-based-authentication-with-azure-powershell"></a>透過 Azure PowerShell 使用憑證式驗證來啟用加密

```powershell
## Fill in 'MySecureRg', 'My-AAD-client-ID', 'MySecureVault, and ‘MySecureVM’.

$rgName = 'MySecureRg';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = ‘MySecureVM’;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

# Fill in the certificate path and the password so the thumbprint can be set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId
```
  
### <a name="enable-encryption-using-certificate-based-authentication-and-a-kek-with-azure-powershell"></a>透過 Azure PowerShell 使用憑證式驗證和 KEK 來啟用加密

```powershell
# Fill in 'MySecureRg', 'My-AAD-client-ID', 'MySecureVault,, 'MySecureVM’, and "KEKName.

$rgName = 'MySecureRg';
$AADClientID ='My-AAD-client-ID';
$KeyVaultName = 'MySecureVault';
$VMName = 'MySecureVM';
$keyEncryptionKeyName ='KEKName';
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

## Fill in the certificate path and the password so the thumbprint can be read and set as a variable. 

$certPath = '$CertPath = "C:\certificates\mycert.pfx';
$CertPassword ='Password'
$Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($CertPath, $CertPassword)
$aadClientCertThumbprint = $cert.Thumbprint;

# Enable disk encryption using the client certificate thumbprint and a KEK

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $VMName -AadClientID $AADClientID -AadClientCertThumbprint $AADClientCertThumbprint -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId
```

## <a name="disable-encryption"></a>停用加密
您可以使用 Azure PowerShell、Azure CLI 或 Resource Manager 範本來停用加密。 

- **透過 Azure PowerShell 停用磁碟加密：** 若要停用加密，請使用 [Disable-AzureRmVMDiskEncryption](/powershell/module/azurerm.compute/disable-azurermvmdiskencryption) cmdlet。 
     ```azurepowershell-interactive
     Disable-AzureRmVMDiskEncryption -ResourceGroupName 'MySecureRG' -VMName 'MySecureVM'
     ```

- **透過 Azure CLI 停用加密：** 若要停用加密，請使用 [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable) 命令。 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
     ```
- **透過 Resource Manager 範本停用加密：** 

    1. 從[在執行中的 Windows VM 上停用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm)範本，按一下 [部署至 Azure]。
    2. 選取訂用帳戶、資源群組、位置、VM、法律條款及協議。
    3.  按一下 [購買] 以在執行中的 Windows VM 上停用磁碟加密。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [啟用適用於 Linux 的 Azure 磁碟加密](azure-security-disk-encryption-linux-aad.md)
