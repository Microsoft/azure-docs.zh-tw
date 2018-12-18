---
title: Azure Analysis Services 教學課程第 12 課：在 Excel 中進行分析 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中使用「在 Excel 中進行分析」。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 1e487f6778c45e554f95489e62ac2dedd01ee3f0
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442947"
---
# <a name="analyze-in-excel"></a>使用 Excel 分析

在這堂課中，您會使用「在 Excel 中進行分析」功能來開啟 Microsoft Excel、自動建立模型工作區的連線，以及自動將樞紐分析表新增至工作表。 「在 Excel 中進行分析」功能是為了在部署模型之前，以快速又輕鬆的方式來測試模型設計的功效。 在這堂課中，您不會執行任何資料分析。 這堂課的目的是讓您 (模型作者) 熟悉可用來測試模型設計的工具。   
  
若要完成本這堂課，就必須將 Excel 安裝在 Visual Studio 所在的同一台電腦。
  
這堂課的預估完成時間：**5 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 11 課︰建立角色](../tutorials/aas-lesson-11-create-roles.md)。  
  
## <a name="browse-using-the-default-and-internet-sales-perspectives"></a>使用 [預設] 和 [網際網路銷售] 檢視方塊來瀏覽  
在這些首要工作中，您將會使用 [預設] 檢視方塊 (包含所有模型物件) 來瀏覽模型，也會使用您稍早建立的 [網際網路銷售] 檢視方塊來瀏覽模型。 [網際網路銷售] 檢視方塊會排除 Customer 資料表物件。  
  
#### <a name="to-browse-by-using-the-default-perspective"></a>使用 [預設] 檢視方塊來瀏覽  
  
1.  按一下 [模型] 功能表 > [在 Excel 中的進行分析]。  
  
2.  於 [在 Excel 中的進行分析] 對話方塊中，按一下 [確定]。  
  
    Excel 會開啟新的活頁簿。 將會使用目前的使用者帳戶建立資料來源連線，並使用 [預設] 檢視方塊來定義可檢視的欄位。 樞紐分析表會自動新增至工作表。  
  
3.  在 Excel 的 [樞紐分析表欄位清單] 中，注意 **DimDate** 和 **FactInternetSales** 量值群組會出現。 **DimCustomer**、**DimDate**、**DimGeography**、**DimProduct**、**DimProductCategory**、**DimProductSubcategory** 和 **FactInternetSales** 資料表與其各自資料行也會出現。  
  
4.  關閉 Excel 而不儲存活頁簿。  
  
#### <a name="to-browse-by-using-the-internet-sales-perspective"></a>使用 [網際網路銷售] 檢視方塊來瀏覽  
  
1.  按一下 [模型] 功能表，然後按一下 [使用 Excel 分析]。  
  
2.  於 [在 Excel 中進行分析] 對話方塊中，保持選取 [目前的 Windows 使用者]，接著在 [檢視方塊] 下拉式清單方塊中，選取 [網際網路銷售]，然後按一下 [確定]。 
    
    ![aas 第 12 課檢視方塊](../tutorials/media/aas-lesson12-perspective.png)
    
3.  在 Excel 的 [樞紐分析表欄位] 中，請注意 DimCustomer 資料表已從欄位清單中排除。  
    
    ![aas 第 12 課欄位](../tutorials/media/aas-lesson12-fields.png)
    
4.  關閉 Excel 而不儲存活頁簿。  
  
## <a name="browse-by-using-roles"></a>使用角色來瀏覽  
角色是任何表格式模型的重要部分。 沒有連一個可將使用者新增為成員的角色也沒有，使用者無法使用您的模型來存取和分析資料。 「在 Excel 中進行分析」功能可讓您測試您已定義的角色。  
  
#### <a name="to-browse-by-using-the-sales-manager-user-role"></a>使用「銷售經理」使用者角色來瀏覽  
  
1.  在 SSDT 中，按一下 [模型] 功能表，然後按一下 [使用 Excel 分析]。  
  
2.  在 [指定要用來連線至模型的使用者名稱或角色] 中，選取 [角色]，接著在下拉式清單方塊中選取 [銷售經理]，然後按一下 [確定]。  
  
    Excel 會開啟新的活頁簿。 將會自動建立樞紐分析表。 [樞紐分析表欄位清單] 包含您的新模型中可用的所有資料欄位。  
      
3.  關閉 Excel 而不儲存活頁簿。  
  
## <a name="whats-next"></a>後續步驟
移至下一個課程︰[第 13 課︰部署](../tutorials/aas-lesson-13-deploy.md)。

  
  
  
