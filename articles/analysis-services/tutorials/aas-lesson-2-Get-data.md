---
title: Azure Analysis Services 教學課程第 2 課：取得資料 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中取得和匯入資料。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 5803bf2c71b2cf3fe7bb145b4d3d664c60642294
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42145334"
---
# <a name="get-data"></a>取得資料

在這堂課中，您將使用 SSDT 中的「取得資料」來連線至 Adventure Works 範例資料庫、選取資料、預覽及篩選，然後匯入您的模型工作區中。  
  
「取得資料」可讓您從各種來源匯入資料︰Azure SQL Database、Oracle、Sybase、OData 摘要、Teradata、檔案等等。 您也可以使用 Power Query M 公式運算式來查詢資料。

> [!NOTE]
> 本教學課程中的工作和影像顯示連接至內部部署伺服器上的 AdventureWorksDW2014 資料庫。 在某些情況下，Azure 上的 Adventure Works 資料庫可能會不同。
  
這堂課的預估完成時間：**10 分鐘**  
  
## <a name="prerequisites"></a>必要條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 1 課︰建立新的表格式模型專案](../tutorials/aas-lesson-1-create-a-new-tabular-model-project.md)。  
  
## <a name="create-a-connection"></a>建立連線  
  
#### <a name="to-create-a-connection-to-the-adventureworksdw2014-database"></a>建立 AdventureWorksDW2014 資料庫的連線  
  
1.  在 [表格式模型總管] 中，以滑鼠右鍵按一下 [資料來源] > [從資料來源匯入]。  
  
    這會啟動「取得資料」，引導您連線至資料來源。 如果您沒有看到 [表格式模型總管] 中，請在 [方案總管] 中按兩下 [Model.bim]，在設計工具中開啟模型。 
    
    ![aas 第 2 課取得資料](../tutorials/media/aas-lesson2-getdata.png)
  
2.  在 [取得資料] 中，按一下 [資料庫] > [SQL Server 資料庫] > [連線]。  
  
3.  在 [SQL Server 資料庫] 對話方塊的 [伺服器] 中，輸入您安裝 AdventureWorksDW2014 資料庫的伺服器名稱，然後按一下 [連線]。  

4.  當系統提示您輸入認證時，您必須指定 Analysis Services 在匯入和處理資料時用來連線至資料來源的認證。 在 [模擬模式] 中，選取 [模擬帳戶]，輸入認證，然後按一下 [連線]。 建議您使用密碼不會過期的帳戶。

    ![aas 第 2 課帳戶](../tutorials/media/aas-lesson2-account.png)
  
    > [!NOTE]  
    > 使用 Windows 使用者帳戶和密碼是連線至資料來源最安全的方法。
  
5.  在 [導覽器] 中，選取 [AdventureWorksDW2014] 資料庫，然後按一下 [確定]。這會建立資料庫的連線。 
  
6.  在 [導覽器] 中，選取下列資料表的核取方塊︰**DimCustomer**、**DimDate**、**DimGeography**、**DimProduct**、**DimProductCategory**、**DimProductSubcategory** 和 **FactInternetSales**，然後按一下 [編輯]。

    ![aas-lesson2-select-tables](../tutorials/media/aas-lesson2-select-tables.png)
  
    查詢編輯器隨即開啟。 在下一個區段中，您只要選取想要匯入的資料。

  
## <a name="filter-the-table-data"></a>篩選資料表資料  
AdventureWorksDW2014 範例資料庫中的資料表有不需要加入模型中的資料。 可能的話，建議您篩選掉不必要的資料，以節省模型所使用的記憶體內部空間。 您會從資料表中篩選掉某些資料行，這樣就不會匯入工作區資料庫中或模型資料庫中 (在部署之後)。 
  
#### <a name="to-filter-the-table-data-before-importing"></a>匯入之前篩選資料表資料  
  
1.  在 [查詢編輯器] 中，選取 [DimCustomer] 資料表。 將會出現資料來源 (AdventureWorksDW2014 範例資料庫) 上的 DimCustomer 資料表檢視。 
  
2.  複選 (Ctrl + 按一下) **SpanishEducation**、**FrenchEducation**、**SpanishOccupation**、**FrenchOccupation**，按一下滑鼠右鍵，然後按一下 [移除資料行]。 

    ![aas 第 2 課移除資料行](../tutorials/media/aas-lesson2-remove-columns.png)
  
    因為這些資料行的值與網際網路銷售分析無關，不需要匯入這些資料行。 去除不必要的資料行可讓您的模型變得更小且更有效率。  

    > [!TIP]
    > 如果發生錯誤，您可以利用在**套用的步驟**中刪除步驟來恢復。   
    
    ![aas 第 2 課移除資料行](../tutorials/media/aas-lesson2-remove-step.png)

  
4.  在每個資料表中移除下列資料行，以篩選其餘資料表︰  
    
    **DimDate**
    
      |欄|  
      |--------|  
      |**DateKey**|  
      |**SpanishDayNameOfWeek**|  
      |**FrenchDayNameOfWeek**|  
      |**SpanishMonthName**|  
      |**FrenchMonthName**|  
  
    **DimGeography**
  
      |欄|  
      |-------------|  
      |**SpanishCountryRegionName**|  
      |**FrenchCountryRegionName**|  
      |**IpAddressLocator**|  
  
    **DimProduct**
  
      |欄|  
      |-----------|  
      |**SpanishProductName**|  
      |**FrenchProductName**|  
      |**FrenchDescription**|  
      |**ChineseDescription**|  
      |**ArabicDescription**|  
      |**HebrewDescription**|  
      |**ThaiDescription**|  
      |**GermanDescription**|  
      |**JapaneseDescription**|  
      |**TurkishDescription**|  
  
    **DimProductCategory**
  
      |欄|  
      |--------------------|  
      |**SpanishProductCategoryName**|  
      |**FrenchProductCategoryName**|  
  
    **DimProductSubcategory**
  
      |欄|  
      |-----------------------|  
      |**SpanishProductSubcategoryName**|  
      |**FrenchProductSubcategoryName**|  
  
    **FactInternetSales**
  
      未移除任何資料行。
  
## <a name="Import"></a>匯入選取的資料表和資料表資料  
既然您已預覽並篩選掉不必要的資料，接下來可以匯入您真正想要的其餘資料。 精靈會匯入資料表資料，以及資料表之間的任何關聯性。 模型中會建立新的資料表和資料行，不會匯入您已篩選掉的資料。  
  
#### <a name="to-import-the-selected-tables-and-column-data"></a>匯入選取的資料表和資料行資料  
  
1.  檢閱您的選擇。 如果一切看起來沒問題，請按一下 [匯入]。 [資料處理] 對話方塊會顯示資料從資料來源匯入工作區資料庫的狀態。
  
    ![aas 第 2 課成功](../tutorials/media/aas-lesson2-success.png) 
  
2.  按一下 [關閉] 。  

  
## <a name="save-your-model-project"></a>儲存模型專案  
請務必經常儲存模型專案。  
  
#### <a name="to-save-the-model-project"></a>儲存模型專案  
  
-   按一下 [檔案] > [全部儲存]。  
  
## <a name="whats-next"></a>後續步驟
[第 3 課：標記為日期資料表](../tutorials/aas-lesson-3-mark-as-date-table.md)。

  
  
