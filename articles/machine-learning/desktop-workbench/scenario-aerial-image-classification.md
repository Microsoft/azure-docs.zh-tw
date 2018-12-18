---
title: 空拍影像分類 | Microsoft Docs
description: 提供關於空拍影像分類實際案例的指示
author: mawah
ms.author: mawah
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.topic: article
ms.service: machine-learning
ms.component: core
services: machine-learning
ms.workload: data-services
ms.date: 12/13/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 5ca47c8234239b56a2d829903828dda8220d53cb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967603"
---
# <a name="aerial-image-classification"></a>空拍影像分類

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



此範例示範如何使用 Azure Machine Learning Workbench 來協調影像分類模型的分散式訓練和運作。 我們會使用兩個方法進行訓練：(i) 使用 [Azure Batch AI](https://docs.microsoft.com/azure/batch-ai/) GPU 叢集將深度類神經網路加以精簡，以及 (ii) 使用 [Microsoft Machine Learning for Apache Spark (MMLSpark)](https://github.com/Azure/mmlspark) 套件，透過預先訓練好的 CNTK 模型建立影像特徵，並使用衍生出來的特徵來訓練分類器。 接著，我們會使用 [Azure HDInsight Spark](https://azure.microsoft.com/services/hdinsight/apache-spark/) 叢集，以平行方式將定型的模型套用到雲端中的大型影像集合，透過新增或移除背景工作角色節點，讓我們可以調整訓練和實施的速度。

此範例會重點摘要兩種方法來傳輸學習，能運用預先定型的模型來避免從頭定型深度類神經網路的費用。 定型深度類神經網路通常需要 GPU 計算，但定型集夠大時，可能會導致較佳的精確度。 在特徵化映像上定型簡單的分類並不需要 GPU 計算、原本就很快速、可任意擴充且符合較少參數。 因此，這個方法很適合可用的訓練範例較少時的情況，而自訂使用案例通常就是這種情況。 

此範例中使用的 Spark 叢集有 40 個背景工作角色節點，運作成本為 ~$40/小時，Batch AI 叢集資源包含八個 GPU，運作成本為 ~$10/小時。 完成此逐步解說需花費約兩個小時。 當您完成時，請按照清除指示來移除您所建立的資源，從而停止產生費用。

## <a name="link-to-the-gallery-github-repository"></a>連結至資源庫 GitHub 存放庫

此實際案例的公用 GitHub 存放庫包含此範例所需的所有資料，包括程式碼範例：

[https://github.com/Azure/MachineLearningSamples-AerialImageClassification](https://github.com/Azure/MachineLearningSamples-AerialImageClassification)

## <a name="use-case-description"></a>使用案例說明

在此案例中，我們會訓練機器學習模型，讓模型為空拍影像中所顯示的土地 (224 公尺 x 224 公尺) 類型分類。 您可以使用土地用途分類模型，利用定期收集的空拍影像追蹤都市化、森林砍伐、濕地流失和其他重大環境趨勢。 我們已備妥訓練和驗證用的影像集合，此集合是由從美國農業部影像計劃 (U.S. National Agriculture Imagery Program) 所取得的影像，以及美國國土覆被資料庫 (U.S. National Land Cover Database) 所發佈的土地用途標籤，共同建構而成。 每個土地用途類別的範例影像如下所示：

![每個土地用途標籤的範例區域](media/scenario-aerial-image-classification/example-labels.PNG)

在訓練和驗證分類器模型後，我們會將它套用到拍攝範圍橫跨麻薩諸塞州密得塞斯郡 (也就是 Microsoft 的新英格蘭研發中心 (NERD) 所在地) 的空拍影像，以示範如何使用這些模型來研究都市發展趨勢。

為了產生使用遷移學習的影像分類器，資料科學家通常會建構具有各種方法的多個模型，並選取效能最好的模型。 Azure Machine Learning Workbench 可以協助資料科學家協調不同計算環境之間的訓練、追蹤和比較多個模型的效能，並將選擇的模型套用到雲端上的大型資料集。

## <a name="scenario-structure"></a>案例結構

在此範例中，影像資料和預先訓練好的模型會存放在 Azure 儲存體帳戶內。 Azure HDInsight Spark 叢集會讀取這些檔案，並使用 MMLSpark 建構影像分類模型。 訓練好的模型和其預測接著會寫入至儲存體帳戶，以供在本機執行的 Jupyter 筆記本進行分析和視覺化。 Azure Machine Learning Workbench 可協調 Spark 叢集上指令碼的遠端執行。 它也能追蹤多個使用不同方法訓練而成之模型的精確度計量，讓使用者能夠選取效能最好的模型。

![空拍影像分類實際案例的圖解](media/scenario-aerial-image-classification/scenario-schematic.PNG)

這些逐步指示一開始會先帶領您建立和準備 Azure 儲存體帳戶與 Spark 叢集，包括資料傳輸和相依性安裝。 接著，它們會說明如何啟動訓練作業，並比較所產生模型的效能。 最後，它們會說明如何將選擇的模型套用到 Spark 叢集上的大型影像集合，並在本機分析預測結果。


## <a name="set-up-the-execution-environment"></a>設定執行環境

下列指示會引導您完成為此範例設定執行環境的程序。

### <a name="prerequisites"></a>必要條件
- [Azure 帳戶](https://azure.microsoft.com/free/) (提供免費試用)
    - 您會建立具有 40 個背景工作節點 (總計 168 個核心) 的 HDInsight Spark 叢集。 請在 Azure 入口網站的訂用帳戶中檢閱 [使用量 + 配額] 索引標籤，以確定您的帳戶具有足夠的可用核心。
       - 如果您的可用核心較少，則可以修改 HDInsight 叢集範本，以減少佈建的背景工作數目。 這方面的指示會出現在 [建立 HDInsight Spark 叢集] 區段下。
    - 這個範例會建立具有兩個 NC6 (1 GPU，6 vCPU) VM 的 Batch AI 定型叢集。 請在 Azure 入口網站的訂用帳戶中檢閱 [使用量 + 配額] 索引標籤，以確定您在美國東部區域的帳戶具有足夠的可用核心。
- [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md)
    - 請遵循[安裝及建立快速入門](../desktop-workbench/quickstart-installation.md)，以安裝 Azure Machine Learning Workbench 並建立測試帳戶和模型管理帳戶。
- [Batch AI](https://github.com/Azure/BatchAI) Python SDK 和 Azure CLI
    - 完成 [Batch AI 配方讀我檔案](https://github.com/Azure/BatchAI/tree/master/recipes) \(英文\) 中的下列小節：
        - "Prerequisites"
        - "Create and get your Azure Active Directory (AAD) application"
        - "Register BatchAI Resource Providers" (在 "Run Recipes Using Azure CLI" 之下)
        - "Install Azure Batch AI Management Client"
        - "Install Azure Python SDK"
    - 記錄指導您建立之 Azure Active Directory 應用程式的用戶端識別碼、祕密和租用戶識別碼。 您稍後將會在本教學課程中使用那些認證。
    - 撰寫本文時，Azure Machine Learning Workbench 和 Azure Batch AI 會使用 Azure CLI 的不同分支。 為了清楚起見，我們將參考 CLI 的 Workbench 版本作為「從 Azure Machine Learning Workbench 啟動的 CLI」，以及一般發行版本 (包含 Batch AI) 作為「Azure CLI」。
- [AzCopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy)，可用來協調 Azure 儲存體帳戶之間檔案傳輸的免費公用程式
    - 請確定包含 AzCopy 可執行檔的資料夾位於您系統的 PATH 環境變數上。 ([這裡](https://support.microsoft.com/help/310519/how-to-manage-environment-variables-in-windows-xp)有提供如何修改環境變數的指示。)
- 針對 SSH 用戶端；我們建議 [PuTTY](http://www.putty.org/)。

此範例已在 Windows 10 電腦上進行過測試；您應該能夠從任何 Windows 機器 (包括 Azure 資料科學虛擬機器) 執行此範例。 根據[這些指示](https://github.com/Azure/azure-sdk-for-python/wiki/Contributing-to-the-tests#getting-azure-credentials)從 MSI 安裝 Azure CLI。 您可能需要稍做修改 (例如，變更 filepaths) 才能在 macOS 上執行此範例。

### <a name="set-up-azure-resources"></a>設定 Azure 資源

此範例需要 HDInsight Spark 叢集和 Azure 儲存體帳戶以便裝載相關檔案。 請遵循下列指示，在新的 Azure 資源群組中建立這些資源：

#### <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「空拍影像分類」並選取範本
5.  按一下 [建立] 
 
#### <a name="create-the-resource-group"></a>建立資源群組

1. 在 Azure Machine Learning Workbench 中載入專案後，按一下 [檔案] -> [開啟命令提示字元] 來開啟命令列介面 (CLI)。
    對於大部分的教學課程使用這個版本的 CLI。 (如有需要，您必須開啟另一版的 CLI 以準備 Batch AI 資源。)

1. 從命令列介面執行下列命令來登入您的 Azure 帳戶：

    ```
    az login
    ```

    系統會要求您瀏覽 URL 和輸入提供的暫存代碼；網站需要您的 Azure 帳戶認證。
    
1. 登入完成後，返回 CLI 並執行下列命令，判斷哪一個 Azure 訂用帳戶可供您的 Azure 帳戶使用：

    ```
    az account list
    ```

    此命令會列出所有與您 Azure 帳戶相關聯的訂用帳戶。 尋找您要使用的訂用帳戶識別碼。 在下列命令中指示的位置寫入訂用帳戶識別碼，然後藉由執行命令來設定作用中的訂用帳戶：

    ```
    az account set --subscription [subscription ID]
    ```

1. 此範例中所建立的 Azure 資源會與 Azure 資源群組儲存在一起。 請選擇唯一的資源群組名稱，並寫入指示的位置，然後執行這兩個命令來建立 Azure 資源群組：

    ```
    set AZURE_RESOURCE_GROUP=[resource group name]
    az group create --location eastus --name %AZURE_RESOURCE_GROUP%
    ```

#### <a name="create-the-storage-account"></a>建立儲存體帳戶

現在，我們會建立裝載專案檔案的儲存體帳戶，而這些專案檔案必須由 HDInsight Spark 存取。

1. 選擇唯一的儲存體帳戶名稱，並寫入下列 `set` 命令中指示的位置，然後透過執行以下兩個命令來建立 Azure 儲存體帳戶：

    ```
    set STORAGE_ACCOUNT_NAME=[storage account name]
    az storage account create --name %STORAGE_ACCOUNT_NAME% --resource-group %AZURE_RESOURCE_GROUP% --sku Standard_LRS
    ```

1. 執行下列命令來列出儲存體帳戶金鑰：

    ```
    az storage account keys list --resource-group %AZURE_RESOURCE_GROUP% --account-name %STORAGE_ACCOUNT_NAME%
    ```

    記錄 `key1` 的值，並作為下列命令中的儲存體金鑰，然後執行命令來儲存值。
    ```
    set STORAGE_ACCOUNT_KEY=[storage account key]
    ```
1. 請使用下列命令在儲存體帳戶中建立名為 `baitshare` 的檔案共用：

    ```
    az storage share create --account-name %STORAGE_ACCOUNT_NAME% --account-key %STORAGE_ACCOUNT_KEY% --name baitshare
    ```
1. 使用您慣用的文字編輯器，從 Azure Machine Learning Workbench 專案的 "Code" 子目錄中載入 `settings.cfg` 檔案，並依據指示插入儲存體帳戶名稱和金鑰。 儲存並關閉 `settings.cfg` 檔案。
1. 如果您尚未這樣做，請下載並安裝 [AzCopy](http://aka.ms/downloadazcopy) 公用程式。 您可以藉由輸入 "AzCopy" 並按下 Enter 以顯示其文件的方式，來確認 AzCopy 可執行檔存在您的系統路徑上。
1. 發佈下列命令，可將所有範例資料、預先定型的模型和模型定型指令碼複製到您儲存體帳戶中的適當位置：

    ```
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/test /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.blob.core.windows.net/test /DestKey:%STORAGE_ACCOUNT_KEY% /S
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/train /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.blob.core.windows.net/train /DestKey:%STORAGE_ACCOUNT_KEY% /S
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/middlesexma2016 /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.blob.core.windows.net/middlesexma2016 /DestKey:%STORAGE_ACCOUNT_KEY% /S
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/pretrainedmodels /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.blob.core.windows.net/pretrainedmodels /DestKey:%STORAGE_ACCOUNT_KEY% /S
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/pretrainedmodels /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.file.core.windows.net/baitshare/pretrainedmodels /DestKey:%STORAGE_ACCOUNT_KEY% /S
    AzCopy /Source:https://mawahsparktutorial.blob.core.windows.net/scripts /SourceSAS:"?sv=2017-04-17&ss=bf&srt=sco&sp=rwl&se=2037-08-25T22:02:55Z&st=2017-08-25T14:02:55Z&spr=https,http&sig=yyO6fyanu9ilAeW7TpkgbAqeTnrPR%2BpP1eh9TcpIXWw%3D" /Dest:https://%STORAGE_ACCOUNT_NAME%.file.core.windows.net/baitshare/scripts /DestKey:%STORAGE_ACCOUNT_KEY% /S
    ```

    預期的檔案傳輸時間大約一個小時。 在等候時，您可以繼續進行下一節：您可能需要透過 Workbench 開啟另一個命令列介面，並在那裡重新定義暫存變數。

#### <a name="create-the-hdinsight-spark-cluster"></a>建立 HDInsight Spark 叢集

建議您使用此專案子資料夾 "Code\01_Data_Acquisition_and_Understanding\01_HDInsight_Spark_Provisioning" 中包含的 HDInsight Spark 叢集 Resource Manager 範本，來建立 HDInsight 叢集。

1. HDInsight Spark 叢集範本是此專案子資料夾 "Code\01_Data_Acquisition_and_Understanding\01_HDInsight_Spark_Provisioning" 下的 "template.json" 檔案。 根據預設，此範本建立的 Spark 叢集包含 40 個背景工作節點。 如果您必須調整該數字，請使用慣用的文字編輯器開啟範本，並以您選取的背景工作節點數目取代任何 "40" 的執行個體。
    - 如果您選擇的背景工作節點數目較小，稍後可能會發生記憶體不足的錯誤。 若要解決記憶體錯誤，您可以在可用資料的子集上執行定型和運算化指令碼 (此文章稍後會說明)。
2. 為 HDInsight 叢集選擇唯一的名稱和密碼，並寫入下列命令中指示的位置：然後發出命令以建立叢集：

    ```
    set HDINSIGHT_CLUSTER_NAME=[HDInsight cluster name]
    set HDINSIGHT_CLUSTER_PASSWORD=[HDInsight cluster password]
    az group deployment create --resource-group %AZURE_RESOURCE_GROUP% --name hdispark --template-file "Code\01_Data_Acquisition_and_Understanding\01_HDInsight_Spark_Provisioning\template.json" --parameters storageAccountName=%STORAGE_ACCOUNT_NAME%.blob.core.windows.net storageAccountKey=%STORAGE_ACCOUNT_KEY% clusterName=%HDINSIGHT_CLUSTER_NAME% clusterLoginPassword=%HDINSIGHT_CLUSTER_PASSWORD%
    ```

部署叢集可能需要 30 分鐘 (包括佈建和指令碼動作執行)。

### <a name="set-up-batch-ai-resources"></a>設定 Batch AI 資源

在等候您的儲存體帳戶檔案傳輸及 Spark 叢集部署完成時，您可以準備 Batch AI 網路檔案伺服器 (NFS) 和 GPU 叢集。 開啟 Azure CLI 命令提示字元並執行下列命令：

```
az --version 
```

確認 `batchai` 列於已安裝的模組。 如果沒有，您可能是使用不同的命令列介面 (例如，是透過 Workbench 開啟的)。

接下來，請檢查提供者註冊已成功完成。 (提供者註冊最多需要十五分鐘的時間，且如果您最近完成[Batch AI 安裝指示](https://github.com/Azure/BatchAI/tree/master/recipes)，可能就仍在進行中。)確認 "Microsoft.Batch" 和 "Microsoft.BatchAI" 出現，且在下列命令的輸出中狀態為「已註冊」：

```
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

如果沒有，請執行下列提供者註冊命令，然後等待約 15 分鐘才能完成註冊。
```
az provider register --namespace Microsoft.Batch
az provider register --namespace Microsoft.BatchAI
```

修改下列命令，將括號運算式取代為您稍早在資源群組和儲存體帳戶建立期間所使用的值。 然後，執行下列命令將值儲存為變數：
```
az account set --subscription [subscription ID]
set AZURE_RESOURCE_GROUP=[resource group name]
set STORAGE_ACCOUNT_NAME=[storage account name]
set STORAGE_ACCOUNT_KEY=[storage account key]
az configure --defaults location=eastus
az configure --defaults group=%AZURE_RESOURCE_GROUP%
```

找出包含您 Azure Machine Learning 專案的資料夾 (例如，`C:\Users\<your username>\AzureML\aerialimageclassification`)。 將括號值取代為資料夾的檔案路徑 (使用沒有尾端有反斜線)，並執行命令：
```
set PATH_TO_PROJECT=[The filepath of your project's root directory]
```
您現在準備好建立本教學課程所需的 Batch AI 資源。

#### <a name="prepare-the-batch-ai-network-file-server"></a>準備 Batch AI 網路檔案伺服器

Batch AI 叢集會存取網路檔案伺服器上的定型資料。 相較於從 Azure 檔案共用或 Azure Blob 儲存體存取檔案，從 NFS 存取檔案時能夠更快速存取數個資料夾的資料。

1. 發出以下命令以建立網路檔案伺服器：

    ```
    az batchai file-server create -n landuseclassifier -u demoUser -p "Dem0Pa$$w0rd" --vm-size Standard_DS2_V2 --disk-count 1 --disk-size 1000 --storage-sku Premium_LRS
    ```

1. 使用下列命令來檢查您網路檔案伺服器的佈建狀態：

    ```
    az batchai file-server list
    ```

    當名為 "landuseclassifier" 的網路檔案伺服器的 "provisioningState" 為「成功」時，就準備好可供使用。 預期佈建需要大約五分鐘的時間。
1. 在前一個命令 ("mountSettings" 下的 "fileServerPublicIp" 屬性) 的輸出中尋找您 NFS 的 IP 位址。 寫入下列命令中指定的 IP 位址，然後執行命令來儲存值：

    ```
    set AZURE_BATCH_AI_TRAINING_NFS_IP=[your NFS IP address]
    ```

1. 使用您最愛的 SSH 工具 (下列範例命令會使用 [PuTTY](http://www.putty.org/))，在 NFS 上執行這個專案的 `prep_nfs.sh` 指令碼，以在該位置傳送定型和驗證映像集合。

    ```
    putty -ssh demoUser@%AZURE_BATCH_AI_TRAINING_NFS_IP% -pw Dem0Pa$$w0rd -m %PATH_TO_PROJECT%\Code\01_Data_Acquisition_and_Understanding\02_Batch_AI_Training_Provisioning\prep_nfs.sh
    ```

    如果資料下載及擷取進度更新捲動完整個殼層視窗太快速而難以辨識，請勿擔心。

如有需要，您可以確認資料傳輸已如計劃中繼續進行，方法為使用您最愛的 SSH 工具登入檔案伺服器，並檢查 `/mnt/data` 目錄內容。 您應該會發現兩個資料夾，training_images 和 validation_images，每個都會包含根據土地使用類別名的子資料夾。  定型和驗證集合應該分別包含 ~44k 和 ~11k 個映像。

#### <a name="create-a-batch-ai-cluster"></a>建立 Batch AI 帳戶

1. 發佈下列命令來建立叢集：

    ```
    az batchai cluster create -n landuseclassifier2 -u demoUser -p "Dem0Pa$$w0rd" --afs-name baitshare --nfs landuseclassifier --image UbuntuDSVM --vm-size STANDARD_NC6 --max 2 --min 2 --storage-account-name %STORAGE_ACCOUNT_NAME% 
    ```

1. 您可以使用下列命令來檢查您叢集的佈建狀態：

    ```
    az batchai cluster list
    ```

    當名為 "landuseclassifier" 的叢集之配置狀態從調整大小變更為穩定，很可能會提交作業。 不過，，在叢集中所有 VM 都離開「準備」狀態之前，作業並不會開始執行。 如果叢集的 "errors" 屬性不是 Null，在叢集建立期間就會發生錯誤，且不應加以使用。

#### <a name="record-batch-ai-training-credentials"></a>記錄 Batch AI 定型認證

當您等候叢集配置完成時，請在您所選擇的文字編輯器中，從這個專案的 "Code" 子目錄開啟 `settings.cfg` 檔案。 使用您自己的認證更新下列變數：
- `bait_subscription_id` (您 36 個字元的 Azure 訂用帳戶識別碼)
- `bait_aad_client_id` (「必要條件」一節中所述的 Azure Active Directory 應用程式/用戶端識別碼)
- `bait_aad_secret` (「必要條件」一節中所述的 Azure Active Directory 應用程式祕密)
- `bait_aad_tenant` (「必要條件」一節中所述的 Azure Active Directory 租用戶識別碼)
- `bait_region` (撰寫本文時，唯一的選項是：eastus)
- `bait_resource_group_name`(您先前選擇的資源群組)

一旦您已指派這些值後，您 settings.cfg 檔案的修改行應該會類似下列文字：

```
[Settings]
    # Credentials for the Azure Storage account
    storage_account_name = yoursaname
    storage_account_key = kpIXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXQ==
    
    # Batch AI training credentials
    bait_subscription_id = 0caXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX9c3
    bait_aad_client_id = d0aXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX7f8
    bait_aad_secret = ygSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX6I=
    bait_aad_tenant = 72fXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXb47
    bait_region = eastus
    bait_resource_group_name = yourrgname
```

儲存並關閉 `settings.cfg`。

您現在可以將執行 Batch AI 資源建立命令的 CLI 視窗關閉。 本教學課程中所有的後續步驟都會使用從 Azure Machine Learning Workbench 啟動的 CLI。

### <a name="prepare-the-azure-machine-learning-workbench-execution-environment"></a>準備 Azure Machine Learning Workbench 執行環境

#### <a name="register-the-hdinsight-cluster-as-an-azure-machine-learning-workbench-compute-target"></a>將 HDInsight 叢集註冊為 Azure Machine Learning Workbench 計算目標

完成 HDInsight 叢集建立後，請將叢集註冊為專案的計算目標，如下所示：

1.  從 Azure Machine Learning 命令列介面發佈下列命令：

    ```
    az ml computetarget attach cluster --name myhdi --address %HDINSIGHT_CLUSTER_NAME%-ssh.azurehdinsight.net --username sshuser --password %HDINSIGHT_CLUSTER_PASSWORD%
    ```

    此命令會將兩個檔案 (`myhdi.runconfig` 和 `myhdi.compute`) 新增至您專案的 `aml_config` 資料夾。

1. 在您慣用的文字編輯器中開啟 `myhdi.compute` 檔案。 修改 `yarnDeployMode: cluster` 行以讀取 `yarnDeployMode: client`，然後儲存並關閉檔案。
1. 執行下列命令，準備要使用的 HDInsight 環境：
   ```
   az ml experiment prepare -c myhdi
   ```

#### <a name="install-local-dependencies"></a>安裝本機相依項目

從 Azure Machine Learning Workbench 開啟 CLI，並藉由發佈下列命令來安裝本機執行所需的相依項目：

```
pip install matplotlib azure-storage==0.36.0 pillow scikit-learn azure-mgmt-batchai
```

## <a name="data-acquisition-and-understanding"></a>資料取得與認知

此案例使用可公開使用的[美國農業部影像計劃 (U.S. National Agriculture Imagery Program)](https://www.fsa.usda.gov/programs-and-services/aerial-photography/imagery-programs/naip-imagery/) 空拍影像資料 (解析度為 1 公尺)。 我們已產生多組從原始 NAIP 資料裁切的 224 像素 x 224 像素 PNG 檔案，並根據[美國國土覆被資料庫 (U.S. National Land Cover Database)](https://www.mrlc.gov/nlcd2011.php) 的土地用途標籤加以排序。 以完整大小顯示具有「已開發」標籤的範例影像：

![開發土地的範例圖格](media/scenario-aerial-image-classification/sample-tile-developed.png)

~44k 和 11k 影像的類別平衡組會分別用於模型定型和驗證。 我們會使用 ~67k 影像集拼湊的麻薩諸塞州密得塞斯郡 (也就是 Microsoft 的新英格蘭研發中心 (NERD) 所在地) 示範模型部署。 如需如何建構這些影像集的詳細資訊，請參閱[不易平行 (Embarrassingly Parallel) 的影像分類 Git 存放庫](https://github.com/Azure/Embarrassingly-Parallel-Image-Classification)。

![麻薩諸塞州密得塞斯郡的位置](media/scenario-aerial-image-classification/middlesex-ma.png)

在安裝期間，此範例中使用的空拍影像已傳送至您建立的儲存體帳戶。 定型、驗證和運算化影像都是 224 像素 x 224 像素的 PNG 檔案 (解析度為每平方公尺一像素)。 定型和驗證影像已根據其土地用途標籤整理至子資料夾中。 (無法得知運算化影像的土地用途標籤，且有多種模稜兩可的情況；這些影像中有些包含多個土地類型。)如需如何建構這些影像集的詳細資訊，請參閱[不易平行 (Embarrassingly Parallel) 的影像分類 Git 存放庫](https://github.com/Azure/Embarrassingly-Parallel-Image-Classification)。

檢視您 Azure 儲存體帳戶中的範例影像 (選用)：
1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 在螢幕頂端的搜尋列中，搜尋您的儲存體帳戶名稱。 按一下搜尋結果中的儲存體帳戶。
2. 按一下儲存體帳戶主窗格上的 "Blob" 連結。
3. 按一下名稱為「定型」的容器。 您應該會看見根據土地用途命名的目錄清單。
4. 按一下這些目錄的其中一個，就可載入其所包含的影像清單。
5. 按一下任何影像，即可下載並檢視影像。
6. 如有需要，按一下名為 "test" 和 "middlesexma2016" 的容器，即可檢視容器內容。

## <a name="modeling"></a>模型化

### <a name="training-models-with-azure-batch-ai"></a>使用 Azure Batch AI 的定型模型

Workbench 專案子資料夾 "Code\02_Modeling" 中的 `run_batch_ai.py` 指令碼會用來發出 Batch AI 定型作業。 這項作業會將使用者 (ImageNet 上預先定型的 AlexNet 或 ResNet 18) 選擇的映像分類 DNN 重新定型。 您也可以指定重新定型的深度：當可用的定型範例不多時，僅重新定型網路的最後一個圖層可能會降低過度學習，而當定型集合夠大時，微調整個網路 (或者，針對 AlexNet 為完全連線層級) 可能會導致更高的模型效能。

定型作業完成時，此指令碼就會將模型 (以及描述模型的整數輸出與字串標籤之間對應的檔案) 和預測儲存到 blob 儲存體。 BAIT 作業的記錄檔會進行剖析，以透過定型 Epoch 擷取錯誤率改進的時程。 錯誤率改進的時程會錯誤到 AML Workbench 的執行歷程記錄功能，以供稍後檢視。

選取您定型模型的名稱、預先定型的模型類型和定型深度。 在下列命令中指示的位置寫入您的選取項目，然後藉由從 Azure ML 命令列介面執行命令來開始重新定型：

```
az ml experiment submit -c local Code\02_Modeling\run_batch_ai.py --config_filename Code/settings.cfg --output_model_name [unique model name, alphanumeric characters only] --pretrained_model_type {alexnet,resnet18} --retraining_type {last_only,fully_connected,all} --num_epochs 10
```

預期執行 Azure Machine Learning 需要約半小時才能完成。 建議您執行一些類似的命令 (不同的輸出模型名稱、預先定型的模型類型和重新定型深度)，以便您可以將定型模型的效能與不同的方法進行比較。

### <a name="training-models-with-mmlspark"></a>MMLSpark 定型模型

Workbench 專案子資料夾 "Code\02_Modeling" 中的 `run_mmlspark.py` 指令碼會用來定型 [MMLSpark](https://github.com/Azure/mmlspark) 模型以進行影像分類。 一開始，指令碼會使用 ImageNet 資料集 (AlexNet 或 18 層 ResNet) 上預先定型的影像分類器 DNN，來為定型集影像建立特徵。 接著，指令碼會使用已建立特徵的影像定型 MMLSpark 模型 (隨機樹系或羅吉斯迴歸模型)，進而將影像分類。 然後使用已定型的模型為測試影像集建立特徵和評分。 測試集上的模型預測精確度經過計算後，會記錄至 Azure Machine Learning Workbench 的執行歷程記錄功能。 最後，經過定型的 MMLSpark 模型和其測試集上的預測會儲存到 Blob 儲存體中。

為您的定型模型選取唯一的輸出模型名稱、預先定型的模型類別和 MMLSpark 模型類別。 在下列命令範本中指示的位置寫入您的選取項目，然後藉由從 Azure ML 命令列介面執行命令來開始重新定型：

```
az ml experiment submit -c myhdi Code\02_Modeling\run_mmlspark.py --config_filename Code/settings.cfg --output_model_name [unique model name, alphanumeric characters only] --pretrained_model_type {alexnet,resnet18} --mmlspark_model_type {randomforest,logisticregression}
```

其他的 `--sample_frac` 參數可用來定型和測試具有可用資料子集的模型。 使用較小部分的範例可減少執行階段和記憶體需求，但會犧牲定型模型的精確度。 (例如，使用 `--sample_frac 0.1` 執行應該需要大約二十分鐘的時間。)如需此參數和其他參數的詳細資訊，請執行 `python Code\02_Modeling\run_mmlspark.py -h`。

我們建議使用者以不同的輸入參數執行此指令碼多次。 然後，產出模型的效能可透過 Azure Machine Learning Workbench 的執行歷程記錄功能進行比較。

### <a name="comparing-model-performance-using-the-workbench-run-history-feature"></a>使用 Workbench 執行歷程記錄功能比較模型效能

執行兩個或更多任一類型的定型執行後，您可以按一下功能表列左側上的時鐘圖示，瀏覽至 Workbench 中的執行歷程記錄功能。 從左側的指令碼清單中選取 `run_mmlspark.py`。 窗格會隨即載入，並比較所有執行的測試組精確度。 若要查看更多詳細資料，向下捲動並按一下個別執行的名稱。

## <a name="deployment"></a>部署

若要在 HDInsight 上使用遠端執行，來將其中一個定型模型套用至空拍影像拼湊的麻薩諸塞州密得塞斯郡，請將您需要的模型名稱插入下列命令並執行：

```
az ml experiment submit -c myhdi Code\03_Deployment\batch_score_spark.py --config_filename Code/settings.cfg --output_model_name [trained model name chosen earlier]
```

其他的 `--sample_frac` 參數可用來運算化具有可用資料子集的模型。 使用較小部分的範例可減少執行階段和記憶體需求，但會犧牲預測的完整性。 如需此參數和其他參數的詳細資訊，請執行 `python Code\03_Deployment\batch_score_spark -h`。

此指令碼會將模型的預測寫入儲存體帳戶。 預測可在下一節的內容中進行檢查。

## <a name="visualization"></a>視覺效果

Workbench 專案子資料夾 "Code\04_Result_Analysis" 中的「模型預測分析」Jupyter 筆記本可將模型預測視覺化。 載入並執行筆記本，如下所示：
1. 在 Workbench 中開啟專案，按一下左側功能表的資料夾 (「檔案」) 圖示，來載入目錄清單。
2. 瀏覽至 "Code\04_Result_Analysis" 子資料夾，並按一下名為「模型預測分析」的筆記本。 筆記本的預覽轉譯應該會隨即顯示。
3. 按一下 [啟動筆記本伺服器] 來載入筆記本。
4. 在第一個資料格中，依指示輸入您想要分析其結果的模型名稱。
5. 按一下 [資料格] -> [全部執行]，在筆記本中執行所有資料格。
6. 閱讀筆記本，深入了解筆記本呈現的分析和視覺效果。

## <a name="cleanup"></a>清除
當您完成範例時，我們建議您從 Azure 命令列介面執行下列命令，來刪除所有已建立的資源：

  ```
  az group delete --name %AZURE_RESOURCE_GROUP%
  ```

## <a name="references"></a>參考

- [不易平行 (Embarrassingly Parallel) 的影像分類存放庫](https://github.com/Azure/Embarrassingly-Parallel-Image-Classification)
   - 描述免費影像和標籤的資料集建構
- [MMLSpark](https://github.com/Azure/mmlspark) GitHub 存放庫
   - 包含其他 MMLSpark 模型定型和評估的範例

## <a name="conclusions"></a>結論

Azure Machine Learning Workbenc 可協助資料科學家輕鬆地在遠端計算目標上部署程式碼。 在此範例中，部署了本機 MMLSpark 定型程式碼以在 HDInsight 叢集上進行遠端執行，且本機指令碼在 Azure Batch AI GPU 叢集上啟動了定型作業。 Azure Machine Learning Workbench 的執行歷程記錄功能可追蹤多個模型的效能，並協助我們識別最精確的模型。 Workbench 的 Jupyter 筆記本功能可協助我們在互動式和圖形化的環境中，將模型預測視覺化。

## <a name="next-steps"></a>後續步驟
深入了解此範例：
- 在 Azure Machine Learning Workbench 的執行歷程記錄功能中，按一下齒輪符號，可選取要顯示哪些圖表和計量。
- 呼叫 `run_logger` 可檢查陳述式的範例指令碼。 確認您了解每個計量是如何記錄的。
- 呼叫 `blob_service` 可檢查陳述式的範例指令碼。 確認您了解定型模型和預測如何進行儲存，以及如何從雲端擷取。
- 瀏覽 Blob 儲存體帳戶中建立的容器內容。 確認您了解哪種指令碼或命令會負責建立每個檔案群組。
- 修改定型指令碼可定型不同 MMLSpark 模型類型，或變更模型的超參數。 使用執行歷程記錄功能可判斷您的變更是否增加或減少模型的精確度。
