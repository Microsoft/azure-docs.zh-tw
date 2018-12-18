---
title: 5 個資料科學問題 - 適用於初學者的資料科學 - Azure Machine Learning | Microsoft Docs
description: 「適用於初學者的資料科學」會運用 5 支短片教導基本概念，首先為「資料科學可以回答的 5 個問題」。 來自 Azure Machine Learning。
keywords: 進行資料科學, 資料科學初學者, 適用於初學者的資料科學, 資料科學基本概念, 資料科學問題, 資料科學影片, 資料科學簡介
services: machine-learning
documentationcenter: na
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cjgronlund
ms.assetid: a01f93ee-01eb-4afe-abbd-cfa035c119b0
ms.service: machine-learning
ms.component: studio
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2018
ms.openlocfilehash: 1d004eb16fcac13d6ba7592cbe432cbeac0401e8
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834146"
---
# <a name="data-science-for-beginners-video-1-the-5-questions-data-science-answers"></a>適用於初學者的資料科學影片 1：資料科學可以回答的 5 個問題
從頂尖資料科學家所提供之「適用於初學者的資料科學」的五個簡短影片中快速認識資料科學。 無論您是對從事資料科學有興趣，或您是和資料科學家一起工作，這些影片都能為您提供基本但很有用的知識。

第一段影片是關於資料科學可以回答何種問題。 若要充分運用這系列影片，請觀賞所有影片。 [瀏覽影片清單](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Shows/SupervisionNotRequired/8/player]
>
>

## <a name="other-videos-in-this-series"></a>系列中的其他影片
 是一個資料科學的快速簡介，總共大約 25 分鐘。 查看全部五支影片：

* 影片 1：資料科學可以回答的 5 個問題
* 影片 2： [您的資料已經可以進行資料科學了嗎？](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 分 56 秒)*
* 影片 3：[詢問您可以使用資料回答的問題](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 分 17 秒)*
* 影片 4：[利用簡單模型預測答案](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 分 42 秒)*
* 影片 5：[複製其他人的工作進行資料科學](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 分 18 秒)*

## <a name="transcript-the-5-questions-data-science-answers"></a>文字記錄：資料科學可以回答的 5 個問題
大家好！ 歡迎觀賞「適用於初學者的資料科學」 系列影片。

資料科學可能令人卻步，因此我將在此處介紹基本概念，而且不會用到任何方程式或電腦程式設計術語。

在第一個影片中，我們將討論「資料科學可以回答的 5 個問題」。

資料科學使用數字和名稱 (也就是類別或標籤) 來預測問題的答案。

您可能會很驚訝，但是「資料科學只能回答下列五種問題」 ：

* 這是 A 或 B 嗎？
* 這很奇怪嗎？
* 有多少？
* 這是如何組織的？
* 我接下來該怎麼辦？

您可以透過不同系列的機器學習方法 (稱為演算法) 來回答這其中每一個問題。

您可以將演算法視為配方，將資料視為原料，這非常有用。 演算法可告訴您如何結合並混合資料，以便獲得答案。 電腦就像是攪拌器。 它們可以為您執行演算法中大部分費力的工作，而且非常快速地執行。

## <a name="question-1-is-this-a-or-b-uses-classification-algorithms"></a>「問題 1︰這是 A 或 B 嗎？」使用分類演算法
讓我們從「這是 A 或 B 嗎？」的問題開始。

![分類演算法：這是 A 或 B 嗎？](./media/data-science-for-beginners-the-5-questions-data-science-answers/classification-algorithms.png)

這系列的演算法稱為雙類別分類。

它適用於任何只有兩個可能答案的問題。

例如︰

* 這個輪胎將會在接下來的 1,000 英哩發生故障嗎：是或否？
* 下列哪一種方式可以帶來更多客戶：一張 5 美元的優待券或一張 25% 的折扣券？

這個問題也可以改換措辭來包含兩個以上的選項︰這是 A 或 B 或 C 或 D，依此類推？  這稱為多元分類，當您有數個 (或數千個) 可能答案時非常有用。 多元分類會選擇可能性最高的答案。

## <a name="question-2-is-this-weird-uses-anomaly-detection-algorithms"></a>「問題 2︰這很奇怪嗎？」使用異常偵測演算法
下一個資料科學可以回答的問題是：這很奇怪嗎？ 這個問題可以透過稱為異常偵測的演算法系列來回答。

![異常偵測演算法：這很奇怪嗎？](./media/data-science-for-beginners-the-5-questions-data-science-answers/anomaly-detection-algorithms.png)

如果您有一張信用卡，您就已經從異常偵測中受益。 信用卡公司會分析您的購買模式，以便他們可以警告您可能發生詐騙事件。 「奇怪」的費用可能是在您通常不會去購物的商店中所產生的購物，或是購買異常昂貴的商品。

這個問題適用於很多方面。 例如：

* 如果您的車輛上配有壓力計，您可能就想知道：這個壓力計的讀數正常嗎？
* 如果您正在監視網際網路，您可能就想知道：這個來自網際網路的訊息正常嗎？

異常偵測會標記未預期或異常的事件或行為。 它提供在何處尋找問題的線索。

## <a name="question-3-how-much-or-how-many-uses-regression-algorithms"></a>「問題 3︰多少？或有多少？」使用迴歸演算法
機器學習服務也可以預測「多少？或有多少？」的答案。 回答這個問題的演算法系列稱為「迴歸」。

![迴歸演算法：多少？或有多少？](./media/data-science-for-beginners-the-5-questions-data-science-answers/regression-algorithms.png)

迴歸演算法會進行數值預測，例如：

* 下星期二的溫度將是多少？  
* 我第四季的銷售量將有多少？

它們有助於回答任何可要求以數字回答的問題。

## <a name="question-4-how-is-this-organized-uses-clustering-algorithms"></a>「問題 4︰這是如何組織的？」使用叢集演算法
現在，最後這兩個問題屬於更進階的問題。

有時您會想要了解資料集的結構 - 這是如何組織的？ 針對這個問題，您沒有已經知道結果的範例。

有很多方法可以梳理出資料結構。 其中一個方法就是叢集。 它會將資料正常分割為「團塊」，以便更容易進行解譯。 透過叢集，就不會有正確的答案。

![叢集演算法︰這是如何組織的？](./media/data-science-for-beginners-the-5-questions-data-science-answers/clustering-algorithms.png)

叢集問題的常見範例包括：

* 哪些觀眾喜歡同一種類型的電影？
* 哪些印表機型號失敗的方式相同？

若要了解資料的組織方式，您可以進一步了解 (和預測) 行為與事件。  

## <a name="question-5-what-should-i-do-now-uses-reinforcement-learning-algorithms"></a>「問題 5︰我現在該怎麼辦？」使用增強式學習演算法
最後一個問題 (我現在該怎麼辦？ ) 會使用稱為增強式學習的演算法系列。

增強式學習是藉由老鼠與人類大腦回應獎懲的方式來激發。 這些演算法會從結果中學習，並決定下一個動作。

一般而言，增強式學習適用於自動化系統，這類系統必須進行許多小型決策，而不需人為指引。

![增強式學習演算法︰我接下來該怎麼辦？](./media/data-science-for-beginners-the-5-questions-data-science-answers/reinforcement-learning-algorithms.png)

它回答的問題一定是應採取何種動作 - 通常透過機器或機器人來執行。 範例包括：

* 如果我是一間房屋的溫度控制系統︰調整溫度或讓它保持恆溫？  
* 如果我是一輛自動駕駛汽車：遇到黃燈時，煞車或加速？  
* 如果是機器人吸塵器︰繼續打掃，還是返回充電站？

增強式學習演算法會在它們執行期間蒐集資料，從試驗和錯誤中學習。

這就是「資料科學可以回答的 5 個問題」。

## <a name="next-steps"></a>後續步驟
* [嘗試使用 Machine Learning Studio 進行您的第一個資料科學實驗](create-experiment.md)
* [在 Microsoft Azure 上取得 Machine Learning 簡介](what-is-machine-learning.md)
