---
title: SQL Database 多租用戶應用程式指引 - Wingtip SaaS | Microsoft Docs
description: 提供使用 Azure SQL Database、Wingtip Tickets SaaS 範例之範例多租用戶應用程式的安裝和執行步驟及指引。
keywords: SQL Database Azure
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: genemi
ms.openlocfilehash: 6c352298c701c827cd01c0ed7f427b7ed6015e29
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646672"
---
# <a name="general-guidance-for-working-with-wingtip-tickets-sample-saas-apps"></a>使用 Wingtip Tickets 範例 SaaS 應用程式的一般指導方針

本文包含執行使用 Azure SQL Database 的 Wingtip Tickets 範例 SaaS 應用程式之一般指引。 

## <a name="download-and-unblock-the-wingtip-tickets-saas-scripts"></a>下載並解除封鎖 Wingtip Tickets SaaS 指令碼

從外部來源下載 zip 檔案並進行解壓縮時，Windows 可能會封鎖可執行的內容 (指令碼、dll)。 從 zip 檔案解壓縮指令碼時，**請遵循下列步驟先解除封鎖 .zip 檔案，再進行解壓縮**。 這可確保允許執行指令碼。

1. 瀏覽至您要探索資料庫租用戶模式的 Wingtip Tickets SaaS GitHub 存放庫： 
    - [WingtipTicketsSaaS-StandaloneApp](https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp)
    - [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant)
    - [WingtipTicketsSaaS-MultiTenantDb](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb)
2. 按一下 [複製或下載]。
3. 按一下 [下載 zip]，並儲存檔案。
4. 請以滑鼠右鍵按一下 zip 檔案，然後選取 [屬性]。 ZIP 檔案會對應至存放庫名稱。 (例如 _WingtipTicketsSaaS-DbPerTenant-master.zip_)
5. 在 [一般] 索引標籤上，選取 [解除封鎖]。
6. 按一下 [確定]。
7. 將檔案解壓縮。

指令碼檔案位於 ..\\學習模組資料夾中。


## <a name="working-with-the-wingtip-tickets-powershell-scripts"></a>使用 Wingtip Tickets PowerShell 指令碼

若要充分利用此範例，您必須深入了解所提供的指令碼。 使用中斷點和逐步執行指令碼，因為它們會執行及檢查不同 SaaS 模式的實作方式。 若要輕鬆地逐步執行所提供的指令碼和模組，以獲得充分瞭解，我們建議使用 [PowerShell ISE](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/introducing-the-windows-powershell-ise)。

### <a name="update-the-configuration-file-for-your-deployment"></a>更新部署的組態檔

使用您在部署期間設定的資源群組和使用者值，編輯 **UserConfig.psm1** 檔案：

1. 開啟 PowerShell ISE 並載入...\\Learning Modules\\*UserConfig.psm1* 
2. 將 ResourceGroupName 和 Name 更新為您部署的特定值 (只在第 10 行和第 11 行)。
3. 儲存變更！

在這裏設定這些值，只是要讓您必須更新每個指令碼中的這些部署特有值。

### <a name="execute-the-scripts-by-pressing-f5"></a>按 F5 執行指令碼

數個指令碼會使用 $PSScriptRoot 來瀏覽資料夾，而只有在按 F5 執行指令碼時才會評估 $PSScriptRoot。  醒目提示並執行選取項目 (**F8**) 可能會導致錯誤，因此請在執行指令碼時按 **F5**。

### <a name="step-through-the-scripts-to-examine-the-implementation"></a>逐步執行指令碼來檢查實作

瞭解指令碼的最佳方法是逐步執行它們以了解其作用。 查看包含的 **Demo-** 指令碼，其可呈現輕鬆遵循的高階工作流程。 **Demo-** 指令碼會顯示完成每個工作所需的步驟，所以設定中斷點並且更深入探詢個別的呼叫，以查看不同 SaaS 模式的實作詳細資料。

探索及逐步執行 PowerShell 指令碼的秘訣：

- 在 PowerShell ISE 中開啟 **Demo-** 指令碼。
- 執行或繼續使用 **F5** (不建議使用 **F8**，因為在執行指令碼的選取項目時不會評估 $PSScriptRoot)。
- 按一下或選取一行，然後按 **F9** 放置中斷點。
- 使用 **F10** 不進入函式或指令碼呼叫。
- 使用 **F11** 逐步執行函式或指令碼呼叫。
- 使用 **Shift + F11** 跳離目前的函式或指令碼呼叫。


## <a name="explore-database-schema-and-execute-sql-queries-using-ssms"></a>使用 SSMS 瀏覽資料庫結構描述並執行 SQL 查詢

使用 [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 來連線及瀏覽應用程式伺服器和資料庫。

部署一開始都有租用戶與目錄 SQL Database 伺服器加以連線。 伺服器的命名取決於資料庫租用戶模式 (如需詳細資訊，請參閱下方)。 

   - **獨立應用程式：** 每個租用戶的伺服器 (例如 contosoconcerthall-&lt;User&gt; 伺服器) 和 catalog-sa-&lt;User&gt;
   - **每一租用戶一個資料庫：** tenants1-dpt-&lt;User&gt; 和 catalog-dpt-&lt;User&gt; 伺服器
   - **多租用戶資料庫：** tenants1-mt-&lt;User&gt; 和 catalog-mt-&lt;User&gt; 伺服器

若要確保示範連線成功，所有伺服器都有允許所有 IP 通過的[防火牆規則](sql-database-firewall-configure.md)。


1. 開啟 *SSMS* 並連線到租用戶。 伺服器名稱取決於您所選取 (如需詳細資訊，請參閱下方) 的租用戶資料庫模式：
    - **獨立應用程式：** 個別租用戶的伺服器 (例如 contosoconcerthall-&lt;User&gt;.database.windows.net) 
    - **每一租用戶一個資料庫：** tenants1-dpt-&lt;User&gt;.database.windows.net
    - **多租用戶資料庫：** tenants1-mt-&lt;User&gt;.database.windows.net 
2. 按一下 [連線] > **[資料庫引擎...]**：

   ![目錄伺服器](media/saas-tenancy-wingtip-app-guidance-tips/connect.png)

3. 示範認證︰ 登入= developer、密碼 = P@ssword1

    下圖會示範*每一租用戶一個資料庫*模式的登入。 
    ![連接](media/saas-tenancy-wingtip-app-guidance-tips/tenants1-connect.png)
    
   

4. 重複步驟 2-3，並連線到目錄伺服器 (請參閱下方以選取之資料庫租用戶模式作為基礎的特定伺服器名稱)
    - **獨立應用程式：** catalog-sa-&lt;User&gt;.database.windows.net
    - **每一租用戶一個資料庫：** catalog-dpt-&lt;User&gt;.database.windows.net
    - **多租用戶資料庫：** catalog-mt-&lt;User&gt;.database.windows.net


成功連線之後，您會看到所有伺服器。 視您已佈建的租用戶而定，您的資料庫清單可能有所不同。

下圖會示範*每一租用戶一個資料庫*模式的登入。

![物件總管](media/saas-tenancy-wingtip-app-guidance-tips/object-explorer.png)



## <a name="next-steps"></a>後續步驟
- [部署 Wingtip Tickets SaaS Standalone 應用程式](saas-standaloneapp-get-started-deploy.md)
- [部署 Wingtip Tickets SaaS Database Per Tenant 應用程式](saas-dbpertenant-get-started-deploy.md)
- [部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式](saas-multitenantdb-get-started-deploy.md)

