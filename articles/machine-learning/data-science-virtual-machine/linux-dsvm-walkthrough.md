---
title: 在 Azure 上搭配 Linux 資料科學虛擬機器來運用資料科學 | Microsoft Docs
description: 如何使用 Linux 資料科學 VM 執行數個常見的資料科學工作。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
editor: cgronlun
ms.assetid: 34ef0b10-9270-474f-8800-eecb183bbce4
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/16/2018
ms.author: gokuma
ms.openlocfilehash: d9b89329e2a9bdb26c9aa1d12bc181c61518dcb8
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39116158"
---
# <a name="data-science-with-a-linux-data-science-virtual-machine-on-azure"></a>在 Azure 上搭配 Linux 資料科學虛擬機器來運用資料科學
本逐步解說示範如何使用 Linux 資料科學 VM 執行數個常見的資料科學工作。 Linux 資料科學虛擬機器 (DSVM) 是 Azure 提供的虛擬機器映像，其中預先安裝了一組常用於執行資料分析和機器學習服務的工具。 重要的軟體元件可在 [佈建 Linux 資料科學虛擬機器](linux-dsvm-intro.md) 主題中找到明細。 VM 映像可讓使用者輕鬆地在幾分鐘內開始執行資料科學，而不需要個別安裝和設定每個工具。 您可以在需要時輕鬆地相應增加 VM，並在不使用時加以停止。 因此，這項資源既有彈性，又符合成本效益。

本逐步解說所示範的資料科學工作遵循 [Team Data Science Process](https://azure.microsoft.com/documentation/learning-paths/data-science-process/)所述的步驟。 此程序可讓使用者以系統化方法執行資料科學，讓資料科學家團隊可以在智慧型應用程式的建置生命週期內有效地共同作業。 資料科學程序也為資料科學提供了可反覆進行的架構供個人遵循。

在本逐步解說中，我們會分析 [spambase](https://archive.ics.uci.edu/ml/datasets/spambase) 資料集。 這是一組標示為垃圾郵件或非垃圾郵件 (亦即這些郵件不是垃圾郵件) 的電子郵件，並同時包含關於電子郵件內容的一些統計資料。 其中所含的統計資料會在下下一節中討論。

## <a name="prerequisites"></a>必要條件
您必須先具有下列項目，才可以使用 Linux 資料科學虛擬機器：

* **Azure 訂用帳戶**。 如果您還沒有訂用帳戶，請參閱 [立即建立免費的 Azure 帳戶](https://azure.microsoft.com/free/)。
* [**Linux 資料科學 VM**](https://azure.microsoft.com/marketplace/partners/microsoft-ads/linux-data-science-vm)。 如需佈建此 VM 的相關資訊，請參閱 [佈建 Linux 資料科學虛擬機器](linux-dsvm-intro.md)。
* [X2Go](http://wiki.x2go.org/doku.php) 已安裝在電腦上並已開啟 XFCE 工作階段。 如需安裝和設定 **X2Go 用戶端**的相關資訊，請參閱[安裝和設定 X2Go 用戶端](linux-dsvm-intro.md#installing-and-configuring-x2go-client)。
* 若要享受更平滑的捲動體驗，請切換 VM FireFox 瀏覽器的 about:config 中的 gfx.xrender.enabled 旗標。 [請參閱這裡深入了解](https://www.reddit.com/r/firefox/comments/4nfmvp/ff_47_unbearable_slow_over_remote_x11/)。 另請考慮將 *mousewheel.enable_pixel_scrolling* 切換為 False。 [請參閱這裡的指示。](https://support.mozilla.org/en-US/questions/981140)
* **AzureML 帳戶**。 如果您還沒有帳戶，請在 [AzureML 首頁](https://studio.azureml.net/)註冊新帳戶。 裡面有免費的使用量層級可幫助您開始使用。

## <a name="download-the-spambase-dataset"></a>下載 spambase 資料集
[spambase](https://archive.ics.uci.edu/ml/datasets/spambase) 資料集是一組較小的資料，裡面只有 4601 個範例。 在示範資料科學 VM 的某些重要功能時，這樣的大小比較方便使用，因為它會讓所需的資源需求保持適中。

> [!NOTE]
> 本逐步解說是以 D2 v2 大小的 Linux 資料科學虛擬機器 (CentOS 版) 為基礎。 這個大小的 DSVM 能夠處理此逐步解說中的程序。
>
>

如果您需要更多儲存空間，您可以建立額外的磁碟，並將它們連接到 VM。 這些磁碟會使用永續性的 Azure 儲存體，因此，即使伺服器因為調整大小或關閉等緣故而重新佈建，磁碟中的資料仍會保留下來。 若要新增磁碟並將它連接到 VM，請遵循[在 Linux VM 中新增磁碟](../../virtual-machines/linux/add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)的指示。 這些步驟使用 Azure 命令列介面 (Azure CLI)，此介面已安裝在 DSVM 上。 因此，您完全可以從 VM 本身來執行這些程序。 另一種可增加儲存體的選項是使用 [Azure 檔案](../../storage/files/storage-how-to-use-files-linux.md)。

若要下載資料，請開啟終端機視窗並執行此命令︰

    wget http://archive.ics.uci.edu/ml/machine-learning-databases/spambase/spambase.data

下載的檔案沒有標題列，因此，讓我們建立另一個有標題的檔案。 執行此命令來建立具有適當標題的檔案︰

    echo 'word_freq_make, word_freq_address, word_freq_all, word_freq_3d,word_freq_our, word_freq_over, word_freq_remove, word_freq_internet,word_freq_order, word_freq_mail, word_freq_receive, word_freq_will,word_freq_people, word_freq_report, word_freq_addresses, word_freq_free,word_freq_business, word_freq_email, word_freq_you, word_freq_credit,word_freq_your, word_freq_font, word_freq_000, word_freq_money,word_freq_hp, word_freq_hpl, word_freq_george, word_freq_650, word_freq_lab,word_freq_labs, word_freq_telnet, word_freq_857, word_freq_data,word_freq_415, word_freq_85, word_freq_technology, word_freq_1999,word_freq_parts, word_freq_pm, word_freq_direct, word_freq_cs, word_freq_meeting,word_freq_original, word_freq_project, word_freq_re, word_freq_edu,word_freq_table, word_freq_conference, char_freq_semicolon, char_freq_leftParen,char_freq_leftBracket, char_freq_exclamation, char_freq_dollar, char_freq_pound, capital_run_length_average,capital_run_length_longest, capital_run_length_total, spam' > headers

然後，使用下列命令串連兩個檔案︰

    cat spambase.data >> headers
    mv headers spambaseHeaders.data

資料集內有多種關於每封電子郵件的統計資料︰

* ***word\_freq\_WORD*** 之類的資料行會指出電子郵件中符合 *WORD* 之字詞的百分比。 例如，如果 *word\_freq\_make* 是 1，則表示電子郵件的所有文字中有 1% 是 *make*。
* ***char\_freq\_CHAR*** 之類的資料行會指出電子郵件的所有字元中 *CHAR* 所佔的百分比。
* ***capital\_run\_length\_longest*** 是一連串大寫字母的最長長度。
* ***capital\_run\_length\_average*** 是所有連串大寫字母的平均長度。
* ***capital\_run\_length\_total*** 是所有連串大寫字母的總長度。
*  indicates whether the email was considered  or not (1 = , 0 = not ).

## <a name="explore-the-dataset-with-microsoft-r-open"></a>使用 Microsoft R Open 探索資料集
讓我們使用 R 來檢查資料並執行某些基本的機器學習服務。資料科學 VM 已預先安裝了 [Microsoft R Open](https://mran.revolutionanalytics.com/open/)。 此版本的 R 中有多執行緒的數學程式庫，可提供比單一執行緒版本還要好的效能。 Microsoft R Open 也可藉由使用 CRAN 封裝儲存機制的快照來提供重現性。

若要取得此逐步解說所使用的程式碼範例複本，請使用 VM 上預先安裝的 git 複製 **Azure-Machine-Learning-Data-Science** 儲存機制。 從 git 命令列執行︰

    git clone https://github.com/Azure/Azure-MachineLearning-DataScience.git

開啟終端機視窗並使用 R 互動主控台或電腦上預先安裝的 RStudio 啟動新的 R 工作階段。


若要匯入資料並設定環境，請執行︰

    data <- read.csv("spambaseHeaders.data")
    set.seed(123)

若要查看關於每個資料行的摘要統計資料︰

    summary(data)

針對資料的不同檢視︰

    str(data)

這會顯示每個變數的類型和資料集內的前幾個值。

「spam」  資料行已讀取為整數，但它實際上是類別變數 (或係數)。 若要設定其類型︰

    data$spam <- as.factor(data$spam)

若要進行一些探勘分析，請使用 [ggplot2](http://ggplot2.org/) 封裝，這是已安裝在 VM 上的適用於 R 的熱門圖形庫。 請注意，在稍早顯示的摘要資料中，我們擁有關於驚嘆號字元出現頻率的摘要統計資料。 在此，讓我們使用下列命令繪製這些頻率︰

    library(ggplot2)
    ggplot(data) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)

由於零軸會影響繪圖的準確性，讓我們將它去除︰

    email_with_exclamation = data[data$char_freq_exclamation > 0, ]
    ggplot(email_with_exclamation) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)

在 1 上面有看起來很有意思的不尋常密度。 讓我們只看該資料︰

    ggplot(data[data$char_freq_exclamation > 1, ]) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)

然後按照垃圾郵件和非垃圾郵件進行分割：

    ggplot(data[data$char_freq_exclamation > 1, ], aes(x=char_freq_exclamation)) +
    geom_density(lty=3) +
    geom_density(aes(fill=spam, colour=spam), alpha=0.55) +
    xlab("spam") +
    ggtitle("Distribution of spam \nby frequency of !") +
    labs(fill="spam", y="Density")

這些範例應該能讓您為其他資料行製作類似繪圖，以探索它們內含的資料。

## <a name="train-and-test-an-ml-model"></a>訓練和測試 ML 模型
現在讓我們訓練幾個機器學習服務模型，將資料集內的電子郵件分類為包含垃圾郵件或非垃圾郵件。 在本節中，我們會訓練決策樹模型和隨機樹系模型，然後測試其預測的精確度。

> [!NOTE]
> 下列程式碼所使用的 RPART (Recursive Partitioning and Regression Trees，遞迴分割和迴歸樹狀結構) 封裝已安裝在資料科學 VM 上。
>
>

首先，讓我們將資料集分割為訓練集和測試集︰

    rnd <- runif(dim(data)[1])
    trainSet = subset(data, rnd <= 0.7)
    testSet = subset(data, rnd > 0.7)

然後建立決策樹來分類電子郵件。

    require(rpart)
    model.rpart <- rpart(spam ~ ., method = "class", data = trainSet)
    plot(model.rpart)
    text(model.rpart)

結果如下︰

![1](./media/linux-dsvm-walkthrough/decision-tree.png)

若要判斷訓練集的表現有多良好，請使用下列程式碼︰

    trainSetPred <- predict(model.rpart, newdata = trainSet, type = "class")
    t <- table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)
    accuracy <- sum(diag(t))/sum(t)
    accuracy

若要判斷測試集的表現有多良好︰

    testSetPred <- predict(model.rpart, newdata = testSet, type = "class")
    t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
    accuracy <- sum(diag(t))/sum(t)
    accuracy

讓我們同時嘗試隨機樹系模型。 隨機樹系會訓練大量決策樹，並輸出屬於所有個別決策樹之分類眾數的類別。 它們能提供更強大的機器學習服務方法，因為它們會校正決策樹模型傾向以過度擬合訓練資料集。

    require(randomForest)
    trainVars <- setdiff(colnames(data), 'spam')
    model.rf <- randomForest(x=trainSet[, trainVars], y=trainSet$spam)

    trainSetPred <- predict(model.rf, newdata = trainSet[, trainVars], type = "class")
    table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)

    testSetPred <- predict(model.rf, newdata = testSet[, trainVars], type = "class")
    t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
    accuracy <- sum(diag(t))/sum(t)
    accuracy


## <a name="deploy-a-model-to-azure-ml"></a>模型部署到 Azure ML
[Azure Machine Learning Studio](https://studio.azureml.net/) (AzureML) 是一項雲端服務，可讓您輕鬆地建置和部署預測性分析模型。 AzureML 的其中一項優秀功能是能夠將任何 R 函數發佈為 Web 服務。 AzureML R 封裝可直接從 DSVM 上的 R 工作階段讓部署作業的執行變得簡單無比。

若要部署上一節的決策樹程式碼，您需要登入 Azure Machine Learning Studio。 您需要工作區識別碼和驗證權杖才能登入。 若要找到這些值並以值初始化 AzureML 變數︰

選取左側功能表上的 [設定]。 記下您的**工作區識別碼**。 ![2](./media/linux-dsvm-walkthrough/workspace-id.png)

從上方的功能表選取 [授權權杖] 並記下您的**主要授權權杖**。![3](./media/linux-dsvm-walkthrough/workspace-token.png)

載入 **AzureML** 封裝，然後在 DSVM 的 R 工作階段中以您的權杖和工作區識別碼設定變數值：

    if(!require("AzureML")) install.packages("AzureML")
    require(AzureML)
    wsAuth = "<authorization-token>"
    wsID = "<workspace-id>"


讓我們簡化模型，以使這項示範更容易實作。 挑選決策樹中最接近根部的三個變數，並只用這三個變數建置新的決策樹︰

    colNames <- c("char_freq_dollar", "word_freq_remove", "word_freq_hp", "spam")
    smallTrainSet <- trainSet[, colNames]
    smallTestSet <- testSet[, colNames]
    model.rpart <- rpart(spam ~ ., method = "class", data = smallTrainSet)

我們需要會以功能做為輸入並傳回預測值的預測函數︰

    predictSpam <- function(newdata) {
      predictDF <- predict(model.rpart, newdata = newdata)
      return(colnames(predictDF)[apply(predictDF, 1, which.max)])
    }


使用 **publishWebService** 函數將 predictSpam 函數發佈至 AzureML︰

    spamWebService <- publishWebService(ws, fun = predictSpam, name="spamWebService", inputSchema = smallTrainSet, data.frame=TRUE)


此函數會採用 **predictSpam** 函數、建立名為 **spamWebService** 的 Web 服務以及定義的輸入和輸出，並傳回新端點的相關資訊。

使用下列命令檢視最新發佈之 Web 服務的詳細資料，包括其 API 端點和存取金鑰︰

    s<-tail(services(ws, name = "spamWebService"), 1)
    ep <- endpoints(ws,s)
    ep

若要對前 10 列測試集試用此服務︰

    consume(ep, smallTestSet[1:10, ])


## <a name="use-other-tools-available"></a>使用其他可用工具
其餘各節示範如何使用一些已安裝在 Linux 資料科學 VM 上的工具。以下是所討論的工具清單︰

* XGBoost
* Python
* Jupyterhub
* Rattle
* PostgreSQL 和 Squirrel SQL
* SQL Server 資料倉儲

## <a name="xgboost"></a>XGBoost
[XGBoost](https://xgboost.readthedocs.org/en/latest/) 工具可提供快速且正確的推進式決策樹實作。

    require(xgboost)
    data <- read.csv("spambaseHeaders.data")
    set.seed(123)

    rnd <- runif(dim(data)[1])
    trainSet = subset(data, rnd <= 0.7)
    testSet = subset(data, rnd > 0.7)

    bst <- xgboost(data = data.matrix(trainSet[,0:57]), label = trainSet$spam, nthread = 2, nrounds = 2, objective = "binary:logistic")

    pred <- predict(bst, data.matrix(testSet[, 0:57]))
    accuracy <- 1.0 - mean(as.numeric(pred > 0.5) != testSet$spam)
    print(paste("test accuracy = ", accuracy))

XGBoost 也可以從 Python 或命令列進行呼叫。

## <a name="python"></a>Python
為了能夠使用 Python 進行開發，Anaconda Python 散發套件 2.7 與 3.5 已安裝在 DSVM 中。

> [!NOTE]
> Anaconda 散發套件包含 [Conda](http://conda.pydata.org/docs/index.html)，可用來為 Python 建立已安裝不同版本和 (或) 封裝的自訂環境。
>
>

讓我們讀入某些 spambase 資料集，並以 scikit-learn 中的支援向量機器分類電子郵件︰

    import pandas
    from sklearn import svm    
    data = pandas.read_csv("spambaseHeaders.data", sep = ',\s*')
    X = data.ix[:, 0:57]
    y = data.ix[:, 57]
    clf = svm.SVC()
    clf.fit(X, y)

若要進行預測︰

    clf.predict(X.ix[0:20, :])

若要顯示如何發佈 AzureML 端點，讓我們和先前發佈 R 模型時一樣，建立只有三個變數的簡化模型。

    X = data[["char_freq_dollar", "word_freq_remove", "word_freq_hp"]]
    y = data.ix[:, 57]
    clf = svm.SVC()
    clf.fit(X, y)

若要將模型發佈至 AzureML：

    # Publish the model.
    workspace_id = "<workspace-id>"
    workspace_token = "<workspace-token>"
    from azureml import services
    @services.publish(workspace_id, workspace_token)
    @services.types(char_freq_dollar = float, word_freq_remove = float, word_freq_hp = float)
    @services.returns(int) # 0 or 1
    def predictSpam(char_freq_dollar, word_freq_remove, word_freq_hp):
        inputArray = [char_freq_dollar, word_freq_remove, word_freq_hp]
        return clf.predict(inputArray)

    # Get some info about the resulting model.
    predictSpam.service.url
    predictSpam.service.api_key

    # Call the model
    predictSpam.service(1, 1, 1)

> [!NOTE]
> 此操作只適用於 Python 2.7，3.5 版則尚未支援。 請使用 **/anaconda/bin/python2.7**來執行。
>
>

## <a name="jupyterhub"></a>Jupyterhub
DSVM 中的 Anaconda 散發套件隨附 Jupyter Notebook，此跨平台環境可用來共用 Python、R 或 Julia 程式碼和分析。 Jupyter 筆記本是透過 JupyterHub 來存取。 您可以在 ***https://\<VM DNS 名稱或 IP 位址\>:8000/*** 使用本機 Linux 使用者名稱和密碼來登入。 JupyterHub 的所有組態檔可在 **eg /etc/ jupyterhub**目錄中找到。

> [!NOTE]
> 若要從目前核心中的 Jupyter 筆記本使用 Python 套件管理員 (透過 `pip` 命令)，則可在程式碼單元中使用下列命令，例如：
```python
   import sys
   ! {sys.executable} -m pip install numpy -y
```
>
>

> [!NOTE]
> 若要從目前核心中的 Jupyter 筆記本使用 Conda 安裝程式 (透過 `conda` 命令)，則可在程式碼單元中使用下列命令，例如：
```python
   import sys
   ! {sys.prefix}/bin/conda install --yes --prefix {sys.prefix} numpy
```
>
>

VM 上已安裝數個 Notebook 範例︰

* 如需 Python 的 Notebook 範例，請參閱 [IntroToJupyterPython.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroToJupyterPython.ipynb) 。
* 如需 [R](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroTutorialinR.ipynb) 的 Notebook 範例，請參閱 **IntroTutorialinR** 。
* 如需 [Python](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IrisClassifierPyMLWebService.ipynb) 的其他 Notebook 範例，請參閱 **IrisClassifierPyMLWebService** 。

> [!NOTE]
> Julia 語言也可從 Linux 資料科學 VM 上的命令列來使用。
>
>

## <a name="rattle"></a>Rattle
[Rattle](https://cran.r-project.org/web/packages/rattle/index.html) (R Analytical Tool To Learn Easily) 是用於資料採礦的 R 圖形化工具。 其直覺式介面可讓您輕鬆地載入、瀏覽和轉換資料以及建置和評估模型。  [Rattle︰R 的資料採礦 GUI](https://journal.r-project.org/archive/2009-2/RJournal_2009-2_Williams.pdf) 一文提供了逐步解說來示範其功能。

使用下列命令安裝並啟動 Rattle︰

    if(!require("rattle")) install.packages("rattle")
    require(rattle)
    rattle()

> [!NOTE]
> 不需要安裝在 DSVM 上。 但 Rattle 可能會在載入時提示您安裝其他封裝。
>
>

Rattle 使用索引標籤式介面。 大部分索引標籤會對應至 [資料科學程序](https://azure.microsoft.com/documentation/learning-paths/data-science-process/)中的步驟，例如載入資料或瀏覽資料。 資料科學程序會由左到右經歷所有索引標籤。 但最後一個索引標籤包含 Rattle 所執行的 R 命令的記錄檔。

若要載入和設定資料集︰

* 若要載入檔案，請選取 [資料]  索引標籤，然後
* 選擇 **Filename** 旁的選取器，然後選擇 **spambaseHeaders.data**。
* 若要載入檔案。 選取最上方按鈕列中的 [執行]。 您應該會看到每個資料行的摘要，包括其識別的資料類型、其為輸入、目標還是其他類型的變數，以及唯一值數目。
* Rattle 已將 [垃圾郵件]  資料行正確地識別為目標。 選取 [垃圾郵件] 資料行，然後將 [目標資料類型] 設定為 [類別]。

若要瀏覽資料︰

* 選取 [瀏覽]  索引標籤。
* 依序按一下 [摘要] 和 [執行]，以查看一些關於變數類型的資訊和某些摘要統計資料。
* 若要檢視關於每個變數的其他類型的統計資料，請選取其他選項，例如 [描述] 或 [基本資訊]。

[瀏覽]  索引標籤也可讓您產生許多具洞察力的繪圖。 若要繪製資料的長條圖︰

* 選取 [分佈] 。
* 為 **word_freq_remove** 和 **word_freq_you** 勾選 [長條圖]。
* 選取 [執行] 。 您應該會在單一圖形視窗中看到這兩個密度圖，其中清楚顯示「you」這個字在電子郵件中的出現頻率遠高於「remove」。

相互關聯圖也很有趣。 若要建立此圖：

* 選擇 [相互關聯] 做為 [類型]，然後
* 選取 [執行] 。
* Rattle 會警告您，它建議的上限為 40 個變數。 選取 [是]  以檢視此圖。

圖中會浮現一些有趣的相互關聯：例如，「technology」與「HP」和「labs」有高度相互關聯性。 它也與「650」有高度相互關聯性，因為資料集捐贈者的區碼是 650。

文字間相互關聯性的數值可在 [瀏覽] 視窗中取得。 舉例來說，值得注意的是「technology」與「your」和「money」負面相關。

Rattle 可以轉換資料集來處理一些常見的問題。 例如，它可讓您調整功能大小、插補遺漏值、處理離群值，以及移除具有遺失資料的變數或觀察值。 Rattle 也可以識別觀察值和 (或) 變數之間的關聯規則。 這些索引標籤不在此入門逐步解說的討論範圍內。

Rattle 也可以執行叢集分析。 讓我們排除部分功能以讓輸出更方便閱讀。 在 [資料] 索引標籤上，選擇每個變數旁的 [忽略]，但下面這十個項目除外︰

* word_freq_hp
* word_freq_technology
* word_freq_george
* word_freq_remove
* word_freq_your
* word_freq_dollar
* word_freq_money
* capital_run_length_longest
* word_freq_business
* spam

然後返回 [叢集] 索引標籤，選擇 [KMeans]，並將 [叢集數目] 設定為 4。 然後**執行**。 結果會顯示在輸出視窗中。 有一個叢集具有高頻率的「george」和「hp」，因此可能是合法的商業電子郵件。

若要建置簡單的決策樹機器學習服務模型︰

* 選取 [模型]  索引標籤。
* 選擇 [樹狀結構] 做為 [類型]。
* 選取 [執行]  ，在輸出視窗中以文字形式顯示樹狀結構。
* 選取 [繪製]  按鈕以檢視圖形化版本。 此版本看起來非常類似我們稍早使用「rpart」 取得的樹狀結構。

Rattle 的其中一項優秀功能是能夠執行數個機器學習服務方法和快速評估這些方法。 程序如下：

* 選擇 [全部] 做為 [類型]。
* 選取 [執行] 。
* 執行完畢後，您可以按一下任何單一 [類型] \(例如 **SVM**) 並檢視結果。
* 您也可以使用 [評估]  索引標籤比較驗證集上模型的效能。例如，[錯誤矩陣]  選取項目會顯示驗證集上每個模型的混淆矩陣、整體錯誤和平均類別錯誤。
* 您也可以繪製 ROC 曲線、執行敏感度分析和進行其他類型的模型評估。

建置完模型之後，選取 [記錄]  索引標籤來檢視 Rattle 在工作階段期間執行的 R 程式碼。 您可以選取 [匯出]  按鈕來加以儲存。

> [!NOTE]
> 最新版 Rattle 中有一個錯誤。 若要修改指令碼或使用它在稍後重複執行步驟，您必須在記錄文字的 *Export this log ... * 前面插入 # 字元。
>
>

## <a name="postgresql--squirrel-sql"></a>PostgreSQL 和 Squirrel SQL
DSVM 隨附安裝 PostgreSQL。 PostgreSQL 是複雜的開放原始碼關聯式資料庫。 本節說明如何將垃圾郵件資料集載入至 PostgreSQL，然後進行查詢。

在載入資料之前，您必須先允許從 localhost 進行密碼驗證。 在命令提示字元中︰

    sudo gedit /var/lib/pgsql/data/pg_hba.conf

組態檔末尾附近有幾行詳細說明允許之連線的文字︰

    # "local" is for Unix domain socket connections only
    local   all             all                                     trust
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            ident
    # IPv6 local connections:
    host    all             all             ::1/128                 ident

將「IPv4 local connections」文字行變更為使用 md5 而非 ident，以便可以使用使用者名稱和密碼來登入︰

    # IPv4 local connections:
    host    all             all             127.0.0.1/32            md5

然後重新啟動 postgres 服務︰

    sudo systemctl restart postgresql

若要啟動 psql (PostgreSQL 的互動終端機)，請以內建 postgres 使用者身分從命令提示字元執行下列命令︰

    sudo -u postgres psql

使用和您目前用來登入之 Linux 帳戶相同的使用者名稱建立新的使用者帳戶，並為它指定密碼︰

    CREATE USER <username> WITH CREATEDB;
    CREATE DATABASE <username>;
    ALTER USER <username> password '<password>';
    \quit

然後，以使用者身分登入 psql︰

    psql

並將資料匯入新的資料庫︰

    CREATE DATABASE spam;
    \c spam
    CREATE TABLE data (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer);
    \copy data FROM /home/<username>/spambase.data DELIMITER ',' CSV;
    \quit

現在，讓我們使用 **Squirrel SQL**來瀏覽資料並執行一些查詢，此圖形化工具可讓您透過 JDBC 驅動程式與資料庫互動。

若要開始使用，請從 [應用程式] 功能表啟動 Squirrel SQL。 若要設定驅動程式︰

* 依序選取 [Windows] 和 [檢視驅動程式]。
* 以滑鼠右鍵按一下 [PostgreSQL]，然後選取 [修改驅動程式]。
* 依序選取 [額外類別路徑] 和 [新增]。
* 輸入 ***/usr/share/java/jdbcdrivers/postgresql-9.4.1208.jre6.jar*** 做為 [檔案名稱]。
* 選取 [開啟] 。
* 選擇 [列出驅動程式]，接著在 [類別名稱] 中選取 [org.postgresql.Driver]，然後選取 [確定]。

若要設定與本機伺服器的連線︰

* 依序選取 [Windows] 和 [檢視別名]。
* 選擇 [+] **+** 按鈕建立新的別名。
* 將其命名為*垃圾郵件資料庫*，然後選擇 [驅動程式] 下拉式清單中的 [PostgreSQL]。
* 將 URL 設定為 *jdbc:postgresql://localhost/spam*。
* 輸入您的*使用者名稱*和*密碼*。
* 按一下 [確定]。
* 若要開啟 [連線] 視窗，請按兩下***垃圾郵件資料庫***別名。
* 選取 [ **連接**]。

若要執行一些查詢︰

* 選取 [SQL]  索引標籤。
* 在 [SQL] 索引標籤頂端的查詢文字方塊中輸入簡單的查詢，例如 `SELECT * from data;` 。
* 按 **Ctrl-Enter** 來加以執行。 依預設，Squirrel SQL 會傳回查詢的前 100 個資料列。

還有許多可供您執行以瀏覽此資料的查詢。 例如，「make」  一字在垃圾郵件和非垃圾郵件之間的出現頻率有何差異？

    SELECT avg(word_freq_make), spam from data group by spam;

或者，經常包含「3d」 的電子郵件有何特性？

    SELECT * from data order by word_freq_3d desc;

大部分頻繁出現「3d」  的電子郵件顯然是垃圾郵件，因此是很適合用來建置預測性模型以分類電子郵件的特徵。

如果您想要對 PostgreSQL 資料庫中儲存的資料執行機器學習服務，請考慮使用 [MADlib](http://madlib.incubator.apache.org/)。

## <a name="sql-server-data-warehouse"></a>SQL Server 資料倉儲
Azure SQL 資料倉儲是一種雲端架構、相應放大的資料庫，可處理大量的關聯式與非關聯式資料。 如需詳細資訊，請參閱 [什麼是 Azure SQL 資料倉儲？](../../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

若要連線到資料倉儲並建立資料表，請從命令提示字元執行下列命令︰

    sqlcmd -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -I

然後，在 sqlcmd 提示字元中︰

    CREATE TABLE spam (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer) WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
    GO

使用 bcp 複製資料︰

    bcp spam in spambaseHeaders.data -q -c -t  ',' -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -F 1 -r "\r\n"

> [!NOTE]
> 所下載檔案中的行尾結束符號為 Windows 樣式，但 bcp 需要 UNIX 樣式，因此我們必須使用 -r 旗標將這一點告訴 bcp。
>
>

並使用 sqlcmd 進行查詢︰

    select top 10 spam, char_freq_dollar from spam;
    GO

您也可以使用 Squirrel SQL 進行查詢。 使用 Microsoft MSSQL Server JDBC 驅動程式 (可在 ***/usr/share/java/jdbcdrivers/sqljdbc42.jar*** 中找到) 按照適用於 PostgreSQL 的類似步驟來進行。

## <a name="next-steps"></a>後續步驟
如需能引導您完成在 Azure 中構成資料科學程序之工作的主題概觀，請參閱 [Team Data Science Process](http://aka.ms/datascienceprocess)。

如需會示範 Team Data Science Process 中適用於特定案例之步驟的其他端對端逐步解說的說明，請參閱 [Team Data Science Process 逐步解說](../team-data-science-process/walkthroughs.md)。 這些逐步解說也示範如何將雲端和內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。
