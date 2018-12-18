---
title: 部署 Azure 檔案同步 | Microsoft Docs
description: 了解部署 Azure 檔案同步的完整程序。
services: storage
author: wmgries
ms.service: storage
ms.topic: article
ms.date: 07/19/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: 261f8dfe41ece0cd56a4a71972e3142ef8440afb
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918104"
---
# <a name="deploy-azure-file-sync"></a>部署 Azure 檔案同步
使用 Azure 檔案同步，將組織的檔案共用集中在 Azure 檔案服務中，同時保有內部部署檔案伺服器的彈性、效能及相容性。 Azure 檔案同步會將 Windows Server 轉換成 Azure 檔案共用的快速快取。 您可以使用 Windows Server 上可用的任何通訊協定來從本機存取資料，包括 SMB、NFS 和 FTPS。 您可以視需要存取多個散佈於世界各地的快取。

強烈建議您先閱讀[規劃 Azure 檔案服務部署](storage-files-planning.md)和[規劃 Azure 檔案同步部署](storage-sync-files-planning.md)，再完成本文章中描述的步驟。

## <a name="prerequisites"></a>必要條件
* Azure 儲存體帳戶和 Azure 檔案共用，須位於您要部署 Azure 檔案同步的區域中。如需詳細資訊，請參閱
    - Azure 檔案同步的[區域可用性](storage-sync-files-planning.md#region-availability)。
    - 如需如何建立儲存體帳戶的逐步說明，請參閱[建立儲存體帳戶](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。
    - [建立檔案共用](storage-how-to-create-file-share.md)以取得如何建立檔案共用的逐步說明。
* 至少有一個 Windows Server 或 Windows Server 叢集的受支援執行個體，以與 Azure 檔案同步進行同步處理。如需 Windows Server 受支援版本的詳細資訊，請參閱[與 Windows Server 的互通性](storage-sync-files-planning.md#azure-file-sync-system-requirements-and-interoperability)。
* 確定 PowerShell 5.1 已安裝在您的 Windows Server 上。 如果是使用 Windows Server 2012 R2，請確定至少執行 PowerShell 5.1\*。 因為 PowerShell 5.1 是預設的現成版本，所以您可以在 Windows Server 2016 上安全地略過這項檢查。 在 Windows Server 2012 R2 上，您可以查看 **$PSVersionTable** 物件的 **PSVersion** 屬性值，確定執行的是否為 PowerShell 5.1.\*：

    ```PowerShell
    $PSVersionTable.PSVersion
    ```

    如果您的 PSVersion 值小於 5.1.\* (大部分的 Windows Server 2012 R2 全新安裝都會發生此類情況)，您可以下載並安裝 [Windows Management Framework (WMF) 5.1](https://www.microsoft.com/download/details.aspx?id=54616) \(英文\) 來輕鬆地升級。 適用於 Windows Server 2012 R2 下載和安裝的適當套件為 **Win8.1AndW2K12R2-KB\*\*\*\*\*\*\*-x64.msu**。

    > [!Note]  
    > Azure 檔案同步在 Windows Server 2012 R2 或 Windows Server 2016 上尚未支援 PowerShell 6+。
* 要與 Azure 檔案同步搭配使用的伺服器上須有 AzureRM PowerShell 模組。如需如何安裝 AzureRM PowerShell 模組的詳細資訊，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。 一律建議使用最新版的 Azure PowerShell 模組。 

## <a name="prepare-windows-server-to-use-with-azure-file-sync"></a>準備 Windows Server 以搭配 Azure 檔案同步使用
針對要與 Azure 檔案同步搭配使用的每個伺服器 (包括容錯移轉叢集中的每個伺服器節點)，停用 [Internet Explorer 增強式安全性設定]。 此動作只需要在初始伺服器註冊時執行。 您可以在註冊伺服器後重新啟用它。

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
1. 開啟 [伺服器管理員]。
2. 按一下 [本機伺服器]：  
    ![位於伺服器管理員 UI 左側的 [本機伺服器]](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
3. 選取 [屬性] 子窗格上的 [IE 增強式安全性設定] 連結。  
    ![位於伺服器管理員 UI 中的 [IE 增強式安全性設定] 窗格](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
4. 在 [Internet Explorer 增強式安全性設定] 對話方塊中，針對 [系統管理員] 和 [使用者] 選取 [關閉]：  
    ![已選取 [關閉] 的 [Internet Explorer 增強式安全性設定] 快顯視窗](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
若要停用 [Internet Explorer 增強式安全性設定]，請從提升權限的 PowerShell 工作階段中執行下列命令：

```PowerShell
# Disable Internet Explorer Enhanced Security Configuration 
# for Administrators
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force

# Disable Internet Explorer Enhanced Security Configuration 
# for Users
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force

# Force Internet Explorer closed, if open. This is required to fully apply the setting.
# Save any work you have open in the IE browser. This will not affect other browsers,
# including Edge.
Stop-Process -Name iexplore -ErrorAction SilentlyContinue
``` 

---

## <a name="install-the-azure-file-sync-agent"></a>安裝 Azure 檔案同步代理程式
Azure 檔案同步代理程式是可下載的套件，可讓 Windows Server 能夠和 Azure 檔案共用進行同步處理。 

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
您可以從 [Microsoft 下載中心](https://go.microsoft.com/fwlink/?linkid=858257) \(英文\) 下載該代理程式。 下載之後，請按兩下 MSI 套件以啟動 Azure 檔案同步代理程式安裝。

> [!Important]  
> 如果您打算將 Azure 檔案同步搭配容錯移轉叢集使用，則必須在叢集中的每個節點上安裝 Azure 檔案同步代理程式。 叢集中的每個節點都必須經過註冊，才能使用 Azure 檔案同步。

建議您採取下列動作︰
- 保留預設的安裝路徑 (C:\Program Files\Azure\StorageSyncAgent)，以簡化疑難排解和伺服器維護。
- 啟用 Microsoft Update 以將 Azure 檔案同步保持在最新狀態。 Azure 檔案同步代理程式的所有更新 (包括功能更新和 Hotfix) 都會從 Microsoft Update 進行。 建議安裝最新的 Azure 檔案同步更新。如需詳細資訊，請參閱 [Azure 檔案同步更新原則](storage-sync-files-planning.md#azure-file-sync-agent-update-policy)。

當 Azure 檔案同步代理程式安裝結束時，伺服器註冊 UI 會自動開啟。 您必須先有儲存體同步服務才能註冊；請參閱下一節以了解如何建立儲存體同步服務。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
執行下列 PowerShell 程式碼，下載適合您作業系統的 Azure 檔案同步代理程式版本，並將其安裝在您的系統上。

> [!Important]  
> 如果您打算將 Azure 檔案同步搭配容錯移轉叢集使用，則必須在叢集中的每個節點上安裝 Azure 檔案同步代理程式。 叢集中的每個節點都必須經過註冊，才能使用 Azure 檔案同步。

```PowerShell
# Gather the OS version
$osver = [System.Environment]::OSVersion.Version

# Download the appropriate version of the Azure File Sync agent for your OS.
if ($osver.Equals([System.Version]::new(10, 0, 14393, 0))) {
    Invoke-WebRequest `
        -Uri https://go.microsoft.com/fwlink/?linkid=875004 `
        -OutFile "StorageSyncAgent.exe" 
}
elseif ($osver.Equals([System.Version]::new(6, 3, 9600, 0))) {
    Invoke-WebRequest `
        -Uri https://go.microsoft.com/fwlink/?linkid=875002 `
        -OutFile "StorageSyncAgent.exe" 
}
else {
    throw [System.PlatformNotSupportedException]::new("Azure File Sync is only supported on Windows Server 2012 R2 and Windows Server 2016")
}

# Extract the MSI from the install package
$tempFolder = New-Item -Path "afstemp" -ItemType Directory
Start-Process -FilePath ".\StorageSyncAgent.exe" -ArgumentList "/C /T:$tempFolder" -Wait

# Install the MSI. Start-Process is used to PowerShell blocks until the operation is complete.
# Note that the installer currently forces all PowerShell sessions closed - this is a known issue.
Start-Process -FilePath "$($tempFolder.FullName)\StorageSyncAgent.msi" -ArgumentList "/quiet" -Wait

# Note that this cmdlet will need to be run in a new session based on the above comment.
# You may remove the temp folder containing the MSI and the EXE installer
Remove-Item -Path ".\StorageSyncAgent.exe", ".\afstemp" -Recurse -Force
```

---

## <a name="deploy-the-storage-sync-service"></a>部署儲存體同步服務 
部署 Azure 檔案同步的第一步，先將**儲存體同步服務**資源放到所選訂用帳戶的資源群組中。 建議您佈建需要的少數資源群組即可。 您將在伺服器與此資源之間建立信任關係，而一個伺服器只能註冊到一個儲存體同步服務。 因此，建議您根據需要部署多個儲存體同步服務，以分隔伺服器群組。 請記住，不同儲存體同步服務中的伺服器不可以彼此同步。

> [!Note]
> 部署至訂用帳戶和資源群組的儲存體同步服務會從訂用帳戶和資源群組中繼承存取權限。 我們建議您仔細檢查誰可以存取此服務。 具有寫入存取權的實體可以開始將新的一組檔案從已註冊伺服器同步到此儲存體同步服務，並讓資料流向實體可存取的 Azure 儲存體。

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
若要部署儲存體同步服務，請移至 [Azure 入口網站](https://portal.azure.com/)，按一下 [新增] 並搜尋 Azure 檔案同步。在搜尋結果中，選取 [Azure 檔案同步]，然後選取 [建立] 以開啟 [部署儲存體同步] 索引標籤。

在開啟的窗格中，輸入下列資訊：

- **名稱**：儲存體同步服務的唯一名稱 (每一訂用帳戶)。
- **訂用帳戶**：您要在其中建立儲存體同步服務的訂用帳戶。 視您組織的設定策略而定，您可能具有一或多個訂用帳戶的存取權限。 Azure 訂用帳戶是針對每個雲端服務 (例如 Azure 檔案服務) 計費的最基本容器。
- **資源群組**：資源群組是 Azure 資源的邏輯群組，例如儲存體帳戶或儲存體同步服務。 針對 Azure 檔案同步，您可以建立新的資源群組或使用現有的資源群組。(建議使用資源群組作為容器，來以邏輯方式為您的組織隔離資源，例如將 HR 資源或適用於特定專案的資源組成群組)。
- **位置**：您要在其中部署 Azure 檔案同步的區域。只有支援的區域會列在此清單中。

完成時，請選取 [建立] 來部署儲存體同步服務。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
與 Azure 檔案同步管理 Cmdlet 互動之前，您必須匯入 DLL 並建立 Azure 檔案同步管理內容。 這是必要的，因為 Azure 檔案同步管理 Cmdlet 還不是 AzureRM PowerShell 模組的一部分。

> [!Note]  
> 在包含 Azure 檔案同步管理 Cmdlet 的 StorageSync.Management.PowerShell.Cmdlets.dll 套件中，其中的 Cmdlet (刻意地) 具有未經核准的動詞命令 (`Login`)。 已選擇 `Login-AzureRmStorageSync` 名稱來符合 AzureRM PowerShell 模組中的 `Login-AzureRmAccount` Cmdlet 別名。 系統將會移除此錯誤訊息 (和 Cmdlet)，Azure 檔案同步代理程式會新增至 AzureRM PowerShell 模組。

```PowerShell
$acctInfo = Login-AzureRmAccount

# The location of the Azure File Sync Agent. If you have installed the Azure File Sync 
# agent to a non-standard location, please update this path.
$agentPath = "C:\Program Files\Azure\StorageSyncAgent"

# Import the Azure File Sync management cmdlets
Import-Module "$agentPath\StorageSync.Management.PowerShell.Cmdlets.dll"

# this variable stores your subscription ID 
# get the subscription ID by logging onto the Azure portal
$subID = $acctInfo.Context.Subscription.Id

# this variable holds your Azure Active Directory tenant ID
# use Login-AzureRMAccount to get the ID from that context
$tenantID = $acctInfo.Context.Tenant.Id

# this variable holds the Azure region you want to deploy 
# Azure File Sync into
$region = '<Az_Region>'

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = @()
Get-AzureRmLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the selected Azure Region or the region is mistyped.")
}

# the resource group to deploy the Storage Sync Service into
$resourceGroup = '<RG_Name>'

# Check to ensure resource group exists and create it if doesn't
$resourceGroups = @()
Get-AzureRmResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    New-AzureRmResourceGroup -Name $resourceGroup -Location $region
}

# the following command creates an AFS context 
# it enables subsequent AFS cmdlets to be executed with minimal 
# repetition of parameters or separate authentication 
Login-AzureRmStorageSync `
    –SubscriptionId $subID `
    -ResourceGroupName $resourceGroup `
    -TenantId $tenantID `
    -Location $region
```

一旦使用 `Login-AzureRmStorageSync` Cmdlet 建立 Azure 檔案同步內容後，您就可以建立儲存體同步服務。 請務必以所需的儲存體同步服務名稱取代 `<my-storage-sync-service>`。

```PowerShell
$storageSyncName = "<my-storage-sync-service>"
New-AzureRmStorageSyncService -StorageSyncServiceName $storageSyncName
```

---

## <a name="register-windows-server-with-storage-sync-service"></a>向儲存體同步服務註冊 Windows Server
向儲存體同步服務註冊 Windows Server 會在您的伺服器 (或叢集) 與儲存體同步服務之間建立信任關係。 一部伺服器只能向一個儲存體同步服務註冊，但可以與其他關聯至相同儲存體同步服務的伺服器和 Azure 檔案共用同步。

> [!Note]
> 伺服器註冊會使用您的 Azure 認證來建立儲存體同步服務和 Windows Server 之間的信任關係；但是，之後伺服器會建立並使用自有的身分識別，只要伺服器維持註冊狀態，且目前的共用存取簽章權杖 (儲存體 SAS) 有效，此身分識別就會是有效的。 如果伺服器未註冊，新的 SAS 權杖就無法發行至伺服器，並會因此移除伺服器存取您 Azure 檔案共用的能力，進而停止任何同步。

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
安裝 Azure 檔案同步代理程式之後，伺服器註冊 UI 應該會自動開啟。 如果沒有，您可以從其檔案位置手動開啟它：C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe。 伺服器註冊 UI 開啟時，請選取 [登入] 以開始。

登入之後，系統會提示您輸入下列資訊：

![伺服器註冊 UI 的螢幕擷取畫面](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Azure 訂用帳戶**：包含儲存體同步服務的訂用帳戶 (請參閱[部署儲存體同步服務](#deploy-the-storage-sync-service))。 
- **資源群組**：包含儲存體同步服務的資源群組。
- **儲存體同步服務**：您要註冊的目標儲存體同步服務名稱。

選取適當的資訊之後，請選取 [註冊] 以完成伺服器註冊。 在註冊過程中，系統會提示您額外再登入一次。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
```PowerShell
$registeredServer = Register-AzureRmStorageSyncServer -StorageSyncServiceName $storageSyncName
```

---

## <a name="create-a-sync-group-and-a-cloud-endpoint"></a>建立同步群組和雲端端點
同步群組定義一組檔案的同步拓撲。 同步群組內的端點會與彼此保持同步。 同步群組必須包含一個雲端端點，代表 Azure 檔案共用和一個或多個伺服器端點。 伺服器端點代表已註冊伺服器上的路徑。 伺服器可以在多個同步群組中有多個伺服器端點。 您可以根據需要建立多個同步群組，以適當地描述您想要的同步拓撲。

雲端端點是指向 Azure 檔案共用的指標。 所有伺服器端點都會與雲端端點同步，使雲端端點成為中樞。 Azure 檔案共用的儲存體帳戶必須位於儲存體同步服務的所在區域。 Azure 檔案共用的全部內容都會同步，但有一個例外狀況：系統會佈建一個特殊資料夾，其相當於 NTFS 磁碟區上隱藏的「系統磁碟區資訊」資料夾。 此目錄稱為 ".SystemShareInformation"。 它包含重要的同步中繼資料，而這不會同步到其他端點。 請勿使用或刪除此資料夾！

> [!Important]  
> 您可以對同步群組中的任何雲端端點或伺服器端點進行變更，您的檔案將會與同步群組中的其他端點同步。 如果直接對雲端端點 (Azure 檔案共用) 進行變更，則必須先由 Azure 檔案同步變更偵測作業探索到該變更。 針對雲端端點的變更偵測作業，每隔 24 小時才會起始一次。 如需詳細資訊，請參閱 [Azure 檔案服務常見問題集](storage-files-faq.md#afs-change-detection)。

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
若要建立同步群組，請在 [Azure 入口網站](https://portal.azure.com/)中移至您的儲存體同步服務，然後選取 [+ 同步群組]：

![在 Azure 入口網站中建立新的同步群組](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

在開啟的窗格中，輸入下列資訊以建立具有雲端端點的同步群組：

- **同步群組名稱**：要建立的同步群組名稱。 此名稱在儲存體同步服務中必須是唯一的，但可以是任何對您而言合理的名稱。
- **訂用帳戶**：您在[部署儲存體同步服務](#deploy-the-storage-sync-service)中部署儲存體同步服務的訂用帳戶。
- **儲存體帳戶**：如果您選取 [選取儲存體帳戶]，則會顯示另一個窗格，您可在其中選取具有您想要同步之 Azure 檔案共用的儲存體帳戶。
- **Azure 檔案共用**：您要同步處理的 Azure 檔案共用名稱。

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
若要建立同步群組，請執行下列 PowerShell。 請記得以所需的同步群組名稱取代 `<my-sync-group>`。

```PowerShell
$syncGroupName = "<my-sync-group>"
New-AzureRmStorageSyncGroup -SyncGroupName $syncGroupName -StorageSyncService $storageSyncName
```

一旦成功建立同步群組之後，您就可以建立雲端端點。 請務必將 `<my-storage-account>` 和 `<my-file-share>` 取代為預期的值。

```PowerShell
# Get or create a storage account with desired name
$storageAccountName = "<my-storage-account>"
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroup | Where-Object {
    $_.StorageAccountName -eq $storageAccountName
}

if ($storageAccount -eq $null) {
    $storageAccount = New-AzureRmStorageAccount `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup `
        -Location $region `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly:$true
}

# Get or create an Azure file share within the desired storage account
$fileShareName = "<my-file-share>"
$fileShare = Get-AzureStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    $fileShare = New-AzureStorageShare -Context $storageAccount.Context -Name $fileShareName
}

# Create the cloud endpoint
New-AzureRmStorageSyncCloudEndpoint `
    -StorageSyncServiceName $storageSyncName `
    -SyncGroupName $syncGroupName ` 
    -StorageAccountResourceId $storageAccount.Id
    -StorageAccountShareName $fileShare.Name
```

---

## <a name="create-a-server-endpoint"></a>建立伺服器端點
伺服器端點代表已註冊伺服器上的特定位置，例如伺服器磁碟區上的資料夾。 伺服器端點必須是已註冊伺服器 (不是裝載共用) 上的路徑，而且若要使用雲端階層，此路徑必須位在非系統磁碟區上。 不支援網路連接儲存裝置 (NAS)。

# <a name="portaltabportal"></a>[入口網站](#tab/portal)
若要新增伺服器端點，請移至新建立的同步群組，然後選取 [新增伺服器端點]。

![在 [同步群組] 窗格中新增新的伺服器端點](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

在 [新增伺服器端點] 窗格中，輸入下列資訊以建立伺服器端點：

- **已註冊的伺服器**：您要建立伺服器端點的伺服器或叢集名稱。
- **路徑**：要隨著同步群組同步處理的 Windows Server 路徑。
- **雲端階層處理**：啟用或停用雲端階層處理的開關。 透過雲端階層處理，可將不常使用或存取的檔案分層處理至 Azure 檔案服務。
- **磁碟區可用空間**：在伺服器端點所在磁碟區上要保留的可用空間數量。 例如，如果具有單一伺服器端點之磁碟區上的磁碟區可用空間是設定為 50%，則大約有一半的資料量會分層處理至 Azure 檔案服務。 無論雲端階層處理是否啟用，您的 Azure 檔案共用在同步群組中一律會有完整的資料複本。

若要新增伺服器端點，請選取 [建立]。 您的檔案現在會在 Azure 檔案共用和 Windows Server 之間保持同步。 

# <a name="powershelltabpowershell"></a>[PowerShell](#tab/powershell)
執行下列 PowerShell 命令以建立伺服器端點，並務必以所需的值取代 `<your-server-endpoint-path>` 和 `<your-volume-free-space>`。

```PowerShell
$serverEndpointPath = "<your-server-endpoint-path>"
$cloudTieringDesired = $true
$volumeFreeSpacePercentage = <your-volume-free-space>

if ($cloudTieringDesired) {
    # Ensure endpoint path is not the system volume
    $directoryRoot = [System.IO.Directory]::GetDirectoryRoot($serverEndpointPath)
    $osVolume = "$($env:SystemDrive)\"
    if ($directoryRoot -eq $osVolume) {
        throw [System.Exception]::new("Cloud tiering cannot be enabled on the system volume")
    }

    # Create server endpoint
    New-AzureRmStorageSyncServerEndpoint `
        -StorageSyncServiceName $storageSyncName `
        -SyncGroupName $syncGroupName `
        -ServerId $registeredServer.Id `
        -ServerLocalPath $serverEndpointPath `
        -CloudTiering $true `
        -VolumeFreeSpacePercent $volumeFreeSpacePercentage
}
else {
    # Create server endpoint
    New-AzureRmStorageSyncServerEndpoint `
        -StorageSyncServiceName $storageSyncName `
        -SyncGroupName $syncGroupName `
        -ServerId $registeredServer.Id `
        -ServerLocalPath $serverEndpointPath 
}
```

---

## <a name="onboarding-with-azure-file-sync"></a>透過 Azure 檔案同步上架
第一次於 Azure 檔案同步上架時，如果不想要停機，同時還要保留完整檔案忠實性和存取控制清單 (ACL)，其建議步驟如下所示：
 
1. 部署儲存體同步服務。
2. 建立同步群組。
3. 在具有完整資料集的伺服器上安裝 Azure 檔案同步代理程式。
4. 註冊該伺服器，並在共用上建立伺服器端點。 
5. 讓同步服務將所有資料完整上傳至 Azure 檔案共用 (雲端端點)。  
6. 完成初始上傳之後，在其餘的各個伺服器上安裝 Azure 檔案同步代理程式。
7. 在其餘的各個伺服器上建立新的檔案共用。
8. 如有需要，請使用雲端階層處理原則，在新的檔案共用上建立伺服器端點。 (此步驟需有額外的儲存體以進行初始設定。)
9. 讓 Azure 檔案同步代理程式快速還原完整命名空間，而不需實際的資料傳輸。 在完整命名空間同步之後，同步引擎將會根據伺服器端點的雲端階層處理原則，填滿本機磁碟空間。 
10. 確定同步完成，並視需要測試拓撲。 
11. 將使用者和應用程式重新導向到這個新共用。
12. 您可以選擇性地刪除伺服器上任何重複的共用。
 
如果您沒有額外儲存體可供初始上架，而想要連結至現有的共用，您可以在 Azure 檔案共用中預先植入資料。 唯有您可接受停機時間，且絕對保證在初始上架程序期間，伺服器共用上的資料不會變更，才建議使用此方法。 
 
1. 確保在上架程序期間，任何伺服器上的資料都無法變更。
2. 透過 SMB 使用任何資料傳輸工具 (例如 Robocopy、直接的 SMB 複製)，以伺服器資料預先植入 Azure 檔案共用。 因為 AzCopy 不會透過 SMB 上傳資料，所以無法用於預先植入。
3. 使用指向現有共用的所需伺服器端點，建立 Azure 檔案同步拓撲。
4. 讓同步服務在所有端點上完成調整程序。 
5. 完成調整後，您可以開啟共用進行變更。
 
請留意目前預先植入的方法有一些限制： 
- 檔案不會保持完整不失真。 例如，檔案會失去 ACL 和時間戳記。
- 在同步拓撲完全啟動並執行之前，伺服器上的資料變更會導致伺服器端點發生衝突。  
- 建立雲端端點之後，Azure 檔案同步會先執行在雲端偵測檔案的程序，再開始初始同步處理。完成此程序所需的時間取決於各種因素，例如網路速度、可用頻寬，以及檔案和資料夾的數目。 根據預覽版本中的粗略估計，偵測程序會以大約每秒 10 個檔案的速度執行。因此，即使預先植入的執行速度快，但若在雲端預先植入資料，則系統完整執行的整體時間可能會更長。

## <a name="migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync"></a>將 DFS 複寫 (DFS-R) 部署移轉至 Azure 檔案同步
若要將 DFS-R 部署移轉至 Azure 檔案同步處理：

1. 建立同步群組來代表要取代的 DFS-R 拓樸。
2. 在要移轉 DFS R 拓撲中，於有一組完整資料的伺服器上啟動。 在該伺服器上安裝 Azure 檔案同步。
3. 註冊該伺服器，並為要移轉的第一個伺服器建立伺服器端點。 請勿啟用雲端階層。
4. 將所有資料同步至 Azure 檔案共用 (雲端端點)。
5. 在其餘的各個 DFS-R 伺服器上安裝並註冊 Azure 檔案同步代理程式。
6. 停用 DFS-R。 
7. 在每一個 DFS-R 伺服器上建立伺服器端點。 請勿啟用雲端階層。
8. 確定同步完成，並視需要測試拓撲。
9. 淘汰 DFS-R。
10. 現在可在所需的任何伺服器端點上啟用雲端階層。

如需詳細資訊，請參閱[Azure 檔案同步與分散式檔案系統 (DFS) 互通](storage-sync-files-planning.md#distributed-file-system-dfs)。

## <a name="next-steps"></a>後續步驟
- [新增或移除 Azure 檔案同步伺服器端點](storage-sync-files-server-endpoint.md)
- [向 Azure 檔案同步註冊或取消註冊伺服器](storage-sync-files-server-registration.md)
