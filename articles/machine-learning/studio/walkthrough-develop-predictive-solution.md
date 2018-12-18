---
title: 使用 Machine Learning 建立的信用風險預測解決方案 | Microsoft Docs
description: 詳細的逐步解說說明如何在 Azure Machine Learning 中為信用風險評估建立預測分析解決方案。
keywords: 信用風險, 預測性分析解決方案, 風險評估
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
ms.assetid: 43300854-a14e-4cd2-9bb1-c55c779e0e93
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/23/2017
ms.openlocfilehash: 9d2cfeb780687e1bf0e61173b3550e26f4019115
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394143"
---
# <a name="walkthrough-develop-a-predictive-analytics-solution-for-credit-risk-assessment-in-azure-machine-learning"></a>逐步解說：在 Azure Machine Learning 中為信用風險評估開發預測分析解決方案

在本逐步解說中，我們將進一步了解在 Machine Learning Studio 中開發預測性分析解決方案的程序。 我們會在 Machine Learning Studio 中開發一個簡單的模型，然後將它部署為 Azure Machine Learning Web 服務，其中模型將可使用新資料來進行預測。 

本逐步解說會假設您之前已至少使用過一次 Machine Learning Studio，而且對機器學習服務概念有一些了解。 但不會假設您對上述任一方面有所專精。

如果您未曾使用過 **Azure Machine Learning Studio**，您可以從[在 Azure Machine Learning Studio 中建立您的第一個資料科學實驗](create-experiment.md)教學課程著手。 該教學課程會引導您完成第一次使用 Machine Learning Studio 的程序。 它會說明基本概念，讓您了解如何將模組拖放到您的實驗、將它們連接在一起、執行實驗及檢視結果。 另一個可協助您開始使用的工具是一張圖，此圖提供 Machine Learning Studio 的功能概觀。 您可以從這裡下載並列印它：[Azure Machine Learning Studio 功能的概觀圖](studio-overview-diagram.md)。
 
如果您大致上算是機器學習服務的新手，有一系列影片可為您提供協助。 此系列稱為[適用於初學者的資料科學](data-science-for-beginners-the-5-questions-data-science-answers.md)，它可透過使用日常語言和概念，為您提供機器學習服務的絕佳簡介。


[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]
 

## <a name="the-problem"></a>問題

假設您必須根據某個人在信用申請書上提供的資訊預測其信用風險。  

信用風險評估是一個複雜的問題，但是我們可以針對這個逐步解說將它稍微簡化。 我們將使用它作為一個範例，以說明您如何使用 Microsoft Azure Machine Learning 來建立預測性分析解決方案。 為了這麼做，我們將使用 Azure Machine Learning Studio 和 Machine Learning Web 服務。  

## <a name="the-solution"></a>解決方案

在這個詳細的逐步解說中，我們將從可公開取得的信用風險資料開始著手，然後根據該資料來開發並訓練預測性模型。 接著，我們會將該模型部署為 Web 服務，以便供其他人用來評估信用風險。

為了建立此信用風險評估解決方案，我們將依照下列步驟操作：  

1. [建立機器學習服務工作區](walkthrough-1-create-ml-workspace.md)
2. [上傳現有資料](walkthrough-2-upload-data.md)
3. [建立實驗](walkthrough-3-create-new-experiment.md)
4. [訓練及評估模型](walkthrough-4-train-and-evaluate-models.md)
5. [部署 Web 服務](walkthrough-5-publish-web-service.md)
6. [存取 Web 服務](walkthrough-6-access-web-service.md)

> [!TIP] 
> 您可以在 [Azure AI 資源庫](https://gallery.cortanaintelligence.com)中，找到我們在這個逐步解說中所開發實驗的工作複本。 移至 **[逐步解說 - 信用風險預測 (英文)](https://gallery.cortanaintelligence.com/Experiment/Walkthrough-Credit-risk-prediction-1)**，然後按一下 [Open in Studio] \(在 Studio 中開啟)，以將實驗的複本下載到您的 Machine Learning Studio 工作區。
> 
> 此逐步解說是以簡化版的[二進位分類：信用風險預測 (英文)](http://go.microsoft.com/fwlink/?LinkID=525270) 範例實驗為基礎，[資源庫](http://gallery.cortanaintelligence.com/)中也有提供此實驗。
