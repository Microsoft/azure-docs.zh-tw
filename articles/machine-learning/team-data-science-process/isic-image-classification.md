---
title: 使用適用於電腦視覺的 Azure Machine Learning 套件和 Team Data Science Process (TDSP) 進行影像分類 | Microsoft Docs
description: 說明如何使用 Team Data Science Process (TDSP) 和適用於電腦視覺的 Azure Machine Learning 套件進行影像分類。
services: machine-learning, team-data-science-process
documentationcenter: ''
author: deguhath
ms.author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: b8fbef77-3e80-4911-8e84-23dbf42c9bee
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/18/2018
ms.openlocfilehash: ee2e797f3838b8b6b36174d14c73e97fe9790315
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49392807"
---
# <a name="skin-cancer-image-classification-with-the-azure-machine-learning-package-for-computer-vision-and-team-data-science-process"></a>使用適用於電腦視覺的 Azure Machine Learning 套件和 Team Data Science Process 進行皮膚癌影像分類

本文說明如何使用[適用於電腦視覺的 Azure Machine Learning 套件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision?view=azure-ml-py-latest)來定型、測試及部署「影像分類」模型。 此範例會在 [Azure Machine Learning Workbench](https://docs.microsoft.com/azure/machine-learning/service/quickstart-installation) 中使用 Team Data Science Process (TDSP) 結構和範本。 本逐步解說提供完整的範例。 它會使用 [Microsoft Cognitive Toolkit](https://www.microsoft.com/en-us/cognitive-toolkit/) 作為深度學習架構，並且在[資料科學虛擬機器](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft-ads.dsvm-deep-learning?tab=Overview) GPU 電腦上執行定型。 部署會使用 Azure Machine Learning 操作化 CLI。

許多電腦視覺領域中的應用都可用影像分類問題的角度來構思。 這些包括建置可解答簡單問題的模型，例如「影像中是否有某個物件？」 其中物件有可能是狗、車或船。 這也包括解答更複雜問題，例如「此病患視網膜掃描顯示的眼睛疾病嚴重性等級為何？」 適用於電腦視覺的 Azure Machine Learning 套件可簡化影像分類資料處理及模型化管線。 

## <a name="link-to-the-github-repository"></a>GitHub 存放庫的連結
本文是範例的相關摘要文件。 您可以在 [GitHub 網站](https://github.com/Azure/MachineLearningSamples-AMLVisionPackage-ISICImageClassification)上找到更廣泛的文件。

## <a name="team-data-science-process-walkthrough"></a>Team Data Science Process 逐步解說

本逐步解說採用 [Team Data Science Process](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/overview) 生命週期。 本逐步解說涵蓋下列生命週期步驟。

### <a name="1-data-acquisitionhttpsgithubcomazuremachinelearningsamples-amlvisionpackage-isicimageclassificationblobmastercode01dataacquisitionandunderstanding"></a>[1.資料擷取](https://github.com/Azure/MachineLearningSamples-AMLVisionPackage-ISICImageClassification/blob/master/code/01_data_acquisition_and_understanding)
影像分類工作會使用「國際皮膚影像合作」(International Skin Imaging Collaboration, ISIC) 資料集。 ISIC 是學界與業界之間的一項合作，旨在推動數位皮膚影像的應用，以期能研究及協助降低黑色素瘤的死亡率。 [ISIC 封存](https://isic-archive.com/#images)包含超過 13,000 張標示為良性或惡性的皮膚病灶影像。 請從 ISIC 封存下載影像範例。

### <a name="2-modelinghttpsgithubcomazuremachinelearningsamples-amlvisionpackage-isicimageclassificationblobmastercode02modeling"></a>[2.模型化](https://github.com/Azure/MachineLearningSamples-AMLVisionPackage-ISICImageClassification/blob/master/code/02_modeling)
在模型化步驟中，會執行下列子步驟。

#### <a name="dataset-creation"></a>資料集建立

提供本機磁碟上的影像根目錄，以在適用於電腦視覺的 Azure Machine Learning 套件中產生資料集物件。 

#### <a name="image-visualization-and-annotation"></a>影像視覺化和註釋

將資料集物件中的影像視覺化，並視需要更正標籤。

#### <a name="image-augmentation"></a>影像增強

使用 [imgaug](https://github.com/aleju/imgaug) 程式庫中所述的轉換來增強資料集物件。

#### <a name="dnn-model-definition"></a>DNN 模型定義

定義定型步驟中所使用的模型架構。 適用於電腦視覺的 Azure Machine Learning 套件支援六個預先定型的深度類神經網路模型：AlexNet、Resnet-18、Resnet-34、Resnet-50、Resnet-101 和 Resnet-152。

#### <a name="classifier-training"></a>分類器定型

使用預設或自訂參數定型類神經網路。

#### <a name="evaluation-and-visualization"></a>評估和視覺化

提供功能來評估獨立測試資料集上的定型模型效能。 評估計量包括準確度、精確度、召回率及 ROC 曲線。

這些子步驟在對應的 Jupyter Notebook 中有詳細說明。 此 Notebook 也提供調整參數 (例如學習率、迷你批次大小和中輟率) 以進一步改善模型效能的指引。

### <a name="3-deploymenthttpsgithubcomazuremachinelearningsamples-amlvisionpackage-isicimageclassificationblobmastercode03deployment"></a>[3.部署](https://github.com/Azure/MachineLearningSamples-AMLVisionPackage-ISICImageClassification/blob/master/code/03_deployment)

此步驟會將產生自模型化步驟的模型變成可操作。 它會導入先決條件和必要的設定。 此外，也會說明 Web 服務的運用。 在本教學課程中，您會了解如何使用適用於電腦視覺的 Azure Machine Learning 套件來建置深度學習模型，然後讓模型能夠在 Azure 中進行操作。

## <a name="next-steps"></a>後續步驟
- 閱讀有關[適用於電腦視覺的 Azure Machine Learning 套件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision?view=azure-ml-py-latest)的額外文件。
- 閱讀 [Team Data Science Process](https://aka.ms/tdsp) 文件來開始使用。


## <a name="references"></a>參考

* [適用於電腦視覺的 Azure Machine Learning 套件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision?view=azure-ml-py-latest) \(英文\)
* [Azure Machine Learning Workbench](https://docs.microsoft.com/azure/machine-learning/service/quickstart-installation)
* [資料科學虛擬機器](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/microsoft-ads.dsvm-deep-learning?tab=Overview)

