---
title: 在 Azure 中備份應用程式
description: 了解如何在 Azure App Service 中建立應用程式的備份。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: jimbe
ms.assetid: 6223b6bd-84ec-48df-943f-461d84605694
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2016
ms.author: cephalin
ms.openlocfilehash: 44b4da7c293da0643fb88cc2de21433c6ea72c5c
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42886399"
---
# <a name="back-up-your-app-in-azure"></a>在 Azure 中備份應用程式
[Azure App Service](app-service-web-overview.md) 中的「備份與還原」功能可讓您以手動或透過排程方式，輕鬆建立應用程式備份。 您可以透過覆寫現有的應用程式或還原到另一個應用程式，將應用程式還原到先前狀態的快照。 

如需從備份還原應用程式的相關資訊，請參閱 [在 Azure 中還原應用程式](web-sites-restore.md)。

<a name="whatsbackedup"></a>

## <a name="what-gets-backed-up"></a>備份什麼項目
App Service 可以將下列資訊備份到您已設定讓應用程式使用的 Azure 儲存體帳戶和容器。 

* 應用程式設定
* 檔案內容
* 已連線到您應用程式的資料庫

備份功能支援下列資料庫解決方案： 
   - [SQL Database](https://azure.microsoft.com/services/sql-database/)
   - [適用於 MySQL 的 Azure 資料庫](https://azure.microsoft.com/services/mysql)
   - [適用於 PostgreSQL 的 Azure 資料庫](https://azure.microsoft.com/services/postgresql)
   - [應用程式內 MySQL](https://blogs.msdn.microsoft.com/appserviceteam/2017/03/06/announcing-general-availability-for-mysql-in-app)
 

> [!NOTE]
>  每個備份都是應用程式的完整離線複本，而不是增量更新。
>  

<a name="requirements"></a>

## <a name="requirements-and-restrictions"></a>需求和限制
* 若要使用「備份與還原」功能，App Service 方案必須屬於**標準**層或**進階**層。 如需有關調整 App Service 方案以使用更高階層的詳細資訊，請參閱 [在 Azure 中調整應用程式規模](web-sites-scale.md)。  
  「進階」層所允許的每日備份數量比「標準」層多。
* 您需要與您即將備份之應用程式隸屬於相同訂用帳戶的 Azure 儲存體帳戶和容器。 如需 Azure 儲存體帳戶的詳細資訊，請參閱本文結尾處的 [連結](#moreaboutstorage) 。
* 備份上限是 10 GB 的應用程式和資料庫內容。 如果備份大小超出此限制，您就會收到錯誤。
* 不支援備份具 SSL 功能的適用於 MySQL 的 Azure 資料庫。 如果設定備份，會得到失敗的備份檔案。
* 不支援備份具 SSL 功能的適用於 PostgreSQL 的 Azure 資料庫。 如果設定備份，會得到失敗的備份檔案。
* 應用程式內 MySQL 資料庫會自動備份 (不含任何設定)。 如果您對應用程式內 MySQL 資料庫進行手動設定 (例如，新增連接字串)，可能就無法正確備份。
* 不支援將具防火牆功能的儲存體帳戶做為備份目的地。 如果設定備份，會得到失敗的備份檔案。


<a name="manualbackup"></a>

## <a name="create-a-manual-backup"></a>建立手動備份
1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您的應用程式頁面，然後選取 [備份]。 [備份] 頁面隨即顯示。
   
    ![Backups page][ChooseBackupsPage]
   
   > [!NOTE]
   > 如果您看到下列訊息，必須先按一下該訊息來升級 App Service 方案，才能繼續進行備份。
   > 如需詳細資訊，請參閱[在 Azure 中為應用程式進行相應增加](web-sites-scale.md)。  
   > ![Choose storage account](./media/web-sites-backup/01UpgradePlan1.png)
   > 
   > 

2. 在 [備份] 頁面中，按一下 [設定]
![按一下 [設定]](./media/web-sites-backup/ClickConfigure1.png)
3. 在 [備份設定] 頁面中，按一下 [儲存體: 未設定] 以設定儲存體帳戶。
   
    ![Choose storage account][ChooseStorageAccount]
4. 選取 [儲存體帳戶] 和 [容器]，以選擇您的備份目的地。 此儲存體帳戶必須與您要備份之應用程式隸屬於相同的訂用帳戶。 如果您希望的話，也可以在個別頁面中，建立新的儲存體帳戶或新的容器。 完成後，按一下 [選取] 。
   
    ![Choose storage account](./media/web-sites-backup/02ChooseStorageAccount1-1.png)
5. 在仍處於開啟狀態的 [備份設定] 頁面中，您可以設定 [備份資料庫]，然後選取您想要包含在備份中的資料庫 (SQL Database 或 MySQL)，然後按一下 [確定]。  
   
    ![Choose storage account](./media/web-sites-backup/03ConfigureDatabase1.png)
   
   > [!NOTE]
   > 若要讓資料庫出現在此清單中，其連接字串必須存在於您應用程式之 [應用程式設定] 頁面的 [連接字串] 區段中。 
   >
   > 應用程式內 MySQL 資料庫會自動備份 (不含任何設定)。 如果您對應用程式內 MySQL 資料庫進行手動設定 (例如，新增連接字串)，可能就無法正確備份。
   > 
   > 
6. 在 [備份設定] 頁面中，按一下 [儲存]。    
7. 在 [備份] 頁面中，按一下 [備份]。
   
    ![BackUpNow button][BackUpNow]
   
    在備份過程中，您會看見進度訊息。

設定儲存體帳戶和容器之後，您可以隨時起始手動備份。  

<a name="automatedbackups"></a>

## <a name="configure-automated-backups"></a>設定自動備份
1. 在 [備份設定] 頁面中，將 [排定的備份] 設定為 [開啟]。 
   
    ![Choose storage account](./media/web-sites-backup/05ScheduleBackup1.png)
2. 將會顯示備份排程選項，請將 [排定的備份] 設定為 [開啟]，然後視需要設定備份排程，並按一下 [確定]。
   
    ![Enable automated backups][SetAutomatedBackupOn]

<a name="partialbackups"></a>

## <a name="configure-partial-backups"></a>設定部分備份
有時您並不想要備份應用程式上的所有項目。 以下是一些範例：

* 您為包含永不變更的靜態內容 (例如舊部落格文章或影像) 的應用程式 [設定每週備份](web-sites-backup.md#configure-automated-backups) 。
* 您應用程式的內容超過 10 GB (這是一次可備份的數量上限)。
* 您不想要備份記錄檔。

部分備份可讓您精確選擇想要備份的檔案。

### <a name="exclude-files-from-your-backup"></a>從備份中排除檔案
假設您有一個應用程式，其中包含已經備份過一次且不會再變更的記錄檔和靜態映像。 在這類情況下，您可以將這些資料夾和檔案排除，而不儲存在您未來的備份中。 若要將檔案和資料夾從您的備份中排除，請在應用程式的 `D:\home\site\wwwroot` 資料夾中建立 `_backup.filter` 檔案。 請在此檔案中指定您想要排除的檔案和資料夾清單。 

有一個可輕鬆存取您檔案的方式，就是使用 Kudu。 請按一下您 Web 應用程式的 [進階工具] > [執行] 設定來存取 Kudu。

![使用入口網站時的 Kuku][kudu-portal]

識別您想要從備份中排除的資料夾。  例如，您想要篩選掉醒目提示的資料夾和檔案。

![Images 資料夾][ImagesFolder]

建立名為 `_backup.filter` 的檔案並將上述清單放入此檔案中，但請移除 `D:\home`。 一行列出一個目錄或檔案。 因此，檔案的內容應該如下：
 ```bash
    \site\wwwroot\Images\brand.png
    \site\wwwroot\Images\2014
    \site\wwwroot\Images\2013
```

使用 [ftp](app-service-deploy-ftp.md) 或任何其他方法，將 `_backup.filter` 檔案上傳到您站台的 `D:\home\site\wwwroot\` 目錄。 如果您希望的話，也可以使用 Kudu `DebugConsole` 直接建立該檔案，然後在該處插入內容。

以平常執行備份的相同方式執行備份：[手動](#create-a-manual-backup)或[自動](#configure-automated-backups)。 現在，會將 `_backup.filter` 中指定的所有檔案和資料夾，從所排定或手動起始的未來備份中排除。 

> [!NOTE]
> 您還原站台部分備份的方式會與[還原一般備份](web-sites-restore.md)的方式相同。 還原程序會執行正確的作業。
> 
> 還原完整備份時，網站上的所有內容都會取代為備份中的內容。 如果檔案在網站上，而不在備份中，系統就會將它刪除。 但是，還原部分備份時，位於其中一個封鎖清單目錄或任何封鎖清單檔案中的任何內容都會保持原狀。
> 


<a name="aboutbackups"></a>

## <a name="how-backups-are-stored"></a>備份的儲存方式
在您為應用程式建立一或多個備份之後，這些備份就會顯示在您儲存體帳戶及應用程式的 [容器] 頁面中。 在儲存體帳戶中，每個備份都是由一個 `.zip` 檔案 (包含備份資料) 和一個 `.xml` 檔案 (包含 `.zip` 檔案內容的資訊清單) 所組成。 如果您要存取備份而不實際執行應用程式還原，則可以將這些檔案解壓縮並加以瀏覽。

應用程式的資料庫備份則儲存在 .zip 檔案的根目錄中。 若是 SQL 資料庫，這會是 BACPAC 檔案 (無副檔名)，而且可以匯入。 若要根據 BACPAC 匯出內容建立的 SQL Database，請參閱[匯入 BACPAC 檔案以建立新的使用者資料庫](http://technet.microsoft.com/library/hh710052.aspx)。

> [!WARNING]
> 對 **websitebackups** 容器中的檔案進行任何變更，都可能導致備份失效，進而無法還原。
> 
> 

## <a name="automate-with-scripts"></a>使用指令碼進行自動化

您可以使用 [Azure CLI](/cli/azure/install-azure-cli) 或 [Azure PowerShell](/powershell/azure/overview)，透過指令碼將備份管理作業自動化。

例如，請參閱：

- [Azure CLI 範例](app-service-cli-samples.md)
- [Azure PowerShell 範例](app-service-powershell-samples.md)

<a name="nextsteps"></a>

## <a name="next-steps"></a>後續步驟
如需從備份還原應用程式的相關資訊，請參閱 [在 Azure 中還原應用程式](web-sites-restore.md)。 


<!-- IMAGES -->
[ChooseBackupsPage]: ./media/web-sites-backup/01ChooseBackupsPage1.png
[ChooseStorageAccount]: ./media/web-sites-backup/02ChooseStorageAccount-1.png
[BackUpNow]: ./media/web-sites-backup/04BackUpNow1.png
[SetAutomatedBackupOn]: ./media/web-sites-backup/06SetAutomatedBackupOn1.png
[SaveIcon]: ./media/web-sites-backup/10SaveIcon.png
[ImagesFolder]: ./media/web-sites-backup/11Images.png
[LogsFolder]: ./media/web-sites-backup/12Logs.png
[GhostUpgradeWarning]: ./media/web-sites-backup/13GhostUpgradeWarning.png
[kudu-portal]:./media/web-sites-backup/kudu-portal.PNG

