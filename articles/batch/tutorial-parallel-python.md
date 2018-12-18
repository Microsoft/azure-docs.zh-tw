---
title: 執行平行工作負載 - Azure Batch Python
description: 教學課程 - 使用 Batch Python 用戶端程式庫透過 Azure Batch 中的 ffmpeg 平行處理媒體檔案
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.devlang: python
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 6bbaa9693bb8d8e54e78f1e83617449cd013ad48
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158729"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-python-api"></a>教學課程：使用 Python API 透過 Azure Batch 執行平行工作負載

使用 Azure Batch 在 Azure 中有效率地執行大規模的平行和高效能運算 (HPC) 批次作業。 本教學課程會逐步說明使用 Batch 執行平行工作負載的 Python 範例。 您會了解 Batch 應用程式的通用工作流程，以及如何以程式設計方式與 Batch 和儲存體資源進行互動。 您會了解如何：

> [!div class="checklist"]
> * 驗證 Batch 和儲存體帳戶
> * 將輸入檔案上傳至儲存體
> * 建立計算節點的集區來執行應用程式
> * 建立作業和工作來處理輸入檔案
> * 監視工作執行
> * 擷取輸出檔案

在本教學課程中，您會使用 [ffmpeg](http://ffmpeg.org/) 開放原始碼工具將 MP4 媒體檔案平行轉換為 MP3 格式。 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先決條件

* [Python 2.7 或 3.3 版或更新版本](https://www.python.org/downloads/)

* [pip](https://pip.pypa.io/en/stable/installing/) 套件管理員

* Azure Batch 帳戶和連結的 Azure 儲存體帳戶。 若要建立這些帳戶，請參閱使用 [Azure 入口網站](quick-create-portal.md)或 [Azure CLI](quick-create-cli.md) 的 Batch 快速入門。

## <a name="sign-in-to-azure"></a>登入 Azure

在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)] 

## <a name="download-and-run-the-sample"></a>下載及執行範例

### <a name="download-the-sample"></a>下載範例

從 GitHub [下載或複製範例應用程式](https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial)。 若要使用 Git 用戶端複製範例應用程式存放庫，請使用下列命令：

```
git clone https://github.com/Azure-Samples/batch-python-ffmpeg-tutorial.git
```

瀏覽至包含 `batch_python_tutorial_ffmpeg.py` 檔案的目錄。

在 Python 環境中，使用 `pip` 安裝必要的套件。

```bash
pip install -r requirements.txt
```

開啟檔案 `batch_python_tutorial_ffmpeg.py`。 使用您帳戶獨有的值來更新 Batch 和儲存體帳戶認證字串。 例如︰


```Python
_BATCH_ACCOUNT_NAME = 'mybatchaccount'
_BATCH_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ=='
_BATCH_ACCOUNT_URL = 'https://mybatchaccount.mybatchregion.batch.azure.com'
_STORAGE_ACCOUNT_NAME = 'mystorageaccount'
_STORAGE_ACCOUNT_KEY = 'xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ=='
```

### <a name="run-the-app"></a>執行應用程式

執行指令碼：

```
python batch_python_tutorial_ffmpeg.py
```

當您執行範例應用程式時，主控台輸出大致如下。 在執行期間，集區的計算節點啟動後，您會在 `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...` 遇到暫停。 
   
```
Sample start: 12/12/2017 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [LinuxFFmpegPool]...
Creating job [LinuxFFmpegJob]...
Adding 5 tasks to job [LinuxFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]....

Sample end: 12/12/2017 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

移至 Azure 入口網站中您的 Batch 帳戶，以監視集區、計算節點、作業和工作。 例如，若要查看集區中計算節點的熱度圖，請按一下 [集區] > [LinuxFFmpegPool]。

當工作正在執行時，熱度圖會如下所示：

![集區熱度圖](./media/tutorial-parallel-python/pool.png)

以預設設定執行應用程式時，一般的執行時間**大約 5 分鐘**。 建立集區佔用大部分的時間。 

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>檢閱程式碼

後面各節將範例應用程式細分為用來處理 Batch 服務中工作負載的數個步驟。 在閱讀本文的其餘部分時，請參考 Python 程式碼，因為本文並不會討論範例中的每一行程式碼。

### <a name="authenticate-blob-and-batch-clients"></a>驗證 Blob 和 Batch 用戶端

為了與儲存體帳戶進行互動，應用程式會使用 [azure-storage-blob](https://pypi.python.org/pypi/azure-storage-blob) 套件來建立 [BlockBlobService](/python/api/azure.storage.blob.blockblobservice.blockblobservice) 物件。

```python
blob_client = azureblob.BlockBlobService(
    account_name=_STORAGE_ACCOUNT_NAME,
    account_key=_STORAGE_ACCOUNT_KEY)
```

應用程式會建立 [BatchServiceClient](/python/api/azure.batch.batchserviceclient) 物件，以在 Batch 服務中建立和管理集區、作業和工作。 範例中的 Batch 用戶端會使用共用金鑰驗證。 Batch 也支援透過 [Azure Active Directory](batch-aad-auth.md) 進行驗證，以便驗證個別使用者或自動執行的應用程式。

```python
credentials = batchauth.SharedKeyCredentials(_BATCH_ACCOUNT_NAME,
    _BATCH_ACCOUNT_KEY)

batch_client = batch.BatchServiceClient(
    credentials,
    base_url=_BATCH_ACCOUNT_URL)
```

### <a name="upload-input-files"></a>上傳輸入檔案

應用程式會使用 `blob_client` 參考，建立輸入 MP4 檔案的儲存體容器和工作輸出的容器。 接著，它會呼叫 `upload_file_to_container` 函式，將本機 `InputFiles` 目錄中的 MP4 檔案上傳至容器。 儲存體中的檔案會定義為 Batch [ResourceFile](/python/api/azure.batch.models.resourcefile) 物件，Batch 之後可將這類物件下載到計算節點。

```python
blob_client.create_container(input_container_name, fail_on_exist=False)
blob_client.create_container(output_container_name, fail_on_exist=False)
input_file_paths = []
    
for folder, subs, files in os.walk(os.path.join(sys.path[0],'./InputFiles/')):
    for filename in files:
        if filename.endswith(".mp4"):
            input_file_paths.append(os.path.abspath(os.path.join(folder, filename)))

# Upload the input files. This is the collection of files that are to be processed by the tasks. 
input_files = [
    upload_file_to_container(blob_client, input_container_name, file_path)
    for file_path in input_file_paths]
```

### <a name="create-a-pool-of-compute-nodes"></a>建立計算節點的集區

接著，範例會呼叫 `create_pool` 以在 Batch 帳戶中建立計算節點集區。 這個已定義的函式會使用 [PoolAddParameter](/python/api/azure.batch.models.pooladdparameter) 類別來設定節點數目、VM 大小和集區設定。 在此，[VirtualMachineConfiguration](/python/api/azure.batch.models.virtualmachineconfiguration) 物件會將 [ImageReference](/python/api/azure.batch.models.imagereference) 指定至 Azure Marketplace 中發佈的 Ubuntu Server 16.04 LTS 映像。 Batch 支援 Azure Marketplace 中各式各樣的 VM 映像，以及自訂 VM 映像。

使用已定義的常數可設定節點數目和 VM 大小。 Batch 支援專用節點和[低優先順序節點](batch-low-pri-vms.md)，而您可以在集區中使用其中一種或同時使用兩種。 專用節點會保留給您的集區使用。 低優先順序節點則會以較低的價格從 Azure 中的剩餘容量提供。 如果 Azure 沒有足夠的容量，便無法使用低優先順序節點。 此範例預設建立的集區只包含 5 個大小為 Standard_A1_v2 的低優先順序節點。 

除了實體節點屬性以外，此集區設定還包含 [StartTask](/python/api/azure.batch.models.starttask) 物件。 StartTask 會在每個節點加入集區以及每次重新啟動節點時，於該節點上執行。 在此範例中，StartTask 會執行 Bash Shell 命令，以在節點上安裝 ffmpeg 套件和相依項目。

[pool.add](/python/api/azure.batch.operations.pooloperations#azure_batch_operations_PoolOperations_add) 方法會將此集區提交至 Batch 服務。

```python
new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
            publisher="Canonical",
            offer="UbuntuServer",
            sku="16.04-LTS",
            version="latest"
            ),
        node_agent_sku_id="batch.node.ubuntu 16.04"),
    vm_size=_POOL_VM_SIZE,
    target_dedicated_nodes=_DEDICATED_POOL_NODE_COUNT,
    target_low_priority_nodes=_LOW_PRIORITY_POOL_NODE_COUNT,
    start_task=batchmodels.StartTask(
        command_line="/bin/bash -c \"apt-get update && apt-get install -y ffmpeg\"",
        wait_for_success=True,
        user_identity=batchmodels.UserIdentity(
            auto_user=batchmodels.AutoUserSpecification(
                scope=batchmodels.AutoUserScope.pool,
                elevation_level=batchmodels.ElevationLevel.admin)),
    )
)
batch_service_client.pool.add(new_pool)
```

### <a name="create-a-job"></a>建立工作

Batch 工作會指定要在其中執行工作的集區及選擇性設定，例如工作的優先順序和排程。 此範例會藉由呼叫 `create_job` 來建立作業。 這個已定義的函式會使用 [JobAddParameter](/python/api/azure.batch.models.jobaddparameter) 類別在您的集區上建立作業。 [job.add](/python/api/azure.batch.operations.joboperations#azure_batch_operations_JobOperations_add) 方法會將集區提交至 Batch 服務。 一開始作業沒有任何工作。

```python
job = batch.models.JobAddParameter(
    id=job_id,
    pool_info=batch.models.PoolInformation(pool_id=pool_id))

batch_service_client.job.add(job)
```

### <a name="create-tasks"></a>建立工作

應用程式會藉由呼叫 `add_tasks` 在作業中建立工作。 這個已定義的函式會使用 [TaskAddParameter](/python/api/azure.batch.models.taskaddparameter) 類別建立工作物件清單。 每項工作都會執行 ffmpeg，以使用 `command_line` 參數來處理輸入 `resource_files` 物件。 ffmpeg 已在先前建立集區時安裝於每個節點上。 在此，命令列會執行 ffmpeg，將每個輸入 MP4 (影片) 檔案轉換為 MP3 (音訊) 檔案。

此範例會在執行命令列之後，為 MP3 檔案建立 [OutputFile](/python/api/azure.batch.models.outputfile) 物件。 每項工作的輸出檔案 (在此例中只有一個輸出檔案) 都會使用工作的 `output_files` 屬性，上傳至所連結儲存體帳戶中的容器。

然後，應用程式會使用 [task.add_collection](/python/api/azure.batch.operations.taskoperations#azure_batch_operations_TaskOperations_add_collection) 方法將工作新增至作業，該方法會將工作排入佇列以在計算節點上執行。 

```python
tasks = list()

for idx, input_file in enumerate(input_files): 
    input_file_path=input_file.file_path
    output_file_path="".join((input_file_path).split('.')[:-1]) + '.mp3'
    command = "/bin/bash -c \"ffmpeg -i {} {} \"".format(input_file_path, output_file_path)
    tasks.append(batch.models.TaskAddParameter(
        id='Task{}'.format(idx),
        command_line=command,
        resource_files=[input_file],
        output_files=[batchmodels.OutputFile(
            file_pattern=output_file_path,
            destination=batchmodels.OutputFileDestination(
                container=batchmodels.OutputFileBlobContainerDestination(
                    container_url=output_container_sas_url)),
            upload_options=batchmodels.OutputFileUploadOptions(
                upload_condition=batchmodels.OutputFileUploadCondition.task_success))]
        )
    )
batch_service_client.task.add_collection(job_id, tasks)
```    

### <a name="monitor-tasks"></a>監視工作

工作新增至作業時，Batch 會自動將工作排入佇列並進行排程，以便在相關聯集區中的計算節點上執行。 根據您指定的設定，Batch 會處理所有工作佇列、排程、重試和其他工作管理責任。 

監視工作執行的方法有許多種。 此範例中的 `wait_for_tasks_to_complete` 函式會使用 [TaskState](/python/api/azure.batch.models.taskstate) 物件來監視某個時限內特定狀態的工作，在此例中為 [已完成] 狀態。

```python
while datetime.datetime.now() < timeout_expiration:
    print('.', end='')
    sys.stdout.flush()
    tasks = batch_service_client.task.list(job_id)

    incomplete_tasks = [task for task in tasks if
                         task.state != batchmodels.TaskState.completed]
    if not incomplete_tasks:
        print()
        return True
    else:
        time.sleep(1)
...
```

## <a name="clean-up-resources"></a>清除資源

應用程式在執行工作之後，會自動刪除它所建立的輸入儲存體容器，並且為您提供用於刪除 Batch 集區和工作的選項。 BatchClient 的 [JobOperations](/python/api/azure.batch.operations.joboperations) 和 [PoolOperations](/python/api/azure.batch.operations.pooloperations) 類別都有刪除方法 (在您確認刪除時呼叫)。 雖然您不需支付作業和工作本身的費用，但您需支付計算節點的費用。 因此，我們建議您只在必要時配置集區。 當您刪除集區時，節點上的所有工作輸出也會跟著刪除。 不過，輸入和輸出檔案會保留在儲存體帳戶中。

若不再需要，可刪除資源群組、Batch 帳戶和儲存體帳戶。 若要在 Azure 入口網站中這麼做，請選取 Batch 帳戶的資源群組，然後按一下 [刪除資源群組]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 驗證 Batch 和儲存體帳戶
> * 將輸入檔案上傳至儲存體
> * 建立計算節點的集區來執行應用程式
> * 建立作業和工作來處理輸入檔案
> * 監視工作執行
> * 擷取輸出檔案

如需更多使用 Python API 來排程和處理 Batch 工作負載的範例，請參閱 GitHub 上的範例。

> [!div class="nextstepaction"]
> [Batch Python 範例](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch)

