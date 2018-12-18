---
title: Azure Batch 計算節點環境變數 | Microsoft Docs
description: Azure Batch 分析的計算節點環境變數參考。
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 05/05/2017
ms.author: danlep
ms.openlocfilehash: ca8d6a6484cd1f145e7d807681bf2d012f2399e0
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30312696"
---
# <a name="azure-batch-compute-node-environment-variables"></a>Azure Batch 計算節點環境變數
[Azure Batch 服務](https://azure.microsoft.com/services/batch/)會在計算節點上設定下列環境變數。 您可以在工作命令列中，以及由該命令列執行的程式及指令碼中，參照這些環境變數。

如需有關將環境變數搭配 Batch 使用的詳細資訊，請參閱[工作的環境設定](https://docs.microsoft.com/azure/batch/batch-api-basics#environment-settings-for-tasks)。

## <a name="environment-variable-visibility"></a>環境變數可見性

這些環境變數僅適用於**工作使用者**的內容中，也就是工作執行所在節點上的使用者帳戶。 如果您透過遠端桌面通訊協定 (RDP) 或安全殼層 (SSH) [遠端連線](https://azure.microsoft.com/documentation/articles/batch-api-basics/#connecting-to-compute-nodes)到計算節點，並列出環境變數，您將*不會*看見這些環境變數。 這是因為用於遠端連線的使用者帳戶與工作所用的帳戶不同。

## <a name="command-line-expansion-of-environment-variables"></a>環境變數的命令列擴充功能

工作在計算節點上執行的命令列不會在殼層下執行。 因此，這些命令列無法以原生方式利用如環境變數擴充功能等殼層功能 (這包括 `PATH`)。 若要利用這類功能，您必須在命令列中**叫用殼層**。 例如，在 Windows 計算節點上啟動 `cmd.exe`，或在 Linux 節點上啟動 `/bin/sh`：

`cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

`/bin/sh -c MyTaskApplication $MY_ENV_VAR`

## <a name="environment-variables"></a>環境變數

| 變數名稱                     | 說明                                                              | 可用性 | 範例 |
|-----------------------------------|--------------------------------------------------------------------------|--------------|---------|
| AZ_BATCH_ACCOUNT_NAME           | 工作所屬之批次帳戶的名稱。                  | 所有工作。   | mybatchaccount |
| AZ_BATCH_CERTIFICATES_DIR       | 系統為 Linux 計算節點儲存憑證所在[工作工作目錄][files_dirs]內的目錄。 請注意，這個環境變數不會套用至 Windows 計算節點。                                                  | 所有工作。   |  /mnt/batch/tasks/workitems/batchjob001/job-1/task001/certs |
| AZ_BATCH_JOB_ID                 | 工作所屬之作業的 ID。 | 啟動工作之外的所有工作。 | batchjob001 |
| AZ_BATCH_JOB_PREP_DIR           | 節點上作業準備[工作目錄][files_dirs]的完整路徑。 | 啟動工作與作業準備工作之外的所有工作。 僅適用於透過作業準備工作設定作業時。 | C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation |
| AZ_BATCH_JOB_PREP_WORKING_DIR   | 節點上作業準備[工作工作目錄][files_dirs]的完整路徑。 | 啟動工作與作業準備工作之外的所有工作。 僅適用於透過作業準備工作設定作業時。 | C:\user\tasks\workitems\jobprepreleasesamplejob\job-1\jobpreparation\wd |
| AZ_BATCH_NODE_ID                | 將工作指派至該節點的識別碼。 | 所有工作。 | tvm-1219235766_3-20160919t172711z |
| AZ_BATCH_NODE_ROOT_DIR          | 節點上所有 [Batch 目錄][files_dirs]其根目錄的完整路徑。 | 所有工作。 | C:\user\tasks |
| AZ_BATCH_NODE_SHARED_DIR        | 節點上[共用目錄][files_dirs]的完整路徑。 在節點上執行的所有工作皆具備此目錄的讀取/寫入存取權。 在其他節點上執行的工作沒有遠端存取此目錄 (它不是「共用的」網路目錄) 的權限。 | 所有工作。 | C:\user\tasks\shared |
| AZ_BATCH_NODE_STARTUP_DIR       | 節點上[啟動工作目錄][files_dirs]的完整路徑。 | 所有工作。 | C:\user\tasks\startup |
| AZ_BATCH_POOL_ID                | 執行工作之集區的 ID。 | 所有工作。 | batchpool001 |
| AZ_BATCH_TASK_DIR               | 節點上[工作目錄][files_dirs]的完整路徑。 此目錄包含工作的 `stdout.txt` 與 `stderr.txt`，以及 AZ_BATCH_TASK_WORKING_DIR。 | 所有工作。 | C:\user\tasks\workitems\batchjob001\job-1\task001 |
| AZ_BATCH_TASK_ID                | 目前工作的 ID。 | 啟動工作之外的所有工作。 | task001 |
| AZ_BATCH_TASK_WORKING_DIR       | 節點上[工作工作目錄][files_dirs]的完整路徑。 目前執行中工作具有此目錄的讀取/寫入存取權。 | 所有工作。 | C:\user\tasks\workitems\batchjob001\job-1\task001\wd |
| CCP_NODES                       | 各節點配置給[多重執行個體工作][multi_instance]的節點清單與核心數目。 列出節點與核心的格式為：`numNodes<space>node1IP<space>node1Cores<space>`<br/>`node2IP<space>node2Cores<space> ...`，其中節點數目後面會加上一或多個節點 IP 位址，及每個節點的核心數目。 |  多重執行個體的主要工作和子工作。 |`2 10.0.0.4 1 10.0.0.5 1` |
| AZ_BATCH_NODE_LIST              | 配置給[多重執行個體工作][multi_instance]的節點清單，清單格式為 `nodeIP;nodeIP`。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4;10.0.0.5` |
| AZ_BATCH_HOST_LIST              | 配置給[多重執行個體工作][multi_instance]的節點清單，清單格式為 `nodeIP,nodeIP`。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4,10.0.0.5` |
| AZ_BATCH_MASTER_NODE            | [多重執行個體工作][multi_instance]的主要工作執行所在的計算節點其 IP 位址與連接埠。 | 多重執行個體的主要工作和子工作。 | `10.0.0.4:6000`|
| AZ_BATCH_TASK_SHARED_DIR | [多重執行個體工作][multi_instance]的主要工作與每個子工作相同的目錄路徑。 此路徑存在於多重執行個體工作執行所在的每個節點上，且可由在該節點上執行的工作命令 ([協調命令][coord_cmd]與[應用程式命令][app_cmd]) 以讀取/寫入的方式存取。 在其他節點上執行的子工作或主要工作沒有遠端存取此目錄 (它不是「共用的」網路目錄) 的權限。 | 多重執行個體的主要工作和子工作。 | C:\user\tasks\workitems\multiinstancesamplejob\job-1\multiinstancesampletask |
| AZ_BATCH_IS_CURRENT_NODE_MASTER | 指定目前節點是否為[多重執行個體工作][multi_instance]的主要節點。 可能的值為 `true` 與 `false`。| 多重執行個體的主要工作和子工作。 | `true` |
| AZ_BATCH_NODE_IS_DEDICATED | 若為 `true`，目前的節點就是專用節點。 若為 `false`，它就是[低優先順序節點](batch-low-pri-vms.md)。 | 所有工作。 | `true` |

[files_dirs]: https://azure.microsoft.com/documentation/articles/batch-api-basics/#files-and-directories
[multi_instance]: https://azure.microsoft.com/documentation/articles/batch-mpi/
[coord_cmd]: https://azure.microsoft.com/documentation/articles/batch-mpi/#coordination-command
[app_cmd]: https://azure.microsoft.com/documentation/articles/batch-mpi/#application-command
