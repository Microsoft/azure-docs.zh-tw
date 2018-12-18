---
title: Azure Analysis Services 教學課程第 10 課：建立分割區 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立分割區。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 7a445cd0e5c737371fa56f1e4dffa49d1915b15a
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37447936"
---
# <a name="create-partitions"></a>建立分割區

在這堂課中，您會建立分割區，將 FactInternetSales 資料表分割成較小的邏輯部分，以獨立於其他分割區來處理 (重新整理)。 根據預設，您在模型中包含的每個資料表會有一個分割區，其中包括資料表的所有資料行和資料列。 對於 FactInternetSales 資料表，我們準備依年份來分割資料，將資料表的五個年份各建立一個分割區。 然後就可以獨立處理每個分割區。 若要深入了解，請參閱[分割區](https://docs.microsoft.com/sql/analysis-services/tabular-models/partitions-ssas-tabular)。 
  
這堂課的預估完成時間：**15 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 9 課：建立階層](../tutorials/aas-lesson-9-create-hierarchies.md)。  
  
## <a name="create-partitions"></a>建立分割區  
  
#### <a name="to-create-partitions-in-the-factinternetsales-table"></a>在 FactInternetSales 資料表中建立分割區  
  
1.  在 [表格式模型總管] 中，展開 [資料表]，然後以滑鼠右鍵按一下 [FactInternetSales] > [分割區]。  
  
2.  在 [分割區管理員] 中，按一下 [複製]，然後將名稱變更為 **FactInternetSales2010**。
  
    因為您只希望分割區包含 2010 年某段期間的資料列，所以必須修改查詢運算式。
  
4.  按一下 [設計] 以開啟查詢編輯器，然後按一下 [FactInternetSales2010] 查詢。

5.  在預覽中，按一下 **OrderDate**資料行標題的向下箭號，然後按一下 [日期/時間篩選] > [介於]。

    ![aas 第 10 課查詢編輯器](../tutorials/media/aas-lesson10-query-editor.png)

6.  在 [篩選資料列] 對話方塊中，保留 [顯示符合條件的列: OrderDate] 中的 [之後或等於]，然後在日期欄位中輸入 **2010/1/1**。 保持選取 [且] 運算子，並選取 [之前]，在日期欄位中輸入 **2011/1/1**，然後按一下 [確定]。

    ![aas-lesson10-filter-rows](../tutorials/media/aas-lesson10-filter-rows.png)
    
    請注意，在查詢編輯器的 APPLIED STEPS 中，您會看到另一個名為 Filtered Rows 步驟。 此篩選條件只會選取 2010 年中的訂單日期。

8.  按一下 [匯入] 。

    在 [分割區管理員] 中，您可以看到查詢運算式現在有額外的 Filtered Rows 子句。

    ![aas 第 10 課查詢](../tutorials/media/aas-lesson10-query.png)
  
    此陳述式指定此分割區所包含的資料，僅限於 Filtered Rows 子句中指定 OrderDate 必須是 2010 日曆年度的那些資料列。  
  
  
#### <a name="to-create-a-partition-for-the-2011-year"></a>建立 2011 年的分割區  
  
1.  在分割區清單中，按一下您建立的 [FactInternetSales2010]分割區，然後按一下 [複製]。  將分割區名稱變更為 **FactInternetSales2011**。 

    您不需要使用查詢編輯器來建立新的 Filtered Rows 子句。 因為是從 2010 年的查詢建立複本，2011 年的查詢只需要稍微變更即可。
  
2.  在 [查詢運算式] 中，為了讓此分割區只包含 2011 年的資料列，請將 Filtered Rows 子句中的年份各改為 **2011** 和 **2012**，如下所示︰  
  
    ```  
    let
        Source = #"SQL/localhost;AdventureWorksDW2014",
        dbo_FactInternetSales = Source{[Schema="dbo",Item="FactInternetSales"]}[Data],
        #"Removed Columns" = Table.RemoveColumns(dbo_FactInternetSales,{"OrderDateKey", "DueDateKey", "ShipDateKey"}),
        #"Filtered Rows" = Table.SelectRows(#"Removed Columns", each [OrderDate] >= #datetime(2011, 1, 1, 0, 0, 0) and [OrderDate] < #datetime(2012, 1, 1, 0, 0, 0))
    in
        #"Filtered Rows"
   
    ```  
  
#### <a name="to-create-partitions-for-2012-2013-and-2014"></a>建立 2012、2013 和 2014 年的分割區。  
  
- 請遵循先前的步驟，建立 2012、2013 和 2014 年的分割區，並變更 Filtered Rows 中的年份，以便只包含該年份的資料列。 
  

## <a name="delete-the-factinternetsales-partition"></a>刪除 FactInternetSales 分割區
現在每一年都有分割區，您可以刪除 FactInternetSales 分割區。處理分割區時，如果選擇 [處理全部]，請避免重疊。

#### <a name="to-delete-the-factinternetsales-partition"></a>刪除 FactInternetSales 分割區
-  按一下 [FactInternetSales] 分割區，然後按一下 [刪除]。



## <a name="process-partitions"></a>處理分割區  
在 [分割區管理員] 中，請注意，您建立的每個分割區的 [上次處理] 資料行會顯示尚未處理這些分割區。 建立分割區時，請執行 [處理分割區] 或 [處理資料表] 作業，以重新整理這些分割區中的資料。  
  
#### <a name="to-process-the-factinternetsales-partitions"></a>處理 FactInternetSales 分割區  
  
1.  按一下 [確定] 來關閉 [分割區管理員]。  
  
2.  按一下 [FactInternetSales] 資料表，然後按一下 [模型] 功能表 > [處理] > [處理分割區]。  
  
3.  在 [處理分割區] 對話方塊中，確認 [模式] 設為 [處理預設]。  
  
4.  對於您建立的五個分割區，將 [處理]資料行的核取方塊全部選取，然後按一下 [確定]。  

    ![aas 第 10 課處理分割區](../tutorials/media/aas-lesson10-process-partitions.png)
  
    如果系統提示您輸入模擬認證，請輸入您在第 2 課指定的 Windows 使用者名稱和密碼。  
  
    [資料處理] 對話方塊隨即出現，顯示每個分割區的處理詳細資料。 請注意，傳送給每個分割區的資料列數目都不同。 每個分割區只會包含 SQL 陳述式的 WHERE 子句已指定之年份的資料列。 處理完成時，請繼續並關閉 [資料處理] 對話方塊。  
  
    ![aas 第 10 課處理完成](../tutorials/media/aas-lesson10-process-complete.png)
  
 ## <a name="whats-next"></a>後續步驟
移至下一個課程︰[第 11 課︰建立角色](../tutorials/aas-lesson-11-create-roles.md)。 
