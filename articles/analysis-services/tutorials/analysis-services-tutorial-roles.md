---
title: 教學課程 - 設定 Azure Analysis Services 伺服器管理員和使用者角色教學課程 | Microsoft Docs
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 07/09/2018
ms.author: owend
ms.reviewer: owend
ms.openlocfilehash: 1c1dd5316eead5e91dd77d3e6b21a7a14d39afeb
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37929224"
---
# <a name="tutorial-configure-server-administrator-and-user-roles"></a>教學課程：設定伺服器管理員和使用者角色

 在本教學課程中，您會使用 SQL Server Management Studio (SSMS) 連線至 Azure 中的伺服器，以設定伺服器管理員和模型資料庫角色。 此外也會介紹[表格式模型指令碼語言 (TMSL)](https://docs.microsoft.com/sql/analysis-services/tabular-model-programming-compatibility-level-1200/tabular-model-programming-for-compatibility-level-1200)。 TMSL 是 1200 或更高相容性層級的表格式模型所適用的 JSON 型指令碼語言。 它可用來自動化許多表格式模型化工作。 TMSL 常會與 PowerShell 搭配使用，但在本教學課程中，您會使用 SSMS 中的 XMLA 查詢編輯器。 透過本教學課程，您會完成下列工作： 
  
> [!div class="checklist"]
> * 從入口網站取得您的伺服器名稱
> * 使用 SSMS 連線至您的伺服器
> * 將使用者或群組新增至伺服器管理員角色 
> * 將使用者或群組新增至模型資料庫管理員角色
> * 新增新的模型資料庫角色，並新增使用者或群組

若要深入了解 Azure Analysis Services 中的使用者安全性，請參閱[驗證和使用者權限](../analysis-services-manage-users.md)。 

## <a name="prerequisites"></a>先決條件

- 訂用帳戶中的 Azure Active Directory。
- 已在您的訂用帳戶中建立 [Azure Analysis Services 伺服器](../analysis-services-create-server.md)。
- 具有[伺服器管理員](../analysis-services-server-admins.md)權限。
- [新增 adventureworks 範例模型](../analysis-services-create-sample-model.md)至您的伺服器。
- [安裝最新版本的 SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入[入口網站](https://portal.azure.com/)。

## <a name="get-server-name"></a>取得伺服器名稱
若要從 SSMS 連線至您的伺服器，首先您必須要有伺服器名稱。 您可以從入口網站取得伺服器名稱。

在 [Azure 入口網站] > 伺服器 > [概觀]  >  [伺服器名稱] 中，複製伺服器名稱。
   
   ![在 Azure 中取得伺服器名稱](./media/analysis-services-tutorial-roles/aas-copy-server-name.png)

## <a name="connect-in-ssms"></a>在 SSMS 中連線

對於其餘的工作，您可以使用 SSMS 來連線和管理伺服器。

1. 在 SSMS > [物件總管] 中，按一下 [連線]  > [分析服務] 。

    ![連線](./media/analysis-services-tutorial-roles/aas-ssms-connect.png)

2. 在 [連線到伺服器] 對話方塊的 [伺服器名稱] 中，貼上您從入口網站複製的伺服器名稱。 在 [驗證] 中選擇 [具 MFA 支援的 Active Directory 通用]，並輸入您的使用者帳戶，然後按 [連線]。
   
    ![在 SSMS 中連線](./media/analysis-services-tutorial-roles/aas-connect-ssms-auth.png)

    > [!TIP]
    > 建議選擇 [具 MFA 支援的 Active Directory 通用]。 此類型的驗證支援[非互動式或多因素驗證](../../sql-database/sql-database-ssms-mfa-authentication.md)。 

3. 在 [物件總管] 中，展開伺服器物件加以檢視。 按一下滑鼠右鍵以檢視伺服器屬性。
   
    ![在 SSMS 中連線](./media/analysis-services-tutorial-roles/aas-connect-ssms-objexp.png)

## <a name="add-a-user-account-to-the-server-administrator-role"></a>將使用者帳戶新增至伺服器管理員角色

在此工作中，您會將使用者或群組帳戶從您的 Azure AD 新增至伺服器管理員角色。 如果您要新增安全性群組，則必須先將 `MailEnabled` 屬性設定為 `True`。

1. 在 [物件總管] 中，以滑鼠右鍵按一下您的伺服器名稱，然後按一下 [屬性]。 
2. 在 [Analysis Server 屬性] 視窗中，按一下 [安全性] > [新增]。
3. 在 [選取使用者或群組] 視窗中，輸入您 Azure AD 中的使用者或群組帳戶，然後按一下 [新增]。 
   
     ![新增伺服器管理員](./media/analysis-services-tutorial-roles/aas-add-server-admin.png)

4. 按一下 [確定] 以關閉 [Analysis Server 屬性]。

    > [!TIP]
    > 您也可以在入口網站中使用 **Analysis Services 管理員**來新增伺服器管理員。 

## <a name="add-a-user-to-the-model-database-administrator-role"></a>將使用者新增至模型資料庫管理員角色

在此工作中，您會將使用者或群組帳戶新增至已存在於模型中的網際網路銷售管理員角色。 此角色具有 adventureworks 範例模型資料庫的完全控制 (管理員) 權限。 此工作會在為您建立的指令碼中使用 [CreateOrReplace](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/createorreplace-command-tmsl) TMSL 命令。

1. 在 [物件總管] 中，展開 [資料庫] > [adventureworks] > [角色]。 
2. 以滑鼠右鍵按一下 [網際網路銷售管理員]，然後按一下 [編寫角色的指令碼為] > [建立或取代為] > [新增查詢編輯器視窗]。

    ![新增查詢編輯器視窗](./media/analysis-services-tutorial-roles/aas-add-db-admin.png)

3. 在 **XMLAQuery** 中，將 **"memberName":** 的值變更為您 Azure AD 中的使用者或群組帳戶。 依預設會包含您登入時所使用的帳戶；但您無須新增自己的帳戶，因為您已經是伺服器管理員。

    ![XMLA 查詢中的 TMSL 指令碼](./media/analysis-services-tutorial-roles/aas-add-db-admin-script.png)

4. 按 **F5** 以執行指令碼。


## <a name="add-a-new-model-database-role-and-add-a-user-or-group"></a>新增新的模型資料庫角色，並新增使用者或群組

在此工作中，您會在 TMSL 指令碼中使用 [Create](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/create-command-tmsl?view=sql-analysis-services-2017) 命令建立新的網際網路銷售全域角色、指定該角色的*讀取*權限，並從您的 Azure AD 新增使用者或群組帳戶。

1. 在 [物件總管] 中，以滑鼠右鍵按一下 **adventureworks**，然後按一下 [新增查詢] > [XMLA]。 
2. 將下列 TMSL 指令碼複製並貼到查詢編輯器中：

    ```JSON
    {
    "create": {
      "parentObject": {
        "database": "adventureworks",
       },
       "role": {
         "name": "Internet Sales Global",
         "description": "All users can query model data",
         "modelPermission": "read",
         "members": [
           {
             "memberName": "globalsales@adventureworks.com",
             "identityProvider": "AzureAD"
           }
         ]
       }
      }
    }
    ```

3. 將 `"memberName": "globalsales@adventureworks.com"` 物件值變更為您 Azure AD 中的使用者或群組帳戶。
4. 按 **F5** 以執行指令碼。

## <a name="verify-your-changes"></a>驗證您的變更

1. 在 [物件總管] 中按一下您的伺服器名稱，然後按一下 [重新整理] 或按 **F5**。
2. 展開 [資料庫] > [adventureworks] > [角色]。 請確認此時會出現您在先前的工作中新增的使用者帳戶和新角色變更。   

    ![在物件總管中確認](./media/analysis-services-tutorial-roles/aas-connect-ssms-verify.png)

## <a name="clean-up-resources"></a>清除資源

不再需要使用者或群組帳戶和角色時，請加以刪除。 若要這樣做，請使用 [角色屬性] > [成員資格] 移除使用者帳戶，或以滑鼠右鍵按一下角色，然後按一下 [刪除]。


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何連線至 Azure AS 伺服器，並在 SSMS 中瀏覽 adventureworks 範例模型資料庫和屬性。 您也已了解如何使用 SSMS 和 TMSL 指令碼將使用者或群組新增至現有和新的角色。 現在，您已為伺服器和範例模型資料庫設定使用者權限，接下來您和其他使用者即可使用 Power BI 之類的用戶端應用程式連線至該資料庫。 若要深入了解，請繼續進行下一個教學課程。 

> [!div class="nextstepaction"]
> [教學課程 - 使用 Power BI Desktop 進行連線](analysis-services-tutorial-pbid.md)

