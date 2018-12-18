---
title: 使用 Azure PowerShell 管理 Azure 檔案共用
description: 了解如何使用 Azure PowerShell 管理 Azure 檔案共用。
services: storage
author: wmgries
ms.service: storage
ms.topic: get-started-article
ms.date: 03/26/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: 4964b6d531d777ea5080e51fbff5a589efd5249d
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386863"
---
# <a name="managing-azure-file-shares-with-azure-powershell"></a>使用 Azure PowerShell 管理 Azure 檔案共用 
[Azure 檔案](storage-files-introduction.md)是 Microsoft 易於使用的雲端檔案系統。 Azure 檔案共用可在 Windows、Linux 和 macOS 中掛接。 本指南會逐步說明透過 PowerShell 來使用 Azure 檔案共用的基本概念。 在本文中，您將了解如何：

> [!div class="checklist"]
> * 建立資源群組和儲存體帳戶
> * 建立 Azure 檔案共用 
> * 建立目錄
> * 上傳檔案 
> * 下載檔案
> * 建立及使用共用快照集

如果您沒有 Azure 訂用帳戶，您可以在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果想要在本機安裝和使用 PowerShell，則在執行本指南時，您必須使用 Azure PowerShell 模組 5.1.1 版或更新版本。 若要確認您所執行的 Azure PowerShell 模組版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

## <a name="create-a-resource-group"></a>建立資源群組
資源群組是在其中部署與管理 Azure 資源的邏輯容器。 如果您還沒有 Azure 資源群組，您可以使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) Cmdlet 新建一個。 

下列範例會在美國東部區域建立名為 *myResourceGroup* 的資源群組：

```azurepowershell-interactive
New-AzureRmResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶
儲存體帳戶是可用來部署 Azure 檔案共用或其他儲存體資源 (例如 Blob 或佇列) 的共用儲存體集區。 儲存體帳戶可包含無限制數目的共用，而共用可儲存無限制數目的檔案，最多可達儲存體帳戶的容量限制。

此範例會建立名為 `mystorageacct<random number>` 的儲存體帳戶，並在變數 **$storageAcct** 中放入該儲存體帳戶的參考。 儲存體帳戶名稱必須是唯一的，因此，請使用 `Get-Random` 為名稱附加一個數字，使其成為唯一名稱。 

```azurepowershell-interactive 
$storageAcct = New-AzureRmStorageAccount `
                  -ResourceGroupName "myResourceGroup" `
                  -Name "mystorageacct$(Get-Random)" `
                  -Location eastus `
                  -SkuName Standard_LRS 
```

## <a name="create-an-azure-file-share"></a>建立 Azure 檔案共用
現在，您可以建立第一個 Azure 檔案共用。 您可以使用 [New-AzureStorageShare](https://docs.microsoft.com/powershell/module/servicemanagement/azure.storage/new-azurestorageshare) Cmdlet 來建立檔案共用。 此範例會建立名為 `myshare` 的共用。

```azurepowershell-interactive
New-AzureStorageShare `
   -Name myshare `
   -Context $storageAcct.Context
```

> [!Important]  
> 共用名稱必須全部使用小寫字母、數字和單一連字號，但開頭不可以是連字號。 如需有關為檔案共用與檔案命名的完整詳細資料，請參閱 [命名和參考共用、目錄、檔案及中繼資料](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata)。

## <a name="work-with-the-contents-of-the-azure-file-share"></a>使用 Azure 檔案共用的內容
您現在已建立 Azure 檔案共用，並且可使用 SMB 將檔案共用掛接在 [Windows](storage-how-to-use-files-windows.md)、[Linux](storage-how-to-use-files-linux.md) 或 [macOS](storage-how-to-use-files-mac.md) 上。 或者，您可以使用 Azure PowerShell 模組來處理 Azure 檔案共用。 此做法會比使用 SMB 掛接檔案共用更有利，因為使用 PowerShell 提出的所有要求都會使用檔案 REST API 來執行，而讓您能夠從下列途徑建立、修改及刪除檔案共用中的檔案和目錄：

- PowerShell Cloud Shell (無法透過 SMB 掛接檔案共用)。
- 無法掛接 SMB 共用的用戶端，例如，未解除封鎖連接埠 445 的內部部署用戶端。
- 無伺服器案例，例如在 [Azure Functions](../../azure-functions/functions-overview.md) 中。 


### <a name="create-directory"></a>建立目錄
若要在 Azure 檔案共用的根目錄建立名為 myDirectory 的新目錄，請使用 [New-AzureStorageDirectory](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestoragedirectory) Cmdlet。


```azurepowershell-interactive
New-AzureStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName "myshare" `
   -Path "myDirectory"
```

### <a name="upload-a-file"></a>上傳檔案
為了示範如何使用 [Set-AzureStorageFileContent](/powershell/module/azure.storage/set-azurestoragefilecontent) Cmdlet 上傳檔案，我們必須先在 PowerShell Cloud Shell 的可用磁碟機內建立要上傳的檔案。 

此範例會將目前的日期和時間放入可用磁碟機上的新檔案中，然後將檔案上傳至檔案共用。

```azurepowershell-interactive
# this expression will put the current date and time into a new file on your scratch drive
Get-Date | Out-File -FilePath "C:\Users\ContainerAdministrator\CloudDrive\SampleUpload.txt" -Force

# this expression will upload that newly created file to your Azure file share
Set-AzureStorageFileContent `
   -Context $storageAcct.Context `
   -ShareName "myshare" `
   -Source "C:\Users\ContainerAdministrator\CloudDrive\SampleUpload.txt" `
   -Path "myDirectory\SampleUpload.txt"
```   

如果您要在本機執行 PowerShell，則應將 `C:\Users\ContainerAdministrator\CloudDrive\` 替換為存在於您的機器上的路徑。

上傳檔案之後，您可以使用 [Get-AzureStorageFile](/powershell/module/Azure.Storage/Get-AzureStorageFile) Cmdlet 來確認檔案已上傳至 Azure 檔案共用。 

```azurepowershell-interactive
Get-AzureStorageFile -Context $storageAcct.Context -ShareName "myshare" -Path "myDirectory" 
```

### <a name="download-a-file"></a>下載檔案
您可以使用 [Get-AzureStorageFileContent](/powershell/module/azure.storage/get-azurestoragefilecontent) Cmdlet，將您剛剛上傳的檔案複本下載到 Cloud Shell 的可用磁碟機。

```azurepowershell-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists because you've run this example before.
Remove-Item 
    `-Path "C:\Users\ContainerAdministrator\CloudDrive\SampleDownload.txt" `
     -Force `
     -ErrorAction SilentlyContinue

Get-AzureStorageFileContent `
    -Context $storageAcct.Context `
    -ShareName "myshare" `
    -Path "myDirectory\SampleUpload.txt" ` 
    -Destination "C:\Users\ContainerAdministrator\CloudDrive\SampleDownload.txt"
```

下載檔案之後，您可以使用 `Get-ChildItem` 來確認檔案已下載到 PowerShell Cloud Shell 的可用磁碟機。

```azurepowershell-interactive
Get-ChildItem -Path "C:\Users\ContainerAdministrator\CloudDrive"
``` 

### <a name="copy-files"></a>複製檔案
常見工作之一，是在不同的檔案共用之間複製檔案，或是對 (從) Azure Blob 儲存體容器複製檔案。 為了示範這項功能，您可以建立新的共用，並使用 [Start-AzureStorageFileCopy](/powershell/module/azure.storage/start-azurestoragefilecopy) Cmdlet 將您剛才上傳的檔案複製到這個新的共用。 

```azurepowershell-interactive
New-AzureStorageShare `
    -Name "myshare2" `
    -Context $storageAcct.Context
  
New-AzureStorageDirectory `
   -Context $storageAcct.Context `
   -ShareName "myshare2" `
   -Path "myDirectory2"

Start-AzureStorageFileCopy `
    -Context $storageAcct.Context `
    -SrcShareName "myshare" `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestShareName "myshare2" `
    -DestFilePath "myDirectory2\SampleCopy.txt" `
    -DestContext $storageAcct.Context
```

現在，當您列出新共用中的檔案時，您應該會看到您所複製的檔案。

```azurepowershell-interactive
Get-AzureStorageFile -Context $storageAcct.Context -ShareName "myshare2" -Path "myDirectory2" 
```

雖然 `Start-AzureStorageFileCopy` Cmdlet 很方便用來進行 Azure 檔案共用與 Azure Blob 儲存體容器之間的臨機操作檔案移動，但我們建議較大型的移動 (就移動的檔案數目和大小而言) 應使用 AzCopy。 深入了解[適用於 Windows 的 AzCopy](../common/storage-use-azcopy.md) 和[適用於 Linux 的 AzCopy](../common/storage-use-azcopy-linux.md)。 AzCopy 必須安裝於本機 - 它不適用於 Cloud Shell。 

## <a name="create-and-modify-share-snapshots"></a>建立及修改共用快照集
可以使用 Azure 檔案共用來執行的另一項實用工作，是建立共用快照集。 快照集會保留 Azure 檔案共用的時間點。 共用快照集類似於您可能已經很熟悉的下列作業系統技術：
- Windows 檔案系統的[磁碟區陰影複製服務 (VSS)](https://docs.microsoft.com/windows/desktop/VSS/volume-shadow-copy-service-portal)，例如 NTFS 和 ReFS
- Linux 系統的[邏輯磁碟區管理員 (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) 快照集
- macOS 的 [Apple 檔案系統 (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) 快照集。 

您可以對檔案共用 (使用 [Get-AzureStorageShare](/powershell/module/azure.storage/get-azurestorageshare) Cmdlet 擷取) 的 PowerShell 物件使用 `Snapshot` 方法，以建立共用的共用快照集。 

```azurepowershell-interactive
$share = Get-AzureStorageShare -Context $storageAcct.Context -Name "myshare"
$snapshot = $share.Snapshot()
```

### <a name="browse-share-snapshots"></a>瀏覽共用快照集
您可以將快照集參考 (`$snapshot`) 傳至 `Get-AzureStorageFile` Cmdlet 的 `-Share` 參數，以瀏覽共用快照集的內容。

```azurepowershell-interactive
Get-AzureStorageFile -Share $snapshot
```

### <a name="list-share-snapshots"></a>列出共用快照集
您可以使用下列命令，檢視您為共用建立的快照集清單。

```azurepowershell-interactive
Get-AzureStorageShare -Context $storageAcct.Context | Where-Object { $_.Name -eq "myshare" -and $_.IsSnapshot -eq $true }
```

### <a name="restore-from-a-share-snapshot"></a>從共用快照集還原
您可以使用我們先前使用的 `Start-AzureStorageFileCopy` 命令來還原檔案。 基於此快速入門的用途，我們會先刪除我們先前上傳的 `SampleUpload.txt` 的檔案，以便從快照集加以還原。

```azurepowershell-interactive
# Delete SampleUpload.txt
Remove-AzureStorageFile `
    -Context $storageAcct.Context `
    -ShareName "myshare" `
    -Path "myDirectory\SampleUpload.txt"

# Restore SampleUpload.txt from the share snapshot
Start-AzureStorageFileCopy `
    -SrcShare $snapshot `
    -SrcFilePath "myDirectory\SampleUpload.txt" `
    -DestContext $storageAcct.Context `
    -DestShareName "myshare" `
    -DestFilePath "myDirectory\SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>刪除共用快照集
您可以使用 [Remove-AzureStorageShare](/powershell/module/azure.storage/remove-azurestorageshare) Cmdlet，並指定包含 `-Share` 參數之 `$snapshot` 參考的變數，來刪除共用快照集。

```azurepowershell-interactive
Remove-AzureStorageShare -Share $snapshot
```

## <a name="clean-up-resources"></a>清除資源
完成作業後，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) Cmdlet 移除資源群組和所有相關資源。 

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup
```

或者，您可以逐一移除資源：

- 以移除我們為此快速入門建立的 Azure 檔案共用。
```azurepowershell-interactive
Get-AzureStorageShare -Context $storageAcct.Context | Where-Object { $_.IsSnapshot -eq $false } | ForEach-Object { 
    Remove-AzureStorageShare -Context $storageAcct.Context -Name $_.Name
}
```

- 以移除儲存體帳戶本身 (這表示將會移除我們建立的 Azure 檔案共用，以及您已建立的任何其他儲存體資源，例如 Azure Blob 儲存體容器)。
```azurepowershell-interactive
Remove-AzureRmStorageAccount -ResourceGroupName $storageAcct.ResourceGroupName -Name $storageAcct.StorageAccountName
```

## <a name="next-steps"></a>後續步驟
- [使用 Azure 入口網站管理檔案共用](storage-how-to-use-files-portal.md)
- [使用 Azure CLI 管理檔案共用](storage-how-to-use-files-cli.md)
- [使用儲存體總管來管理檔案共用](storage-how-to-use-files-storage-explorer.md)
- [規劃 Azure 檔案部署](storage-files-planning.md)
