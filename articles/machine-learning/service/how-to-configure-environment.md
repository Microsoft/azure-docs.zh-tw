---
title: 設定 Azure Machine Learning 的開發環境 | Microsoft Docs
description: 了解如何在使用 Azure Machine Learning 服務時設定開發環境。 在本文件中，您將了解如何使用 Conda 環境、建立組態檔，以及設定 Jupyter Notebook、Azure Notebooks、IDE 和資料科學虛擬機器。
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.reviewer: larryfr
manager: cgronlun
ms.topic: conceptual
ms.date: 8/6/2018
ms.openlocfilehash: 675dae022376fc62292f3b079bd735939b9199c2
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220290"
---
# <a name="configure-a-development-environment-for-the-azure-machine-learning-service"></a>設定 Azure Machine Learning 服務的開發環境

了解如何設定開發環境以使用 Azure Machine Learning 服務。 您將了解如何建立組態檔，以便讓您的環境與 Azure Machine Learning 工作區產生關聯。 您也將了解如何設定下列開發環境：

* 您自己電腦上的 Jupyter Notebook
* Visual Studio Code
* 您偏好的程式碼編輯器

建議方法是使用 Continuum Anaconda [Conda 虛擬環境](https://conda.io/docs/user-guide/tasks/manage-environments.html)來隔離您的工作環境，以避免套件之間的相依性衝突。 本文將說明設定 Conda 環境的步驟，並用它來執行 Azure Machine Learning。


## <a name="prerequisites"></a>必要條件

* Azure Machine Learning 服務工作區。 若要建立工作區，請使用[開始使用 Azure Machine Learning 服務](quickstart-get-started.md)文件中的步驟。

* [Continuum Anaconda](https://www.anaconda.com/download/) 或 [Miniconda](https://conda.io/miniconda.html) 套件管理員。

 * [若使用 Visual Studio Code，請使用 Python 擴充功能](https://code.visualstudio.com/docs/python/python-tutorial)。

## <a name="create-workspace-configuration-file"></a>建立工作區組態檔

SDK 會使用工作區組態檔來和您的 Azure 機器學習服務工作區進行通訊。  有兩種方式可取得此檔案：

* 完成建立工作區和組態檔的[快速入門](quickstart-get-started.md)。 這會在 Azure Notebook 中為您建立 `config.json` 檔案。  此檔案包含您工作區的組態資訊。  將此檔案下載或複製到參考此工作區的指令碼或 Notebook 所在目錄。


* 使用下列步驟自行建立組態檔：

    1. 在 [Azure 入口網站](https://portal.azure.com)中開啟工作區。 複製__工作區名稱__、__資源群組__和__訂用帳戶識別碼__。 這些值會用來建立組態檔。

        ![Azure 入口網站](./media/how-to-configure-environment/configure.png) 
    
    1. 以此 Python 程式碼建立檔案。 在參考工作區的指令碼或 Notebook 所在目錄中，執行該程式碼：
        ```
        from azureml.core import Workspace

        subscription_id ='<subscription-id>'
        resource_group ='<resource-group>'
        workspace_name = '<workspace-name>'
        
        try:
           ws = Workspace(subscription_id = subscription_id, resource_group = resource_group, workspace_name = workspace_name)
           ws.write_config()
           print('Library configuration succeeded')
        except:
           print('Workspace not found')
        ```
        這會寫入下列 `aml_config/config.json` 檔案： 
    
        ```json
        {
        "subscription_id": "<subscription-id>",
        "resource_group": "<resource-group>",
        "workspace_name": "<workspace-name>"
        }
        ```
        您可以將 `aml_config` 目錄或 `config.json` 複製到任何其他參考工作區的目錄中。

>[!NOTE] 
>相同目錄或其下方目錄中的其他指令碼或 Notebook 會以 `ws=Workspace.from_config()` 來載入工作區

## <a name="azure-notebooks-and-data-science-virtual-machine"></a>Azure Notebooks 與資料科學虛擬機器

Azure Notebooks 和 Azure 資料科學虛擬機器 (DSVM) 已預先設定為可搭配使用 Azure Machine Learning 服務。 這些環境中已安裝必要的元件，例如 Azure Machine Learning SDK。

Azure Notebooks 是 Azure 雲端中的 Jupyter Notebook 服務。 資料科學虛擬機器是專為資料科學工作預先設定的 VM 映像。 VM 中包含熱門工具、IDE 及 Jupyter Notebook、PyCharm 和 Tensorflow 等套件。

但您仍然需要工作區組態檔來使用這些環境。

如需有關資料科學虛擬機器的詳細資訊，請參閱[資料科學虛擬機器](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)的文件。

如需搭配使用 Azure Notebooks 和 Azure Machine Learning 服務的範例，請參閱[開始使用 Azure Machine Learning 服務](quickstart-get-started.md)的文件。

## <a name="configure-jupyter-notebooks-on-your-own-computer"></a>在您自己的電腦上設定 Jupyter Notebook

1. 開啟命令提示字元或殼層。

2. 若要建立 Conda 環境，請使用下列命令：

    ```shell
    # create a new conda environment with Python 3.6, numpy and cython
    conda create -n myenv Python=3.6 cython numpy

    # activate the conda environment
    conda activate myenv

    # If you are running Mac OS you should run
    source activate myenv
    ```

    建立環境可能需要幾分鐘的時間，因為或許需要下載 Python 3.6 和其他元件。

3. 若要安裝 Azure Machine Learning SDK 和 Notebook 的額外項目，請使用下列命令：

     ```shell
    pip install --upgrade azureml-sdk[notebooks,automl]
    ```

    安裝 SDK 可能需要幾分鐘的時間。

4. 若要為您的機器學習實驗安裝套件，請使用下列命令，並將 `<new package>` 取代為您想要安裝的套件：

    ```shell
    conda install <new package>
    ```

5. 若要安裝可感知 Conda 的 Jupyter Notebook 伺服器，並啟用實驗 widget (用於檢視執行資訊)，請使用下列命令：

    ```shell
    # install Jupyter 
    conda install nb_conda

    # install experiment widget
    jupyter nbextension install --py --user azureml.train.widgets

    # enable experiment widget
    jupyter nbextension enable --py --user azureml.train.widgets
    ```

6. 若要啟動 Jupyter Notebook，請使用下列命令：

    ```shell
    jupyter notebook
    ```

7. 開啟新的 Notebook，然後選取 "myenv" 作為您的核心。 然後藉由在 Notebook 資料格中執行下列命令，確認您已安裝 Azure Machine Learning SDK：

    ```python
    import azureml.core
    azureml.core.VERSION
    ```

## <a name="configure-visual-studio-code"></a>設定 Visual Studio Code

1. 開啟命令提示字元或殼層。

2. 若要建立 Conda 環境，請使用下列命令：

    ```shell
    # create a new conda environment with Python 3.6, numpy and cython
    conda create -n myenv Python=3.6 cython numpy

    # activate the conda environment
    conda activate myenv

    # If you are running Mac OS you should run
    source activate myenv
    ```

2. 若要安裝 Azure Machine Learning SDK，請使用下列命令：
 
    ```shell
    pip install --upgrade azureml-sdk[automl]
    ```

4. 若要安裝適用於 AI 的 Visual Studio 程式碼工具，請參閱適用於 [AI 工具](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai)的 Visual Studio 市集項目。 

5. 若要為您的機器學習實驗安裝套件，請使用下列命令，並將 `<new package>` 取代為您想要安裝的套件：

    ```shell
    conda install <new package>
    ```

6. 啟動 Visual Studio Code，然後使用 __CTRL-SHIFT-P__ 來取得__命令選擇區__。 輸入「Python：選取解譯器」，然後選取您所建立的 Conda 環境。

    > [!NOTE]
    > Visual Studio Code 會自動感知您電腦上的 Conda 環境。 如需詳細資訊，請參閱 [Visual Studio 程式碼文件](https://code.visualstudio.com/docs/python/environments#_conda-environments)。

7. 若要驗證組態，請使用 Visual Studio Code 與下列程式碼來建立新的 Python 指令碼檔案，然後執行它：

    ```python
    import azureml.core
    azureml.core.VERSION
    ```

## <a name="configure-code-editor-of-your-choice"></a>依據您的偏好來設定程式碼編輯器

若要搭配使用自訂程式碼編輯器和 Azure 機器學習服務 SDK，請先建立 Conda 環境，如上面所述。 然後依照每個編輯器的指示來使用 Conda 環境。 例如，PyCharm 的指示位於 [https://www.jetbrains.com/help/pycharm/2018.2/conda-support-creating-conda-virtual-environment.html](https://www.jetbrains.com/help/pycharm/2018.2/conda-support-creating-conda-virtual-environment.html)。
 
## <a name="next-steps"></a>後續步驟

* [使用 MNIST 資料集訓練 Azure Machine Learning 上的模型](tutorial-train-models-with-aml.md)
