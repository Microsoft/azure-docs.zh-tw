---
title: Azure Machine Learning 測試服務組態檔
description: 這份文件詳細說明 Azure ML 測試服務的組態集。
services: machine-learning
author: gokhanuluderya-msft
ms.author: gokhanu
manager: haining
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/28/2017
ROBOTS: NOINDEX
ms.openlocfilehash: abce80528479ba180783dbab604d4c836ddb7f1e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950772"
---
# <a name="azure-machine-learning-experimentation-service-configuration-files"></a>Azure Machine Learning 測試服務組態檔

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

當您在 Azure Machine Learning (Azure ML) Workbench 中執行指令碼時，執行的行為是由 **aml_config** 資料夾中的檔案所控制。 此資料夾位於您專案資料夾的根目錄下。 請務必了解這些檔案的內容，以便以最佳方式達到執行所需的結果。

以下是這個資料夾下的相關檔案：
- conda_dependencies.yml
- spark_dependencies.yml
- 計算目標檔案
    - \<計算目標名稱>.compute
- 回合組態檔
    - \<回合組態名稱>.runconfig

>[!NOTE]
>您通常會有計算目標檔案，並針對您所建立的每個計算目標回合組態檔。 不過，您可以獨立建立這些檔案，並讓多個回合組態檔指向相同的計算目標。

## <a name="condadependenciesyml"></a>conda_dependencies.yml
這個檔案是 [conda 環境檔案](https://conda.io/docs/using/envs.html#create-environment-file-by-hand)，會指定 Python 執行階段版本和您程式碼所依據的套件。 當 Azure ML Workbench 在 Docker 容器或 HDInsight 叢集中執行指令碼時，它會建立 [conda 環境](https://conda.io/docs/using/envs.html)以供您的指令碼執行。 

在此檔案中，您可以指定指令碼執行所需的 Python 套件。 Azure ML 測試服務會根據您的相依性清單建立 conda 環境。 執行引擎必須可透過下列通道觸達這裡所列的套件：

* [continuum.io](https://anaconda.org/conda-forge/repo)
* [PyPI](https://pypi.python.org/pypi)
* 可公開存取的端點 (URL)
* 或本機檔案路徑
* 執行引擎可觸達的其他項目

>[!NOTE]
>在 HDInsight 叢集上執行時，Azure ML Workbench 會針對您特定的執行建立 conda 環境。 這可讓不同的使用者在相同個叢集上的不同 Python 環境中執行。  

以下範例是典型的 **conda_dependencies.yml** 檔案。
```yaml
name: project_environment
dependencies:
  # Python version
  - python=3.5.2
  
  # some conda packages
  - scikit-learn
  - cryptography
  
  # use pip to install some more packages
  - pip:
     # a package in PyPi
     - azure-storage
     
     # a package hosted in a public URL endpoint
     - https://cntk.ai/PythonWheel/CPU-Only/cntk-2.1-cp35-cp35m-win_amd64.whl
     
     # a wheel file available locally on disk (this only works if you are executing against local Docker target)
     - C:\temp\my_private_python_pkg.whl
```

Azure ML Workbench 不需重建即可使用相同的 conda 環境，只要 **conda_dependencies.yml** 保持相同。 如果您的相依性改變，它將會重建您的環境。

>[!NOTE]
>如果您針對_本機_計算內容執行目標，就**不會**使用 **conda_dependencies.yml** 檔案。 必須手動安裝您本機 Azure ML Workbench Python 環境的套件相依性。

## <a name="sparkdependenciesyml"></a>spark_dependencies.yml
當您提交必須安裝的 PySpark 指令碼和 Spark 套件時，這個檔案會指定 Spark 應用程式名稱。 您也可以指定公開 Maven 存放庫，以及可以在這些 Maven 存放庫中找到的 Spark 套件。

下列是一個範例：

```yaml
configuration:
  # Spark application name
  "spark.app.name": "ClassifyingIris"
  
repositories:
  # Maven repository hosted in Azure CDN
  - "https://mmlspark.azureedge.net/maven"
  
  # Maven repository hosted in spark-packages.org
  - "https://spark-packages.org/packages"
  
packages:
  # MMLSpark package hosted in the Azure CDN Maven
  - group: "com.microsoft.ml.spark"
    artifact: "mmlspark_2.11"
    version: "0.5"
    
  # spark-sklearn packaged hosted in the spark-packages.org Maven
  - group: "databricks"
    artifact: "spark-sklearn"
    version: "0.2.0"
```

>[!NOTE]
>諸如背景工作大小及核心等叢集微調參數都應該移入 spark_dependecies.yml 檔案中的「組態」一節 

>[!NOTE]
>如果您在 Python 環境中執行指令碼，就會忽略 spark_dependencies.yml 檔案。 僅在您針對 Spark (在 Docker 或 HDInsight 叢集上) 執行時才會使用。

## <a name="run-configuration"></a>回合組態
若要指定特定回合組態，您需要 .compute 檔案和 .runconfig 檔案。 這些通常是使用 CLI 命令所產生的。 您也可以複製現有檔案、加以重新命名並且編輯。

```azurecli
# create a compute target pointing to a VM via SSH
$ az ml computetarget attach remotedocker -n <compute target name> -a <IP address or FQDN of VM> -u <username> -w <password>

# create a compute context pointing to an HDI cluster head-node via SSH
$ az ml computetarget attach cluster -n <compute target name> -a <IP address or FQDN of HDI cluster> -u <username> -w <password> 
```

此命令會以指定的計算目標作為基礎建立一組檔案。 假設您將計算目標命名為 _foo_。 此命令會在您的 **aml_config** 資料夾中產生 _foo.compute_ 和 _foo.runconfig_。

>[!NOTE]
> 回合組態檔的_本機_或 _docker_ 是任意名稱。 當您為方便起見而建立空白專案時，Azure ML Workbench 會新增這兩個回合組態。 您可以將專案範本隨附的 "<run configuration name>.runconfig" 檔案重新命名，或使用任何您需要的名稱建立新的檔案。

### <a name="compute-target-namecompute"></a>\<計算目標名稱>.compute
_\<compute target name>.compute_ 檔案會指定用於計算目標的連線和組態資訊。 它是成對的名稱和數值清單。 以下是支援的設定：

**類型**：計算環境的類型。 支援的值包括：
  - local
  - remote
  - docker
  - remotedocker
  - 叢集

**baseDockerImage**：用來執行 Python/PySpark 指令碼的 Docker 映像。 預設值是 _microsoft/mmlspark:plus-0.7.91_。 我們也支援一個其他映像：_microsoft/mmlspark:plus-gpu-0.7.91_，其可讓您以 GPU 存取主機電腦 (如果 GPU 存在)。

**位址**：虛擬機器的 IP 位址或 FQDN (完整的網域名稱) 或 HDInsight 叢集前端節點。

**使用者名稱**：用來存取虛擬機器或 HDInsight 前端節點的 SSH 使用者名稱。

**密碼**：SSH 連線的加密密碼。

**sharedVolumes**：表示執行引擎應該使用 Docker 共用磁碟區功能來回傳遞專案檔案的旗標。 開啟這個旗標可以加速執行，因為 Docker 可以直接存取專案，而不需要將它們進行複製。 如果 Docker 引擎是在 Windows 上執行，建議您最好設定 _false_，因為 Windows 上的 Docker 磁碟區共用可能很脆弱。 如果是在 macOS 或 Linux 上執行，請將它設定為 _true_。

**nvidiaDocker**：相對於一般 _docker_ 命令是啟動 Docker 映像，當此旗標設定為 _true_ 時，會告訴 Azure ML 測試服務使用 _nvidia docker_ 命令。 _Nvidia docker_ 引擎可讓 Docker 容器存取 GPU 硬體。 如果您需要在 Docker 容器中執行 GPU 執行，就必須進行設定。 僅 Linux 主機支援 _nvidia-docker_。 例如，Azure 中以 Linux 為基礎的 DSVM 會隨附 _nvidia-docker_。 目前在 Windows 上不支援 _nvidia-docker_。

**nativeSharedDirectory**：此屬性會指定基礎目錄 (例如：_~/.azureml/share/_)，可供檔案儲存，以在相同計算目標的執行上進行共用。 如果在 Docker 容器上執行時使用這個設定，_sharedVolumes_ 必須設定為 true。 否則，執行就會失敗。

**userManagedEnvironment**：此屬性會指定此計算目標是由使用者直接管理或是透過實驗服務管理。  

**pythonLocation**：這個屬性會指定要在計算目標上用來執行使用者程式的 python 執行階段位置。 

### <a name="run-configuration-namerunconfig"></a>\<回合組態名稱>.runconfig
_\<run configuration name>.runconfig_ 會指定 Azure ML 測試執行行為。 您可以設定諸如追蹤執行歷程記錄等執行行為，或是要與其他許多人共同使用的計算目標。 回合組態檔的名稱可用來填入 Azure ML Workbench 桌面應用程式中的執行內容下拉式清單。

**ArgumentVector**：這個區段會指定要包含在此執行及指令碼參數的一部分執行的指令碼。 例如，如果您的 "<run configuration name>.runconfig" 檔案中有下列程式碼片段 

```
 "ArgumentVector":[
  - "myscript.py"
  - 234
  - "-v" 
 ] 
```
_"az ml experiment submit foo.runconfig"_ 會使用傳入 234 作為參數的 _myscript.py_ 檔案自動執行命令，並且設定 --verbose 旗標。

**目標**：這個參數是 _/runconfig_ 檔案參考的 _.compute_ 檔案名稱。 通常會指向 _foo.compute_ 檔案，但是您可以編輯，以指向不同的計算目標。

**環境變數**：這個區段會讓使用者將環境變數設定為其執行的一部分。 使用者可以使用成對的名稱和數值來指定環境變數，格式如下：
```
EnvironmentVariables:
  "EXAMPLE_ENV_VAR1": "Example Value1"
  "EXAMPLE_ENV_VAR2": "Example Value2"
```

可以在使用者的程式碼中存取這些環境變數。 例如，此 Phyton 程式碼會列印名為 "EXAMPLE_ENV_VAR" 的環境變數
```
print(os.environ.get("EXAMPLE_ENV_VAR1"))
```

**Framework**：這個屬性會指定 Azure ML Workbench 是否應該啟動 Spark 工作階段來執行指令碼。 預設值為 _PySpark_。 如果您不是執行 PySpark 程式碼，將它設定為 _Python_ 可協助您更快啟動作業且負荷較低。

**CondaDependenciesFile**：這個屬性會指向 aml_config 資料夾中指定 conda 環境相依性的檔案。 如果設為 _null_，它就會指向預設 **conda_dependencies.yml** 檔案。

**SparkDependenciesFile**：這個屬性會指向 aml_config 資料夾中指定 Spark 相依性的檔案。 依預設會設定為 _null_，且會指向預設的 **spark_dependencies.yml** 檔案。

**PrepareEnvironment**：當這個屬性設定為 _true_ 時，會告訴測試服務以您初始回合中指定的 conda 相依性作為基礎，將 conda 環境準備好。 只有在針對 Docker 環境執行時，這個屬性才會生效。 如果您是針對_本機_環境執行，此設定就不會生效。 

**TrackedRun**：此旗標會表示測試服務，無論是否要追蹤 Azure ML Workbench 執行歷程記錄基礎結構中的回合。 預設值為 _true_。 

**UseSampling**：_UseSampling_ 會指定資料來源的作用中範例資料集是否用於回合。 如果設定為 _false_，就會內嵌資料來源，並使用讀取自資料存放區的完整資料。 如果設定為 _true_，就會使用作用中的範例。 使用者可以使用 **DataSourceSettings** 來指定他們需要覆寫作用中範例時，所要使用的是哪些特定範例資料集。 

**DataSourceSettings**：這個組態區段會指定資料來源設定。 在本節中，使用者會指定要作為執行一部分的特定資料來源的現有資料範例。 

下列組態設定會指定名為 "MySample" 的範例要用於名為 "MyDataSource" 的資料來源
```
DataSourceSettings:
    MyDataSource.dsource:
    Sampling:
    Sample: MySample
```

**DataSourceSubstitutions**：當使用者需要在不變更其程式碼的情況下，從一個資料來源切換至另一個資料來源時，可以使用資料來源的替代項目。 例如，使用者可以從取樣的清單、本機檔案切換至 Azure Blob 中所儲存的原始、較大資料集，方法是變更資料來源參考。 使用替代時，Azure ML Workbench 會執行您的資料來源和資料準備套件，方法是參考替代資料來源。

下列範例會將 Azure ML 資料來源和資料準備套件中的 "mylocal.datasource" 參考取代為 "myremote.dsource"。 
 
```
DataSourceSubstitutions:
    mylocal.dsource: myremote.dsource
```

以上述替代作為基礎，下列程式碼範例現在會讀取自 "myremote.dsource" 而不是 "mylocal.dsource"，而無須使用者變更其程式碼。
```
df = datasource.load_datasource('mylocal.dsource')
```
## <a name="next-steps"></a>後續步驟
深入了解[測試服務組態](experimentation-service-configuration.md)。
