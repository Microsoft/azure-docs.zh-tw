---
title: 使用適用於電腦視覺的 Azure Machine Learning 套件建置及部署影像相似性模型。
description: 了解如何使用適用於電腦視覺的 Azure Machine Learning 套件，來建置、定型、測試和部署電腦視覺影像相似性模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: netahw
author: nhaiby
ms.date: 05/20/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 371903e1ee080d2b98fd46ac4d6d9838416e1335
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973417"
---
# <a name="build-and-deploy-image-similarity-models-with-azure-machine-learning"></a>使用 Azure Machine Learning 建置及部署影像相似性模型

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


在本文中，您將了解如何使用適用於電腦視覺的 Azure Machine Learning 套件 (AMLPCV) 來定型、測試及部署影像相似性模型。 如需此套件的概觀及其詳細參考文件，請參閱 https://aka.ms/aml-packages/vision \(英文\)。

電腦視覺領域中一大部分的問題，都可以使用影像相似性來解決。 這些問題包括建置可回答問題的模型，例如：
+ _物件存在於影像中嗎？例如，「狗」、「車子」、「船」等等_
+ _此病患的視網膜掃描顯示的眼睛疾病嚴重性等級為何？_

使用 AMLPCV 建置及部署此模型時，您會經歷下列步驟：
1. 資料集建立
2. 影像視覺化和註釋
3. 影像增強
4. 深度類神經網路 (DNN) 模型定義
5. 分類器定型
6. 評估和視覺化
7. Web 服務部署
8. Web 服務負載測試

[CNTK](https://www.microsoft.com/cognitive-toolkit/) 當作深入學習架構使用，定型則會在採用 GPU 機器的本機上執行，例如 ([Deep learning Data Science VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning?tab=Overview))，而且部署會使用 Azure ML 作業化 CLI。

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

1. 您必須設定並安裝下列帳戶和應用程式：
   - Azure Machine Learning 測試帳戶 
   - Azure Machine Learning 模型管理帳戶
   - 已安裝 Azure Machine Learning Workbench

   如果還未建立或安裝這三者，請依照 [Azure Machine Learning 快速入門和 Workbench 安裝](../desktop-workbench/quickstart-installation.md)一文來進行。 

1. 您必須安裝適用於電腦視覺的 Azure Machine Learning 套件。 若要了解如何安裝此套件，請移至 https://aka.ms/aml-packages/vision \(英文\)。

## <a name="sample-data-and-notebook"></a>範例資料和 Notebook

### <a name="get-the-jupyter-notebook"></a>取得 Jupyter Notebook

下載 Notebook 以自行執行此處所述的範例。

> [!div class="nextstepaction"]
> [取得 Jupyter Notebook](https://aka.ms/aml-packages/vision/notebooks/object_detection) \(英文\)

### <a name="load-the-sample-data"></a>載入範例資料

下列範例使用包含 63 個餐具影像的資料集。 每個影像都有標示屬於哪一個類別 (bowl、cup、cutlery、plate)。 此範例中的影像數目不多，因此可以快速執行此範例。 在實務上，每個類別應該至少提供 100 個影像。 所有影像都位於 *"../sample_data/imgs_recycling/"*，稱為 "bowl"、"cup"、"cutlery" 和 "plate" 的子目錄中。

![Azure Machine Learning 資料集](media/how-to-build-deploy-image-classification-models/recycling_examples.jpg)

## <a name="next-steps"></a>後續步驟

在這些文章中，深入了解適用於電腦視覺的 Azure Machine Learning 套件：

+ 了解如何[改善此模型的精確度](how-to-improve-accuracy-for-computer-vision-models.md)。

+ 參閱[套件概觀](https://aka.ms/aml-packages/vision) \(英文\)。

+ 瀏覽此套件的[參考文件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision)。

+ 了解[適用於 Azure Machine Learning 的其他 Python 套件](reference-python-package-overview.md)。
