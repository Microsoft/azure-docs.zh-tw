---
title: 設定 Azure Machine Learning 測試服務 | Microsoft Docs
description: 本文提供有關 Azure Machine Learning 測試服務的高階概觀，以及如何設定的指示。
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
ms.openlocfilehash: 3c5084e548bbb72fa38aae8b60aa46fb4d462dca
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990343"
---
# <a name="configuring-azure-machine-learning-experimentation-service"></a>設定 Azure Machine Learning 測試服務

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

## <a name="overview"></a>概觀
Azure Machine Learning 測試服務可讓資料科學家使用 Azure Machine Learning 執行和執行管理功能來執行其測試。 它提供的架構可透過快速反覆項目進行敏捷式實驗。 Azure Machine Learning Workbench 可讓您開始在電腦上使用本機執行，以及提供相應增加和相應放大到其他環境 (例如具有 GPU 的遠端資料科學 VM 或執行 Spark 的 HDInsight 叢集) 的簡單路徑。

測試服務的建置目的是提供隔離、重現問題，以及一致執行您的測試。 它可協助您管理計算目標、執行環境和回合組態。 您可以使用 Azure Machine Learning Workbench 執行和執行管理功能，輕鬆地在不同環境之間移動。 

您可以在 Workbench 專案中以本機方式，或在雲端中大量執行 Python 或 PySpark 指令碼。 

您可以在以下位置執行指令碼： 

* Workbench 安裝在本機電腦上的 Python (3.5.2) 環境
* 本機電腦上 Docker 容器內部的 Conda Python 環境
* 遠端 Linux 機器上由您擁有及管理的 Python 環境
* 遠端 Linux 電腦上 Docker 容器內部的 Conda Python 環境。 例如，[Azure 上以 Ubuntu 為基礎的 DSVM] (https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu)
* Azure 上的 [HDInsight for Spark](https://azure.microsoft.com/services/hdinsight/apache-spark/)

>[!IMPORTANT]
>Azure Machine Learning 測試服務目前分別支援 Python 3.5.2 和 Spark 2.1.11 作為 Python 和 Spark 執行階段版本。 


### <a name="key-concepts-in-experimentation-service"></a>測試服務中的重要概念
請務必在 Azure Machine Learning 測試執行中了解下列概念。 在後續章節中，我們會詳細討論如何使用這些概念。 

#### <a name="compute-target"></a>計算目標
_計算目標_會指定要執行程式的位置，例如您的桌面、VM 上的遠端 Docker 或叢集。 計算目標必須可供您定址且可存取。 Workbench 讓您能夠使用 Workbench 應用程式和 CLI 來建立計算目標並加以管理。 

CLI 中的 _az ml computetarget attach_ 命令可讓您建立能用於執行中的計算目標。

支援的計算節點包括：
* Workbench 安裝在電腦上的本機 Python (3.5.2) 環境。
* 電腦上的本機 Docker
* 遠端 Linux-Ubuntu VM 上由使用者管理的 Python 環境。 例如，[Azure 上以 Ubuntu 作為基礎的 DSVM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu)
* Linux Ubuntu VM 上的遠端 Docker。 例如，[Azure 上以 Ubuntu 作為基礎的 DSVM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu)
* Azure 上的 [HDInsight for Spark 叢集](https://azure.microsoft.com/services/hdinsight/apache-spark/)

測試服務目前分別支援 Python 3.5.2 和 Spark 2.1.11 作為 Python 和 Spark 執行階段版本。 

>[!IMPORTANT]
> **不**支援執行 Docker 的 Windows VM 作為遠端計算目標。

#### <a name="execution-environment"></a>執行環境
_執行環境_會定義執行時間設定和 Workbench 中執行程式所需的相依性。

如果是在 Workbench 預設的執行階段上執行，您就會使用最愛的工具和套件管理員來管理本機執行環境。 

Conda 是用來管理本機 Docker 和遠端 Docker 執行和以 HDInsigh 作為基礎的執行。 針對這些計算目標，執行環境設定是透過 **Conda_dependencies.yml** 和 **Spark_dependencies.yml files** 來管理。 這些檔案是位於專案內的 **aml_config** 資料夾。

**執行環境的支援執行階段包括：**
* Python 3.5.2
* Spark 2.1.11

### <a name="run-configuration"></a>回合組態
除了計算目標和執行環境，Azure Machine Learning 還會提供一個架構，可定義及變更回合組態。 每次執行實驗可能需要不同組態作為反覆試驗的一部分。 您可能要清除不同的參數範圍、使用不同的資料來源及調整 Spark 參數。 測試服務所提供的架構可用來管理回合組態。

執行 _az ml computetarget attach_ 命令會在您專案中的 **aml_config** 資料夾產生兩個檔案：_<your_computetarget_name>.compute_ 及 _<your_computetarget_name>.runconfig_。 當您建立計算目標時，為了方便起見，會自動建立 .runconfig 檔案。 您可以使用 CLI 中的 _az ml runconfigurations_ 命令來建立和管理其他回合組態。 您也可以在檔案系統上加以建立及編輯。

Workbench 中的回合組態也可讓您指定環境變數。 您可以指定環境變數，並在程式碼中加以使用，方法是在 .runconfig 檔案中新增下一節。 

```
EnvironmentVariables:
    "EXAMPLE_ENV_VAR1": "Example Value1"
    "EXAMPLE_ENV_VAR2": "Example Value2"
```

可以在您的程式碼中存取這些環境變數。 例如，此 Phyton 程式碼片段會列印名為 "EXAMPLE_ENV_VAR1" 的環境變數
```
print(os.environ.get("EXAMPLE_ENV_VAR1"))
```

_**下圖顯示初始實驗執行的高階流程。**_
![](media/experimentation-service-configuration/experiment-execution-flow.png)

## <a name="experiment-execution-scenarios"></a>測試執行案例
在本節中，我們要深入探討執行情節，並深入了解 Azure Machine Learning 會如何在遠端 VM 及在 HDInsight 叢集上執行測試，特別是在本機執行測試。 本節是從建立計算目標開始到執行您實驗的逐步解說。

>[!NOTE]
>本文章的其餘部分中，我們要使用 CLI (命令列介面) 命令來顯示概念和功能。 也可從 Workbench 使用此處所述的功能。

## <a name="launching-the-cli"></a>啟動 CLI
啟動 CLI 的簡單方法是在 Workbench 中開啟專案，並導覽至 [檔案] --> [開啟命令提示字元]。

![](media/experimentation-service-configuration/opening-cli.png)

這個命令會啟動終端機視窗，您可以在其中輸入命令，以在目前的專案資料夾中執行指令碼。 此終端機視窗是使用 Workbench 所安裝的 Python 3.5.2 環境設定。

>[!NOTE]
> 當您從命令視窗中執行任何 _az ml_ 命令時，需要針對 Azure 進行驗證。 CLI 會使用獨立的驗證快取，然後是桌面應用程式，因此登入 Workbench 並不表示已在 CLI 環境中進行驗證。 若要驗證，請使用下列步驟。 驗證權杖會在本機快取一段時間，讓您只在權杖過期時，才需要重複這些步驟。 當權杖到期或您看到驗證錯誤時，請執行下列命令：

```
# to authenticate 
$ az login

# to list subscriptions
$ az account list -o table

# to set current subscription to a particular subscription ID 
$ az account set -s <subscription_id>

# to verify your current Azure subscription
$ az account show
```

>[!NOTE] 
>當您在專案資料夾內執行 _az ml_ 命令時，請確認專案屬於_目前_ Azure 訂用帳戶上的 Azure Machine Learning 測試帳戶。 否則可能會發生執行錯誤。


## <a name="running-scripts-and-experiments"></a>執行指令碼和實驗
您可以透過 Workbench，使用 _az ml experiment submit_ 命令在各種計算目標上執行您的 Python 和 PySpark 指令碼。 此命令需要回合組態定義。 

當您建立計算目標時，Workbench 會建立對應的 runconfig 檔案，但您可以使用 _az ml runconfiguration  create_ 命令來建立其他的回合組態。 您也可以手動編輯回合組態檔。

回合組態會在 Workbench 中顯示作為測試執行體驗的一部分。 

>[!NOTE]
>您可以在[實驗執行組態參考](experimentation-service-configuration-reference.md)一節中進一步了解回合組態檔。

## <a name="running-a-script-locally-on-workbench-installed-runtime"></a>在安裝 Workbench 的執行階段上本機執行指令碼
Workbench 可讓您對安裝 Workbench 的 Python 3.5.2 執行階段直接執行指令碼。 會在 Workbench 安裝時間安裝這個預設的執行階段，並包含 Azure Machine Learning 程式庫和相依性。 本機執行的執行結果與構件仍會儲存在雲端中的執行歷程記錄服務。

不同於以 Docker 作為基礎的執行，此組態_不_是由 Conda 所管理。 必須手動佈建您本機 Workbench Python 環境的套件相依性。

您可以執行下列命令，在安裝 Workbench 的 Python 環境中本機執行您的指令碼。 

```
$az ml experiment submit -c local myscript.py
```

您可以在 Workbench CLI 視窗中輸入下列命令，找到預設 Python 環境的路徑。
```
$ conda env list
```

>[!NOTE]
>目前**不**支援直接針對本機 Spark 環境執行 PySpark。 Workbench 支援在本機 Docker 上執行的 PySpark 指令碼。 Azure Machine Learning 基礎 Docker 映像隨附於預先安裝的 Spark 2.1.11。 

_**Python 指令碼的本機執行概觀：**_
![](media/experimentation-service-configuration/local-native-run.png)

## <a name="running-a-script-on-local-docker"></a>在本機 Docker 上執行指令碼
您也可以透過測試服務，在本機電腦上的 Docker 容器執行您的專案。 Workbench 提供的基礎 Docker 映像隨附於 Azure Machine Learning 程式庫以及 Spark 2.1.11 執行階段，以方便本機 Spark 執行。 Docker 必須已在本機電腦上執行。

如需在本機 Docker 上執行 Python 或 PySpark 指令碼，您可以在 CLI 中執行下列命令。

```
$az ml experiment submit -c docker myscript.py
```
或
```
az ml experiment submit --run-configuration docker myscript.py
```

已使用 Azure Machine Learning 基礎 Docker 映像準備好本機 Docker 上的執行環境。 第一次執行並利用您的 conda_dependencies.yml 檔案中指定的套件重疊時，Workbench 會下載此映像。 這項作業會使初始執行較慢，但是受惠於 Workbench 重複使用快取的層，後續執行速度會大幅提升。 

>[!IMPORTANT]
>您必須先執行 _az ml experiment prepare -c docker_ 命令來準備您第一次執行所需的 Docker 映像。 您也可以在 docker.runconfig 檔案中將 **PrepareEnvironment** 參數設定為 true。 此動作會自動準備您的環境作為您執行執行的一部分。  

>[!NOTE]
>如果是在 Spark 上執行 PySpark 指令碼，除了 conda_dependencies.yml 之外也會使用 spark_dependencies.yml。

在 Docker 映像上執行指令碼會提供下列優點：

1. 確保您的指令碼能夠可靠地在其他執行環境中執行。 在 Docker 容器上執行可協助您找出並避免可能會影響可攜性的任何本機參考。 

2. 它可讓您快速測試執行階段和架構上安裝和設定複雜的程式碼，例如 Apache Spark，而不必自行安裝。


_**Python 指令碼的本機 Docker 執行概觀：**_
![](media/experimentation-service-configuration/local-docker-run.png)

## <a name="running-a-script-on-a-remote-docker"></a>在遠端 Docker 上執行指令碼
在某些情況下，您的本機電腦上可用的資源可能不夠定型所需的模型。 在此情況下，測試服務可提供輕鬆的方式，使用遠端 Docker 執行在更強大的 VM 上執行 Python 或 PySpark 指令碼。 

遠端 VM 應滿足下列需求：
* 遠端 VM 必須執行 Linux-Ubuntu，且應該可透過 SSH 存取。 
* 遠端 VM 必須執行 Docker。

>[!IMPORTANT]
> **不**支援執行 Docker 的 Windows VM 作為遠端計算目標


您可以使用下列命令來建立計算目標定義和以 Docker 作為基礎的遠端執行之回合組態。

```
az ml computetarget attach remotedocker --name "remotevm" --address "remotevm_IP_address" --username "sshuser" --password "sshpassword" 
```

一旦您設定計算目標後，可以使用下列命令來執行指令碼。
```
$ az ml experiment submit -c remotevm myscript.py
```
>[!NOTE]
>請記住，已使用 conda_dependencies.yml 中的規格設定執行環境。 如果在 .runconfig 檔案中指定 PySpark 架構，也會使用 spark_dependencies.yml。 

遠端 VM 的 Docker 建構程序與本機 Docker 執行的程序完全相同，因此您會獲得類似的執行體驗。

>[!TIP]
>如果您要避免第一次執行建置 Docker 映像時所造成的延遲，在執行您的指令碼之前，可以使用下列命令準備計算目標。 az ml experiment prepare -c remotedocker

_**Python 指令碼的遠端 VM 執行概觀：**_
![](media/experimentation-service-configuration/remote-vm-run.png)

## <a name="running-a-script-on-a-remote-vm-targeting-user-managed-environments"></a>在由遠端 VM 鎖定且由使用者管理的環境上執行指令碼
測試服務也支援在使用者自己的 Python 環境 (在遠端 Ubuntu 虛擬機器內) 上執行指令碼。 這可讓您管理自己的執行環境，並且仍然可以使用 Azure Machine Learning 的功能。 

請遵循下列步驟，在您自己的環境中執行指令碼。
* 透過安裝相依性來準備遠端 Ubuntu VM 或 DSVM 上的 Python 環境。
* 使用下列命令來安裝 Azure Machine Learning 需求。

```
pip install -I --index-url https://azuremldownloads.azureedge.net/python-repository/preview --extra-index-url https://pypi.python.org/simple azureml-requirements
```

>[!TIP]
>在某些情況下，您可能需要在 sudo 模式下執行此命令 (視您的權限而定)。 
```
sudo pip install -I --index-url https://azuremldownloads.azureedge.net/python-repository/preview --extra-index-url https://pypi.python.org/simple azureml-requirements
```
 
* 針對遠端 VM 執行上由使用者管理的回合，使用下列命令來建立計算目標定義和回合組態。
```
az ml computetarget attach remote --name "remotevm" --address "remotevm_IP_address" --username "sshuser" --password "sshpassword" 
```
>[!NOTE]
>這會將 .compute 組態檔中的 "userManagedEnvironment" 參數設定為 true。

* 在 .Compute 檔案中設定 Python 執行階段可執行檔的位置。 您應該參考 Python 可執行檔的完整路徑。 
```
pythonLocation: python3
```

一旦您設定計算目標後，可以使用下列命令來執行指令碼。
```
$ az ml experiment submit -c remotevm myscript.py
```

>[!NOTE]
> 當您在 DSVM 上執行時，您應該使用下列命令

如果您想直接在 DSVM 的全域 Python 環境上執行，請執行此命令。
```
sudo /anaconda/envs/py35/bin/pip install <package>
```


## <a name="running-a-script-on-an-hdinsight-cluster"></a>在 HDInsight 叢集上執行指令碼
HDInsight 是支援 Apache Spark 的巨量資料分析常用平台。 Workbench 可讓您使用 HDInsight Spark 叢集測試巨量資料。 

>[!NOTE]
>HDInsight 叢集必須使用 Azure Blob 作為主要儲存體。 尚未支援使用 Azure Data Lake 儲存體。

您可以使用下列命令來建立計算目標，以及 HDInsight Spark 叢集的回合組態：

```
$ az ml computetarget attach cluster --name "myhdi" --address "<FQDN or IP address>" --username "sshuser" --password "sshpassword"  
```

>[!NOTE]
>如果您是使用 FQDN 而不是 IP 位址，且 HDI Spark 叢集名為 _foo_，SSH 端點就會位於名為 _foo-ssh.azurehdinsight.net_ 的驅動程式節點。 使用 _--address_ 參數的 FQDN 時，別忘了伺服器名稱中的 **-ssh** 後置詞。


擁有計算內容之後，您可以執行下列命令以執行 PySpark 指令碼。

```
$ az ml experiment submit -c myhdi myscript.py
```

Workbench 會使用 Conda 來準備及管理 HDInsight 叢集上的執行環境。 組態是由 _conda_dependencies.yml_ 和 _spark_dependencies.yml_ 組態檔所管理。 

您需要 HDInsight 叢集的 SSH 存取權，才能在此模式中執行測試。 

>[!NOTE]
>支援的組態是執行 Linux 的 HDInsight Spark 叢集 (包含 Python/PySpark 3.5.2 和 Spark 2.1.11 的 Ubuntu)。

_**以 HDInsight 為基礎的 PySpark 指令碼執行概觀**_
![](media/experimentation-service-configuration/hdinsight-run.png)


## <a name="running-a-script-on-gpu"></a>在 GPU 上執行指令碼
若要在 GPU 上執行指令碼，您可以遵循本文章中的指導方針：[如何在 Azure Machine Learning 中使用 GPU](how-to-use-gpu.md)

## <a name="using-ssh-key-based-authentication-for-creating-and-using-compute-targets"></a>使用以 SSH 金鑰為基礎的驗證來建立及使用計算目標
除了以使用者名稱/密碼為基礎的配置之外，Azure Machine Learning Workbench 還可讓您使用以 SSH 金鑰為基礎的驗證，以建立及使用計算目標。 使用 remotedocker 或叢集作為計算目標時，就可以使用這項功能。 使用此配置時，Workbench 會建立一個公開/私密金鑰組，並回報公開金鑰。 您可以針對使用者名稱將公開金鑰附加至 ~/.ssh/authorized_keys 檔案。 然後 Azure Machine Learning Workbench 就會使用以 SSH 金鑰為基礎的驗證，在這個計算目標上進行存取與執行作業。 由於計算目標的私密金鑰是儲存在工作區的金鑰存放區中，因此工作區的其他使用者也能以同樣方式使用計算目標，方法是提供針對建立計算目標所提供的使用者名稱。  

您可以遵循這些步驟來使用這項功能。 

- 使用下列其中一個命令建立計算目標。

```
az ml computetarget attach remotedocker --name "remotevm" --address "remotevm_IP_address" --username "sshuser" --use-azureml-ssh-key
```
或
```
az ml computetarget attach remotedocker --name "remotevm" --address "remotevm_IP_address" --username "sshuser" -k
```
- 將 Workbench 產生的公開金鑰附加到已附加之計算目標上的 ~/.ssh/authorized_keys 檔案。 

>[!IMPORTANT]
>您必須使用用來建立計算目標的相同使用者名稱登入計算目標。 

- 您現再已可以使用以 SSH 金鑰為基礎的驗證來準備及使用計算目標。

```
az ml experiment prepare -c remotevm
```

## <a name="next-steps"></a>後續步驟
* [建立與安裝 Azure Machine Learning](quickstart-installation.md)
* [模型管理](model-management-overview.md)
