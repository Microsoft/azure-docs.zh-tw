---
title: 在 Azure 上佈建 Windows 資料科學虛擬機器 | Microsoft Docs
description: 在 Azure 上設定和建立資料科學虛擬機器以進行分析和機器學習。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: e1467c0f-497b-48f7-96a0-7f806a7bec0b
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.devlang: na
ms.topic: article
ms.date: 08/30/2018
ms.author: gokuma
ms.openlocfilehash: 37a30055e567b27f4e215dac162642d4447a8e49
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298195"
---
# <a name="provision-the-windows-data-science-virtual-machine-on-azure"></a>在 Azure 上佈建 Windows 資料科學虛擬機器
Microsoft 資料科學虛擬機器 (DSVM) 是 Windows Azure 虛擬機器 (VM) 映像。 它會預先安裝並且以數個用於資料分析和機器學習的工具進行設定。 包含下列工具：

* [Azure Machine Learning](../service/index.yml) Workbench。
* [Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/index) Developer 版本。
* Anaconda Python 散發套件。
* Jupyter Notebook (使用 R、Python、PySpark 核心)。
* Microsoft Visual Studio Community。
* Microsoft Power BI 桌面。
* Microsoft SQL Server 2017 Developer 版本。
* 用於本機開發與測試的獨立 Apache Spark 執行個體。
* [JuliaPro](https://juliacomputing.com/products/juliapro.html)。
* 機器學習與資料分析工具：
  * 深度學習架構。 VM 中涵蓋一組豐富的 AI 架構：[Microsoft Cognitive Toolkit](https://www.microsoft.com/en-us/cognitive-toolkit/)、[TensorFlow](https://www.tensorflow.org/)、[Chainer](https://chainer.org/)、mxNet 和 Keras。
  * [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit)。 快速的機器學習系統，支援線上雜湊，allreduce、簡化、learning2search、主動式和互動式學習等技巧。
  * [XGBoost](https://xgboost.readthedocs.org/en/latest/)。 此工具可提供快速且正確的推進式決策樹實作。
  * [Rattle](https://togaware.com/rattle/) (R Analytical Tool To Learn Easily)。 此工具可讓您輕鬆地使用 R 架構開始進行資料分析與機器學習。其中包含 GUI 型資料探索，以及自動產生 R 程式碼來建立模型。
  * [Weka](http://www.cs.waikato.ac.nz/ml/weka/)。 Java 中的視覺化資料採礦和機器學習服務軟體。
  * [Apache Drill](https://drill.apache.org/)。 適用於 Hadoop、NoSQL 和雲端儲存體的無結構描述 SQL 查詢引擎。  可支援 ODBC 和 JDBC 介面，以便從 PowerBI、Microsoft Excel 和 Tableau 等標準 BI 工具查詢 NoSQL 和檔案。
* R 和 Python 語言的程式庫，可用於 Azure Machine Learning 和其他 Azure 服務。
* 包括 Git Bash 的 Git，可搭配原始程式碼存放庫 (包括 GitHub 和 Azure DevOps) 運作。 Git 會提供數個可在 Git Bash 和命令提示字元上存取的熱門 Linux 命令列公用程式。 範例包括 awk、sed、perl、grep、find、wget 和 curl。

資料科學涉及反覆進行一連串的工作︰

1. 尋找、載入及前置處理資料。
1. 建置與測試模型。
1. 部署要在智慧型應用程式中使用的模型。

資料科學家會使用各種工具來進行這些工作。 尋找適當的軟體版本，然後加以下載並安裝是耗費時間的工作。 Microsoft 資料科學虛擬機器提供可佈建於 Azure 上的現成映像，其中含有已預先安裝和設定的數種常用工具，有助於節省時間。 

Microsoft 資料科學虛擬機器可快速啟動分析專案。 您可以用各種語言處理工作，包含 R、Python、SQL 與 C#。 Visual Studio 提供容易使用的整合式開發環境 (IDE) 來開發和測試您的程式碼。 Azure SDK 包含在 VM 中。 所以您可以使用 Microsoft 雲端平台上的各種服務來建置應用程式。 

這個資料科學 VM 映像沒有任何軟體費用。 您只需支付 Azure 使用費用。 費用取決於您佈建的虛擬機器大小。 計算費用的詳細資訊位於[資料科學虛擬機器](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.windows-data-science-vm?tab=PlansAndPrice)頁面的 [定價詳細資料] 一節中。 

## <a name="other-versions-of-the-data-science-virtual-machine"></a>其他版本的資料科學虛擬機器
* [Ubuntu](dsvm-ubuntu-intro.md) 映像。 其中有許多類似 DSVM 的工具，以及一些其他的深度學習架構。 
* [Linux CentOS](linux-dsvm-intro.md) 映像。
* 資料科學虛擬機器的 [Windows Server 2012 版本](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.standard-data-science-vm)。 有一些工具僅適用於 Windows Server 2016 版。 除此之外，本文也適用於 Windows Server 2012 版本。

## <a name="prerequisite"></a>必要條件
若要建立 Microsoft 資料科學虛擬機器，您必須具有 Azrue 訂用帳戶。 請參閱[取得 Azure 免費試用](http://azure.com/free)。


## <a name="create-your-microsoft-data-science-virtual-machine"></a>建立 Microsoft 資料科學虛擬機器
若要建立 Microsoft 資料科學虛擬機器的執行個體，請遵循下列步驟：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com/#create/microsoft-dsvm.dsvm-windowsserver-2016)上的虛擬機器清單。 如果您尚未登入 Azure 帳戶，系統可能會提示您登入。
1. 選取底部的 [建立] 按鈕以進入精靈。

  ![configure-data-science-vm](./media/provision-vm/configure-data-science-virtual-machine.png) 

1. 建立 Microsoft 資料科學虛擬機器的精靈需要**輸入**資料。 需要輸入下列資料，才能設定圖右所示的每個步驟：

  a. **基本**：

    i. **名稱**。 您所建立的資料科學伺服器名稱。  

    ii. **VM 磁碟類型**。 選擇 **SSD** 或 **HDD**。 針對 NC_v1 GPU 執行個體 (像是 NVidia Tesla K80 型)，請選擇 **HDD** 作為磁碟類型。   

    iii. **使用者名稱**。 要登入的系統管理員帳戶識別碼。   

    iv. **密碼**。 系統管理員帳戶密碼。  

    v. **訂用帳戶**。 如果您有多個訂用帳戶，請選取要用來建立機器和開立帳單的訂用帳戶。   

    vi. **資源群組**。 您可以建立新群組或使用現有的群組。   

    vii. **位置**。 選取最適合的資料中心。 如需最快速的網路存取，請選取擁有您大部分資料或是最接近您實際位置的資訊中心。   

  b. **大小**。 選取其中一個符合您的功能性需求和成本條件約束的伺服器類型。 如需更多 VM 大小的選項，請選取 [檢視全部]。  

  c. **設定**：  

    i. **使用受控磁碟**。 如果您想要 Azure 來管理 VM 的磁碟，則選擇 [受控]。 否則，您必須指定新的或現有的儲存體帳戶。  

    ii. **其他參數**。 您可以使用預設值。 如果您想要使用非預設值，可將滑鼠停留在特定欄位的資訊連結上以取得說明。  

  d. **摘要**。 請確認您輸入的所有資訊都正確無誤。 選取 [建立] 。 

> [!NOTE]
> * VM 除了您在 [大小] 步驟中所選伺服器大小的計算成本之外，不會收取任何其他費用。 
> * 佈建需要大約 10 到 20 分鐘的時間。 其狀態會顯示在 Azure 入口網站上。
> 
> 

## <a name="how-to-access-the-microsoft-data-science-virtual-machine"></a>如何存取 Microsoft 資料科學虛擬機器
建立及佈建 VM 之後，您可以使用您在前面 [基本] 區段中設定的系統管理員帳戶認證從遠端桌面登入 VM。 您已準備好開始使用在 VM 上安裝及設定的工具。 許多工具都具有開始功能表圖格和桌面圖示。 


## <a name="tools-installed-on-the-microsoft-data-science-virtual-machine"></a>Microsoft 資料科學虛擬機器上所安裝的工具

### <a name="microsoft-machine-learning-server-developer-edition"></a>Microsoft Machine Learning Server Developer 版本
您可以使用 Microsoft 企業程式庫來擴充 R 或 Python 以便進行分析，因為 VM 上已安裝 Microsoft ML Server Developer 版本。 Machine Learning Server 先前稱為 Microsoft R Server，是一個能夠廣泛部署的企業級分析平台。 它同時適用於 R 和 Python，可支援商業事務，而且很安全。 

Machine Learning Server 支援各種巨量資料統計資料、預測模型和機器學習工作。 它支援全範圍的分析：探索、分析、視覺化和模型化。 藉由使用及擴充開放原始碼的 R 和 Python，Machine Learning Server 可與 R 和 Python 指令碼和函式相容。 它也可與 CRAN、pip 和 Conda 套件相容，以分析企業規模的資料。 

Machine Learning Server 會藉由新增資料的平行和區塊處理，解決開放原始碼 R 的記憶體內部限制問題。 所以您可以對遠大於主記憶體可負荷的資料量執行分析。 VM 隨附 Visual Studio Community。 它具有 Visual Studio R 工具和 Visual Studio Python 工具 (PTVS) 擴充功能，提供搭配 R 或 Python 使用的完整整合式開發環境 (IDE)。 我們也提供其他 IDE，例如 VM 上的 [RStudio](http://www.rstudio.com) 和 [PyCharm Community 版本](https://www.jetbrains.com/pycharm/)。 

### <a name="python"></a>Python
為了能夠使用 Python 進行開發，已安裝 Anaconda Python 散發套件 2.7 與 3.6。 這些散發套件具有基底 Python 以及大約 300 個最受歡迎的數學運算、工程設計和資料分析封裝。 您可以使用 Visual Studio Community 2017 內安裝的 PTVS。 或者，您可以使用其中一個隨附於 Anaconda 的 IDE，像是 IDLE 或 Spyder。 搜尋並啟動其中一個套件 (Win + S)。

> [!NOTE]
> 若要將「適用於 Visual Studio 的 Python 工具」指向 Anaconda Python 2.7，您必須為每個版本建立自訂的環境。 若要在 Visual Studio 2017 Community 中設定這些環境路徑，請瀏覽至 [工具] > [Python 工具] > [Python 環境]。 然後選取 [+ 自訂]。 
> 
> 

Anaconda Python 3.6 安裝在 **C:\Anaconda** 之下。 Anaconda Python 2.7 則安裝在 **C:\Anaconda\envs\python2** 之下。 如需詳細步驟，請參閱 [PTVS 文件](https://docs.microsoft.com/visualstudio/python/installing-python-interpreters)。 

### <a name="the-jupyter-notebook"></a>Jupyter Notebook
Jupyter Notebook 也隨附 Anaconda 散發套件，這是一個共用程式碼與分析的環境。 Jupyter Notebook 伺服器已預先設定 Python 2.7、Python 3.x、PySpark、Julia 及 R 核心。 若要啟動 Jupyter 伺服器和瀏覽器來存取 Notebook 伺服器，有一個名為 **Jupyter Notebook** 的桌面圖示。 

我們在 Python 和 r 中封裝數個範例 Notebook。在您存取 Jupyter 之後，Notebook 會示範如何使用下列技術：

* Machine Learning Server。
* SQL Server 機器學習服務、資料庫內分析。 
* Python。
* Microsoft 辨識工具組。
* Tensorflow。
* 其他 Azure 技術。 

當您使用在較早步驟中建立的密碼向 Jupyter Notebook 驗證之後，您可在 Notebook 首頁看到範例的連結。 

### <a name="visual-studio-community-2017"></a>Visual Studio Community 2017
VM 上安裝的 Visual Studio Community。 它是 Microsoft 提供的熱門 IDE 的免費版本，可供用於進行評估，且適合小型團隊。 請參閱[授權條款](https://www.visualstudio.com/support/legal/mt171547)。 

按兩下桌面圖示或 [開始] 功能表以開啟 Visual Studio。 搜尋程式 (Win + S)，後面接著 **Visual Studio**。 從那裡，您可以使用像是 C#、Python、R 及 node.js 等語言來建立專案。 已安裝的外掛程式方便您使用下列 Azure 服務：
* Azure 資料目錄
* Azure HDInsight Hadoop 和 Spark
* Azure Data Lake 

還有一個稱為 ```Visual Studio Tools for AI``` 的外掛程式，可緊密整合至 Azure Machine Learning 並協助您快速建置 AI 應用程式。 

> [!NOTE]
> 您可能會收到一則訊息，表示您的評估期間已過期。 請輸入您的 Microsoft 帳戶認證。 或建立新的免費帳戶，以取得 Visual Studio Community 的存取權。 
> 
> 

### <a name="sql-server-2017-developer-edition"></a>SQL Server 2017 Developer Edition
VM 上提供含機器學習服務的 SQL Server 2017 開發人員版本 (可使用 R 或 Python)，以執行資料庫內分析。 機器學習服務提供用於開發和部署智慧型應用程式的平台。 您可以使用這些語言和社群的許多封裝來建立模型，並為 SQL Server 資料產生預測。 您可以讓分析貼近資料，因為機器學習服務(資料庫內) 會整合 R 和 Python 語言與 SQL Server。 此整合可免除與資料移動相關聯的成本和安全性風險。

> [!NOTE]
> SQL Server Developer 版本只能用於進行開發和測試。 您需要授權，才能在生產環境中執行它。 
> 
> 

您可藉由啟動 Microsoft SQL Server Management Studio 來存取 SQL Server。 您的 VM 名稱會填入為 [伺服器名稱]。 在 Windows 上以系統管理員身分登入時，請使用 Windows 驗證。 當您在 SQL Server Management Studio 中，您可以建立其他使用者、建立資料庫、匯入資料以及執行 SQL 查詢。 

若要使用 SQL 機器學習服務啟用資料庫內分析，在以伺服器系統管理員身分登入後，請在 SQL Server Management Studio 中執行下列命令 (一次性動作)： 

        CREATE LOGIN [%COMPUTERNAME%\SQLRUserGroup] FROM WINDOWS 

        (Please replace the %COMPUTERNAME% with your VM name)


### <a name="azure"></a>Azure
有數個 Azure 工具會安裝於 VM 上：

* 前往 Azure SDK 文件的桌面捷徑。 
* **AzCopy** 用於在 Azure 儲存體帳戶中移入和移出資料。 若要查看使用情況，在命令提示字元中輸入 **Azcopy**。 
* 使用 **Azure 儲存體總管**來瀏覽您儲存在 Azure 儲存體帳戶中的物件。 它也會從 Azure 儲存體來回傳輸資料。 若要存取此工具，您可以在 [搜尋] 欄位中輸入**儲存體總管**。 或在 Windows [開始] 功能表上找到。 
* **Adlcopy** 會將資料移至 Azure Data Lake。 若要查看使用情況，請在命令提示字元中輸入 **adlcopy**。 
* **dtui**︰在 Azure Cosmos DB (雲端上的 NoSQL 資料庫) 上來回移動資料。 請在命令提示字元中輸入**dtui**。 
* **Azure Data Factory Integration Runtime** 可在內部部署資料來源與雲端之間移動資料。 其使用於 Azure Data Factory 之類的工具。 
* **Microsoft Azure PowerShell** 是利用 Powershell 指令碼語言來管理 Azure 資源的工具。 它也會安裝在 VM 上。 

### <a name="power-bi"></a>Power BI
為了協助您建立儀表板和視覺效果，已安裝 **Power BI Desktop**。 使用此工具以從不同來源提取資料，來撰寫您的儀表板和報告並發佈至雲端。 如需詳細資訊，請參閱 [Power BI](http://powerbi.microsoft.com) 網站。 您可以在 [開始] 功能表上找到 Power BI 桌面。 

> [!NOTE]
> 您需要 Microsoft Office 365 帳戶才能存取 Power BI。 
> 
> 

### <a name="azure-machine-learning-workbench"></a>Azure Machine Learning Workbench

Azure Machine Learning Workbench 是桌面應用程式和命令列介面。 Workbench 有內建資料準備功能，可在您採取您的資料準備步驟時進行學習。 並且提供專案管理、執行歷程記錄與筆記本整合來提高您的生產力。 

您可以使用開放原始碼架構 (包括 TensorFlow、Cognitive Toolkit、Spark ML 及 scikit-learn) 來開發您的模型。 在 DSVM 上，我們提供一個桌面圖示，可將 Azure Machine Learning Workbench 安裝至個別使用者的 **%LOCALAPPDATA%** 目錄。 

Workbench 的每個使用者都必須採取一次性動作。 按兩下 ```AzureML Workbench Setup``` 桌面圖示，以安裝 Workbench 的執行個體。 Azure Machine Learning 也會建立並使用在 **%LOCALAPPDATA%\amlworkbench\python** 目錄中擷取的每一使用者 Python 環境。

## <a name="more-microsoft-development-tools"></a>其他 Microsoft 開發工具
[Microsoft Web Platform Installer](https://www.microsoft.com/web/downloads/platform.aspx) 用來尋找並下載其他 Microsoft 開發工具。 另外在 Microsoft 資料科學虛擬機器桌面上也有提供工具的捷徑。  

## <a name="important-directories-on-the-vm"></a>VM 上的重要目錄
| Item | 目錄 |
| --- | --- |
| Jupyter Notebook 伺服器組態 | C:\ProgramData\jupyter |
| Jupyter Notebook 範例的主目錄 | c:\dsvm\notebooks 和 c:\users\<使用者名稱>\notebooks |
| 其他範例 | c:\dsvm\samples |
| Anaconda (預設值)︰Python 3.6 | c:\Anaconda |
| Anaconda Python 2.7 環境 | c:\Anaconda\envs\python2 |
| Microsoft Machine Learning Server (獨立式) Python | C:\Program Files\Microsoft\ML Server\PYTHON_SERVER |
| 預設 R 執行個體、Machine Learning Server (獨立式) | C:\Program Files\Microsoft\ML Server\R_SERVER |
| SQL 機器學習服務 (資料庫內) 執行個體目錄 | C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER |
| Azure Machine Learning Workbench (每位使用者) | %localappdata%\amlworkbench | 
| 其他工具 | c:\dsvm\tools |

> [!NOTE]
> 在 Windows Server 2012 版本的 DSVM 及 2018 年 3 月之前的 Windows Server 2016 版本上，預設的 Anaconda 環境是 Python 2.7。 次要環境是位於 **c:\Anaconda\envs\py35** 的 Python 3.5。 
> 
> 

## <a name="next-steps"></a>後續步驟

* 選取 [開始] 功能表，以探索資料科學 VM 上的工具。
* 若要了解 Azure 機器學習服務和 Azure Machine Learning Workbench，可瀏覽產品的[快速入門與教學課程頁面](../service/index.yml)。 
* 瀏覽至 **C:\Program Files\Microsoft\ML Server\R_SERVER\library\RevoScaleR\demoScripts**，以取得在 R 中使用 RevoScaleR 程式庫的範例，其支援企業規模的資料分析。  
* 閱讀文章：[您可以在 Data Science Virtual Machine 上做的十件事](http://aka.ms/dsvmtenthings)。
* 了解如何使用 [Team Data Science Process](../team-data-science-process/index.yml)，以系統化方式建置端對端分析方案。
* 瀏覽 [Azure AI 資源庫](http://gallery.cortanaintelligence.com)，可取得在 Azure 上使用 Azure Machine Learning 和相關資料服務的機器學習和資料分析範例。 我們也已經在虛擬機器的 [開始] 功能表與桌面上提供此資源庫的圖示。

