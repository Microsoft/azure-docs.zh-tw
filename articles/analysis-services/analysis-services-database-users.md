---
title: 管理 Azure Analysis Services 中的使用者角色和使用者 | Microsoft Docs
description: 了解如何在 Azure 中管理 Analysis Services 伺服器上的資料庫角色和使用者。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 8c777d5376614f7afe59342dc5a9fbfa37ca4556
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37441051"
---
# <a name="manage-database-roles-and-users"></a>管理資料庫角色和使用者

在模型資料庫層級上，所有使用者都必須屬於某個角色。 角色可針對模型資料庫定義具有特定權限的使用者。 任何新增至某個角色的使用者或安全性群組都必須在與伺服器相同的訂用帳戶中具有 Azure AD 租用戶的帳戶。 

定義角色的方式會根據您使用的工具而有所不同，但效果一樣。

角色權限包括：
*  **系統管理員** - 使用者具有資料庫的完整權限。 具有系統管理員權限的資料庫角色與伺服器管理員不同。
*  **處理** - 使用者可以連線到資料庫並對其執行處理作業，以及分析模型資料庫的資料。
*  **讀取** - 使用者可以使用用戶端應用程式來連接和分析模型資料庫的資料。

建立表格式模型專案時，您可在 SSDT 中使用角色管理員來建立角色，並將使用者或群組新增至這些角色。 若部署至伺服器，您可使用 SSMS、[Analysis Services PowerShell Cmdlet](https://msdn.microsoft.com/library/hh758425.aspx) 或[表格式模型指令碼語言](https://msdn.microsoft.com/library/mt614797.aspx) (TMSL)，新增或移除角色和使用者成員。

> [!NOTE]
> 安全性群組必須將 `MailEnabled` 屬性設定為 `True`。

## <a name="to-add-or-manage-roles-and-users-in-ssdt"></a>在 SSDT 中新增或管理角色和使用者  
  
1.  在 SSDT > [表格式模型總管] 中，以滑鼠右鍵按一下 [角色]。  
  
2.  在 [角色管理員] 中，按一下 [新增]。  
  
3.  輸入角色的名稱。  
  
     根據預設，每個新角色的預設角色名稱是以累加方式編號。 建議您輸入可清楚識別成員類型的名稱，例如「財務經理」或「人力資源專員」。  
  
4.  選取下列其中一個權限：  
  
    |權限|說明|  
    |----------------|-----------------|  
    |**None**|成員無法修改模型結構描述，也無法查詢資料。|  
    |**讀取**|成員可以查詢資料 (根據資料列篩選條件)，但無法修改模型結構描述。|  
    |**讀取和處理**|成員可以查詢資料 (根據資料列層級的篩選條件)，並執行「處理」和「全部處理」作業，但無法修改模型結構描述。|  
    |**處理程序**|成員可以執行「處理」和「全部處理」作業。 無法修改模型結構描述，也無法查詢資料。|  
    |**系統管理員**|成員可以修改模型結構描述及查詢所有資料。|   
  
5.  如果您建立的角色具有「讀取」或「讀取和處理」權限，您可以使用 DAX 公式來新增資料列篩選條件。 按一下 [資料列篩選條件] 索引標籤，然後選取資料表，再按一下 [DAX 篩選條件] 欄位，然後輸入 DAX 公式。
  
6.  按一下 [成員] >  [新增外部]。  
  
8.  在 [新增外部成員] 中，依照電子郵件地址輸入 Azure AD 租用戶中的使用者或群組。 按一下 [確定] 並關閉 [角色管理員] 後，角色和角色成員就會出現在 [表格式模型總管] 中。 
 
     ![表格式模型總管中的角色和使用者](./media/analysis-services-database-users/aas-roles-tmexplorer.png)

9. 部署到 Azure Analysis Services 伺服器。


## <a name="to-add-or-manage-roles-and-users-in-ssms"></a>在 SSMS 中新增或管理角色和使用者
若要將角色和使用者新增至已部署的模型資料庫，您必須以伺服器管理員身分連線到伺服器，或已經是具有系統管理員權限的資料庫角色。

1. 在 [物件總館] 中，以滑鼠右鍵按一下 [角色] > [新增角色]。

2. 在 [建立角色] 中，輸入角色名稱和描述。

3. 選取權限。
   |權限|說明|  
   |----------------|-----------------|  
   |**完全控制 (系統管理員)**|成員可以修改模型結構描述、程序，以及查詢所有資料。| 
   |**處理資料庫**|成員可以執行「處理」和「全部處理」作業。 無法修改模型結構描述，也無法查詢資料。|  
   |**讀取**|成員可以查詢資料 (根據資料列篩選條件)，但無法修改模型結構描述。|  
  
4. 按一下 [成員資格]，然後依照電子郵件地址輸入 Azure AD 租用戶中的使用者或群組。

     ![新增使用者](./media/analysis-services-database-users/aas-roles-adduser-ssms.png)

5. 如果您建立的角色具有「讀取」權限，您可以使用 DAX 公式來新增資料列篩選條件。 按一下 [資料列篩選條件]，選取資料表，然後在 [DAX 篩選條件] 欄位中輸入 DAX 公式。 

## <a name="to-add-roles-and-users-by-using-a-tmsl-script"></a>使用 TMSL 指令碼新增角色和使用者
您可以在 SSMS 中 或使用 PowerShell，在 XMLA 視窗中執行 TMSL 指令碼。 使用 [CreateOrReplace](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/createorreplace-command-tmsl) 命令和 [Roles](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-objects/roles-object-tmsl) 物件。

**TMSL 指令碼範例**

在此範例中，B2B 外部使用者和群組會新增至 SalesBI 資料庫中具讀取權限的分析師角色。 外部使用者和群組必須在相同的 Azure AD 租用戶中。

```
{
  "createOrReplace": {
    "object": {
      "database": "SalesBI",
      "role": "Analyst"
    },
    "role": {
      "name": "Users",
      "description": "All allowed users to query the model",
      "modelPermission": "read",
      "members": [
        {
          "memberName": "user1@contoso.com",
          "identityProvider": "AzureAD"
        },
        {
          "memberName": "group1@adventureworks.com",
          "identityProvider": "AzureAD"
        }
      ]
    }
  }
}
```

## <a name="to-add-roles-and-users-by-using-powershell"></a>使用 PowerShell 新增角色和使用者
[SqlServer](https://msdn.microsoft.com/library/hh758425.aspx) 模組提供特定工作的資料庫管理 Cmdlet，以及可接受表格式模型指令碼語言 (TMSL) 查詢或指令碼的一般用途 Invoke-ASCmd Cmdlet。 下列 Cmdlet 用來管理資料庫角色和使用者。
  
|Cmdlet|說明|
|------------|-----------------| 
|[Add-RoleMember](https://msdn.microsoft.com/library/hh510167.aspx)|將成員新增到資料庫角色。| 
|[Remove-RoleMember](https://msdn.microsoft.com/library/hh510173.aspx)|從資料庫角色移除成員。|   
|[Invoke-ASCmd](https://msdn.microsoft.com/library/hh479579.aspx)|執行 TMSL 指令碼。|

## <a name="row-filters"></a>資料列篩選條件  
資料列篩選條件定義特定角色的成員可以查詢資料表中的哪些資料列。 使用 DAX 公式，針對模型中的每個資料表定義資料列篩選條件。  
  
只能針對具有「讀取」和「讀取和處理」權限的角色定義資料列篩選條件。 根據預設，如果未針對特定資料表定義資料列篩選條件，則成員可以查詢資料表中的所有資料列，除非從另一個資料表套用交叉篩選。
  
 資料列篩選條件需要 DAX 公式，其必須評估為 TRUE/FALSE 值，以定義該特定角色的成員可以查詢的資料列。 無法查詢 DAX 公式中未包含的資料列。 例如，具有資料列篩選條件運算式 (=Customers [Country] = “USA”) 的「客戶」資料表，「銷售」角色成員只能查看美國的客戶。  
  
資料列篩選條件會套用至指定的資料列和相關資料列。 若資料表具有多個關聯性，篩選條件就會套用作用中關聯性的安全性。 資料列篩選條件會與針對相關資料表定義的其他資料列篩選條件產生交集，例如：  
  
|資料表|DAX 運算式|  
|-----------|--------------------|  
|區域|=Region[Country]=”USA”|  
|ProductCategory|=ProductCategory[Name]=”Bicycles”|  
|交易|=Transactions[Year]=2016|  
  
 淨效應是成員可以查詢客戶位於美國、產品類別為自行車且 2016 年之資料的資料列。 使用者無法查詢美國以外的交易、非自行車的交易或不在 2016 年的交易，除非他們是授與這些權限之另一個角色的成員。
  
 您可以使用 =FALSE() 篩選條件，拒絕存取整個資料表的所有資料列。

## <a name="next-steps"></a>後續步驟
  [管理伺服器管理員](analysis-services-server-admins.md)   
  [使用 PowerShell 管理 Azure Analysis Services](analysis-services-powershell.md)  
  [表格式模型指令碼語言 (TMSL) 參考](https://docs.microsoft.com/sql/analysis-services/tabular-model-scripting-language-tmsl-reference)

