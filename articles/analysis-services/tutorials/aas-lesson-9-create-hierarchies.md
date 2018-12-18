---
title: Azure Analysis Services 教學課程第 9 課：建立階層 | Microsoft Docs
description: 描述如何在表格式模型中建立階層。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 928fe227a74c5c63ccdfb364b0e2423d7b544864
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443013"
---
# <a name="create-hierarchies"></a>建立階層

在這堂課中，您會建立階層。 階層是以層級排列的資料行群組。 例如，地理階層可能有國家/地區、州、縣和市的子層級。 階層可以與報告用戶端應用程式欄位清單中的其他資料行分開顯示，讓用戶端使用者更易於在報告中導覽及包含階層。 若要深入了解，請參閱[階層](https://docs.microsoft.com/sql/analysis-services/tabular-models/hierarchies-ssas-tabular)
  
若要建立階層，請使用 [圖表檢視] 中的模型設計工具。 不支援在 [資料檢視] 中建立及管理階層。  
  
這堂課的預估完成時間：**20 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 8 課︰建立檢視方塊](../tutorials/aas-lesson-8-create-perspectives.md)。  
  
## <a name="create-hierarchies"></a>建立階層  
  
#### <a name="to-create-a-category-hierarchy-in-the-dimproduct-table"></a>若要在 DimProduct 資料表中建立 Category 階層  
  
1.  在模型設計工具 (圖表檢視) 中，以滑鼠右鍵按一下 [DimProduct] 資料表 > [建立階層]。 新階層會出現在資料表視窗的底部。 將階層重新命名為 **Category**。  
  
2.  按一下 **ProductCategoryName** 資料行並拖曳至新的 **Category** 階層。  
  
3.  在 **Category** 階層中，以滑鼠右鍵按一下 [ProductCategoryName] >  [重新命名]，然後輸入 **Category**。  
  
    > [!NOTE]  
    > 重新命名階層中的資料行，並不會將資料表中的該資料行重新命名。 階層中的資料行只是資料表中的資料行表示法。  
  
4.  按一下 **ProductSubcategoryName** 資料行並拖曳至 **Category** 階層。 將它重新命名為 **Subcategory**。 
  
5.  以滑鼠右鍵按一下 [ModelName] 資料行 > [新增至階層]，然後選取 **Category**。 將它重新命名為 **Model**。

6.  最後，將 **EnglishProductName** 新增至 Category 階層。 將它重新命名 **Product**。  

    ![aas-lesson9-category](../tutorials/media/aas-lesson9-category.png)
  
#### <a name="to-create-hierarchies-in-the-dimdate-table"></a>若要在 DimDate 資料表中建立階層  
  
1.  在 **DimDate** 資料表中，建立名為 **Calendar** 的階層。  
  
3.  依序新增下列資料行︰

    *  CalendarYear
    *  CalendarSemester
    *  CalendarQuarter
    *  MonthCalendar
    *  DayNumberOfMonth
    
4.  在 **DimDate** 資料表中，建立 **Fiscal** 階層。 依序包含下列資料行︰  
  
    *  FiscalYear
    *  FiscalSemester
    *  FiscalQuarter
    *  MonthCalendar
    *  DayNumberOfMonth
  
5.  最後，在 **DimDate** 資料表中，建立 **ProductionCalendar** 階層。 依序包含下列資料行︰  
    *  CalendarYear
    *  WeekNumberOfYear
    *  DayNumberOfWeek
  
 ## <a name="whats-next"></a>後續步驟
[第 10 課：建立資料分割](../tutorials/aas-lesson-10-create-partitions.md)。 
  
  
