---
title: 搭配 Windows 使用 Azure 檔案共用 | Microsoft Docs
description: 了解如何搭配 Windows 和 Windows Server 使用 Azure 檔案共用。
services: storage
documentationcenter: na
author: RenaShahMSFT
manager: aungoo
editor: tamram
ms.assetid: ''
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/07/2018
ms.author: renash
ms.openlocfilehash: 54e084e6480c872ff6dd4625b8c87d5a60a181ba
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39392262"
---
# <a name="use-an-azure-file-share-with-windows"></a>搭配 Windows 使用 Azure 檔案共用
[Azure 檔案服務](storage-files-introduction.md)是 Microsoft 易於使用的雲端檔案系統。 Azure 檔案共用可在 Windows 和 Windows Server 中順暢地使用。 本文討論搭配 Windows 和 Windows Server 使用 Azure 檔案共用的考量。

若要在 Azure 區域之外使用 Azure 檔案共用，例如內部部署環境或是在不同的 Azure 區域，作業系統必須支援 SMB 3.0。 

您可以在於 Azure VM 或內部部署環境中執行的 Windows 安裝上使用 Azure 檔案共用。 下表說明哪些作業系統版本在哪個環境中支援存取檔案共用：

| Windows 版本        | SMB 版本 | 可在 Azure VM 中掛接 | 可在內部部署環境掛接 |
|------------------------|-------------|-----------------------|----------------------|
| Windows Server 2019 (預覽)<sup>1</sup> | SMB 3.0 | 是 | 是 |
| Windows 10<sup>2</sup> | SMB 3.0 | 是 | 是 |
| Windows Server 半年度通道<sup>3</sup> | SMB 3.0 | 是 | 是 |
| Windows Server 2016    | SMB 3.0     | 是                   | 是                  |
| Windows 8.1            | SMB 3.0     | 是                   | 是                  |
| Windows Server 2012 R2 | SMB 3.0     | 是                   | 是                  |
| Windows Server 2012    | SMB 3.0     | 是                   | 是                  |
| Windows 7              | SMB 2.1     | 是                   | 否                   |
| Windows Server 2008 R2 | SMB 2.1     | 是                   | 否                   |

<sup>1</sup>Windows Server 2019 (預覽) 可透過 [Windows Server Insiders 方案](https://insider.windows.com/for-business-getting-started-server/)取得。 雖然 Windows Server 2019 尚不支援用於生產環境，請讓我們知道除了 [Windows 疑難排解指南](storage-troubleshoot-windows-file-connection-problems.md)中所說明的問題，您是否有連線到 Azure 檔案共用的問題。  
<sup>2</sup>Windows 10 版本 1507、1607、1703、1709 和 1803。  
<sup>3</sup>Windows Server 版本 1709 和 1803。

> [!Note]  
> 我們一律建議針對您的 Windows 版本採取最新的 KB。

## <a name="prerequisites"></a>必要條件 
* **儲存體帳戶名稱**：若要掛接 Azure 檔案共用，您需要儲存體帳戶的名稱。

* **儲存體帳戶金鑰**：若要掛接 Azure 檔案共用，您需要主要 (或次要) 金鑰。 掛接目前不支援 SAS 金鑰。

* **確定已開啟連接埠 445**：SMB 通訊協定要求 TCP 連接埠 445 為開啟狀態；如果連接埠 445 遭到封鎖，連線將會失敗。 您可以使用 `Test-NetConnection` Cmdlet，查看您的防火牆是否封鎖連接埠 445。 下列 PowerShell 程式碼假設您已安裝 AzureRM PowerShell 模組，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)以獲得詳細資訊。 請記得以儲存體帳戶的相關名稱取代 `<your-storage-account-name>` 和 `<your-resoure-group-name>`。

    ```PowerShell
    $resourceGroupName = "<your-resource-group-name>"
    $storageAccountName = "<your-storage-account-name>"

    # This command requires you to be logged into your Azure account, run Login-AzureRmAccount if you haven't
    # already logged in.
    $storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName

    # The ComputerName, or host, is <storage-account>.file.core.windows.net for Azure Public Regions.
    # $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as soverign clouds
    # or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
    Test-NetConnection -ComputerName [System.Uri]::new($storageAccount.Context.FileEndPoint).Host -Port 445
    ```

    如果連線成功，您應會看見下列輸出：

    ```
    ComputerName     : <storage-account-host-name>
    RemoteAddress    : <storage-account-ip-address>
    RemotePort       : 445
    InterfaceAlias   : <your-network-interface>
    SourceAddress    : <your-ip-address>
    TcpTestSucceeded : True
    ```

    > [!Note]  
    > 上述命令會傳回儲存體帳戶的目前 IP 位址。 此 IP 位址不保證會維持不變，而且可能隨時變更。 請勿將此 IP 位址硬式編碼到任何指令碼，或防火牆組態中。 

## <a name="using-an-azure-file-share-with-windows"></a>搭配 Windows 使用 Azure 檔案共用
若要搭配 Windows 使用 Azure 檔案共用，您必須掛接它 (這表示將磁碟機代號或掛接點路徑指派給它)，或透過 [UNC 路徑](https://msdn.microsoft.com/library/windows/desktop/aa365247.aspx)存取它。 

不同於其他您可能有互動的 SMB 共用 (例如 Windows Server、Linux Samba 伺服器或 NAS 裝置上裝載的共用)，Azure 檔案共用目前不支援使用您的 Active Directory (AD) 或 Azure Active Directory (AAD) 身分識別進行 Kerberos 驗證，雖然這是我們正在[努力](https://feedback.azure.com/forums/217298-storage/suggestions/6078420-acl-s-for-azurefiles)的功能。 相反地，對於包含 Azure 檔案共用的儲存體帳戶，您必須使用儲存體帳戶金鑰來存取 Azure 檔案共用。 儲存體帳戶金鑰是儲存體帳戶的管理員金鑰，包括您所存取檔案共用內所有檔案和資料夾的系統管理員權限，以及您儲存體帳戶內所有檔案共用和其他儲存體資源 (blob、佇列、資料表等) 的系統管理員權限。 如果這不足以處理您的工作負載，[Azure 檔案同步](storage-files-planning.md#data-access-method)可以在過渡期間處理缺乏 Kerberos 驗證和 ACL 支援，直到可公開取得以 AAD 為基礎的 Kerberos 驗證和 ACL 支援為止。

將應有 SMB 檔案共用的企業營運 (LOB) 應用程式隨即移轉至 Azure 的常見模式，就是使用 Azure 檔案共用作為在 Azure VM 中執行專用 Windows 檔案伺服器的替代方式。 成功移轉企業營運應用程式以使用 Azure 檔案共用的一項重要考量，就是許多企業營運應用程式使用有限的系統權限 (而不是 VM 的系統管理帳戶)，在專屬服務帳戶的內容之下執行。 因此，您必須確定從服務帳戶的內容 (而不是系統管理帳戶) 掛接/儲存 Azure 檔案共用的認證。

### <a name="persisting-azure-file-share-credentials-in-windows"></a>在 Windows 中保存 Azure 檔案共用認證  
[Cmdkey](https://docs.microsoft.com/windows-server/administration/windows-commands/cmdkey) 公用程式可讓您在 Windows 內儲存您的儲存體帳戶認證。 這表示當您嘗試透過 UNC 路徑存取 Azure 檔案共用或掛接 Azure 檔案共用時，您不需要指定認證。 若要儲存儲存體帳戶的認證，請執行下列 PowerShell 命令，並視情況取代 `<your-storage-account-name>` 和 `<your-resoure-group-name>`。

```PowerShell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"

# These commands require you to be logged into your Azure account, run Login-AzureRmAccount if you haven't
# already logged in.
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$storageAccountKeys = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName

# The cmdkey utility is a command-line (rather than PowerShell) tool. We use Invoke-Expression to allow us to 
# consume the appropriate values from the storage account variables. The value given to the add parameter of the
# cmdkey utility is the host address for the storage account, <storage-account>.file.core.windows.net for Azure 
# Public Regions. $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as soverign 
# clouds or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
Invoke-Expression -Command "cmdkey /add:$([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) " + `
    "/user:AZURE\$($storageAccount.StorageAccountName) /pass:$($storageAccountKeys[0].Value)"
```

您可以使用 list 參數，確認 cmdkey 公用程式已儲存儲存體帳戶的認證：

```PowerShell
cmdkey /list
```

如果已成功儲存 Azure 檔案共用的認證，預期的輸出會如下所示 (清單中可能會儲存額外的金鑰)：

```
Currently stored credentials:

Target: Domain:target=<storage-account-host-name>
Type: Domain Password
User: AZURE\<your-storage-account-name>
```

您現在應該能夠掛接或存取共用，而不需提供額外的認證。

#### <a name="advanced-cmdkey-scenarios"></a>進階 cmdkey 案例
請考慮 cmdkey 的額外兩個案例：在電腦上儲存另一位使用者的認證 (如服務帳戶)，以及使用 PowerShell 遠端處理在遠端電腦上儲存認證。

在電腦上儲存另一位使用者的認證非常簡單：登入您的帳戶後，只要執行下列 PowerShell 命令即可：

```PowerShell
$password = ConvertTo-SecureString -String "<service-account-password>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "<service-account-username>", $password
Start-Process -FilePath PowerShell.exe -Credential $credential -LoadUserProfile
```

這會在服務帳戶 (或使用者帳戶) 的使用者內容之下，開啟新的 PowerShell 視窗。 您可以接著使用[上述](#persisting-azure-file-share-credentials-in-windows)的 cmdkey 公用程式。

但是，不可能使用 PowerShell 遠端處理在遠端電腦上儲存認證，因為使用者若透過 PowerShell 遠端處理登入，cmdkey 並不允許存取 (即使是新增項目) 其認證存放區。 我們建議透過[遠端桌面](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/windows)登入電腦。

### <a name="mount-the-azure-file-share-with-powershell"></a>使用 PowerShell 掛接 Azure 檔案共用
從一般 (也就是未提升權限) 的 PowerShell 工作階段執行下列命令，以掛接 Azure 檔案共用。 請記得使用正確的資訊取代 `<your-resource-group-name>`、`<your-storage-account-name>`、`<your-file-share-name>` 和 `<desired-drive-letter>`。

```PowerShell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"
$fileShareName = "<your-file-share-name>"

# These commands require you to be logged into your Azure account, run Login-AzureRmAccount if you haven't
# already logged in.
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$storageAccountKeys = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName
$fileShare = Get-AzureStorageShare -Context $storageAccount.Context | Where-Object { 
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    throw [System.Exception]::new("Azure file share not found")
}

# The value given to the root parameter of the New-PSDrive cmdlet is the host address for the storage account, 
# <storage-account>.file.core.windows.net for Azure Public Regions. $fileShare.StorageUri.PrimaryUri.Host is 
# used because non-Public Azure regions, such as soverign clouds or Azure Stack deployments, will have different 
# hosts for Azure file shares (and other storage resources).
$password = ConvertTo-SecureString -String $storageAccountKeys[0].Value -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "AZURE\$($storageAccount.StorageAccountName)", $password
New-PSDrive -Name <desired-drive-letter> -PSProvider FileSystem -Root "\\$($fileShare.StorageUri.PrimaryUri.Host)\$($fileShare.Name)" -Credential $credential -Persist
```

> [!Note]  
> 如果已儲存認證，在 `New-PSDrive` cmdlet 上使用 `-Persist` 選項，只會允許在開機時重新掛接檔案共用。 您可以使用[先前所述](#persisting-azure-file-share-credentials-in-windows)的 cmdkey 來儲存認證。 

如有需要，您可使用下列 PowerShell cmdlet 來卸載 Azure 檔案共用。

```PowerShell
Remove-PSDrive -Name <desired-drive-letter>
```

### <a name="mount-the-azure-file-share-with-file-explorer"></a>使用檔案總管掛接 Azure 檔案共用
> [!Note]  
> 請注意，下列指示會顯示在 Windows 10 上，與較舊版本可能稍有不同。 

1. 開啟檔案總管。 做法是從 [開始] 功能表中開啟，或按 Win + E 捷徑。

2. 瀏覽至視窗左側的 [本機] 項目。 這會變更功能區中所提供的功能表。 在 [電腦] 功能表之下，選取 [連線網路磁碟機]。
    
    ![「連線網路磁碟機」下拉式功能表的螢幕擷取畫面](./media/storage-how-to-use-files-windows/1_MountOnWindows10.png)

3. 從 Azure 入口網站的 [連線] 窗格中複製 UNC 路徑。 

    ![[Azure 檔案服務連線] 窗格中的 UNC 路徑](./media/storage-how-to-use-files-windows/portal_netuse_connect.png)

4. 選取磁碟機代號並輸入 UNC 路徑。 
    
    ![「連線網路磁碟機」對話方塊的螢幕擷取畫面](./media/storage-how-to-use-files-windows/2_MountOnWindows10.png)

5. 使用前面加上 `AZURE\` 的儲存體帳戶名稱作為使用者名稱，並使用儲存體帳戶金鑰作為密碼。
    
    ![網路認證對話方塊的螢幕擷取畫面](./media/storage-how-to-use-files-windows/3_MountOnWindows10.png)

6. 視需要使用 Azure 檔案共用。
    
    ![現在已掛接 Azure 檔案共用](./media/storage-how-to-use-files-windows/4_MountOnWindows10.png)

7. 當您準備好要卸載 Azure 檔案共用時，您可以滑鼠右鍵按一下 [檔案總管] 中 [網路位置] 下的共用項目，然後選取 [中斷連線]。

## <a name="securing-windowswindows-server"></a>保護 Windows/Windows Server
為了在 Windows 上掛接 Azure 檔案共用，必須可以存取連接埠 445。 許多組織會封鎖連接埠 445，因為 SMB 1 固有的安全性風險。 SMB 1 也稱為 CIFS (Common Internet File System)，是 Windows 和 Windows Server 隨附的舊版檔案系統通訊協定。 SMB 1 已過期、沒有效率，而且最重要的是不安全的通訊協定。 好消息是 Azure 檔案服務不支援 SMB 1，而且所有支援的 Windows 和 Windows Server 版本都允許移除或停用 SMB 1。 我們始終[強烈建議](https://aka.ms/stopusingsmb1)先在 Windows 中移除或停用 SMB 1 用戶端和伺服器，再於生產環境中使用 Azure 檔案共用。

下表針對每個 Windows 版本提供 SMB 1 狀態的詳細資訊：

| Windows 版本                           | SMB 1 預設狀態 | 停用/移除方法       | 
|-------------------------------------------|----------------------|-----------------------------|
| Windows Server 2019 (預覽)             | 已停用             | 利用 Windows 功能移除 |
| Windows Server 版本 1709+            | 已停用             | 利用 Windows 功能移除 |
| Windows 10 版本 1709+                | 已停用             | 利用 Windows 功能移除 |
| Windows Server 2016                       | 已啟用              | 利用 Windows 功能移除 |
| Windows 10 版本 1507、1607 和 1703 | 已啟用              | 利用 Windows 功能移除 |
| Windows Server 2012 R2                    | 已啟用              | 利用 Windows 功能移除 | 
| Windows 8.1                               | 已啟用              | 利用 Windows 功能移除 | 
| Windows Server 2012                       | 已啟用              | 利用登錄停用       | 
| Windows Server 2008 R2                    | 已啟用              | 利用登錄停用       |
| Windows 7                                 | 已啟用              | 利用登錄停用       | 

### <a name="auditing-smb-1-usage"></a>稽核 SMB 1 使用量
> 套用至 Windows Server 2019 (預覽)、Windows Server 半年度通道 (版本 1709 和 1803)、Windows Server 2016、Windows 10 (版本 1507、1607、1703、1709 和 1803)、Windows Server 2012 R2 和 Windows 8.1

移除您環境中的 SMB 1 之前，您可以稽核 SMB 1 使用情況，以查看是否有任何用戶端會因此變更而中斷。 如果對含有 SMB 1 的 SMB 共用 提出任何要求，稽核事件將會記錄在 `Applications and Services Logs > Microsoft > Windows > SMBServer > Audit` 之下的事件記錄中。 

> [!Note]  
> 若要在 Windows Server 2012 R2 和 Windows 8.1 上啟用稽核支援，請至少安裝 [KB4022720](https://support.microsoft.com/help/4022720/windows-8-1-windows-server-2012-r2-update-kb4022720)。

若要啟用稽核，請從提高權限的 PowerShell 工作階段執行下列 Cmdlet︰

```PowerShell
Set-SmbServerConfiguration –AuditSmb1Access $true
```

### <a name="removing-smb-1-from-windows-server"></a>從 Windows Server 移除 SMB 1
> 套用至 Windows Server 2019 (預覽)、Windows Server 半年度通道 (版本 1709 和 1803)、Windows Server 2016、Windows Server 2012 R2

若要從 Windows Server 執行個體移除 SMB 1，請從提高權限的 PowerShell 工作階段執行下列 Cmdlet︰

```PowerShell
Remove-WindowsFeature -Name FS-SMB1
```

若要完成移除程序，請重新啟動您的伺服器。 

> [!Note]  
> 從 Windows 10 和 Windows Server 版本 1709 開始，預設不會安裝 SMB 1，而且 SMB 1 用戶端和 SMB 1 伺服器具有不同的 Windows 功能。 我們一律建議不要安裝 SMB 1 伺服器 (`FS-SMB1-SERVER`) 和 SMB 1 用戶端 (`FS-SMB1-CLIENT`)。

### <a name="removing-smb-1-from-windows-client"></a>從 Windows 用戶端移除 SMB 1
> 套用至 Windows 10 (版本 1507、1607、1703、1709 和 1803) 和 Windows 8.1

若要從 Windows 用戶端移除 SMB 1，請從提高權限的 PowerShell 工作階段執行下列 Cmdlet︰

```PowerShell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

若要完成移除程序，請重新啟動您的電腦。

### <a name="disabling-smb-1-on-legacy-versions-of-windowswindows-server"></a>在舊版 Windows/Windows Server 上停用 SMB 1
> 套用至 Windows Server 2012、Windows Server 2008 R2 和 Windows 7

SMB 1 無法完全從舊版的 Windows/Windows Server 中移除，但是可以透過登錄停用。 若要停用 SMB 1，請在 `HKEY_LOCAL_MACHINE > SYSTEM > CurrentControlSet > Services > LanmanServer > Parameters` 之下建立類型為 `DWORD` 且值為 `0` 的新登錄機碼 `SMB1`。

您也可以使用下列 PowerShell Cmdlet 輕鬆達成：

```PowerShell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" SMB1 -Type DWORD -Value 0 –Force
```

建立此登錄機碼之後，您必須重新啟動伺服器，才能停用 SMB 1。

### <a name="smb-resources"></a>SMB 資源
- [停只使用 SMB 1](https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/)
- [SMB 1 產品情報交換所](https://blogs.technet.microsoft.com/filecab/2017/06/01/smb1-product-clearinghouse/)
- [使用 DSCEA 探索您環境中的 SMB 1](https://blogs.technet.microsoft.com/ralphkyttle/2017/04/07/discover-smb1-in-your-environment-with-dscea/)
- [透過群組原則停用 SMB 1](https://blogs.technet.microsoft.com/secguide/2017/06/15/disabling-smbv1-through-group-policy/)

## <a name="next-steps"></a>後續步驟
請參閱這些連結，以取得 Azure 檔案服務的詳細資訊：
- [規劃 Azure 檔案部署](storage-files-planning.md)
* [常見問題集](../storage-files-faq.md)
* [在 Windows 上進行疑難排解](storage-troubleshoot-windows-file-connection-problems.md)      