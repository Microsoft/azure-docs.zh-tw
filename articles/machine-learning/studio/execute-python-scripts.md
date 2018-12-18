---
title: 執行 Python 機器學習服務指令碼 | Microsoft Docs
description: 概述 Azure Machine Learning 中對於 Python 指令碼目前支援基礎之下的設計原則，以及基本使用案例、功能及限制。
keywords: python 機器學習服務,pandas,python pandas,python 指令碼, 執行 python 指令碼
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: ee9eb764-0d3e-4104-a797-19fc29345d39
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.openlocfilehash: 537839295deb631c3b9811c8d40db8608954e8a1
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835248"
---
# <a name="execute-python-machine-learning-scripts-in-azure-machine-learning-studio"></a>在 Azure Machine Learning Studio 中執行 Python 機器學習服務指令碼

本主題說明 Azure Machine Learning 中對於 Python 指令碼目前支援基礎之下的設計原則。 也說明了所提供的主要功能，包括：

- 執行基本的使用方式案例
- 為 Web 服務中的實驗評分
- 支援匯入現有的程式碼
- 匯出視覺效果
- 執行受監督的功能選取
- 了解部分限制

[Python](https://www.python.org/) 是許多資料科學家工具櫃中不可或缺的工具。 其中包含：

* 優雅簡潔的語法， 
* 跨平台支援， 
* 大量功能強大的程式庫集合，以及 
* 臻致完善的開發工具。 

Python 是在工作流程的所有階段中使用 (通常使用於機器學習模型化)：

- 資料內嵌和處理 
- 功能建構
- 模型訓練 
- 模型驗證
- 模型部署

Azure Machine Learning Studio 支援將 Python 指令碼內嵌至機器學習實驗的各個部分，並且順暢地將其發佈為 Microsoft Azure 上的 Web 服務。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]


## <a name="design-principles-of-python-scripts-in-machine-learning"></a>在 Machine Learning 中設計 Python 指令碼的原則

在 Azure Machine Learning Studio 中，存取 Python 的主要介面是透過圖 1 所示的[執行 Python 指令碼][execute-python-script]模組。

![image1](./media/execute-python-scripts/execute-machine-learning-python-scripts-module.png)

![image2](./media/execute-python-scripts/embedded-machine-learning-python-script.png)

圖 1. **執行 Python 指令碼** 模組。

Azure ML Studio 中的[執行 Python 指令碼][execute-python-script]模組最多接受三個輸入，而且最多產生兩個輸出 (於下一節中討論)，就像其同類的 R 一樣：[執行 R 指令碼][execute-r-script]模組。 要執行的 Python 程式碼會輸入至稱為 `azureml_main` 之特殊命名進入點函式的參數方塊。 以下是用來實作此模組的關鍵設計原則：

1. *必須是 Python 使用者慣用的。* 大多數 Python 使用者將其程式碼視為模組中的函式。 因此，將大量的可執行陳述式放在最上層的模組相當罕見。 因此，指令碼方塊也會採用特殊命名的 Python 函數，而不是採用一系列的陳述式。 在函式中公開的物件是標準 Python 程式庫類型 (例如 [Pandas](http://pandas.pydata.org/)資料框架和 [NumPy](http://www.numpy.org/) 陣列)。
2. *必須在本機和雲端執行之間具有高畫質。* 用來執行 Python 程式碼的後端是以 [Anaconda](https://store.continuum.io/cshop/anaconda/) (廣為使用的跨平台科學 Python 發行版本) 為基礎。 它隨附將近 200 個最常見的 Python 套件。 因此，資料科學家可在其本機 Azure Machine Learning 相容的 Anaconda 環境中，對程式碼進行偵錯與限定。 然後使用現有的開發環境 (例如 [IPython](http://ipython.org/) 筆記本或[適用於 Visual Studio 的 Python 工具](http://aka.ms/ptvs)) 來執行它，以作為 Azure ML 實驗的一部分執行。 此外，`azureml_main` 進入點是原始 Python 函式，因此****不需要 Azure ML 特定程式碼或安裝 SDK 即可編寫。
3. *必須可以順暢地與其他 Azure Machine Learning 模組組合。* [執行 Python 指令碼][execute-python-script]模組接受標準 Azure Machine Learning 資料集作為輸入和輸出。 底層架構明確而有效率地橋接了 Azure ML 和 Python 執行階段。 因此，Python 可以用於與現有 Azure ML 工作流程接合，包括呼叫 R 和 SQLite 的那些工作流程。 因此，資料科學家可以撰寫執行下列工作的工作流程：
   * 使用 Python 和 Pandas 進行資料前處理和清除
   * 將資料饋送至 SQL 轉換、聯結多個資料集以形成功能
   * 使用 Azure Machine Learning 中的演算法來訓練模型 
   * 使用 R 評估和後處理結果。


## <a name="basic-usage-scenarios-in-ml-for-python-scripts"></a>ML 中 Python 指令碼的基本使用案例

在本節中，我們調查[執行 Python 指令碼][execute-python-script]模組的一些基本用法。 對 Python 模組的任何輸入都會公開為 Pandas 資料框架。 函式必須傳回在 Python [序列](https://docs.python.org/2/c-api/sequence.html) (例如 tuple、清單或 NumPy 陣列) 內封裝的單一 Pandas 資料框架。 然後會在模組的第一個輸出連接埠中傳回此序列的第一個元素。 此配置顯示在「圖 2」中。

![image3](./media/execute-python-scripts/map-of-python-script-inputs-outputs.png)

圖 2. 將輸入連接埠對應至參數，並且將值傳回至輸出連接埠。

輸入連接埠如何對應至 `azureml_main` 函數的參數的更詳細語意如「表 1」所示：

![image1T](./media/execute-python-scripts/python-script-inputs-mapped-to-parameters.png)

表 1. 將輸入連接埠對應至函數參數。

輸入連接埠和函式參數之間的對應是有位置關係的：

- 第一個連接的輸入連接埠會對應至函式的第一個參數。 
- 第二個輸入連接埠 (如果連接) 會對應至函式的第二個參數。

如需 Python Pandas 以及如何以有效且有效率的方式使用它來處理資料的詳細資訊，請參閱 W. McKinney 所撰寫的 *Python for Data Analysis* (O'Reilly, 2012)。 


## <a name="translation-of-input-and-output-types"></a>輸入和輸出類型的轉譯 
Azure ML 中的輸入資料集會轉換為 Pandas 中的資料框架。 輸出資料框架會轉換回 Azure ML 資料集。 會執行下列轉換：

1. 字串和數值資料行會如現狀轉換，資料集中的遺漏值則會在 Pandas 中轉換為 ‘NA’ 值。 會在反向發生相同轉換 (Pandas 中的 NA 值會轉換為 Azure ML 中的遺漏值)。
2. Azure ML 不支援 Pandas 中的索引向量。 Python 函式中所有的輸入資料框架一律具有 64 位元的數值索引，範圍從 0 到資料列數目減 1。 
3. Azure ML 資料集無法具有重複的資料行名稱或非字串的資料行名稱。 如果輸出資料框架包含非數值資料行，則架構會在資料行名稱上呼叫 `str` 。 同樣地，任何重複的資料行名稱會自動錯位，以確保名稱是唯一的。 尾碼 (2) 會新增至第一個重複項目，尾碼 (3) 新增至第二個重複項目等等。


## <a name="operationalizing-python-scripts"></a>運作 Python 指令碼

當評分實驗中使用的任何[執行 Python 指令碼][execute-python-script]模組發佈為 Web 服務時，系統會呼叫這些模組。 例如，「圖 3」顯示計分實驗，其中包含用以評估單一 Python 運算式的程式碼。 

![image4](./media/execute-python-scripts/figure3a.png)

![image5](./media/execute-python-scripts/python-script-with-python-pandas.png)

圖 3. 用以評估 Python 運算式的 Web 服務。

從此實驗建立的 Web 服務：

- 會將 Python 運算式作為輸入 (字串)
- 會將它傳送給 Python 解譯器 
- 會傳回包含運算式以及評估結果的表格。


## <a name="importing-existing-python-script-modules"></a>匯入現有的 Python 指令碼模組

對於許多資料科學家，常見的使用案例是將現有 Python 指令碼併入 Azure ML 實驗。 [執行 Python 指令碼][execute-python-script]模組不需要將所有程式碼串連並貼到單一指令碼方塊中，而是在第三個輸入連接埠接受包含 Python 模組的 zip 檔案。 該檔案會由執行架構在執行階段解壓縮，且內容會新增至 Python 解譯器的程式庫路徑。 然後 `azureml_main` 進入點函式可以直接匯入這些模組。

作為範例，請考量包含簡單 “Hello, World” 函式的 Hello.py 檔案。

![image6](./media/execute-python-scripts/figure4.png)

圖 4. Hello.py 檔案中的使用者定義函式。

接下來，我們會建立包含 Hello.py 的 Hello.zip 檔案：

![image7](./media/execute-python-scripts/figure5.png)

圖 5. 包含使用者定義 Python 程式碼的 Zip 檔案。

將此 zip 檔案上傳至 Azure Machine Learning Studio 作為資料集。 然後，建立並執行使用 Hello.zip 檔案中之 Python 程式碼的實驗，方法是將它附加到**執行 Python 指令碼**模組的第三個輸入連接埠，如此圖所示。

![image8](./media/execute-python-scripts/figure6a.png)

![image9](./media/execute-python-scripts/figure6b.png)

圖 6. 範例實驗，具有上傳為 zip 檔案的使用者定義 Python 程式碼。

模組輸出顯示 zip 檔案已解除封裝，且函式 `print_hello` 已執行。
 
![image10](./media/execute-python-scripts/figure7.png)

圖 7. [執行 Python 指令碼][execute-python-script]模組內正在使用的使用者定義函式。


## <a name="working-with-visualizations"></a>使用視覺效果

[執行 Python 指令碼][execute-python-script]可以傳回以 MatplotLib 建立而可呈現在瀏覽器的繪圖。 但是繪圖不會在使用 R 時自動重新導向至映像。因此使用者必須明確地將任何繪圖儲存為 PNG 檔案，才能傳回至 Azure Machine Learning。 

若要從 MatplotLib 產生影像，您必須完成下列程序：

* 將後端從預設 Qt 型轉譯器切換為 “AGG” 
* 建立新的圖表物件 
* 取得軸並且對其產生所有繪圖 
* 將圖表儲存為 PNG 檔案 

以下的圖 8 會說明此程序，其使用 Pandas 中的 scatter_matrix 函式來建立散佈圖矩陣。

![image1v](./media/execute-python-scripts/figure-v1-8.png)

圖 8. 將 MatplotLib 圖表儲存為影像的程式碼。

圖 9 顯示的實驗會使用先前顯示的指令碼，透過第二個輸出連接埠傳回繪圖。

![image2v](./media/execute-python-scripts/figure-v2-9a.png) 

![image2v](./media/execute-python-scripts/figure-v2-9b.png) 

圖 9. 視覺化從 Python 程式碼產生的繪圖。

可藉由將多個圖表儲存為不同的影像以傳回它們，Azure Machine Learning 執行階段會挑選所有影像並加以串連，以產生視覺效果。


## <a name="advanced-examples"></a>進階範例

安裝在 Azure Machine Learning 中的 Anaconda 環境包含常見的套件，例如 NumPy、SciPy 和 Scikits-learn。 這些套件可以有效地用於機器學習管線中的各種資料處理工作。 作為範例，下列實驗和指令碼說明在 Scikits-Learn 中使用集成學習器以計算資料集的功能重要性分數。 然後，可以使用該分數先執行受監督的功能選取，再饋送至其他 ML 模型。

以下 Python 函式是用於根據分數計算重要性分數及將功能排序：

![image11](./media/execute-python-scripts/figure8.png)

圖 10. 用於依分數將功能排名的函式。
 然後下列實驗會在 Azure Machine Learning 中的 “Pima Indian Diabetes” 資料集計算及傳回功能的重要性分數：

![image12](./media/execute-python-scripts/figure9a.png)
![image13](./media/execute-python-scripts/figure9b.png)    

圖 11. 實驗，在 Pima Indian Diabetes 資料集中排名功能。

## <a name="limitations"></a>限制
[執行 Python 指令碼][execute-python-script]目前具有下列限制：

1. *沙箱化執行。* Python 執行階段目前已沙箱化，因此，不允許以永續方式存取網路或本機檔案系統。 所有本機儲存的文件都會被隔離，並且在模組結束時加以刪除。 Python 程式碼無法在其執行的機器上存取大部分的目錄，但目前的目錄及其子目錄例外。
2. *缺少精細的開發和偵錯支援。* Python 模組目前不支援 IDE 功能，例如 intellisense 和偵錯。 此外，如果模組在執行階段失敗，則會提供完整的 Python 堆疊追蹤。 但是，它必須在模組的輸出記錄檔中檢視。 我們目前建議您在如 IPython 的環境中開發 Python 指令碼及進行偵錯，然後將程式碼匯入到模組。
3. *單一資料框架輸出。* Python 進入點是唯一獲得允許的位置，可以將單一資料框架傳回為輸出。 目前無法直接將任意 Python 物件 (例如訓練模型) 傳回 Azure Machine Learning 執行階段。 如同[執行 R 指令碼][execute-r-script] (具有相同的限制)，在許多情況下可以將物件存放至位元組陣列，然後在資料框架內傳回。
4. *無法自訂 Python 安裝*。 目前，新增自訂 Python 模組的唯一方法是透過稍早所述的 zip 檔案機制。 對於小模組可行，但是對於大模組 (特別是具有原生 DLL 的模組) 和大量模組而言則顯得繁瑣。 

## <a name="conclusions"></a>結論
[執行 Python 指令碼][execute-python-script]模組可讓資料科學家將現有 Python 程式碼併入 Azure Machine Learning 中雲端託管的機器學習服務工作流程，並當作 Web 服務的一部分來順暢運作。 Python 指令碼模組可與 Azure Machine Learning 中的其他模組自然互通。 模組可以用於許多工作，從資料探索到預先處理和功能擷取，然後到結果評估與後續處理。 用於執行的後端執行階段是以 Anaconda 為根據，這是一個經過良好測試及廣泛使用的 Python 散佈。 此後端可讓您方便將現有程式碼資產加入雲端。

我們預期在[執行 Python 指令碼][execute-python-script]模組中提供其他功能，例如能夠在 Python 中訓練和運作模型，以及新增更好的支援在 Azure Machine Learning Studio 中開發和偵錯程式碼。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Python 開發人員中心](https://azure.microsoft.com/develop/python/)。

<!-- Module References -->
[execute-python-script]: https://msdn.microsoft.com/library/azure/cdb56f95-7f4c-404d-bde7-5bb972e6f232/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
