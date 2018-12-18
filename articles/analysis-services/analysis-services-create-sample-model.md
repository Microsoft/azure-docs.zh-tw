---
title: 教學課程 - 使用入口網站將基本範例模型新增至 Azure Analysis Services 伺服器 | Microsoft Docs
description: 在本教學課程中，了解如何在 Azure Analysis Services 中新增範例模型。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: c63995a461cee6bc39603a43604b8080942bd88b
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442802"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>教學課程：從入口網站新增範例模型

在本教學課程中，您會將範例 Adventure Works 表格式模型資料庫新增至您的伺服器。 此範例模型是 Adventure Works Internet Sales (1200) 範例資料模型的完整版本。 範例模型有助於測試模型管理、與工具和用戶端應用程式連線，以及查詢模型資料。 本教學課程是使用 [Azure 入口網站](https://portal.azure.com)和 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)： 

> [!div class="checklist"]
> * 將完整的範例表格式資料模型新增至伺服器 
> * 使用 SSMS 連線到模型

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="before-you-begin"></a>開始之前

若要完成本教學課程，您需要：

- Azure Analysis Services 伺服器。 若要深入了解，請參閱[建立伺服器 - 入口網站](analysis-services-create-server.md)。
- 伺服器管理員權限
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入[入口網站](https://portal.azure.com/)。

## <a name="add-a-sample-model"></a>新增範例模型

1. 在伺服器的 [概觀] 中，按一下 [新增模型]。

    ![建立範例模型](./media/analysis-services-create-sample-model/aas-create-sample-new-model.png)

2. 在 [新增模型] > [選擇資料來源] 中，確認已選取 [範例資料]，然後按一下 [新增]。

    ![選取範例資料](./media/analysis-services-create-sample-model/aas-create-sample-data.png)

3. 在 [概觀] 中，確認已新增 `adventureworks` 範例模型。

    ![選取範例資料](./media/analysis-services-create-sample-model/aas-create-sample-verify.png)


## <a name="clean-up-resources"></a>清除資源

範例模型會使用快取記憶體資源。 如果您不將範例模型用於測試，則應將其從伺服器中移除。

下列步驟說明如何使用 SSMS 從伺服器中刪除模型。 您也可以使用預覽的 Web 設計工具功能來刪除模型。

1. 在 SSMS > [物件總管] 中，按一下 [連線]  > [分析服務] 。

2. 在 [連線到伺服器] 中貼上伺服器名稱，然後在 [驗證] 中選擇 [具 MFA 支援的 Active Directory - 通用]，並輸入您的使用者名稱，然後按一下 [連線]。

    ![登入](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png)

3. 在 [物件總管] 中，以滑鼠右鍵按一下 `adventureworks`，然後按一下 [刪除]。

    ![刪除範例資料庫](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png)

## <a name="next-steps"></a>後續步驟 

在本教學課程中，您已撩解如何將基本範例模型新增至您的伺服器。 您現在有一個模型資料庫，您可以從 SQL Server Management Studio 連線到該資料庫，並新增使用者角色。 若要深入了解，請繼續下一個教學課程。

> [!div class="nextstepaction"]
> [教學課程：設定伺服器管理員和使用者角色](analysis-services-database-users.md)


