---
title: Azure Batch 集區的 Azure 檔案共用 | Microsoft Docs
description: 如何從 Linux 中的計算節點或 Azure Batch 中的 Windows 集區，裝載 Azure Files 共用。
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: big-compute
ms.date: 05/24/2018
ms.author: danlep
ms.custom: ''
ms.openlocfilehash: 88d7c0d033d7b517a396df27468de8be7ae20be9
ms.sourcegitcommit: 6cf20e87414dedd0d4f0ae644696151e728633b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/06/2018
ms.locfileid: "34811766"
---
# <a name="use-an-azure-file-share-with-a-batch-pool"></a>搭配 Batch 集區使用 Azure 檔案共用

[Azure Files](../storage/files/storage-files-introduction.md) 提供雲端中完全受控的檔案共用，可透過伺服器訊息區 (SMB) 通訊協定存取。 本文章提供在集區計算節點上裝載和使用 Azure 檔案共用的資訊和程式碼範例。 程式碼範例使用 Batch .NET 和 Python SDK，但您可以使用其他 Batch SDK 和工具來執行類似作業。

Batch 提供原生 API 支援，您可使用 Azure 儲存體 BLOb 讀取和寫入資料。 然而，在某些情況下，您可能會想從集區計算節點存取 Azure 檔案共用。 例如，您有舊版工作負載相依於 SMB 檔案共用，或您的工作需要存取共用資料或產生共用輸出。 

## <a name="considerations-for-use-with-batch"></a>搭配 Batch 的使用考量

* 當集區執行的平行工作數目相對較低時，請考慮使用 Azure 檔案共用。 根據您預期的集區大小和資產檔案數量，檢閱[效能和擴充目標](../storage/files/storage-files-scale-targets.md)，以判斷是否要使用 (使用 Azure 儲存體帳戶的) Azure Files。 

* Azure 檔案共用不但[符合成本效益](https://azure.microsoft.com/pricing/details/storage/files/)，還能設定將資料複寫到另一個區域，因此達成全域備援的目標。 

* 您可同時從內部部署電腦裝載 Azure 檔案共用。

* 另請參閱 Azure 檔案共用的一般[規劃考量](../storage/files/storage-files-planning.md)。


## <a name="create-a-file-share"></a>建立檔案共用

在儲存體帳戶[建立檔案共用](../storage/files/storage-how-to-create-file-share.md)，並連結至您的 Batch 帳戶，或者在個別的儲存體帳戶建立。

## <a name="mount-a-share-on-a-windows-pool"></a>在 Windows 集區上裝載共用

本章節提供在 Windows 節點的集區上裝載和使用 Azure 檔案共用的步驟和程式碼範例。 如需其他背景資訊，請參閱在 Windows 裝載 Azure 檔案共用的[文件](../storage/files/storage-how-to-use-files-windows.md)。 

在 Batch 中，每次在 Windows 節點上執行工作時都需要裝載共用。 目前，無法在 Windows 節點上的工作之間保持網路連線。

例如，在每個工作命令列中包括 `net use` 命令以裝載檔案共用。 若要裝載檔案共用，需要以下認證：

* **使用者名稱**：AZURE\\\<storageaccountname\>，例如 AZURE\\*mystorageaccountname*
* **密碼**：<StorageAccountKeyWhichEnds in==>，例如 *XXXXXXXXXXXXXXXXXXXXX==*

下列命令在儲存體帳戶 *mystorageaccountname* 裝載檔案共用 *myfileshare* 作為 *S:* 磁碟機：

```
net use S: \\mystorageaccountname.file.core.windows.net\myfileshare /user:AZURE\mystorageaccountname XXXXXXXXXXXXXXXXXXXXX==
```

為了簡單說明，這裡的範例會直接在文字中傳遞認證。 實際上，我們強烈建議使用環境變數、憑證或 Azure Key Vault 等解決方案來管理認證。

若要簡化裝載作業，可選擇性保存節點上的認證。 接著您就可以裝載共用而無需認證。 請執行以下兩個步驟：

1. 使用集區設定中的啟動工作來執行 `cmdkey` 命令列公用程式。 這麼做可保存每個 Windows 節點上的認證。 啟動工作命令列會類似於：

  ```
  cmd /c "cmdkey /add:mystorageaccountname.file.core.windows.net /user:AZURE\mystorageaccountname /pass:XXXXXXXXXXXXXXXXXXXXX=="

  ```

2. 使用 `net use` 在每個節點上裝載共用做為每個工作的一部分。 例如，下列工作命令列裝載檔案共用做為 *S:* 磁碟機。 參考此共用的命令或指令碼會隨後執行。 呼叫 `net use` 時使用快取的認證。 這個步驟假設您使用的工作使用者身分識別與您在集區啟動工作中所使用的相同，而此步驟並不適用於所有情況。

  ```
  cmd /c "net use S: \\mystorageaccountname.file.core.windows.net\myfileshare" 
  ```

### <a name="c-example"></a>C# 範例
下列 C# 範例會顯示如何使用啟動工作在 Windows 集區保存認證。 儲存體檔案服務名稱和儲存體認證會以定義的常數傳遞。 在這裡，啟動工作會在具有集區範圍的標準 (非管理員) 自動使用者帳戶下執行。

```csharp
...
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: PoolId,
    targetDedicatedComputeNodes: PoolNodeCount,
    virtualMachineSize: PoolVMSize,
    virtualMachineConfiguration: virtualMachineConfiguration);

// Start task to store credentials to mount file share
string startTaskCommandLine = String.Format("cmd /c \"cmdkey /add:{0} /user:AZURE\\{1} /pass:{2}\"", StorageFileService, StorageAccountName, StorageAccountKey);

pool.StartTask = new StartTask
{
    CommandLine = startTaskCommandLine,
    UserIdentity = new UserIdentity(new AutoUserSpecification(
        elevationLevel: ElevationLevel.NonAdmin, 
        scope: AutoUserScope.Pool))
};

pool.Commit();
```

儲存認證後，使用您的工作命令列裝載共用並參考共用或寫入作業中的讀取。 在基本範例中，以下程式碼片段中的工作命令列會使用 `dir` 命令來列出檔案共用中的檔案。 請務必使用用於集區中執行啟動工作的相同[使用者識別碼](batch-user-accounts.md)來執行作業工作。 

```csharp
...
string taskId = "myTask";
string taskCommandLine = String.Format("cmd /c \"net use {0} {1} & dir {2}\"", ShareMountPoint, StorageFileShare, ShareMountPoint);

CloudTask task = new CloudTask(taskId, taskCommandLine);
task.UserIdentity = new UserIdentity(new AutoUserSpecification(
    elevationLevel: ElevationLevel.NonAdmin,
    scope: AutoUserScope.Pool));
tasks.Add(task);
```

## <a name="mount-a-share-on-a-linux-pool"></a>在 Linux 集區裝載共用

可以使用 [CIFS 核心用戶端](https://wiki.samba.org/index.php/LinuxCIFS)將 Azure 檔案共用掛接在 Linux 發行版本中。 下列範例顯示如何在 Ubuntu 16.04 LTS 計算節點的集區上裝載檔案共用。 如果您使用不同的 Linux 發行版本，一般步驟類似，但請使用適合該發行版本的套件管理員。 如需詳細資訊和其他範例，請參閱[搭配 Linux 使用 Azure Files](../storage/files/storage-how-to-use-files-linux.md)。

首先，以系統管理員使用者身分，在本機檔案系統中安裝 `cifs-utils` 套件，並建立裝載點 (例如 */mnt/MyAzureFileShare*)。 裝載點的資料夾可以在檔案系統上任何位置建立，但是常見慣例是在 `/mnt` 資料夾底下建立這個資料夾。 請確認不要直接在 `/mnt` (Ubuntu 上) 或 `/mnt/resource` (其他散發套件上) 建立裝載點。

```
apt-get update && apt-get install cifs-utils && sudo mkdir -p /mnt/MyAzureFileShare
```

接著，執行 `mount` 命令以裝載檔案共用，藉此提供這些認證：

* **使用者名稱**：\<storageaccountname\>，例如 *mystorageaccountname*
* **密碼**：<StorageAccountKeyWhichEnds in==>，例如 *XXXXXXXXXXXXXXXXXXXXX==*

下列命令在 */mnt/MyAzureFileShare* 的儲存體帳戶 *mystorageaccountname* 裝載檔案共用 *myfileshare*： 

```
mount -t cifs //mystorageaccountname.file.core.windows.net/myfileshare /mnt/MyAzureFileShare -o vers=3.0,username=mystorageaccountname,password=XXXXXXXXXXXXXXXXXXXXX==,dir_mode=0777,file_mode=0777,serverino && ls /mnt/MyAzureFileShare
```

為了簡單說明，這裡的範例會直接在文字中傳遞認證。 實際上，我們強烈建議使用環境變數、憑證或 Azure Key Vault 等解決方案來管理認證。

在 Linux 集區，您可以在單一啟動工作中結合所有這些步驟，或在指令碼中執行。 在集區以系統管理員使用者身分執行啟動工作。 設定您的啟動工作並等待成功完成作業，再於集區執行其他參考此共用的工作。

### <a name="python-example"></a>Python 範例

下列 Python 範例顯示如何設定 Ubuntu 集區以在啟動工作中裝載共用。 裝載點、檔案共用端點和儲存體認證會以定義的常數傳遞。 啟動工作在具有集區範圍的系統管理員自動使用者帳戶下執行。

```python
pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration = batchmodels.VirtualMachineConfiguration(
        image_reference = batchmodels.ImageReference(
            publisher="Canonical",
            offer="UbuntuServer",
            sku="16.04.0-LTS",
            version="latest"),
        node_agent_sku_id = "batch.node.ubuntu 16.04"),
    vm_size=_POOL_VM_SIZE,
    target_dedicated_nodes=_POOL_NODE_COUNT,
    start_task=batchmodels.StartTask(
        command_line="/bin/bash -c \"apt-get update && apt-get install cifs-utils && mkdir -p {} && mount -t cifs {} {} -o vers=3.0,username={},password={},dir_mode=0777,file_mode=0777,serverino\"".format(_COMPUTE_NODE_MOUNT_POINT, _STORAGE_ACCOUNT_SHARE_ENDPOINT, _COMPUTE_NODE_MOUNT_POINT, _STORAGE_ACCOUNT_NAME, _STORAGE_ACCOUNT_KEY),
        wait_for_success=True,
        user_identity=batchmodels.UserIdentity(
            auto_user=batchmodels.AutoUserSpecification(
                scope=batchmodels.AutoUserScope.pool,
                elevation_level=batchmodels.ElevationLevel.admin)),
    )
)
batch_service_client.pool.add(pool)
```

裝載共用並定義作業後，使用您工作命令列中的共用。 例如，下列基本命令會使用 `ls` 來列出檔案共用中的檔案。

```python
...
task = batch.models.TaskAddParameter(
    id='mytask',
    command_line="/bin/bash -c \"ls {}\"".format(_COMPUTE_NODE_MOUNT_POINT))

batch_service_client.task.add(job_id, task)
```


## <a name="next-steps"></a>後續步驟

* 如需在 Batch 中讀取和寫入資料的其他選項，請參閱 [Batch 功能概觀](batch-api-basics.md)和[持續作業及工作輸出](batch-task-output.md)。

* 另請參閱 [Batch Shipyard](https://github.com/Azure/batch-shipyard) 工具組，其中包括 [Shipyard 訣竅](https://github.com/Azure/batch-shipyard/tree/master/recipes)，以部署 Batch 容器工作負載的檔案系統。