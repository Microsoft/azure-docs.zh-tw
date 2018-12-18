---
title: Azure 快速入門 - 使用 Azure PowerShell 在物件儲存體中建立 Blob | Microsoft Docs
description: 在本快速入門中，您會在物件 (Blob) 儲存體中使用 Azure PowerShell。 然後使用 PowerShell，將 blob 上傳至 Azure 儲存體、下載 blob，以及列出容器中的 blob。
services: storage
author: roygara
ms.custom: mvc
ms.service: storage
ms.topic: quickstart
ms.date: 04/09/2018
ms.author: rogarana
ms.openlocfilehash: b482379c05133dcf58e54bd01f38f0c3cee95e8d
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39398588"
---
# <a name="quickstart-upload-download-and-list-blobs-using-azure-powershell"></a>快速入門：使用 Azure PowerShell 上傳、下載及列出 Blob

Azure PowerShell 模組用於從 PowerShell 命令列或在指令碼中建立和管理 Azure 資源。 本指南詳細說明如何使用 PowerShell 在本機磁碟和 Azure Blob 儲存體之間傳輸檔案。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

本快速入門需要 Azure PowerShell 模組 3.6 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。

[!INCLUDE [storage-quickstart-tutorial-intro-include-powershell](../../../includes/storage-quickstart-tutorial-intro-include-powershell.md)]

## <a name="create-a-container"></a>建立容器

Blob 一律會上傳到容器中。 您可以組織 Blob 群組，方式如同在電腦的資料夾中組織檔案。

設定容器名稱，然後使用 [New-AzureStorageContainer](/powershell/module/azure.storage/new-azurestoragecontainer) 建立容器，並將權限設定為 'blob' 以允許檔案的公用存取。 此範例中的容器名稱是 *quickstartblobs*。

```powershell
$containerName = "quickstartblobs"
New-AzureStorageContainer -Name $containerName -Context $ctx -Permission blob
```

## <a name="upload-blobs-to-the-container"></a>將 Blob 上傳到容器

Blob 儲存體支援區塊 Blob、附加 Blob 和分頁 Blob。 用來備份 IaaS VM 的 VHD 檔案是分頁 Blob。 附加 Blob 用於記錄，例如當您想要寫入檔案，並繼續新增更多資訊時。 儲存在 Blob 儲存體中的大部分檔案都是區塊 Blob。 

若要將檔案上傳至區塊 Blob，請取得容器參考，然後取得該容器中區塊 Blob 的參考。 取得 Blob 參考之後，即可使用 [Set-AzureStorageBlobContent](/powershell/module/azure.storage/set-azurestorageblobcontent) 將資料上傳給它。 如果沒有 Blob，此作業會建立 Blob，如已存在，則予以覆寫。

下列範例會將本機磁碟上 D:\\_TestImages 資料夾中的 Image001.jpg 和 Image002.png 上傳至您所建立的容器。

```powershell
# upload a file
Set-AzureStorageBlobContent -File "D:\_TestImages\Image001.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 

# upload another file
Set-AzureStorageBlobContent -File "D:\_TestImages\Image002.png" `
  -Container $containerName `
  -Blob "Image002.png" `
  -Context $ctx
```

請上傳您所需的檔案數量，再繼續進行。

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob

使用 [Get-AzureStorageBlob](/powershell/module/azure.storage/get-azurestorageblob) 取得容器中的 Blob 清單。 此範例只會顯示已上傳之 Blob 的名稱。

```powershell
Get-AzureStorageBlob -Container $ContainerName -Context $ctx | select Name 
```

## <a name="download-blobs"></a>下載 Blob

將 blob 下載到本機磁碟。 若要下載每個 Blob，請設定名稱並呼叫 [Get-AzureStorageBlobContent](/powershell/module/azure.storage/get-azurestorageblobcontent) 以下載 Blob。

此範例會將 Blob 下載到本機磁碟上的 D:\\_TestImages\Downloads。 

```powershell
# download first blob
Get-AzureStorageBlobContent -Blob "Image001.jpg" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 

# download another blob
Get-AzureStorageBlobContent -Blob "Image002.png" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 
```

## <a name="data-transfer-with-azcopy"></a>使用 AzCopy 進行資料轉送

[AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 公用程式是適用於 Azure 儲存體的高效能可編寫指令碼資料轉送的另一個選項。 您可以使用 AzCopy 在 Blob、檔案和表格儲存體之間傳輸資料。

這裡提供一個快速範例，從 PowerShell 視窗利用 AzCopy 命令將稱為 *myfile.txt* 的檔案上傳至 *mystoragecontainer* 容器。

```PowerShell
./AzCopy `
    /Source:C:\myfolder `
    /Dest:https://mystorageaccount.blob.core.windows.net/mystoragecontainer `
    /DestKey:<storage-account-access-key> `
    /Pattern:"myfile.txt"
```

## <a name="clean-up-resources"></a>清除資源

移除您已建立的所有資產。 最簡單的做法是刪除資源群組。 這會同時刪除群組內含的所有資源。 在本例中，它會移除儲存體帳戶和資源群組本身。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何在本機磁碟和 Azure Blob 儲存體之間傳輸檔案。 若要深入了解如何透過 PowerShell 使用 Blob 儲存體，請繼續參閱「如何搭配使用 Azure PowerShell 與 Azure 儲存體」。

> [!div class="nextstepaction"]
> [搭配使用 Azure PowerShell 與 Azure 儲存體](../common/storage-powershell-guide-full.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

### <a name="microsoft-azure-powershell-storage-cmdlets-reference"></a>Microsoft Azure PowerShell 儲存體 Cmdlet 參考

* [儲存體 PowerShell Cmdlet](/powershell/module/azurerm.storage#storage)

### <a name="microsoft-azure-storage-explorer"></a>Microsoft Azure 儲存體總管

* [Microsoft Azure 儲存體總管](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。