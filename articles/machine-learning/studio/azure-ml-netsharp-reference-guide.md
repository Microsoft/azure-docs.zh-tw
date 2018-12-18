---
title: Net# 類神經網路規格語言指南 - Azure Machine Learning | Microsoft Docs
description: Net# 類神經網路規格語言的語法，以及如何使用 Net# 建立自訂類神經網路模型的範例
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: cfd1454b-47df-4745-b064-ce5f9b3be303
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 03/01/2018
ms.openlocfilehash: 2571420e3497e85ec1bf37b340015b061e8c91d4
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833908"
---
# <a name="guide-to-net-neural-network-specification-language-for-azure-machine-learning"></a>適用於 Azure Machine Learning 的 Net# 類神經規格語言指南

Net# 是由 Microsoft 所開發的語言，可用來定義類神經網路架構。 使用 Net# 定義類神經網路的結構，可讓您定義如深度類神經網路或任意維度的迴旋，而這些項目都可改善影像、音訊或視訊等資料的學習。

您可以在這些內容中使用 Net# 架構規格：

+ Microsoft Azure Machine Learning Studio 中的所有類神經網路模組：[多類別類神經網路](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/multiclass-neural-network)、[雙類別類神經網路](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/two-class-neural-network)和[類神經網路迴歸](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/neural-network-regression)
+ MicrosoftML 中的類神經網路功能：針對 R 語言為 [NeuralNet](https://docs.microsoft.com/machine-learning-server/r-reference/microsoftml/neuralnet) 和 [rxNeuralNet](https://docs.microsoft.com/machine-learning-server/r-reference/microsoftml/rxneuralnet)，針對 Python 為 [rx_neural_network](https://docs.microsoft.com/machine-learning-server/python-reference/microsoftml/rx-neural-network)。


本文說明使用 Net# 開發自訂類神經網路所需的基本概念和語法： 

+ 類神經網路的需求和主要元件的定義方式
+ Net# 規格語言的語法和關鍵字
+ 使用 Net# 建立的自訂類神經網路範例 

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="neural-network-basics"></a>類神經網路基本概念

類神經網路的組成結構，包含以層組織的節點，和節點之間的加權連線 (或邊緣)。 這些連線是雙向的，且每個連線都有來源節點和目的地節點。  

每個可訓練的層 (隱藏或輸出層) 都有一或多個**連線套件組合**。 連線套組由來源層和來自該來源層之連線的規格所組成。 一個套件組合中的所有連線會共用相同的來源層和相同的目的地層。 在 Net# 中，連線套組會被視為由套組的目的地層所屬。

Net# 支援多種不同的連線套組，可讓您自訂輸入對應至隱藏層和對應至輸出的方式。

預設或標準套組為**完整套組**；在此類套組中，來源層中的每個節點會分別連接到目的地層中的各個節點。

此外，Net# 也支援下列四種進階連線套組：

+ **篩選套組**。 使用者可使用來源層節點和目的地層節點的位置來定義述詞。 只要述詞為 True，即會連接節點。

+ **迴旋套組**。 使用者可在來源層中定義小型的節點臨近地區。 目的地層中的每個節點都會連接到來源層中的一個節點鄰區。

+ **集區套組**和**回應正規化套組**。 這些套組類似於可供使用者在來源層中定義小型節點臨近地區的迴旋套組。 差別在於這些套組中的邊緣加權無法訓練。 因此會將預先定義的函數套用至來源節點值來判斷目的地節點值。


## <a name="supported-customizations"></a>支援的自訂

您在 Azure Machine Learning 中建立的類神經網路模型架構，可以使用 Net# 廣泛地進行自訂。 您可以：

+ 建立隱藏層及控制每一層中的節點數目。
+ 指定各層彼此連接的方式。
+ 定義特殊連線結構，例如迴旋和加權共用套組。
+ 指定不同的啟用函數。

如需規格語言語法的詳細資訊，請參閱[結構規格](#Structure-specifications)。  

如需為某些一般機器學習工作定義類神經網路的範例 (包括簡單與複雜)，請參閱[範例](#Examples-of-Net#-usage)。

## <a name="general-requirements"></a>一般需求

+ 必須只能有一個輸出層，和至少一個輸入層，以及零或多個隱藏層。 
+ 每一層都有固定的節點數目，在概念上會以任意維度的矩形陣列編排。 
+ 輸入層沒有相關聯的訓練參數，而會顯示執行個體資料進入網路的點。 
+ 可訓練層 (隱藏層和輸出層) 具有相關聯的訓練參數，一般稱之為加權和偏差。 
+ 來源和目的地節點必須位於不同層內。 
+ 連線必須是非循環的，換句話說，連線不可以有往回導向至初始來源節點的鏈結。
+ 輸出層不可以是連線套組的來源層。  

## <a name="structure-specifications"></a>結構規格

類神經網路結構規格由三個區段所組成：**常數宣告**、**層宣告**和**連線宣告**。 另外還有一個選擇性的**共用宣告**區段。 這幾個部分可依任意順序指定。

## <a name="constant-declaration"></a>常數宣告

常數宣告是選擇性宣告。 它提供的機制可用來定義在類神經網路定義中使用的值。 宣告陳述式的組成元素是識別碼之後尾隨加號和值運算式。

例如，下列陳述式會定義常數 `x`：  

`Const X = 28;`

若要同時定義兩個或更多常數，請將識別碼名稱和值放在大括號中，並使用分號分隔。 例如︰

`Const { X = 28; Y = 4; }`

各個指派運算式的右側可以是整數、實數、布林值 (True/False) 或數學運算式。 例如︰

`Const { X = 17 * 2; Y = true; }`

## <a name="layer-declaration"></a>層宣告

層宣告是必要宣告。 它定義層的大小和來源，包括層的連線套組和屬性。 宣告陳述式以層的名稱開頭 (輸入、隱藏或輸出)，其後是層的維度 (正整數的 Tuple)。 例如︰

```Net#
input Data auto;
hidden Hidden[5,20] from Data all;
output Result[2] from Hidden all;  
```

+ 維度的乘積為層中的節點數目。 此範例中有兩個維度 [5,20]，這表示層中有 100 個節點。
+ 層能以任何順序宣告，僅有一項例外：如果定義了多個輸入層，則這些層的宣告順序必須符合輸入資料中各項特性的順序。

若要指定讓系統自動決定層中的節點數目，請使用 `auto` 關鍵字。 `auto` 關鍵字的效果會隨著層而不同：

+ 在輸入層宣告中，節點數目是輸入資料中的特性數目。
+ 在隱藏層宣告中，節點數目是**隱藏節點數目**的參數值所指定的數目。
+ 在輸出層宣告中，雙類別分類的節點數目是 2 個 (1 個用於迴歸)，且等於多類別分類的輸出節點數目。

例如，下列網路定義會讓系統自動決定所有層的大小：

```Net#
input Data auto;
hidden Hidden auto from Data all;
output Result auto from Hidden all;  
```

可訓練層 (隱藏層或輸出層) 的層宣告可選擇性地包含輸出函數 (也稱為啟用函數)，其預設值為適用於分類模型的 **sigmoid** 以及適用於迴歸模型的 **linear**。 即使您使用預設值，您也可以在需要釐清時明確陳述啟用函數。

支援的輸出函數如下：

+ sigmoid
+ linear
+ softmax
+ rlinear
+ square
+ sqrt
+ srlinear
+ abs
+ tanh
+ brlinear

例如，下列宣告將使用 **softmax** 函數：

`output Result [100] softmax from Hidden all;`

## <a name="connection-declaration"></a>連線宣告

在定義可訓練層之後，您必須隨即宣告您所定義的層之間的連線。 連線套件組合宣告以關鍵字 `from` 開頭，其後是套件組合的來源層名稱和所要建立的連線套件組合類型。

目前支援五種連線套組：

+ **完整**套件組合，以關鍵字 `all` 指出
+ **篩選**套件組合，以關鍵字 `where` 指出，其後接著述詞運算式
+ **迴旋**套件組合，以關鍵字 `convolve` 指出，其後接著迴旋屬性
+ **集區**套組，以關鍵字 **max pool** 或 **mean pool** 指出
+ **回應正規化**套組，以關鍵字 **response norm** 指出。

## <a name="full-bundles"></a>完整套組

完整連線套組包含從來源層中的每個節點連至目的地層中各個節點的連線。 這是預設的網路連線類型。

## <a name="filtered-bundles"></a>篩選套組

篩選連線套組規格包含一個述詞，此述詞在語法上的表示方式非常類似 C# lambda 運算式。 以下範例將定義兩個篩選套組：

```Net#
input Pixels [10, 20];
hidden ByRow[10, 12] from Pixels where (s,d) => s[0] == d[0];
hidden ByCol[5, 20] from Pixels where (s,d) => abs(s[1] - d[1]) <= 1;  
```

+ 在 `ByRow` 的述詞中 `s` 參數代表輸入層 `Pixels` 之節點矩形陣列的索引，而 `d` 參數代表隱藏層 `ByRow` 之節點陣列的索引。 `s` 和 `d` 的類型為長度二之整數的 Tuple。 在概念上，`s` 的範圍介於所有 `0 <= s[0] < 10` 與 `0 <= s[1] < 20` 的整數對之間，而且 `d` 的範圍介於所有 `0 <= d[0] < 10` 與 `0 <= d[1] < 12` 的整數對之間。 

+ 述詞運算式右側會有一個條件。 在此範例中，每個條件為 True 的 `s` 值和 `d` 值都會有一個從來源層節點到目的地層節點的邊緣。 因此，此篩選運算式表示在 s[0] 等於 d[0] 的所有情況下，套件組合都會包含從 `s` 所定義的節點到 `d` 所定義之節點的連線。

您可以選擇性地為篩選套組指定一組加權。 **Weights** 屬性的值必須是長度與套組所定義的連線數目相符之浮點值的 Tuple。 依預設會隨機產生加權。

加權值會依目的地節點索引分組。 也就是說，如果第一個目的地節點連線到 `K` 個來源節點，則**加權** Tuple 的前 K 個項目將會是第一個目的地節點的加權 (依來源索引順序)。 其餘目的地節點也適用相同的原則。

可以直接指定加權當做常數值。 比方說，如果您先前已了解比重，您可以使用此語法將它們指定為常數：

`const Weights_1 = [0.0188045055, 0.130500451, ...]`

## <a name="convolutional-bundles"></a>迴旋套組

當訓練資料具有同質結構時，迴旋連線通常會用來學習資料的高階特性。 例如，在影像、音訊或視訊資料中，空間或暫時維度可能會相當一致。  

迴旋套組採用滑經維度的矩形**核心**。 基本上，每個核心會定義一組在本端鄰區中套用的加權，稱為**核心應用程式**。 每個核心應用程式都會對應至來源層中名為**中心節點**的節點。 一個核心的加權會在許多連線間共用。 在迴旋套組中，每個核心都是矩形，且所有核心應用程式的大小皆相同。  

迴旋套組支援下列屬性：

**InputShape** 會針對此迴旋套組的用途，定義來源層的維度。 其值必須是正整數的 Tuple。 整數的乘積必須等於來源層中的節點數目，但另一方面並不需要符合為來源層宣告的維度。 此 Tuple 的長度會成為迴旋套組的 **Arity** 值。 Arity 通常會參照一個函式可取用的引數或運算元數目。

若要定義核心的形狀和位置，請使用 **KernelShape**、**Stride**、**Padding**、**LowerPad** 和 **UpperPad** 等屬性：   

+ **KernelShape**：(必要) 定義迴旋套組各個核心的維度。 其值必須是長度等於套組 Arity 之正整數的 Tuple。 此 Tuple 的每個元件皆不可大於 **InputShape** 的對應元件。 

+ **Stride**：(選用) 定義迴旋的滑動步階大小 (每個維度的步階大小)，即中心節點之間的距離。 其值必須是長度為套組 Arity 之正整數的 Tuple。 此 Tuple 的每個元件皆不可大於 **KernelShape** 的對應元件。 預設值是所有元件皆等於一的 Tuple。 

+ **Sharing**：(選用) 為迴旋的每個維度定義加權共用。 其值可以是單一布林值，或是長度為套組 Arity 之布林值的 Tuple。 單一布林值可擴充為具有正確長度、所有元件皆等於指定值的 Tuple。 預設值是全部由 True 值組成的 Tuple。 

+ **MapCount**：(選用) 定義迴旋套組的特性對應數目。 其值可以是單一正整數，或是長度為套組 Arity 之正整數的 Tuple。 單一整數值可擴充為具有正確長度、第一個元件皆等於指定值且其餘元件皆等於一的 Tuple。 預設值為一。 特性對應的總數是 Tuple 元件的乘積。 各個元件的此一總數經過因式分解後，將決定特性對應值在目的地節點中的分組方式。 

+ **Weights**：(選用) 定義套組的初始加權。 其值必須是長度為核心數目乘以每個核心的加權數目之浮點值的 Tuple，如本文稍後的定義。 預設加權會隨機產生。  

有兩組屬性控制填補，這些屬性互斥：

+ **Padding**：(選用) 決定是否應使用**預設的填補配置**填補輸入。 其值可以是單一布林值，或是長度為套組 Arity 之布林值的 Tuple。 

    單一布林值可擴充為具有正確長度、所有元件皆等於指定值的 Tuple。 
    
    如果維度的值為 True，則來源在邏輯上會以零值儲存格在該維度中進行填補，以支援其他核心應用程式，使該維度中第一個和最後一個核心的中心節點成為該維度在來源層中的第一個和最後一個節點。 據此將會自動產生每個維度中的「虛擬」節點數目，以將 `(InputShape[d] - 1) / Stride[d] + 1` 個核心配適到已填補的來源層中。 
    
    如果維度的值為 False，則在定義核心時，會使每一端排除的節點數目保持相同 (差距不會超過 1 個)。 此屬性的預設值是所有元件皆為 False 的 Tuple。

+ **UpperPad** 和 **LowerPad**：(選用) 更能控制所要使用的填補量。 **重要事項：** 只有在***未***定義上述的 **Padding** 屬性時，才能定義這些屬性。 其值應為長度為套組 Arity 的整數值 Tuple。 在指定這些屬性時，將會在輸入層各個維度的下端和上端新增「虛擬」節點。 在每個維度中的下端和上端新增的節點數目，分別取決於 **LowerPad**[i] 和 **UpperPad**[i]。 

    若要確定核心只會對應至「實際」節點而非「虛擬」節點，必須要符合下列條件：
      - **LowerPad** 的每個元件皆務必小於 `KernelShape[d]/2`。 
      - **UpperPad** 的每個元件皆不可大於 `KernelShape[d]/2`。 
      - 這些屬性的預設值是所有元件皆等於 0 的 Tuple。 

    **Padding** = true 設定允許可將核心的「中心」保持在「真實」輸入內所需的填補量。 這麼做會略為改變輸出大小的計算方式。 一般而言，輸出大小 D 的計算方式為 `D = (I - K) / S + 1`，其中 `I` 是輸入大小，`K` 是核心大小，`S` 是分散，而 `/` 是整數除法 (趨近於零)。 如果您設定 UpperPad = [1, 1]，輸入大小 `I` 實際上是 29，因此 `D = (29 - 5) / 2 + 1 = 13`。 不過，當 **Padding** = true 時，基本上 `I` 會藉由 `K - 1` 而提高；因此 `D = ((28 + 4) - 5) / 2 + 1 = 27 / 2 + 1 = 13 + 1 = 14`。 藉由指定 **UpperPad** 和 **LowerPad** 的值，您對填補的控制權會遠遠超過只設定 **Padding** = true。

如需迴旋網路及其應用程式的詳細資訊，請參閱下列文章： 

+ [http://deeplearning.net/tutorial/lenet.html ](http://deeplearning.net/tutorial/lenet.html)
+ [http://research.microsoft.com/pubs/68920/icdar03.pdf](http://research.microsoft.com/pubs/68920/icdar03.pdf) 
+ [http://people.csail.mit.edu/jvb/papers/cnn_tutorial.pdf](http://people.csail.mit.edu/jvb/papers/cnn_tutorial.pdf)  

## <a name="pooling-bundles"></a>集區套組

**集區套組**會套用類似於迴旋連線的幾何圖形，但會對來源節點值使用預先定義的函數，以衍生目的地節點值。 因此，集區套組並沒有可訓練的狀態 (加權或偏差)。 集區套組支援 **Sharing**、**MapCount** 和 **Weights** 以外的所有迴旋屬性。

一般而言，以相鄰集區單元彙總的核心並不會重疊。 如果每個維度中的 Stride[d] 皆等於 KernelShape[d]，則取得的層將會是傳統本端集區層，這通常運用在迴旋類神經網路中。 每個目的地節點都會計算其核心在來源層中的活動數上限或平均值。  

以下透過範例說明集區套組： 

```Net#
hidden P1 [5, 12, 12]
  from C1 max pool {
  InputShape  = [ 5, 24, 24];
   KernelShape = [ 1,  2,  2];
   Stride      = [ 1,  2,  2];
  }  
```

+ 套件的 Arity 為 3：也就是，Tuple `InputShape`、`KernelShape` 和 `Stride` 的長度。 
+ 來源層中的節點數目為 `5 * 24 * 24 = 2880`。 
+ 這是傳統本端集區層，因為 **KernelShape** 和 **Stride** 是相等的。 
+ 目的地層中的節點數目為 `5 * 12 * 12 = 1440`。

如需集區層的詳細資訊，請參閱下列文章：  

+ [http://www.cs.toronto.edu/~hinton/absps/imagenet.pdf](http://www.cs.toronto.edu/~hinton/absps/imagenet.pdf) (第 3.4 節)
+ [http://cs.nyu.edu/~koray/publis/lecun-iscas-10.pdf](http://cs.nyu.edu/~koray/publis/lecun-iscas-10.pdf) 
+ [http://cs.nyu.edu/~koray/publis/jarrett-iccv-09.pdf](http://cs.nyu.edu/~koray/publis/jarrett-iccv-09.pdf)

## <a name="response-normalization-bundles"></a>回應正規化套組

**回應正規化**是一種本端正規化配置，最早是由 Geoffrey Hinton 等人發表於 [ImageNet Classiﬁcation with Deep Convolutional Neural Networks](http://www.cs.toronto.edu/~hinton/absps/imagenet.pdf) 報告中。 

回應正規化可用來輔助類神經網路中的一般化。 當一個神經元在非常高的啟動層級上引發時，本端回應正規化層將會抑制周遭神經元的啟動層級。 此動作會使用三個參數 (`α`、`β` 和 `k`) 與迴旋結構 (或鄰區分布型態) 來完成。 目的地層中的每個神經元 **y**，會分別對應至來源層中的一個神經元 **x**。 **y** 的啟用層級來自於下列公式，其中，`f` 是神經元的啟用層級，`Nx` 是核心 (或是包含 **x** 的鄰區中各神經元的集合)，如下列迴旋結構所定義：  

![迴旋結構的公式](./media/azure-ml-netsharp-reference-guide/formula_large.png)

回應正規化套組支援 **Sharing**、**MapCount** 和 **Weights** 以外的所有迴旋屬性。  

+ 如果核心包含與 ***x*** 位於相同對應中的神經元，則正規化配置稱為**相同對應正規化**。 若要定義相同對應正規化，**InputShape** 中的第一個座標必須具有值 1。

+ 如果核心包含與 ***x*** 位於相同空間位置中的神經元，但這些神經元位於其他對應中，則此正規化配置稱為**跨對應正規化**。 這種類型的回應正規化會實作實際神經元的類型所衍生出來的一種橫向抑制，而在不同對應上運算的神經元輸出之間產生對大量啟用層級的競用。 若要定義跨對應正規化，第一個座標必須是大於一、且不大於對應數的整數，而其餘座標必須具有值 1。

回應正規化套組會將預先定義的函數套用至來源節點值以決定目的地節點值，因此並不具有可訓練的狀態 (加權或偏差)。

> [!NOTE]
> 目的地層中的節點會對應至在核心中作為中心節點的神經元。 例如，如果 `KernelShape[d]` 是奇數，則 `KernelShape[d]/2` 會對應至中央核心節點。 如果 `KernelShape[d]` 是偶數，中央節點位於 `KernelShape[d]/2 - 1`。 因此，如果 `Padding[d]` 為 False，則第一個和最後一個 `KernelShape[d]/2` 的節點在目的地層中沒有對應的節點。 若要避免發生此狀況，請將 **Padding** 定義為 [true, true, …, true]。

除了前述的四個屬性以外，回應正規化套組也支援下列屬性：

+ **Alpha**：(必要) 指定在前述公式中對應至 `α` 的浮點值。 
+ **Beta**：(必要) 指定在前述公式中對應至 `β` 的浮點值。 
+ **Offset**：(選用) 指定在前述公式中對應至 `k` 的浮點值。 其預設值為 1。

下列範例將定義使用這些屬性的回應正規化套組：  

```Net#
hidden RN1 [5, 10, 10]
from P1 response norm {
  InputShape  = [ 5, 12, 12];
  KernelShape = [ 1,  3,  3];
  Alpha = 0.001;
  Beta = 0.75;
  }  
```

+ 來源層包含五個對應，每個對應有 12x12 的維度，共計有 1440 個節點。 
+ **KernelShape** 的值指出這是相同對應正規化層，其中，鄰區為 3x3 矩形。 
+ **Padding** 的預設值為 False，因此目的地層的每個維度中只有 10 個節點。 若要在目的地層中加入一個與來源層中的每個節點相對應的節點，請新增 Padding = [true, true, true]，並將 RN1 的大小變更為 [5, 12, 12]。  

## <a name="share-declaration"></a>共用宣告

Net# 可選擇性地支援以共用加權定義多個套組的作業。 任何兩個套組如果具有相同的結構，即可共用加權。 下列語法定義使用共用加權的套組：  

```Net#
share-declaration:
  share    {    layer-list    }
  share    {    bundle-list    }
  share    {    bias-list    }

  layer-list:
    layer-name    ,    layer-name
    layer-list    ,    layer-name

  bundle-list:
    bundle-spec    ,    bundle-spec
    bundle-list    ,    bundle-spec

  bundle-spec:
    layer-name    =>     layer-name

  bias-list:
    bias-spec    ,    bias-spec
    bias-list    ,    bias-spec

  bias-spec:
    1    =>    layer-name

  layer-name:
    identifier
```

例如，下列共用宣告會指定層名稱，指出應同時共用加權和偏差：  

```Net#
Const {
  InputSize = 37;
  HiddenSize = 50;
  }
input {
  Data1 [InputSize];
  Data2 [InputSize];
  }
hidden {
  H1 [HiddenSize] from Data1 all;
  H2 [HiddenSize] from Data2 all;
  }
output Result [2] {
  from H1 all;
  from H2 all;
  }
share { H1, H2 } // share both weights and biases  
```

+ 輸入特性會分割為兩個大小相等的輸入層。 
+ 接著，隱藏層會對兩個輸入層計算層級較高的特性。 
+ 共用宣告會指定 *H1* 和 *H2* 必須由其各自的輸入以相同方式計算。  

或者，這也可以用兩個不同的共用宣告來指定，如下所示：  

```Net#
share { Data1 => H1, Data2 => H2 } // share weights  
<!-- -->
    share { 1 => H1, 1 => H2 } // share biases  
```

只有當層中包含單一套組時，才可以使用速記格式。 一般而言，只有在相關結構完全相同，也就是具有相同大小、相同迴旋幾何等等時，才能使用共用。  

## <a name="examples-of-net-usage"></a>Net# 的使用範例

本節將透過幾個範例，說明如何使用 Net# 來新增隱藏層、定義隱藏層與其他層的互動方式，以及建置迴旋網路。

### <a name="define-a-simple-custom-neural-network-hello-world-example"></a>定義簡易的自訂類神經網路："Hello World" 範例

這個簡易範例將說明如何建立具有單一隱藏層的類神經網路模型。

```Net#
input Data auto;
hidden H [200] from Data all;
output Out [10] sigmoid from H all;  
```

此範例說明某些基本命令，如下所示：  

+ 第一行會定義輸入層 (名為 `Data`)。 當您使用 `auto` 關鍵字，類神經網路會自動包含輸入範例中的所有功能資料行。 
+ 第二行會建立隱藏層。 隱藏層會被指派名稱 `H`，其中包含 200 個節點。 此層會與輸入層完全相連。
+ 第三行定義輸出層 (名為 `O`)，其中包含 10 個輸出節點。 如果類神經網路用於分類，每個類別會有一個輸出節點。 關鍵字 **sigmoid** 指出套用至輸出層的輸出函數。

### <a name="define-multiple-hidden-layers-computer-vision-example"></a>定義多個隱藏層：電腦願景範例

下列範例說明如何定義略為複雜、具有多個自訂隱藏層的類神經網路。  

```Net#
// Define the input layers 
input Pixels [10, 20];
input MetaData [7];

// Define the first two hidden layers, using data only from the Pixels input
hidden ByRow [10, 12] from Pixels where (s,d) => s[0] == d[0];
hidden ByCol [5, 20] from Pixels where (s,d) => abs(s[1] - d[1]) <= 1;

// Define the third hidden layer, which uses as source the hidden layers ByRow and ByCol
hidden Gather [100] 
{
from ByRow all;
from ByCol all;
}

// Define the output layer and its sources
output Result [10]  
{
from Gather all;
from MetaData all;
}  
```

此範例說明類神經網路規格語言的幾項特性：

+ 此結構具有兩個輸入層：`Pixels` 和 `MetaData`。
+ `Pixels` 層是兩個連線套件組合的來源層，其目的地層為 `ByRow` 和 `ByCol`。
+ `Gather` 層和 `Result` 層是多個連線套件組合中的目的地層。
+ 輸出層 `Result` 是兩個連線套件組合中的目的地層；一個套件組合以第二層級的隱藏層 `Gather` 作為目的地層，另一個套件組合以輸入層 `MetaData` 作為目的地層。
+ 隱藏層 `ByRow` 和 `ByCol` 使用述詞運算式指定篩選連線。 更精確地說，在 [x, y] 上，`ByRow` 中的節點會連線到 `Pixels` 中第一個索引座標等於節點的第一個座標 x 的節點。 同樣地，在 [x, y] 上，`ByCol` 中的節點會連接到 `Pixels` 中在其中一個節點的第二個座標 y 內具有第二個索引座標的節點。

### <a name="define-a-convolutional-network-for-multiclass-classification-digit-recognition-example"></a>定義多類別分類的迴旋網路：數字辨識範例

下列網路的定義設計來辨識數字，說明自訂類神經網路的某些進階技術。  

```Net#
input Image [29, 29];
hidden Conv1 [5, 13, 13] from Image convolve 
  {
  InputShape  = [29, 29];
  KernelShape = [ 5,  5];
  Stride      = [ 2,  2];
  MapCount    = 5;
  }
hidden Conv2 [50, 5, 5]
from Conv1 convolve 
  {
  InputShape  = [ 5, 13, 13];
  KernelShape = [ 1,  5,  5];
  Stride      = [ 1,  2,  2];
  Sharing     = [false, true, true];
  MapCount    = 10;
  }
hidden Hid3 [100] from Conv2 all;
output Digit [10] from Hid3 all;  
```

+ 此結構具有單一輸入層：`Image`。
+ 關鍵字 `convolve` 指出名為 `Conv1` 和 `Conv2` 的層為迴旋層。 這些層宣告的後面分別會加上一份迴旋屬性清單。
+ 網路具有第三個隱藏層 `Hid3`，與第二個隱藏層 `Conv2` 完全相連。
+ 輸出層 `Digit` 僅連線到第三個隱藏層 `Hid3`。 關鍵字 `all` 指出輸出層與 `Hid3` 完全相連。
+ 迴旋的 Arity 為三：Tuple `InputShape`、`KernelShape`、`Stride, and ` Sharing 的長度。 
+ 每個核心的加權數目是 `1 + KernelShape\[0] * KernelShape\[1] * KernelShape\[2] = 1 + 1 * 5 * 5 = 26`。 或 `26 * 50 = 1300`。
+ 您可以用下列方式計算每個隱藏層中的節點數：

    `NodeCount\[0] = (5 - 1) / 1 + 1 = 5` `NodeCount\[1] = (13 - 5) / 2 + 1 = 5` `NodeCount\[2] = (13 - 5) / 2 + 1 = 5`

+ 節點總數可使用該層的宣告維度 [50, 5, 5] 來計算，如下所示：`MapCount * NodeCount\[0] * NodeCount\[1] * NodeCount\[2] = 10 * 5 * 5 * 5`
+ 由於只有 `d == 0` 時，`Sharing[d]` 才會是 False，因此核心數為 `MapCount * NodeCount\[0] = 10 * 5 = 50`。 

## <a name="acknowledgements"></a>通知

自訂類神經網路架構的 Net# 語言是由 Shon Katzenberger (架構、機器學習服務) 和 Alexey Kamenev (軟體工程師、Microsoft Research) 在 Microsoft 開發。 它在內部用於機器學習服務專案與應用程式，範圍從映像偵測到文字分析。 如需詳細資訊，請參閱 [Azure ML 中的類神經網路 - Net# 簡介](http://blogs.technet.com/b/machinelearning/archive/2015/02/16/neural-nets-in-azure-ml-introduction-to-net.aspx)
