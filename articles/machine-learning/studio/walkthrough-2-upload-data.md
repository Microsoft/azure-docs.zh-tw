---
title: 步驟 2：將資料上傳至 Machine Learning 實驗中 | Microsoft Docs
description: 開發預測解決方案逐步解說步驟 2：將儲存的公用資料上傳至 Azure Machine Learning Studio 中。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 9f4bc52e-9919-4dea-90ea-5cf7cc506d85
ms.service: machine-learning
ms.component: studio
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2017
ms.openlocfilehash: dfa6ae8011da0299c270035b8b781d70a8e80119
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835785"
---
# <a name="walkthrough-step-2-upload-existing-data-into-an-azure-machine-learning-experiment"></a>逐步解說步驟 2：將現有資料上傳至 Azure Machine Learning 實驗中
這是 [在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)

1. [建立機器學習服務工作區](walkthrough-1-create-ml-workspace.md)
2. **上傳現有資料**
3. [建立新實驗](walkthrough-3-create-new-experiment.md)
4. [訓練及評估模型](walkthrough-4-train-and-evaluate-models.md)
5. [部署 Web 服務](walkthrough-5-publish-web-service.md)
6. [存取 Web 服務](walkthrough-6-access-web-service.md)

- - -
為了開發信用風險的預測模型，我們需要可以用於訓練和測試模型的資料。 針對此逐步教學，我們將使用 UCI Irvine Machine Learning Repository 中的「UCI Statlog (德國信用資料) 資料集」。 您可以在下列位置找到此儲存機制：  
<a href="http://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)">http://archive.ics.uci.edu/ml/datasets/Statlog+(German+Credit+Data)</a>

您可以使用名為 **german.data**的檔案。 將此檔案下載至您的本機硬碟。  

**german.data** 資料集包含過去 1000 名信用額度申請者的 20 個變數資料列。 這 20 個變數代表資料集的特徵集 (「特徵向量」)，可分別提供每個信用額度申請者的識別特性。 每個資料列另外會有一個資料行代表申請者計算後的信用風險，其中有 700 名申請者被認定為低信用風險，300 名為高風險。

UCI 網站提供了這項資料的特徵向量的屬性描述。 包括財務資訊、信用歷史記錄、工作狀態、個人資訊。 每個申請者都會有一個二進位評等，指出他們屬於低信用風險還是高風險。 

我們將使用這項資料來訓練預測分析模型。 完成之後，我們的模型應能夠接受新申請者的特徵向量，並預測他或她是屬於低信用風險還是高風險。  

以下提供一個有趣的論點。 UCI 網站上的資料集描述提及，如果我們錯誤分類一個人的信用風險需付出何種代價。
如果模型將某個實際為低信用風險的人預測為高信用風險，則此模型做了錯誤分類。
但反向的錯誤分類對金融機構而言需要付出五倍的代價：如果模型將某個實際為高信用風險的人預測為低信用風險。

因此，我們想要訓練模型，讓後者的這一個錯誤分類類型的成本比另一種錯誤分類方式高出五倍。
在我們的實驗中，在訓練模型時執行此動作的一個簡單方式是，複製 (五次) 那些代表某個具有高信用風險之人員的項目。 然後，如果模型將某人錯誤分類為低信用風險，但他們實際為高風險時，模型即會會進行該相同錯誤分類五次，針對每個重複項目進行一次。 這會在訓練結果中增加此誤差的成本。


## <a name="convert-the-dataset-format"></a>轉換資料集格式
原始資料集使用以空格分隔的格式。 Machine Learning Studio 在使用逗號分隔值 (CSV) 檔案時更能適當運作，因此我們將以逗號取代空格，進行資料集轉換。  

有許多方法可以轉換此資料。 其中一種是使用下列的 Windows PowerShell 命令：   

    cat german.data | %{$_ -replace " ",","} | sc german.csv  

另一種方法是使用 Unix Sed 命令：  

    sed 's/ /,/g' german.data > german.csv  

在任一案例中，我們已經以名為 **german.csv** 的檔案建立逗號分隔版本的資料，可在我們的實驗中加以使用。

## <a name="upload-the-dataset-to-machine-learning-studio"></a>將資料集上傳至 Machine Learning Studio
在資料轉換為 CSV 格式後，我們必須將其上傳至 Machine Learning Studio 中。 

1. 開啟 Machine Learning Studio 首頁 ([https://studio.azureml.net](https://studio.azureml.net))。 

2. 按一下視窗左上角的功能表![功能表][1]，按一下 [Azure Machine Learning]，選取 [Studio] 然後登入。

3. 按一下視窗底部的 [ **+新增** ]。

4. 選取 [ **資料集**]。

5. 選取 [ **從本機檔案**]。

    ![從本機檔案新增資料集][2]

6. 在 [上傳新的資料集] 對話方塊中，按一下 [瀏覽]，然後尋找您建立的 **german.csv** 檔案。

7. 輸入資料集的名稱。 在此逐步解說中，將它稱為 "UCI German Credit Card Data"。

8. 針對資料類型，請選取 **不具標頭的一般 CSV 檔案 (.nh.csv)**。

9. 視需要新增說明。

10. 按一下 **[確定]** \(打勾記號)。  

    ![上傳資料集][3]

這會將資料上傳至我們可在實驗中使用的資料集模組。

您可以管理您已上傳至 Studio 的資料集，請按一下 Studio 視窗左側的 [資料集] 索引標籤。

![管理資料集][4]

如需將其他種資料類型匯入實驗的詳細資訊，請參閱[將訓練資料匯入 Azure Machine Learning Studio](import-data.md)。

**下一步：[建立新實驗](walkthrough-3-create-new-experiment.md)**

[1]: media/walkthrough-2-upload-data/menu.png
[2]: media/walkthrough-2-upload-data/add-dataset.png
[3]: media/walkthrough-2-upload-data/upload-dataset.png
[4]: media/walkthrough-2-upload-data/dataset-list.png
