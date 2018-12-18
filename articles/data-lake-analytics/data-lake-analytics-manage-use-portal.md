---
title: 使用 Azure 入口網站管理 Azure Data Lake Analytics
description: 本文說明如何使用 Azure 入口網站來管理 Data Lake Analytics 帳戶、資料來源、使用者和作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: a0e045f1-73d6-427f-868d-7b55c10f811b
ms.topic: conceptual
ms.date: 12/05/2016
ms.openlocfilehash: 1d49403ec04b2ec35291869385c316cb5ab3b0da
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43044903"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-portal"></a>使用 Azure 入口網站來管理 Azure Data Lake Analytics
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用 Azure 入口網站來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。


<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-lake-analytics-accounts"></a>管理 Data Lake Analytics 帳戶

### <a name="create-an-account"></a>建立帳戶

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下 [建立資源] > [智慧 + 分析] > [Data Lake Analytics]。
3. 選取下列項目的值︰ 
   1. **名稱**：Data Lake Analytics 帳戶的名稱。
   2. **訂用帳戶**：用於此帳戶的 Azure 訂用帳戶。
   3. **資源群組**：在其中建立帳戶的 Azure 資源群組。 
   4. **位置**：Data Lake Analytics 帳戶的 Azure 資料中心。 
   5. **Data Lake Store**：Data Lake Analytics 帳戶所要使用的預設存放區。 Azure Data Lake Store 帳戶和 Data Lake Analytics 帳戶必須位於相同位置。
4. 按一下頁面底部的 [新增] 。 

### <a name="delete-a-data-lake-analytics-account"></a>刪除 Data Lake Analytics 帳戶

在您刪除 Data Lake Analytics 帳戶前，請先刪除其預設 Data Lake Store 帳戶。

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [刪除] 。
3. 輸入帳戶名稱。
4. 按一下 [刪除] 。

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-sources"></a>管理資料來源

Data Lake Analytics 支援下列資料來源：

* Data Lake Store
* Azure 儲存體

您可以使用 [資料總管] 來瀏覽資料來源和執行基本檔案管理作業。 

### <a name="add-a-data-source"></a>建立資料來源

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [資料來源]。
3. 按一下 [ **新增資料來源**]。
    
   * 若要新增 Azure Data Lake Store 帳戶，您需要帳戶名稱及帳戶的存取權才可進行查詢。
   * 若要新增 Azure Blob 儲存體，您需要儲存體帳戶和帳戶金鑰。 若要找到它們，請在入口網站中移至此儲存體帳戶。

## <a name="set-up-firewall-rules"></a>設定防火牆規則

您可以使用 Data Lake Analytics，進一步在網路層級鎖定 Data Lake Analytics 帳戶的存取。 您可以啟用防火牆、指定 IP 位址或為受信任的用戶端定義 IP 位址範圍。 啟用這些量值之後，只有具有定義範圍內 IP 位址的用戶端可以連線到存放區。

如果有其他 Azure 服務 (例如 Azure Data Factory 或 VM) 連線到 Data Lake Analytics 帳戶，請確定 [允許 Azure 服務] 已 [開啟]。 

### <a name="set-up-a-firewall-rule"></a>設定防火牆規則

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 在左側功能表上，按一下 [防火牆]。

## <a name="add-a-new-user"></a>新增使用者

您可以使用 [新增使用者精靈] 輕鬆地佈建新的 Data Lake 使用者。

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 在左側 [快速入門] 之下，按一下 [新增使用者精靈]。
3. 選取使用者，然後按一下 [選取]。
4. 選取角色，然後按一下 [選取]。 若要設定新的開發人員以使用 Azure Data Lake，請選取 [Data Lake Analytics 開發人員] 角色。
5. 選取 U-SQL 資料庫的存取控制清單 (ACL)。 當您對您的選擇感到滿意時，請按一下 [選取]。
6. 選取檔案的 ACL。 對於預設存放區，請不要變更根資料夾 "/" 和 /system 資料夾的 ACL。 按一下 [選取] 。
7. 檢閱您選取的所有變更，然後按一下 [執行]。
8. 當精靈完成時，按一下 [完成]。

## <a name="manage-role-based-access-control"></a>管理角色型存取控制

如同其他 Azure 服務，您可以使用角色型存取控制 (RBAC) 來控制使用者與服務互動的方式。

標準 RBAC 角色具有下列功能：
* **擁有者**：可以提交、監視、取消任何使用者的作業，以及設定帳戶。
* **參與者**：可以提交、監視、取消任何使用者的作業，以及設定帳戶。
* **讀取者**：可以監視作業。

使用 Data Lake Analytics 開發人員角色來啟用 U-SQL 開發人員，以使用 Data Lake Analytics 服務。 您可以使用 Data Lake Analytics 開發人員角色來：
* 提交作業。
* 監視任何使用者所提交作業的作業狀態和進度。
* 請參閱任何使用者所提交作業中的 U-SQL 指令碼。
* 只取消您自己的作業。

### <a name="add-users-or-security-groups-to-a-data-lake-analytics-account"></a>將使用者或安全性群組新增到 Data Lake Analytics 帳戶

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [存取控制 (IAM)] > [新增]。
3. 選取角色。
4. 新增使用者。
5. 按一下 [確定]。

>[!NOTE]
>如果使用者或安全性群組需要提交作業，他們也需要有存放區帳戶的權限。 如需詳細資訊，請參閱[保護儲存在 Data Lake Store 中的資料](../data-lake-store/data-lake-store-secure-data.md)。
>

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-jobs"></a>管理工作

### <a name="submit-a-job"></a>提交作業

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。

2. 按一下 [ **新增工作**]。 對於每項作業，請設定：

    1. **作業名稱**：作業名稱。
    2. **優先順序**：數字越小，優先順序越高。 如果有兩項作業排入佇列，優先順序值較小的作業會優先執行。
    3. **平行處理原則**：要為此作業保留的計算程序數目上限。

3. 按一下 [ **提交作業**]。

### <a name="monitor-jobs"></a>監視工作

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [檢視所有作業]。 隨即顯示帳戶中所有作用中和最近完成的作業清單。
3. 選擇性，按一下 [篩選] 可協助您依照 [時間範圍]、[作業名稱] 和 [作者] 值尋找作業。 

### <a name="monitoring-pipeline-jobs"></a>監視管線作業
屬於某個管線的作業會搭配運作 (通常會循序進行)，以完成特定案例。 例如，您可以有一個管線，來清除、擷取、轉換、彙總客戶深入解析的使用。 提交作業之後，可使用 [管線] 屬性來找到管線作業。 使用 ADF V2 排程的作業會自動填入此屬性。 

若要檢視屬於管線的 U-SQL 作業清單： 

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [作業深入解析]。 預設會開啟 [所有作業] 索引標籤，其中顯示執行中、已佇列和已結束的作業清單。
3. 按一下 [管線作業] 索引標籤。這會顯示管線作業清單，以及每個管線的彙總統計資料。

### <a name="monitoring-recurring-jobs"></a>監視週期性作業
週期性作業是具有相同商務邏輯，但每次執行都會使用不同輸入資料的作業。 在理想情況下，週期性作業一律會成功，而且執行時間相當穩定；監視這些行為有助於確保作業狀況良好。 可使用 [週期性] 屬性來找到週期性作業。 使用 ADF V2 排程的作業會自動填入此屬性。

若要檢視週期性的 U-SQL 作業清單： 

1. 在 Azure 入口網站中，移至您的 Data Lake Analytics 帳戶。
2. 按一下 [作業深入解析]。 預設會開啟 [所有作業] 索引標籤，其中顯示執行中、已佇列和已結束的作業清單。
3. 按一下 [週期性作業] 索引標籤。這會顯示週期性作業清單，以及每個週期性作業的彙總統計資料。

## <a name="next-steps"></a>後續步驟

* [Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Azure PowerShell 管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-powershell.md)
* [使用原則來管理 Azure Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-policies)
