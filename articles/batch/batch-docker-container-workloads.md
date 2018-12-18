---
title: Azure Batch 上的容器工作負載 | Microsoft Docs
description: 了解如何在 Azure Batch 上透過容器映像執行應用程式。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.workload: na
ms.date: 06/04/2018
ms.author: danlep
ms.openlocfilehash: a85db0315a2ee8aa9fd34b8c18893f4cb1068528
ms.sourcegitcommit: e32ea47d9d8158747eaf8fee6ebdd238d3ba01f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39090957"
---
# <a name="run-container-applications-on-azure-batch"></a>在 Azure Batch 上執行容器應用程式

Azure Batch 可讓您在 Azure 上執行及縮放大量批次運算作業。 Batch 工作可以直接在 Batch 集區中的虛擬機器 (節點) 上執行，但您也可以將 Batch 集區設定為在節點上的 Docker 相容容器中執行工作。 本文會示範如何建立可支援執行中容器工作的運算節點集區，以及如何在集區上執行容器工作。 

您應該要熟悉容器概念，以及如何建立 Batch 集區與作業。 程式碼範例使用了 Batch .NET 和 Python SDK。 您也可以使用其他 Batch SDK 和工具，包括 Azure 入口網站，來建立啟用容器的 Batch 集區，並執行容器工作。

## <a name="why-use-containers"></a>為何要使用容器？

使用容器可讓您輕鬆地執行 Batch 工作，無須管理環境和相依性，即可執行應用程式。 容器會將應用程式部署為輕量、可攜且自給自足的單位，以便在多種不同的環境中執行。 例如，您可以在本機建置及測試容器，然後將容器映像上傳至 Azure 或其他位置中的登錄。 容器部署模型可確保您應用程式的執行階段環境一律會正確地安裝和設定 (無論您的應用程式裝載在何處)。 Batch 中容器型的工作也可利用非容器工作的功能，包括應用程式套件及資源檔和輸出檔案的管理。 

## <a name="prerequisites"></a>必要條件

* **SDK 版本**：自以下版本起的 Batch SDK 可支援容器映像：
    * Batch REST API 2017-09-01.6.0 版
    * Batch .NET SDK 8.0.0 版
    * Batch Python SDK 4.0 版
    * Batch Java SDK 3.0 版
    * Batch Node.js SDK 3.0 版

* **帳戶**：在您的 Azure 訂用帳戶中，您需要建立 Batch 帳戶，並選擇性地建立 Azure 儲存體帳戶。

* **支援的虛擬機器映像**：只有從下一節＜支援的虛擬機器映像＞中所述的映像，使用虛擬機器設定建立的集區，才可支援容器。 如果您提供了自訂映像，請參閱下一節中的考量，以及[使用受控自訂映像來建立虛擬機器的集區](batch-custom-images.md)中的需求。 

### <a name="limitations"></a>限制

* Batch 只會對 Linux 集區上執行的容器提供 RDMA 支援

* 對於 Windows 容器工作負載，建議您為集區選擇多核心 VM 大小

## <a name="supported-virtual-machine-images"></a>支援的虛擬機器映像

請使用下列其中一個支援的 Windows 或 Linux 映像，來建立適用於容器工作負載的虛擬機器計算節點集區。 如需與 Batch 相容的 Marketplace 映像詳細資訊，請參閱[虛擬機器映像的清單](batch-linux-nodes.md#list-of-virtual-machine-images)。 

### <a name="windows-images"></a>Windows 映像

對於 Windows 容器工作負載，Batch 目前支援 Azure Marketplace 中的 **Windows Server 2016 Datacenter with Containers** 映像。 Windows 只支援 Docker 容器映像。

您也可以從 Windows 上執行 Docker 的虛擬機器中建立自訂映像。

### <a name="linux-images"></a>Linux 映像

針對 Linux 容器工作負載，Batch 目前支援下列 Microsoft Azure Batch 在 Azure Marketplace 中發行的 Linux 映像：

* **適用於 Azure Batch 容器集區的 CentOS**

* **適用於 Azure Batch 容器集區的 CentOS (包含 RDMA 驅動程式)**

* **適用於 Azure Batch 容器集區的 Ubuntu Server**

* **適用於 Azure Batch 容器集區的 Ubuntu Server (包含 RDMA 驅動程式)**

這些映像僅支援在 Azure Batch 集區中使用。 它們具有：

* 預先安裝的 [Moby](https://github.com/moby/moby) 容器執行階段 

* 預先安裝的 NVIDIA GPU 驅動程式，可簡化 Azure N 系列虛擬機器上的部署

* 包含或不包含預先安裝 RDMA 驅動程式的映像；這些驅動程式允許集區節點在具備 RDMA 功能的虛擬機器大小部署時，存取 Azure RDMA 網路  

您也可以從與 Batch 相容的其中一個 Linux 發佈上執行 Docker 的虛擬機器建立自訂映像。 如果您選擇提供您自己自訂的 Linux 映像，請參閱[使用受控自訂映像來建立虛擬機器的集區](batch-custom-images.md)中的指示。

如需自訂映像上的 Docker 支援，請安裝 [Docker Community Edition (CE)](https://www.docker.com/community-edition) 或 [Docker Enterprise Edition (EE)](https://www.docker.com/enterprise-edition)。

使用自訂 Linux 映像的其他考量：

* 若要在使用自訂映像時利用 Azure N 系列大小的 GPU 效能，請預先安裝 NVIDIA 驅動程式。 此外，您需要安裝適用於 NVIDIA GPU 的 Docker 引擎公用程式 [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker)。

* 若要存取 Azure RDMA 網路，請使用具備 RDMA 功能的虛擬機器大小。 必要的 RDMA 驅動程式會安裝在 Batch 支援的 CentOS HPC 和 Ubuntu 映像上。 若要執行 MPI 工作負載，則可能需要其他設定。 請參閱[在 Batch 集區中使用具備 RDMA 功能或已啟用 GPU 功能的執行個體](batch-pool-compute-intensive-sizes.md)。


## <a name="container-configuration-for-batch-pool"></a>Batch 集區的容器設定

若要啟用 Batch 集區以執行容器工作負載，您必須在集區的 [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) 物件中指定 [ContainerConfiguration](/dotnet/api/microsoft.azure.batch.containerconfiguration) 設定。 (本文提供 Batch .NET API 參考的連結。 對應設定位於 [Batch Python](/python/api/azure.batch) API。)

不論是否有預先擷取的容器映像，您都可以建立啟用容器的集區，如下列範例所示。 提取 (或預先擷取) 程序可讓您從 Docker Hub 或網際網路上的另一個容器登錄預先載入容器映像。 為了達到最佳效能，請使用與 Batch 帳戶位於相同區域中的 [Azure 容器登錄](../container-registry/container-registry-intro.md)。

預先擷取容器映像的好處是，當工作第一次開始執行時，不必等待容器映像下載。 容器設定會在集區建立時將容器映像提取到虛擬機器中。 然後，在集區上執行的工作就可以參考容器映像清單和容器執行選項。


### <a name="pool-without-prefetched-container-images"></a>沒有預先擷取容器映像的集區

若要在沒有預先擷取容器映像的情況下，設定啟用容器的集區，請定義 `ContainerConfiguration` 和 `VirtualMachineConfiguration` 物件，如下列 Python 範例所示。 這個範例會使用 Marketplace 中 Azure Batch 容器集區映像的 Ubuntu Server。


```python
image_ref_to_use = batch.models.ImageReference(
        publisher='microsoft-azure-batch',
        offer='ubuntu-server-container',
        sku='16-04-lts',
        version='latest')

"""
Specify container configuration. This is required even though there are no prefetched images.
"""

container_conf = batch.models.ContainerConfiguration()

new_pool = batch.models.PoolAddParameter(
        id=pool_id,
        virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
            image_reference=image_ref_to_use,
            container_configuration=container_conf,
            node_agent_sku_id='batch.node.ubuntu 16.04'),
        vm_size='STANDARD_D1_V2',
        target_dedicated_nodes=1)
...
```


### <a name="prefetch-images-for-container-configuration"></a>為容器設定預先擷取映像

若要在集區上預先擷取容器映像，請將容器映像清單 (`container_image_names`，在 Python 中) 新增至 `ContainerConfiguration`。 

下列基本 Python 範例示範如何從 [Docker 中樞](https://hub.docker.com)預先擷取標準 Ubuntu 容器映像。

```python
image_ref_to_use = batch.models.ImageReference(
    publisher='microsoft-azure-batch',
    offer='ubuntu-server-container',
    sku='16-04-lts',
    version='latest')

"""
Specify container configuration, fetching the official Ubuntu container image from Docker Hub. 
"""

container_conf = batch.models.ContainerConfiguration(container_image_names=['ubuntu'])

new_pool = batch.models.PoolAddParameter(
    id=pool_id,
    virtual_machine_configuration=batch.models.VirtualMachineConfiguration(
        image_reference=image_ref_to_use,
        container_configuration=container_conf,
        node_agent_sku_id='batch.node.ubuntu 16.04'),
    vm_size='STANDARD_D1_V2',
    target_dedicated_nodes=1)
...
```


下列範例 C# 範例假設您想要從 [Docker 中樞](https://hub.docker.com)預先擷取 TensorFlow 映像。 此範例包含在集區節點上 VM 主機中執行的啟動工作。 舉例而言，您可以在主機中執行啟動工作，以掛接可以從容器存取的檔案伺服器。

```csharp

ImageReference imageReference = new ImageReference(
    publisher: "microsoft-azure-batch",
    offer: "ubuntu-server-container",
    sku: "16-04-lts",
    version: "latest");

// Specify container configuration, prefetching Docker images
ContainerConfiguration containerConfig = new ContainerConfiguration(
    containerImageNames: new List<string> { "tensorflow/tensorflow:latest-gpu" } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    containerConfiguration: containerConfig,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");

// Set a native host command line start task
StartTask startTaskNative = new StartTask( CommandLine: "<native-host-command-line>" );

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration, startTaskContainer);
...
```


### <a name="prefetch-images-from-a-private-container-registry"></a>從私人容器登錄中預先擷取映像

您也可以透過驗證私人容器登錄伺服器，來預先擷取容器映像。 在下列範例中，`ContainerConfiguration` 和 `VirtualMachineConfiguration` 物件會從私人 Azure 容器登錄中預先擷取私人 TensorFlow 映像。 映像參考與先前範例相同。

```csharp
// Specify a container registry
ContainerRegistry containerRegistry = new ContainerRegistry (
    registryServer: "myContainerRegistry.azurecr.io",
    username: "myUserName",
    password: "myPassword");

// Create container configuration, prefetching Docker images from the container registry
ContainerConfiguration containerConfig = new ContainerConfiguration(
    containerImageNames: new List<string> {
        "myContainerRegistry.azurecr.io/tensorflow/tensorflow:latest-gpu" },
    containerRegistries: new List<ContainerRegistry> { containerRegistry } );

// VM configuration
VirtualMachineConfiguration virtualMachineConfiguration = new VirtualMachineConfiguration(
    imageReference: imageReference,
    containerConfiguration: containerConfig,
    nodeAgentSkuId: "batch.node.ubuntu 16.04");

// Create pool
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: 4,
    virtualMachineSize: "Standard_NC6",
    virtualMachineConfiguration: virtualMachineConfiguration);
...
```


## <a name="container-settings-for-the-task"></a>工作的容器設定

若要在計算節點上執行容器工作，您必須指定容器專屬的設定，例如工作執行選項、要使用的映像和登錄。

使用工作類別的 `ContainerSettings` 屬性來設定容器專屬設定。 這些設定會由 [TaskContainerSettings](/dotnet/api/microsoft.azure.batch.taskcontainersettings) 類別定義。

如果您在容器映像上執行工作，[雲端工作](/dotnet/api/microsoft.azure.batch.cloudtask)和[作業管理員工作](/dotnet/api/microsoft.azure.batch.cloudjob.jobmanagertask)會需要容器設定。 但是，[啟動工作](/dotnet/api/microsoft.azure.batch.starttask)、[作業準備工作](/dotnet/api/microsoft.azure.batch.cloudjob.jobpreparationtask)和[作業解除工作](/dotnet/api/microsoft.azure.batch.cloudjob.jobreleasetask)不需要容器設定 (也就是這些工作可以在容器內容中執行或直接在節點上執行)。

Azure Batch 容器工作的命令列是在容器的工作目錄中執行，與 Batch 為一般 (非容器) 工作設定的環境非常類似：

* 以遞迴方式位於 `AZ_BATCH_NODE_ROOT_DIR` (Azure Batch 在節點上的根目錄) 下方的所有目錄都會對應至容器
* 所有工作環境變數都會對應至容器
* 應用程式工作目錄會設定為與一般工作相同，因此您可以使用應用程式套件和資源檔等功能

由於 Batch 變更您容器中的預設工作目錄，因此工作會在不同於一般容器進入點的位置執行 (例如，預設為 `c:\` (Windows 容器上) 或 `/` (Linux 上))。 確定您工作命令列或容器進入點指定絕對路徑 (如果尚未如此設定)。

下列 Python 程式碼片段顯示的基本命令列，是在提取自 Docker 中樞的 Ubuntu 容器中執行。 容器執行選項對工作執行的 `docker create` 命令是額外的引數。 在這裡，`--rm` 選項會在工作完成之後移除容器。

```python
task_id = 'sampletask'
task_container_settings = batch.models.TaskContainerSettings(
    image_name='ubuntu', 
    container_run_options='--rm')
task = batch.models.TaskAddParameter(
    id=task_id,
    command_line='/bin/echo hello',
    container_settings=task_container_settings
)

```

下列 C# 範例會示範雲端工作的基本容器設定：

```csharp
// Simple container task command

string cmdLine = "c:\myApp.exe";

TaskContainerSettings cmdContainerSettings = new TaskContainerSettings (
    imageName: "tensorflow/tensorflow:latest-gpu",
    containerRunOptions: "--rm --read-only"
    );

CloudTask containerTask = new CloudTask (
    id: "Task1",
    containerSettings: cmdContainerSettings,
    commandLine: cmdLine); 
```


## <a name="next-steps"></a>後續步驟

* 如何透過 [Batch Shipyard 訣竅](https://github.com/Azure/batch-shipyard/tree/master/recipes)在 Azure Batch 上輕鬆部署容器工作負載，另請參閱 [Batch Shipyard](https://github.com/Azure/batch-shipyard) 工具組。

* 如需有關在 Linux 上安裝和使用 Docker CE 的詳細資訊，請參閱 [Docker](https://docs.docker.com/engine/installation/) 文件。

* 如需使用自訂映像的詳細資訊，請參閱[使用受控自訂映像來建立虛擬機器的集區](batch-custom-images.md)。

* 深入了解 [Moby 專案](https://mobyproject.org/)，這是一個用來建立容器型系統的架構。