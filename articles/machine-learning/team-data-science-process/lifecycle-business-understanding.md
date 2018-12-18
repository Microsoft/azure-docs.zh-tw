---
title: Team Data Science Process 生命週期的了解商務階段 - Azure | Microsoft Docs
description: 資料科學專案了解商務階段的目標、工作和交付項目
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
ms.openlocfilehash: 6d8eedbbf4a682443e73ecb9cf9496f3cdd1cd9d
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837196"
---
# <a name="business-understanding"></a>了解商務

本文將概要說明與 Team Data Science Process (TDSP) 了解商務階段相關聯的目標、工作和交付項目。 此流程會提供建議的生命週期，供您建構資料科學專案。 生命週期可描繪出專案一般執行時 (通常會反覆進行) 的主要階段：

   1. **了解商務**
   2. **資料取得與認知**
   3. **模型化**
   4. **部署**
   5. **客戶接受度**

以下是 TDSP 生命週期的視覺表示法： 

![TDSP 生命週期](./media/lifecycle/tdsp-lifecycle2.png) 


## <a name="goals"></a>目標
* 指定要做為模型目標以及要使用誰的相關標準來判斷專案是否成功的重要變數。
* 找到企業有權存取或需要取得的相關資料來源。

## <a name="how-to-do-it"></a>作法
此階段會解決兩項主要工作︰ 

   * **定義目標**︰與客戶和其他專案關係人合作，以了解並找出企業的問題。 制訂定義資料科學技術可鎖定之商務目標的問題。
   * **識別資料來源**︰尋找可協助您回答定義出專案目標之問題的相關資料。

### <a name="define-objectives"></a>定義目標
1. 此步驟的中心目標是要識別分析工作需要預測的重要業務變數。 我們把這些變數稱為*模型目標*，我們使用與其相關聯的標準決定專案是否成功。 這類目標的兩個範例是銷售預測或訂單被詐騙的機率。

2. 詢問並改進相關、特定且模稜兩可的「尖銳」問題以定義專案目標。 資料科學是使用名稱和數字來回答這類問題的程序。 如需詢問尖銳問題的詳細資訊，請參閱[如何進行資料科學](https://blogs.technet.microsoft.com/machinelearning/2016/03/28/how-to-do-data-science/) \(英文\) 部落格。 您通常會使用資料科學或機器學習來回答五種問題︰
 
   * 有多少？ (迴歸)
   * 什麼類別？ (分類)
   * 哪個群組？ (叢集)
   * 這很奇怪嗎？ (異常偵測)
   * 應採用哪個選項？ (建議)

   決定這些問題中要詢問哪些，以及如何解答以達到商務目標。

3. 指定專案小組成員的角色和責任以定義專案小組。 在您發現更多資訊時，開發可逐一查看的高階里程碑計劃。 

4. 定義成功標準。 比方說，您想要獲得一份客戶流失預測。 您需要這個三個月專案結束時有 "x" 百分比的準確率。 使用此資料，您可以提供客戶促銷活動以減少流失。 標準必須夠**聰明 (SMART)**： 

   * **S**pecific (具體) 
   * **M**easurable (可測量)
   * **A**chievable (可達成) 
   * **R**elevant (具有相關性) 
   * **T**ime-bound (有時限) 

### <a name="identify-data-sources"></a>識別資料來源
識別包含已知的尖銳問題解答範例之資料來源。 尋找下列資料︰

* 與問題相關的資料。 您有目標的量值和與目標相關的特徵嗎？
* 可精確測量模型目標和感興趣之特徵的資料。

舉例來說，您可能發現現有的系統必須收集並記錄其他類型的資料，才能解決問題並達成專案目標。 在此情況下，您可能會想要尋找外部資料來源或更新系統以收集新資料。

## <a name="artifacts"></a>構件
以下是此階段的交付項目︰

   * [規範文件](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Project/Charter.md)：TDSP 專案結構定義中會提供標準範本。 規範文件是即時文件。 當您進行新的探索，以及當商務需求變更時，您就要更新整個專案的範本。 關鍵在於要反覆審視這份文件，並隨著探索程序的進行，新增更多詳細資料。 讓客戶和其他專案關係人參與變更，並清楚地說明進行這些變更的原因。  
   * [資料來源](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Data_Report/Data%20Defintion.md#raw-data-sources) \(英文\)：TDSP 專案 **Data report** 資料夾中 **Data definition** 報告的 **Raw data sources** 區段包含資料來源。 此區段指定未經處理資料的原始位置和目的地位置。 在後面的階段中，您要填入其他詳細資料，例如用來將資料移至分析環境的指令碼。  
   * [資料字典](https://github.com/Azure/Azure-TDSP-ProjectTemplate/tree/master/Docs/Data_Dictionaries) \(英文\)：本文件提供用戶端所提供之資料的說明。 這些說明包含結構描述的相關資訊 (資料類型、驗證規則相關資訊 (如果有))，以及實體關聯性圖表 (如果有)。

## <a name="next-steps"></a>後續步驟

以下是 TDSP 生命週期中每個步驟的連結：

   1. [了解商務](lifecycle-business-understanding.md)
   2. [資料取得與認知](lifecycle-data.md)
   3. [模型化](lifecycle-modeling.md)
   4. [部署](lifecycle-deployment.md)
   5. [客戶接受度](lifecycle-acceptance.md)

我們也會提供完整的端對端逐步解說，說明特定案例之程序中的所有步驟。 [範例逐步解說](walkthroughs.md)一文提供有連結和縮圖描述的案例清單。 這些逐步解說示範如何將雲端、內部部署工具與服務組合成工作流程或管線，以建立智慧型應用程式。 

如需如何在使用 Azure Machine Learning Studio 的 TDSP 中執行步驟的範例，請參閱[配合使用 Team Data Science Process 與 Azure Machine Learning](http://aka.ms/datascienceprocess)。
