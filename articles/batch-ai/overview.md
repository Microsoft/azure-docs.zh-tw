---
title: Azure Batch AI 服務 - AI 訓練 | Microsoft Docs
description: 了解如何使用受控 Azure Batch AI 服務，在 GPU 和 CPU 叢集上定型人工智慧 (AI) 和其他機器學習模型。
services: batch-ai
documentationcenter: ''
author: alexsuttonms
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 08/01/2018
ms.author: danlep
ms.openlocfilehash: 98497812e75d07fc153e0e351331c05484164fdd
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44052694"
---
# <a name="what-is-azure-batch-ai"></a>什麼是 Azure Batch AI？

Azure Batch AI 是受控服務，可協助資料科學家和 AI 研究人員在 Azure 中大規模地將機器學習和 AI 模型定型，無需管理複雜的基礎結構。 您只需要描述計算資源、想要執行的作業，以及要在哪裡儲存模型的輸入和輸出，Batch AI 便會完成其餘工作。

您可以獨立使用 Batch AI，也可以作為較大型開發工作流程的一部分來執行模型訓練：

* 單獨使用 Batch AI 在 [GPU](../virtual-machines/linux/sizes-gpu.md) 或 CPU 叢集上訓練、測試和批次處理評分機器學習和 AI 模型。 

* 使用 [Azure Machine Learning](../machine-learning/service/overview-what-is-azure-ml.md) 或其他 [Azure AI 平台工具](https://azure.microsoft.com/overview/ai-platform/)來鎖定工作流程中的 Batch AI 叢集。 Azure ML 可提供豐富的資料準備、測試及作業記錄體驗。 Azure ML 也可以將經過訓練的模型封裝到容器中，並針對推斷或 IoT 裝置部署模型。  

## <a name="train-machine-learning-and-ai-models"></a>訓練機器學習和 AI 模型

使用 Batch AI 可訓練機器學習模型，以及深度類神經網路 (深度學習) 和其他 AI 方法。 Batch AI 內建支援熱門的開放原始碼 AI 程式庫和架構，包括 [TensorFlow](https://github.com/tensorflow/tensorflow)、[PyTorch](https://github.com/pytorch/pytorch)、[Chainer](https://github.com/chainer/chainer) 和 [MicrosoftCognitive Toolkit (CNTK)](https://github.com/Microsoft/CNTK)。

識別問題所在區域並準備好資料之後，請以互動方式使用 Batch AI 來測試模型概念。 然後，當您準備好進行大規模實驗時，啟動跨多個 GPU (具有 MPI 或其他通訊程式庫) 的作業，並以平行方式執行更多的實驗。

Batch AI 可協助您以數種方式大規模地訓練模型。 例如︰ 

|  |  |
|---------|---------|
| **散發訓練**<br/>![分散式訓練](./media/overview/distributed-training.png)  | 將跨數個網路連線 GPU 的單一作業訓練相應增加，以使用大量資料訓練較大型的網路。|
| **實驗**<br/>![測試](./media/overview/experimentation.png) | 相應放大具有多個作業的訓練。 執行參數掃掠來測試新的想法，或微調超參數以獲得最佳精確度和效能。 |
| **以平行方式執行**![以平行方式執行](./media/overview/parallel-execution.png) | 一次針對跨一組伺服器平行執行的許多模型進行訓練或評分，以便更快地完成作業。|

訓練好模型時，請使用 Batch AI 來測試模型 (如果訓練指令碼中沒有這個部分)，或執行批次評分。

## <a name="how-it-works"></a>運作方式

使用 Batch AI SDK、命令列指令碼或 Azure 入口網站，來管理計算資源並排定作業來訓練和測試 AI： 

* **佈建和調整 VM 叢集** - 選擇節點 (VM) 數目，然後選取符合訓練需求的具 GPU 功能 VM 大小或其他 VM 大小。 以自動或手動方式相應增加或減少節點數目，以便只在需要時使用資源。 

* **管理相依性和容器** - 根據預設，Batch AI 叢集會執行已預先安裝相依性的 Linux VM 映像，以在 GPU 或 CPU 上執行容器型訓練架構。 如需其他設定，請帶入自訂映像或執行啟動指令碼。

* **散發資料** - 選擇一或多個儲存體選項來管理輸入資料、指令碼和作業輸出：Azure 檔案、Azure Blob 儲存體或受控 NFS 伺服器。 Batch AI 也支援自訂儲存體解決方案，包括高效能平行檔案系統。 使用簡單的設定檔將儲存體檔案系統掛接到叢集節點和作業容器。

* **排程作業** - 將作業提交至以優先順序為基礎的作業佇列，以共用叢集資源並利用低優先順序的 VM 和保留執行個體。

* **處理失敗** - 監視作業狀態，並在可能長時間執行的作業期間發生 VM 失敗時重新啟動作業。

* **收集結果** - 輕鬆地存取輸出記錄、Stdout、Stderr 和經過訓練的模型。 設定 Batch AI 作業，將輸出直接推送至已掛接的儲存體。

作為 Azure 服務，Batch AI 可支援角色型存取控制 (RBAC) 和 Azure 虛擬網路等常用工具以確保安全性。  

## <a name="next-steps"></a>後續步驟

* 深入了解 [Batch AI 資源](resource-concepts.md)以便訓練機器學習或 AI 模型。

* 開始使用 Batch AI 來[訓練深度學習模型範例](quickstart-tensorflow-training-cli.md)。

* 查看熱門 AI 架構的[訓練配方](https://github.com/Azure/BatchAI/blob/master/recipes)範例。
