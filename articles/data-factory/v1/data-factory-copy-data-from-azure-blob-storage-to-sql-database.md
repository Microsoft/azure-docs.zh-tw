---
title: 將資料從 Blob 儲存體複製到 SQL Database - Azure | Microsoft Docs
description: 本教學課程向您說明如何使用 Azure Data Factory 管線中的複製活動，將資料從 Blob 儲存體複製到 SQL Database。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: e4035060-93bf-4e8d-bf35-35e2d15c51e0
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 4538e5b49b161f22ba6d5979234786a58cae5783
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37047721"
---
# <a name="tutorial-copy-data-from-blob-storage-to-sql-database-using-data-factory"></a>教學課程：使用 Data Factory 將資料從 Blob 儲存體複製到 SQL Database
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [複製精靈](data-factory-copy-data-wizard-tutorial.md)
> * [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager 範本](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-dot-net.md)。 

在本教學課程中，您會建立 Data Factory 與管線，以將資料從 Blob 儲存體複製到 SQL Database。

複製活動會在 Azure Data Factory 中執行資料移動。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱 [資料移動活動](data-factory-data-movement-activities.md) 文章。  

> [!NOTE]
> 如需 Data Factory 服務的詳細概觀，請參閱 [Azure Data Factory 簡介](data-factory-introduction.md) 一文。
>
>

## <a name="prerequisites-for-the-tutorial"></a>教學課程的必要條件
開始進行本教學課程之前，您必須具備下列必要條件：

* **Azure 訂用帳戶**。  如果您沒有訂用帳戶，則只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [免費試用](http://azure.microsoft.com/pricing/free-trial/) 一文。
* **Azure 儲存體帳戶**。 在本教學課程中，您會使用 Blob 儲存體做為 **來源** 資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱 [建立儲存體帳戶](../../storage/common/storage-create-storage-account.md#create-a-storage-account) 一文以取得建立步驟。
* **Azure SQL Database**。 在本教學課程中，您會使用 Azure SQL Database 做為 **目的地** 資料存放區。 如果您沒有可在教學課程中使用的 Azure SQL Database，請參閱 [如何建立和設定 Azure SQL Database](../../sql-database/sql-database-get-started.md) 建立一個。
* **SQL Server 2012/2014 或 Visual Studio 2013**。 您會使用 SQL Server Management Studio 或 Visual Studio，建立範例資料庫以及檢視資料庫中的結果資料。  

## <a name="collect-blob-storage-account-name-and-key"></a>收集 Blob 儲存體帳戶名稱和金鑰
您需要有 Azure 儲存體帳戶的帳戶名稱和帳戶金鑰，才能進行這個教學課程。 記下 Azure 儲存體帳戶的**帳戶名稱**和**帳戶金鑰**。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 按一下左側功能表中的 [所有服務]，然後選取 [儲存體帳戶]。

    ![瀏覽 - 儲存體帳戶](media/data-factory-copy-data-from-azure-blob-storage-to-sql-database/browse-storage-accounts.png)
3. 在 [儲存體帳戶] 刀鋒視窗中，選取您想要在本教學課程中使用的 [Azure 儲存體帳戶]。
4. 選取 [設定] 底下的 [存取金鑰] 連結。
5. 按一下 [儲存體帳戶名稱] 文字方塊旁的 [複製 (影像)] 按鈕，然後將它儲存/貼到某個位置 (例如：在文字檔中)。
6. 重複上述步驟，複製或記下 **key1**。

    ![儲存體存取金鑰](media/data-factory-copy-data-from-azure-blob-storage-to-sql-database/storage-access-key.png)
7. 按一下 **X**，關閉所有刀鋒視窗。

## <a name="collect-sql-server-database-user-names"></a>收集 SQL Server、資料庫、使用者名稱
您需要有 Azure SQL 伺服器、資料庫和使用者的名稱，才能進行這個教學課程。 記下 Azure SQL Database 的**伺服器**、**資料庫**和**使用者**名稱。

1. 在 **Azure 入口網站**中，按一下左側的 [所有服務]，然後選取 [SQL 資料庫]。
2. 在 [SQL Database] 刀鋒視窗中，選取您想要在本教學課程中使用的**資料庫**。 記下 **資料庫名稱**。  
3. 在 [SQL Database] 刀鋒視窗中，按一下 [設定] 下的 [屬性]。
4. 記下 [伺服器名稱] 和 [伺服器系統管理員登入] 的值。
5. 按一下 **X**，關閉所有刀鋒視窗。

## <a name="allow-azure-services-to-access-sql-server"></a>允許 Azure 服務存取 SQL Server
確定**開啟** Azure SQL 伺服器的 [允許存取 Azure 服務] 設定，讓 Data Factory 服務可以存取您的 Azure SQL 伺服器。 若要確認並開啟此設定，請執行下列步驟：

1. 按一下左側的 [所有服務] 中樞，然後按一下 [SQL 伺服器]。
2. 選取您的伺服器，然後按一下 [設定] 下的 [防火牆]。
3. 在 [防火牆設定] 刀鋒視窗中，對 [允許存取 Azure 服務] 按一下 [開啟]。
4. 按一下 **X**，關閉所有刀鋒視窗。

## <a name="prepare-blob-storage-and-sql-database"></a>準備 Blob 儲存體和 SQL Database
現在，請執行下列步驟，準備本教學課程中的 Azure Blob 儲存體和 Azure SQL Database。  

1. 啟動 [記事本]。 複製以下文字，並命名為 **emp.txt**，然後儲存至您硬碟上的 **C:\ADFGetStarted** 資料夾。

    ```
    John, Doe
    Jane, Doe
    ```
2. 使用 [Azure 儲存體總管](http://storageexplorer.com/)這類的工具建立 **adftutorial** 容器，以及將 **emp.txt** 檔案上傳至該容器。

    ![Azure 儲存體總管。 將資料從 Blob 儲存體複製到 SQL Database](./media/data-factory-copy-data-from-azure-blob-storage-to-sql-database/getstarted-storage-explorer.png)
3. 使用以下 SQL 指令碼，在您的 Azure SQL Database 中建立 **emp** 資料表。  

    ```SQL
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50),
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

    **如果您的電腦上已安裝 SQL Server 2012/2014：** 請遵循[使用 SQL Server Management Studio 管理 Azure SQL Database](../../sql-database/sql-database-manage-azure-ssms.md) 中的指示，連線到您的 Azure SQL Server，並執行 SQL 指令碼。 

    如果不允許您的用戶端存取 Azure SQL Server，則必須將 Azure SQL Server 的防火牆設定成允許從您的電腦 (IP 位址) 存取。 請參閱 [本文](../../sql-database/sql-database-configure-firewall-settings.md) 的步驟，為 Azure SQL Server 設定防火牆。

## <a name="create-a-data-factory"></a>建立 Data Factory
您已完成必要條件。 您可以使用下列其中一個方式建立 Data Factory。 按一下頂端下拉式清單中的其中一個選項，或按一下下列連結以執行教學課程。     

* [複製精靈](data-factory-copy-data-wizard-tutorial.md)
* [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
* [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
* [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
* [Azure Resource Manager 範本](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
* [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
* [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 它不會轉換輸入資料來產生輸出資料。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置第一個管線來轉換資料](data-factory-build-your-first-pipeline.md)。
> 
> 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[在 Data Factory 中排程和執行](data-factory-scheduling-and-execution.md)。 
