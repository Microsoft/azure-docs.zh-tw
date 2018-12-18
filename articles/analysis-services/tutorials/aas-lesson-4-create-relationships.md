---
title: Azure Analysis Services 教學課程第 4 課：建立關聯性 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立關聯性。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 5007e18db0af40621ab4b30a16d705d3a5b3915c
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443812"
---
# <a name="create-relationships"></a>建立關聯性

在這堂課中，您可以驗證當您匯入資料時自動建立的關聯性，並新增新不同資料表之間的關聯性。 關聯性是兩個資料表之間的關係，規定這些資料表中的資料應該如何相互關聯。 例如，DimProduct 資料表和 DimProductSubcategory 資料表有關聯性，理由是每個產品都屬於某個子類別。 若要深入了解，請參閱[關聯性](https://docs.microsoft.com/sql/analysis-services/tabular-models/relationships-ssas-tabular)。
  
這堂課的預估完成時間：**10 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 3 課：標記為日期資料表](../tutorials/aas-lesson-3-mark-as-date-table.md)。 
  
## <a name="review-existing-relationships-and-add-new-relationships"></a>檢閱現有的關聯性和新增關聯性  
當您使用「取得資料」匯入資料時，您從 AdventureWorksDW2014 資料庫取得七個資料表。 一般而言，當您從關聯式來源匯入資料時，自動會隨著資料一起匯入現有的關聯性。 為了讓「取得資料」自動在資料模型中建立關聯性，資料來源的資料表之間必須有關聯性。

在繼續製作模型之前，您應該確認資料表之間的關聯性已正確建立。 本教學課程中，您也會新增三個新的關聯性。  

  
#### <a name="to-review-existing-relationships"></a>檢閱現有的關聯性  
  
1.  按一下 [模型] 功能表 > [模型檢視] > [圖表檢視]。  

    模型設計師現在會出現在 [圖表檢視] 中，以圖形格式顯示您匯入的所有資料表，而且資料表之間都有線條。 資料表之間的線條代表您匯入資料時自動建立的關聯性。
    
    ![aas-lesson4-diagram](../tutorials/media/aas-lesson4-diagram.png)
  
    > [!NOTE]
    > 如果您在資料表之間沒有看到任何關聯性，也可能表示在資料來源中的這些資料表之間沒有任何關聯性。

    使用模型設計工具右下角的迷你地圖控制項，盡可能包含最多資料表。 您也可以按一下資料表並拖曳至不同的位置，讓資料表彼此更靠近，或依特定順序排列。 移動資料表不會影響資料表之間存在的關聯性。 若要檢視特定資料表中的所有資料行，請按一下資料表邊緣並拖曳，以放大或縮小。  
  
2.  按一下 **DimCustomer** 資料表和 **DimGeography** 資料表之間的實線。 這兩個資料表之間的實線表示此關聯性為作用中，也就是計算 DAX 公式時依預設會用到它。  
  
    請注意，**DimCustomer** 資料表中的 **GeographyKey** 資料行和 **DimGeography** 資料表中的 **GeographyKey** 資料行現在都出現在方塊內。 這些資料行會使用於關聯性中。 關聯性的屬性現在也會出現在 [屬性] 視窗中。  
  
    > [!TIP]  
    > 除了使用模型設計師的圖表檢視，您也可以使用 [管理關聯性] 對話方塊，以表格格式顯示所有資料表之間的關聯性。 在 [表格式模型總管] 中，以滑鼠右鍵按一下 [關聯性] > [管理關聯性]。
  
3.  確認從 AdventureWorksDW 資料庫匯入每個資料表時已建立下列關聯性︰  
  
    |Active|資料表|相關的查閱資料表|  
    |----------|---------|------------------------|  
    |yes|**DimCustomer [GeographyKey]**|**DimGeography [GeographyKey]**|  
    |yes|**DimProduct [ProductSubcategoryKey]**|**DimProductSubcategory [ProductSubcategoryKey]**|  
    |yes|**DimProductSubcategory [ProductCategoryKey]**|**DimProductCategory [ProductCategoryKey]**|  
    |yes|**FactInternetSales [CustomerKey]**|**DimCustomer [CustomerKey]**|  
    |yes|**FactInternetSales [ProductKey]**|**DimProduct [ProductKey]**|  
  
    如果遺漏任何關聯性，請確認您的模型包含下列資料表：DimCustomer、DimDate、DimGeography、DimProduct、DimProductCategory、DimProductSubcategory 和 FactInternetSales。 如果是在不同時間從相同的資料來源連線匯入資料表，則不會建立這些資料表之間的關聯性，必須手動建立。 如果沒有出現任何關聯性，表示資料來源沒有任何關聯性。 您可以手動在資料模型中加以建立。

### <a name="take-a-closer-look"></a>進一步了解
在 [圖表] 檢視中，請注意在顯示資料表之間關聯性的線條上有箭號、星號和數字。

![aas-lesson4-line](../tutorials/media/aas-lesson4-line.png)

箭號顯示篩選方向。 星號顯示此資料表是關聯性基數的多數邊，1 顯示此資料表是關聯性的單數邊。 如果您需要編輯關聯性，例如，變更關聯性的篩選方向或基數，請按兩下關聯性線條以開啟 [編輯關聯性] 對話方塊。

![aas-lesson4-edit](../tutorials/media/aas-lesson4-edit.png)

這些功能主要用於進階資料模型化，已超出本教學課程的範圍。 若要深入了解，請參閱 [Analysis Services 中表格式模型的雙向交叉篩選條件](https://docs.microsoft.com/sql/analysis-services/tabular-models/bi-directional-cross-filters-tabular-models-analysis-services)。

在某些情況下，您可能需要在模型中的資料表之間建立額外的關聯性，以支援特定的商務邏輯。 本教學課程中，您需要在 FactInternetSales 資料表和 DimDate 資料表之間再多建立三個關聯性。  
  
#### <a name="to-add-new-relationships-between-tables"></a>在資料表之間新增關聯性  
  
1.  在模型設計師中，在 [FactInternetSales] 資料表中按住 [OrderDate] 資料行，然後將游標拖曳至 [DimDate] 資料表中的 [Date] 資料行，然後放開。  

    隨即出現一條實線，表示您已在 **InternetSales** 資料表的 **OrderDate** 資料行和 **Date** 資料表的 **Date** 資料行之間建立作用中的關聯性。 
  
      ![aas-lesson4-new](../tutorials/media/aas-lesson4-new.png) 
  
    > [!NOTE]  
    > 在建立關聯性時，自動會選取主要資料表和相關查閱資料表之間的基數和篩選方向。  
  
2.  在 [FactInternetSales] 資料表中，按住 [DueDate] 資料行，然後將游標拖曳至 [DimDate] 資料表中的 [Date] 資料行，然後放開。  
  
    隨即出現一條虛線，表示您已在 **FactInternetSales** 資料表的 **DueDate** 資料行和 **DimDate** 資料表的 **Date** 資料行之間建立非作用中的關聯性。 您在資料表之間可以有多個關聯性，但一次只能有一個關聯性是作用中。 非作用中的關聯性可以變成作用中，以便在自訂 DAX 運算式中執行特殊彙總。  
  
3.  最後，再建立一個關聯性。 在 [FactInternetSales] 資料表中，按住 [ShipDate] 資料行，然後將游標拖曳至 [DimDate] 資料表中的 [Date] 資料行，然後放開。  
    
     ![aas-lesson4-newinactive](../tutorials/media/aas-lesson4-newinactive.png)
  
## <a name="whats-next"></a>後續步驟
[第 5 課：建立計算結果欄](../tutorials/aas-lesson-5-create-calculated-columns.md)。
  
  
  
