---
title: Azure Analysis Services 教學課程第 5 課：建立計算結果欄 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立計算結果欄。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: d97a365d1eae21a58e2b33b82dc2593343248e0e
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445930"
---
# <a name="create-calculated-columns"></a>建立計算結果欄

在這堂課中，您會在模型中新增計算結果欄來建立資料。 您可以在使用「取得資料」(使用查詢編輯器) 時建立計算結果欄 (作為自訂資料行)，或稍後在模型設計師中再如法泡製。 若要深入了解，請參閱[計算結果欄](https://docs.microsoft.com/sql/analysis-services/tabular-models/ssas-calculated-columns)。
  
您會在三個不同的資料表中建立五個新的計算結果欄。 每個工作的步驟稍有不同，以顯示有數種方式可建立資料行、重新命名資料行，並將它們放在資料表中的不同位置。  

您會在這堂課中第一次使用 Data Analysis Expressions (DAX)。 DAX 是一種特殊語言，可以為表格式模型建立可靈活自訂的公式運算式。 在本教學課程中，您會使用 DAX 來建立計算結果欄、量值和角色篩選條件。 若要深入了解，請參閱[表格式模型中的 DAX](https://docs.microsoft.com/sql/analysis-services/tabular-models/understanding-dax-in-tabular-models-ssas-tabular)。 
  
這堂課的預估完成時間：**15 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 4 課︰建立關聯性](../tutorials/aas-lesson-4-create-relationships.md)。 
  
## <a name="create-calculated-columns"></a>建立計算結果欄  
  
#### <a name="create-a-monthcalendar-calculated-column-in-the-dimdate-table"></a>在 DimDate 資料表中建立 MonthCalendar 計算結果欄  
  
1.  按一下 [模型] 功能表 > [模型檢視] > [資料檢視]。  
  
    只有在資料檢視中使用模型設計師，才能建立計算結果欄。  
  
2.  在模型設計師中，按一下 [DimDate] 資料表 (索引標籤)。  
  
3.  以滑鼠右鍵按一下 [CalendarQuarter] 資料行標頭，然後按一下 [插入資料行]。  
  
    [日曆季] 資料行左側會插入名為 [計算結果欄 1] 的新資料行。  
  
4.  在資料表上方的公式列中，輸入下列 DAX 公式：「自動完成」會協助您輸入資料行和資料表的完整名稱，並列出可用的函式。  
  
    ```  
    =RIGHT(" " & FORMAT([MonthNumberOfYear],"#0"), 2) & " - " & [EnglishMonthName]  
    ``` 
  
    接著，計算結果欄的所有資料列就會填入值。 如果您在資料表中向下捲動，根據每個資料列中的資料，您會看到各資料列在此資料行中會有不同的值。    
  
5.  將此資料行重新命名為 **MonthCalendar**。 

    ![aas 第 5 課新增資料行](../tutorials/media/aas-lesson5-newcolumn.png) 
  
MonthCalendar 計算結果欄提供可排序的月份名稱。  
  
#### <a name="create-a-dayofweek-calculated-column-in-the-dimdate-table"></a>在 DimDate 資料表中建立 DayOfWeek 計算結果欄  
  
1.  繼續使用 **DimDate** 資料表，按一下 [資料行] 功能表，然後按一下 [新增資料行]。  
  
2.  在公式列中，輸入下列公式︰  
    
    ```
    =RIGHT(" " & FORMAT([DayNumberOfWeek],"#0"), 2) & " - " & [EnglishDayNameOfWeek]  
    ```
    
    當您完成建立公式時，請按 ENTER 鍵。 新的資料行會新增至資料表的最右邊。  
  
3.  將資料行重新命名為 **DayOfWeek**。  
  
4.  按一下資料行標題，然後將資料行拖曳到 **EnglishDayNameOfWeek** 資料行和 **DayNumberOfMonth** 資料行之間。  
  
    > [!TIP]  
    > 移動資料表中的資料行可讓您更輕鬆地瀏覽。  
  
DayOfWeek 計算結果欄提供可排序的星期幾名稱。  
  
#### <a name="create-a-productsubcategoryname-calculated-column-in-the-dimproduct-table"></a>在 DimProduct 資料表中建立 ProductSubcategoryName 計算結果欄  
  
  
1.  在 **DimProduct** 資料表中，捲動到資料表最右邊。 請注意，最右邊的資料行名為 **[新增資料行]** \(斜體)，請按一下資料行標題。  
  
2.  在公式列中，輸入下列公式︰  
    
    ```
    =RELATED('DimProductSubcategory'[EnglishProductSubcategoryName])  
    ```
  
3.  將資料行重新命名為 **ProductSubcategoryName**。  
  
ProductSubcategoryName 計算結果欄用於在 DimProduct 資料表中建立階層，此階層包含來自 DimProductSubcategory 資料表之 EnglishProductSubcategoryName 資料行的資料。 階層不可跨越多個資料表。 稍後您會在第 9 課中建立階層。  
  
#### <a name="create-a-productcategoryname-calculated-column-in-the-dimproduct-table"></a>在 DimProduct 資料表中建立 ProductCategoryName 計算結果欄  
  
1.  繼續使用 **DimProduct** 資料表，按一下 [資料行] 功能表，然後按一下 [新增資料行]。  
  
2.  在公式列中，輸入下列公式︰  
  
    ```
    =RELATED('DimProductCategory'[EnglishProductCategoryName]) 
    ```
    
3.  將資料行重新命名為 **ProductCategoryName**。  
  
ProductCategoryName 計算結果欄用於在 DimProduct 資料表中建立階層，此階層包含來自 DimProductCategory 資料表之 EnglishProductCategoryName 資料行的資料。 階層不可跨越多個資料表。  
  
#### <a name="create-a-margin-calculated-column-in-the-factinternetsales-table"></a>在 FactInternetSales 資料表中建立 Margin 計算結果欄  
  
1.  在模型設計師中，選取 [FactInternetSales] 資料表。  
  
2.  在 **SalesAmount** 資料行和 **TaxAmt** 資料行之間建立新的計算結果欄。  
  
3.  在公式列中，輸入下列公式︰  
  
    ```
    =[SalesAmount]-[TotalProductCost]
    ``` 

4.  將資料行重新命名為 **Margin**。  
 
      ![aas 第 5 課新增獲利率](../tutorials/media/aas-lesson5-newmargin.png)
      
    Margin 計算結果欄用來分析每一筆銷售的獲利率。  
  
## <a name="whats-next"></a>後續步驟
[第 6 課：建立量值](../tutorials/aas-lesson-6-create-measures.md)。
  
  
  
