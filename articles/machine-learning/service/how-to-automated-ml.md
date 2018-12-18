---
title: 什麼是自動化機器學習服務 - Azure Machine Learning
description: 在此文章中，您可以了解自動化機器學習服務。 Azure Machine Learning 服務可自動為您挑選演算法，並從中產生模型。 自動化機器學習服務可使用您所提供以為模型選取最佳演算法的參數與準則，協助節省您的時間。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: krishnan
author: krishnaanumalasetty
ms.date: 9/24/2018
ms.openlocfilehash: 2a9c05b68d05102fab80b2aa8fb1c1dad8a367ea
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960034"
---
# <a name="what-is-automated-machine-learning"></a>什麼是自動化機器學習服務？

在此文章中，您可以了解自動化機器學習服務。 Azure Machine Learning 服務可自動為您挑選演算法，並從中產生模型。 自動化機器學習服務可使用您所提供以為模型選取最佳演算法的參數與準則，協助節省您的時間。

## <a name="how-it-works"></a>運作方式

1. 您設定嘗試要解決的機器學習服務問題類型。 支援兩種監督式學習類別：
   + 分類
   + 迴歸

   查看 Azure Machine Learning 可在定型時嘗試的[模型清單](how-to-configure-auto-train.md#select-your-experiment-type)。

1. 您指定用於將資料定型的來源與格式。 資料必須要加上標籤，而且儲存在您的開發環境或 Azure Blob 儲存體中。 如果資料儲存在開發環境上，它必須位於與您的定型指令碼相同的目錄。 此目錄會複製到您為定型選取的計算目標。

    在定型指令碼中，可以將資料讀入 Numpy 陣列或 Pandas 資料框架。

    您可以設定選取定型和驗證資料的分割選項，或可以指定不同的定型和驗證資料集。

1. 設定用來將模型定型的[計算目標](how-to-set-up-training-targets.md)。

1. 設定自動化機器學習設定。 這可以控制當 Azure Machine Learning 逐一查看不同的模型、超參數設定，以及在決定最佳模型時要查看哪些計量時會使用的參數。 

1. 提交定型回合。

在定型期間，Azure Machine Learning 服務會建立數個管道，這些管道分別會嘗試不同的演算法與參數。 在達到您提供的反覆項目限制之後，或達到您為計量指定的目標值時，它隨即停止。

[ ![自動化機器學習服務](./media/how-to-automated-ml/automated-machine-learning.png) ](./media/how-to-automated-ml/automated-machine-learning.png#lightbox)

您可以檢查所記錄的回合資訊，其中包含回和執行期間所收集的計量。 定型回合也會產生 Python 序列化物件 (.pkl 檔案)，其中包含模型與資料前置處理。

## <a name="next-steps"></a>後續步驟

查看範例並了解如何使用自動化機器學習服務來建置模型：

+ [教學課程：使用自動化的 Azure Machine Learning 自動將分類模型定型](tutorial-auto-train-models.md)

+ [如何設定自動定型的設定](how-to-configure-auto-train.md)

+ [如何在遠端資源使用自動定型](how-to-auto-train-remote.md) 
