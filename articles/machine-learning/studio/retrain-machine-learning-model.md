---
title: 重新訓練機器學習模型 | Microsoft Docs
description: 了解如何在 Azure Machine Learning 中重新定型模型，以及使用新定型的模型來更新 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: d1cb6088-4f7c-4c32-94f2-f7523dad9059
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.openlocfilehash: 46aa2c209f782706357f9a928ddbaa6321abdd77
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39115522"
---
# <a name="retrain-a-machine-learning-model"></a>重新定型機器學習服務模型
在 Azure Machine Learning 中進行機器學習服務模型的實作程序時，需要定型並儲存您的模型。 接著，使用它來建立預測性 Web 服務。 接著才能在網站、儀表板及行動應用程式取用 Web 服務。 

您使用 Machine Learning 建立的模型通常不是靜態。 因為當有新資料或 API 取用者有自己的資料時，模型就必須重新定型。 

重新定型可能經常會發生。 有了程式設計重新定型 API 功能後，您能夠使用重新定型 API 以程式設計方式重新定型模型，並使用新定型的模型來更新 Web 服務。 

本文說明重新定型程序，並示範如何使用重新定型 API。

## <a name="why-retrain-defining-the-problem"></a>為何要重新定型：定義問題
進行 Machine Learning 定型程序時，會使用一組資料來將模型定型。 您使用 Machine Learning 建立的模型通常不是靜態。 因為當有新資料或 API 取用者有自己的資料時，模型就必須重新定型。

在這些情況下，程式設計 API 可方便您或 API 取用者建立能夠單次或定期使用自己的資料重新定型模型的用戶端。 接著可以評估重新定型的結果，然後更新 Web 服務 API 以使用新定型的模型。

> [!NOTE]
> 如果您現在已有訓練實驗和新式 Web 服務，建議您查看＜重新定型現有預測性 Web 服務＞而不是遵循下一節中所述的逐步解說。
> 
> 

## <a name="end-to-end-workflow"></a>端對端工作流程
此程序包含下列部分：訓練實驗以及以 Web 服務形式發佈的預測性實驗。 若要啟用已定型模型的重新定型功能，必須利用定型模型的輸出將訓練實驗發佈為 Web 服務。 這樣做可讓 API 存取模型進行重新定型。 

下列步驟適用於新式和傳統 Web 服務︰

建立初始的預測性 Web 服務︰

* 建立訓練實驗
* 建立預測性 Web 實驗
* 部署預測性 Web 服務

重新定型 Web 服務：

* 更新訓練實驗以便能重新定型
* 部署重新訓練的 Web 服務
* 使用批次執行服務程式碼重新定型模型

如需上述步驟的逐步解說，請參閱[以程式設計方式重新定型機器學習服務模型](retrain-models-programmatically.md)。

> [!NOTE] 
> 若要部署新的 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。 

如果您部署了傳統 Web 服務：

* 在預測性 Web 服務上建立新端點
* 取得 PATCH URL 和程式碼
* 使用 PATCH URL 以指出位於重新定型模型中的新端點 

如需上述步驟的逐步解說，請參閱[重新定型傳統 Web 服務](retrain-a-classic-web-service.md)。

如果您在重新定型傳統 Web 服務時遇到麻煩，請參閱[針對 Azure Machine Learning 傳統 Web 服務的重新訓練進行疑難排解](troubleshooting-retraining-models.md)。

如果您部署了新式 Web 服務：

* 登入您的 Azure Resource Manager 帳戶
* 取得 Web 服務定義
* 將 Web 服務定義匯出為 JSON
* 在 JSON 中更新 `ilearner` Blob 的參考
* 將 JSON 匯入至 Web 服務定義
* 使用新的 Web 服務定義更新 Web 服務

如需上述步驟的逐步解說，請參閱[使用 Machine Learning Management PowerShell Cmdlet 重新定型新的 Web 服務](retrain-new-web-service-using-powershell.md)。

傳統 Web 服務設定重新定型的程序包含下列步驟：

![重新定型程序概觀][1]

新式 Web 服務設定重新定型的程序包含下列步驟：

![重新定型程序概觀][7]

## <a name="other-resources"></a>其他資源
* [使用 Azure Data Factory 重新定型和更新 Azure Machine Learning 模型](https://azure.microsoft.com/blog/retraining-and-updating-azure-machine-learning-models-with-azure-data-factory/)
* [使用 PowerShell，從一個實驗中建立許多機器學習服務模型和 Web 服務端點](create-models-and-endpoints-with-powershell.md)
* [使用 API 的 AML 重新定型模型](https://www.youtube.com/watch?v=wwjglA8xllg)影片會示範如何使用重新定型 API 和 PowerShell，將 Azure Machine Learning 中所建立的機器學習服務模型重新定型。

<!--image links-->
[1]: ./media/retrain-machine-learning-model/machine-learning-retrain-models-programmatically-IMAGE01.png
[7]: ./media/retrain-machine-learning-model/machine-learning-retrain-models-programmatically-IMAGE07.png

