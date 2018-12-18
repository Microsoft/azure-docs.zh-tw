---
title: 您可以在 Azure 的 Data Science Virtual Machine 上做的十件事 | Microsoft Docs
description: 在 Data science Virtual Machine 上執行各種資料探索和模型分析工作。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
editor: cgronlun
ms.assetid: 145dfe3e-2bd2-478f-9b6e-99d97d789c62
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: gokuma
ms.openlocfilehash: faabdb4c2d2e434863a6bed0b2cd85a05c94eab1
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395724"
---
# <a name="ten-things-you-can-do-on-the-windows-data-science-virtual-machine"></a>您可以在 Windows Data Science Virtual Machine 上做的十件事

Windows Data Science Virtual Machine (DSVM) 是強大的資料科學開發環境，可讓您執行各種資料探索和模型分析工作。 此環境已內建數個熱門的資料分析工具，以便您快速開始進行內部部署或混合式部署的分析。 DSVM 與多項 Azure 服務密切合作，並且能夠讀取及處理已儲存在 Azure SQL 資料倉儲、Azure Data Lake、Azure 儲存體或 Azure Cosmos DB 中的資料。 它也可以利用 Azure Machine Learning 和 Azure Data Factory 等其他分析工具。

在本文中，您將了解如何使用 DSVM 來執行各種資料科學工作，並與其他 Azure 服務互動。 以下是您可以在 DSVM 上做的一些事：

1. 使用 Microsoft ML Server 或 Python 來探索資料並在 DSVM 本機開發模型
2. 使用 Jupyter 筆記本，在採用 Python 2、Python 3、Microsoft R (針對效能設計的 R 企業就緒版) 的瀏覽器上試驗您的資料
3. 在 Azure Machine Learning 上部署使用 R 和 Python 建置的模型，以便用戶端應用程式使用簡單的 Web 服務介面來存取您的模型
4. 使用 Azure 入口網站或 Powershell 管理您的 Azure 資源
5. 建立 Azure 檔案儲存體作為 DSVM 上可掛接的磁碟機，以擴充您的儲存空間並與整個小組共用大型資料集/程式碼
6. 使用 GitHub 來與您的小組共用程式碼，並使用預先安裝的 Git 用戶端 (Git Bash、Git GUI) 來存取您的存放庫。
7. 存取各種 Azure 資料和分析服務，例如 Azure Blob 儲存體，Azure Data Lake、Azure HDInsight (Hadoop)、Azure Cosmos DB、Azure SQL 資料倉儲和資料庫
8. 使用在 DSVM 上預先安裝的 Power BI Desktop 來建立報告和儀表板並雲端上加以部署
9. 動態調整 DSVM 以符合您的專案需求
10. 在虛擬機器上安裝其他工具   

> [!NOTE]
> 額外的使用費適用於在本文中列出的許多其他資料儲存體和分析服務。 如需詳細資料，請參閱 [Azure 定價](https://azure.microsoft.com/pricing/)頁面。
> 
> 

**先決條件**

* 您需要 Azure 訂用帳戶。 您可以在[這裡](https://azure.microsoft.com/free/)註冊免費試用。
* 如需在 Azure 入口網站上佈建資料科學虛擬機器的指示，請參閱[建立虛擬機器](https://portal.azure.com/#create/microsoft-dsvm.dsvm-windowsserver-2016)。

## <a name="1-explore-data-and-develop-models-using-microsoft-ml-server-or-python"></a>1.使用 Microsoft ML Server 或 Python 來探索資料及開發模型
您可以使用 R 和 Python 等語言在 DSVM 上進行資料分析。

針對 R，您可以使用 RStudio 這類 IDE，在 [開始] 功能表、桌面或「Visual Studio R 工具」即可找到這類 IDE。 除了開放原始碼/CRAN-R 以外，Microsoft 還提供額外的程式庫以便進行可調整的分析，以及分析大於平行區塊分析所允許之記憶體大小的資料。 

針對 Python，您可以使用 Visual Studio Community Edition 這類已預先安裝「適用於 Visual Studio 的 Python 工具」(PTVS) 延伸模組的 IDE。 根據預設，在 PTVS 上只有設定 Python 3.6 (根 Conda 環境)。 若要啟用 Anaconda Python 2.7，您必須執行下列步驟：

* 為每個版本建立自訂環境，方法是瀏覽至 [工具] -> [Python 工具] -> [Python 環境]，然後按一下 Visual Studio Community Edition 中的 [+ 自訂]
* 提供描述並將環境前置詞路徑設定為 *c:\anaconda\envs\python2* (適用於 Anaconda Python 2.7)
* 按一下 [自動偵測]，然後按一下 [套用] 來儲存環境。

以下是自訂環境設定在 Visual Studio 中的外觀。

![PTVS 設定](./media/vm-do-ten-things/PTVSSetup.png)

如需有關如何建立 Python 環境的詳細資訊，請參閱 [PTVS 文件](http://aka.ms/ptvsdocs) 。

現在您將建立新的 Python 專案。 瀏覽至 [檔案] -> [新增] -> [專案] -> [Python]，然後選取您要建置的 Python 應用程式類型。 您能以滑鼠右鍵按一下 [Python 環境]、選取 [新增/移除 Python 環境]，然後選取所需的環境，以將目前專案的 Python 環境設定為所需的版本 (Python 2.7 或 3.6)。 您可以在產品[文件](http://aka.ms/ptvsdocs)中找到關於使用 PTVS 的詳細資訊。

## <a name="2-using-a-jupyter-notebook-to-explore-and-model-your-data-with-python-or-r"></a>2.使用 Jupyter Notebook 來探索您的資料，並以 Python 或 R 進行模型分析
Jupyter Notebook 是一個強大的環境，可提供以瀏覽器為基礎的 "IDE" 進行資料探索和模型分析。 您可以在 Jupyter Notebook 使用 Python 2、Python 3 或 R (開放原始碼和 Microsoft R Server)。

若要啟動 Jupyter 筆記本，請按一下標題為 [Jupyter 筆記本] 的 [開始] 功能表圖示 / 桌面圖示。 在 DSVM 命令提示字元上，您也可以從有現有筆記本或您想要建立新筆記本的目錄中，執行 ```jupyter notebook``` 命令。  

啟動 Jupyter 之後，您應該會看到一個目錄，當中包含幾個預先封裝至 DSVM 中的範例筆記本。 現在您可以：

* 按一下筆記本以查看程式碼。
* 按 **SHIFT-ENTER** 執行每個儲存格。
* 按一下 [儲存格] -> [執行] 以執行整個筆記本
* 按一下 Jupyter 圖示 (左上角)，接著按一下右邊的 [新增] 按鈕，然後選擇筆記本語言 (也稱為核心)，以建立新的筆記本。   

> [!NOTE]
> 目前支援 Python 2.7、Python 3.6、R、Julia 及 Jupyter 中的 PySpark 核心。 R 核心同時支援以開放原始碼 R 及高效能 Microsoft R 進行程式設計。   
> 
> 

一旦位於筆記本，您即可探索資料、建立模型，以及使用您所選的程式庫測試模型。

## <a name="3-build-models-using-r-or-python-and-operationalize-them-using-azure-machine-learning"></a>3.使用 R 或 Python 建置模型並且使用 Azure Machine Learning 實作
建置並驗證您的模型之後，下一個步驟通常是將它部署到生產環境中。 這可讓您的用戶端應用程式以即時方式或分批模式叫用模型預測。 Azure Machine Learning 提供一個機制來實作以 R 或 Python 建置的模型。

當您在 Azure Machine Learning 中實作模型時，系統會公開一項可讓用戶端進行 REST 呼叫的 Web 服務，以便傳入輸入參數以及接收來自模型的預測做為輸出。   

> [!NOTE]
> 如果您尚未註冊 Azure Machine Learning，只要瀏覽 [Azure Machine Learning Studio](https://studio.azureml.net/) 首頁並按一下 [開始使用]，即可取得免費的工作區或標準工作區。   
> 
> 

### <a name="build-and-operationalize-python-models"></a>建置和實作 Python 模型
以下是在 Python Jupyter Notebook 來開發的程式碼片段，它會使用 Scikit-learn 程式庫建置簡單的模型。

    #IRIS classification
    from sklearn import datasets
    from sklearn import svm
    clf = svm.SVC()
    iris = datasets.load_iris()
    X, y = iris.data, iris.target
    clf.fit(X, y)

用來部署 python 模型到 Azure Machine Learning 的方法，會將模型預測包裝成函式，然後以預先安裝之 Azure Machine Learning python 程式庫所提供的屬性裝飾它，代表您的 Azure Machine Learning 工作區識別碼、API 金鑰，以及輸入和傳回參數。  

    from azureml import services
    @services.publish(workspaceid, auth_token)
    @services.types(sep_l = float, sep_w = float, pet_l=float, pet_w=float)
    @services.returns(int) #0, or 1, or 2
    def predictIris(sep_l, sep_w, pet_l, pet_w):
     inputArray = [sep_l, sep_w, pet_l, pet_w]
    return clf.predict(inputArray)

用戶端現在可以呼叫 Web 服務。 有一個可建構 REST API 要求的便利包裝函式。 以下是用來取用 Web 服務的程式碼範例。

    # Consume through web service URL and keys
    from azureml import services
    @services.service(url, api_key)
    @services.types(sep_l = float, sep_w = float, pet_l=float, pet_w=float)
    @services.returns(float)
    def IrisPredictor(sep_l, sep_w, pet_l, pet_w):
    pass

    IrisPredictor(3,2,3,4)


> [!NOTE]
> Azure Machine Learning 程式庫目前只在 Python 2.7 上支援。   
> 
> 

### <a name="build-and-operationalize-r-models"></a>建置和實作 R 模型
您可以用類似於處理 Python 的方式，將在 Data Science Virtual Machine 或其他位置建置的 R 模型部署到 Azure Machine Learning。 步驟如下：

* 建立 settings.json 檔案，以提供您的工作區識別碼和驗證權杖 
* 針對模型的預測函式，撰寫一個包裝函式。
* 呼叫 Azure Machine Learning 程式庫中的 ```publishWebService``` 以傳入函式包裝函式。  

以下是可用來安裝、建置、發佈和取用模型作為 Azure Machine Learning Web 服務的程序和程式碼片段。

#### <a name="setup"></a>設定

* 在主目錄下名為 ```.azureml``` 的目錄下建立 settings.json 檔案，並輸入來自 Azure Machine Learning 工作區的參數：

settings.json 檔案結構：

    {"workspace":{
    "id"                  : "ENTER YOUR AZUREML WORKSPACE ID",
    "authorization_token" : "ENTER YOUR AZUREML AUTH TOKEN"
    }}


#### <a name="build-a-model-in-r-and-publish-it-in-azure-machine-learning"></a>以 R 建置模型並在 Azure Machine Learning 中發佈
    library(AzureML)
    ws <- workspace(config="~/.azureml/settings.json")

    if(!require("lme4")) install.packages("lme4")
    library(lme4)
    set.seed(1)
    train <- sleepstudy[sample(nrow(sleepstudy), 120),]
    m <- lm(Reaction ~ Days + Subject, data = train)

    # Define a prediction function to publish based on the model:
    sleepyPredict <- function(newdata){
          predict(m, newdata=newdata)
    }

    ep <- publishWebService(ws, fun = sleepyPredict, name="sleepy lm", inputSchema = sleepstudy, data.frame=TRUE)

#### <a name="consume-the-model-deployed-in-azure-machine-learning"></a>取用 Azure Machine Learning 中部署的模型
為了從用戶端應用程式取用模型，我們使用 Azure Machine Learning 程式庫來透過 `services` API 呼叫依名稱查閱已發佈的 Web 服務，進而決定端點。 然後您只要呼叫 `consume` 函式並傳入要預測的資料框架。
下列程式碼用來取用已發佈為 Azure Machine Learning Web 服務的模型。

    library(AzureML)
    library(lme4)
    ws <- workspace(config="~/.azureml/settings.json")

    s <-  services(ws, name = "sleepy lm")
    s <- tail(s, 1) # use the last published function, in case of duplicate function names

    ep <- endpoints(ws, s)

    # OK, try this out, and compare with raw data
    ans = consume(ep, sleepstudy)$ans

Azure Machine Learning R 程式庫的詳細資訊可以在[這裡](https://cran.r-project.org/web/packages/AzureML/AzureML.pdf)找到。

## <a name="4-administer-your-azure-resources-using-azure-portal-or-powershell"></a>4.使用 Azure 入口網站或 Powershell 管理您的 Azure 資源
DSVM 不僅可讓您在虛擬機器本機建置分析解決方案，也可讓您存取 Microsoft Azure 雲端上的服務。 Azure 提供數個可從 DSVM 管理和存取的計算、儲存、資料分析服務及其他服務。

若要管理 Azure 訂用帳戶和雲端資源，您可以使用瀏覽器並指向 [Azure 入口網站](https://portal.azure.com)。 您也可以使用 Azure Powershell 並透過指令碼管理 Azure 訂用帳戶和資源。
您可以從桌面上的捷徑或從標題為 "Microsoft Azure Powershell" 的 [開始] 功能表執行 Azure Powershell。 如需如何使用 Windows Powershell 指令碼管理 Azure 訂用帳戶和資源的詳細資訊，請參閱 [Microsoft Azure Powershell 文件](../../powershell-azure-resource-manager.md) 。

## <a name="5-extend-your-storage-space-with-a-shared-file-system"></a>5.使用共用的檔案系統來擴充您的儲存空間
資料科學家可以在小組內共用大型資料集、程式碼或其他資源。 DSVM 本身有大約 45 GB 的可用空間。 若要擴充您的儲存體，您可以使用「Azure 檔案服務」，然後將它掛接在一或多個 DSVM 執行個體上或透過 REST API 存取。  您也可以使用 [Azure 入口網站](../../virtual-machines/windows/attach-managed-disk-portal.md)或使用 [Azure Powershell](../../virtual-machines/windows/attach-disk-ps.md) 來新增額外的專用資料磁碟。 

> [!NOTE]
> 「Azure 檔案服務」共用的空間上限為 5 TB，個別檔案大小限制為 1 TB。 
> 
> 

您可以使用 Azure PowerShell 建立 Azure 檔案服務共用。 以下是可在 Azure PowerShell 下執行以建立 Azure 檔案服務共用的指令碼。

    # Authenticate to Azure.
    Connect-AzureRmAccount
    # Select your subscription
    Get-AzureRmSubscription –SubscriptionName "<your subscription name>" | Select-AzureRmSubscription
    # Create a new resource group.
    New-AzureRmResourceGroup -Name <dsvmdatarg>
    # Create a new storage account. You can reuse existing storage account if you wish.
    New-AzureRmStorageAccount -Name <mydatadisk> -ResourceGroupName <dsvmdatarg> -Location "<Azure Data Center Name For eg. South Central US>" -Type "Standard_LRS"
    # Set your current working storage account
    Set-AzureRmCurrentStorageAccount –ResourceGroupName "<dsvmdatarg>" –StorageAccountName <mydatadisk>

    # Create a Azure File Service Share
    $s = New-AzureStorageShare <<teamsharename>>
    # Create a directory under the FIle share. You can give it any name
    New-AzureStorageDirectory -Share $s -Path <directory name>
    # List the share to confirm that everything worked
    Get-AzureStorageFile -Share $s


您現在已建立 Azure 檔案共用，即可將它掛接在 Azure 的任何虛擬機器中。 強烈建議 VM 位於與儲存體帳戶相同的 Azure 資料中心，以避免延遲和資料傳輸費用。 以下是您可以在 Azure Powershell 上執行以在 DSVM 上掛接磁碟機的命令。

    # Get storage key of the storage account that has the Azure file share from Azure portal. Store it securely on the VM to avoid prompted in next command.
    cmdkey /add:<<mydatadisk>>.file.core.windows.net /user:<<mydatadisk>> /pass:<storage key>

    # Mount the Azure file share as Z: drive on the VM. You can chose another drive letter if you wish
    net use z:  \\<mydatadisk>.file.core.windows.net\<<teamsharename>>


您現在可以將此磁碟機當作 VM 上的任何一般磁碟機存取。

## <a name="6-share-code-with-your-team-using-github"></a>6.使用 GitHub 與您的小組共用程式碼
GitHub 是一個程式碼存放庫，您可以在其中找到由開發人員社群所分享、適用於 使用各種技術之不同工具的範例程式碼和來源。 它會使用 Git 技術來追蹤和儲存各版本的程式碼檔案。 GitHub 也是一個平台，您可以在其中建立自己的存放庫，以儲存您小組共用的程式碼和文件、實作版本控制，也可以控制有權檢視和貢獻程式碼的人員。 如需有關使用 Git 的詳細資訊，請瀏覽 [GitHub 說明頁面](https://help.github.com/) \(英文\)。 您可以使用 GitHub 來與您的小組共同作業、使用社群所開發的程式碼，以及將程式碼貢獻回社群。

DSVM 已在命令列和 GUI 上載入用戶端工具，以便存取 GitHub 存放庫。 可搭配 Git 與 GitHub 使用的命令列工具稱為 Git Bash。 DSVM 上安裝的 Visual Studio 具有 Git 延伸模組。 您可以在 [開始] 功能表和桌面上找到這些工具的啟動圖示。

若要從 GitHub 存放庫下載程式碼，您可以使用 ```git clone``` 命令。 例如，若要將 Microsoft 所發佈的資料科學存放庫下載到目前的目錄中，您可以在 ```git-bash``` 中執行下列命令。

    git clone https://github.com/Azure/DataScienceVM.git

您可以在 Visual Studio 中執行相同的複製作業。 下列螢幕擷取畫面示範如何在 Visual Studio 中存取 Git 和 GitHub 工具。

![Visual Studio 中的 Git](./media/vm-do-ten-things/VSGit.PNG)

從 github.com 上提供的幾個資源，即可找到透過 Git 使用 GitHub 存放庫的詳細資訊。 [功能提要](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf) 是有用的參考資料。

## <a name="7-access-various-azure-data-and-analytics-services"></a>7.存取各種 Azure 資料和分析服務
### <a name="azure-blob"></a>Azure Blob
Azure blob 是可靠、划算的雲端儲存體，可存放大型和小型的資料。 本節說明如何將資料移至 Azure Blob 及存取儲存在 Azure Blob 中的資料。

**先決條件**

* **從 [Azure 入口網站](https://portal.azure.com)建立 Azure Blob 儲存體帳戶。**

![Create_Azure_Blob](./media/vm-do-ten-things/Create_Azure_Blob.PNG)

* 確認預先安裝的命令列 AzCopy 工具位於 ```C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy.exe```。 包含 azcopy.exe 的目錄已經您的 PATH 環境變數上，如此便無須在執行此工具時輸入完整的命令路徑。 如需有關 AzCopy 工具的詳細資訊，請參閱 [AzCopy 文件](../../storage/common/storage-use-azcopy.md)
* 啟動 Azure 儲存體總管工具。 您可以從 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)下載此工具。 

![AzureStorageExplorer_v4](./media/vm-do-ten-things/AzureStorageExplorer_v4.png)

**將資料從 VM 移至 Azure Blob：AzCopy**

若要在本機檔案與 Blob 儲存體之間移動資料，您可以在命令列或 PowerShell 中使用 AzCopy：

    AzCopy /Source:C:\myfolder /Dest:https://<mystorageaccount>.blob.core.windows.net/<mycontainer> /DestKey:<storage account key> /Pattern:abc.txt

以您的檔案儲存路徑取代 **C:\myfolder**，以您的 Blob 儲存體帳戶名稱取代 **mystorageaccount**、以容器名稱取代 **mycontainer**，並以您的 Blob 儲存體存取金鑰取代 **storage account key**。 您可以在 [Azure 入口網站](https://portal.azure.com)中尋找您的儲存體帳戶認證。

![StorageAccountCredential_v2](./media/vm-do-ten-things/StorageAccountCredential_v2.png)

在 PowerShell 或從命令提示字元執行 AzCopy 命令。 以下是使用 AzCopy 命令的一些範例：

    # Copy *.sql from local machine to a Azure Blob
    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:"c:\Aaqs\Data Science Scripts" /Dest:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /DestKey:[ENTER STORAGE KEY] /S /Pattern:*.sql

    # Copy back all files from Azure Blob container to Local machine

    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Dest:"c:\Aaqs\Data Science Scripts\temp" /Source:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /SourceKey:[ENTER STORAGE KEY] /S



執行 AzCopy 命令來複製到 Azure Blob 之後，您會看到檔案隨即出現在「Azure 儲存體總管」中。

![AzCopy_run_finshed_Storage_Explorer_v3](./media/vm-do-ten-things/AzCopy_run_finshed_Storage_Explorer_v3.png)

**將資料從 VM 移至 Azure Blob：Azure 儲存體總管**

您也可以使用 Azure 儲存體總管，從 VM 中的本機檔案上載資料：

* 若要將資料上傳至容器，請選取目標容器，然後按一下 [上傳] 按鈕。![儲存體總管中的 [上傳]](./media/vm-do-ten-things/storage-accounts.png)
* 按一下 [檔案] 方塊右邊的 [...]，從檔案系統中選取一或多個要上傳的檔案，然後按一下 [上傳] 以開始上傳檔案。![上傳檔案到 Blob](./media/vm-do-ten-things/upload-files-to-blob.png)

**從 Azure Blob 讀取資料：Machine Learning 讀取器模組**

在 Azure Machine Learning Studio 中，您可以使用**匯入資料模組**從 Blob 讀取資料。

![AML_ReaderBlob_Module_v3](./media/vm-do-ten-things/AML_ReaderBlob_Module_v3.png)

**從 Azure Blob 讀取資料：Python ODBC**

您可以使用 **BlobService** 程式庫，直接從 Jupyter 筆記本或 Python 程式的 Blob 中讀取資料。

首先，匯入所需的封裝：

    import pandas as pd
    from pandas import Series, DataFrame
    import numpy as np
    import matplotlib.pyplot as plt
    from time import time
    import pyodbc
    import os
    from azure.storage.blob import BlobService
    import tables
    import time
    import zipfile
    import random

然後插入您的 Azure Blob 帳戶認證並從 Blob 讀取資料：

    CONTAINERNAME = 'xxx'
    STORAGEACCOUNTNAME = 'xxxx'
    STORAGEACCOUNTKEY = 'xxxxxxxxxxxxxxxx'
    BLOBNAME = 'nyctaxidataset/nyctaxitrip/trip_data_1.csv'
    localfilename = 'trip_data_1.csv'
    LOCALDIRECTORY = os.getcwd()
    LOCALFILE =  os.path.join(LOCALDIRECTORY, localfilename)

    #download from blob
    t1 = time.time()
    blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILE)
    t2 = time.time()
    print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))

    #unzipping downloaded files if needed
    #with zipfile.ZipFile(ZIPPEDLOCALFILE, "r") as z:
    #    z.extractall(LOCALDIRECTORY)

    df1 = pd.read_csv(LOCALFILE, header=0)
    df1.columns = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime','passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude']
    print 'the size of the data is: %d rows and  %d columns' % df1.shape

資料以資料框架的形式讀入：

![IPNB_data_readin](./media/vm-do-ten-things/IPNB_data_readin.PNG)

### <a name="azure-data-lake"></a>Azure Data Lake
Azure Data Lake 儲存體是巨量資料分析工作負載的超大規模儲存機制，與 Hadoop 分散式檔案系統 (HDFS) 相容。 它可以與 Hadoop、Spark 及 Azure Data Lake Analytics 搭配運作。 在本節中，您將了解如何將資料移至 Azure Data Lake Store，然後使用 Azure Data Lake Analytics 來執行分析。

**先決條件**

* 在 [Azure 入口網站](https://portal.azure.com)中建立 Azure Data Lake Analytics。

![Azure_Data_Lake_Create_v2](./media/vm-do-ten-things/Azure_Data_Lake_Create_v2.png)

* 在這個[連結](https://www.microsoft.com/download/details.aspx?id=49504)找到之 **Visual Studio** 中的 **Azure Data Lake Tools** 已經安裝在虛擬機器上的 Visual Studio Community Edition 上。 啟動 Visual Studio 並登入您的 Azure 訂用帳戶之後，您應該會在 Visual Studio 的左面板中看到 Azure 資料分析帳戶和儲存體。

![Azure_Data_Lake_PlugIn_v2](./media/vm-do-ten-things/Azure_Data_Lake_PlugIn_v2.PNG)

**將資料從 VM 移至資料湖：Azure 資料湖總管**

您可以使用 **Azure Data Lake Explorer** ，將資料從虛擬機器中的本機檔案上傳至 Data Lake 儲存體。

![Azure_Data_Lake_UploadData](./media/vm-do-ten-things/Azure_Data_Lake_UploadData.PNG)

您也可以使用 [Azure Data Factory (ADF)](https://azure.microsoft.com/services/data-factory/) 來建置資料管線，以將資料在 Azure Data Lake 的移入和移出操作化。 請參閱此[文章](https://azure.microsoft.com/blog/creating-big-data-pipelines-using-azure-data-lake-and-azure-data-factory/) ，以引導您完成建置資料管線的步驟。

**將資料從 Azure Blob 讀取至資料湖：U-SQL**

如果您的資料位於 Azure Blob 儲存體中，您可以在 U-SQL 查詢中從 Azure 儲存體 Blob 直接讀取資料。 撰寫 U-SQL 查詢之前，請確定您的 Blob 儲存體帳戶已連結到您的 Azure 資料湖。 移至 **Azure 入口網站**、尋找您的 Azure Data Lake Analytics 儀表板、按一下 [新增資料來源]、選取 [Azure 儲存體] 做為儲存體類型，並插入您的 Azure 儲存體帳戶名稱和金鑰。 然後您可以參考儲存體帳戶中儲存的資料。

![輸入儲存體帳戶和金鑰](./media/vm-do-ten-things/Link_Blob_to_ADLA_v2.PNG)

您可以在 Visual Studio 中，從 Blob 儲存體讀取資料、進行一些資料操作、功能工程，以及將結果資料輸出至 Azure 資料湖或 Azure Blob 儲存體。 當您參考 Blob 儲存體中的資料時，請使用 **wasb://**；當您參考 Azure Data Lake 中的資料時，請使用 **swbhdfs://**

![資料框架](./media/vm-do-ten-things/USQL_Read_Blob_v2.PNG)

您可以在 Visual Studio 中，使用下列 U-SQL 查詢：

    @a =
        EXTRACT medallion string,
                hack_license string,
                vendor_id string,
                rate_code string,
                store_and_fwd_flag string,
                pickup_datetime string,
                dropoff_datetime string,
                passenger_count int,
                trip_time_in_secs double,
                trip_distance double,
                pickup_longitude string,
                pickup_latitude string,
                dropoff_longitude string,
                dropoff_latitude string

        FROM "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Input Data File Name>"
        USING Extractors.Csv();

    @b =
        SELECT vendor_id,
        COUNT(medallion) AS cnt_medallion,
        SUM(passenger_count) AS cnt_passenger,
        AVG(trip_distance) AS avg_trip_dist,
        MIN(trip_distance) AS min_trip_dist,
        MAX(trip_distance) AS max_trip_dist,
        AVG(trip_time_in_secs) AS avg_trip_time
        FROM @a
        GROUP BY vendor_id;

    OUTPUT @b   
    TO "swebhdfs://<Azure Data Lake Storage Account Name>.azuredatalakestore.net/<Folder Name>/<Output Data File Name>"
    USING Outputters.Csv();

    OUTPUT @b   
    TO "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Output Data File Name>"
    USING Outputters.Csv();



將查詢提交至伺服器之後，會顯示一個圖表來顯示您的作業狀態。

![作業狀態圖表](./media/vm-do-ten-things/USQL_Job_Status.PNG)

**查詢資料湖中的資料：U-SQL**

將資料集擷取到 Azure Data Lake 之後，您可以使用 [U-SQL 語言](../../data-lake-analytics/data-lake-analytics-u-sql-get-started.md)來查詢和探索資料。 U-SQL 語言類似於 T-SQL，但結合了 C# 的一些功能，以便使用者撰寫自訂的模組、使用者定義的功能等。您可以在上一個步驟中使用指令碼。

將查詢提交到伺服器之後，很快就可以在 **Azure Data Lake Explorer** 中找到 tripdata_summary.CSV，以滑鼠右鍵按一下該檔案即可預覽資料。

![Azure Data Lake Explorer 中的檔案](./media/vm-do-ten-things/USQL_create_summary.png)

若要查看檔案資訊：

![檔案摘要](./media/vm-do-ten-things/USQL_tripdata_summary.png)

### <a name="hdinsight-hadoop-clusters"></a>HDInsight Hadoop 叢集
Azure HDInsight 是在雲端上的受控 Apache Hadoop、Spark、HBase 和 Storm 服務。 您可以輕鬆地從資料科學虛擬機器使用 Azure HDInsight 叢集。

**先決條件**

* 從 [Azure 入口網站](https://portal.azure.com)建立 Azure Blob 儲存體帳戶。 此儲存體帳戶用來儲存 HDInsight 叢集的資料。

![建立 Azure Blob 儲存體帳戶](./media/vm-do-ten-things/Create_Azure_Blob.PNG)

* 從 [Azure 入口網站](../team-data-science-process/customize-hadoop-cluster.md)
  
  * 在建立 HDInsight 叢集後，將所建立的儲存體帳戶與 HDInsight 叢集連結。 此儲存體帳戶用於存取可以在叢集內處理的資料。

![連結到以 HDInsight 叢集建立的儲存體帳戶](./media/vm-do-ten-things/Create_HDI_v4.PNG)

* 在建立叢集後，對前端節點啟用 [遠端存取]。 請記住您在這裡指定的遠端存取認證，在後續程序中將會需要用到它們。

![啟用遠端存取](./media/vm-do-ten-things/Create_HDI_dashboard_v3.PNG)

* 建立 Azure Machine Learning 工作區。 您的機器學習實驗將會儲存在此 Machine Learning 工作區中。 在入口網站中選取反白顯示的選項，如下列螢幕擷取畫面所示：

![建立 Azure Machine Learning 工作區](./media/vm-do-ten-things/Create_ML_Space.PNG)

* 然後輸入工作區的參數

![輸入 Machine Learning 工作區參數](./media/vm-do-ten-things/Create_ML_Space_step2_v2.PNG)

* 使用 IPython Notebook 上傳資料。 先匯入所需的封裝、插入認證在儲存體帳戶中建立資料庫，然後將資料載入 HDI 叢集。

        #Import required Packages
        import pyodbc
        import time as time
        import json
        import os
        import urllib
        import urllib2
        import warnings
        import re
        import pandas as pd
        import matplotlib.pyplot as plt
        from azure.storage.blob import BlobService
        warnings.filterwarnings("ignore", category=UserWarning, module='urllib2')


        #Create the connection to Hive using ODBC
        SERVER_NAME='xxx.azurehdinsight.net'
        DATABASE_NAME='nyctaxidb'
        USERID='xxx'
        PASSWORD='xxxx'
        DB_DRIVER='Microsoft Hive ODBC Driver'
        driver = 'DRIVER={' + DB_DRIVER + '}'
        server = 'Host=' + SERVER_NAME + ';Port=443'
        database = 'Schema=' + DATABASE_NAME
        hiveserv = 'HiveServerType=2'
        auth = 'AuthMech=6'
        uid = 'UID=' + USERID
        pwd = 'PWD=' + PASSWORD
        CONNECTION_STRING = ';'.join([driver,server,database,hiveserv,auth,uid,pwd])
        connection = pyodbc.connect(CONNECTION_STRING, autocommit=True)
        cursor=connection.cursor()


        #Create Hive database and tables
        queryString = "create database if not exists nyctaxidb;"
        cursor.execute(queryString)

        queryString = """
                        create external table if not exists nyctaxidb.trip
                        (
                            medallion string,
                            hack_license string,
                            vendor_id string,
                            rate_code string,
                            store_and_fwd_flag string,
                            pickup_datetime string,
                            dropoff_datetime string,
                            passenger_count int,
                            trip_time_in_secs double,
                            trip_distance double,
                            pickup_longitude double,
                            pickup_latitude double,
                            dropoff_longitude double,
                            dropoff_latitude double)  
                        PARTITIONED BY (month int)
                        ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\\n'
                        STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/trip' TBLPROPERTIES('skip.header.line.count'='1');
                    """
        cursor.execute(queryString)

        queryString = """
                        create external table if not exists nyctaxidb.fare
                        (
                            medallion string,
                            hack_license string,
                            vendor_id string,
                            pickup_datetime string,
                            payment_type string,
                            fare_amount double,
                            surcharge double,
                            mta_tax double,
                            tip_amount double,
                            tolls_amount double,
                            total_amount double)
                        PARTITIONED BY (month int)
                        ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\\n'
                        STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/fare' TBLPROPERTIES('skip.header.line.count'='1');
                    """
        cursor.execute(queryString)


        #Upload data from blob storage to HDI cluster
        for i in range(1,13):
            queryString = "LOAD DATA INPATH 'wasb:///nyctaxitripraw2/trip_data_%d.csv' INTO TABLE nyctaxidb2.trip PARTITION (month=%d);"%(i,i)
            cursor.execute(queryString)
            queryString = "LOAD DATA INPATH 'wasb:///nyctaxifareraw2/trip_fare_%d.csv' INTO TABLE nyctaxidb2.fare PARTITION (month=%d);"%(i,i)  
            cursor.execute(queryString)


* 或者，您可以遵循此[逐步解說](../team-data-science-process/hive-walkthrough.md)，將 NYC 計程車資料上傳至 HDI 叢集。 主要步驟包括：
  
  * AzCopy：從公用 Blob 將已壓縮的 CSV 下載到本機資料夾
  * AzCopy：從本機資料夾將已解壓縮的 CSV 上傳至 HDI 叢集
  * 登入 Hadoop 叢集的前端節點，並準備進行探索資料分析

在資料載入至 HDI 叢集之後，您可以在 Azure 儲存體總管中檢查您的資料。 而您會在 HDI 叢集中建立資料庫 nyctaxidb。

**資料探索：Python 中的 Hive 查詢**

因為資料位於 Hadoop 叢集中，您可以使用 pyodbc 封裝連接到 Hadoop 叢集，並使用 Hive 查詢資料庫以便進行探索和設計功能。 您可以檢視在先決條件步驟中建立的現有資料表。

    queryString = """
        show tables in nyctaxidb2;
        """
    pd.read_sql(queryString,connection)


![檢視現有的表格](./media/vm-do-ten-things/Python_View_Existing_Tables_Hive_v3.PNG)

讓我們看一下每個月的記錄筆數以及車程資料表中已付小費或未付小費的頻率：

    queryString = """
        select month, count(*) from nyctaxidb.trip group by month;
        """
    results = pd.read_sql(queryString,connection)

    %matplotlib inline

    results.columns = ['month', 'trip_count']
    df = results.copy()
    df.index = df['month']
    df['trip_count'].plot(kind='bar')


![每個月的記錄數繪圖](./media/vm-do-ten-things/Exploration_Number_Records_by_Month_v3.PNG)

    queryString = """
        SELECT tipped, COUNT(*) AS tip_freq
        FROM
        (
            SELECT if(tip_amount > 0, 1, 0) as tipped, tip_amount
            FROM nyctaxidb.fare
        )tc
        GROUP BY tipped;
        """
    results = pd.read_sql(queryString,connection)

    results.columns = ['tipped', 'trip_count']
    df = results.copy()
    df.index = df['tipped']
    df['trip_count'].plot(kind='bar')


![小費頻率繪圖](./media/vm-do-ten-things/Exploration_Frequency_tip_or_not_v3.PNG)

您也可以計算上車位置與下車位置之間的距離，然後再與車程距離做比較。

    queryString = """
                    select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, trip_distance, trip_time_in_secs,
                        3959*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
                        *radians(180)/180/2),2)-cos(pickup_latitude*radians(180)/180)
                        *cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2)))
                        /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*radians(180)/180/2),2)
                        +cos(pickup_latitude*radians(180)/180)*cos(dropoff_latitude*radians(180)/180)*
                        pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2))) as direct_distance
                        from nyctaxidb.trip
                        where month=1
                            and pickup_longitude between -90 and -30
                            and pickup_latitude between 30 and 90
                            and dropoff_longitude between -90 and -30
                            and dropoff_latitude between 30 and 90;
                """
    results = pd.read_sql(queryString,connection)
    results.head(5)


![上下車資料表](./media/vm-do-ten-things/Exploration_compute_pickup_dropoff_distance_v2.PNG)

    results.columns = ['pickup_longitude', 'pickup_latitude', 'dropoff_longitude',
                       'dropoff_latitude', 'trip_distance', 'trip_time_in_secs', 'direct_distance']
    df = results.loc[results['trip_distance']<=100] #remove outliers
    df = df.loc[df['direct_distance']<=100] #remove outliers
    plt.scatter(df['direct_distance'], df['trip_distance'])


![上下車距離及車程距離繪圖](./media/vm-do-ten-things/Exploration_direct_distance_trip_distance_v2.PNG)

現在讓我們準備縮減取樣 (1%) 資料集，以便進行模型分析。 您可以在 Machine Learning 讀取器模組中使用此資料。

        queryString = """
        create  table if not exists nyctaxi_downsampled_dataset_testNEW (
        medallion string,
        hack_license string,
        vendor_id string,
        rate_code string,
        store_and_fwd_flag string,
        pickup_datetime string,
        dropoff_datetime string,
        pickup_hour string,
        pickup_week string,
        weekday string,
        passenger_count int,
        trip_time_in_secs double,
        trip_distance double,
        pickup_longitude double,
        pickup_latitude double,
        dropoff_longitude double,
        dropoff_latitude double,
        direct_distance double,
        payment_type string,
        fare_amount double,
        surcharge double,
        mta_tax double,
        tip_amount double,
        tolls_amount double,
        total_amount double,
        tipped string,
        tip_class string
        )
        row format delimited fields terminated by ','
        lines terminated by '\\n'
        stored as textfile;
        """
        cursor.execute(queryString)

        --- now insert contents of the join into the preceding internal table

        queryString = """
        insert overwrite table nyctaxi_downsampled_dataset_testNEW
        select
        t.medallion,
        t.hack_license,
        t.vendor_id,
        t.rate_code,
        t.store_and_fwd_flag,
        t.pickup_datetime,
        t.dropoff_datetime,
        hour(t.pickup_datetime) as pickup_hour,
        weekofyear(t.pickup_datetime) as pickup_week,
        from_unixtime(unix_timestamp(t.pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),'u') as weekday,
        t.passenger_count,
        t.trip_time_in_secs,
        t.trip_distance,
        t.pickup_longitude,
        t.pickup_latitude,
        t.dropoff_longitude,
        t.dropoff_latitude,
        t.direct_distance,
        f.payment_type,
        f.fare_amount,
        f.surcharge,
        f.mta_tax,
        f.tip_amount,
        f.tolls_amount,
        f.total_amount,
        if(tip_amount>0,1,0) as tipped,
        if(tip_amount=0,0,
        if(tip_amount>0 and tip_amount<=5,1,
        if(tip_amount>5 and tip_amount<=10,2,
        if(tip_amount>10 and tip_amount<=20,3,4)))) as tip_class
        from
        (
        select
        medallion,
        hack_license,
        vendor_id,
        rate_code,
        store_and_fwd_flag,
        pickup_datetime,
        dropoff_datetime,
        passenger_count,
        trip_time_in_secs,
        trip_distance,
        pickup_longitude,
        pickup_latitude,
        dropoff_longitude,
        dropoff_latitude,
        3959*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
        radians(180)/180/2),2)-cos(pickup_latitude*radians(180)/180)
        *cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2)))
        /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*radians(180)/180/2),2)
        +cos(pickup_latitude*radians(180)/180)*cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2))) as direct_distance,
        rand() as sample_key

        from trip
        where pickup_latitude between 30 and 90
            and pickup_longitude between -90 and -30
            and dropoff_latitude between 30 and 90
            and dropoff_longitude between -90 and -30
        )t
        join
        (
        select
        medallion,
        hack_license,
        vendor_id,
        pickup_datetime,
        payment_type,
        fare_amount,
        surcharge,
        mta_tax,
        tip_amount,
        tolls_amount,
        total_amount
        from fare
        )f
        on t.medallion=f.medallion and t.hack_license=f.hack_license and t.pickup_datetime=f.pickup_datetime
        where t.sample_key<=0.01
        """
        cursor.execute(queryString)

不久之後，您可以看見資料已載入 Hadoop 叢集中：

    queryString = """
        select * from nyctaxi_downsampled_dataset limit 10;
        """
    cursor.execute(queryString)
    pd.read_sql(queryString,connection)


![資料表](./media/vm-do-ten-things/DownSample_Data_For_Modeling_v2.PNG)

**使用 Machine Learning 從 HDI 讀取資料：讀取器模組**

您也可以在 Machine Learning Studio 中使用**讀取器**模組來存取 Hadoop 叢集中的資料庫。 請插入 HDI 叢集和「Azure 儲存體帳戶」的認證，以允許使用 HDI 叢集中的資料庫來建置機器學習模型。

![讀取器模組屬性](./media/vm-do-ten-things/AML_Reader_Hive.PNG)

接著可以檢視評分資料集：

![檢視已評分的資料集](./media/vm-do-ten-things/AML_Model_Results.PNG)

### <a name="azure-sql-data-warehouse--databases"></a>Azure SQL 資料倉儲和資料庫
Azure SQL 資料倉儲是彈性的資料倉儲即服務，具有企業層級的 SQL Server 體驗。

您可以透過依照此[文章](../../sql-data-warehouse/sql-data-warehouse-get-started-provision.md)中提供的指示，來佈建您的 Azure SQL 資料倉儲。 佈建「Azure SQL 資料倉儲」之後，您就可以使用這個[逐步解說](../team-data-science-process/sqldw-walkthrough.md)來進行資料上傳、探索，以及使用「SQL 資料倉儲」內的資料來建立模型。

#### <a name="azure-cosmos-db"></a>Azure Cosmos DB
Azure Cosmos DB 是雲端中的一種 NoSQL 資料庫。 它可讓您使用 JSON 等文件，並可讓您儲存和查詢文件。

以下是從 DSVM 存取 Azure Cosmos DB 所需的先決條件步驟：

1. 已經在 DSVM 上安裝 Azure Cosmos DB Python SDK (從命令提示字元執行 ```pip install pydocumentdb --upgrade``` 來進行更新)
2. 從 [Azure 入口網站](https://portal.azure.com)建立 Azure Cosmos DB 帳戶和資料庫
3. 從[這裡](http://www.microsoft.com/downloads/details.aspx?FamilyID=cda7703a-2774-4c07-adcc-ad02ddc1a44d)下載「Azure Cosmos DB 移轉工具」並解壓縮至您所選的目錄
4. 對移轉工具使用下列命令參數 (Cosmos DB 移轉工具安裝目錄中的 dtui.exe)，將儲存在[公用 Blob](https://cahandson.blob.core.windows.net/samples/volcano.json) 的 JSON 資料 (Volcano 資料) 匯入 Cosmos DB。 輸入下面的來源和目標位置參數：
   
    `/s:JsonFile /s.Files:https://cahandson.blob.core.windows.net/samples/volcano.json /t:DocumentDBBulk /t.ConnectionString:AccountEndpoint=https://[DocDBAccountName].documents.azure.com:443/;AccountKey=[[KEY];Database=volcano /t.Collection:volcano1`

匯入資料之後，您即可移至 Jupyter 並開啟標題為 *DocumentDBSample*  且包含 Python 程式碼的筆記本，以存取 Azure Cosmos DB 及進行一些基本查詢。 如需深入了解 Cosmos DB，請參閱服務[文件頁面](https://docs.microsoft.com/azure/cosmos-db/)。

## <a name="8-build-reports-and-dashboard-using-the-power-bi-desktop"></a>8.使用 Power BI Desktop 建立報表和儀表板
您可以在 Power BI 中將前述 Cosmos DB 範例中的 Volcano JSON 檔案視覺化，以深入了解資料。 在 [Power BI 文章](../../cosmos-db/powerbi-visualize.md)中可找到詳細的步驟。 高階步驟如下：

1. 開啟 Power BI Desktop 並執行「取得資料」。 將 URL 指定為︰ https://cahandson.blob.core.windows.net/samples/volcano.json
2. 您應該會看到匯入為清單的 JSON 記錄
3. 將清單轉換成資料表供 PowerBI 使用
4. 按一下展開圖示 (資料行左側具有「向左箭號與向右箭號」的圖示)，即可展開資料行
5. 請注意，該位置是「記錄」欄位。 展開記錄，然後只選取座標。 座標是清單資料行
6. 加入新的資料行，以將清單座標資料行轉換成逗號分隔的 LatLong 資料行，其使用公式 ```Text.From([coordinates]{1})&","&Text.From([coordinates]{0})```來串連座標清單欄位中的兩個元素。
7. 最後將 ```Elevation``` 資料行轉換成 [十進位]，然後選取 [關閉] 和 [套用]。

除了前述步驟，您可以貼上下列程式碼，此程式碼會以指令碼撰寫在 PowerBI 的 [進階編輯器] 中使用的步驟，讓您以查詢語言撰寫資料轉換。

    let
        Source = Json.Document(Web.Contents("https://cahandson.blob.core.windows.net/samples/volcano.json")),
        #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
        #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}, {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}),
        #"Expanded Location" = Table.ExpandRecordColumn(#"Expanded Column1", "Location", {"coordinates"}, {"coordinates"}),
        #"Added Custom" = Table.AddColumn(#"Expanded Location", "LatLong", each Text.From([coordinates]{1})&","&Text.From([coordinates]{0})),
        #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Elevation", type number}})
    in
        #"Changed Type"



您的 Power BI 資料模型中現在會有資料。 您的 Power BI Desktop 應該如下所示：

![Power BI 桌面](./media/vm-do-ten-things/PowerBIVolcanoData.png)

您可以開始使用資料模型來建立報告和視覺效果。 您可以依照此 [Power BI 文章](../../cosmos-db/powerbi-visualize.md#build-the-reports)中的步驟來建立報告。 輸出會是一個如以下所示的報告。

![Power BI Desktop 報告檢視 - Power BI 連接器](./media/vm-do-ten-things/power_bi_connector_pbireportview2.png)

## <a name="9-dynamically-scale-your-dsvm-to-meet-your-project-needs"></a>9.動態調整 DSVM 以符合您的專案需求
您可以相應增加和減少 DSVM 以符合您的專案需求。 如果您在晚上或週末不需要使用 VM，就可以從 [Azure 入口網站](https://portal.azure.com)關閉 VM。

> [!NOTE]
> 如果您只使用 VM 上的作業系統關機按鈕，則需支付計算費用。  
> 
> 

如果您需要處理一些大規模分析，而需要更多 CPU 和/或記憶體和/或磁碟容量，您可以找到許多在 CPU 核心、用於深度學習的 GPU 型執行個體、記憶體容量及磁碟類型 (包括固態硬碟機) 方面，符合您計算和預算需求的 VM 大小選擇。 [Azure 虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/)頁面提供完整 VM 清單和其每小時的計算價格。

同樣地，如果您的 VM 處理容量需求降低 (例如：將主要工作負載移至 Hadoop 或 Spark 叢集)，您可以從 [Azure 入口網站](https://portal.azure.com)移至您的 VM 執行個體設定來相應減少叢集。 以下為螢幕擷取畫面。

![VM 執行個體設定](./media/vm-do-ten-things/VMScaling.PNG)

## <a name="10-install-additional-tools-on-your-virtual-machine"></a>10.在虛擬機器上安裝其他工具
DSVM 中已預先建置數個工具，可應付許多常見的資料分析需求。 這可讓您因無須逐一安裝和設定環境而節省您的時間，因只需支付所用的資源而節省您的成本。

您可以運用本文略述的其他 Azure 資料和分析服務，以增強您的分析環境。 在某些情況下，您的需求可能需要額外的工具，包括一些專屬的協力廠商工具。 您有完整的系統管理權限可在虛擬機器上安裝您所需的新工具。 您也可以在 Python 和 R 中安裝其他未預先安裝的封裝。 對於 Python，您可以使用 ```conda``` 或 ```pip```。 對於 R，您可以在 R 主控台中使用 ```install.packages()```，或使用整合式開發環境 (IDE) 並選擇 [套件] -> [安裝套件]。

## <a name="summary"></a>總結
這些只是您可以在 Microsoft Data Science Virtual Machine 上可做的一些事情。 您還可以做更多事情，讓它成為有效的分析環境。

