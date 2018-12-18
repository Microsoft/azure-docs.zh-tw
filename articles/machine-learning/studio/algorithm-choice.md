---
title: 如何選擇機器學習演算法 | Microsoft Docs
description: 如何在叢集、分類或迴歸實驗中選擇適用於經指導和未經指導學習的 Azure Machine Learning Studio 演算法。
services: machine-learning
author: pakalra
ms.author: pakalra
manager: cgronlun
editor: cgronlun
tags: ''
ms.assetid: a3b23d7f-f083-49c4-b6b1-3911cd69f1b4
ms.service: machine-learning
ms.component: studio
ms.topic: article
ms.date: 12/18/2017
ms.openlocfilehash: 92db2a86f5f698945cd2252476e951d658dc75ac
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959201"
---
# <a name="how-to-choose-algorithms-for-azure-machine-learning-studio"></a>如何選擇 Azure Machine Learning Studio 的演算法

「我該使用何種機器學習演算法？ 」的答案永遠都是「視情況。 」 這可視資料的大小、品質和本質而定。 也可取決於您想用這個答案來做些什麼。 或是取決於演算法的數學運算如何針對您正在使用的電腦轉譯成指令。 而這又需視您有多少時間。 即使經驗最豐富的資料科學家，在沒有嘗試之前，也無法確認哪一個演算法效果會最好。

## <a name="the-machine-learning-algorithm-cheat-sheet"></a>機器學習演算法小祕技

**Microsoft Azure Machine Learning 演算法速查表**可協助您從 Azure Machine Learning Studio 演算法資源庫中選擇適合您預測性分析解決方案的機器學習演算法。
本文將引導您如何使用它。

> [!NOTE]
> 若要下載小祕技，並搭配本文使用，請移至 [適用於 Microsoft Azure Machine Learning Studio 的機器學習演算法小祕技](algorithm-cheat-sheet.md)。
> 
> 

請記住，這份小祕技有非常特定的預設對象：一位剛起步的資料科學家，其機器學習的經驗為大學生程度，正試著在 Azure Machine Learning Studio 中選擇要開始使用的演算法。 這表示小祕技可能會比較概括且過於簡化，但它為您指引一個可靠的方向。 同時這也意味著還有許多演算法並未列入其中。 當 Azure 機器學習成長到擁有一組更完整的可用方法時，我們就會新增這些演算法。

這些建議是收集許多資料科學家與機器學習專家的意見反應和提示所編撰而成。 雖然我們的想法並不一致，但我已試著將我們的意見整理成粗略的共識。 而大部分的爭論其實都具有同一個考量：「視情況而定。」

### <a name="how-to-use-the-cheat-sheet"></a>如何使用小祕技

請將圖表上的路徑和演算法標籤解讀為「如果需要*&lt;路徑標籤&gt;* 則使用*&lt;演算法&gt;*。」 例如「如果需要 *speed* (速度)，則使用 *two class logistic regression* (雙類別羅吉斯迴歸)。」 有時候適用於多個分支。
有時候則不完全適用。 這些建議通常是來自經驗法則，因此不必擔心是否準確。
我和一些資料科學家討論過，他們都認為唯有全部試用一次，才能找出最佳的演算法。

以下是 [Azure AI 資源庫](http://gallery.cortanaintelligence.com/)中的實驗範例，該實驗對相同的資料嘗試數種演算法，並比較其結果：[比較多類別分類器：字母辨識](http://gallery.cortanaintelligence.com/Details/a635502fc98b402a890efe21cec65b92)。

> [!TIP]
> 若要下載並列印提供 Machine Learning Studio 功能概觀的圖表，請參閱 [Azure Machine Learning Studio 功能的概觀圖](studio-overview-diagram.md)。
> 
> 

## <a name="flavors-of-machine-learning"></a>機器學習的類型

### <a name="supervised"></a>監督式

監督式學習演算法會根據一組範例做出預測。 例如，利用歷史股價來大膽猜測未來的價格。 用於定型的各個範例都會標上需要關注的值，在這裡指的就是股價。 監督式學習演算法會在這些值標籤中尋找模式。 它可以使用任何可能相關的資訊 (星期幾、季度、公司的財務資料、產業類型、是否有破壞性的地緣政治事件等)，然後每個演算法就會尋找不同類型的模式。 當演算法找到最佳模式之後，它會使用這種模式為沒有標示的測試資料 (也就是未來的股價) 做出預測。

監督式學習是常見且實用的機器學習類型。 除了一個例外之外，Azure Machine Learning 中的所有模組都是監督式學習演算法。 Azure 機器學習中有幾個代表性的特定監督式學習類型：分類、迴歸和異常偵測。

* **分類**。 當資料用來預測類別時，這種監督式學習也稱為分類。 將影像指定為 'cat' 或 'dog' 的圖片便屬這種情況。 如果只有兩個選擇，則稱作**雙類別**或**二項式分類**。 如果有多個類別，例如預測 NCAA 季後賽的優勝隊伍，則這個問題就稱為 **多類別分類**。
* **迴歸**。 如果要預測值，例如股價，這種監督式學習稱為迴歸。
* **異常偵測**。 有時候它的目的只是要找出異常的資料點。 例如在偵測詐騙時，只要是極不尋常的信用卡消費模式都有嫌疑。 由於詐騙可能產生的變化過多，而定型的範例過少，因此難以學習何謂詐騙活動。 異常偵測採用的方法，只能使用非詐騙交易的歷史記錄來了解何謂正常活動，並找出與正常活動明顯不同的情況。

### <a name="unsupervised"></a>未監督式

在未監督的學習中，資料點沒有與其相關聯的標籤。 然而，未經指導的學習演算法的目標在於以某種方式組織資料或描述其結構。 這種方式可能是將資料劃分為叢集，或尋找各種查看複雜資料的方式，讓資料變得更簡單或更整齊。

### <a name="reinforcement-learning"></a>增強式學習

在增強式學習中，演算法需要選擇一個動作來回應每個資料點。 此學習演算法也會在短時間內收到獎勵訊號，指出決策的好壞程度。
演算法會據此修改其策略，以達到最高的獎勵。 Azure 機器學習中目前沒有增強式學習演算法模組。 增強式學習是機器人領域中的常見方法，其中在某個時間點的感應器讀數集就是一個資料點，而演算法必須選擇機器人的下一個動作。 它的性質也很適合物聯網應用。

## <a name="considerations-when-choosing-an-algorithm"></a>選擇演算法時的考量

### <a name="accuracy"></a>精確度

您不一定常常需要取得最準確的答案。
視您的用途而定，有時候近似值便已足夠。 如果是這樣，您就能採用近似法，並大幅縮短處理時間。 近似法的另一項優點是，它們會自然傾向於避免 [過度學習](https://youtu.be/DQWI1kvmwRg)。

### <a name="training-time"></a>定型時間

定型出一個模型可能需要幾分鐘或幾小時，這在各個演算法間有很大的差異。 定型時間通常取決於精確度，這兩者的關係密不可分。 此外，有些演算法對資料點的數目較為敏感。
如果有時間限制，就可以促使演算法做出選擇 (尤其是資料集很大時)。

### <a name="linearity"></a>線性

許多機器學習演算法都會使用線性。 線性分類演算法會假設可以直線 (或較高維度類比) 分隔類別。 這些演算法包括羅吉斯迴歸和支援向量機器 (如同 Azure 機器學習中所實作)。
線性迴歸演算法會假設資料趨勢依循著一條直線。 這類假設對某些問題而言還不錯，但在其他問題上會降低精確度。

![非線性類別界限][1]

***非線性類別界限*** *- 依賴線性分類演算法會造成低精確度的結果*

![具有非線性趨勢的資料][2]

***具有非線性趨勢的資料*** *：使用線性迴歸方法會產生較大且不必要的誤差*

儘管有風險，線性演算法對於首次攻擊而言仍是一種非常熱門的方式。 這種演算法定型起來通常又快又簡單。

### <a name="number-of-parameters"></a>參數數目

參數是資料科學家在設定演算法時的必經之路。 參數就是會影響演算法行為的數值，例如容錯或反覆運算次數，或是演算法運作方式的變化選項。 定型時間和演算法的精確度有時候很容易因為設定是否正確而受到影響。 一般而言，具有大量參數的演算法需要最多次的反覆試驗，才能找出好的組合。

或者，Azure 機器學習中有 [參數掃掠](algorithm-parameters-optimize.md) 模組區塊，會依照您選擇的細微性，自動嘗試所有參數組合。 雖然這是確認是否定義出參數空間的好方法，但定型模型時所需的時間，仍會以指數方式隨著參數數目而增加。

一般而言，具有許多參數的優點是可讓演算法有更大的彈性。 這通常可以達到很高的精確度。 不過前提是您得為參數設定找到正確的組合。

### <a name="number-of-features"></a>特徵數目

就特定的資料類型而言，可能會有比資料點數目更龐大的特徵數目。 基因學或文字資料通常屬於這種情況。 大量的特徵會使一些學習演算法陷入膠著，讓定型時間長到無法作業。 支援向量機器就特別適合此情況 (請參閱下述)。

### <a name="special-cases"></a>特殊案例

有些學習演算法會對資料結構或想要的結果做出特定假設。 如果可以找到符合需求的假設，您就能獲得更實用的結果、更精確的預測或更快的定型時間。

| **演算法** | **精確度** | **定型時間** | **線性** | **參數** | **注意事項** |
| --- |:---:|:---:|:---:|:---:| --- |
| **雙類別分類** | | | | | |
| [羅吉斯迴歸](https://msdn.microsoft.com/library/azure/dn905994.aspx) | |● |● |5 | |
| [決策樹系](https://msdn.microsoft.com/library/azure/dn906008.aspx) |● |○ | |6 | |
| [決策叢林](https://msdn.microsoft.com/library/azure/dn905976.aspx) |● |○ | |6 |低記憶體使用量 |
| [促進式決策樹](https://msdn.microsoft.com/library/azure/dn906025.aspx) |● |○ | |6 |高記憶體使用量 |
| [類神經網路](https://msdn.microsoft.com/library/azure/dn905947.aspx) |● | | |9 |[支援其他自訂項目](http://go.microsoft.com/fwlink/?LinkId=402867) |
| [平均感知器](https://msdn.microsoft.com/library/azure/dn906036.aspx) |○ |○ |● |4 | |
| [支援向量機器](https://msdn.microsoft.com/library/azure/dn905835.aspx) | |○ |● |5 |適用於大型特徵集 |
| [本機深度支援向量機器](https://msdn.microsoft.com/library/azure/dn913070.aspx) |○ | | |8 |適用於大型特徵集 |
| [貝氏點機器](https://msdn.microsoft.com/library/azure/dn905930.aspx) | |○ |● |3 | |
| **多類別分類** | | | | | |
| [羅吉斯迴歸](https://msdn.microsoft.com/library/azure/dn905853.aspx) | |● |● |5 | |
| [決策樹系](https://msdn.microsoft.com/library/azure/dn906015.aspx) |● |○ | |6 | |
| [決策叢林 ](https://msdn.microsoft.com/library/azure/dn905963.aspx) |● |○ | |6 |低記憶體使用量 |
| [類神經網路](https://msdn.microsoft.com/library/azure/dn906030.aspx) |● | | |9 |[支援其他自訂項目](http://go.microsoft.com/fwlink/?LinkId=402867) |
| [one-v-all](https://msdn.microsoft.com/library/azure/dn905887.aspx) |- |- |- |- |請參閱選取的兩個類別方法的屬性 |
| **迴歸** | | | | | |
| [線性](https://msdn.microsoft.com/library/azure/dn905978.aspx) | |● |● |4 | |
| [貝氏線性](https://msdn.microsoft.com/library/azure/dn906022.aspx) | |○ |● |2 | |
| [決策樹系](https://msdn.microsoft.com/library/azure/dn905862.aspx) |● |○ | |6 | |
| [促進式決策樹](https://msdn.microsoft.com/library/azure/dn905801.aspx) |● |○ | |5 |高記憶體使用量 |
| [快速樹系分量](https://msdn.microsoft.com/library/azure/dn913093.aspx) |● |○ | |9 |分佈而不是點預測 |
| [類神經網路](https://msdn.microsoft.com/library/azure/dn905924.aspx) |● | | |9 |[支援其他自訂項目](http://go.microsoft.com/fwlink/?LinkId=402867) |
| [波氏](https://msdn.microsoft.com/library/azure/dn905988.aspx) | | |● |5 |技術上為對數線性。 針對預測計算 |
| [序數](https://msdn.microsoft.com/library/azure/dn906029.aspx) | | | |0 |針對預測順位排序 |
| **異常偵測** | | | | | |
| [支援向量機器](https://msdn.microsoft.com/library/azure/dn913103.aspx) |○ |○ | |2 |特別適用於大型特徵集 |
| [PCA 型異常偵測](https://msdn.microsoft.com/library/azure/dn913102.aspx) | |○ |● |3 | |
| [K-means](https://msdn.microsoft.com/library/azure/5049a09b-bd90-4c4e-9b46-7c87e3a36810/) | |○ |● |4 |叢集演算法 |

**演算法屬性：**

**●** ：顯示優異的精確度、快速定型時間及使用線性

**○** ：顯示不錯的精確度和適度的定型時間

## <a name="algorithm-notes"></a>演算法備註

### <a name="linear-regression"></a>線性迴歸

如同之前提到， [線性迴歸](https://msdn.microsoft.com/library/azure/dn905978.aspx) 會使一條線 (或平面或超平面) 符合資料集。 它是常被使用的主力，簡單又快速，但可能會過度簡化某些問題。
請查看這裡的 [線性迴歸教學課程](linear-regression-in-azure.md)。

![具有線性趨勢的資料][3]

***具有線性趨勢的資料***

### <a name="logistic-regression"></a>羅吉斯迴歸

雖然名稱中的「迴歸」讓人感到有點奇怪，但羅吉斯迴歸對[雙類別](https://msdn.microsoft.com/library/azure/dn905994.aspx)和[多類別](https://msdn.microsoft.com/library/azure/dn905853.aspx)分類而言，其實是很強大的工具。 它既快速又簡單。 它事實上會使用 'S' 形的曲線而不是一條直線，這使它在性質上很適合將資料分組。 羅吉斯迴歸提供線性類別界限，因此使用它時，請確定線性近似值是您能接受的結果。

![羅吉斯迴歸與只有一項特徵的雙類別資料][4]

***羅吉斯迴歸與只有一項特徵的雙類別資料*** *- 類別界限的點就是羅吉斯曲線接近這兩個類別的地方*

### <a name="trees-forests-and-jungles"></a>樹、樹系和叢林

決策樹系 ([迴歸](https://msdn.microsoft.com/library/azure/dn905862.aspx)、[雙類別](https://msdn.microsoft.com/library/azure/dn906008.aspx)和[多類別](https://msdn.microsoft.com/library/azure/dn906015.aspx))、決策叢林 ([雙類別](https://msdn.microsoft.com/library/azure/dn905976.aspx)和[多類別](https://msdn.microsoft.com/library/azure/dn905963.aspx)) 以及促進式決策樹 ([迴歸](https://msdn.microsoft.com/library/azure/dn905801.aspx)和[雙類別](https://msdn.microsoft.com/library/azure/dn906025.aspx))，都是以基本的機器學習概念「決策樹」做為基礎。 決策樹有許多變化，但是用途都相同：將特徵空間細分成區域，這些區域大多具有相同的標記。 根據您是執行分類或迴歸而定，這些區域可能會有一致的類別或常數值。

![細分特徵空間的決策樹][5]

***此決策樹將特徵空間細分為值大致統一的區域***

由於特徵空間可以任意細分成較小的區域，因此會很容易推斷出每個區域都能細分成只有一個資料點。 而這就是過度學習的極端範例。 若要避免這個問題，建構一大組樹需要採取特殊的數學計算方式，也就是讓樹與樹之間沒有相互關聯。 這種「決策樹系」的平均就是可避免過度學習的樹。 決策樹系會使用大量記憶體。 決策叢林則是使用較少記憶體的變體，但代價是定型時間較長。

促進式決策樹可藉由限制細分的次數，以及每個區域中允許的最少資料點，來避免過度學習。 此演算法會建構一連串的樹，其中每個樹都會學習彌補前一個樹所留下來的錯誤。 這種學習方式雖然非常精確，但通常會使用大量記憶體。 如需完整的技術說明，請參閱 [Friedman 的原始文件](http://www-stat.stanford.edu/~jhf/ftp/trebst.pdf)。

[快速樹系分量迴歸](https://msdn.microsoft.com/library/azure/dn913093.aspx) 是決策樹的變化之一，適用的特殊案例是，您不只想要知道區域內資料的一般 (中位數) 值，還想知道資料在分量形式中的分佈。

### <a name="neural-networks-and-perceptrons"></a>類神經網路和感知器

類神經網路是受大腦啟發的學習演算法，其中涵蓋[多類別](https://msdn.microsoft.com/library/azure/dn906030.aspx)、[雙類別](https://msdn.microsoft.com/library/azure/dn905947.aspx)和[迴歸](https://msdn.microsoft.com/library/azure/dn905924.aspx)問題。 雖然它們具有無限的變化，但 Azure 機器學習內的類神經網路都是定向非循環圖的形式。 這表示輸入的特徵在轉換成輸出前，會在一連串的層中一直向前傳遞，且永遠不會向後。 在每個層中，會以各種組合加權輸入、再加以加總，然後傳遞到下一個層。 這種簡單的計算組合能夠學習複雜的類別界限和資料趨勢，就好像魔術一樣。 這類擁有許多層的網路會執行所謂的「深度學習」，聽起來很適合當作科技報導和科幻小說題材的能力。

可惜這種高效能並非隨手可得。 類神經網路需要用很長的時間來定型，特別是有許多特徵的大型資料集。 它們也比大部分的演算法具有更多參數，這表示參數掃掠會大幅延長定型時間。
但對於那些想要 [自行指定專屬網路結構](http://go.microsoft.com/fwlink/?LinkId=402867)的佼佼者而言，它們所賦予的可能性將會是取之不盡的。

![類神經網路所學到的界限][6]
***類神經網路所學到的界限可能會相當複雜且不規則***

[雙類別的平均感知器](https://msdn.microsoft.com/library/azure/dn906036.aspx) 是可以急速定型的類神經網路。 它使用的網路結構會提供線性類別界限。 以現今的標準來看，它似乎過於簡略，但它長期以來具有穩定運作的歷史記錄，而且小到足以快速學會。

### <a name="svms"></a>SVM

支援向量機器 (SVM) 會尋找使用盡可能寬的邊界來分隔類別的界限。 如果無法清楚地分隔兩個類別，演算法就會盡量找出最佳界限。 如同 Azure 機器學習中所述，[雙類別 SVM](https://msdn.microsoft.com/library/azure/dn905835.aspx) 會以一條直線來尋找 (但以 SVM 的說法，應該是使用線性核心)。因為使用這種線性近似值，所以能夠相當快速地執行。 可以真正發揮它功效的地方是特徵密集的資料，例如文字或基因資料。 在這些情況中，SVM 能比其他大部分的演算法更快區隔出類別，也較不容易過度學習，只不過它需要適度的記憶體量才能執行。

![支援向量機器類別界限][7]

***典型的支援向量機器類別界限，會將分隔兩個類別的邊界最大化***

另一個 Microsoft Research 的產品 [雙類別本機深度 SVM](https://msdn.microsoft.com/library/azure/dn913070.aspx) 是 SVM 的非線性變體，能夠保留線性版本中大部分的速度和記憶體效率。 當線性方法的答案不夠精確時，就非常適合使用它。 開發人員藉由將問題分成一堆小型線性 SVM 問題，讓它能保持快速。 請閱讀[完整說明](http://proceedings.mlr.press/v28/jose13.html) ，其中有如何完成這個技巧的詳細資料。

使用聰明的非線性 SVM 延伸模組 (即 [單一類別 SVM](https://msdn.microsoft.com/library/azure/dn913103.aspx) )，可繪製出緊密圍繞整個資料集的界限。 這適合用於異常偵測。 任何遠超出該界限外的新資料點，其異常程度都值得特別注意。

### <a name="bayesian-methods"></a>貝氏方法

貝氏方法具有令人滿意的高品質：可避免過度學習。 這些方法會針對可能的答案分佈事先做出一些假設。 這種方法的另一個附加好處在於其參數非常少。 Azure 機器學習中的分類 ([雙類別貝氏點機器](https://msdn.microsoft.com/library/azure/dn905930.aspx)) 和迴歸 ([貝氏線性迴歸](https://msdn.microsoft.com/library/azure/dn906022.aspx)) 都各有貝氏演算法。
請注意，這些假設是建立在資料可以用一條直線分割或符合一條直線的情況下。

在歷史記錄中，貝氏點機器是由 Microsoft Research 所開發。 它們有一些格外出色的理論做為後盾。 有興趣的學生可參考 [JMLR 中的原始文章](http://jmlr.org/papers/volume1/herbrich01a/herbrich01a.pdf)和 [Chris Bishop 深入探討的部落格](http://blogs.technet.com/b/machinelearning/archive/2014/10/30/embracing-uncertainty-probabilistic-inference.aspx)。

### <a name="specialized-algorithms"></a>專門的演算法
如果您有非常特定的目標，那麼您的可能運氣特別好。 在 Azure Machine Learning 集合中，有演算法專門：

- 排名預測 ([序數迴歸](https://msdn.microsoft.com/library/azure/dn906029.aspx))、
- 計算預測 ([波氏迴歸](https://msdn.microsoft.com/library/azure/dn905988.aspx))、
- 異常偵測 (一個以[主體元件分析](https://msdn.microsoft.com/library/azure/dn913102.aspx)為基礎，一個以[支援向量機器](https://msdn.microsoft.com/library/azure/dn913103.aspx)為基礎)
- 叢集 ([K-means](https://msdn.microsoft.com/library/azure/5049a09b-bd90-4c4e-9b46-7c87e3a36810/))

![PCA 型異常偵測][8]

***PCA 型異常偵測*** *：大部分的資料均可分成舊式的散佈；而大幅偏離該散佈的點都是可疑之處*

![使用 K-means 分組的資料集][9]

***資料集使用 K-means 分為五個叢集***

另外還有整體的[一對多多類別分類器](https://msdn.microsoft.com/library/azure/dn905887.aspx)，其中會將 N 類別分類問題分成 N-1 雙類別分類問題。 精確度、定型時間和線性屬性是由使用的雙類別分類器來決定。

![雙類別分類器結合成三個類別分類器][10]

***一組雙類別分類器結合成三個類別分類器***

Azure 機器學習中也可存取 [Vowpal Wabbit](https://msdn.microsoft.com/library/azure/8383eb49-c0a3-45db-95c8-eb56a1fef5bf)標題下，一個功能強大的機器學習架構。
VW 背離這裡的歸納，因為它可以學習分類和迴歸問題，甚至還能從一些沒有標記的資料中學習。 您可以將它設定為任意使用一些學習演算法、損失函式以及最佳化演算法。 它從一開始就是設計為高效率、可平行工作且速度極快。 它可以輕鬆處理大到不可思議的特徵集。
由創辦 Microsoft Research 的 John Langford 所發起及領導的 VW，可謂原裝賽車演算法領域中的一級方程式項目。 並非所有問題都符合 VW，但如果能符合，或許會值得您花時間在它的介面中了解它的學習曲線。 它也有以數種語言提供 [獨立的開放原始程式碼](https://github.com/JohnLangford/vowpal_wabbit) 。

## <a name="next-steps"></a>後續步驟

* 如需描述演算法並提供範例的可下載資訊圖詳細資訊，請參閱[可下載的資訊圖：機器學習服務基本概念和演算法範例](basics-infographic-with-algorithm-examples.md)。

* 如需 Machine Learning Studio 中依類別排序的所有可用機器學習演算法清單，請參閱＜Machine Learning Studio 演算法和模組說明＞中的[起始模型](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/machine-learning-initialize-model)。

* 如需 Machine Learning Studio 中按字母順序排列的完整演算法和模組清單，請參閱＜Machine Learning Studio 演算法和模組說明＞中的 [Machine Learning Studio 模組的 A-Z 清單](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/a-z-module-list)。

* 若要下載並列印提供 Machine Learning Studio 功能概觀的圖表，請參閱 [Azure Machine Learning Studio 功能的概觀圖](studio-overview-diagram.md)。

<!-- Media -->

[1]: ./media/algorithm-choice/image1.png
[2]: ./media/algorithm-choice/image2.png
[3]: ./media/algorithm-choice/image3.png
[4]: ./media/algorithm-choice/image4.png
[5]: ./media/algorithm-choice/image5.png
[6]: ./media/algorithm-choice/image6.png
[7]: ./media/algorithm-choice/image7.png
[8]: ./media/algorithm-choice/image8.png
[9]: ./media/algorithm-choice/image9.png
[10]: ./media/algorithm-choice/image10.png
