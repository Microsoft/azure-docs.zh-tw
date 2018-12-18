---
title: 使用 Azure 檔案同步管理已註冊的伺服器 | Microsoft Docs
description: 了解如何向 Azure 檔案同步儲存體同步服務註冊及取消註冊 Windows Server。
services: storage
author: wmgries
ms.service: storage
ms.topic: article
ms.date: 07/19/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: 1aa1bd085a312e379dc996a860c7f97b2e0dfe73
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918871"
---
# <a name="manage-registered-servers-with-azure-file-sync"></a>使用 Azure 檔案同步管理已註冊的伺服器
Azure 檔案同步可讓您將組織的檔案共用集中在「Azure 檔案服務」中，而不需要犧牲內部部署檔案伺服器的靈活度、效能及相容性。 它會將您的 Windows Server 轉換成 Azure 檔案共用的快速快取來達到這個目的。 您可以使用 Windows Server 上可用的任何通訊協定來存取本機資料 (包括 SMB、NFS 和 FTPS)，並且可以在世界各地擁有任何所需數量的快取。

下列文章說明如何使用儲存體同步服務來註冊及管理伺服器。 如需如何從頭到尾部署 Azure 檔案同步的資訊，請參閱[如何部署 Azure 檔案同步](storage-sync-files-deployment-guide.md)。

## <a name="registerunregister-a-server-with-storage-sync-service"></a>使用儲存體同步服務來註冊/取消註冊伺服器
使用 Azure 檔案同步來註冊伺服器可在 Windows Server 與 Azure 之間建立信任關係。 此關係可用來在伺服器上建立伺服器端點，其代表應與 Azure 檔案共用 (也稱為雲端端點) 同步的特定資料夾。 

### <a name="prerequisites"></a>必要條件
若要使用儲存體同步服務來註冊伺服器，您必須先準備好符合下列必要條件的伺服器：

* 伺服器必須執行支援的 Windows 版本。 如需詳細資訊，請參閱 [Azure 檔案同步系統需求和互通性](storage-sync-files-planning.md#azure-file-sync-system-requirements-and-interoperability)。
* 確定已部署儲存體同步服務。 如需如何部署儲存體同步服務的詳細資訊，請參閱[如何部署 Azure 檔案同步](storage-sync-files-deployment-guide.md)。
* 確定伺服器已連線到網際網路，而且 Azure 可供存取。
* 使用 [伺服器管理員] UI 停用系統管理員的 [IE 增強式安全性設定]。
    
    ![醒目提示 [IE 增強式安全性設定] 的 [伺服器管理員] UI](media/storage-sync-files-server-registration/server-manager-ie-config.png)

* 確定您的伺服器上已安裝 AzureRM PowerShell 模組。 如果您的伺服器是容錯移轉叢集的成員，則叢集中的每個節點都需要 AzureRM 模組。 您可以在 [Install and configure Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps) (安裝和設定 Azure PowerShell) 中，找到有關如何安裝 AzureRM 模組的更多詳細資料。

    > [!Note]  
    > 建議使用最新版 AzureRM PowerShell 模組來註冊/取消註冊伺服器。 如果先前在此伺服器上已安裝 AzureRM 套件 (而且此伺服器上的 PowerShell 版本為 5.* 或更新版本)，您可以使用 `Update-Module` Cmdlet 更新此套件。 
* 如果您在您的環境中使用網路 Proxy 伺服器，請在伺服器上針對要使用的同步代理程式設定 Proxy 設定。
    1. 判斷 Proxy IP 位址和連接埠號碼
    2. 編輯這兩個檔案：
        * C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config
        * C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
    3. 在上述兩個檔案中的 /System.ServiceModel 底下，新增圖 1 (本節下方) 中的行，將 127.0.0.1:8888 變更為正確的 IP 位址 (取代 127.0.0.1) 以及正確的連接埠號碼 (取代 8888)：
    4. 透過命令列設定 WinHTTP Proxy 設定：
        * 顯示 Proxy：netsh winhttp show proxy
        * 設定 Proxy：netsh winhttp set proxy 127.0.0.1:8888
        * 重設 Proxy：netsh winhttp reset proxy
        * 如果是在安裝代理程式之後設定，則重新啟動我們的同步代理程式：   net stop filesyncsvc
    
```XML
    Figure 1:
    <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
            <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
    </system.net>
```    

### <a name="register-a-server-with-storage-sync-service"></a>向儲存體同步服務註冊伺服器
您必須使用儲存體同步服務來註冊伺服器，才能在 Azure 檔案同步的同步群組中，將伺服器當作伺服器端點使用。 一次只能使用一個儲存體同步服務來註冊一部伺服器。

#### <a name="install-the-azure-file-sync-agent"></a>安裝 Azure 檔案同步代理程式
1. [下載 Azure 檔案同步代理程式](https://go.microsoft.com/fwlink/?linkid=858257)。
2. 啟動 Azure 檔案同步代理程式安裝程式。
    
    ![Azure 檔案同步代理程式安裝程式的第一個窗格](media/storage-sync-files-server-registration/install-afs-agent-1.png)

3. 請務必使用 Microsoft Update 來啟用 Azure 檔案同步代理程式的更新。 這很重要，因為伺服器套件的重大安全性問題修正與功能加強隨附於 Microsoft Update。

    ![確定已在 Azure 檔案同步代理程式安裝程式的 [Microsoft Update] 窗格中啟用 Microsoft Update](media/storage-sync-files-server-registration/install-afs-agent-2.png)

4. 如果先前尚未註冊伺服器，完成安裝之後就會立即快顯伺服器註冊 UI。

> [!Important]  
> 如果伺服器是容錯移轉叢集的成員，則必須在叢集中的每個節點上安裝 Azure 檔案同步代理程式。

#### <a name="register-the-server-using-the-server-registration-ui"></a>使用伺服器註冊 UI 註冊伺服器
> [!Important]  
> 雲端解決方案提供者 (CSP) 訂用帳戶無法使用伺服器註冊 UI。 請改用 PowerShell (本節下方)。

1. 如果完成 Azure 檔案同步代理程式的安裝之後未立即啟動伺服器註冊 UI，您可以執行 `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe` 手動將它啟動。
2. 按一下 [登入] 以存取您的 Azure 訂用帳戶。 

    ![伺服器註冊 UI 的開啟中對話方塊](media/storage-sync-files-server-registration/server-registration-ui-1.png)

3. 從對話方塊中，選取正確的訂用帳戶、資源群組和儲存體同步服務。

    ![儲存體同步服務資訊](media/storage-sync-files-server-registration/server-registration-ui-2.png)

4. 在預覽版中，需要再登入一次才能完成此程序。 

    ![登入對話方塊](media/storage-sync-files-server-registration/server-registration-ui-3.png)

> [!Important]  
> 如果伺服器是容錯移轉叢集的成員，每部伺服器都必須執行伺服器註冊。 當您在 Azure 入口網站中檢視已註冊的伺服器時，Azure 檔案同步會自動將每個節點識別為相同容錯移轉叢集的成員，並將它們適當地分組在一起。

#### <a name="register-the-server-with-powershell"></a>使用 PowerShell 註冊伺服器
您也可以透過 PowerShell 執行伺服器註冊。 這是雲端解決方案提供者 (CSP) 訂用帳戶唯一支援的伺服器註冊方法：

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll"
Login-AzureRmStorageSync -SubscriptionID "<your-subscription-id>" -TenantID "<your-tenant-id>"
Register-AzureRmStorageSyncServer -SubscriptionId "<your-subscription-id>" - ResourceGroupName "<your-resource-group-name>" - StorageSyncService "<your-storage-sync-service-name>"
```

### <a name="unregister-the-server-with-storage-sync-service"></a>向儲存體同步服務取消註冊伺服器
向儲存體同步服務取消註冊伺服器需要執行幾個步驟。 讓我們看看如何正確地取消註冊伺服器。

> [!Warning]  
> 除非 Microsoft 工程師明確指示，否則請勿取消註冊後再註冊伺服器，或將伺服器端點移除後再重新建立，嘗試對同步、雲端階層或任何其他方面的 Azure 檔案同步問題進行疑難排解。 取消註冊和移除伺服器端點是破壞性作業，而且註冊伺服器並重新建立伺服器端點之後，在包含伺服器端點之磁碟區上的階層式檔案並不會「重新連接」至其在 Azure 檔案共用的位置，進而導致同步錯誤。 另請注意，位在伺服器端點命名空間外部的階層式檔案可能會永久遺失。 即使從未啟用雲端階層，伺服器端點仍可能有階層式檔案存在。

#### <a name="optional-recall-all-tiered-data"></a>(選擇性) 重新叫用階層式資料
如果您希望在移除 Azure 檔案共用 (也就是說這不是測試環境，而是生產環境) 之後仍可使用檔案目前的分層，請在包含伺服器端點的每個磁碟區上回收所有檔案。 停用所有伺服器端點的雲端階層，然後執行下列 PowerShell Cmdlet：

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <a-volume-with-server-endpoints-on-it>
```

> [!Warning]  
> 如果裝載伺服器端點的本機磁碟區沒有足以回收所有已分層資料的可用空間，`Invoke-StorageSyncFileRecall` Cmdlet 將會失敗。  

#### <a name="remove-the-server-from-all-sync-groups"></a>從所有同步群組中移除伺服器
在儲存體同步服務上將伺服器取消註冊之前，必須先移除該伺服器上的所有伺服器端點。 這可以透過 Azure 入口網站來完成：

1. 瀏覽至您的伺服器註冊所在的儲存體同步服務。
2. 在儲存體同步服務的每個同步群組中，移除此伺服器的所有伺服器端點。 這可以透過以滑鼠右鍵按一下 [同步群組] 窗格中的相關伺服器端點來完成。

    ![從同步群組移除伺服器端點](media/storage-sync-files-server-registration/sync-group-server-endpoint-remove-1.png)

這也可以透過簡單的 PowerShell 指令碼來完成：

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll"

$accountInfo = Connect-AzureRmAccount
Login-AzureRmStorageSync -SubscriptionId $accountInfo.Context.Subscription.Id -TenantId $accountInfo.Context.Tenant.Id -ResourceGroupName "<your-resource-group>"

$StorageSyncService = "<your-storage-sync-service>"

Get-AzureRmStorageSyncGroup -StorageSyncServiceName $StorageSyncService | ForEach-Object { 
    $SyncGroup = $_; 
    Get-AzureRmStorageSyncServerEndpoint -StorageSyncServiceName $StorageSyncService -SyncGroupName $SyncGroup.Name | Where-Object { $_.DisplayName -eq $env:ComputerName } | ForEach-Object { 
        Remove-AzureRmStorageSyncServerEndpoint -StorageSyncServiceName $StorageSyncService -SyncGroupName $SyncGroup.Name -ServerEndpointName $_.Name 
    } 
}
```

#### <a name="unregister-the-server"></a>取消註冊伺服器
回收所有資料並從所有同步群組移除伺服器之後，即可將伺服器取消註冊。 

1. 在 Azure 入口網站中，瀏覽到儲存體同步服務的 [已註冊的伺服器] 區段。
2. 以滑鼠右鍵按一下您要取消註冊的伺服器，然後按一下 [取消註冊伺服器]。

    ![將伺服器取消註冊](media/storage-sync-files-server-registration/unregister-server-1.png)

## <a name="ensuring-azure-file-sync-is-a-good-neighbor-in-your-datacenter"></a>確認 Azure 檔案同步對於您的資料中心不會有不良影響 
因為資料中心中不只有 Azure 檔案同步這項服務，因此您可能會想要限制 Azure 檔案同步的網路和儲存體使用量。

> [!Important]  
> 設定的限制過低會影響 Azure 檔案同步的同步和回收效能。

### <a name="set-azure-file-sync-network-limits"></a>設定 Azure 檔案同步網路限制
您可以使用 `StorageSyncNetworkLimit` Cmdlet，節流 Azure 檔案同步的網路使用量。 

例如，您可以建立新的節流限制，以確保 Azure 檔案同步在工作日早上 9 點至下午 5 點 (17:00h) 之間不會使用超過 10 Mbps： 

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
New-StorageSyncNetworkLimit -Day Monday, Tuesday, Wednesday, Thursday, Friday -StartHour 9 -EndHour 17 -LimitKbps 10000
```

您可以使用下列 Cmdlet 來查看限制：

```PowerShell
Get-StorageSyncNetworkLimit # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

若要移除網路限制，請使用 `Remove-StorageSyncNetworkLimit`。 例如，下列命令會移除所有網路限制︰

```PowerShell
Get-StorageSyncNetworkLimit | ForEach-Object { Remove-StorageSyncNetworkLimit -Id $_.Id } # assumes StorageSync.Management.ServerCmdlets.dll is imported
```

### <a name="use-windows-server-storage-qos"></a>使用 Windows Server 儲存體服務品質 (QoS) 
當 Azure 檔案同步裝載於 Windows Server 虛擬主機上執行的虛擬機器時，您可以使用儲存體 QoS (儲存體服務品質) 來規範儲存體 IO 耗用量。 儲存體 QoS 原則可以設定為最大值 (或限制，如上述強制執行 StorageSyncNetwork 限制的方式) 或最小值 (或保留)。 設定最小值而不是最大值時，可讓 Azure 檔案同步在其他工作負載不使用時，盡可能使用可用的儲存體頻寬。 如需詳細資訊，請參閱[儲存體服務品質](https://docs.microsoft.com/windows-server/storage/storage-qos/storage-qos-overview)。

## <a name="see-also"></a>另請參閱
- [規劃 Azure 檔案同步部署](storage-sync-files-planning.md)
- [部署 Azure 檔案同步](storage-sync-files-deployment-guide.md) 
- [針對 Azure 檔案同步進行移難排解](storage-sync-files-troubleshoot.md)
