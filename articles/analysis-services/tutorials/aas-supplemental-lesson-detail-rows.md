---
title: Azure Analysis Services 教學課程補充課程：詳細資料列 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程中建立詳細資料列運算式。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 626258488afec4b3c3f025ae85bd3b5866aa0cf3
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443972"
---
# <a name="supplemental-lesson---detail-rows"></a>補充課程 - 詳細資料列

在此補充課程中，您會使用 DAX 編輯器來定義自訂詳細資料列運算式。 詳細資料列運算式是一個量值屬性，可提供使用者有關量值彙總結果的詳細資訊。 
  
這堂課的預估完成時間：**10 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
此補充課程是表格式模型教學課程的一部分。 在此補充課程中執行工作之前，您應已完成所有先前的課程或已完成 Adventure Works 網際網路銷售範例模型專案。  
  
## <a name="whats-the-issue"></a>有什麼問題？
讓我們先看看 InternetTotalSales 量值的詳細資料，再新增詳細資料列運算式。

1.  在 SSDT 中，按一下 [模型] 功能表 > [使用 Excel 分析] 以開啟 Excel 並建立空白的樞紐分析表。
  
2.  在 [樞紐分析表欄位] 中，將 FactInternetSales 資料表中的 [InternetTotalSales] 量值新增至 [值]，將 DimDate 資料表中的 [CalendarYear] 新增至 [資料行]，以即將 [EnglishCountryRegionName] 新增至 [資料列]。 樞紐分析表現在會依照區域和年度提供 InternetTotalSales 量值的彙總結果。 

    ![aas-lesson-detail-rows-pivottable](../tutorials/media/aas-lesson-detail-rows-pivottable.png)

3. 在樞紐分析表中，按兩下某個年度和區域名稱的彙總值。 澳洲 2014 年的值。 包含資料 (但非實用資料) 的新工作表隨即開啟。

    ![aas-lesson-detail-rows-pivottable](../tutorials/media/aas-lesson-detail-rows-sheet.png)
  
此處的目標是資料表，其中包含構成 InternetTotalSales 量值彙總結果的資料行和資料列。 若要這麼做，新增詳細資料列運算式作為量值的屬性。

## <a name="add-a-detail-rows-expression"></a>新增詳細資料列運算式

#### <a name="to-create-a-detail-rows-expression"></a>若要新增詳細資料列運算式 
  
1. 在 FactInternetSales 資料表的量值方格中，按一下 [InternetTotalSales] 量值。 

2. 在 [屬性] > [詳細資料列運算式] 中，按一下編輯器按鈕以開啟 DAX 編輯器。

    ![aas-lesson-detail-rows-ellipse](../tutorials/media/aas-lesson-detail-rows-ellipse.png)

3. 在 DAX 編輯器中，輸入下列運算式︰

    ```
    SELECTCOLUMNS(
    FactInternetSales,
    "Sales Order Number", FactInternetSales[SalesOrderNumber],
    "Customer First Name", RELATED(DimCustomer[FirstName]),
    "Customer Last Name", RELATED(DimCustomer[LastName]),
    "City", RELATED(DimGeography[City]),
    "Order Date", FactInternetSales[OrderDate],
    "Internet Total Sales", [InternetTotalSales]
    )

    ```

    此運算式可指定 FactInternetSales 資料表中的名稱、資料行和量值結果，而當使用者按兩下樞紐分析表或報告中的彙總結果時，就會傳回相關資料表。

4. 回到 Excel，刪除在步驟 3 中建立的工作表，然後按兩下彙總值。 此時，使用針對量值定義的詳細資料列運算式屬性，包含許多實用資料的新工作表隨即開啟。

    ![aas-lesson-detail-rows-detailsheet](../tutorials/media/aas-lesson-detail-rows-detailsheet.png)

5. 重新部署您的模型。

  
## <a name="see-also"></a>另請參閱  

[SELECTCOLUMNS 函式 (DAX)](https://msdn.microsoft.com/library/mt761759.aspx)   
[補充課程 - 動態安全性](../tutorials/aas-supplemental-lesson-dynamic-security.md)   
[補充課程 - 不完全階層](../tutorials/aas-supplemental-lesson-ragged-hierarchies.md)   
 