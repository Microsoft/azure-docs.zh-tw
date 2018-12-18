---
title: 步驟 6：存取 Machine Learning Web 服務 | Microsoft Docs
description: 開發預測解決方案的逐步解說步驟 6：存取使用中的 Azure Machine Learning Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 6a65c89a-40ab-4673-8dd8-8eee0a150e3b
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2017
ms.openlocfilehash: be63a04d0fe71039b19eb6bc0678a319f622a811
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836699"
---
# <a name="walkthrough-step-6-access-the-azure-machine-learning-web-service"></a>逐步解說步驟 6：存取 Azure Machine Learning Web 服務

這是 [在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)

1. [建立機器學習服務工作區](walkthrough-1-create-ml-workspace.md)
2. [上傳現有資料](walkthrough-2-upload-data.md)
3. [建立新實驗](walkthrough-3-create-new-experiment.md)
4. [訓練及評估模型](walkthrough-4-train-and-evaluate-models.md)
5. [部署 Web 服務](walkthrough-5-publish-web-service.md)
6. **存取 Web 服務**

- - -
在此逐步解說先前的步驟中，我們已經部署一個使用我們的信用風險預測模型的 Web 服務。 現在使用者能夠將資料傳送至服務並接收結果。 

此 Web 服務是可使用 REST API，透過下列其中一種方式接收和傳回資料的 Azure Web 服務：  

* **要求/回應** - 使用者以 HTTP 通訊協定，將一或多列的信用資料傳送給服務，然後服務回應一或多組結果。
* **批次執行** - 使用者在 Azure Blob 中儲存一或多列信用資料，然後將 Blob 位置傳送給服務。 服務會給輸入 Blob 中的所有資料列評分，將結果儲存在另一個 Blob 中，再傳回該容器的 URL。  

存取傳統 Web 服務最簡單快速的方式，是透過 [Azure ML 要求-回應服務 Web 應用程式 (英文)](https://azure.microsoft.com/marketplace/partners/microsoft/azuremlaspnettemplateforrrs/) 或 [Azure ML 批次執行服務 Web 應用程式範本 (英文)](https://azure.microsoft.com/marketplace/partners/microsoft/azuremlbeswebapptemplate/)。

這些 Web 應用程式範本可以建立自訂的 Web 應用程式，該應用程式知道您的 Web 服務輸入資料和將傳回的內容。 您所需要做的就是提供 Web 服務和資料的存取權限，範本會執行其餘部分。

如需使用 Web 應用程式範本的詳細資訊，請參閱[使用 Azure Machine Learning Web 服務與 Web 應用程式範本](consume-web-service-with-web-app-template.md)。

您也可以使用以 R、C# 和 Python 程式語言為您提供的起始程式碼來開發自訂應用程式以存取 Web 服務。

您可以在[如何使用 Azure Machine Learning Web 服務](consume-web-services.md)中找到完整詳細資料。

