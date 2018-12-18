---
title: 在 Azure 上搭配深度學習資料科學虛擬機器來運用資料科學 | Microsoft Docs
description: 如何使用深度學習資料科學 VM 執行數個常見的資料科學工作。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: 34ef0b10-9270-474f-8800-eecb183bbce4
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: 2053ed8cc420183d493097eeb2cd2ad93c82c70c
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32167240"
---
# <a name="using-the-deep-learning-virtual-machine"></a>使用深度學習虛擬機器

佈建深度學習虛擬機器 (DLVM) 之後，您就可以在電腦視景和語言理解之類的領域中，開始建立深度類神經網路模型，以建置 AI 應用程式。 

Deep Learning VM for AI 上提供許多工具。 [深度學習和 AI 架構頁面](dsvm-deep-learning-ai-frameworks.md)包含如何使用這些工具的詳細資料。 

## <a name="deep-learning-tutorials-and-walkthroughs"></a>深度學習教學課程和逐步解說

除了以架構為基礎的範例，我們也提供一套完整的逐步解說，都已在 DLVM 上驗證過。 這些逐步解說有助於您在影像和文字/語言理解之類的領域中，快速開發深度學習應用程式。 我們會持續新增更多跨不同領域和技術的端對端教學課程。   


- [跨不同架構執行類神經網路](https://github.com/ilkarman/DeepLearningFrameworks)：完整的逐步解說，示範如何將程式碼從一種架構移轉至另一種架構。 其中也示範如何跨架構來比較模型和執行階段效能。 

- [建置端對端解決方案在影像內偵測產品的操作說明指南](https://github.com/Azure/cortana-intelligence-product-detection-from-images)：影像偵測是一種可在影像內找出並分類物體的技術。 這項技術在許多現實生活商業領域中潛藏龐大的商機。 例如，零售商可以利用這項技術來判斷客戶從貨架上挑選什麼產品。 這項資訊進而有助於商店管理產品庫存。 

- [從 PubMed 摘要的具名實體擷取](https://docs.microsoft.com/azure/machine-learning/preview/scenario-tdsp-biomedical-recognition) 本教學課程示範如何從非結構化文字中擷取具名實體，例如藥物名稱或疾病名稱。 它會在 1800 萬筆 PubMed 摘要的文字語料庫上訓練自訂字組內嵌、使用該模型針對實體擷取建立長短期記憶 (LSTM) 遞迴式類神經網路模型，並示範該特定領域字組內嵌模型在實體擷取的表現可優於一般字組內嵌。

- [音訊的深度學習](https://blogs.technet.microsoft.com/machinelearning/2018/01/30/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure/) 本教學課程示範如何在[都市音效資料集](https://serv.cusp.nyu.edu/projects/urbansounddataset/urbansound8k.html)偵測音訊事件來訓練深度學習模型，並提供如何使用音訊資料的概觀。

- [文字文件分類](https://github.com/anargyri/lstm_han)：本逐步解說示範如何建立和定型兩個不同的類神經網路架構：階層式注意力網路和長短期記憶體 (LSTM) 網路。 這些類神經網路會使用深入學習的 Keras API 將文字文件分類。 Keras 是三個最受歡迎深度學習架構的前端：Microsoft Cognitive Toolkit、TensorFlow 和 Theano。

## <a name="next-steps"></a>後續步驟

[範例頁面](dsvm-samples-and-walkthroughs.md)提供指示來取得每個架構的 VM 上預先載入的程式碼範例，協助您快速入門。 
