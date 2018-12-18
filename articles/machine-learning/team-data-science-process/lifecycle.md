---
title: Team Data Science Process 生命週期 - Azure | Microsoft Docs
description: 執行資料科學專案所需的步驟
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: b1f677bb-eef5-4acb-9b3b-8a5819fb0e78
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: dca71a13db33f44fed551e4791625c5667cfdc21
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837492"
---
# <a name="the-team-data-science-process-lifecycle"></a>Team Data Science Process 生命週期

Team Data Science Process (TDSP) 會提供建議的生命週期供您建構資料科學專案。 生命週期會概述專案在執行時通常會遵循之從開始到完成的步驟。 如果您使用另一個資料科學生命週期，例如資料採礦的跨業界標準流程 (Cross Industry Standard Process for Data Mining) [(CRISP-DM)](https://wikipedia.org/wiki/Cross_Industry_Standard_Process_for_Data_Mining)、資料庫中的知識探索 (Knowledge Discovery in Databases) [(KDD)](https://wikipedia.org/wiki/Data_mining#Process)，或您組織自己的自訂程序，您仍然可以使用以工作為基礎的 TDSP。 

此生命週期是針對要在智慧型應用程式中隨附的資料科學專案所設計。 這些應用程式會部署機器學習服務或人工智慧模型來做預測性分析。 探勘資料科學專案和臨機操作分析專案也可以從使用此程序而獲益。 但是對於這些專案，可能不需要此處所描述的一些步驟。 

TDSP 生命週期是由反覆執行的五個主要階段所組成。 這些階段包含：

   1. [了解商務](lifecycle-business-understanding.md)
   2. [資料取得與認知](lifecycle-data.md)
   3. [模型化](lifecycle-modeling.md)
   4. [部署](lifecycle-deployment.md)
   5. [客戶接受度](lifecycle-acceptance.md)

以下是 TDSP 生命週期的視覺表示法： 

![TDSP 生命週期](./media/lifecycle/tdsp-lifecycle2.png) 


TDSP 生命週期會建立為一系列反覆執行步驟的模型，可針對需要使用預測模型的工作來提供指引。 您要在打算用來建置智慧型應用程式的生產環境中部署預測模型。 此程序生命週期的目標是要繼續將資料科學專案推向清楚的參與終點。 資料科學是一種進行研究和探索的練習。 如果能使用一組採用標準化範本、妥善定義的成品與您的小組和客戶溝通工作，有助於避免誤解。 使用這些範本也會增加成功完成複雜資料科學專案的機會。

對於每個階段，我們會提供下列資訊：

   * **目標**︰特定的目標。
   * **作法**：概述特定工作和如何完成的指引。
   * **成品**：交付項目和生產的支援。

## <a name="next-steps"></a>後續步驟

我們也會提供完整的端對端逐步解說，說明特定案例之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)一文提供有連結和縮圖描述的案例清單。 這些逐步解說示範如何將雲端、內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。 

如需如何在使用 Azure Machine Learning Studio 的 TDSP 中執行步驟的範例，請參閱[配合使用 Team Data Science Process 與 Azure Machine Learning](http://aka.ms/datascienceprocess)。
