---
title: 使用多重執行個體工作執行 MPI 應用程式 - Azure Batch | Microsoft Docs
description: 了解如何在 Azure Batch 中使用多重執行個體工作類來執行訊息傳遞介面 (MPI) 應用程式。
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 83e34bd7-a027-4b1b-8314-759384719327
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.date: 06/12/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a443dd7ed4f95b3e283603fa8938a08c2c177827
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39174299"
---
# <a name="use-multi-instance-tasks-to-run-message-passing-interface-mpi-applications-in-batch"></a>在 Batch 中使用多重執行個體工作來執行訊息傳遞介面 (MPI) 應用程式

多重執行個體工作可讓您在多個計算節點上同時執行 Azure Batch 工作。 這些工作可以在 Batch 中實現高效能運算案例，例如訊息傳遞介面 (MPI) 應用程式。 在本文中，您將了解如何使用 [Batch .NET][api_net] 程式庫來執行多重執行個體工作。

> [!NOTE]
> 雖然本文中的範例著重於 Batch .NET、MS-MPI 和 Windows 計算節點，不過此處所討論的多重執行個體工作概念也適用於其他平台和技術 (如 Python 和 Linux 節點上的 Intel MPI)。
>
>

## <a name="multi-instance-task-overview"></a>多重執行個體工作概觀
在 Batch 中，每個工作通常是在單一計算節點上執行 -- 您將多個工作提交給作業，而 Batch 服務會將每個工作排定在節點上執行。 不過，藉由設定工作的**多重執行個體設定**，即可告知 Batch 改為建立一個主要工作，以及數個會接著在多個節點上執行的子工作。

![多重執行個體工作概觀][1]

將具有多重執行個體設定的工作提交給作業時，Batch 會執行多重執行個體工作特有的幾個步驟：

1. Batch 服務會根據多重執行個體設定，建立一個**主要**工作和數個**子工作**。 工作總數 (主要工作加上所有子工作) 與您在多重執行個體設定中指定的**執行個體** (計算節點) 數目相符。
2. Batch 能將一個計算節點指定為**主要**節點，然後將主要工作排程在主要節點上執行。 它會將子工作排程在配置給多重執行個體工作所剩餘的計算節點上執行，每個節點一個子工作。
3. 主要工作和子工作會下載您在多重執行個體設定中指定的任何**一般資源檔**。
4. 下載一般資源檔之後，主要工作和子工作會執行您在多重執行個體設定中指定的 **協調命令** 。 協調命令通常用來準備執行工作所需的節點。 其中包括啟動背景服務 (如 [Microsoft MPI][msmpi_msdn] 的 `smpd.exe`)，以及確認節點已準備好處理節點間的訊息。
5. 主要工作及所有子工作順利完成協調命令「之後」，主要工作會在主要節點上執行**應用程式命令**。 應用程式命令是多重執行個體工作自有的命令列，而且只有主要工作能執行。 在 [MS-MPI][msmpi_msdn] 架構的方案中，您會在這裡使用 `mpiexec.exe` 執行已啟用 MPI 的應用程式。

> [!NOTE]
> 雖然「多重執行個體工作」在功能上不同，但不是特殊的工作類型，例如 [StartTask][net_starttask] 或 [JobPreparationTask][net_jobprep]。 多重執行個體工作只是已設定多重執行個體設定的 Standard Batch 工作 (Batch .NET 中的 [CloudTask][net_task])。 在本文中，我們將它稱為 **多重執行個體工作**。
>
>

## <a name="requirements-for-multi-instance-tasks"></a>多重執行個體工作的需求
多重執行個體工作需要有**已啟用節點間通訊**和**已停用並行工作執行**的集區。 若要停用並行工作執行，請將 [CloudPool.MaxTasksPerComputeNode](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool#Microsoft_Azure_Batch_CloudPool_MaxTasksPerComputeNode) 屬性設定為 1。

> [!NOTE]
> 批次[限制](batch-quota-limit.md#other-limits)已啟用節點間通訊之集區的大小。


此程式碼片段會顯示如何使用批次 .NET 程式庫來建立要供多重執行個體工作使用的集區。

```csharp
CloudPool myCloudPool =
    myBatchClient.PoolOperations.CreatePool(
        poolId: "MultiInstanceSamplePool",
        targetDedicatedComputeNodes: 3
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Multi-instance tasks require inter-node communication, and those nodes
// must run only one task at a time.
myCloudPool.InterComputeNodeCommunicationEnabled = true;
myCloudPool.MaxTasksPerComputeNode = 1;
```

> [!NOTE]
> 如果您嘗試在已停用節點間通訊，或「maxTasksPerNode」  值大於 1 的集區中執行多重執行個體工作，則永遠不會排定工作--它會無限期停留在「作用中」狀態。 


### <a name="use-a-starttask-to-install-mpi"></a>使用 StartTask 安裝 MPI
若要執行具有多重執行個體工作的 MPI 應用程式，您必須先在集區的計算節點上安裝 MPI 實作 (例如 MS-MPI 或 Intel MPI)。 這是使用 [StartTask][net_starttask] 的好時機，每當節點加入集區或重新啟動時，它就會執行。 此程式碼片段會建立 StartTask，指定 MS-MPI 安裝套件來做為[資源檔][net_resourcefile]。 資源檔下載至節點後，便會執行啟動工作的命令列。 在此案例中，命令列會執行 MS-MPI 的自動安裝。

```csharp
// Create a StartTask for the pool which we use for installing MS-MPI on
// the nodes as they join the pool (or when they are restarted).
StartTask startTask = new StartTask
{
    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
    UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin)),
    WaitForSuccess = true
};
myCloudPool.StartTask = startTask;

// Commit the fully configured pool to the Batch service to actually create
// the pool and its compute nodes.
await myCloudPool.CommitAsync();
```

### <a name="remote-direct-memory-access-rdma"></a>遠端直接記憶體存取 (RDMA)
當您在 Batch 集區中選擇 [支援 RDMA 大小](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (如 A9 計算節點) 時，MPI 應用程式可以利用 Azure 高效能、低延遲的遠端直接記憶體存取 (RDMA) 網路。

在下列文章中尋找指定為「支援 RDMA」的大小︰

* **CloudServiceConfiguration** 集區

  * [雲端服務的大小](../cloud-services/cloud-services-sizes-specs.md) (僅限 Windows)
* **VirtualMachineConfiguration** 集區

  * [Azure 中的虛擬機器大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux)
  * [Azure 中的虛擬機器大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows)

> [!NOTE]
> 若要在 [Linux 計算節點](batch-linux-nodes.md)上利用 RDMA，您必須在節點上使用 **Intel MPI**。 
>

## <a name="create-a-multi-instance-task-with-batch-net"></a>使用 Batch .NET 建立多個執行個體工作
既然我們已討論過集區需求和 MPI 套件安裝，現在讓我們建立多重執行個體工作。 在此片段中，我們會建立標準 [CloudTask][net_task]，然後設定其 [MultiInstanceSettings][net_multiinstance_prop] 屬性。 如先前所述，多重執行個體工作不是獨特的工作類型，而只是已設定多重執行個體設定的標準 Batch 工作。

```csharp
// Create the multi-instance task. Its command line is the "application command"
// and will be executed *only* by the primary, and only after the primary and
// subtasks execute the CoordinationCommandLine.
CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");

// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
// the primary and all subtasks.
myMultiInstanceTask.MultiInstanceSettings =
    new MultiInstanceSettings(numberOfNodes) {
    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
    CommonResourceFiles = new List<ResourceFile> {
    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
                     "MyMPIApplication.exe")
    }
};

// Submit the task to the job. Batch will take care of splitting it into subtasks and
// scheduling them for execution on the nodes.
await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);
```

## <a name="primary-task-and-subtasks"></a>主要工作和子工作
當您建立工作的多重執行個體設定時，您需要指定用來執行工作的計算節點數目。 當您將工作提交給作業時，Batch 服務會建立一個**主要工作**和足夠的**子工作**，而且合計符合您指定的節點數。

系統會指派範圍介於 0 到 *numberOfInstances* - 1 的整數識別碼給這些工作。 識別碼 0 的工作是主要工作，其他所有識別碼都是子工作。 比方說，如果您為工作建立下列多重執行個體設定，則主要工作的識別碼為 0，而子工作的識別碼為 1 到 9。

```csharp
int numberOfNodes = 10;
myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);
```

### <a name="master-node"></a>主要節點
當您提交多重執行個體工作時，Batch 服務能將一個計算節點指定為「主要」節點，然後將主要工作排程在主要節點上執行。 它會將子工作排程在配置給多重執行個體工作所剩餘的節點上執行。

## <a name="coordination-command"></a>協調命令
主要工作和子工作都會執行 **協調命令** 。

阻止叫用協調命令 -- 在所有子工作的協調命令順利傳回之前，Batch 不會執行應用程式命令。 因此，協調命令應該啟動任何所需的背景服務，確認它們已準備好可供使用，然後結束。 比方說，在使用 MS-MPI 第 7 版的方案中，此協調命令會在節點上啟動 SMPD 服務，然後結束：

```
cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d
```

請注意此協調命令中使用 `start` 。 這是必要的，因為 `smpd.exe` 應用程式不會在執行後立即傳回。 如果不使用 [start][cmd_start] 命令，此協調命令就不會傳回，因此將阻止執行應用程式命令。

## <a name="application-command"></a>應用程式命令
主要工作及所有子工作完成執行協調命令之後，「只有」 主要工作會執行多重執行個體工作的命令列。 我們將此命令列稱為 **應用程式命令** ，以便與協調命令有所區別。

針對 MS-MPI 應用程式，使用應用程式命令以 `mpiexec.exe`執行已啟用 MPI 的應用程式。 例如，以下是使用 MS-MPI 第 7 版的方案所執行的應用程式命令：

```
cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe
```

> [!NOTE]
> 因為 MS-MPI 的 `mpiexec.exe` 預設會使用 `CCP_NODES` 變數 (請參閱[環境變數](#environment-variables))，上述範例應用程式命令列會排除它。
>
>

## <a name="environment-variables"></a>環境變數
Batch 會在配置給多重執行個體工作的計算節點上建立幾個多個執行個體工作特有的[環境變數][msdn_env_var]。 您的協調和應用程式命令列可以參考這些環境變數，它們執行的指令碼和程式也可以。

以下是 Batch 服務為多重執行個體工作所建立的環境變數︰

* `CCP_NODES`
* `AZ_BATCH_NODE_LIST`
* `AZ_BATCH_HOST_LIST`
* `AZ_BATCH_MASTER_NODE`
* `AZ_BATCH_TASK_SHARED_DIR`
* `AZ_BATCH_IS_CURRENT_NODE_MASTER`

如需這些變數和其他 Batch 計算節點環境變數的完整詳細資料 (包括其內容和可見性)，請參閱[計算節點環境變數][msdn_env_var]。

> [!TIP]
> Batch Linux MPI 程式碼範例含有幾個如何使用這些環境變數的範例。 [coordination-cmd][coord_cmd_example] Bash 指令碼能從 Azure 儲存體下載一般應用程式和輸入檔案、啟用主要節點上的網路檔案系統 (NFS) 共用，以及將配置給多重執行個體工作的其他節點設定為 NFS 用戶端。
>
>

## <a name="resource-files"></a>資源檔
多重執行個體工作需要考量兩組資源檔：「所有」工作 (主要工作和子工作) 下載的**一般資源檔**，以及為多重執行個體工作本身指定的**資源檔** (「只有主要工作」會下載)。

您可以在工作的多重執行個體設定中指定一或多個 **一般資源檔** 。 主要工作及所有子工作會從 [Azure 儲存體](../storage/common/storage-introduction.md)，將這些一般資源檔下載到每個節點的**工作共用目錄**。 您可以使用 `AZ_BATCH_TASK_SHARED_DIR` 環境變數從應用程式命令和協調命令列存取工作共用目錄。 每個配置給多重執行個體工作之節點上的 `AZ_BATCH_TASK_SHARED_DIR` 路徑均完全相同，因此您可以讓主要工作和所有子工作共用一個協調命令。 從遠端存取方面來看，Batch 不會「共用」目錄，不過您可以將它當做掛接點或共用點，如同先前有關環境變數的提示所述。

依預設，您為多重執行個體工作本身所指定的資源檔會下載到工作的工作目錄 ( `AZ_BATCH_TASK_WORKING_DIR`) 中。 如前文所述，相較於一般資源檔，只有主要工作會下載為多重執行個體工作本身所指定的資源檔。

> [!IMPORTANT]
> 在命令列中，請一律使用環境變數 `AZ_BATCH_TASK_SHARED_DIR` 和 `AZ_BATCH_TASK_WORKING_DIR` 來參考這些目錄。 請勿嘗試以手動方式建構路徑。
>
>

## <a name="task-lifetime"></a>工作存留期
主要工作的存留期控制整個多重執行個體工作的存留期。 當主要工作結束時，所有子工作就會終止。 主要工作的結束代碼就是工作的結束代碼，因此在重試用途上用於判斷工作成功或失敗。

如果任何子工作失敗，比方說結束時傳回碼不是零，則整個多重執行個體工作會失敗。 接著會終止並重試多重執行個體工作，直到觸及重試限制為止。

當您刪除多重執行個體工作時，Batch 服務也會刪除主要工作和所有子工作。 所有子工作目錄及其檔案會從計算節點中刪除，如同在標準工作中一樣。

多重執行個體工作的 [TaskConstraints][net_taskconstraints] (例如 [MaxTaskRetryCount][net_taskconstraint_maxretry]、[MaxWallClockTime][net_taskconstraint_maxwallclock], 和 [RetentionTime][net_taskconstraint_retention] 屬性) 都視為用於標準工作，並套用至主要工作和所有子工作。 不過，如果您在多重執行個體工作新增到作業之後變更 [RetentionTime][net_taskconstraint_retention] 屬性，這項變更只會套用到主要作業。 所有的子工作會繼續使用原始 [RetentionTime][net_taskconstraint_retention]。

如果最近的工作是多重執行個體工作的一部分，計算節點的最近工作清單會反映子工作的識別碼。

## <a name="obtain-information-about-subtasks"></a>取得子工作的相關資訊
若要使用 Batch .NET 程式庫取得子工作的詳細資訊，請呼叫 [CloudTask.ListSubtasks][net_task_listsubtasks] 方法。 這個方法會傳回所有子工作的相關資訊，以及已執行工作的計算節點的相關資訊。 您可以根據這項資訊判斷每項子工作的根目錄、集區識別碼、其目前狀態、結束代碼等等。 您可以使用這項資訊結合 [PoolOperations.GetNodeFile][poolops_getnodefile] 方法，以取得子工作的檔案。 請注意，這個方法不會傳回主要工作 (識別碼 0) 的相關資訊。

> [!NOTE]
> 除非另有指明，否則在多重執行個體 [CloudTask][net_task] 本身執行的 Batch .NET 方法「只會」套用到主要工作。 例如，當您在多重執行個體工作上呼叫 [CloudTask.ListNodeFiles][net_task_listnodefiles] 方法時，只會傳回主要工作的檔案。
>
>

下列程式碼片段示範如何取得子工作資訊，以及從它們執行所在的節點要求檔案的內容。

```csharp
// Obtain the job and the multi-instance task from the Batch service
CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");

// Now obtain the list of subtasks for the task
IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();

// Asynchronously iterate over the subtasks and print their stdout and stderr
// output if the subtask has completed
await subtasks.ForEachAsync(async (subtask) =>
{
    Console.WriteLine("subtask: {0}", subtask.Id);
    Console.WriteLine("exit code: {0}", subtask.ExitCode);

    if (subtask.State == SubtaskState.Completed)
    {
        ComputeNode node =
            await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
                                                                 subtask.ComputeNodeInformation.ComputeNodeId);

        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardOutFileName);
        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardErrorFileName);
        stdOut = await stdOutFile.ReadAsStringAsync();
        stdErr = await stdErrFile.ReadAsStringAsync();

        Console.WriteLine("node: {0}:", node.Id);
        Console.WriteLine("stdout.txt: {0}", stdOut);
        Console.WriteLine("stderr.txt: {0}", stdErr);
    }
    else
    {
        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
    }
});
```

## <a name="code-sample"></a>程式碼範例
GitHub 上的 [MultiInstanceTasks][github_mpi] 程式碼範例示範如何使用多重執行個體工作在 Batch 計算節點上執行 [MS-MPI][msmpi_msdn] 應用程式。 請遵循[準備](#preparation)和[執行](#execution)中的步驟來執行範例。

### <a name="preparation"></a>準備工作
1. 遵循[如何編譯及執行簡單的 MS-MPI 程式][msmpi_howto]中的前兩個步驟。 這可滿足下一個步驟的必要條件。
2. 建置 [MPIHelloWorld][helloworld_proj] 範例 MPI 程式的*發行*版本。 這是多重執行個體工作將在計算節點上執行的程式。
3. 建立包含 `MPIHelloWorld.exe` (您在步驟 2 所建置) 和 `MSMpiSetup.exe` (您在步驟 1 所下載) 的 zip 檔案。 您會在下一個步驟中，將此 zip 檔案上傳為應用程式套件。
4. 使用 [Azure 入口網站][portal]建立稱為「MPIHelloWorld」的 Batch [應用程式](batch-application-packages.md)，並將您在上一個步驟所建立的 zip 檔案指定為應用程式套件「1.0」版。 如需詳細資訊，請參閱[上傳及管理應用程式](batch-application-packages.md#upload-and-manage-applications)。

> [!TIP]
> 建置 `MPIHelloWorld.exe` 的「發行」版本，以便不需要在應用程式套件中包含任何其他相依項目 (例如，`msvcp140d.dll` 或 `vcruntime140d.dll`)。
>
>

### <a name="execution"></a>執行
1. 從 GitHub 下載 [azure-batch-samples][github_samples_zip]。
2. 在 Visual Studio 2017 中開啟 MultiInstanceTasks **方案**。 `MultiInstanceTasks.sln` 方案檔位於︰

    `azure-batch-samples\CSharp\ArticleProjects\MultiInstanceTasks\`
3. 在 **Microsoft.Azure.Batch.Samples.Common** 專案的 `AccountSettings.settings` 中輸入 Batch 和儲存體帳戶的認證。
4. **建置並執行** MultiInstanceTasks 方案，以在 Batch 集區的計算節點上執行 MPI 範例應用程式。
5. *選擇性*︰請先使用 [Azure 入口網站][portal] 或 [Batch Explorer][batch_labs] 檢查範例集區、作業和工作 ("MultiInstanceSamplePool"、"MultiInstanceSampleJob"、"MultiInstanceSampleTask")，然後才刪除資源。

> [!TIP]
> 如果您沒有 Visual Studio，您可以免費下載 [Visual Studio Community][visual_studio]。
>
>

`MultiInstanceTasks.exe` 的輸出大致如下：

```
Creating pool [MultiInstanceSamplePool]...
Creating job [MultiInstanceSampleJob]...
Adding task [MultiInstanceSampleTask] to job [MultiInstanceSampleJob]...
Awaiting task completion, timeout in 00:30:00...

Main task [MultiInstanceSampleTask] is in state [Completed] and ran on compute node [tvm-1219235766_1-20161017t162002z]:
---- stdout.txt ----
Rank 2 received string "Hello world" from Rank 0
Rank 1 received string "Hello world" from Rank 0

---- stderr.txt ----

Main task completed, waiting 00:00:10 for subtasks to complete...

---- Subtask information ----
subtask: 1
        exit code: 0
        node: tvm-1219235766_3-20161017t162002z
        stdout.txt:
        stderr.txt:
subtask: 2
        exit code: 0
        node: tvm-1219235766_2-20161017t162002z
        stdout.txt:
        stderr.txt:

Delete job? [yes] no: yes
Delete pool? [yes] no: yes

Sample complete, hit ENTER to exit...
```

## <a name="next-steps"></a>後續步驟
* Microsoft HPC 和 Azure Batch 小組部落格討論 [Azure Batch 上的 Linux MPI 支援][blog_mpi_linux]，其中包含搭配使用 [OpenFOAM][openfoam] 和 Batch 的相關資訊。 您可以在 [GitHub][github_mpi] 上找到 OpenFOAM 範例的 Python 程式碼範例。
* 了解如何[建立 Linux 計算節點的集區](batch-linux-nodes.md)以在 Azure Batch MPI 方案中使用。

[helloworld_proj]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks/MPIHelloWorld

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[blog_mpi_linux]: https://blogs.technet.microsoft.com/windowshpc/2016/07/20/introducing-mpi-support-for-linux-on-azure-batch/
[cmd_start]: https://technet.microsoft.com/library/cc770297.aspx
[coord_cmd_example]: https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/article_samples/mpi/data/linux/openfoam/coordination-cmd
[github_mpi]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[msdn_env_var]: https://msdn.microsoft.com/library/azure/mt743623.aspx
[msmpi_msdn]: https://msdn.microsoft.com/library/bb524831.aspx
[msmpi_sdk]: http://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: http://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx
[openfoam]: http://www.openfoam.com/
[visual_studio]: https://www.visualstudio.com/vs/community/

[net_jobprep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[portal]: https://portal.azure.com
[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "多重執行個體概觀"
