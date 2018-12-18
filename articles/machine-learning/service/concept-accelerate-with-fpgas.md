---
title: 什麼是 FPGA 和 Project Brainwave？ - Azure Machine Learning
description: 了解如何在 Azure 上使用 FPGA 為模型和深度類神經網路加速。 本文所介紹的是現場可程式化閘陣列 (FPGA)，並說明 Azure Machine Learning 如何在您對 Azure FPGA 部署模型時提供即時人工智慧 (AI)。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: tedway
author: tedway
ms.reviewer: jmartens
ms.date: 9/24/2018
ms.openlocfilehash: adcd812bc63f98e4cbde949946bf32a037c6d704
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158680"
---
# <a name="what-is-fpga-and-project-brainwave"></a>什麼是 FPGA 和 Project Brainwave？

本文所介紹的是現場可程式化閘陣列 (FPGA)，並說明 Azure Machine Learning 如何在您對 Azure FPGA 部署模型時提供即時人工智慧 (AI)。

FPGA 包含可程式化邏輯區塊的陣列，以及可重新設定互連的階層。 互連可讓您在製造之後以不同方式設定這些區塊。 與其他晶片相比，FPGA 結合了可編程性和效能。

## <a name="fpgas-vs-cpu-gpu-and-asic"></a>FPGA 與CPU、GPU、ASIC 的比較

![Azure Machine Learning FPGA 比較](./media/concept-accelerate-with-fpgas/azure-machine-learning-fpga-comparison.png)

|處理器||說明|
|---|:-------:|------|
|應用程式特定的積體電路|ASIC|自訂電路 (例如，Google 的 TensorFlow 處理器 (TPU)) 可提供最高效率。 這些電路無法隨著需求的變更加以重新設定。|
|現場可程式化閘陣列|FPGA|FPGA 可提供接近 ASIC 的效能，但有彈性和重新設定能力而能在一段時間後實作新的邏輯，Azure 所提供的即為此類。|
|圖形處理器|GPU|這是頗受歡迎的 AI 運算選擇，由於能提供平行處理功能，使得它在轉譯影像的速度快過 CPU。|
|中央處理器|CPU|一般用途的處理器，其效能不適合圖形和影片處理。|

## <a name="project-brainwave-on-azure"></a>Azure 的 Project Brainwave

[Project Brainwave](https://www.microsoft.com/research/project/project-brainwave/) 是 Microsoft 所推出的經濟實惠硬體架構，其以 Intel 的 FPGA 裝置為基礎，可供資料科學家和開發人員用於加快即時 AI 的運算速度。  這個具有 FPGA 功能的架構可提供**效能**、**彈性**和**擴展能力**，並可在 Azure 上取得。

**FPGA 可以實現低延遲的即時推斷請求。** 批次處理的意思是，收集大量資料並饋送至處理器，以改善硬體使用率。 批次處理可能會造成延遲 (因為需要處理更多資料)，但可以改善輸送量。 使用類神經處理器的 Project Brainwave 實作不需要批次處理；因此，其延遲比 CPU 和 GPU 低好幾倍。

### <a name="reconfigurable-power"></a>可重新設定的電源
**FPGA 可以重新設定為不同類型的機器學習模型。** 擁有此種彈性，就更易於依據最佳的數值有效位數和所用的記憶體模型來加速應用程式。

目前正在定期開發新的機器學習技術，Project Brainwave 的硬體設計也在迅速發展。 由於 FPGA 可重新設定，因此可與 AI 演算法快速變化的需求保持同步。

### <a name="whats-supported-on-azure"></a>Azure 上所支援的項目
**Microsoft Azure 是 FPGA 全球最大的雲端投資。** 您可以在 Azure 的縮放基礎結構上執行 Project Brainwave。

目前，Project Brainwave 支援：
+ 影像分類和辨識案例
+ TensorFlow 部署
+ DNN：ResNet 50、ResNet 152、VGG-16、SSD-VGG 和 DenseNet-121
+ Intel FPGA 硬體 

使用這個具有 FPGA 功能的硬體架構，已定型的類神經網路執行速度會更快，延遲也較低。 Project Brainwave 可以在 FPGA 上平行處理預先定型的深度類神經網路 (DNN)，相應放大您的服務。 DNN 可以預先定型為深度 Featurizer 以傳輸學習，也可以使用更新過的加權來微調。

### <a name="scenarios-and-applications"></a>案例和應用程式

Project Brainwave 會與 Azure Machine Learning 整合。 Microsoft 使用 FPGA (以評估 DNN)、Bing 搜尋順位排序和軟體定義網路 (SDN) 加速來減少延遲，同時釋放 CPU 以供執行其他工作。

以下為在 Project Brainwave 架構上使用 FPGA 的案例：
+ [自動化光學檢查系統](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)

+ [地表覆蓋對應](https://blogs.technet.microsoft.com/machinelearning/2018/05/29/how-to-use-fpgas-for-deep-learning-inference-to-perform-land-cover-mapping-on-terabytes-of-aerial-images/)

## <a name="deploy-to-fpgas-on-azure"></a>部署到 Azure 上的 FPGA

以下工作流程可在 Azure 中建立影像辨識服務，過程中會使用支援的 DNN 作為 Featurizer 以將服務部署至 Azure FPGA：

1. 使用 Azure Machine Learning SDK for Python 建立服務定義，此定義檔會以 TensorFlow 為基礎來描述圖形管線 (輸入、Featurizer 和分類器)。 部署命令會自動將定義和圖表壓縮為 ZIP 檔案，然後將 ZIP 檔案上傳到 Azure Blob 儲存體。  DNN 已部署於 Project Brainwave 以便在 FPGA 上執行。

1. 使用 SDK 與 Azure Blob 儲存體中的 ZIP 檔案註冊模型。

1. 使用 SDK 部署帶有所註冊模型的服務。

您可以透過**[在 FPGA 上將模型部署為 Web 服務](how-to-deploy-fpga-web-service.md)** 這篇文章，開始將已定型的 DNN 模型部署至 Azure 雲端中的 FPGA。


## <a name="next-steps"></a>後續步驟

請參閱這些影片與部落格：

+ [超大規模硬體：Azure + FPGA 頂級規模的 ML：2018 組建 (英文影片)](https://www.youtube.com/watch?v=BMgQAHIx2eY)

+ [深入了解 Microsoft FPGA 架構的可設定雲端 (英文影片)](https://channel9.msdn.com/Events/Build/2017/B8063)

+ [適用於即時 AI 的 Project Brainwave：專案首頁](https://www.microsoft.com/research/project/project-brainwave/)
