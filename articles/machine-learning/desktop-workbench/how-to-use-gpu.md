---
title: 如何針對 Azure Machine Learning 使用 GPU | Microsoft Docs
description: 本文說明如何使用圖形處理器 (GPU) 在 Azure Machine Learning Workbench 中針對深度類神經網路進行定型處理。
services: machine-learning
author: rastala
ms.author: roastala
manager: jhubbard
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 8e8944a9bdfa2b75f6442e03d45dfb6977c2a122
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974352"
---
# <a name="how-to-use-gpu-in-azure-machine-learning"></a>如何在 Azure Machine Learning 中使用 GPU

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

圖形處理器 (GPU) 廣泛用於處理需進行大量運算的工作，因此適用於針對特定深度類神經網路的定型處理工作。 透過使用 GPU，您將能大幅降低模型的定型時間。 在本文件中，您將了解如何設定 Azure ML Workbench，以使用配備 GPU 的 [DSVM (資料科學虛擬機器)](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) \(英文\) 作為執行目標。 

## <a name="prerequisites"></a>必要條件
- 若要逐步執行本作法指南，您需要先[安裝 Azure ML Workbench](quickstart-installation.md)。
- 您需要能夠存取配備 NVidia GPU 的電腦。
    - 您可以直接在具有 GPU 的本機電腦 (Windows 或 macOS) 上執行指令碼。
    - 您也可以在具 GPU 的 Linux 電腦上的 Docker 容器中執行指令碼。

## <a name="execute-in-local-environment-with-gpus"></a>在具有 GPU 的「本機」環境中執行
您可以在配備 GPU 的電腦上安裝 Azure ML Workbench，並針對「本機」環境執行。 這可以是：
- 實體的 Windows 或 macOS 電腦。
- Windows VM (虛擬機器)，例如使用 Azure NC 系列之 VM 範本所佈建的 Windows DSVM。

在此情況下，Azure ML Workbench 中不需要有任何特殊設定。 您只需確定已於本機安裝所有必要的驅動程式、工具組，以及已啟用 GPU 的機器學習程式庫。 您只需針對 Python 執行階段可在其中直接存取本機 GPU 硬體的「本機」環境執行。

1. 開啟 AML Workbench。 移至 [檔案] 和 [開啟命令提示字元]。 
2. 從命令列安裝已啟用 GPU 的深度學習架構，例如，Microsoft 辨識工具組、TensorFlow 等。例如︰

```batch
REM install latest TensorFlow with GPU support
C:\MyProj> pip install tensorflow-gpu

REM install Microsoft Cognitive Toolkit 2.5 with GPU support on Windows
C:\MyProj> pip install https://cntk.ai/PythonWheel/GPU/cntk_gpu-2.5.1-cp35-cp35m-win_amd64.whl
```

3. 撰寫能利用深度學習程式庫的 Python 程式碼。
4. 選擇 [本機] 作為計算環境並執行 Python 程式碼。

```batch
REM execute Python script in local environment
C:\MyProj> az ml experiment submit -c local my-deep-learning-script.py
```

## <a name="execute-in-docker-environment-on-linux-vm-with-gpus"></a>在具備 GPU 之 Linux VM 上的 _Docker_ 環境中執行
Azure ML Workbench 也支援在 Azure Linux VM 的 Docker 中執行。 您可以藉此在功能強大的虛擬硬體中執行需進行大量運算的作業，並透過在完成後將它關閉來僅支付實際使用量的費用。 原則上，您應該能夠在任何 Linux 虛擬機器上使用 GPU，但 Ubuntu 架構的 DSVM 已預先安裝必要的 CUDA 驅動程式和程式庫，讓設定變得更為簡單。 請遵循下列步驟：

### <a name="create-a-ubuntu-based-linux-data-science-virtual-machine-in-azure"></a>在 Azure 中建立 Ubuntu 架構的 Linux 資料科學虛擬機器
1. 開啟網頁瀏覽器，然後移至 [Azure 入口網站](https://portal.azure.com)。

2. 選取入口網站左側的 [+ 新增]。

3. 在市集中搜尋「適用於 Linux (Ubuntu) 的資料科學虛擬機器」。

4. 按一下 [建立] 以建立 Ubuntu DSVM。

5. 在 [基本] 表單中填入必要資訊。
選取 VM 的位置時，請留意 GPU VM 僅於特定的 Azure 區域提供，例如「美國中南部」。 請參閱[依區域提供的產品](https://azure.microsoft.com/regions/services/)。
按一下 [確定] 以儲存 [基本] 資訊。

6. 選擇虛擬機器的大小。 選取一個具 NC 前置詞之 VM (代表其配備 NVidia GPU 晶片) 的大小。  視需要按一下 [全部檢視] 以查看完整清單。 深入了解[配備 GPU 的 Azure VM](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-gpu)。

7. 完成其餘的設定，然後檢閱購買資訊。 按一下 [購買] 以建立 VM。 記下配置給該虛擬機器的 IP 位址。 

### <a name="create-a-new-project-in-azure-ml-workbench"></a>在 Azure ML Workbench 中建立新專案 
您可以使用「使用 TensorFlow 對 MNIST 進行分類」範例，或「使用 CNTK 對 MNIST 資料集進行分類」範例。

### <a name="create-a-new-compute-target"></a>建立新的計算目標
從 Azure ML Workbench 啟動命令列。 輸入下列命令。 使用您自己的名稱、IP 位址、使用者名稱及密碼值，來取代下列範例中的預留位置文字。 

```batch
C:\MyProj> az ml computetarget attach remotedocker --name "my_dsvm" --address "my_dsvm_ip_address" --username "my_name" --password "my_password" 
```

### <a name="configure-azure-ml-workbench-to-access-gpu"></a>設定 Azure ML Workbench 以存取 GPU
返回專案並開啟 [檔案檢視]，然後點擊 [重新整理] 按鈕。 現在您會看到兩個新的設定檔：`my_dsvm.compute` 和 `my_dsvm.runconfig`。
 
開啟 `my_dsvm.compute`。 將 `baseDockerImage` 變更為 `microsoft/mmlspark:plus-gpu-0.9.9`，並加入新的一行 `nvidiaDocker: true`。 如此一來，檔案應含有下列這兩行：
 
```yaml
...
baseDockerImage: microsoft/mmlspark:plus-gpu-0.9.9
nvidiaDocker: true
```
 
現在開啟 `my_dsvm.runconfig`，將 `Framework` 值從 `PySpark` 變更為 `Python`。 如果您沒有看到這一行，請加入它，因為預設值會是 `PySpark`。

```yaml
"Framework": "Python"
```
### <a name="reference-deep-learning-framework"></a>參考深度學習架構 
開啟 `conda_dependencies.yml` 檔案，並確定您使用的是 GPU 版本的深度學習架構 Python 套件。 範例專案會列出 CPU 版本，因此您需要進行這項變更。

適用於 TensorFlow 的範例： 
```
name: project_environment
dependencies:
  - python=3.5.2
  # latest TensorFlow library with GPU support
  - tensorflow-gpu
```

適用於 Microsoft 辨識工具組的範例：
```yaml
name: project_environment
dependencies:
  - python=3.5.2
  - pip: 
    # use the Linux build of Microsoft Cognitive Toolkit 2.5 with GPU support
    - https://cntk.ai/PythonWheel/GPU/cntk_gpu-2.5.1-cp35-cp35m-win_amd64.whl
```

### <a name="execute"></a>執行
您現在已可執行您的 Python 指令碼。 您可以在 Azure ML Workbench 內使用 `my_dsvm` 內容來執行它們，或是從命令列啟動它：
 
```batch
C:\myProj> az ml experiment submit -c my_dsvm my_tensorflow_or_cntk_script.py
```
 
若要確認已使用 GPU，請檢查執行輸出，查看是否有如下的內容：

```
name: Tesla K80
major: 3 minor: 7 memoryClockRate (GHz) 0.8235
pciBusID 06c3:00:00.0
Total memory: 11.17GiB
Free memory: 11.11GiB
```

恭喜！ 您的指令碼已透過 Docker 容器運用了 GPU 的運算能力！

## <a name="next-steps"></a>後續步驟
請在 Azure ML 資源庫中參閱使用 GPU 加速深度類神經網路定型的範例。
