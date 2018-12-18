---
title: Team Data Science Process 生命週期的客戶接受度階段 - Azure | Microsoft Docs
description: 資料科學專案客戶接受度階段的目標、工作和交付項目
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
ms.openlocfilehash: 82819ab59d9bab6f256cd01a376f62ac8dac26b3
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837135"
---
# <a name="customer-acceptance"></a>客戶接受度

本文將概要說明與 Team Data Science Process (TDSP) 客戶接受度階段相關聯的目標、工作和交付項目。 此流程會提供建議的生命週期，供您建構資料科學專案。 生命週期可描繪出專案一般執行時 (通常會反覆進行) 的主要階段：

   1. **了解商務**
   2. **資料取得與認知**
   3. **模型化**
   4. **部署**
   5. **客戶接受度**

以下是 TDSP 生命週期的視覺表示法： 

![TDSP 生命週期](./media/lifecycle/tdsp-lifecycle2.png) 


## <a name="goal"></a>目標
**完成專案的交付項目**︰確認生產環境中的管線、模型及其部署滿足客戶的目標。

## <a name="how-to-do-it"></a>作法
此階段會解決兩項主要工作︰

   * **系統驗證**︰確認所部署的模型和管線符合客戶的需求。
   * **專案移交**︰將專案移交給將在生產環境中執行系統的實體。

客戶應該驗證系統是否符合其商務需求並以可接受的精確度回答問題，以將系統部署到生產環境以供其用戶端的應用程式使用。 所有文件皆已完成並檢閱。 專案已移交給負責運作的實體。 比方說，這個實體可能是 IT 或客戶的資料科學小組，或負責在生產環境中執行系統的客戶代理人。 

## <a name="artifacts"></a>構件
此最終階段所產生的主要成品是**客戶專案結束報表**。 此份技術報告包含可用於了解如何操作系統的所有專案詳細資料。 TDSP 提供[結束報告](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Project/Exit%20Report.md)範本。 您可以直接使用範本，或者針對特定的用戶端需求加以自訂。 


## <a name="next-steps"></a>後續步驟

以下是 TDSP 生命週期中每個步驟的連結：

   1. [了解商務](lifecycle-business-understanding.md)
   2. [資料取得與認知](lifecycle-data.md)
   3. [模型化](lifecycle-modeling.md)
   4. [部署](lifecycle-deployment.md)
   5. [客戶接受度](lifecycle-acceptance.md)

我們也會提供完整的端對端逐步解說，說明特定案例之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)一文提供有連結和縮圖描述的案例清單。 這些逐步解說示範如何將雲端、內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。 

如需如何在使用 Azure Machine Learning Studio 的 TDSP 中執行步驟的範例，請參閱[配合使用 Team Data Science Process 與 Azure Machine Learning](http://aka.ms/datascienceprocess)。
