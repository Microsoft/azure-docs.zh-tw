---
title: Azure Analysis Services 教學課程第 6 課：建立量值 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立量值。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 6cf880cf132168d60ff49c2e7ea571cd8e89d36f
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37446651"
---
# <a name="create-measures"></a>建立量值

在這堂課中，您將會建立要加入模型中的量值。 類似於您所建立的計算結果欄，量值是利用 DAX 公式所建立的計算結果。 不過，與計算結果欄不同，量值評估是根據使用者選取的「篩選條件」。 例如，在樞紐分析表的 [資料列標籤] 欄位中新增的特定資料行或交叉分析篩選器。 然後會以套用的量值，計算篩選條件中每個資料格的值。 量值是功能強大又靈活的計算，在幾乎所有表格式模型中都會派上用場，可以對數值資料執行動態計算。 若要深入了解，請參閱[量值](https://docs.microsoft.com/sql/analysis-services/tabular-models/measures-ssas-tabular)。
  
若要建立量值，您需要使用「量值方格」。 根據預設，每個資料表都有一個空白量值方格；不過，您通常不會為每個資料表建立量值。 在資料檢視中，量值方格中會出現在模型設計師中的資料表下方。 若要隱藏或顯示資料表的量值方格，請按一下 [資料表] 功能表，然後按一下 [顯示量值方格]。  
  
若要建立量值，您可以按一下量值方格中的空資料格，然後在公式列中輸入 DAX 公式。 當您按 ENTER 完成公式時，量值就會出現在資料格中。 您也可以按一下資料行，然後按一下工具列上的 [自動加總] 按鈕 (**∑**)，利用標準彙總函式建立量值。 使用「自動加總」功能所建立的量值會出現在資料行正下方的量值方格資料格中，但是可以移動。  
  
在這堂課中，您會在公式列中輸入 DAX 公式和使用「自動加總」功能來建立量值。  
  
這堂課的預估完成時間：**30 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 5 課︰建立計算結果欄](../tutorials/aas-lesson-5-create-calculated-columns.md)。  
  
## <a name="create-measures"></a>建立量值  
  
#### <a name="to-create-a-dayscurrentquartertodate-measure-in-the-dimdate-table"></a>在 DimDate 資料表中建立 DaysCurrentQuarterToDate 量值  
  
1.  在模型設計師中，按一下 [DimDate] 資料表。  
  
2.  在量值方格中，按一下左上方的空資料格。  
  
3.  在公式列中，輸入下列公式︰  
  
    ```
    DaysCurrentQuarterToDate:=COUNTROWS( DATESQTD( 'DimDate'[Date])) 
    ```
  
    請注意，左上方的資料格現在包含量值名稱 **DaysCurrentQuarterToDate**，後面接著結果 **92**。 此時的結果不相關，因為未套用使用者篩選。
    
      ![aas 第 6 課新增量值](../tutorials/media/aas-lesson6-newmeasure.png) 
    
    不同於計算結果欄，量值公式可讓您輸入量值名稱，後面接著一個冒號，最後再接著公式運算式。

  
#### <a name="to-create-a-daysincurrentquarter-measure-in-the-dimdate-table"></a>在 DimDate 資料表中建立 DaysInCurrentQuarter 量值  
  
1.  在模型設計師中繼續使用 **DimDate** 資料表，然後在量值方格中，按一下您建立之量值下方的空資料格。  
  
2.  在公式列中，輸入下列公式︰  
  
    ```
    DaysInCurrentQuarter:=COUNTROWS( DATESBETWEEN( 'DimDate'[Date], STARTOFQUARTER( LASTDATE('DimDate'[Date])), ENDOFQUARTER('DimDate'[Date])))
    ```
  
    在一個不完整期間與前一個期間之間建立比率時， 此公式必須計算該期間已流逝的比例，並與前一個期間的相同比例做比較。 在此案例中，[DaysCurrentQuarterToDate]/[DaysInCurrentQuarter] 計算出目前期間已流逝的比例。  
  
#### <a name="to-create-an-internetdistinctcountsalesorder-measure-in-the-factinternetsales-table"></a>在 FactInternetSales 資料表中建立 InternetDistinctCountSalesOrder 量值  
  
1.  按一下 [FactInternetSales] 資料表。   
  
2.  按一下 [SalesOrderNumber] 資料行標題。  
  
3.  在工具列上，按一下 [自動加總] \(**∑**) 按鈕旁的向下箭號，然後選取 [DistinctCount]。  
  
    「自動加總」功能會使用 DistinctCount 標準彙總公式，自動為選取的資料行建立量值。  
    
       ![aas 第 6 課新增量值 2](../tutorials/media/aas-lesson6-newmeasure2.png)
  
4.  在量值方格中，按一下新量值，然後在 [屬性] 視窗的 [量值名稱] 中，將量值重新命名為 **InternetDistinctCountSalesOrder**。 
 
  
#### <a name="to-create-additional-measures-in-the-factinternetsales-table"></a>在 FactInternetSales 資料表中建立其他量值  
  
1.  使用「自動加總」功能建立並命名下列量值︰  

    |欄|量值名稱|自動加總 (∑)|公式|  
    |----------------|----------|-----------------|-----------|  
    |SalesOrderLineNumber|InternetOrderLinesCount|Count|=COUNTA([SalesOrderLineNumber])|  
    |OrderQuantity|InternetTotalUnits|總和|=SUM([OrderQuantity])|  
    |DiscountAmount|InternetTotalDiscountAmount|總和|=SUM([DiscountAmount])|  
    |TotalProductCost|InternetTotalProductCost|總和|=SUM([TotalProductCost])|  
    |SalesAmount|InternetTotalSales|總和|=SUM([SalesAmount])|  
    |Margin|InternetTotalMargin|總和|=SUM([Margin])|  
    |TaxAmt|InternetTotalTaxAmt|總和|=SUM([TaxAmt])|  
    |Freight|InternetTotalFreight|總和|=SUM([Freight])|  
  
2.  按一下量值方格中的空資料格並使用公式列，依序建立下列自訂量值：  
  
      ```
      InternetPreviousQuarterMargin:=CALCULATE([InternetTotalMargin],PREVIOUSQUARTER('DimDate'[Date]))
      ```
      
      ```
      InternetCurrentQuarterMargin:=TOTALQTD([InternetTotalMargin],'DimDate'[Date])
      ```
  
      ```
      InternetPreviousQuarterMarginProportionToQTD:=[InternetPreviousQuarterMargin]*([DaysCurrentQuarterToDate]/[DaysInCurrentQuarter])
      ```
  
      ```
      InternetPreviousQuarterSales:=CALCULATE([InternetTotalSales],PREVIOUSQUARTER('DimDate'[Date]))
      ```
  
      ```
      InternetCurrentQuarterSales:=TOTALQTD([InternetTotalSales],'DimDate'[Date])
      ```
      
      ```
      InternetPreviousQuarterSalesProportionToQTD:=[InternetPreviousQuarterSales]*([DaysCurrentQuarterToDate]/[DaysInCurrentQuarter])
      ```
  
針對 FactInternetSales 資料表所建立的量值可用來分析重要財務資料，例如，使用者選取的篩選條件所定義之商品的銷售額、成本和獲利率。  
  
## <a name="whats-next"></a>後續步驟
[第 7 課：建立關鍵效能指標](../tutorials/aas-lesson-7-create-key-performance-indicators.md)。  

  
