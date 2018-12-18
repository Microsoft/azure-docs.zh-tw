---
title: 在 Azure 上佈建 Linux (Ubuntu) 資料科學虛擬機器 | Microsoft Docs
description: 在 Azure 上設定和建立 Linux (Ubuntu) 適用的資料科學虛擬機器以進行分析和機器學習服務。
services: machine-learning
documentationcenter: ''
author: gopitk
ms.author: gokuma
manager: cgronlun
ms.assetid: 3bab0ab9-3ea5-41a6-a62a-8c44fdbae43b
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.openlocfilehash: d9008956d3a5542d71438ee13050a3951230e101
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338802"
---
# <a name="provision-the-data-science-virtual-machine-for-linux-ubuntu"></a>佈建適用於 Linux (Ubuntu) 的資料科學虛擬機器

適用於 Linux 的資料科學虛擬機器是以 Ubuntu 為基礎的虛擬機器映像，可幫助您在 Azure 上輕鬆展開機器學習，包括深度學習。 深入學習工具包含：

  * [Caffe](http://caffe.berkeleyvision.org/)︰一種深入學習架構，講求速度、表現度及模組化
  * [Caffe2](https://github.com/caffe2/caffe2)：Caffe 的跨平台版本
  * [Microsoft 辨識工具組](https://github.com/Microsoft/CNTK) \(英文\)︰來自 Microsoft Research 的深度學習軟體工具組
  * [H2O](https://www.h2o.ai/)開放原始碼巨量資料平台和圖形化使用者介面
  * [Keras](https://keras.io/)：以 Python 撰寫且適用於 Theano 和 TensorFlow 的高層級類神經網路 API
  * [MXNet](http://mxnet.io/)：彈性、有效率的深入學習程式庫，包含許多語言繫結
  * [NVIDIA DIGITS](https://developer.nvidia.com/digits)：一種圖形化系統，可簡化常見的深入學習工作
  * [PyTorch](http://pytorch.org/)：高層級的 Python 程式庫，支援動態網路
  * [TensorFlow](https://www.tensorflow.org/)︰Google 提供的機器智慧開放原始碼程式庫
  * [Theano](http://deeplearning.net/software/theano/)：一種 Python 程式庫，可定義、最佳化和有效地評估涉及多維陣列的數學運算式
  * [Torch](http://torch.ch/)︰廣泛支援機器學習演算法的科學運算架構
  * CUDA、cuDNN 和 NVIDIA 驅動程式
  * 許多範例 Jupyter 筆記本

所有程式庫均為 GPU 版本，但它們也可在 CPU 上執行。

適用於 Linux 的資料科學虛擬機器也包含資料科學和開發活動常用的工具，包括︰

* Microsoft R Server Developer Edition 含 Microsoft R Open
* Anaconda Python 散發套件 (2.7 版和 3.5 版)，包括常用的資料分析程式庫
* JuliaPro - 使用受歡迎科學和資料分析程式庫的 Julia 語言策劃分佈
* 獨立 Spark 執行個體和單一節點 Hadoop (HDFS、Yarn)
* JupyterHub - 多使用者的 Jupyter Notebook 伺服器，支援 R、Python、PySpark、Julia 核心
* Azure 儲存體總管
* 用於管理 Azure 資源的 Azure 命令列介面 (CLI)
* 機器學習工具
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit)︰快速的機器學習系統，支援像是線上、雜湊，allreduce、簡化、learning2search、主動和互動式學習的技術
  * [XGBoost](https://xgboost.readthedocs.org/en/latest/)︰提供快速且正確的推進式決策樹實作的工具
  * [Rattle](https://togaware.com/rattle/)︰一種圖形化工具，可幫助您輕鬆地開始使用 R 中的資料分析和機器學習
  * [LightGBM](https://github.com/Microsoft/LightGBM)︰快速、分散式的高效能漸層提升架構
* Java、Python、node.js、Ruby、PHP 中的 Azure SDK
* R 和 Python 語言的程式庫，可用於 Azure Machine Learning 和其他 Azure 服務
* 開發工具和編輯器 (RStudio、PyCharm、IntelliJ、Emacs、vim)


執行資料科學涉及反覆進行一連串的工作︰

1. 尋找、載入和前置處理資料
1. 建置和測試模型
1. 部署要在智慧型應用程式中使用的模型

資料科學家使用各種工具來完成這些工作。 尋找適當版本的軟體然後下載、編譯並安裝這些版本是相當耗費時間的工作。

Linux 適用的資料科學虛擬機器可以大幅減輕這樣的負擔。 使用它可以快速啟動分析專案。 它能讓您用各種語言處理工作，包含 R、Python、SQL、Java 與 C++。 VM 中所含的 Azure SDK 可讓您使用適用於 Microsoft 雲端平台之 Linux 上的各種服務，來建置您的應用程式。 此外，您可以存取也已預先安裝的其他語言，例如 Ruby、Perl、PHP 和 node.js。

這個資料科學 VM 映像沒有任何軟體費用。 您僅需支付 Azure 硬體使用費，這是根據您所佈建的虛擬機器大小來評估。 如需計算費用的詳細資訊，請參閱 [Azure Marketplace 上的 VM 清單頁面 (英文)](https://azure.microsoft.com/marketplace/partners/microsoft-ads/linux-data-science-vm/)。

## <a name="other-versions-of-the-data-science-virtual-machine"></a>其他版本的資料科學虛擬機器
另提供 [CentOS](linux-dsvm-intro.md) 映像，其中包含許多與 Ubuntu 映像相同的工具。 亦提供 [Windows](provision-vm.md) 映像。

## <a name="prerequisites"></a>必要條件
您必須先具有 Azure 訂用帳戶，才可以建立 Linux 適用的資料科學虛擬機器。 若要取得訂用帳戶，請參閱[取得 Azure 免費試用](https://azure.microsoft.com/free/)。

## <a name="create-your-data-science-virtual-machine-for-linux"></a>建立 Linux 適用的資料科學虛擬機器
建立 Linux 適用的資料科學虛擬機器執行個體的步驟如下：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com/#create/microsoft-dsvm.linux-data-science-vm-ubuntulinuxdsvmubuntu)上的虛擬機器清單。 如果您還沒登入，則系統會提示您登入 Azure 帳戶。 
1. 按一下底部的 [建立] 以進入 wizard.![configure-data-science-vm](./media/dsvm-ubuntu-intro/configure-data-science-virtual-machine.png)
1. 下列各節提供精靈中每個步驟 (列舉於上圖的右邊) 的輸入，可用來建立 Microsoft 資料科學虛擬機器。 以下是設定每個步驟所需的輸入：
   
   a. **基本**：
   
   * **名稱**：您建立的資料科學伺服器名稱。
   * **VM 磁碟類型**：如果您偏好固態硬碟 (SSD)，請選擇 [進階 SSD]。 否則請選擇 [標準 HDD]。 
   * **使用者名稱**：第一個帳戶登入識別碼。
   * **密碼**︰第一個帳戶密碼 (您可以使用 SSH 公開金鑰來代替密碼)。
   * **訂用帳戶**：如果您有多個訂用帳戶，請選取要用來建立機器和開立帳單的訂用帳戶。 您必須有此訂用帳戶的資源建立權限。
   * **資源群組**：您可以建立新群組或使用現有的群組。
   * **位置**：選取最適合的資料中心。 它通常是擁有您大部分資料的資訊中心，或是最接近您實際位置可進行最快速網路存取的資料中心。
   
   b. **大小**：
   
   * 選取其中一個符合您的功能性需求和成本條件約束的伺服器類型。 針對 GPU 型 VM 執行個體，選取一個 NC 或 ND 類別的 VM。 [依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)頁面會列出使用 GPU 的區域。
   
   c. **設定**：
   
   * 在大部分情況下，您可以只使用預設值。 若考慮使用非預設值，可將滑鼠停留在特定欄位的資訊連結上以取得說明。
   
   d. **摘要**：
   
   * 請確認您輸入的所有資訊都正確無誤。 系統會提供使用規定的連結。 VM 除了計算您在 [大小]  步驟中所選擇的伺服器大小之外，不會收取任何其他費用。 若要開始佈建，請按一下 [建立]。 
   
佈建大約需要 5 分鐘。 在 Azure 入口網站中會顯示佈建的狀態。

## <a name="how-to-access-the-data-science-virtual-machine-for-linux"></a>如何存取 Linux 適用的資料科學虛擬機器

您可以使用三種方法存取 Ubuntu DSVM：
1. SSH (如果是終端機工作階段)
1. X2Go (如果是圖形化工作階段)
1. JupyterHub 和 JupyterLab (如果是 Jupyter 筆記本)

### <a name="ssh"></a>SSH

建立 VM 之後，您就可以使用 SSH 登入。 針對文字殼層介面，使用您在步驟 3 的 **基本** 區段中建立的帳戶認證。 在 Windows 上，您可以下載 SSH 用戶端工具，例如 [Putty](http://www.putty.org)。 如果您偏好圖形化桌面 (X Windows 系統)，您可以在 Putty 上使用 X11 轉寄或安裝 X2Go 用戶端。

> [!NOTE]
> 在測試中，X2Go 用戶端的效能優於 X11 轉寄。 我們建議您使用 X2Go 用戶端作為圖形化桌面介面。
> 
> 

### <a name="x2go"></a>X2Go
Linux VM 已經佈建了 X2Go 伺服器，並準備接受用戶端連接。 若要連線到 Linux VM 圖形化桌面，請在用戶端上完成下列程序：

1. 從 [X2Go](http://wiki.x2go.org/doku.php/doc:installation:x2goclient)下載並安裝您用戶端平台適用的 X2Go 用戶端。    
1. 執行 X2Go 用戶端，然後選取 [新增工作階段] 。 會開啟具有多個索引標籤的組態視窗。 輸入下列組態參數︰
   * **[工作階段] 索引標籤**：
     * **主機**︰Linux 資料科學 VM 的主機名稱或 IP 位址。
     * **登入**︰Linux VM 上的使用者名稱。
     * **SSH 連接埠**︰保留預設值 22。
     * **工作階段類型**︰將值變更為 XFCE。 Linux VM 目前僅支援 XFCE 桌面。
   * **[媒體] 索引標籤**︰您可以關閉聲音支援和用戶端列印，如果不需要使用的話。
   * **共用資料夾**︰如果您想要用戶端機器的目錄掛接在 Linux VM 上，請在此索引標籤上加入要與 VM 分享的目錄。

當您透過 X2Go 用戶端使用 SSH 用戶端或 XFCE 圖形化桌面登入 VM 之後，便可開始使用已安裝並設定於 VM 上的工具。 在 XFCE 上，您可以看到許多工具的應用程式功能表捷徑和桌面圖示。

### <a name="jupyterhub-and-jupyterlab"></a>JupyterHub 和 JupyterLab

Ubuntu DSVM 執行 [JupyterHub](https://github.com/jupyterhub/jupyterhub)，這是一個多使用者的 Jupyter 伺服器。 若要連線，請在您的膝上型電腦或桌上型電腦瀏覽至 https://your-vm-ip:8000，輸入您用來建立 VM 的使用者名稱和密碼，然後登入。 有許多範例筆記本可供您瀏覽和嘗試。

也提供 JupyterLab (新一代的 Jupyter 筆記本) 與 JupyterHub。 若要存取，請登入 JupyterHub，然後瀏覽至 URL https://your-vm-ip:8000/user/your-username/lab。 您可以在 /etc/jupyterhub/jupyterhub_config.py 中加入下面這一行，設定 JupyterLab 為預設的筆記本伺服器：

    c.Spawner.default_url = '/lab'

## <a name="tools-installed-on-the-data-science-virtual-machine-for-linux"></a>適用於 Linux 的資料科學虛擬機器上所安裝的工具
### <a name="deep-learning-libraries"></a>深入學習程式庫

#### <a name="cntk"></a>CNTK
Microsoft 辨識工具組是開放原始碼的深度學習工具組。 Python 繫結位於根環境和 py35 Conda 環境中。 它也包含已經在 PATH 中的一種命令列工具 (cntk)。

您可以在 JupyterHub 中找到範例 Python 筆記本。 若要於命令列執行基本範例，請在殼層中執行以下命令：

    cd /home/[USERNAME]/notebooks/CNTK/HelloWorld-LogisticRegression
    cntk configFile=lr_bs.cntk makeMode=false command=Train

如需詳細資訊，請參閱 [GitHub](https://github.com/Microsoft/CNTK) 上的 CNTK 章節，以及 [CNTK wiki](https://github.com/Microsoft/CNTK/wiki)。

#### <a name="caffe"></a>Caffe
Caffe 是 Berkeley Vision and Learning Center 中的一種深入學習架構。 它位於 /opt/caffe。 您可以在 /opt/caffe/examples 中找到範例。

#### <a name="caffe2"></a>Caffe2
Caffe2 是來自 Facebook (以 Caffe 為基礎而建置) 的深入學習架構。 您可以在 Conda 根環境中的 Python 2.7 中找到它。 若要啟動它，請從殼層執行下列命令：

    source /anaconda/bin/activate root

您可以在 JupyterHub 找到一些範例筆記本。

#### <a name="h2o"></a>H2O
H2O 是快速、記憶體內的分散式機器學習和預測性分析平台。 Python 封裝同時安裝於根環境和 py35 Anaconda 環境中。 同時也會安裝 R 封裝。 若要從命令列啟動 H2O，請執行 `java -jar /dsvm/tools/h2o/current/h2o.jar`；有各種您可能會想要設定的[命令列選項](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/starting-h2o.html#from-the-command-line) \(英文\)。 可以瀏覽至 http://localhost:54321 存取 Flow Web UI 以開始使用。 您也可以在 JupyterHub 找到範例筆記本。

#### <a name="keras"></a>Keras
Keras 是以 Python 撰寫的高層級類神經網路 API，可在 TensorFlow 或 Theano 上執行。 它位於根環境和 py35 Python 環境中。 

#### <a name="mxnet"></a>MXNet
MXNet 是兼具效率和彈性的深入學習架構。 它在 DSVM 中包含了 R 和 Python 繫結。 範例筆記本內含在 JupyterHub 中，而範例程式碼則位於 /dsvm/samples/mxnet。

#### <a name="nvidia-digits"></a>NVIDIA DIGITS
NVIDIA 深入學習 GPU 訓練系統 (稱為 DIGITS) 是為了簡化常見的深入學習工作 (如管理資料、設計和訓練 GPU 系統上的類神經網路及使用進階視覺效果即時監視效能) 的一種系統。 

DIGITS 是做為服務 (稱為 digits) 提供。 啟動此服務並瀏覽至 http://localhost:5000 以開始使用。

DIGITS 也會安裝為 Conda 根環境中的 Python 模組。

#### <a name="tensorflow"></a>TensorFlow
TensorFlow 是 Google 的深入學習程式庫。 它是使用資料流程圖進行數值計算的開放原始碼軟體程式庫。 TensorFlow 位於 py35 Python 環境中，一些範例筆記本則內含在 JupyterHub 中。

#### <a name="theano"></a>Theano
Theano 是能進行高效數值計算的 Python 程式庫。 它位於根環境和 py35 Python 環境中。 

#### <a name="torch"></a>Torch
Torch 是廣泛支援機器學習演算法的科學運算架構。 它位於 /dsvm/tools/torch 中，而 th 互動式工作階段和 luarocks 封裝管理員則是透過命令列叫用。 範例位於 /dsvm/samples/torch。

PyTorch 也位於根 Anaconda 環境中。 範例位於 /dsvm/samples/pytorch。

### <a name="microsoft-r-server"></a>Microsoft R 伺服器
R 是其中一種最受歡迎的資料分析和機器學習語言。 如果您想要將 R 用於分析，VM 會具有 Microsoft R 伺服器 (MRS)，它具有 Microsoft R Open (MRO) 和數學核心程式庫 (MKL)。 MKL 會將分析演算法中常見的數學運算作業最佳化。 MRO 可與 CRAN-R 完全相容，CRAN 中發佈的任何 R 程式庫都可安裝在 MRO 上。 MRS 可讓您將 R 模型調整和實施到 Web 服務。 您可以在其中一個預設編輯器 (如 RStudio、vi 或 Emacs) 中編輯您的 R 程式。 如果您偏好使用 Emacs 編輯器，它已經預先安裝完畢。 Emacs 套件 ESS (Emacs Speaks Statistics) 能夠簡化在 Emacs 編輯器中處理 R 檔案的程序。

若要啟動 R 主控台，您只需在殼層中輸入 **R**。 這會帶您前往互動式環境。 若要開發 R 程式，您通常會使用編輯器 (例如 Emacs 或 vi)，然後在 R 內執行指令碼。使用 RStudio，您就會有完整的圖形化整合式開發環境 (IDE) 來開發 R 程式。

如果您想要的話，另外也會有 R 指令碼讓您安裝[最熱門的 20 個 R 套件](http://www.kdnuggets.com/2015/06/top-20-r-packages.html)。 當您位於 R 互動式介面 (在殼層中輸入 **R** (如上所述)，即可進入) 時，就可以執行這個指令碼。  

### <a name="python"></a>Python
Anaconda Python 是隨著 Python 2.7 和 3.5 環境一起安裝。 2.7 環境稱為 _root_，而 3.5 環境稱為 _py35_。 這個散發套件包含基本的 Python 以及大約 300 個最受歡迎的數學運算、工程設計和資料分析封裝。 

py35 環境是預設環境。 啟用 root (2.7) 環境：

    source activate root

再次啟用 py35 環境：

    source activate py35

若要叫用 Python 互動式工作階段，只需在殼層中輸入 **python** 。 

使用 ```conda``` 或 ````pip```` 安裝其他 Python 程式庫。 對於 pip，如果您不想要預設值，請先啟用正確的環境：

    source activate root
    pip install <package>

或指定 pip 的完整路徑：

    /anaconda/bin/pip install <package>
    
對於 conda，您一律要指定環境名稱 (_py35_ 或 _root_)：

    conda install <package> -n py35

如果您位於圖形化介面，或已設定 X11 轉寄，則可輸入 **pycharm** 來啟動 PyCharm Python IDE。 您可以使用預設的文字編輯器。 此外，您也可以使用 Spyder，這是與 Anaconda Python 散發套件配套的 Python IDE。 Spyder 需要圖形化桌面或 X11 轉寄。 圖形化桌面中提供了 Spyder 的捷徑。

### <a name="jupyter-notebook"></a>Jupyter 筆記本
Jupyter Notebook 中也隨附 Anaconda 散發套件，這是一個共用程式碼與分析的環境。 Jupyter 筆記本是透過 JupyterHub 來存取。 您可以使用本機 Linux 使用者名稱和密碼來登入。

Jupyter 筆記本伺服器已使用 Python 2、Python 3 及 R 核心進行預先設定。 有一個名為「Jupyter 筆記本」的桌面圖示可以啟動瀏覽器來存取 Notebook 伺服器。 如果您透過 SSH 或 X2Go 用戶端進入 VM，則也可以造訪 [https://localhost:8000/](https://localhost:8000/) 來存取 Jupyter 筆記本伺服器。

> [!NOTE]
> 如果您收到任何憑證警告，請繼續。
> 
> 

您可以從任何主機存取 Jupyter Notebook 伺服器。 只要輸入 https://\<VM DNS 名稱或 IP 位址\>:8000/

> [!NOTE]
> 在佈建 VM 時，防火牆預設會開啟連接埠 8000。
> 
> 

我們已封裝範例 Notebook - 一個用於 Python，另一個用於 R。 選取 [新增] 然後選取適當的語言核心，便可建立新的 Notebook。 如果您沒有看到 [新增] 按鈕，請按一下左上角的 **Jupyter** 圖示移至 Notebook 伺服器的首頁。

### <a name="apache-spark-standalone"></a>獨立 Apache Spark 
Apache Spark 的獨立執行個體已預先安裝在 Linux DSVM，可協助您在大型叢集上測試及部署之前，先在本機上開發 Spark 應用程式。 您可以透過 Jupyter 核心執行 PySpark 程式。 當您開啟 Jupyter 時，請按一下 [新增] 按鈕，您應該會看到可用核心的清單。 "Spark - Python" 是 PySpark 核心，可讓您使用 Python 語言建置 Spark 應用程式。 您也可以使用像是 PyCharm 或 Spyder 的 Python IDE 來建置您的 Spark 程式。 因為這是獨立執行個體，Spark 堆疊會在呼叫用戶端程式中執行。 相較於在 Spark 叢集上進行開發，這樣可以更快速且輕鬆地針對問題進行疑難排解。 

範例 PySpark Notebook 在 Jupyter 上提供，您可以在 Jupyter 的主目錄 ($HOME/notebooks/SparkML/pySpark) 底下的 "SparkML" 目錄中找到。 

如果您是針對 Spark 在 R 中進行程式設計，您可以使用 Microsoft R 伺服器、SparkR 或 sparklyr。 

在 Microsoft R 伺服器中的 Spark 內容中執行之前，您必須執行一次性設定步驟，以啟用本機單一節點 Hadoop HDFS 和 Yarn 執行個體。 根據預設，Hadoop 服務已安裝但是在 DSVM 上已停用。 若要啟用它，您需要在第一次以 root 身分執行下列命令︰

    echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
    cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
    chmod 0600 ~hadoop/.ssh/authorized_keys
    chown hadoop:hadoop ~hadoop/.ssh/id_rsa
    chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
    chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
    systemctl start hadoop-namenode hadoop-datanode hadoop-yarn

您可以在不需要它們時停止 Hadoop 相關服務，方法是執行 ````systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn```` 示範如何在遠端 Spark 內容中開發和測試 MRS (這是 DSVM 上的獨立 Spark 執行個體) 的範例，以及 `/dsvm/samples/MRS` 目錄中的可用項目。 

### <a name="ides-and-editors"></a>IDE 和編輯器
您可以選擇數個程式碼編輯器。 這包括 vi/VIM、Emacs、PyCharm、RStudio 和 IntelliJ。 IntelliJ、RStudio 和 PyCharm 是圖形化編輯器，您需要登入圖形化桌面才能使用它們。 這些編輯器有桌面和應用程式功能表的捷徑可以啟動它們。

**VIM** 和 **Emacs** 是文字型編輯器。 在 Emacs 上，我們已安裝稱為 Emacs Speaks Statistics (ESS) 的附加元件封裝，它可以方便於在 Emacs 編輯器中使用 R。 如需詳細資訊，請參閱 [ESS](http://ess.r-project.org/)。

**LaTex** 透過 texlive 封裝與 Emacs 附加元件 [auctex](https://www.gnu.org/software/auctex/manual/auctex/auctex.html) 封裝一起安裝，可簡化 Emacs 內的 LaTex 文件撰寫。  

### <a name="databases"></a>資料庫

#### <a name="graphical-sql-client"></a>圖形化 SQL 用戶端
已提供圖形化 SQL 用戶端 **SQuirrel SQL** 來連接至不同的資料庫 (例如 Microsoft SQL Server 及 MySQL)，並執行 SQL 查詢。 您可以從圖形化桌面工作階段 (例如使用 X2Go 用戶端) 執行它。 若要叫用 SQuirrel SQL，您可以從桌面上的圖示啟動，或是在殼層上執行下列命令。

    /usr/local/squirrel-sql-3.7/squirrel-sql.sh

第一次使用之前，請設定驅動程式和資料庫別名。 JDBC 驅動程式位於︰

*/usr/share/java/jdbcdrivers*

如需詳細資訊，請參閱 [SQuirrel SQL](http://squirrel-sql.sourceforge.net/index.php?page=screenshots)。

#### <a name="command-line-tools-for-accessing-microsoft-sql-server"></a>存取 Microsoft SQL Server 用的命令列工具
SQL Server 的 ODBC 驅動程式封裝也隨附兩個命令列工具：

**bcp**：bcp 公用程式會在 Microsoft SQL Server 執行個體和使用者指定格式的資料檔之間大量複製資料。 bcp 公用程式可用來將大量新資料列匯入 SQL Server 資料表，或是將資料表的資料匯出成資料檔案。 若要將資料匯入資料表，您必須使用為該資料表建立的格式檔案，或是了解資料表的結構，以及適用於其資料行的資料類型。

如需詳細資訊，請參閱 [連接 bcp](https://msdn.microsoft.com/library/hh568446.aspx)。

**sqlcmd**：您可以利用 sqlcmd 公用程式，在命令列提示字元中輸入 Transact-SQL 陳述式以及系統程序和指令碼檔案。 這個公用程式使用 ODBC 來執行 Transact-SQL 批次。

如需詳細資訊，請參閱 [使用 sqlcmd 連接](https://msdn.microsoft.com/library/hh568447.aspx)。

> [!NOTE]
> 此公用程式在 Linux 和 Windows 平台之間有一些差異。 如需詳細資訊，請參閱上述文件。
> 
> 

#### <a name="database-access-libraries"></a>資料庫存取程式庫
R 和 Python 中提供可用於存取資料庫的程式庫。

* 在 R 中，**RODBC** 套件或 **dplyr** 套件可讓您查詢資料庫伺服器或在其上執行 SQL 陳述式。
* 在 Python 中， **pyodbc** 程式庫提供使用 ODBC 作為基礎層的資料庫存取。  

### <a name="azure-tools"></a>Azure 工具
VM 上安裝了下列 Azure 工具：

* **Azure 命令列介面**：Azure CLI 可讓您透過殼層命令來建立和管理 Azure 資源。 若要叫用 Azure 工具，只需輸入 **azure help**。 如需詳細資訊，請參閱 [Azure CLI 文件頁面](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)。
* **Microsoft Azure 儲存體總管**：Microsoft Azure 儲存體總管是一個圖形化工具，可用來瀏覽您在 Azure 儲存體帳戶中所儲存的物件，也可以上傳和下載 Azure Blob 的資料。 您可以從桌面捷徑圖示存取儲存體總管。 從殼層命令提示字元叫用它則是輸入 **StorageExplorer**。 您需要從 X2Go 用戶端登入，或是已設定 X11 轉寄。
* **Azure 程式庫**：以下是一些預先安裝的程式庫。
  
  * **Python**：Python 中已安裝的 Azure 相關程式庫為 **azure**、**azureml**、**pydocumentdb** 和 **pyodbc**。 透過前三個程式庫，您可以存取 Azure 儲存體服務、Azure Machine Learning 和 Azure Cosmos DB (Azure 上的 NoSQL 資料庫)。 第四個程式庫 pyodbc (以及 Microsoft ODBC Driver for SQL Server) 可讓您使用 ODBC 介面，從 Python 中存取 SQL Server、Azure SQL Database 和 Azure SQL 資料倉儲。 輸入 **pip list** 以查看所有列出的程式庫。 請務必在 Python 2.7 和 3.5 的環境中執行此命令。
  * **R**：R 中已安裝的 Azure 相關程式庫為 **AzureML** 和 **RODBC**。
  * **Java**︰Azure Java 程式庫清單位於 VM 上的 **/dsvm/sdk/AzureSDKJava** 目錄中。 關鍵的程式庫為 Azure 儲存體和管理 API、Azure Cosmos DB 和 JDBC Driver for SQL Server。  

您可以從預先安裝的 Firefox 瀏覽器存取 [Azure 入口網站](https://portal.azure.com) 。 在 Azure 入口網站中，您可以建立、管理和監視 Azure 資源。

### <a name="azure-machine-learning"></a>Azure Machine Learning
Azure Machine Learning 是完全受控的雲端服務，讓您能夠建置、部署及共用預測性分析解決方案。 您可以從 Azure Machine Learning Studio 中建置實驗和模型。 其可從資料科學虛擬機器上的網頁瀏覽器中存取，請造訪 [Microsoft Azure Machine Learning](https://studio.azureml.net)。

當您登入 Azure Machine Learning Studio 之後，就能存取實驗畫布，您可以在這裡建置機器學習演算法的邏輯流程。 您也可以存取裝載於 Azure Machine Learning 的 Jupyter 筆記本，並且可以順暢地在 Machine Learning Studio 中使用實驗。 讓您建置的機器學習模型能夠運作，方法是將它們包裝在 Web 服務介面中。 這讓使用任何語言撰寫的用戶端可從機器學習模型中叫用預測。 如需詳細資訊，請參閱 [機器學習文件](https://azure.microsoft.com/documentation/services/machine-learning/)。

您可以也在 VM 上，以 R 或 Python 建置模型，然後將它部署在 Azure Machine Learning 的生產環境中。 我們已安裝 R (**AzureML**) 和 Python (**azureml**) 的程式庫來啟用這項功能。

如需如何將 R 和 Python 的模型部署到 Azure Machine Learning 的相關資訊，請參閱 [您可以在 Data Science Virtual Machine 上做的十件事](vm-do-ten-things.md) (特別是＜使用 R 或 Python 建置模型並且使用 Azure Machine Learning 實作＞一節)。

> [!NOTE]
> 這些指示是針對資料科學 VM 的 Windows 版本所撰寫。 但所提供關於將模型部署到 Azure Machine Learning 的資訊也適用於 Linux VM。
> 
> 

### <a name="machine-learning-tools"></a>機器學習工具
VM 隨附一些已預先編譯且預先安裝在本機上的機器學習工具和演算法。 其中包含：

* **Vowpal Wabbit**：快速線上學習演算法。
* **xgboost**：提供最佳化推進式決策樹演算法的工具。
* **Rattle**：以 R 為基礎的圖形化工具，可輕鬆地進行資料瀏覽和模組化。
* **Python**：Anaconda Python 會與含有像是 Scikit-learn 的程式庫的機器學習演算法進行配套。 您可以使用 `pip install` 程式庫。
* **LightGBM**︰以決策樹演算法為基礎的一種快速、分散式的高效能漸層提升架構。
* **R**：有豐富的機器學習函式程式庫可供 R 使用。lm、glm、randomForest、rpart 是一些預先安裝的程式庫。 您可以執行下列命令來安裝其他程式庫：
  
        install.packages(<lib name>)

以下是一些關於清單中前三個機器學習工具的其他資訊。

#### <a name="vowpal-wabbit"></a>Vowpal Wabbit
Vowpal Wabbit 是一個機器學習系統，其會使用像是線上、雜湊，allreduce、簡化、learning2search、主動和互動式學習的技術。

若要在基本的範例上執行此工具，請執行下列命令：

    cp -r /dsvm/tools/VowpalWabbit/demo vwdemo
    cd vwdemo
    vw house_dataset

該目錄中有其他更大的示範。 如需 VW 的詳細資訊，請參閱 [GitHub 的這一節](https://github.com/JohnLangford/vowpal_wabbit)，以及 [Vowpal Wabbit wiki](https://github.com/JohnLangford/vowpal_wabbit/wiki)。

#### <a name="xgboost"></a>XGBoost
這是針對推進式決策 (樹) 演算法設計及最佳化的的程式庫。 此程式庫的目標是要將機器的運算限制推到所需的極致，以提供可調整、可攜且精確的大規模樹狀推進式決策。

它提供了命令列以及 R 程式庫。

若要在 R 中使用此程式庫，您可以啟動互動式 R 工作階段 (只要在殼層中輸入 **R** ) 並載入程式庫。

以下是簡單的範例，您可以在 R 提示字元中執行︰

    library(xgboost)

    data(agaricus.train, package='xgboost')
    data(agaricus.test, package='xgboost')
    train <- agaricus.train
    test <- agaricus.test
    bst <- xgboost(data = train$data, label = train$label, max.depth = 2,
                    eta = 1, nthread = 2, nround = 2, objective = "binary:logistic")
    pred <- predict(bst, test$data)

若要執行 xgboost 命令列，以下是要在殼層中執行的命令︰

    cp -r /dsvm/tools/xgboost/demo/binary_classification/ xgboostdemo
    cd xgboostdemo
    xgboost mushroom.conf


.model 檔案會寫入指定的目錄。 如需此示範範例的相關資訊，請參閱 [GitHub](https://github.com/dmlc/xgboost/tree/master/demo/binary_classification)。

如需 xgboost 的詳細資訊，請參閱 [xgboost 文件頁面 (英文)](https://xgboost.readthedocs.org/en/latest/) 及其 [GitHub 存放庫 (英文)](https://github.com/dmlc/xgboost)。

#### <a name="rattle"></a>Rattle
Rattle (**R** **A**nalytical **T**ool **T**o **L**earn **E**asily) 會使用 GUI 型資料探索和模型化。 它會提供資料的統計和視覺化摘要、轉換已經可以輕易模型化的資料、從資料建置未經指導和經過指導的模型、以圖形方式呈現模型的效能，以及評分新的資料集。 它也會產生 R 程式碼，複寫 UI 中可以直接在 R 中執行的作業，或是作為進一步分析之起點的作業。

若要執行 Rattle，您必須位於圖形化桌面登入工作階段。 在終端機中，輸入 ```R``` 進入 R 環境。 在 R 提示字元中輸入下列命令：

    library(rattle)
    rattle()

現在，將會開啟具有一組索引標籤的圖形化介面。 以下是在 Rattle 中使用範例天氣資料集和建立模型所需的快速入門步驟。 在下面的部分步驟中，會提示您自動安裝並載入一些系統上還沒有的必要 R 封裝。

> [!NOTE]
> 如果您沒有存取系統目錄 (預設) 中封裝的存取權，可能會在 R 主控台視窗上看到提示，詢問是否將封裝安裝至您的個人程式庫。 如果您看到這些提示，請回答 「是」  。
> 
> 

1. 按一下 [Execute (執行)] 。
1. 隨即會顯示對話方塊，詢問您是否要使用範例天氣資料集。 按一下 [是]  載入範例。
1. 按一下 [模型]  索引標籤。
1. 按一下 [執行]  建立決策樹。
1. 按一下 [繪圖]  顯示決策樹。
1. 按一下 [樹系] 選項按鈕，然後按一下 [執行] 建置隨機樹系。
1. 按一下 [評估]  索引標籤。
1. 按一下 [風險] 選項按鈕，然後按一下 [執行] 顯示兩個風險 (累積) 效能繪圖。
1. 按一下 [記錄檔] 索引標籤，顯示針對先前作業產生的 R 程式碼。
   (由於目 Rattle 版本中的錯誤，您必須在記錄檔的文字中，於 *Export this log ...* 前面插入 *#* 字元。)
1. 按一下 [匯出] 按鈕，將名為 weather_script.R 的 R 指令碼檔案儲存到主資料夾。

您可以結束 Rattle 和 R。現在您可以修改產生的 R 指令碼，或照原樣使用它，隨時執行，以便重複執行在 Rattle UI 中完成的所有動作。 這是特別適合 R 初學者的簡單方式，讓他們得以在簡單的圖形介面中快速進行分析和機器學習，同時自動產生 R 程式碼來修改及/或學習。

## <a name="next-steps"></a>後續步驟
以下是如何繼續進行學習和探索的方式：

* [適用於 Linux 的資料科學虛擬機器上的資料科學](linux-dsvm-walkthrough.md)逐步解說示範如何使用此處佈建的 Linux 資料科學 VM 來執行數個常見的資料科學工作。 
* 試試本文中所述的工具，在資料科學 VM 上探索各種資料科學工具。 您也可以在虛擬機器內的殼層上執行 *dsvm-more-info* ，以獲得關於 VM 上所安裝工具的基本簡介和詳細資訊的指標。  
* 了解如何使用 [Team Data Science Process](http://aka.ms/tdsp)，以系統化方式建置端對端分析方案。
* 瀏覽 [Azure AI 資源庫](https://gallery.azure.ai/)，可取得使用 Azure AI 服務的機器學習和資料分析範例。

