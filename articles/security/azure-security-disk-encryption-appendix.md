---
title: Windows 和 Linux IaaS VM 適用的 Azure 磁碟加密 | Microsoft Docs
description: 本文是 Windows 和 Linux IaaS VM 適用的 Microsoft Azure 磁碟加密附錄。
services: security
documentationcenter: na
author: mestew
manager: MBaldwin
ms.assetid: 98bbcb84-8e6c-4eb2-8490-c2a0c67aad79
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/24/2018
ms.author: mstewart
ms.openlocfilehash: 9efd8730af292e6f720c3bacd5707c48f0eab7ac
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42887928"
---
# <a name="appendix-for-azure-disk-encryption"></a>Azure 磁碟加密的附錄 
本文是[IaaS VM 適用的 Azure 磁碟加密](azure-security-disk-encryption-overview.md)附錄。 請務必先閱讀 IaaS VM 適用的 Azure 磁碟加密文章，以了解內容。 本文說明如何準備預先加密的 VHD 和其他工作。

## <a name="connect-to-your-subscription"></a>連線至您的訂用帳戶
在開始之前，請檢閱[必要條件](azure-security-disk-encryption-prerequisites.md)一文。 符合所有必要條件之後，請執行下列 Cmdlet 來連線到您的訂用帳戶︰

### <a name="bkmk_ConnectPSH"> 使用 PowerShell</a> 連線到您的訂用帳戶

1. 開始 Azure PowerShell 工作階段，並使用下列命令登入您的 Azure 帳戶：

     ```powershell
     Connect-AzureRmAccount 
     ```
2. 如果您有多個訂用帳戶，並想要指定要使用的其中一個，請輸入下列內容以查看您帳戶的訂用帳戶：
     
     ```powershell
     Get-AzureRmSubscription
     ```
3. 若要指定要使用的訂用帳戶，請輸入：
 
     ```powershell
      Select-AzureRmSubscription -SubscriptionName <Yoursubscriptionname>
     ```
4. 若要確認訂用帳戶設定正確無誤，請輸入：
     
     ```powershell
     Get-AzureRmSubscription
     ```
5. 如有需要，使用 [Connect-AzureAD](/powershell/module/azuread/connect-azuread) 連線至 Azure AD。
     
     ```powershell
     Connect-AzureAD
     ```
6. 若要確認已安裝 Azure 磁碟加密 Cmdlet，請輸入：
     
     ```powershell
     Get-command *diskencryption*
     ```
                       
7. 如有需要，檢閱[開始使用 Azure PowerShell](/powershell/azure/get-started-azureps) 和 [AzureAD](/powershell/module/azuread)。

### <a name="bkmk_ConnectCLI"></a> 使用 Azure CLI 連線至您的 Azure 訂用帳戶

1. 使用 [az login](/cli/azure/authenticate-azure-cli#interactive-log-in) 登入 Azure。 
     
     ```azurecli
     az login
     ```

2. 如果您想要選取用於登入的租用戶，請使用：
    
     ```azurecli
     az login --tenant <tenant>
     ```

3. 如果您有多個訂用帳戶並想要指定特定訂用帳戶，請使用 [az account list](/cli/azure/account#az-account-list) 取得訂用帳戶清單並以 [az account set](/cli/azure/account#az-account-set) 進行指定。
     
     ```azurecli
     az account list
     az account set --subscription "<subscription name or ID>"
     ```

4. 確認已安裝的版本。
     
     ```azurecli
        az --version
     ``` 

5. 如有需要，檢閱[開始使用 Azure CLI 2.0](/cli/azure/get-started-with-azure-cli)。 

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Azure 磁碟加密的範例 PowerShell 指令碼 

- **列出訂用帳戶中的所有已加密 VM**

     ```azurepowershell-interactive
     $osVolEncrypted = {(Get-AzureRmVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
     $dataVolEncrypted= {(Get-AzureRmVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
     Get-AzureRmVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
     ```

- **列出所有用來加密金鑰保存庫中 VM 的磁碟加密祕密** 

     ```azurepowershell-interactive
     Get-AzureKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
     ```

### <a name="bkmk_prereq-script"></a> 使用 Azure 磁碟加密先決條件 PowerShell 指令碼
如果您已經熟悉 Azure 磁碟加密的必要條件，您可以使用 [Azure 磁碟加密必要條件 PowerShell 指令碼](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/ResourceManager/Compute/Commands.Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 )。 如需使用此 PowerShell 指令碼的範例，請參閱[加密 VM 快速入門](quick-encrypt-vm-powershell.md)。 您可以從指令碼區段 (起自 211 行) 中移除註解，以對現有資源群組中現有 VM 的所有磁碟加密。 

下表顯示可在 PowerShell 指令碼中使用的參數： 


|參數|說明|屬於必要項目|
|------|------|------|
|$resourceGroupName| 金鑰保存庫所屬資源群組的名稱。  如果不存在此名稱的應用程式，將會以此名稱建立新的資源群組。| True|
|$keyVaultName|要用來放置加密金鑰的金鑰保存庫名稱。 如果不存在此名稱的應用程式，將會以此名稱建立新的保存庫。| True|
|$location|金鑰保存庫的位置。 請確定金鑰保存庫和要加密的 VM 位於相同位置。 使用 `Get-AzureRMLocation` 取得位置清單。|True|
|$subscriptionId|要使用的 Azure 訂用帳戶識別碼。  您可以使用 `Get-AzureRMSubscription` 取得您的訂用帳戶識別碼。|True|
|$aadAppName|會用來將祕密寫入到金鑰保存庫的 Azure AD 應用程式名稱。 如果不存在此名稱的應用程式，將會以此名稱建立新的應用程式。 如果此應用程式已經存在，請將 aadClientSecret 參數傳遞至指令碼。|False|
|$aadClientSecret|稍早建立的 Azure AD 應用程式用戶端密碼。|False|
|$keyEncryptionKeyName|金鑰保存庫中選用金鑰加密金鑰的名稱。 如果不存在此名稱的應用程式，將會以此名稱建立新的金鑰。|False|


## <a name="resource-manager-templates"></a>Resource Manager 範本

<!--   - [Create a key vault](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create) -->

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>加密或解密沒有 Azure AD 應用程式的 VM


- [在現有或執行中的 IaaS Windows VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad)
- [在現有或執行中的 IaaS Windows VM 上停用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm-without-aad)
- [在現有或執行中的 IaaS Linux VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad)  
 -  [在執行中 Linux VM 上停用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) 
    - 只能在 Linux VM 的資料磁碟區上停用加密。  

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>加密或解密具有 Azure AD 應用程式 (舊版) 的 VM 
 
- [在現有或執行中的 IaaS Windows VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm)

- [在現有或執行中的 IaaS Linux VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)    

- [在執行中 Windows IaaS 上停用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-windows-vm) 

-  [在執行中 Linux VM 上停用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm) 
    - 只能在 Linux VM 的資料磁碟區上停用加密。 

- [在來自 Marketplace 的新 IaaS Windows VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image)
    - 此範本會建立使用 Windows Server 2012 資源庫映像的新加密 Windows VM。

- [從資源庫映像建立新加密的 Windows IaaS 受控磁碟虛擬機器](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-new-vm-gallery-image-managed-disks)
    - 此範本會使用 Windows Server 2012 資源庫映像，建立具有受控磁碟的新加密 Windows VM。

- [搭配使用完整磁碟加密與受控磁碟來部署 RHEL 7.2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-full-disk-encrypted-rhel)
    - 此範本會使用受控磁碟在 Azure 中建立已完整加密的 RHEL 7.2 VM。 其內含加密的 30 GB OS 磁碟機和加密的 200 GB 陣列 (RAID-0)，並掛接於 /mnt/raidencrypted。 請參閱[常見問題集](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport)一文中支援的 Linux 伺服器散發套件。 

- [搭配使用完整磁碟加密與非受控磁碟來部署 RHEL 7.2](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-full-disk-encrypted-rhel-unmanaged)
    - 此範本會在 Azure 中建立已完整加密的 RHEL 7.2 VM，此 VM 內含加密的 30 GB OS 磁碟機和加密的 200 GB 陣列 (RAID-0)，並掛接於 /mnt/raidencrypted。 請參閱[常見問題集](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport)一文中支援的 Linux 伺服器散發套件。 

- [在 Windows 或 Linux 預先加密的 VHD 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-create-pre-encrypted-vm)

- [從預先加密的 VHD/儲存體 Blob 建立新加密的受控磁碟](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - 建立已加密的新受控磁碟，新的受控磁碟具備預先加密的 VHD 以及對應的加密設定

- [使用 Azure AD 用戶端憑證指紋在執行中的 Windows VM 上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-aad-client-cert)
    
- [在執行中的 Linux 虛擬機器擴展集上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-vmss-linux)

- [在執行中的 Windows 虛擬機器擴展集上啟用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-vmss-windows)

 - [使用 Jumpbox 部署 Linux VM 的 VM 擴展集並在 Linux VMSS 上啟用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-linux-jumpbox)

 - [使用 Jumpbox 部署 Windows VM 的 VM 擴展集並在 Windows VMSS 上啟用加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-vmss-windows-jumpbox)

- [在執行中的 Linux 虛擬機器擴展集上停用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-vmss-linux)

- [在執行中的 Windows 虛擬機器擴展集上停用磁碟加密](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-vmss-windows)

## <a name="bkmk_preWin"></a>準備已預先加密的 Windows VHD
下列各節是準備預先加密的 Windows VHD 以在 Azure IaaS 中部署為加密的 VHD 的必要項目。 使用該資訊以在 Azure Site Recovery 或 Azure 上準備並啟動全新的 Windows VM (VHD)。 如需有關如何準備和上傳 VHD 的詳細資訊，請參閱[上傳一般化 VHD 並使用它在 Azure 中建立新的 VM](../virtual-machines/windows/upload-generalized-managed.md)。

### <a name="update-group-policy-to-allow-non-tpm-for-os-protection"></a>更新群組原則以對作業系統保護允許非 TPM
設定 **BitLocker 磁碟機加密**的 BitLocker 群組原則設定，其位於此路徑**本機電腦原則** > **電腦設定** > **系統管理範本** > **Windows 元件**。 將這個設定變更為**作業系統磁碟機** > **啟動時需要其他驗證** > **在不含相容 TPM 的情形下允許使用 BitLocker**，如下圖所示：

![Azure 中的 Microsoft Antimalware](./media/azure-security-disk-encryption/disk-encryption-fig8.png)

### <a name="install-bitlocker-feature-components"></a>安裝 BitLocker 功能元件
針對 Windows Server 2012 和更新版本，請使用下列命令︰

    dism /online /Enable-Feature /all /FeatureName:BitLocker /quiet /norestart

針對 Windows Server 2008 R2，請使用下列命令︰

    ServerManagerCmd -install BitLockers
### <a name="prepare-the-os-volume-for-bitlocker-by-using-bdehdcfg"></a>使用 `bdehdcfg` 準備用於 BitLocker 的 OS 磁碟區
若要壓縮作業系統磁碟分割並為 BitLocker 準備電腦，請視需要執行 [bdehdcfg](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-basic-deployment#using-bitlocker-to-encrypt-volumescommand)：

    bdehdcfg -target c: shrink -quiet 


### <a name="protect-the-os-volume-by-using-bitlocker"></a>使用 BitLocker 保護作業系統磁碟區
使用 [`manage-bde`](https://technet.microsoft.com/library/ff829849.aspx) 命令，以使用外部的金鑰保護裝置在開機磁碟區上啟用加密。 也將外部金鑰 (.bek 檔案) 放置於外部磁碟機或磁碟區。 下次重新開機後，會在系統/開機磁碟區上啟用加密。

    manage-bde -on %systemdrive% -sk [ExternalDriveOrVolume]
    reboot

> [!NOTE]
> 使用不同的資料/資源 VHD 來準備 VM，才能使用 BitLocker 取得外部金鑰。

## <a name="bkmk_LinuxRunning"></a>在執行中的 Linux VM 上加密 OS 磁碟機

### <a name="prerequisites-for-os-disk-encryption"></a>OS 磁碟加密的必要條件

* VM 必須使用與 OS 磁碟加密相容的散發套件，如 [Azure 磁碟加密常見問題集](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport)中所列示。 
* 必須從 Azure Resource Manager 的Marketplace 映像建立 VM。
* 具有至少 4 GB RAM 的 Azure VM (建議大小為 7 GB)。
* (適用於 RHEL 和 CentOS) 停用 SELinux。 若要停用 SELinux，請參閱「4.4.2. 停用 SELinux」，其位於 VM 上的 [SELinux 使用者和系統管理員指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes.html#sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Disabling_SELinux)。
* 停用 SELinux 之後，至少重新啟動 VM 一次。

### <a name="steps"></a>步驟
1. 使用先前指定的其中一個散發套件來建立 VM。

 若為 CentOS 7.2，支援透過特殊映像加密 OS 磁碟。 若要使用此映像，請在建立 VM 時指定 "7.2n" 做為 SKU︰

 ```powershell
    Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName "OpenLogic" -Offer "CentOS" -Skus "7.2n" -Version "latest"
 ```
2. 根據您的需求設定 VM。 如果您要加密所有 (OS + 資料) 磁碟機，必須可從 /etc/fstab 指定和掛接資料磁碟機。

 > [!NOTE]
 > 使用 UUID=... 來指定 /etc/fstab 中的資料磁碟機，而不是指定區塊裝置名稱 (例如 /dev/sdb1)。 加密期間，VM 上的磁碟機順序會變更。 如果 VM 依賴特定的區塊裝置順序，則無法在加密後掛接裝置。

3. 登出 SSH 工作階段。

4. 若要加密 OS，請在啟用加密時將 volumeType 指定為 **All** 或 **OS**。

 > [!NOTE]
 > 未以 `systemd` 服務的形式執行的所有使用者空間程序皆應使用 `SIGKILL` 來終止。 重新啟動 VM。 當您在執行中的 VM 上啟用 OS 磁碟加密時，請規劃 VM 停機時間。

5. 使用[下一節](#monitoring-os-encryption-progress)的指示定期監視加密進度。

6. 在 Get-AzureRmVmDiskEncryptionStatus 顯示「VMRestartPending」之後，請登入 VM 或使用入口網站、PowerShell 或 CLI 來重新啟動 VM。
    ```powershell
    C:\> Get-AzureRmVmDiskEncryptionStatus  -ResourceGroupName $ResourceGroupName -VMName $VMName
    -ExtensionName $ExtensionName

    OsVolumeEncrypted          : VMRestartPending
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk successfully encrypted, reboot the VM
    ```
重新開機之前，我們建議您儲存 VM 的[開機診斷](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/)。

## <a name="monitoring-os-encryption-progress"></a>監視 OS 加密進度
有三種方式可監視 OS 加密進度：

* 使用 `Get-AzureRmVmDiskEncryptionStatus` Cmdlet，並查看 ProgressMessage 欄位︰
    ```powershell
    OsVolumeEncrypted          : EncryptionInProgress
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk encryption started
    ```
 在 VM 達到 "OS disk encryption started" 之後，需要在進階儲存體所支援的 VM 上耗費大約 40 至 50 分鐘。

 由於 WALinuxAgent 發生 [388 號問題](https://github.com/Azure/WALinuxAgent/issues/388)，某些散發套件的 `OsVolumeEncrypted` 和 `DataVolumesEncrypted` 會顯示為 `Unknown`。 若使用 WALinuxAgent 2.1.5 和更新版本，就會自動修正此問題。 如果您在輸出中看到 `Unknown`，可以使用 Azure 資源 Explorer 確認磁碟加密狀態。

 移至 [Azure 資源 Explorer](https://resources.azure.com/)，然後在左側選取面板中展開這個階層︰

 ~~~~
 |-- subscriptions
     |-- [Your subscription]
          |-- resourceGroups
               |-- [Your resource group]
                    |-- providers
                         |-- Microsoft.Compute
                              |-- virtualMachines
                                   |-- [Your virtual machine]
                                        |-- InstanceView
~~~~                

 在 InstanceView 中，向下捲動以查看磁碟機的加密狀態。

 ![VM 執行個體檢視](./media/azure-security-disk-encryption/vm-instanceview.png)

* 查看[開機診斷](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/)。 來自 ADE 擴充功能的訊息應該會在前面加上 `[AzureDiskEncryption]`。

* 透過 SSH 登入 VM，並從下列位置取得擴充功能記錄檔：

    /var/log/azure/Microsoft.Azure.Security.AzureDiskEncryptionForLinux

 不建議在 OS 加密進行時登入 VM。 其他兩個方法都失敗時，只複製記錄檔。

## <a name="bkmk_preLinux"></a>準備已預先加密的 Linux VHD
準備預先加密的 VHD 會根據散發套件而有所不同。 有關於準備 [Ubuntu 16](#bkmk_Ubuntu)、[openSUSE 13.2](#bkmk_openSUSE) 和 [CentOS 7](#bkmk_CentOS) 的範例可用。 

### <a name="bkmk_Ubuntu"></a> Ubuntu 16
在發佈安裝期間執行下列步驟以設定加密︰

1. 在分割磁碟時選取 [設定加密的磁碟區]。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig1.png)

2. 另外建立一個不得加密的開機磁碟機。 加密根磁碟機。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig2.png)

3. 提供複雜密碼。 這是您上傳至金鑰保存庫的複雜密碼。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig3.png)

4. 完成分割。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig4.png)

5. 啟動 VM 時會要求輸入複雜密碼，請使用步驟 3 中提供的複雜密碼。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig5.png)

6. 使用[這些指示](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-create-upload-ubuntu/)準備要上傳到 Azure 的 VM。 還不要執行最後一個步驟 (解除佈建 VM)。

設定加密來與 Azure 搭配運作，請執行下列步驟：

1. 在 /usr/local/sbin/azure_crypt_key.sh 下，使用以下指令碼的內容建立檔案。 請注意 KeyFileName，因為它是 Azure 使用的複雜密碼檔案名稱。

    ```
    #!/bin/sh
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint
    modprobe vfat >/dev/null 2>&1
    modprobe ntfs >/dev/null 2>&1
    sleep 2
    OPENED=0
    cd /sys/block
    for DEV in sd*; do

        echo "> Trying device: $DEV ..." >&2
        mount -t vfat -r /dev/${DEV}1 $MountPoint >/dev/null||
        mount -t ntfs -r /dev/${DEV}1 $MountPoint >/dev/null
        if [ -f $MountPoint/$KeyFileName ]; then
                cat $MountPoint/$KeyFileName
                umount $MountPoint 2>/dev/null
                OPENED=1
                break
        fi
        umount $MountPoint 2>/dev/null
    done

      if [ $OPENED -eq 0 ]; then
        echo "FAILED to find suitable passphrase file ..." >&2
        echo -n "Try to enter your password: " >&2
        read -s -r A </dev/console
        echo -n "$A"
     else
        echo "Success loading keyfile!" >&2
    fi
```

2. 在 */etc/crypttab* 中變更 crypt 設定。 它看起來應該如下所示：
 ```
    xxx_crypt uuid=xxxxxxxxxxxxxxxxxxxxx none luks,discard,keyscript=/usr/local/sbin/azure_crypt_key.sh
    ```

3. 如果您正在 Windows 中編輯 *azure_crypt_key.sh* 並將它複製到 Linux，執行 `dos2unix /usr/local/sbin/azure_crypt_key.sh`。

4. 在指令碼中新增可執行檔權限︰
 ```
    chmod +x /usr/local/sbin/azure_crypt_key.sh
 ```
5. 以加上程式碼行的方式編輯 /etc/initramfs-tools/modules：
 ```
    vfat
    ntfs
    nls_cp437
    nls_utf8
    nls_iso8859-1
```
6. 執行 `update-initramfs -u -k all` 來更新 initramfs，以讓 `keyscript` 生效。

7. 現在您可以取消佈建 VM。

 ![Ubuntu 16.04 設定](./media/azure-security-disk-encryption/ubuntu-1604-preencrypted-fig6.png)

8. 繼續進行下一個步驟，並將 VHD 上傳到 Azure。

### <a name="bkmk_openSUSE"></a>  openSUSE 13.2
若要在發佈安裝期間設定加密，請執行下列步驟︰
1. 分割磁碟時，請選取 [加密磁碟區群組]，然後輸入密碼。 這是您要上傳到金鑰保存庫的密碼。

 ![openSUSE 13.2 設定](./media/azure-security-disk-encryption/opensuse-encrypt-fig1.png)

2. 使用您的密碼啟動 VM。

 ![openSUSE 13.2 設定](./media/azure-security-disk-encryption/opensuse-encrypt-fig2.png)

3. 請依照[準備 Azure 的 SLES 或 openSUSE 虛擬機器](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-suse-create-upload-vhd/#prepare-opensuse-131)中的指示準備 VM 以上傳至 Azure。 還不要執行最後一個步驟 (解除佈建 VM)。

若要設定加密來與 Azure 搭配運作，請執行下列步驟：
1. 編輯 /etc/dracut.conf 並新增下面這一行︰
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```
2. 註解化 /usr/lib/dracut/modules.d/90crypt/module-setup.sh 檔案結尾的這幾行程式碼：
 ```
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
 ```

3. 在 /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh 檔案開頭附加下面這一行：
 ```
    DRACUT_SYSTEMD=0
 ```
變更所有出現的：
 ```
    if [ -z "$DRACUT_SYSTEMD" ]; then
 ```
至：
```
    if [ 1 ]; then
```
4. 編輯 /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh 並將它附加至「# Open LUKS device」：

    ```
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```
5. 執行 `/usr/sbin/dracut -f -v` 以更新 initrd。

6. 現在您可以取消佈建 VM，並將 VHD 上傳到 Azure。

### <a name="bkmk_CentOS"></a> CentOS 7
若要在發佈安裝期間設定加密，請執行下列步驟︰
1. 在分割磁碟時選取 [加密資料]。

 ![CentOS 7 設定](./media/azure-security-disk-encryption/centos-encrypt-fig1.png)

2. 確定已為根分割選取 [加密]。

 ![CentOS 7 設定](./media/azure-security-disk-encryption/centos-encrypt-fig2.png)

3. 提供複雜密碼。 這是您要上傳到金鑰保存庫的複雜密碼。

 ![CentOS 7 設定](./media/azure-security-disk-encryption/centos-encrypt-fig3.png)

4. 啟動 VM 時會要求輸入複雜密碼，請使用步驟 3 中提供的複雜密碼。

 ![CentOS 7 設定](./media/azure-security-disk-encryption/centos-encrypt-fig4.png)

5. 請依照[準備 Azure 的 CentOS 式虛擬機器](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-create-upload-centos/#centos-70)中的「CentOS 7.0+」指示準備 VM 以上傳至 Azure。 還不要執行最後一個步驟 (解除佈建 VM)。

6. 現在您可以取消佈建 VM，並將 VHD 上傳到 Azure。

若要設定加密來與 Azure 搭配運作，請執行下列步驟：

1. 編輯 /etc/dracut.conf 並新增下面這一行︰
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```

2. 註解化 /usr/lib/dracut/modules.d/90crypt/module-setup.sh 檔案結尾的這幾行程式碼：
```
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
```

3. 在 /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh 檔案開頭附加下面這一行：
```
    DRACUT_SYSTEMD=0
```
變更所有出現的：
```
    if [ -z "$DRACUT_SYSTEMD" ]; then
```
to
```
    if [ 1 ]; then
```
4. 編輯 /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh 並將下列內容附加在「# Open LUKS device」之後：
    ```
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```    
5. 執行「/usr/sbin/dracut -f -v」來更新 initrd。

![CentOS 7 設定](./media/azure-security-disk-encryption/centos-encrypt-fig5.png)

## <a name="bkmk_UploadVHD"></a>將已加密的 VHD 上傳至 Azure 儲存體帳戶
啟用 BitLocker 加密或 DM-Crypt 加密之後，需要將本機加密的 VHD 上傳至儲存體帳戶。
```powershell
    Add-AzureRmVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```
## <a name="bkmk_UploadSecret"></a>將已預先加密 VM 的祕密上傳至金鑰保存庫
使用 Azure AD 應用程式 (舊版) 加密時，先前取得的磁碟加密密碼必須上傳，作為金鑰保存庫中的密碼。 金鑰保存庫必須為 Azure AD 用戶端啟用磁碟加密以及權限。

```powershell 
 $AadClientId = "My-AAD-Client-Id"
 $AadClientSecret = "My-AAD-Client-Secret"

 $key vault = New-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -Location $Location

 Set-AzureRmKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -ServicePrincipalName $AadClientId -PermissionsToKeys all -PermissionsToSecrets all
 Set-AzureRmKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -EnabledForDiskEncryption
``` 

### <a name="bkmk_SecretnoKEK"></a>未使用 KEK 加密的磁碟加密祕密
若要在金鑰保存庫中設定密碼，請使用 [Set-AzureKeyVaultSecret](/powershell/module/azurerm.keyvault/set-azurekeyvaultsecret)。 如果您有 Windows 虛擬機器，bek 檔案會以 base64 字串編碼，然後使用 `Set-AzureKeyVaultSecret` Cmdlet 上傳至您的金鑰保存庫。 如果是 Linux，複雜密碼會以 base64 字串編碼，然後上傳至金鑰保存庫。 此外，請確定在金鑰保存庫中建立密碼時會設定下列標籤。

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


在下一個步驟中使用 `$secretUrl`，以便[在不使用 KEK 的狀況下連接 OS 磁碟](#bkmk_URLnoKEK)。

### <a name="bkmk_SecretKEK"></a>使用 KEK 加密的磁碟加密祕密
將密碼上傳至金鑰保存庫之前，您可以選擇性地使用金鑰加密金鑰來加密密碼。 使用包裝 [API](https://msdn.microsoft.com/library/azure/dn878066.aspx) 先加密使用金鑰加密金鑰的密碼。 這個包裝作業的輸出是 base64 URL 編碼的字串，您可以接著使用 [`Set-AzureKeyVaultSecret`](/powershell/module/azurerm.keyvault/set-azurekeyvaultsecret) Cmdlet 將它上傳做為密碼。

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzureKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzureKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

在下一個步驟中使用 `$KeyEncryptionKey` 和 `$secretUrl`，以便[使用 KEK 連接 OS 磁碟](#BKMK_URLKEK)。

##  <a name="bkmk_SecretURL"></a>在連結 OS 磁碟時指定祕密 URL

###  <a name="bkmk_URLnoKEK"></a>不使用 KEK
當您在連接 OS 磁碟時，需要傳遞 `$secretUrl`。 URL 已在「未使用 KEK 加密的磁碟加密密碼」一節中產生。
```powershell
    Set-AzureRmVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="bkmk_URLKEK"></a>使用 KEK
連接 OS 磁碟時，傳遞 `$KeyEncryptionKey` 和 `$secretUrl`。 URL 已在「使用 KEK 加密的磁碟加密祕密」一節中產生。
```powershell
    Set-AzureRmVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
