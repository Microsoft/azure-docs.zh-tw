---
title: 用於轉譯的 Azure Batch 儲存體和資料移動
description: 轉譯工作負載的儲存體和資料移動選項
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: d5b006fd744e463c73ee0a32388f254017e96354
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036706"
---
# <a name="storage-and-data-movement-options-for-rendering-asset-and-output-files"></a>轉譯資產和輸出檔案的儲存體和資料移動選項

有多個選項可讓場景和資產檔案供集區 VM 上的轉譯應用程式使用：

* [Azure Blob 儲存體](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)：
  * 場景和資產檔案可從本機檔案系統上傳至 Blob 儲存體。 有工作執行應用程式時，所需的檔案會從 Blob 儲存體複製到 VM 上，使其可供轉譯應用程式存取。 輸出檔案會由轉譯應用程式寫入至 VM 磁碟，然後複製到 Blob 儲存體。  如有必要，可將輸出檔案從 Blob 儲存體下載到本機檔案系統。
  * Azure Blob 儲存體對較小的專案而言是簡單且符合成本效益的選項。  如果每個集區 VM 都需要所有的資產檔案，則在資產檔案的大小與數量增加時，則務必留意以確保檔案傳輸盡可能維持高效率。  
* 使用 [blobfuse](https://docs.microsoft.com/azure/storage/blobs/storage-how-to-mount-container-linux) 將 Azure 儲存體作為檔案系統：
  * 針對 Linux VM，在使用 blobfuse 虛擬檔案系統驅動程式時，儲存體帳戶可公開並作為檔案系統。
  * 此選項的優點是非常符合成本效益，因為檔案系統不需要任何 VM，且 VM 上的 blobfuse 快取可避免為多個作業和工作重複下載相同的檔案。  資料移動也非常簡單，因為檔案都只是 Blob，並且可使用標準 API 和工具 (例如 azcopy) 在內部部署檔案系統與 Azure 儲存體之間檔案複製。
* 檔案系統或檔案共用：
  * 根據 VM 作業系統和效能/調整需求，選項包括 [Azure 檔案服務](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)、將 VM 搭配連接的磁碟用於 NFS、將多個 VM 搭配連接的磁碟用於分散式檔案系統 (例如 GlusterFS)，或使用第三方供應項目。
  * [Avere 系統](http://www.averesystems.com/)現在是 Microsoft 的一部分，且近期將會有適合用於大規模、高效能轉譯的解決方案。  Avere 解決方案將可讓您建立以 Azure 為基礎的 NFS 或 SMB 快取，與 Blob 儲存體或內部部署 NAS 裝置搭配使用。
  * 透過檔案系統，檔案將可直接讀取或寫入至檔案系統，或是在檔案系統與集區 VM 之間複製。
  * 共用的檔案系統可讓您在要使用的專案和工作之間共用大量資產，且轉譯工作只會存取所需的項目。

## <a name="using-azure-blob-storage"></a>使用 Azure Blob 儲存體

您應使用 Blob 儲存體帳戶或一般用途 v2 儲存體帳戶。  這兩種儲存體帳戶類型在設定時可使用遠高於一般用途 v1 儲存體帳戶的限制，如[此部落格文章](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/)所詳述。  經設定後，較高的限制將可支援更較佳的效能和延展性，尤其是在有許多集區 VM 存取儲存體帳戶時。

### <a name="copying-files-between-client-and-blob-storage"></a>在用戶端與 Blob 儲存體之間複製檔案

若要將檔案複製到 Azure 儲存體或從中複製檔案，有多種機制可供使用，包括儲存體 Blob API、[Azure 儲存體資料移動程式庫](https://github.com/Azure/azure-storage-net-data-movement)、適用於 [Windows](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy) 或 [Linux](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-linux) 的 azcopy 命令列工具、[Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)和 [Azure Batch Explorer](https://azure.github.io/BatchExplorer/)。

以使用 azcopy 為例，資料夾中的所有資產都可傳輸如下：


`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /Y`

若只要複製修改過的檔案，您可以使用 /XO 參數：

`azcopy /source:. /dest:https://account.blob.core.windows.net/rendering/project /destsas:"?st=2018-03-30T16%3A26%3A00Z&se=2020-03-31T16%3A26%3A00Z&sp=rwdl&sv=2017-04-17&sr=c&sig=sig" /XO /Y`

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>將輸入資產檔案從 Blob 儲存體複製到 Batch 集區 VM

有幾種不同的方法可用來複製檔案，最理想的方法取決於作業資產的大小。
最簡單的方法，是將每個作業的所有資產檔案複製到集區 VM：

* 如果某些檔案專屬於某項作業，但同時也是該作業所有工作的必要檔案，則可以指定[作業準備工作](https://docs.microsoft.com/rest/api/batchservice/job/add#jobpreparationtask)以複製所有檔案。  作業準備工作會在對 VM 執行第一個作業工作時執行，但對於後續的作業工作則不會再執行。
* 您應指定[作業解除工作](https://docs.microsoft.com/rest/api/batchservice/job/add#jobreleasetask)，以在作業完成後隨即移除個別作業檔案；這可以避免 VM 磁碟被所有作業資產檔案填滿。
* 如果有多個作業使用相同的資產，且僅對每個作業的資產進行累加式變更，則仍會複製所有資產檔案，即使只有子集更新亦然。  有許多大型資產檔案時，此方式將效率不彰。

如果在作業之間重複使用資產檔案，且僅在作業之間進行累加式變更，則將資產儲存在 VM 上的共用資料夾中，並同步變更的檔案，會是較有效率的方法，但也稍加複雜。

* 作業準備工作會使用 azcopy 與 /XO 參數對 AZ_BATCH_NODE_SHARED_DIR 環境變數所指定的 VM 共用資料夾執行複製。  這只會將已變更的檔案複製到每個 VM。
* 您必須考量所有資產的大小，以確保它們可容納於集區 VM 的暫存磁碟機。

Azure Batch 具有在儲存體帳戶與 Batch 集區 VM 之間複製檔案的內建支援。  工作[資源檔](https://docs.microsoft.com/rest/api/batchservice/job/add#resourcefile)會將檔案從儲存體複製到集區 VM，並且可指定於作業準備工作。  然而，當檔案高達數百個時，就有可能達到限制而導致工作失敗。  有大量資產時，建議您在在作業準備工作中使用 azcopy 命令列，如此即可使用萬用字元，而不會有限制。

### <a name="copying-output-files-to-blob-storage-from-batch-pool-vms"></a>將輸出檔案從 Batch 集區 VM 複製到 Blob 儲存體

[輸出檔案](https://docs.microsoft.com/rest/api/batchservice/task/add#outputfile)可用來將檔案從集區 VM 複製到儲存體。  在工作完成後，可以將一或多個檔案從 VM 複製到指定的儲存體帳戶。  此時應複製已轉譯的輸出，但可能也應儲存記錄檔。

## <a name="using-a-blobfuse-virtual-file-system-for-linux-vm-pools"></a>將 blobfuse 虛擬檔案系統用於 Linux VM 集區

Blobfuse 是 Azure Blob 儲存體的虛擬檔案系統驅動程式，可讓您透過 Linux 檔案系統存取在儲存體帳戶中儲存為 Blob 的檔案。

集區節點可在啟動時掛接檔案系統，或在作業準備工作進行時執行掛接 – 該準備工作只會在作業中的第一個工作執行於節點時執行。  Blobfuse 可設定為同時使用 ramdisk 和 VM 本機 SSD 來快取檔案，這在節點上的多個工作存取某些相同的檔案時，將可大幅提升效能。

[範例範本可供使用](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/vray/render-linux-with-blobfuse-mount)，以透過 blobfuse 檔案系統執行獨立 V-Ray 轉譯器，並且可作為其他應用程式範本的基礎。

### <a name="accessing-files"></a>存取檔案

作業工作會使用已掛接的檔案系統指定輸入檔案和輸出檔案的路徑。

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>將輸入資產檔案從 Blob 儲存體複製到 Batch 集區 VM

如果檔案只是 Azure 儲存體中的 Blob，則可以使用標準 Blob API、工具和 UI 在內部部署檔案系統與 Blob 儲存體之間複製檔案；例如 azcopy、儲存體總管、Batch Explorer 等。

## <a name="using-azure-files-with-windows-vms"></a>搭配使用 Azure 檔案服務與 Windows VM

[Azure 檔案服務](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)可提供在雲端中完全受控、並且可透過 SMB 通訊協定來存取的檔案共用。  Azure 檔案服務以 Azure Blob 儲存體為基礎，不但[符合成本效益](https://azure.microsoft.com/pricing/details/storage/files/)，還能設定將資料複寫到另一個區域，而達成全域備援的目標。  您應檢閱[調整目標](https://docs.microsoft.com/azure/storage/files/storage-files-scale-targets#azure-files-scale-targets)，以根據指定的預測集區大小和資產檔案數目，判斷是否應使用 Azure 檔案服務。

您可以參考相關[部落格文章](https://blogs.msdn.microsoft.com/windowsazurestorage/2014/05/26/persisting-connections-to-microsoft-azure-files/)和[文件](https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-windows)以了解如何掛接 Azure 檔案共用。

### <a name="mounting-an-azure-files-share"></a>掛接 Azure 檔案共用

若要在 Batch 中使用，掛接作業必須在每次工作執行期間執行，因為連線無法跨工作保存。  若要這樣做，最簡單的方式是透過 cmdkey 使用集區組態中的啟動工作來保存認證，然後在每個工作之前掛接共用。

集區範本中的 cmdkey 使用範例 (在 JSON 檔案中使用時須逸出) – 請注意，在區隔 cmdkey 呼叫與 net use 呼叫時，啟動工作的使用者內容必須與用來執行工作的內容相同：

```
"startTask": {
  "commandLine": "cmdkey /add:storageaccountname.file.core.windows.net
    /user:AZURE\\markscuscusbatch /pass:storage_account_key",
  "userIdentity":{
    "autoUser": {
      "elevationLevel": "nonadmin",
      "scope": "pool"
    }
}
```

範例作業工作命令列：
```
"commandLine":"net use S:
  \\\\storageaccountname.file.core.windows.net\\rendering &
3dsmaxcmdio.exe -v:5 -rfw:0 -10 -end:10
  -bitmapPath:\"s:\\3dsMax\\Dragon\\Assets\"
  -outputName:\"s:\\3dsMax\\Dragon\\RenderOutput\\dragon.jpg\"
  -w:1280 -h:720
  \"s:\\3dsMax\\Dragon\\Assets\\Dragon_Character_Rig.max\""
```

### <a name="accessing-files"></a>存取檔案

作業工作會使用已掛接的檔案系統指定輸入檔案和輸出檔案的路徑 (使用對應磁碟機或 UNC 路徑)。

### <a name="copying-input-asset-files-from-blob-storage-to-batch-pool-vms"></a>將輸入資產檔案從 Blob 儲存體複製到 Batch 集區 VM

所有具有 Azure 儲存體支援的主要 API 和工具均支援 Azure 檔案服務，例如 azcopy、Azure CLI、儲存體總管、Azure PowerShell、Batch Explorer 等。

[Azure 檔案同步](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning)可用來在內部部署檔案系統與 Azure 檔案共用之間自動同步處理檔案。

## <a name="next-steps"></a>後續步驟

如需關於儲存體選項的詳細資訊，請參閱深入文件：

* [Azure Blob 儲存體](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction)
* [Blobfuse](https://docs.microsoft.com/azure/storage/blobs/storage-how-to-mount-container-linux)
* [Azure 檔案](https://docs.microsoft.com/azure/storage/files/storage-files-introduction)
