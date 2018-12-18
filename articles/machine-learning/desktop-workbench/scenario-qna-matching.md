---
title: 使用 Azure Machine Learning Workbench 的問與答比對 | Microsoft Docs
description: 如何使用各種有效的機器學習方法，來比對開放式查詢與既存的常見問題集問/答組合。
services: machine-learning
documentationcenter: ''
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/15/2017
ROBOTS: NOINDEX
ms.openlocfilehash: e0f6148e1fb28838bf99c63fbfbfbfe8cd127c8c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973196"
---
#  <a name="q--a-matching-using-azure-machine-learning-workbench"></a>使用 Azure Machine Learning Workbench 的問與答比對

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


開放式問題不容易回答，因此通常需要由主題專家 (SME) 親自回覆。 為了減少對於內部 SME 的需求，各家公司通常會建立常見問題集 (FAQ) 清單來協助使用者。 此範例會展示各種有效的機器學習方法，以供您比對開放式查詢與既存的常見問題集問/答組合。 此範例會示範簡單的開發程序，讓您了解如何使用 Azure Machine Learning Workbench 來建置這類解決方案。 

## <a name="link-to-the-gallery-github-repository"></a>連結至資源庫 GitHub 存放庫
[https://github.com/Azure/MachineLearningSamples-QnAMatching](https://github.com/Azure/MachineLearningSamples-QnAMatching)

## <a name="overview"></a>概觀

針對您在將使用者所提出的問題對應到既存之問題與解答 (問與答) 組合時所遇到的難題，此範例會提出解決之道，問題與解答組合通常會利用常見問題集 (也就是 FAQ) 清單來提供，或是利用網站 (例如 [Stack Overflow](https://stackoverflow.com/)) 上所顯示的問與答組合來提供。 有多種方法可為問題比對出正確的解答，例如尋找與問題最為類似的解答。 不過在此範例中，我們會藉由假設常見問題集中的每個答案都能回答多個有同樣語意的問題，而將開放式問題與先前詢問過的問題進行比對。

為了提供這個解決方案所需執行的主要步驟如下所示：

1. 清除並處理文字資料。
2. 了解具有資訊價值的片語，也就是在以序列檢視時會比單獨處理時提供更多資訊的多文字序列。
3. 擷取文字資料的特徵。
4. 訓練文字分類模型，並評估模型的效能。


## <a name="prerequisites"></a>必要條件

要執行此範例所需符合的必要條件如下：

1. [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
2. 遵循[快速入門安裝指南](quickstart-installation.md)所安裝的 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md) 版本，以便安裝程式並建立 Workbench。
3. 此範例可在任何計算環境中執行。 不過，建議將它執行於多核心機器上，且該機器至少有 16 GB 的記憶體和 5 GB 的磁碟空間。

## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中輸入「問與答比對」並選取範本
5.  按一下 [建立] 

## <a name="data-description"></a>資料說明

此範例所使用的資料集是 Stack Exchange 資料傾印，其儲存在 [archive.org](https://archive.org/details/stackexchange)。這項資料是 [Stack Exchange 網路](https://stackexchange.com/)上由所有使用者貢獻之內容的匿名傾印。 該網路中的每個網站都會格式化為個別的封存檔，而此封存檔含有 XML 檔案並且是透過 7-zip (使用 bzip2 壓縮) 所壓縮而成的。 每個網站封存檔都包含貼文、使用者、投票、註解、PostHistory 和 PostLinks。 

### <a name="data-source"></a>資料來源

此範例使用 Stack Overflow 網站上的 [Posts data (10 GB)](https://archive.org/download/stackexchange/stackoverflow.com-Posts.7z) 和 [PostLinks data (515 MB)](https://archive.org/download/stackexchange/stackoverflow.com-PostLinks.7z)。 如需完整的結構描述資訊，請參閱 [readme.txt](https://ia800500.us.archive.org/22/items/stackexchange/readme.txt)。 

Posts data 中的 `PostTypeId` 欄位會指出貼文是 `Question` 還是 `Answer`。 PostLinks data 中的 `PostLinkTypeId` 欄位則會指出兩個貼文是相關還是重複。 問題貼文通常會包含某些標籤，也就是用來將某個問題與其他類似/重複問題歸類在一起的關鍵字。 有些標籤的出現頻率很高，例如 `javascript`、`java`、`c#`、`php` 等等，因此會包含大量的問題貼文。 此範例會擷取一部分具有 `javascript` 標籤的問與答組合。

此外，一個問題貼文可能會與多個解答貼文或重複的問題貼文有關。 為了從這兩個資料集建構常見問題集清單，我們得考慮某些資料收集準則。 我們會使用 SQL 指令碼 (未放入此範例中) 選取三組編譯過的資料。 所產生之資料的說明如下：

- `Original Questions (Q)`：問題貼文是在 Stack Overflow 網站上詢問和回答。
- `Duplications (D)`：問題貼文與其他既有問題重複 (`PostLinkTypeId = 3`)，後者是原始的問題。 重複問題會被認為和原始問題有同樣的語意，因為針對原始問題所提供的解答同樣能回答新提出的重複問題。
- `Answers (A)`：每個原始問題和其重複問題可能會連結至多個解答貼文。 此範例只會選取原始作者 (利用 `AcceptedAnswerId` 來篩選) 所接受的解答貼文，或是原始問題中分數最高 (利用 `Score` 來排名) 的解答貼文。 

結合這三個資料集就會建立問與答組合，其中解答 (A) 會對應至一個原始問題 (Q) 和多個重複的問題 (D)，如下列資料圖所示：

<table><tr><td><img id ="data_diagram" src="media/scenario-qna-matching/data_diagram.png"></td></tr></table>

### <a name="data-structure"></a>資料結構

您可以在下表中找到三個資料集的資料結構描述和直接下載連結：

| Dataset | 欄位 | 類型 | 說明
| ----------|------------|------------|--------
| [問題](https://bostondata.blob.core.windows.net/stackoverflow/orig-q.tsv.gz) | id | 字串 | 問題的唯一識別碼 (主索引鍵)
|  | AnswerId | 字串 | 每個問題之解答的唯一識別碼
|  | Text0 | 字串 | 未經處理的文字資料，包括問題的標題和內文
|  | CreationDate | Timestamp | 提問時間戳記
| [重複問題](https://bostondata.blob.core.windows.net/stackoverflow/dup-q.tsv.gz) | id | 字串 | 重複問題的唯一識別碼 (主索引鍵)
|  | AnswerId | 字串 | 與重複問題相關聯的解答識別碼
|  | Text0 | 字串 | 未經處理的文字資料，包括重複問題的標題和內文
|  | CreationDate | Timestamp | 重複問題提問時間戳記
| [解答](https://bostondata.blob.core.windows.net/stackoverflow/ans.tsv.gz)  | id | 字串 | 解答的唯一識別碼 (主索引鍵)
|  | text0 | 字串 | 解答的未經處理文字資料


## <a name="scenario-structure"></a>案例結構

問與答比對範例會透過三種檔案來呈現。 第一種是一系列的 Jupyter 筆記本，以供顯示整個工作流程的逐步說明。 第二種是一組 Python 檔案，內含自訂 Python 模組以便學習片語和擷取特徵。 這些 Python 模組非常通用，不只適用於這個範例，也適用於其他使用案例。 第三種是一組 Python 檔案，這組檔案可使用 Azure Machine Learning Workbench 來微調超參數和追蹤模型效能。

此範例中的檔案整理如下。

| 檔案名稱 | 類型 | 說明
| ----------|------------|--------
| `Image` | 資料夾 | 此資料夾用於儲存讀我檔案的影像
| `notebooks` | 資料夾 | Jupyter 筆記本資料夾
| `modules` | 資料夾 | Python 模組資料夾
| `scripts` | 資料夾 | Python 檔案資料夾
| `notebooks/Part_1_Data_Preparation.ipynb` | Jupyter Notebook | 存取資料範例、對文字進行前置處理，並準備訓練和測試用的資料集
| `notebooks/Part_2_Phrase_Learning.ipynb` | Jupyter Notebook | 了解具有資訊價值的片語，並透過語彙基元將文字 (text) 轉化為以文字袋 (Bag-of-Words, BOW) 來表示
| `notebooks/Part_3_Model_Training_and_Evaluation.ipynb` | Jupyter Notebook | 擷取特徵、訓練文字分類模型，並評估模型的效能
| `modules/__init__.py` | Python 檔案 | Python 套件 init 檔案
| `modules/phrase_learning.py` | Python 檔案 | 用來轉換未經處理之資料並了解具有資訊價值之片語的 Python 模組
| `modules/feature_extractor.py` | Python 檔案 | Python 模組會擷取特徵來訓練模型
| `scripts/naive_bayes.py` | Python 檔案 | 用來在貝氏機率分類模型中微調超參數的 Python
| `scripts/random_forest.py` | Python 檔案 | 用來在隨機樹系模型中微調超參數的 Python
| `README.md` | Markdown 檔案 | Markdown 讀我檔案

> [!NOTE]
> 筆記本系列是在 Python 3.5 底下所建立。
> 

### <a name="data-ingestion-and-transformation"></a>資料擷取和轉換

三個已編譯的資料集會儲存在 Blob 儲存體，並於 `Part_1_Data_Preparation.ipynb` 筆記本中擷取。  

在訓練文字分類模型前，問題中的文字會先進行清除和前置處理以排除程式碼片段。 系統會對訓練材料套用非監督式片語學習，以了解具有資訊價值的多文字序列。 在文字分類模型所使用的下游文字袋 (Bag-of-Words, BOW) 特徵中，這些片語會以單一複合文字單位來表示。

文字前置處理和片語學習分別可以在筆記本的 `Part_1_Data_Preparation.ipynb` 和 `Part_2_Phrase_Learning.ipynb` 中找到詳細的逐步說明。

### <a name="modeling"></a>模型化

此範例的設計目的是要根據既有的問與答組合來為新的問題評分，其採用的方法是訓練文字分類模型，其中，每個既有的問與答組合都是唯一的類別，而且每個問與答組合都會提供一部分的重複問題來作為訓練材料。 在此範例中，我們保留了原始問題和 75% 的重複問題來進行訓練，並扣下張貼時間最近的 25% 重複問題來作為評估資料。

分類模型會使用整體方法來彙總三個基底分類器，包括**貝氏機率分類**、**支援向量機器**和**隨機樹系**。 每個基底分類器都會以 `AnswerId` 作為類別標籤，並以 BOW 表示法作為特徵。

`Part_3_Model_Training_and_Evaluation.ipynb` 中會說明模型訓練程序。

### <a name="evaluation"></a>評估

我們會使用兩個不同的評估計量數據來評估效能。 
1. `Average Rank (AR)`：指出在擷取的問與答組合清單 (一整組共有 103 個解答類別) 中找到正確解答時的平均位置。 
2. `Top 3 Percentage`：指出可以在傳回的排名清單中前三個選項擷取到正確解答的測試問題百分比。 

`Part_3_Model_Training_and_Evaluation.ipynb` 中會示範評估程序。


## <a name="conclusion"></a>結論

此範例重點說明如何使用已知的文字分析技術 (例如片語學習和文字分類) 來產生穩固的模型。 此範例也會展示 Azure Machine Learning Workbench 如何協助您開發互動式解決方案及追蹤模型的效能。 

此範例的一些重點精華如下：

- 問題與解答比對難題可利用片語學習和文字分類模型來有效地解決。
- 示範如何使用 Azure Machine Learning Workbench 和 Jupyter 筆記本來開發互動式模型。
- Azure Machine Learning Workbench 會記錄評估計量數據以供比較，藉此來管理執行歷程記錄和所學習到的模型。 這些特徵可讓您快速微調超參數，並協助您找出效能最佳的模型。


## <a name="references"></a>參考

Timothy J. Hazen, Fred Richardson, [_Modeling Multiword Phrases with Constrained Phrases Tree for Improved Topic Modeling of Conversational Speech_](http://people.csail.mit.edu/hazen/publications/Hazen-SLT-2012.pdf). Spoken Language Technology Workshop (SLT), 2012 IEEE. IEEE, 2012.

Timothy J. Hazen, [_MCE Training Techniques for Topic Identification of Spoken Audio Documents_](http://ieeexplore.ieee.org/abstract/document/5742980/) in IEEE Transactions on Audio, Speech, and Language Processing, vol. 19, no. 8, pp. 2451-2460, Nov. 2011.
