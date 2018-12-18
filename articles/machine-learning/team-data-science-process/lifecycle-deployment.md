---
title: Team Data Science Process 生命週期的部署階段 - Azure | Microsoft Docs
description: 資料科學專案部署階段的目標、工作和交付項目
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: 9c54e93eca181331117f2f7faad3e7d07274412d
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837026"
---
# <a name="deployment"></a>部署

本文將概要說明與 Team Data Science Process (TDSP) 部署相關聯的目標、工作和交付項目。 此流程會提供建議的生命週期，供您建構資料科學專案。 生命週期可描繪出專案一般執行時 (通常會反覆進行) 的主要階段：

   1. **了解商務**
   2. **資料取得與認知**
   3. **模型化**
   4. **部署**
   5. **客戶接受度**

以下是 TDSP 生命週期的視覺表示法： 

![TDSP 生命週期](./media/lifecycle/tdsp-lifecycle2.png) 


## <a name="goal"></a>目標
將有資料管線的模型部署至生產環境或類生產環境，以進行最後的使用者接受度測試。 

## <a name="how-to-do-it"></a>作法
此階段解決的主要工作︰

**實作模型**︰將模型和管線部署到生產環境或類生產環境以供取用應用程式。

### <a name="operationalize-a-model"></a>模型運作
在您擁有一組妥善執行的模型後，就可以運作這些模型，供其他應用程式取用。 預測可根據商業需求，以即時或批次的形式執行。 若要部署模型，您要使用開放的 API 介面公開模型。 介面可讓模型能從各種應用程式輕鬆地取用，例如：

   * 線上網站
   * 試算表 
   * 儀表板
   * 企業營運應用程式 
   * 後端應用程式 

如需使用 Azure Machine Learning Web 服務實作模型的範例，請參閱[部署 Azure Machine Learning Web 服務](../studio/publish-a-machine-learning-web-service.md)。 對於您部署的生產模型和資料管線，建議您在其中建置遙測和監視。 這種做法有助於後續的系統狀態報告和疑難排解。  

## <a name="artifacts"></a>構件

* 顯示系統健康情況和重要計量的狀態儀表板
* 具有部署詳細資料的最終模型報告
* 最終的方案架構文件


## <a name="next-steps"></a>後續步驟

以下是 TDSP 生命週期中每個步驟的連結：

   1. [了解商務](lifecycle-business-understanding.md)
   2. [資料取得與認知](lifecycle-data.md)
   3. [模型化](lifecycle-modeling.md)
   4. [部署](lifecycle-deployment.md)
   5. [客戶接受度](lifecycle-acceptance.md)

我們也會提供完整的端對端逐步解說，說明特定案例之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)一文提供有連結和縮圖描述的案例清單。 這些逐步解說示範如何將雲端、內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。 

如需如何在使用 Azure Machine Learning Studio 的 TDSP 中執行步驟的範例，請參閱[配合使用 Team Data Science Process 與 Azure Machine Learning](http://aka.ms/datascienceprocess)。
