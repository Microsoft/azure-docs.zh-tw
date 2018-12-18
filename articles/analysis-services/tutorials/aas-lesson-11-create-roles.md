---
title: Azure Analysis Services 教學課程第 11 課：建立角色 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程專案中建立角色。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 38c69b65d33a915d0b7cf43dc8ef5d43413163eb
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441238"
---
# <a name="create-roles"></a>建立角色

在這堂課中，您會建立角色。 角色可以限制只有身為角色成員的使用者才能存取，提供模型資料庫物件和資料的安全性。 每個角色都定義有單一權限︰「無」、「讀取」、「讀取和處理」、「處理」或「系統管理員」。 在製作模型期間可以使用 [角色管理員] 來定義角色。 在部署模型之後，您可以使用 SQL Server Management Studio (SSMS) 來管理角色。 若要深入了解，請參閱[角色](https://docs.microsoft.com/sql/analysis-services/tabular-models/roles-ssas-tabular)。
  
> [!NOTE]  
> 完成本教學課程並不需要建立角色。 根據預設，您目前登入的帳戶具有模型的「系統管理員」權限。 不過，若要讓組織中的其他使用者利用報告用戶端瀏覽模型，您必須建立至少一個具有「讀取」權限的角色，並將這些使用者新增為成員。  
  
您可建立三個角色︰  
  
-   **銷售經理** – 此角色可包含組織中對所有模型物件和資料應該具有「讀取」權限的使用者。  
  
-   **美國銷售分析師** – 此角色可包含組織中只能夠瀏覽美國地區相關銷售資料的使用者。 對於此角色，您可使用 DAX 公式來定義「資料列篩選條件」，以限制成員只能瀏覽美國方面的資料。  
  
-   **系統管理員** – 此角色可包含應該擁有「系統管理員」權限的使用者，以授與無限制的存取和權限，而能夠在模型資料庫上執行管理工作。  
  
因為組織中的 Windows 使用者和群組帳戶都是獨一無二，您可以將特定組織中的帳戶新增至成員。 不過，在本教學課程中，您也可以將成員留白。 稍後在「第 12 課︰在 Excel 中進行分析」中，您會測試每個角色的效果。  
  
這堂課的預估完成時間：**15 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
本主題是表格式模型教學課程的一部分，請依序完成。 在這堂課中執行工作之前，您必須已完成上一堂課︰[第 10 課︰建立分割區](../tutorials/aas-lesson-10-create-partitions.md)。  
  
## <a name="create-roles"></a>建立角色  
  
#### <a name="to-create-a-sales-manager-user-role"></a>建立「銷售經理」使用者角色  
  
1.  在 [表格式模型總管] 中，以滑鼠右鍵按一下 [角色] > [角色]。  
  
2.  在 [角色管理員] 中，按一下 [新增]。  
  
3.  按一下新角色，然後在 [名稱] 資料行中，將角色重新命名為**銷售經理**。  
  
4.  在 [權限] 資料行中，按一下下拉式清單，然後選取 [讀取] 權限。 

    ![aas 第 11 課新的角色](../tutorials/media/aas-lesson11-new-role.png) 
  
5.  選擇性：按一下 [成員] 索引標籤，然後按一下 [新增]。 在 [選取使用者或群組] 對話方塊中，輸入您想從組織中加入此角色的 Windows 使用者或群組。  
  
#### <a name="to-create-a-sales-analyst-us-user-role"></a>建立「美國銷售分析師」使用者角色  
  
1.  在 [角色管理員] 中，按一下 [新增]。    
  
2.  將角色重新命名為**美國銷售分析師**。  
  
3.  將 [讀取] 權限授與此角色。  
  
4.  按一下 [資料列篩選條件] 索引標籤，然後只針對 **DimGeography** 資料表，在 [DAX 篩選] 資料行中輸入下列公式︰  
  
    ```Administrator
    =DimGeography[CountryRegionCode] = "US" 
    ```
    
    「資料列篩選條件」公式必須解析成布林值 (TRUE/FALSE)。 在這個公式中，您指定只有 [國家區域碼] 值為 "US" 的資料列才能被使用者看到。  
    ![aas 第 11 課角色篩選](../tutorials/media/aas-lesson11-role-filter.png) 
  
6.  選擇性：按一下 [成員] 索引標籤，然後按一下 [新增]。 在 [選取使用者或群組] 對話方塊中，輸入您想從組織中加入此角色的 Windows 使用者或群組。  
  
#### <a name="to-create-an-administrator-user-role"></a>建立「系統管理員」使用者角色  
  
1.  按一下 [新增] 。  
  
2.  將角色重新命名為「系統管理員」。  
  
3.  將 [系統管理員] 權限授與此角色。  
  
4.  選擇性：按一下 [成員] 索引標籤，然後按一下 [新增]。 在 [選取使用者或群組] 對話方塊中，輸入您想從組織中加入此角色的 Windows 使用者或群組。 
  
  
## <a name="whats-next"></a>後續步驟
[第 12 課：在 Excel 中進行分析](../tutorials/aas-lesson-12-analyze-in-excel.md)

  
  
