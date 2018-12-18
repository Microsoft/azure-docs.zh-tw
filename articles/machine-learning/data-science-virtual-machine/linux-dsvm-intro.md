---
title: 在 Azure 上佈建 Linux CentOS 資料科學虛擬機器 | Microsoft Docs
description: 在 Azure 上設定和建立 Linux 資料科學虛擬機器以進行分析和機器學習服務。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: 3bab0ab9-3ea5-41a6-a62a-8c44fdbae43b
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: db1d449e5c9f66446f00b637c5550dc9144920ff
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39446760"
---
# <a name="provision-a-linux-centos-data-science-virtual-machine-on-azure"></a>在 Azure 上佈建 Linux CentOS 資料科學虛擬機器

Linux 資料科學虛擬機器是以 CentOS 為基礎的 Azure 虛擬機器，其中隨附預先安裝的工具集合。 這些工具通常用來執行資料分析和機器學習。 包含的重要軟體元件如下︰

* 作業系統：Linux CentOS 散發套件。
* Microsoft R Server Developer Edition
* Anaconda Python 散發套件 (2.7 版和 3.5 版)，包括常用的資料分析程式庫
* JuliaPro - 使用受歡迎科學和資料分析程式庫的 Julia 語言策劃分佈
* 獨立 Spark 執行個體和單一節點 Hadoop (HDFS、Yarn)
* JupyterHub - 多使用者的 Jupyter Notebook 伺服器，支援 R、Python、PySpark、Julia 核心
* Azure 儲存體總管
* 用於管理 Azure 資源的 Azure 命令列介面 (CLI)
* PostgresSQL 資料庫
* 機器學習工具
  * [辨識工具組](https://github.com/Microsoft/CNTK) \(英文\)︰來自 Microsoft Research 的深度學習軟體工具組。
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit)︰快速的機器學習系統，支援像是線上、雜湊，allreduce、簡化、learning2search、主動和互動式學習的技術。
  * [XGBoost](https://xgboost.readthedocs.org/en/latest/)︰提供快速且正確的推進式決策樹實作的工具。
  * [Rattle](http://rattle.togaware.com/) (R Analytical Tool To Learn Easily)︰此工具可讓您利用 GUI 型資料探索，輕鬆地使用 R 架構開始進行資料分析與機器學習，以及自動產生 R 程式碼來建立模型。
* Java、Python、node.js、Ruby、PHP 中的 Azure SDK
* R 和 Python 語言的程式庫，可用於 Azure Machine Learning 和其他 Azure 服務
* 開發工具和編輯器 (RStudio、PyCharm、IntelliJ、Emacs、gedit、vi)


執行資料科學涉及反覆進行一連串的工作︰

1. 尋找、載入和前置處理資料
1. 建置和測試模型
1. 部署要在智慧型應用程式中使用的模型

資料科學家使用各種工具來完成這些工作。 尋找適當版本的軟體然後下載、編譯並安裝這些版本是相當耗費時間的工作。

Linux 資料科學虛擬機器可以大幅減輕這樣的負擔。 使用它可以快速啟動分析專案。 它能讓您用各種語言處理工作，包含 R、Python、SQL、Java 與 C++。 Eclipse 提供整合式開發環境 (IDE) 以開發和測試易於使用的程式碼。 VM 中所含的 Azure SDK 可讓您使用適用於 Microsoft 雲端平台之 Linux 上的各種服務，來建置您的應用程式。 此外，您可以存取也已預先安裝的其他語言，例如 Ruby、Perl、PHP 和 node.js。

這個資料科學 VM 映像沒有任何軟體費用。 您僅需支付 Azure 硬體使用費，這是根據您使用 VM 映像所佈建的虛擬機器大小來評估。 如需計算費用的詳細資訊，請參閱 [Azure Marketplace 上的 VM 清單頁面 (英文)](https://azure.microsoft.com/marketplace/partners/microsoft-ads/linux-data-science-vm/)。

## <a name="other-versions-of-the-data-science-virtual-machine"></a>其他版本的資料科學虛擬機器
另提供 [Ubuntu](dsvm-ubuntu-intro.md) 映像，其中包含許多與 CentOS 映像相同的工具及深入學習架構。 亦提供 [Windows](provision-vm.md) 映像。

## <a name="prerequisites"></a>必要條件
您必須先具有下列項目，才可以建立 Linux 資料科學虛擬機器：

* **Azure 訂用帳戶**：若要取得訂用帳戶，請參閱 [取得 Azure 免費試用](https://azure.microsoft.com/free/)。
* **Azure 儲存體帳戶**：若要建立帳戶，請參閱 [建立 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md#create-a-storage-account)。 或者，如果您不想使用現有的帳戶，可以在建立 VM 的過程中建立儲存體帳戶。

## <a name="create-your-linux-data-science-virtual-machine"></a>建立 Linux 資料科學虛擬機器
建立 Linux 資料科學虛擬機器執行個體的步驟如下：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com/#create/microsoft-ads.linux-data-science-vmlinuxdsvm)上的虛擬機器清單。
1. 按一下底部的 [建立] 以進入 wizard.![configure-data-science-vm](./media/linux-dsvm-intro/configure-linux-data-science-virtual-machine.png)
1. 下列各節提供精靈中每個步驟 (列舉於上圖的右邊) 的輸入，可用來建立 Microsoft 資料科學虛擬機器。 以下是設定每個步驟所需的輸入：
   
   a. **基本**：
   
   * **名稱**：您建立的資料科學伺服器名稱。
   * **使用者名稱**：第一個帳戶登入識別碼。
   * **密碼**︰第一個帳戶密碼 (您可以使用 SSH 公開金鑰來代替密碼)。
   * **訂用帳戶**：如果您有多個訂用帳戶，請選取要用來建立機器和開立帳單的訂用帳戶。 您必須有此訂用帳戶的資源建立權限。
   * **資源群組**：您可以建立新群組或使用現有的群組。
   * **位置**：選取最適合的資料中心。 它通常是擁有您大部分資料的資訊中心，或是最接近您實際位置可進行最快速網路存取的資料中心。
   
   b. ：
   
   * 選取其中一個符合您的功能性需求和成本條件約束的伺服器類型。 選取 [檢視全部]  以查看更多 VM 大小的選項。
   
   c. **設定**：
   
   * **磁碟類型**：如果您偏好固態硬碟 (SSD)，請選擇 [進階]。 否則請選擇 [標準]。
   * **儲存體帳戶**：您可以在訂用帳戶中建立新的 Azure 儲存體帳戶，或使用在精靈的**基本**步驟上選擇的相同位置中的現有儲存體帳戶。
   * **其他參數**：在大部分情況下只會使用預設值。 若考慮使用非預設值，可將滑鼠停留在特定欄位的資訊連結上以取得說明。
   
   d. **摘要**：
   
   * 請確認您輸入的所有資訊都正確無誤。
   
   e. ：
   
   * 若要開始佈建，按一下 [購買] 。 會提供一個交易條款的連結。 VM 除了計算您在 [大小]  步驟中所選擇的伺服器大小之外，不會收取任何其他費用。

佈建大約 10-20 分鐘。 在 Azure 入口網站中會顯示佈建的狀態。

## <a name="how-to-access-the-linux-data-science-virtual-machine"></a>如何存取 Linux 資料科學虛擬機器
建立 VM 之後，您就可以使用 SSH 登入。 針對文字殼層介面，使用您在步驟 3 的 **基本** 區段中建立的帳戶認證。 在 Windows 上，您可以下載 SSH 用戶端工具，例如 [Putty](http://www.putty.org)。 如果您偏好圖形化桌面 (X Windows 系統)，您可以在 Putty 上使用 X11 轉寄或安裝 X2Go 用戶端。

> [!NOTE]
> 在測試中，X2Go 用戶端的效能明顯優於 X11 轉寄。 我們建議您使用 X2Go 用戶端作為圖形化桌面介面。
> 
> 

## <a name="installing-and-configuring-x2go-client"></a>安裝和設定 X2Go 用戶端
Linux VM 已經佈建了 X2Go 伺服器，並準備接受用戶端連接。 若要連接到 Linux VM 圖形化桌面，請在用戶端上進行下列動作：

1. 從 [X2Go](http://wiki.x2go.org/doku.php/doc:installation:x2goclient)下載並安裝您用戶端平台適用的 X2Go 用戶端。    
1. 執行 X2Go 用戶端，然後選取 [新增工作階段] 。 會開啟具有多個索引標籤的組態視窗。 輸入下列組態參數︰
   * **[工作階段] 索引標籤**：
     * **主機**︰Linux 資料科學 VM 的主機名稱或 IP 位址。
     * **登入**︰Linux VM 上的使用者名稱。
     * **SSH 連接埠**︰保留預設值 22。
     * **工作階段類型**︰將值變更為 XFCE。 Linux VM 目前僅支援 XFCE 桌面。
   * **[媒體] 索引標籤**︰如果您不需要使用聲音支援和用戶端列印，可以將它們關閉。
   * **共用資料夾**︰如果您想要用戶端機器的目錄掛接在 Linux VM 上，請在此索引標籤上加入要與 VM 分享的目錄。

當您透過 X2Go 用戶端使用 SSH 用戶端或 XFCE 圖形化桌面登入 VM 之後，便可開始使用已安裝並設定於 VM 上的工具。 在 XFCE 上，您可以看到許多工具的應用程式功能表捷徑和桌面圖示。

## <a name="tools-installed-on-the-linux-data-science-virtual-machine"></a>Linux 資料科學虛擬機器上所安裝的工具
### <a name="microsoft-r-server"></a>Microsoft R 伺服器
R 是其中一種最受歡迎的資料分析和機器學習語言。 如果您想要將 R 用於分析，VM 會具有 Microsoft R 伺服器 (MRS)，它具有 Microsoft R Open (MRO) 和數學核心程式庫 (MKL)。 MKL 會將分析演算法中常見的數學運算作業最佳化。 MRO 可與 CRAN-R 完全相容，CRAN 中發佈的任何 R 程式庫都可安裝在 MRO 上。 MRS 可讓您將 R 模型調整和實施到 Web 服務。 您可以在其中一個預設編輯器中編輯您的 R 程式，例如 RStudio、vi、Emacs 或 gedit。 如果您使用 Emacs 編輯器，請注意，已預先安裝 Emacs 封裝 ESS (Emacs Speaks Statistics)，它可簡化在 Emacs 編輯器內的 R 檔案處理。

若要啟動 R 主控台，您只需在殼層中輸入 **R**。 這會帶您前往互動式環境。 若要開發 R 程式，您通常會使用編輯器 (例如 Emacs、vi 或 gedit)，然後在 R 內執行指令碼。使用 RStudio，您就會有完整的圖形化 IDE 環境來開發 R 程式。

如果您想要的話，另外也會有 R 指令碼讓您安裝 [前 20 名 R 封裝](http://www.kdnuggets.com/2015/06/top-20-r-packages.html) 。 當您位於 R 互動式介面 (在殼層中輸入 **R** (如上所述)，即可進入) 時，就可以執行這個指令碼。  

### <a name="python"></a>Python
為了能夠使用 Python 進行開發，我們已安裝了 Anaconda Python 散佈 2.7 與 3.5。 這個散發套件包含基本的 Python 以及大約 300 個最受歡迎的數學運算、工程設計和資料分析封裝。 您可以使用預設的文字編輯器。 此外，您也可以使用 Spyder，這是與 Anaconda Python 散發套件配套的 Python IDE。 Spyder 需要圖形化桌面或 X11 轉寄。 圖形化桌面中提供了 Spyder 的捷徑。

由於我們有 Python 2.7 和 3.5，您必須明確啟動您想要在目前工作階段中使用的 Python 版本 (conda 環境)。 啟動程序會將 PATH 變數設定為所需的 Python 版本。

若要啟用 Python 2.7 conda 環境，請從殼層執行下列命令：

    source /anaconda/bin/activate root

Python 2.7 安裝於「/anaconda/bin」 。

若要啟用 Python 3.5 conda 環境，請從殼層執行下列命令：

    source /anaconda/bin/activate py35


Python 3.5 安裝於 */anaconda/envs/py35/bin*上。

若要叫用 Python 互動式工作階段，只需在殼層中輸入 **python** 。 如果您位於圖形化介面，或已設定 X11 轉寄，則可輸入 **pycharm** 來啟動 PyCharm Python IDE。

若要安裝其他的 Python 程式庫，您需要在 sudo 底下執行 ```conda``` 或 ````pip```` 命令，並提供 Python 套件管理員的完整路徑 (conda 或 pip) 以安裝到正確的 Python 環境。 例如︰

    sudo /anaconda/bin/pip install <package> #pip for Python 2.7
    sudo /anaconda/envs/py35/bin/pip install <package> #pip for Python 3.5
    sudo /anaconda/bin/conda install [-n py27] <package> #conda for Python 2.7, default behavior
    sudo /anaconda/bin/conda install -n py35 <package> #conda for Python 3.5


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
Apache Spark 的獨立執行個體已預先安裝在 Linux DSVM，可協助您在大型叢集上測試及部署之前，先在本機上開發 Spark 應用程式。 您可以透過 Jupyter 核心執行 PySpark 程式。 當您開啟 Jupyter 並按一下 [新增] 按鈕時，應該會看到可用核心的清單。 "Spark - Python" 是 PySpark 核心，可讓您使用 Python 語言建置 Spark 應用程式。 您也可以使用像是 PyCharm 或 Spyder 的 Python IDE 來建置您的 Spark 程式。 因為這是獨立執行個體，Spark 堆疊會在呼叫用戶端程式中執行。 相較於在 Spark 叢集上進行開發，這樣可以更快速且輕鬆地針對問題進行疑難排解。 

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
您可以選擇數個程式碼編輯器。 這包括 vi/VIM、Emacs、gEdit、PyCharm、RStudio、Eclipse 和 IntelliJ。 gEdit、Eclipse、IntelliJ、RStudio 和 PyCharm 是圖形化編輯器，您需要登入圖形化桌面才能使用它們。 這些編輯器有桌面和應用程式功能表的捷徑可以啟動它們。

**VIM** 和 **Emacs** 是文字型編輯器。 在 Emacs 上，我們已安裝稱為 Emacs Speaks Statistics (ESS) 的附加元件封裝，它可以方便於在 Emacs 編輯器中使用 R。 如需詳細資訊，請參閱 [ESS](http://ess.r-project.org/)。

**Eclipse** 是可延伸的 IDE 開放原始碼，可支援多種語言。 Java 開發人員版本是安裝在 VM 上的執行個體。 有提供幾種熱門語言的外掛程式，您可以安裝它們以擴充環境。 我們也在 Eclipse 中安裝了一個外掛程式，稱為 **Azure Toolkit for Eclipse**。 它可讓您使用支援如 Java 等語言的 Eclipse 開發環境，建立、開發、測試和部署 Azure 應用程式。 另外還有 **Azure SDK for Java** ，它可讓您從 Java 環境中存取不同的 Azure 服務。 適用於 Eclipse 的 Azure 工具組的詳細資訊位於 [適用於 Eclipse 的 Azure 工具組](../../azure-toolkit-for-eclipse.md)。

**LaTex** 透過 texlive 封裝與 Emacs 附加元件 [auctex](https://www.gnu.org/software/auctex/manual/auctex/auctex.html) 封裝一起安裝，可簡化 Emacs 內的 LaTex 文件撰寫。  

### <a name="databases"></a>資料庫
#### <a name="postgres"></a>Postgres
開放原始碼資料庫 **Postgres** 可在 VM 上使用，且服務正在執行中，並已完成 initdb。 您仍需要建立資料庫和使用者。 如需詳細資訊，請參閱 [Postgres 文件](https://www.postgresql.org/docs/)。  

#### <a name="graphical-sql-client"></a>圖形化 SQL 用戶端
已提供圖形化 SQL 用戶端 **SQuirrel SQL** 來連接至不同的資料庫 (例如 Microsoft SQL Server、Postgres 及 MySQL)，並執行 SQL 查詢。 您可以從圖形化桌面工作階段 (例如使用 X2Go 用戶端) 執行它。 若要叫用 SQuirrel SQL，您可以從桌面上的圖示啟動，或是在殼層上執行下列命令。

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

存取 **Postgres**：

* 從 R︰使用 **RPostgreSQL**封裝。
* 從 Python︰使用 **psycopg2** 程式庫。

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

* **Microsoft 辨識工具組**：一種深度學習工具組。
* **Vowpal Wabbit**：快速線上學習演算法。
* **xgboost**：提供最佳化推進式決策樹演算法的工具。
* **Python**：Anaconda Python 會與含有像是 Scikit-learn 的程式庫的機器學習演算法進行配套。 您可以使用 `pip install` 程式庫。
* **R**：有豐富的機器學習函式程式庫可供 R 使用。lm、glm、randomForest、rpart 是一些預先安裝的程式庫。 您可以執行下列命令來安裝其他程式庫：
  
        install.packages(<lib name>)

以下是一些關於清單中前三個機器學習工具的其他資訊。

#### <a name="microsoft-cognitive-toolkit"></a>Microsoft 辨識工具組
這是一個開放原始碼的深層學習工具組。 它是一種命令列工具 (cntk)，而且已經在 PATH 中。

若要執行基本範例，請在殼層中執行以下命令：

    cd /home/[USERNAME]/notebooks/CNTK/HelloWorld-LogisticRegression
    cntk configFile=lr_bs.cntk makeMode=false command=Train

如需詳細資訊，請參閱 [GitHub](https://github.com/Microsoft/CNTK) 上的 CNTK 章節，以及 [CNTK wiki](https://github.com/Microsoft/CNTK/wiki)。

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

* [Data science on the Linux Data Science Virtual Machine (Linux 資料科學虛擬機器上的資料科學)](linux-dsvm-walkthrough.md) 逐步解說會示範如何使用此處佈建的 Linux 資料科學 VM 來執行數個常見的資料科學工作。 
* 試試本文中所述的工具，在資料科學 VM 上探索各種資料科學工具。 您也可以在虛擬機器內的殼層上執行 *dsvm-more-info* ，以獲得關於 VM 上所安裝工具的基本簡介和詳細資訊的指標。  
* 了解如何使用 [Team Data Science Process](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)，以系統化方式建置端對端分析方案。
* 請造訪 [Cortana Analytics Gallery](http://gallery.cortanaanalytics.com) ，以取得使用 Cortana Analytics Suite 的機器學習和資料分析範例。

