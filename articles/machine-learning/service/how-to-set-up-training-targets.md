---
title: 使用 Azure Machine Learning 服務為模型定型設定計算目標 | Microsoft Docs
description: 了解如何選取及設定用來將機器學習服務模型定型的定型環境 (計算目標)。 Azure Machine Learning 服務可讓您輕鬆切換定型環境。 在本機開始定型作業，並在需要相應放大時切換至雲端式計算目標。
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
ms.reviewer: larryfr
manager: cgronlun
ms.service: machine-learning
ms.component: core
ms.topic: article
ms.date: 09/24/2018
ms.openlocfilehash: e5b44ed2435986ffd500cade1f7c8ff8047d353d
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452292"
---
# <a name="select-and-use-a-compute-target-to-train-your-model"></a>選取並使用計算目標將您的模型定型

使用 Azure Machine Learning 服務時，您可以在幾個不同的環境中將您的模型定型。 這些模型 (稱為「計算目標」) 可位於本機或雲端。 在此文件中，您將會了解支援的計算目標類型有哪些，以及如何使用它們。

計算目標為執行您的定型指令碼，或將模型部署為 Web 服務時裝載模型的資源。 您可以使用 Azure Machine Learning SDK 或 CLI 來建立及管理它們。 如果您有透過其他程序 (例如 Azure 入口網站或 Azure CLI) 建立的計算目標，可以透過將它們附加至 Azure Machine Learning 服務工作區來使用它們。

您可以在自己的電腦上開始本機執行，然後相應增加或相應放大到其他環境，例如具備 GPU 或 Azure Batch AI 的遠端資料科學虛擬機器。 

## <a name="supported-compute-targets"></a>支援的計算目標

Azure Machine Learning 支援下列計算目標：

|計算目標| GPU 加速 | 自動化超參數調整 | 自動化模型選取 | 可在管線中使用|
|----|:----:|:----:|:----:|:----:|
|[本機電腦](#local)| 可能 | &nbsp; | ✓ | &nbsp; |
|[資料科學虛擬機器 (DSVM)](#dsvm) | ✓ | ✓ | ✓ | ✓ |
|[Azure Batch AI](#batch)| ✓ | ✓ | ✓ | ✓ | ✓ |
|[Azure HDInsight](#hdinsight)| &nbsp; | &nbsp; | &nbsp; | ✓ |

__[Azure 容器執行個體 (ACI)](#aci)__ 也可用來將模型定型。 它是無伺服器的雲端供應項目，價格實惠且容易建立及使用。 ACI 不支援 GPU 加速、自動化超參數調整，或自動化模型選取。 而且，它也無法在管線中使用。

計算目標之間的重要差異如下：
* __GPU 加速__：GPU 可與資料科學虛擬機器和 Azure Batch AI 搭配使用。 依據所安裝的硬體、驅動程式與架構而定，您可能可以使用自己本機電腦上的 GPU。
* __自動化超參數調整__：Azure Machine Learning 自動化超參數調整可協助您找出模型的最佳超參數。
* __自動化模型選取__：Azure Machine Learning 可以在建置模型時，很聰明地建議應選取哪一種演算法與超參數。 相較於手動嘗試不同組合，自動化模型選取可協助您更快速地組成高品質模型。 如需詳細資訊，請參閱[教學課程：使用自動化的 Azure Machine Learning 自動將分類模型定型](tutorial-auto-train-models.md)文件。
* __管線__：Azure Machine Learning 可讓您將不同工作 (例如定型與部署) 合併至管線中。 管線可以平行執行或依序執行，並提供可靠的自動化機制。 如需詳細資訊，請參閱[使用 Azure Machine Learning 服務建置機器學習服務管線](concept-ml-pipelines.md)文件。

您可以使用 Azure Machine Learning SDK、Azure CLI 或 Azure 入口網站來建立計算目標。 您也可以將現有的計算目標新增 (附加) 至工作區來使用它們。

> [!IMPORTANT]
> 您無法將現有的 Azure 容器執行個體附加至工作區。 必須建立新的執行個體。
>
> 您無法在工作區內建立 Azure HDInsight 叢集。 您必須改為附加現有叢集。

## <a name="workflow"></a>工作流程

使用 Azure Machine Learning 開發及部署模型的工作流程包含下列步驟：

1. 使用 Python 開發機器學習服務定型指令碼。
1. 建立及設定，或附加現有的計算目標。
1. 將定型指令碼提交至計算目標。
1. 檢查結果以找出最佳模型。
1. 在模型登錄中註冊模型。
1. 部署模型。

> [!IMPORTANT]
> 定型指令碼並未繫結到特定計算目標。 您可以先在本機電腦開始定型，然後切換計算目標，不需要重新撰寫定型指令碼。

從某個計算目標切換到其他計算目標的作業牽涉到建立[回合組態](concept-azure-machine-learning-architecture.md#run-configuration)。 回合組態會定義如何在計算目標上執行指令碼。

## <a name="training-scripts"></a>將指令碼定型

開始定型回合時，會提交包含定型指令碼的整個目錄。 系統會建立快照集並傳送到計算目標。 如需詳細資訊，請參閱[快照集](concept-azure-machine-learning-architecture.md#snapshot)。

## <a id="local"></a>本機電腦

在本機定型時，可以使用 SDK 提交定型作業。 您可以使用使用者管理或系統管理的環境執行定型作業。

### <a name="user-managed-environment"></a>使用者管理的環境

在使用者管理的環境中，您必須負責確保選擇用來執行指令碼的 Python 環境中有所有必要套件。 下列程式碼片段為針對使用者管理的環境設定定型的範例：

```python
from azureml.core.runconfig import RunConfiguration

# Editing a run configuration property on-fly.
run_config_user_managed = RunConfiguration()

run_config_user_managed.environment.python.user_managed_dependencies = True

# You can choose a specific Python environment by pointing to a Python path 
#run_config.environment.python.interpreter_path = '/home/ninghai/miniconda3/envs/sdk2/bin/python'
```

如需在使用者管理環境中示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/02.train-on-local/02.train-on-local.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/02.train-on-local/02.train-on-local.ipynb)。
  
### <a name="system-managed-environment"></a>系統管理的環境

系統管理的環境依賴 Conda 來管理相依性。 Conda 會建立名為 `conda_dependencies.yml` 的檔案，其中包含相依性清單。 接著，您可以要求系統建置新的 Conda 環境並在其中執行您的指令碼。 只要 `conda_dependencies.yml` 檔案維持不變，系統管理的環境之後就能重複使用。 

視所需相依性的大小而定，新環境的初始設定可能需要幾分鐘來完成。 下列程式碼片段示範如何建立取決於 scikit-learn 的系統管理環境：

```python
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies

run_config_system_managed = RunConfiguration()

run_config_system_managed.environment.python.user_managed_dependencies = False
run_config_system_managed.prepare_environment = True

# Specify conda dependencies with scikit-learn

run_config_system_managed.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
```

如需在系統管理環境中示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/02.train-on-local/02.train-on-local.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/02.train-on-local/02.train-on-local.ipynb)。

## <a id="dsvm"></a>資料科學虛擬機器

您的本機電腦可能沒有將模型定型時所需的計算或 GPU 資源。 此時可以透過新增額外的計算目標 (例如資料科學虛擬機器 (DSVM)) 來相應增加或相應放大定型程序。

> [!WARNING]
> Azure Machine Learning 只支援執行 Ubuntu 的虛擬機器。 建立新的虛擬機器或選取現有的虛擬機器時，必須選取使用 Ubuntu 的虛擬機器。

下列步驟會使用 SDK 將資料科學虛擬機器 (DSVM) 設定為定型目標：

1. 建立或附加虛擬機器
    
    * 若要建立新的 DSVM，請先檢查您是否已有名稱相同的 DSVM。如果沒有，請建立一個新的 VM：
    
        ```python
        from azureml.core.compute import DsvmCompute
        from azureml.core.compute_target import ComputeTargetException

        compute_target_name = 'mydsvm'

        try:
            dsvm_compute = DsvmCompute(workspace = ws, name = compute_target_name)
            print('found existing:', dsvm_compute.name)
        except ComputeTargetException:
            print('creating new.')
            dsvm_config = DsvmCompute.provisioning_configuration(vm_size = "Standard_D2_v2")
            dsvm_compute = DsvmCompute.create(ws, name = compute_target_name, provisioning_configuration = dsvm_config)
            dsvm_compute.wait_for_completion(show_output = True)
        ```
    * 若要附加現有的虛擬機器作為計算目標，則必須提供虛擬機器的完整網域名稱、登入名稱與密碼。  在範例中，請以 VM 的公用完整網域名稱或公用 IP 位址取代 ```<fqdn>```。 以 VM 的 SSH 使用者名稱與密碼取代 ```<username>``` 與 ```<password>```：

        ```python
        from azureml.core.compute import RemoteCompute

        dsvm_compute = RemoteCompute.attach(ws,
                                        name="attach-dsvm",
                                        username='<username>',
                                        address="<fqdn>",
                                        ssh_port=22,
                                        password="<password>")

        dsvm_compute.wait_for_completion(show_output=True)
    
   It takes around 5 minutes to create the DSVM instance.

1. Create a configuration for the DSVM compute target. Docker and conda are used to create and configure the training environment on DSVM:

    ```python
    from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies

    # Load the "cpu-dsvm.runconfig" file (created by the above attach operation) in memory
    run_config = RunConfiguration(framework = "python")

    # Set compute target to the Linux DSVM
    run_config.target = compute_target_name

    # Use Docker in the remote VM
    run_config.environment.docker.enabled = True

    # Use CPU base image from DockerHub
    run_config.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
    print('Base Docker image is:', run_config.environment.docker.base_image)

    # Ask system to provision a new one based on the conda_dependencies.yml file
    run_config.environment.python.user_managed_dependencies = False

    # Prepare the Docker and conda environment automatically when used the first time.
    run_config.prepare_environment = True

    # specify CondaDependencies obj
    run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])

    ```

1. 若要在完成作業時刪除計算資源，請使用下列程式碼：

    ```python
    dsvm_compute.delete()
    ```

如需在資料科學虛擬機器上示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/04.train-on-remote-vm/04.train-on-remote-vm.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/04.train-on-remote-vm/04.train-on-remote-vm.ipynb)。

## <a id="batch"></a>Azure Batch AI

如果將模型定型時需要花費很長的時間，可以使用 Azure Batch AI 將定型作業分散到雲端中的整個計算資源叢集。 也可以設定 Batch AI 以啟用 GPU 資源。

下列範例會依名稱尋找現有的 Batch AI 叢集。 如果找不到，就會建立一個叢集：

```python
from azureml.core.compute import BatchAiCompute
from azureml.core.compute import ComputeTarget

# choose a name for your cluster
batchai_cluster_name = ws.name + "cpu"

found = False
# see if this compute target already exists in the workspace
for ct in ws.compute_targets():
    print(ct.name, ct.type)
    if (ct.name == batchai_cluster_name and ct.type == 'BatchAI'):
        found = True
        print('found compute target. just use it.')
        compute_target = ct
        break
        
if not found:
    print('creating a new compute target...')
    provisioning_config = BatchAiCompute.provisioning_configuration(vm_size = "STANDARD_D2_V2", # for GPU, use "STANDARD_NC6"
                                                                #vm_priority = 'lowpriority', # optional
                                                                autoscale_enabled = True,
                                                                cluster_min_nodes = 1, 
                                                                cluster_max_nodes = 4)

    # create the cluster
    compute_target = ComputeTarget.create(ws,batchai_cluster_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current BatchAI cluster status, use the 'status' property    
    print(compute_target.status.serialize())
```

若要附加現有的 Batch AI 叢集作為計算目標，必須提供 Azure 資源識別碼。 若要從 Azure 入口網站取得資源識別碼，請使用下列步驟：
1. 在 [所有服務]底下搜尋 `Batch AI` 服務
1. 按一下您的叢集所屬工作區的名稱
1. 選取叢集
1. 按一下 [屬性]
1. 複製**識別碼**

下列範例會使用 SDK 將叢集附加到您的工作區。 在範例中，請以計算的任何名稱取代 `<name>`。 此名稱不需要與叢集名稱相符。 以上述 Azure 資源識別碼取代 `<resource-id>`：

```python
from azureml.core.compute import BatchAiCompute
BatchAiCompute.attach(workspace=ws,
                      name=<name>,
                      resource_id=<resource-id>)
```

您也可以使用下列 Azure CLI 命令來檢查 Batch AI 叢集與作業狀態：

- 檢查叢集狀態。 您可以查看有多少節點使用 `az batchai cluster list` 執行。
- 檢查作業狀態。 您可以查看有多少作業使用 `az batchai job list` 執行。

需要大約 5 分鐘的時間建立 Batch AI 叢集。

如需在 Batch AI 叢集中示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/training/03.train-hyperparameter-tune-deploy-with-tensorflow/03.train-hyperparameter-tune-deploy-with-tensorflow.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/training/03.train-hyperparameter-tune-deploy-with-tensorflow/03.train-hyperparameter-tune-deploy-with-tensorflow.ipynb)。

## <a name='aci'></a>Azure 容器執行個體 (ACI)

Azure 容器執行個體是隔離的容器，具備更快速的啟動時間，且不需要使用者管理任何虛擬機器。 Azure Machine Learning 服務使用 Linux 容器，此服務可在 westus、eastus、westeurope、northeurope、westus2 與 southeastasia 區域使用。 如需詳細資訊，請參閱[區域可用性](https://docs.microsoft.com/azure/container-instances/container-instances-quotas#region-availability)。 

下列範例示範如何使用 SDK 來建立 ACI 計算目標，並使用它將模型定型： 

```python
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies

# create a new runconfig object
run_config = RunConfiguration()

# signal that you want to use ACI to run script.
run_config.target = "containerinstance"

# ACI container group is only supported in certain regions, which can be different than the region the Workspace is in.
run_config.container_instance.region = 'eastus'

# set the ACI CPU and Memory 
run_config.container_instance.cpu_cores = 1
run_config.container_instance.memory_gb = 2

# enable Docker 
run_config.environment.docker.enabled = True

# set Docker base image to the default CPU-based image
run_config.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
#run_config.environment.docker.base_image = 'microsoft/mmlspark:plus-0.9.9'

# use conda_dependencies.yml to create a conda environment in the Docker image
run_config.environment.python.user_managed_dependencies = False

# auto-prepare the Docker image when used for the first time (if it is not already prepared)
run_config.auto_prepare_environment = True

# specify CondaDependencies obj
run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
```

可能需要幾秒鐘至幾分鐘的時間建立 ACI 計算目標。

如需在 Azure 容器執行個體中示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/03.train-on-aci/03.train-on-aci.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/03.train-on-aci/03.train-on-aci.ipynb)。

## <a id="hdinsight"></a>附加 HDInsight 叢集 

HDInsight 是巨量資料分析的熱門平台。 它提供了 Apache Spark，可用來將您的模型定型。

> [!IMPORTANT]
> 您必須先建立 HDInsight 叢集，才能使用它將您的模型定型。 若要在 HDInsight 叢集上建立 Spark，請參閱[在 HDInsight 中建立 Spark 叢集](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql)文件。
>
> 建立叢集時，必須指定 SSH 使用者名稱與密碼。 請記下這些值，因為使用 HDInsight 作為計算目標時會需要它們。
>
> 建立叢集之後，它會有完整網域名稱 (FQDN) `<clustername>.azurehdinsight.net`，其中 `<clustername>` 是您提供的叢集名稱。 您需要此位址 (或叢集的公用 IP 位址) 才能使用它作為計算目標

若要將 HDInsight 設定為計算目標，必須提供 HDInsight 叢集的完整網域名稱、叢集登入名稱與密碼。 下列範例會使用 SDK 將叢集附加到您的工作區。 在範例中，請以叢集的公用完整網域名稱或公用 IP 位址來取代 `<fqdn>`。 以叢集的 SSH 使用者名稱與密碼來取代 `<username>` 與 `<password>`：

> [!NOTE]
> 若要尋找您叢集的 FQDN，請瀏覽 Azure 入口網站並選取您的 HDInsight 叢集。 FQDN 位於 [概觀] 區段的 __URL__ 項目中。 請將 `https://` 從值的開頭移除。
>
> ![URL 項目已反白顯示之 HDInsight 叢集概觀的螢幕擷取畫面](./media/how-to-set-up-training-targets/hdinsight-overview.png)

```python
from azureml.core.compute import HDInsightCompute

try:
    # Attaches a HDInsight cluster as a compute target.
    HDInsightCompute.attach(ws,name = "myhdi",
                            address = "<fqdn>",
                            username = "<username>",
                            password = "<password>")
except UserErrorException as e:
    print("Caught = {}".format(e.message))
    print("Compute config already attached.")

# Configure HDInsight run
# load the runconfig object from the "myhdi.runconfig" file generated by the attach operaton above.
run_config = RunConfiguration.load(project_object = project, run_config_name = 'myhdi')

# ask system to prepare the conda environment automatically when used for the first time
run_config.auto_prepare_environment = True
```

## <a name="submit-training-run"></a>提交定型回合
    
無論計算目標為何，用於提交定型回合的程式碼都是相同的：

* 使用計算目標的回合組態建立 `ScriptRunConfig` 物件。
* 提交回合。
* 等待回合完成。

下列範例使用此文件中稍早建立的系統管理本機計算目標設定：

```pyghon
src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_config_system_managed)
run = exp.submit(src)
run.wait_for_completion(show_output = True)
```

如需使用 HDInsight 上的 Spark 示範定型的 Jupyter Notebook，請參閱 [https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/05.train-in-spark/05.train-in-spark.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/01.getting-started/05.train-in-spark/05.train-in-spark.ipynb)。

## <a name="view-and-set-up-compute-using-the-azure-portal"></a>使用 Azure 入口網站來檢視及設定計算

您可以從 Azure 入口網站檢視有哪些計算目標與您的工作區關聯。 若要取得清單，請使用下列步驟：

1. 瀏覽 [Azure 入口網站](https://portal.azure.com)，然後瀏覽到您的工作區。
2. 按一下 [應用程式] 區段底下的 [計算] 連結。

    ![[檢視計算] 索引標籤](./media/how-to-set-up-training-targets/compute_tab.png)

### <a name="create-a-compute-target"></a>建立計算目標

依照上述步驟檢視計算目標清單，然後使用下列步驟建立計算目標：

1. 按一下 __+__ 號以新增計算目標。

    ![新增計算 ](./media/how-to-set-up-training-targets/add_compute.png)

1. 輸入計算目標的名稱。
1. 選取要附加以進行__定型__的計算類型。 
1. 選取 [建立新項目] 並填寫必要表單。 
1. 選取 [建立] 
1. 您可以從清單選取計算目標以檢視狀態建立作業。

    ![檢視計算清單](./media/how-to-set-up-training-targets/View_list.png) 您接著將會看到該計算的詳細資料。
    ![檢視詳細資料](./media/how-to-set-up-training-targets/vm_view.PNG)
1. 現在您可以依照上述詳細說明，針對這些目標提交回合。

### <a name="reuse-existing-compute-in-your-workspace"></a>重複使用工作區中現有的計算

請依照上述步驟檢視計算目標清單，然後使用下列步驟重複使用計算目標：

1. 按一下 **+** 號以新增計算目標。
2. 輸入計算目標的名稱。
3. 選取要附加以進行定型的計算類型。 入口網站目前支援使用 Batch AI 與虛擬機器進行定型。
4. 選取 [使用現有項目]。
    - 附加 Batch AI 叢集時，請從下拉式清單選取計算目標，選取 Batch AI 工作區與 Batch AI 叢集，然後按一下 [建立]。
    - 附加虛擬機器時，請輸入 IP 位址、使用者名稱/密碼組合、私人/公開金鑰，以及連接埠，然後按一下 [建立]。

    > [!NOTE]
    > Microsoft 建議使用 SSH 金鑰，因為它們比密碼更安全。 密碼很容易遭受暴力密碼破解攻擊，而 SSH 金鑰則是仰賴密碼編譯簽章。 如需建立 SSH 金鑰以搭配 Azure 虛擬機器使用的資訊，請參閱下列文件：
    >
    > * [在 Linux 或 macOS 上建立及使用 SSH 金鑰]( https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)
    > * [在 Windows 上建立及使用 SSH 金鑰]( https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)

5. 您可以從計算清單選取計算目標，以檢視佈建狀態的狀態。
6. 現在您可以針對這些目標提交回合。

## <a name="examples"></a>範例
下列 Notebook 示範了此文章中說明的概念：
* `01.getting-started/02.train-on-local/02.train-on-local.ipynb`
* `01.getting-started/04.train-on-remote-vm/04.train-on-remote-vm.ipynb`
* `01.getting-started/03.train-on-aci/03.train-on-aci.ipynb`
* `01.getting-started/05.train-in-spark/05.train-in-spark.ipynb`
* `01.getting-started/07.hyperdrive-with-sklearn/07.hyperdrive-with-sklearn.ipynb`

取得這些 Notebook：[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟

* [Azure Machine Learning SDK 參照](http://aka.ms/aml-sdk)
* [教學課程：將模型定型](tutorial-train-models-with-aml.md)
* [部署模型的位置](how-to-deploy-and-where.md)
* [使用 Azure Machine Learning 服務建置機器學習服務管線](concept-ml-pipelines.md)
