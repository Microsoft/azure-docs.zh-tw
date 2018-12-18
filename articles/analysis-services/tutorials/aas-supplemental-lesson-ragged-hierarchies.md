---
title: Azure Analysis Services 教學課程補充課程：不完全階層 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程中修正不完全階層。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 9fdf332727c4d66af2d5394fb26e84f6ea9d963f
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37440760"
---
# <a name="supplemental-lesson---ragged-hierarchies"></a>補充課程 - 不完全階層

在這堂補充課程中，您可解決對不同層級包含空白值 (成員) 的階層進行樞紐分析時的常見問題。 例如，組織中部門經理和非部門經理同時為高階經理的直屬主管。 或者，由國家/地區-區域-城市組成的地理階層，其中有些城市缺少上層的州或省，例如華盛頓特區、梵蒂岡。 當階層具有空白成員時，它通常會向下延伸至不同或不完全的層級。

![aas-lesson-detail-ragged-hierarchies-table](../tutorials/media/aas-lesson-detail-ragged-hierarchies-table.png)

位於 1400 相容性層級的表格式模型具有階層的其他 [隱藏成員] 屬性。 [預設] 設定假設任何層級都沒有空白成員。 [隱藏空白成員] 設定會在成員新增至樞紐分析表或報告時，從階層中排除空白成員。  
  
這堂課的預估完成時間：**20 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
此補充課程主題是表格式模型教學課程的一部分。 在此補充課程中執行工作之前，您應已完成所有先前的課程或已完成 Adventure Works 網際網路銷售範例模型專案。 

如果您已建立 AW 網際網路銷售專案作為本教學課程的一部分，您的模型尚未包含任何資料或不完全的階層。 若要完成本補充課程，您必須先藉由新增一些額外的資料表來建立問題、建立關聯性、計算結果欄、量值以及新的組織階層。 該部分大約需要 15 分鐘。 然後，您在短短幾分鐘內就可以解決。  

## <a name="add-tables-and-objects"></a>新增資料表和物件
  
### <a name="to-add-new-tables-to-your-model"></a>將新資料表新增至您的模型
  
1.  在 [表格式模型總管] 中，依序展開 [資料來源]，然後以滑鼠右鍵按一下您的連線 > [匯入新資料表]。
  
2.  在 [導覽器] 中，選取 [DimEmployee] 和 [FactResellerSales]，然後按一下 [確定]。

3.  在 [查詢編輯器] 中，按一下 [匯入]

4.  建立下列[關聯性](../tutorials/aas-lesson-4-create-relationships.md)：

    | 表 1           | 欄       | 篩選方向   | 表 2     | 欄      | Active |
    |-------------------|--------------|--------------------|-------------|-------------|--------|
    | FactResellerSales | OrderDateKey | 預設值            | DimDate     | 日期        | yes    |
    | FactResellerSales | DueDate      | 預設值            | DimDate     | 日期        | 否     |
    | FactResellerSales | ShipDateKey  | 預設值            | DimDate     | 日期        | 否     |
    | FactResellerSales | ProductKey   | 預設值            | DimProduct  | ProductKey  | yes    |
    | FactResellerSales | EmployeeKey  | 對兩個資料表 | DimEmployee | EmployeeKey | yes    |

5. 在 [DimEmployee] 資料表中，建立下列[計算結果欄](../tutorials/aas-lesson-5-create-calculated-columns.md)： 

    **路徑** 
    ```
    =PATH([EmployeeKey],[ParentEmployeeKey])
    ```

    **FullName** 
    ```
    =[FirstName] & " " & [MiddleName] & " " & [LastName]
    ```

    **Level1** 
    ```
    =LOOKUPVALUE(DimEmployee[FullName],DimEmployee[EmployeeKey],PATHITEM([Path],1,1)) 
    ```

    **Level2** 
    ```
    =LOOKUPVALUE(DimEmployee[FullName],DimEmployee[EmployeeKey],PATHITEM([Path],2,1)) 
    ```

    **Level3** 
    ```
    =LOOKUPVALUE(DimEmployee[FullName],DimEmployee[EmployeeKey],PATHITEM([Path],3,1)) 
    ```

    **Level4** 
    ```
    =LOOKUPVALUE(DimEmployee[FullName],DimEmployee[EmployeeKey],PATHITEM([Path],4,1)) 
    ```

    **Level5** 
    ```
    =LOOKUPVALUE(DimEmployee[FullName],DimEmployee[EmployeeKey],PATHITEM([Path],5,1)) 
    ```

6.  在 [DimEmployee] 資料表中，建立名為 **Organization** 的[階層](../tutorials/aas-lesson-9-create-hierarchies.md)。 依序新增下列資料行︰**Level1**、**Level2**、**Level3**、**Level4**、**Level5**。

7.  在 [FactResellerSales] 資料表中，建立下列[量值](../tutorials/aas-lesson-6-create-measures.md)：

    ```
    ResellerTotalSales:=SUM([SalesAmount])
    ```

8.  使用[使用 Excel 分析](../tutorials/aas-lesson-12-analyze-in-excel.md)以開啟 Excel 並自動建立樞紐分析表。

9.  在 [樞紐分析表欄位] 中，將 [DimEmployee] 資料表中的 [組織] 階層新增至 [資料列]，以及將 [FactResellerSales] 資料表中的 [ResellerTotalSales] 量值新增至 [值]。

    ![aas-lesson-detail-ragged-hierarchies-pivottable](../tutorials/media/aas-lesson-detail-ragged-hierarchies-pivottable.png)

    如您在樞紐分析表中所見，階層會顯示不完全的資料列。 有許多資料列中顯示空白的成員。

## <a name="to-fix-the-ragged-hierarchy-by-setting-the-hide-members-property"></a>藉由設定 [隱藏成員] 屬性來修正不完全的階層

1.  在 [表格式模型總管] 中，依序展開 [資料表] > [DimEmployee] > [階層] > [組織]。

2.  在 [屬性] > [隱藏成員] 中，選取 [隱藏空白成員]。 

    ![aas-lesson-detail-ragged-hierarchies-hidemembers](../tutorials/media/aas-lesson-detail-ragged-hierarchies-hidemembers.png)

3.  回到 Excel，重新整理樞紐分析表。 

    ![aas-lesson-detail-ragged-hierarchies-pivottable-refresh](../tutorials/media/aas-lesson-detail-ragged-hierarchies-pivottable-refresh.png)

    現在看起來好多了！

## <a name="see-also"></a>另請參閱   
[第 9 課：建立階層](../tutorials/aas-lesson-9-create-hierarchies.md)  
[補充課程 - 動態安全性](../tutorials/aas-supplemental-lesson-dynamic-security.md)  
[補充課程 - 詳細資料列](../tutorials/aas-supplemental-lesson-detail-rows.md)  
