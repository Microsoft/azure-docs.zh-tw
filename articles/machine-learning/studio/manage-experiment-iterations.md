---
title: 在 Machine Learning Studio 中管理實驗逐一查看 | Microsoft Docs
description: 如何在 Azure Machine Learning Studio 中管理實驗逐一查看
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 6a53530f-20d5-40ae-9b49-7b499ccb44b7
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.openlocfilehash: 4dcae0bb3cb89e65079b88f7be68ddf360ce1b8c
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834806"
---
# <a name="manage-experiment-iterations-in-azure-machine-learning-studio"></a>在 Azure Machine Learning Studio 中管理實驗逐一查看
開發預測分析模型是一種逐一查看過程 - 您修改實驗的各種函數及參數，結果會不斷收斂，直到您對已訓練的有效模型感到滿意為止。 此程序的關鍵是追蹤實驗參數和組態的各種逐一查看。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

您可以隨時檢閱先前的實驗執行，以挑戰、重新瀏覽及最終確認或調整先前的假設。 當您執行實驗時，Machine Learning Studio 會保留執行歷程記錄，包括資料集、模組及連接埠連接和參數。 此歷程記錄也會擷取結果、執行階段資訊，例如開始和停止時間、記錄訊息及執行狀態。 您可以隨時回顧任何執行，以檢閱實驗和中繼結果的年表。 您甚至可以使用上一次的實驗執行，啟動到新的階段，在您的路徑上查詢和探索，以建立簡單、複雜或甚至集成模型解決方案。

> [!NOTE]
> 當您檢視上一次的實驗執行時，該版本的實驗已遭到鎖定，無法編輯。 但是，您可以儲存複本，方法是按一下 [ **另存新檔** ] 並且為複本提供新名稱。 Machine Learning Studio 會開啟新複本，然後您可以編輯及執行。 實驗的此複本可以在含有所有其他實驗的 [ **實驗** ] 清單中取得。
> 
> 

## <a name="viewing-the-prior-run"></a>檢視上一次執行
當您開啟已至少執行一次的實驗時，可以藉由按一下內容窗格的 [ **上一次執行** ] 以檢視先前的實驗執行。

例如，假設您建立實驗並且在下列時間執行其版本：11:23、11:42 及 11:55。 如果您開啟最後實驗執行 (11:55) 並且按一下 [ **上一次執行**]，則會開啟您在 11:42 執行的版本。

## <a name="viewing-the-run-history"></a>檢視執行歷程記錄
您可以檢視所有先前的實驗執行，方法是在開啟的實驗中按一下 [ **檢視執行歷程記錄** ]。

例如，假設您建立實驗，其具有[線性迴歸][linear-regression]模組，且您想要觀察在實驗結果中變更**學習速度**值的效果。 您針對此參數以不同值執行實驗多次，如下所示：

| 學習速度值 | 執行開始時間 |
| --- | --- |
| 0.1 |2014/9/11 下午 4:18:58 |
| 0.2 |2014/9/11 下午 4:24:33 |
| 0.4 |2014/9/11 下午 4:28:36 |
| 0.5 |2014/9/11 下午 4:33:31 |

如果您按一下 [ **檢視執行歷程記錄**]，您會看見所有執行的清單：

![範例執行歷程記錄][runhistory]

按一下任何執行以檢視您執行該實驗之時間的實驗快照。 保留組態、參數值、註解及結果，給予您該實驗執行的完整記錄。

> [!TIP]
> 若要記載您的實驗逐一查看，您可以在每次執行時修改標題，可以更新內容窗格中實驗的 **摘要** ，以及可以在個別模組中新增或更新註解以記錄您的變更。 標題、摘要及模組註解會在每一次執行實驗時儲存。
> 
> 

Machine Learning Studio 中 [實驗] 索引標籤的實驗清單一律會顯示最新版本的實驗。 如果您開啟實驗的上一次執行 (使用 [上一次執行] 或 [檢視執行歷程記錄])，您可以返回草稿版本，方法是按一下 [檢視執行歷程記錄] 並且選取 [狀態] 為 [可編輯] 的逐一查看。

## <a name="iterating-on-a-previous-run"></a>逐一查看上一次執行
當您按一下 [上一次執行] 或 [檢視執行歷程記錄]，並且開啟上一次執行時，您可以在唯讀模式中檢視完成的實驗。

如果想要從您為上一次執行設定的方式開始實驗的逐一查看，您可以藉由開啟執行並且按一下 [ **另存新檔**] 來完成。 這會建立新的實驗，具有新的標題、空白的執行歷程記錄及上一次執行的所有元件和參數值。 此新的實驗會列在 Machine Learning Studio 首頁的 [實驗]  索引標籤中，您可以修改及執行、為實驗的逐一查看起始新的執行歷程記錄。 

例如，假設您有上一個章節顯示的實驗執行歷程記錄。 您想要觀察當您將 [學習速度] 參數設為 0.4，並且針對 [訓練 epoch 數目] 參數嘗試不同值時，會發生什麼狀況。

1. 按一下 [ **檢視執行歷程記錄** ] 並且開啟您在下午 4:28:36 執行的實驗逐一查看 (您在其中將參數值設為 0.4)。
2. 按一下 [ **另存新檔**]。
3. 輸入新標題，然後按一下 [ **確定** ] 核取方塊。 實驗的新複本隨即建立。
4. 修改 [ **訓練 epoch 數目** ] 參數。
5. 按一下 [ **執行**]。

您現在可以繼續修改及執行此實驗版本，建立新的執行歷程記錄以記錄您的工作。

<!-- Images -->
[runhistory]:./media/manage-experiment-iterations/viewrunhistory.jpg


<!-- Module References -->
[linear-regression]: https://msdn.microsoft.com/library/azure/31960a6f-789b-4cf7-88d6-2e1152c0bd1a/
