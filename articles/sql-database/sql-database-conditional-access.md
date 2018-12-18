---
title: 條件式存取 - Azure SQL Database 和 Data Warehouse | Microsoft Doc
description: 了解如何設定 Azure SQL Database 和資料倉儲的條件式存取。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sql-data-warehouse
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: e7386e2f926132f5657bdb17011dafe1e1c7b101
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063076"
---
# <a name="conditional-access-mfa-with-azure-sql-database-and-data-warehouse"></a>使用 Azure SQL Database 和資料倉儲的條件式存取 (MFA)  

Azure [SQL Database](sql-database-technical-overview.md) 與 [SQL 資料倉儲](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md)都支援 Microsoft 條件式取。 

> [!NOTE]
> 本主題適用於 Azure SQL 伺服器，以及在 Azure SQL Server 上建立的 SQL Database 和 SQL 資料倉儲資料庫。 為了簡單起見，參考 SQL Database 和 SQL 資料倉儲時都會使用 SQL Database。

下列步驟示範如何將 SQL Database 設定為強制執行條件式存取原則。  

## <a name="prerequisites"></a>必要條件  
- 您必須將 SQL Database 或 SQL 資料倉儲設定為支援 Azure Active Directory 驗證。 如需特定步驟，請參閱[使用 SQL Database 或 SQL 資料倉儲設定和管理 Azure Active Directory 驗證](sql-database-aad-authentication-configure.md)。  
- 啟用多重要素驗證時，您必須在支援的工具進行連線，例如最新的 SSMS。 如需詳細資訊，請參閱[設定適用於 SQL Server Management Studio 的 Azure SQL Database 多重要素驗證](sql-database-ssms-mfa-authentication-configure.md)。  

## <a name="configure-ca-for-azure-sql-dbdw"></a>針對 Azure SQL DB/DW 設定 CA  
1.  登入入口網站、選取 **Azure Active Directory**，然後選取**條件式存取**。 如需詳細資訊，請參閱 [Azure Active Directory 條件式存取的技術參考](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-technical-reference)。  
  ![[條件式存取] 刀鋒視窗](./media/sql-database-conditional-access/conditional-access-blade.png) 
     
2.  在 [條件式存取原則] 刀鋒視窗中，按一下 [新增原則]、提供名稱，然後按一下 [設定規則]。  
3.  在 [指派] 下，選取 [使用者和群組]，核取 [選取使用者和群組]，然後選取要進行條件式存取的使用者或群組。 按一下 [選取]，然後按一下 [完成] 可接受您的選擇。  
  ![選取 [使用者和群組]](./media/sql-database-conditional-access/select-users-and-groups.png)  

4.  選取 [雲端應用程式]，按一下 [選取應用程式]。 您會看到所有可供條件式存取使用的應用程式。 選取底部的 [Azure SQL Database]按一下 [選取]然後按一下 [完成]。  
  ![選取 SQL Database](./media/sql-database-conditional-access/select-sql-database.png)  
  如果找不到下列第三個螢幕擷取畫面中所列的 **Azure SQL Database**，請完成下列步驟：   
  - 使用 SSMS 搭配 AAD 管理帳戶來登入您的 Azure SQL DB/DW 執行個體。  
  - 執行 `CREATE USER [user@yourtenant.com] FROM EXTERNAL PROVIDER`。  
  - 登入 AAD，並確認 Azure SQL Database 和資料倉儲已列在 AAD 的應用程式中。  

5.  依序選取 [存取控制]、[授與]，然後核取您想要套用的原則。 例如，我們選取 [需要多重要素驗證]。  
  ![選取授與存取權](./media/sql-database-conditional-access/grant-access.png)  

## <a name="summary"></a>總結  
允許使用 Azure AD Premium 連線到 Azure SQL DB/DW 的選取應用程式 (Azure SQL Database)，現在會強制執行選取的條件式存取原則，**必要的多重要素驗證。**  
若有關於多重要素驗證的 Azure SQL Database 和資料倉儲相關問題，請連絡 MFAforSQLDB@microsoft.com。  

## <a name="next-steps"></a>後續步驟  

如需教學課程，請參閱[保護 Azure SQL Database](sql-database-security-tutorial.md)。
