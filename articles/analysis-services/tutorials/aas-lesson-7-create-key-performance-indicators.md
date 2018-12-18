---
title: Azure Analysis Services 教學課程第 7 課︰建立關鍵效能指標 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立關鍵效能指標。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 1f7d54129ef9f85bb4141d26b9be8d4a0652bff8
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37448181"
---
# <a name="create-key-performance-indicators"></a>建立關鍵效能指標

在這堂課中，您可以建立關鍵效能指標 (KPI)。 根據由量值或絕對值所定義的「目標」值，KPI 用來測量「基準」量值所定義的值效能。 在報告用戶端應用程式中，KPI 可讓商務專業人士輕鬆、快速地概括了解商務成果或找出趨勢。 若要深入了解，請參閱 [KPI](https://docs.microsoft.com/sql/analysis-services/tabular-models/kpis-ssas-tabular)
  
這堂課的預估完成時間：**15 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 6 課︰建立量值](../tutorials/aas-lesson-6-create-measures.md)。   
  
## <a name="create-key-performance-indicators"></a>建立關鍵效能指標  
  
#### <a name="to-create-an-internetcurrentquartersalesperformance-kpi"></a>若要建立 InternetCurrentQuarterSalesPerformance KPI  
  
1.  在模型設計工具中，按一下 **FactInternetSales** 資料表。  
  
2.  在量值方格中，按一下空的儲存格。  
  
3.  在資料表上方的公式列中，輸入下列公式︰ 
 
    ```  
    InternetCurrentQuarterSalesPerformance :=DIVIDE([InternetCurrentQuarterSales]/[InternetPreviousQuarterSalesProportionToQTD],BLANK())  
    ```

    此量值作為 KPI 的基準量值。  
  
4.  在量值方格中，以滑鼠右鍵按一下 [InternetCurrentQuarterSalesPerformance] > [建立 KPI]。   
  
5.  在 [關鍵效能指標 (KPI)] 對話方塊中，於 [目標] 中選取 [絕對值]，然後輸入 **1.1**。  
  
7.  在左側 (低) 滑桿欄位中，輸入 **1**，然後在右側 (高) 滑桿欄位中，輸入 **1.07**。  
  
8.  在 [選取圖示樣式] 中，選取菱形 (紅色)、三角形 (黃色)、圓形 (綠色) 圖示類型。
  
    ![aas-lesson7-kpi](../tutorials/media/aas-lesson7-kpi.png)
    
    > [!TIP]  
    > 請注意，可用圖示樣式下方的可展開 [描述] 標籤。 使用各種 KPI 元素的描述，使其更容易在用戶端應用程式中識別。  
  
9. 按一下 [確定]  來完成 KPI。  
  
    在量值方格中，請注意 [InternetCurrentQuarterSalesPerformance] 量值旁邊的圖示。 這個圖示表示此量值做為 KPI 的基準值。  
  
#### <a name="to-create-an-internetcurrentquartermarginperformance-kpi"></a>若要建立 InternetCurrentQuarterMarginPerformance KPI  
  
1.  在 **FactInternetSales** 資料表的量值方格中，按一下空的儲存格。  
  
2.  在資料表上方的公式列中，輸入下列公式︰  

    ```
    InternetCurrentQuarterMarginPerformance :=IF([InternetPreviousQuarterMarginProportionToQTD]<>0,([InternetCurrentQuarterMargin]-[InternetPreviousQuarterMarginProportionToQTD])/[InternetPreviousQuarterMarginProportionToQTD],BLANK())  
    ```
 
3.  以滑鼠右鍵按一下 [InternetCurrentQuarterMarginPerformance] > [建立 KPI]。  
  
4.  在 [關鍵效能指標 (KPI)] 對話方塊中，於 [目標] 中選取 [絕對值]，然後輸入 **1.25**。   
  
5.  在左側 (低) 滑桿欄位中，滑動直到欄位顯示 **0.8** 為止，然後滑動右側 (高) 滑桿欄位，直到欄位顯示 **1.03** 為止。  
  
6.  在 [選取圖示樣式] 中，選取菱形 (紅色)、三角形 (黃色)、圓形 (綠色) 圖示類型，然後按一下 [確定]。  
  
## <a name="whats-next"></a>後續步驟
[第 8 課：建立檢視方塊](../tutorials/aas-lesson-8-create-perspectives.md)。
  
  
