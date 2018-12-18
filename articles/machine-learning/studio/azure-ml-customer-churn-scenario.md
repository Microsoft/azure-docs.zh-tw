---
title: 使用機器學習分析客戶流失 | Microsoft Docs
description: 開發整合式模型以分析及評分客戶流失的案例研究
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 1333ffe2-59b8-4f40-9be7-3bf1173fc38d
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/18/2017
ms.openlocfilehash: 1beba951a6785aa90eef22a63a8064e9da1bb27f
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835112"
---
# <a name="analyzing-customer-churn-using-azure-machine-learning"></a>使用 Azure Machine Learning 分析客戶流失
## <a name="overview"></a>概觀
本文提供使用 Azure Machine Learning 建置客戶流失分析專案的參考實作。 在本文中將討論關聯的一般模型，可全面性地解決產業客戶流失的問題。 對於以機器學習建立的模型，我們也衡量其正確性，同時評定進一步開發的方向。  

### <a name="acknowledgements"></a>通知
這項實驗是由 Microsoft 的首席資料科學家 Serge Berger 和 Microsoft Azure Machine Learning 的前產品經理 Roger Barga 共同開發和測試。 Azure 文件小組高度認可其專業知識，並感謝他們分享這份白皮書。

> [!NOTE]
> 這項實驗中使用的資料無法公開使用。 如需如何建置客戶流失分析的機器學習模型範例，請參閱︰[Azure AI 資源庫](http://gallery.cortanaintelligence.com/)中的[售業客戶流失模型範本](https://gallery.cortanaintelligence.com/Collection/Retail-Customer-Churn-Prediction-Template-1)
> 
> 

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="the-problem-of-customer-churn"></a>客戶流失的問題
消費者市場和所有企業產業中的公司都必須處理客戶流失的問題。 有時，客戶流失太多會影響政策決定。 傳統的解決方案是預測流失傾向較高的客戶，透過特別禮遇、行銷活動或給予特殊待遇，滿足他們的需求。 這些作法隨著產業而不同， 甚至在同一產業的不同消費群之間也會有所不同 (例如電信業)。

共同點就是公司必須將這些額外的客戶維繫工作量降到最低。 因此，一般的作法是以流失機率來評比每位客戶，然後解決排名前 N 位的客戶。 排名前面的客戶可能是對公司最有利潤的客戶。 例如，在更複雜的情況下，選擇要給予特殊待遇的候選者時會採用利潤函數。 這些考量只不過是處理客戶流失的整體策略的一部分。 公司也需要考慮風險 (和相關的風險容忍度)、介入的程度和成本，以及合理的客戶區隔。  

## <a name="industry-outlook-and-approaches"></a>產業展望和作法
駕輕就熟地處理客戶流失是成熟產業的一項特徵。 電信產業是典型的例子，用戶更換供應商的頻率很高。 這種志願性流失是最主要問題所在。 再者，供應商對於「流失成因」 已累積大量經驗，亦即促成客戶更換供應商的因素。

例如，在行動電話業務中，手機或裝置選擇是導致客戶流失的一個明顯因素。 因此，常用的手段是補助新用戶的手機價格，但對既有客戶收取全額的升級費用。 根據歷史經驗，這種手段反而造成使客戶跳槽到另一家供應商來尋找新的折扣。 這使得供應商不得不修正策略

手機供應項目日新月異，是造成以最新手機款式為主的客戶流失模型迅速失效的一項因素。 此外，行動電話不只是通訊裝置，也是流行的代表 (例外 iPhone)。 這些社會性指標並不在一般的電信資料集之內。

建構模型的最後結果就是，單純地排除客戶流失的已知原因，就不可能設計出一套完美的策略。 事實上，持續性的模型建構策略是 **必要的**，包括量化類別變項的傳統模型 (例如決策樹)。

組織對客戶利用巨量資料集來進行巨量資料分析 (尤其是根據巨量資料的客戶流失偵測)，當做解決問題的有效手段。 您可以在＜建議＞中的 ETL 一節深入了解對付客戶流失問題的巨量資料作法。  

## <a name="methodology-to-model-customer-churn"></a>客戶流失模型建構方法
圖 1-3 顯示解決客戶流失時常用的問題解決流程。  

1. 風險模型可讓您考量動作如何影響機率和風險。
2. 介入模型讓您考量介入程度如何影響客戶流失機率和顧客終身價值 (CLV) 的數量。
3. 此分析本身會形成定性分析，再逐步發展成以客戶族群為目標的積極行銷活動，以提供最佳供應項目。  

![][1]

這個預視方法是應付流失最好的方法，但是會伴隨複雜性：我們必須開發多模型原型，並且追蹤模型之間的相依性。 模型之間的互動可封裝起來，如下圖所示：  

![][2]

*圖 4：多模型整合原型*  

如果要對客戶維繫提出一套全面性解決辦法，則不同模型之間的互動是關鍵。 每個模型必然隨著時間而退化；因此，架構是一種隱性的環路 (類似於以 CRISP-DM 資料採礦標準所設定的原型，[***3***])。  

風險-決策-行場區隔/分解的整個循環仍然是一般化結構，適用於解決許多商業問題。 客戶流失分析只是這群問題的一種很明顯的典型例子，因為它指出複雜商業問題的所有特徵，而這種問題無法以簡單的預測方案來解決。 方法中並未特別指出現代處理客戶流失辦法中的社會層面，社會層面封裝在模型原型中，就像在任何模型中一樣。  

此處增添的一項重要部分是巨量資料分析。 現今，電信業和零售業都竭盡所能地收集客戶的資料，可明顯預期銜接多重模型的需求將成為共同的趨勢，浮現的趨勢例如物聯網和普及化裝置，讓公司能夠將智慧型解決方案運用在許多層面上。  

 

## <a name="implementing-the-modeling-archetype-in-machine-learning-studio"></a>在 Machine Learning Studio 中實作模型原型
根據剛才說明的問題，實作一套整合的模型化和評分方法的最佳方式為何？ 在本節中，我們將示範如何利用 Azure Machine Learning Studio 來達成這項目的。  

針對客戶流失來設計整體原型時，多重模型方法不可或缺。 即使是作法中的評分 (預測) 部分也應該為多重模型。  

下圖顯示我們已建立的原型，在 Machine Learning Studio 中採用四種評分演算法來預測客戶流失。 使用多重模型方法的理由不只是為了建立集成分類器來提高準確性，也是為了避免過度訓練，同時改善制式的特徵選擇。  

![][3]

*圖 5：客戶流失模型建構方法的原型*  

以下幾節提供我們使用 Machine Learning Studio 實作的原型評分模型的詳細資料。  

### <a name="data-selection-and-preparation"></a>選擇和準備資料
用來建立模型和給客戶評分的資料取自於一個 CRM 垂直解決方案，為了保護客戶隱私，資料皆經過模糊處理。 資料包含美國 8,000 個訂用帳戶的相關資訊，並結合了三個來源：佈建資料 (訂用帳戶中繼資料)、活動資料 (系統的使用方式)，以及客戶支援資料。 資料不包含客戶的任何業務相關資訊；例如，不含忠誠度中繼資料或信用分數。  

為了簡單起見，ETL 和資料淨化處理不在討論範圍內，因為我們假設資料準備已在別處完成。   

模型的特徵選擇取決於預測變數組的初步顯著計分，包含在利用隨機森林模組的程序中。 在 Machine Learning Studio 中實作時，我們計算代表性特徵的平均值、中位數和範圍。 舉例來說，我們加上定性資料的總合，例如使用者活動的最小值和最大值。    

我們也取得最近 6 個月的暫時資訊。 我們分析一年的資料後，發現即使存在統計顯著趨勢，對客戶流失的效應也會在六個月後遞減。  

最重要的一點是整個過程 (包括 ETL、特徵選擇和模型化) 都在 Machine Learning Studio 中實作，並採用 Microsoft Azure 中的資料來源。   

下圖顯示使用的資料。  

![][4]

*圖 6：資料來源摘錄 (經過模糊處理)*  

![][5]

*圖 7：從資料來源擷取的特徵*
 

> 請注意，這項資料為私人所有，因此不能分享模型和資料。
> 如需使用公開可用資料的類似模型，請參閱 [Azure AI 資源庫](http://gallery.cortanaintelligence.com/)中的實驗範例：[電信公司客戶流失](http://gallery.cortanaintelligence.com/Experiment/31c19425ee874f628c847f7e2d93e383)。
> 
> 若要深入了解如何使用 Cortana Intelligence Suite 實作客戶流失分析模型，我們也推薦資深程式經理 Wee Hyong Tok 的 [這段影片](https://info.microsoft.com/Webinar-Harness-Predictive-Customer-Churn-Model.html) 。 
> 
> 

### <a name="algorithms-used-in-the-prototype"></a>原型中使用的演算法
我們採用下列四種機器學習演算法來建立原型 (沒有自訂)：  

1. 邏輯式迴歸 (LR)
2. 推進式決策樹 (BT)
3. 平均感知器 (AP)
4. 支援向量機器 (SVM)  

下圖顯示實驗設計介面的一部分，指出模型的建立順序：  

![][6]  

*圖 8：在 Machine Learning Studio 中建立模型*  

### <a name="scoring-methods"></a>評分方法
我們使用已標示的訓練資料集來對這四種模型評分。  

我們也提交評分資料集給使用 SAS Enterprise Miner 12 桌上型版本建立的比較模型。 我們測量 SAS 模型和所有四個 Machine Learning Studio 模型的正確性。  

## <a name="results"></a>結果
在本節中，我們根據評分資料集，提出關於模型正確性的調查結果。  

### <a name="accuracy-and-precision-of-scoring"></a>評分的正確性和準確度
一般而言，在 Azure Machine Learning 中實作的正確性低於 SAS 大約 10-15% (曲線下面積或 AUC)。  

不過，客戶流失中最重要的度量是分類誤判率；亦即，在分類器所預測排名前 N 位的流失客戶中，哪些實際上並 **沒有** 流失，但仍享受到特殊待遇？ 下圖比較所有模型的此項分類誤判率：  

![][7]

*圖 9：Passau 原型曲線下面積*

### <a name="using-auc-to-compare-results"></a>使用 AUC 來比較結果
曲線下面積 (AUC) 是一種度量，代表正負母體的計分分布之間「可分性」  的總體量測。 它類似傳統的「受測者操作特徵」(ROC) 圖形，但一項重要的差別是 AUC 度量不需要您選擇臨界值。 它會總結「所有」  可能選擇的結果。 反之，傳統 ROC 圖會在垂直軸顯示正比率，在水平軸顯示誤判率，而分類臨界值會變化。   

AUC 通常用來判斷不同演算法 (或不同系統) 是否有用處，因為它可根據 AUC 值來比較模型。 這在如氣象和生物科學等領域是常用的方法。 因此，AUC 是一種評估分類器表現的普遍工具。  

### <a name="comparing-misclassification-rates"></a>比較分類誤判率
我們使用大約 8,000 個訂用帳戶的 CRM 資料，在相關資料集上比較分類誤判率。  

* SAS 分類誤判率為 10-15%。
* Machine Learning Studio 對排名前 200-300 位流失客戶的分類誤判率為 15-20%。  

在電信業中，只提供特別禮遇或其他特殊待遇給最可能流失的客戶是很重要的。 在這方面，Machine Learning Studio 實作所獲得的成果與 SAS 模型旗鼓相當。  

基於同樣的理由，正確性比準確度更重要，因為我們最在意的是正確地將可能流失的客戶分類。  

下圖取自於維基百科，以生動、簡單明瞭的圖形來描述其中的關聯性：  

![][8]

*圖 10：正確性和準確度之間的取捨*

### <a name="accuracy-and-precision-results-for-boosted-decision-tree-model"></a>推進式決策樹模型的正確性與準確度結果
下列圖表針對推進式決策樹模型，列出利用 Machine Learning Studio 原型來評分所得出的原始結果，剛好就是四種模型中最正確的模型：  

![][9]

*圖 11：推進式決策樹模型的特性*

## <a name="performance-comparison"></a>表現比較
我們使用 Machine Learning Studio 模型和一個以 SAS Enterprise Miner 12.1 桌上型版本建立的比較模型，比較資料評分的速度。  

下表總結演算法的表現：  

*表 1.演算法的整體表現 (正確性)*

| LR | BT | AP | SVM |
| --- | --- | --- | --- |
| 平均模型 |最佳模型 |表現不佳 |平均模型 |

在執行速度方面，裝載於 Machine Learning Studio 中的模型比 SAS 快 15-25%，但正確性大致相等。  

## <a name="discussion-and-recommendations"></a>討論與建議
在電信業中，已出現許多作法來分析客戶流失，包括：  

* 導出四個基本類別的度量：
  * **實體 (例如訂用帳戶)**。 提供訂用帳戶及/或客戶 (客源流失主題的對象) 的基本資訊。
  * **活動**。 取得與實體相關的所有可能使用資訊，例如登入數目。
  * **客戶支援**。 從客戶支援記錄中取得資訊，指出訂用帳戶是否有問題或曾經與客戶支援的互動。
  * **競爭性和商務資料**。 取得與客戶有關的任何可能資訊 (例如可能無法取得或難以追蹤)。
* 使用重要性來引導特徵選擇。 這意味著推進式決策樹模型一定是可行的方法。  

使用這四個類別製造出一種假象，讓人誤以為簡單的「確定性」  分析方法 (根據在每個類別的合理因素上形成的跡象) 就足以發現有流失風險的客戶。 可惜，雖然這種想法看似可行，但卻是一種誤解。 原因在於客戶流失是一種短暫效應，而造成客戶流失的因素通常是暫時狀態。 今天導致客戶想要離開的原因，到了明天可能就不一樣，且六個月之後一定不同。 因此，有必要使用「機率」  模型。  

公司經常忽略這項重要的觀察，通常寧可選擇商業智慧導向的方法而不做分析，主要是因為前者較吸引人，直接了當就自動完成。  

然而，採用 Machine Learning Studio 進行自助式分析的價值在於，這四種資訊 (依部門分類) 將成為在客戶流失方面進行機器學習時的寶貴資料來源。  

Azure Machine Learning 中另一項吸引人的功能是可以將自訂模組加入既有預先定義之模組的儲存機制中。 這項功能基本上讓人有機會針對垂直市場選取程式庫和建立範本。 這是 Azure Machine Learning 在市場中脫穎而出的一項重要利器。  

我們希望未來繼續探討這個主題，特別是關於巨量資料分析。
  

## <a name="conclusion"></a>結論
本文說明一套實用的方法，採取通用的架構來處理客戶流失這個普遍的問題。 我們採用原型來給模型評分，並使用 Azure Machine Learning 來實作。 最後，我們依據 SAS 中可比較的演算法，評估原型解決方案的正確性和表現。  

 

## <a name="references"></a>參考
[1] Predictive Analytics: Beyond the Predictions (預測性分析：超出預測)，W. McKnight，資訊管理，2011 年 7 月/8 月，第 18-20 頁。  

[2] 維基百科文章：[正確性與準確度](http://en.wikipedia.org/wiki/Accuracy_and_precision)

[3] [CRISP-DM 1.0：資料採礦逐步指南](http://www.the-modeling-agency.com/crisp-dm.pdf)   

[4] [巨量資料行銷：更有效地吸引您的客戶和促進價值](http://www.amazon.com/Big-Data-Marketing-Customers-Effectively/dp/1118733894/ref=sr_1_12?ie=UTF8&qid=1387541531&sr=8-12&keywords=customer+churn)

[5] [Azure AI 資源庫](http://gallery.cortanaintelligence.com/)中的[電信公司客戶流失模型範本](http://gallery.cortanaintelligence.com/Experiment/Telco-Customer-Churn-5) 
 

## <a name="appendix"></a>附錄
![][10]

*圖 12：客戶流失原型的簡報擷取畫面*

[1]: ./media/azure-ml-customer-churn-scenario/churn-1.png
[2]: ./media/azure-ml-customer-churn-scenario/churn-2.png
[3]: ./media/azure-ml-customer-churn-scenario/churn-3.png
[4]: ./media/azure-ml-customer-churn-scenario/churn-4.png
[5]: ./media/azure-ml-customer-churn-scenario/churn-5.png
[6]: ./media/azure-ml-customer-churn-scenario/churn-6.png
[7]: ./media/azure-ml-customer-churn-scenario/churn-7.png
[8]: ./media/azure-ml-customer-churn-scenario/churn-8.png
[9]: ./media/azure-ml-customer-churn-scenario/churn-9.png
[10]: ./media/azure-ml-customer-churn-scenario/churn-10.png
