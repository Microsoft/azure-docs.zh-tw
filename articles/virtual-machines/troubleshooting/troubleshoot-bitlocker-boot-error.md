---
title: 針對 Azure VM 上的 BitLocker 開機錯誤進行疑難排解 | Microsoft Docs
description: 了解如何針對 Azure VM 中的 BitLocker 開機錯誤進行疑難排解
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: v-jesits
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/20/2020
ms.author: genli
ms.custom: has-adal-ref
ms.openlocfilehash: 87bf311b5199ec187c24c28a42314d9dc6787998
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98633022"
---
# <a name="bitlocker-boot-errors-on-an-azure-vm"></a>Azure VM 上的 BitLocker 開機錯誤

 本文說明當您在 Microsoft Azure 中啟動 Windows 虛擬機器 (VM) 時，可能會遇到的 BitLocker 錯誤。

## <a name="symptom"></a>徵狀

 Windows 虛擬機器未啟動。 當您檢查[開機診斷](./boot-diagnostics.md)視窗中的螢幕擷取畫面時，您看到下列其中一個錯誤訊息：

- 插入的 USB 驅動程式具有 BitLocker 金鑰

- 您已遭到鎖定！ 請輸入復原金鑰以便重新繼續 (鍵盤配置：美國)。輸入錯誤登入資訊太多次，因此已將電腦鎖定來保護隱私權。 若要擷取復原金鑰，請從其他電腦或行動裝置前往 https://windows.microsoft.com/recoverykeyfaq。 如有需要，金鑰識別碼為 XXXXXXX。 或者，您也可以重設電腦。

- 輸入密碼以將此磁碟機 [] 解除鎖定。按 Insert 鍵可看到輸入的密碼。
- 輸入復原金鑰。從 USB 裝置載入復原金鑰。

## <a name="cause"></a>原因

如果 VM 無法找到 BitLocker 復原金鑰 (BEK) 檔案來將已加密的磁碟解密，就可能發生此問題。

## <a name="solution"></a>解決方案

> [!TIP]
> 如果您有最新的 VM 備份，您可以嘗試 [從備份還原 vm](../../backup/backup-azure-arm-restore-vms.md) 以修正開機問題。

若要解決此問題，請停止並解除配置 VM，然後啟動 VM。 此作業會強制讓 VM 從 Azure Key Vault 擷取 BEK 檔案，然後放到加密的磁碟中。 

如果此方法未解決問題，請遵循下列步驟來手動還原 BEK 檔案：

1. 擷取受影響虛擬機器系統磁碟的快照集作為備份。 如需詳細資訊，請參閱[擷取磁碟快照集](../windows/snapshot-copy-managed-disk.md)。
2. [將系統磁碟連結至復原 VM](troubleshoot-recovery-disks-portal-windows.md)。 若要在步驟7中執行 [manage-bde](/windows-server/administration/windows-commands/manage-bde) 命令，必須在復原 VM 中啟用 **BitLocker 磁碟機加密** 功能。

    當您連結至受控磁碟時，可能會收到「包含加密設定，因此無法作為資料磁碟」的錯誤訊息。 在此情況下，請執行下列程式碼以重新試著連結磁碟：

    ```Powershell
    $rgName = "myResourceGroup"
    $osDiskName = "ProblemOsDisk"
    # Set the EncryptionSettingsEnabled property to false, so you can attach the disk to the recovery VM.
    New-AzDiskUpdateConfig -EncryptionSettingsEnabled $false |Update-AzDisk -diskName $osDiskName -ResourceGroupName $rgName

    $recoveryVMName = "myRecoveryVM" 
    $recoveryVMRG = "RecoveryVMRG" 
    $OSDisk = Get-AzDisk -ResourceGroupName $rgName -DiskName $osDiskName;

    $vm = get-AzVM -ResourceGroupName $recoveryVMRG -Name $recoveryVMName 

    Add-AzVMDataDisk -VM $vm -Name $osDiskName -ManagedDiskId $osDisk.Id -Caching None -Lun 3 -CreateOption Attach 

    Update-AzVM -VM $vm -ResourceGroupName $recoveryVMRG
    ```
     您無法將受控磁碟連結至從 Blob 映像還原的 VM。

3. 連結磁碟後，請對復原 VM 建立遠端桌面連線，以便執行一些 Azure PowerShell 指令碼。 請確定您已在復原 VM 上安裝[最新版的 Azure PowerShell](/powershell/azure/)。

4. 開啟已提高權限的 Azure PowerShell 工作階段 (以系統管理員的身分執行)。 執行下列命令以登入 Azure 訂用帳戶：

    ```Powershell
    Add-AzAccount -SubscriptionID [SubscriptionID]
    ```

5. 執行下列指令碼以檢查 BEK 檔案的名稱：

    ```powershell
    $vmName = "myVM"
    $vault = "myKeyVault"
    Get-AzKeyVaultSecret -VaultName $vault | where {($_.Tags.MachineName -eq $vmName) -and ($_.ContentType -match 'BEK')} `
            | Sort-Object -Property Created `
            | ft  Created, `
                @{Label="Content Type";Expression={$_.ContentType}}, `
                @{Label ="MachineName"; Expression = {$_.Tags.MachineName}}, `
                @{Label ="Volume"; Expression = {$_.Tags.VolumeLetter}}, `
                @{Label ="DiskEncryptionKeyFileName"; Expression = {$_.Tags.DiskEncryptionKeyFileName}}
    ```

    以下是輸出範例。 在此情況下，我們假設檔案名是 EF7B2F5A-50C6-4637-0001-7F599C12F85C。BEK.

    ```
    Created               Content Type Volume MachineName DiskEncryptionKeyFileName
    -------               ------------ ------ ----------- -------------------------
    11/20/2020 7:41:56 AM BEK          C:\    myVM   EF7B2F5A-50C6-4637-0001-7F599C12F85C.BEK
    ```
    如果您看到兩個重複的磁碟區，時間戳記較新的磁碟區是復原 VM 目前使用的 BEK 檔案。

    如果 [內容類型] 值是 [包裝的 BEK]，請移至[金鑰加密金鑰 (KEK) 案例](#key-encryption-key-scenario)。

    您已得到磁碟機的 BEK 檔案名稱，接下來您必須建立 secret-file-name.BEK 檔案，以將磁碟機解除鎖定。

6.  將 BEK 檔案下載到復原磁碟。 下列範例會將 BEK 檔案儲存至 C:\BEK 資料夾。 請先確定 `C:\BEK\` 路徑存在，再執行指令碼。

    ```powershell
    $vault = "myKeyVault"
    $bek = "EF7B2F5A-50C6-4637-0001-7F599C12F85C"
    $keyVaultSecret = Get-AzKeyVaultSecret -VaultName $vault -Name $bek
    $bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($keyVaultSecret.SecretValue)
    $bekSecretBase64 = [Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
    $bekFileBytes = [Convert]::FromBase64String($bekSecretbase64)
    $path = "C:\BEK\DiskEncryptionKeyFileName.BEK"
    [System.IO.File]::WriteAllBytes($path,$bekFileBytes)
    ```

7.  若要使用 BEK 檔案來解除鎖定連接的磁片，請執行下列命令。

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-0001-7F599C12F85C.BEK
    ```
    在此範例中，連結的 OS 磁碟是磁碟機 F。請確定您使用的是正確的磁碟機代號。 

8. 使用 BEK 機碼成功解除鎖定磁片之後，請從復原 VM 卸離磁片，然後使用這個新的 OS 磁片來重新建立 VM。

    > [!NOTE]
    > 使用磁片加密的 Vm 不支援交換 OS 磁片。

9. 如果新的 VM 仍無法正常開機，請在解除鎖定磁片磁碟機之後，嘗試下列其中一個步驟：

    - 藉由執行下列動作，暫停保護以暫時關閉 BitLocker：

    ```console
    manage-bde -protectors -disable F: -rc 0
    ```

    - 完整解密磁片磁碟機。 若要這樣做，請執行下列命令：

    ```console
    manage-bde -off F:
    ```

### <a name="key-encryption-key-scenario"></a>金鑰加密金鑰案例

若為金鑰加密金鑰案例，請遵循下列步驟：

1. 確定所登入的使用者帳戶需要 Key Vault 存取原則 (**使用者|金鑰權限|密碼編譯作業|解除包裝金鑰**) 中的「解除包裝」權限。
2. 將下列腳本儲存至。PS1 檔案：

    ```powershell
    #Set the Parameters for the script
    param (
            [Parameter(Mandatory=$true)]
            [string] 
            $keyVaultName,
            [Parameter(Mandatory=$true)]
            [string] 
            $kekName,
            [Parameter(Mandatory=$true)]
            [string]
            $secretName,
            [Parameter(Mandatory=$true)]
            [string]
            $bekFilePath,
            [Parameter(Mandatory=$true)]
            [string] 
            $adTenant
            )
    # Load ADAL Assemblies
    $adal = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms = "${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:ProgramFiles}\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    If ((Test-Path -Path $adal) -and (Test-Path -Path $adalforms)) { 

    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)
     }
     else
     {
    $adal="${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms ="${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts\$(((dir ${env:userprofile}\Documents\WindowsPowerShell\Modules\Az.Accounts).name) | select -last 1)\PreloadAssemblies\Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"
    [System.Reflection.Assembly]::LoadFrom($adal)
    [System.Reflection.Assembly]::LoadFrom($adalforms)
     }  

    # Set well-known client ID for AzurePowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2" 
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURI = "https://vault.azure.net"
    # Set Authority to Azure AD Tenant
    $authority = "https://login.windows.net/$adtenant"
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $platformParameters = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList "Auto"
    $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI, $clientId, $redirectUri, $platformParameters).result
    # Generate auth header 
    $authHeader = $authResult.CreateAuthorizationHeader()
    # Set HTTP request headers to include Authorization header
    $headers = @{'x-ms-version'='2014-08-01';"Authorization" = $authHeader}

    ########################################################################################################################
    # 1. Retrieve wrapped BEK
    # 2. Make KeyVault REST API call to unwrap the BEK
    # 3. Convert the Base64Url string returned by KeyVault unwrap to Base64 string 
    # 4. Convert Base64 string to bytes and write to the BEK file
    ########################################################################################################################

    #Get wrapped BEK and place it in JSON object to send to KeyVault REST API
    $keyVaultSecret = Get-AzKeyVaultSecret -VaultName $keyVaultName -Name $secretName
    $bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($keyVaultSecret.SecretValue)
    $wrappedBekSecretBase64 = [Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
    $jsonObject = @"
    {
    "alg": "RSA-OAEP",
    "value" : "$wrappedBekSecretBase64"
    }
    "@

    #Get KEK Url
    $kekUrl = (Get-AzKeyVaultKey -VaultName $keyVaultName -Name $kekName).Key.Kid;
    $unwrapKeyRequestUrl = $kekUrl+ "/unwrapkey?api-version=2015-06-01";

    #Call KeyVault REST API to Unwrap 
    $result = Invoke-RestMethod -Method POST -Uri $unwrapKeyRequestUrl -Headers $headers -Body $jsonObject -ContentType "application/json" -Debug

    #Convert Base64Url string returned by KeyVault unwrap to Base64 string
    $base64UrlBek = $result.value;
    $base64Bek = $base64UrlBek.Replace('-', '+');
    $base64Bek = $base64Bek.Replace('_', '/');
    if($base64Bek.Length %4 -eq 2)
    {
        $base64Bek+= '==';
    }
    elseif($base64Bek.Length %4 -eq 3)
    {
        $base64Bek+= '=';
    }

    #Convert base64 string to bytes and write to BEK file
    $bekFileBytes = [System.Convert]::FromBase64String($base64Bek);
    [System.IO.File]::WriteAllBytes($bekFilePath,$bekFileBytes)

    #Delete the key from the memory
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($bstr)
    clear-variable -name wrappedBekSecretBase64
    ```
3. 設定參數。 此指令碼會處理 KEK 祕密以建立 BEK 金鑰，再將它儲存到復原 VM 上的本機資料夾。 如果您在執行腳本時收到錯誤，請參閱 [腳本疑難排解](#script-troubleshooting) 一節。

4. 指令碼開始時，您會看到下列輸出：

    GAC 版本位置                                                                              
    ---    -------        --------                                                                              
    False v 4.0.30319 C:\Program Files\WindowsPowerShell\Modules\Az.Accounts \. .。。 False v 4.0.30319 C:\Program Files\WindowsPowerShell\Modules\Az.Accounts \. .。。

    指令碼完成時，您會看到下列輸出︰

    ```output
    VERBOSE: POST https://myvault.vault.azure.net/keys/rondomkey/<KEY-ID>/unwrapkey?api-
    version=2015-06-01 with -1-byte payload
    VERBOSE: received 360-byte response of content type application/json; charset=utf-8
    ```

5. 若要使用 BEK 檔案將連結的磁碟解除鎖定，請執行下列命令：

    ```powershell
    manage-bde -unlock F: -RecoveryKey "C:\BEK\EF7B2F5A-50C6-4637-9F13-7F599C12F85C.BEK
    ```
    在此範例中，連結的 OS 磁碟是磁碟機 F。請確定您使用的是正確的磁碟機代號。 

6. 使用 BEK 機碼成功解除鎖定磁片之後，請從復原 VM 卸離磁片，然後使用這個新的 OS 磁片來重新建立 VM。 

    > [!NOTE]
    > 使用磁片加密的 Vm 不支援交換 OS 磁片。

7. 如果新的 VM 仍無法正常開機，請在解除鎖定磁片磁碟機之後，嘗試下列其中一個步驟：

    - 藉由執行下列命令，暫停保護以暫時關閉 BitLocker：

    ```console
    manage-bde -protectors -disable F: -rc 0
    ```

    - 完整解密磁片磁碟機。 若要這樣做，請執行下列命令：

    ```console
    manage-bde -off F:
    ```

## <a name="script-troubleshooting"></a>腳本疑難排解

**錯誤：無法載入檔案或元件**

發生此錯誤的原因是 ADAL 元件的路徑錯誤。 您可以搜尋 `Az.Accounts` 資料夾來尋找正確的路徑。

**錯誤：無法辨識 Get-AzKeyVaultSecret 或 Get-AzKeyVaultSecret 作為 Cmdlet 的名稱**

如果您使用舊的 AZ PowerShell 模組，您必須將兩個命令變更為 `Get-AzureKeyVaultSecret` 和 `Get-AzureKeyVaultSecret` 。

**參數範例**

| 參數  | 值範例  |註解   |
|---|---|---|
|  $keyVaultName | myKeyVault2112852926  | 儲存金鑰的金鑰保存庫名稱 |
|$kekName   |mykey   | 用來加密 VM 的金鑰名稱|
|$secretName   |7EB4F531-5FBA-4970-8E2D-C11FD6B0C69D  | VM 金鑰的秘密名稱|
|$bekFilePath   |c:\bek\7EB4F531-5FBA-4970-8E2D-C11FD6B0C69D.BEK |寫入 BEK 檔案的路徑。|
|$adTenant  |contoso.onmicrosoft.com   | 裝載金鑰保存庫 Azure Active Directory 的 FQDN 或 GUID |
