---
title: 資料科學特徵工程設計 | Microsoft Docs
description: 說明機器學習服務的資料增強程序中特性工程設計的目的，並提供其角色的範例。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 3fde69e8-5e7b-49ad-b3fb-ab8ef6503a4d
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/21/2017
ms.author: deguhath
ms.openlocfilehash: b4194ef5ab1c2c09206ea0acf78cb539bc2fc0b7
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836512"
---
# <a name="feature-engineering-in-data-science"></a>資料科學特徵工程設計
本文說明特徵工程設計的目的，並透過提供的範例，顯示它在機器學習服務的資料增強程序中扮演的角色。 用來說明此程序的範例是取自 Azure Machine Learning Studio。 

[!INCLUDE [cap-create-features-data-selector](../../../includes/cap-create-features-selector.md)]

這個**功能表**所連結的文章會說明如何在各種環境中建立資料的特徵。 此工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)中的一個步驟。

特徵工程設計藉由以協助學習程序的原始資料來建立特徵，以嘗試增加學習演算法的預測能力。 特徵的工程設計與選取是何謂 [Team Data Science Process 生命週期？](overview.md)中所概述 TDSP 程序的其中一部分。 特徵工程設計和選取屬於 TDSP 的 **開發特徵** 步驟。 

* **特性工程設計**：此程序嘗試從資料中的現有原始特性建立其他相關特性，以及增加學習演算法的預測功效。
* **特性選取**：此程序嘗試選取主要的原始資料特性子集，以縮小定型問題的維度。

通常會先套用**特性工程設計**以產生其他特定，然後執行**特性選取**步驟以排除不相關、多餘或高度相關的特性。

從收集的原始資料擷取特性，通常可以增強機器學習服務中使用的定型資料。 學習環境中工程設計特性範例，分類手寫字元的影像的方式是是建立從原始位元分配資料建構的位元密度對應。 相較於直接使用原始分配，此對應有助於更有效地找出字元的界限。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="create-features-from-your-data---feature-engineering"></a>從您的資料建立特徵 - 特徵工程設計
定型資料包含由範例所組成的矩陣 (資料列中儲存的記錄或 觀察)，而每個範例都有一組特性 (資料行中儲存的變數或欄位)。 在實驗設計中指定的特性預計會將資料中的模式特性化。 儘管許多原始資料欄位都可以直接包含在選取用來將模型定型的特性集中，但通常還是需要從原始資料中的特性建構其他 (經過工程設計的) 特性，才能產生增強的定型資料集。

應建立何種特性，才能在模型定型時增強資料集？ 經過工程設計的特性可增強定型，還會提供用以區分資料中模式的資訊。 我們期望新特徵可提供原始或現有特徵集中未清楚擷取或顯而易見的其他資訊。 但此程序是一種藝術。 健全且有建設性的決策通常需要一些網域.專業知識。

從 Azure 機器學習著手時，最簡單的方式是透過 Studio 中提供的範例具體地領會此程序。 以下呈現兩個範例：

* 在已知目標值的監督實驗中的 [單車租用數量預測](http://gallery.cortanaintelligence.com/Experiment/Regression-Demand-estimation-4) 迴歸範例
* 使用 [特性雜湊](https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/)

## <a name="example-1-add-temporal-features-for-a-regression-model"></a>範例 1：新增迴歸模型的暫時特徵
讓我們在 Azure Machine Learning Studio 中使用「單車的需求預測」實驗，示範如何設計迴歸工作的特性。 這項實驗的目標在於預測單車需求，也就是再特定月份/日期/小時內單車租用的數量。 資料集「單車租用 UCI 資料集」作為原始輸入資料使用。 此資料集是以在美國華盛頓特區維護單車出租網路的 Capital Bikeshare 公司所提供的實際資料為基礎。 此資料集代表 2011 年和 2012 年中特定一個小時內的單車租用數量，總共包含 17379 個資料列和 17 個資料行。 原始特性集包含天氣條件 (溫度/溼度/風速) 和當天的類型 (假日/工作日)。 要預測的欄位為 "cnt" 計數，代表特定小時內單車租用的計數，其範圍介於 1 到 977 之間。

為了達到在定型資料中建構有效特性的目的，會使用相同的演算法建立四個各有不同定型資料集的迴歸模型， 這四個資料集代表相同的原始輸入資料，但設定的特性數量增加。 這些特性可分為四類：

1. A = 預測日的天氣 + 假日 + 工作日 + 週末特性
2. B = 過去的 12 小時以來，每小時租出的單車數量
3. C = 過去的 12 天以來，每天在同一個時間租出的單車數量
4. D = 過去的 12 週以來，在同一天同一個時間租出的單車數量

除了已存在於原先未經處理資料中的特徵集 A 以外，其他三個特徵集都是透過特徵工程設計程序來建立。 特性集 B 會擷取最近的單車需求。 特性集 C 會擷取某一個小時的單車需求。 特性集 D 會擷取一週當中某一天某一個小時的單車需求。 四個定型資料集分別包含特性集 A、A+B、A+B+C 和 A+B+C+D。

在 Azure 機器學習實驗中，這四個定型資料集是透過預先處理的輸入資料集中的分支形成。 除了最左邊的分支以外，每個分支都包含[執行 R 指令碼](https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/)模組，衍生特徵 (特徵集 B、C 和 D) 是在模組中分別建構並附加至匯入的資料集。 下圖示範左邊第二個分支中用來建立特性集 B 的 R 指令碼。

![建立特性](./media/create-features/addFeature-Rscripts.png)

下表彙總了四個模型的效能結果比較： 

![結果比較](./media/create-features/result1.png)

特性 A+B+C 所呈現的結果最理想。 請注意，當定型資料中包含其他特性集時，錯誤率會降低。 這證實了我們的推測：特徵集 B、C 會針對迴歸工作提供其他相關資訊。 但新增 D 特性似乎不會讓錯誤率降低。

## <a name="example2"></a> 範例 2：在文字採礦中建立特性
特性工程設計廣泛運用於文字採礦的相關工作，例如文件分類和情感分析。 例如，當您想要將文件分為數個類別時，通常會假設包含在一個文件類別中的文字/片語比較不可能出現在其他文件類別中。 換言之，文字/片語分配的次數能夠描述不同文件類別的特徵。 在文字採礦應用程式中，因為個別的文字內容通常可作為輸入資料，所以建立文字/片語次數相關特性時需要特性工程設計程序。

為了達成此工作，會套用名為 **特性雜湊** 的技術，有效地將任意文字特性變成索引。 此方法不會將每個文字特性 (文字/片語) 關聯至特定索引，而是將雜湊函數套用至特性並直接使用其雜湊值作為索引。

Azure 機器學習中有一個 [特性雜湊](https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/) 模組，方便建立這些文字/片語特性。 下圖顯示使用此模組的範例。 輸入資料集包含兩個資料行：1 至 5 的書籍評比，以及實際評論內容。 此 [特性雜湊](https://msdn.microsoft.com/library/azure/c9a82660-2d9c-411d-8122-4d9e0b3ce92a/) 模組的目標在於擷取一些新特性，以顯示特定書籍評論中對應文字/片語的發生次數。 若要使用此模組，請完成下列步驟：

* 第一步，選取包含輸入文字的資料行 (此例中的 "Col2")。
* 第二步，將 "Hashing bitsize" 設定為 8，表示將建立 2^8=256 個特性。 所有文字中的文字/片語會雜湊至 256 個索引。 "Hashing bitsize" 參數的範圍是 1 至 31。 如果將此值設定為較大的數字，文字/片語比較不可能雜湊至相同的索引。
* 第三步，將 "N-grams" 參數設定為 2。 此值可從輸入文字中取得 unigrams (適用於每一個文字的特徵) 和 bigrams (適用於每一對相鄰文字的特徵) 的發生次數。 "N-grams" 參數的範圍是 0 至 10，這表示要包含在一個特性中的循序文字數目上限。  

![「特性雜湊」模組](./media/create-features/feature-Hashing1.png)

下圖顯示這些新特徵的外觀。

![「特性雜湊」範例](./media/create-features/feature-Hashing2.png)

## <a name="conclusion"></a>結論
工程設計和選取的特徵可提高下列定型過程的效率：嘗試擷取資料中內含的重要資訊。 此外，還可改善這些模型的功效，正確地分類輸入資料以及更精確地預測感興趣的結果。 特性工程設計和選取也可結合起來，讓學習更易於以運算方式處理。 其作法是提高而後減少校正或定型模型所需的特性數量。 從數學的角度來看，選取用來定型模型的特性是極小的一組獨立變數，可供解釋資料中的模式，然後成功地預測結果。

您不一定要執行特徵工程設計或特徵選取。 需要與否取決於所擁有或收集的資料、所選擇的演算法，以及實驗的目標。

