---
title: 在 Azure 中還原應用程式
description: 了解如何從備份還原您的應用程式。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: jimbe
ms.assetid: 4444dbf7-363c-47e2-b24a-dbd45cb08491
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2016
ms.author: cephalin
ms.openlocfilehash: 69e0e8282ee0b8503fe11a57b8ba6037247822dd
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39160496"
---
# <a name="restore-an-app-in-azure"></a>在 Azure 中還原應用程式
本文說明如何在 [Azure App Service](../app-service/app-service-web-overview.md) 中還原您先前備份的應用程式 (請參閱[在 Azure 中備份應用程式](web-sites-backup.md))。 您可以依需求將應用程式及其連結的資料庫還原到先前的狀態，或是根據您的其中一個原始應用程式備份來建立新的應用程式。 Azure App Service 支援使用下列資料庫來進行備份與還原︰
- [SQL Database](https://azure.microsoft.com/services/sql-database/)
- [適用於 MySQL 的 Azure 資料庫](https://azure.microsoft.com/services/mysql)
- [適用於 PostgreSQL 的 Azure 資料庫](https://azure.microsoft.com/services/postgresql)
- [應用程式內 MySQL](https://blogs.msdn.microsoft.com/appserviceteam/2017/03/06/announcing-general-availability-for-mysql-in-app)

從備份還原是提供給在**標準**和**進階**層中執行的應用程式使用。 如需有關相應增加應用程式的詳細資訊，請參閱 [在 Azure 中調整應用程式規模](web-sites-scale.md)。 **進階**層所允許執行的每日備份數量比**標準**層多。

<a name="PreviousBackup"></a>

## <a name="restore-an-app-from-an-existing-backup"></a>從現有備份還原應用程式
1. 在 Azure 入口網站中您應用程式的 [設定] 頁面上，按一下 [備份] 以顯示 [備份] 頁面。 然後按一下 [還原]。
   
    ![選擇立即還原][ChooseRestoreNow]
2. 在 [還原] 頁面中，先選取備份來源。
   
    ![](./media/web-sites-restore/021ChooseSource1.png)
   
    [應用程式備份]  選項會顯示目前應用程式的所有現有備份，您可以輕鬆地選取其中一個。
    [儲存體]  選項可讓您從您訂用帳戶中任何現有 Azure 儲存體帳戶和容器中，選取任何備份 ZIP 檔案。
    如果您想要還原另一個應用程式的備份，請使用 [儲存體]  選項。
3. 接著，在 [還原目的地] 中指定應用程式還原目的地。
   
    ![](./media/web-sites-restore/022ChooseDestination1.png)
   
   > [!WARNING]
   > 如果您選擇 [覆寫]，則目前應用程式中的所有現有資料都將被清除並覆寫。 按一下 [確定] 之前，請確定這確實是您想要執行的動作。
   > 
   > 
   
   > [!WARNING]
   > 如果 App Service 在您還原資料庫時將資料寫入資料庫，則可能會導致主索引鍵違規和資料遺失等徵兆。 建議您在開始還原資料庫之前，先停止 App Service。
   > 
   > 
   
    您可以選取 [現有應用程式]  ，將應用程式備份還原到相同資源群組中的另一個應用程式。 使用此選項之前，您應該已經在資源群組中，建立具有應用程式備份中所定義之資料庫組態的鏡像資料庫組態的另一個應用程式。 您也可以建立**新**應用程式作為還原內容的目標。

4. 按一下 [確定]。

<a name="StorageAccount"></a>

## <a name="download-or-delete-a-backup-from-a-storage-account"></a>從儲存體帳戶下載或刪除備份
1. 從 Azure 入口網站的主要 [瀏覽] 頁面中，選取 [儲存體帳戶]。 隨即會顯示您現有的儲存體帳戶清單。
2. 選取包含您想要下載或刪除之備份的儲存體帳戶。 儲存體帳戶的頁面隨即顯示。
3. 在儲存體帳戶頁面中，選取所需的容器
   
    ![檢視容器][ViewContainers]
4. 選取您想要下載或刪除的備份檔案。
   
    ![ViewContainers](./media/web-sites-restore/03ViewFiles.png)
5. 按一下 [下載] 或 [刪除]，視您要進行的作業而定。  

<a name="OperationLogs"></a>

## <a name="monitor-a-restore-operation"></a>監視還原作業
若要查看有關應用程式還原作業成功或失敗的詳細資訊，請瀏覽至 Azure 入口網站中的 [活動記錄] 頁面。  
 

向下捲動以尋找所要的還原作業，並按一下以選取它。

詳細資料頁面會顯示與還原作業有關的可用資訊。

## <a name="automate-with-scripts"></a>使用指令碼進行自動化

您可以使用 [Azure CLI](/cli/azure/install-azure-cli) 或 [Azure PowerShell](/powershell/azure/overview)，透過指令碼將備份管理作業自動化。

例如，請參閱：

- [Azure CLI 範例](app-service-cli-samples.md)
- [Azure PowerShell 範例](app-service-powershell-samples.md)

<!-- ## Next Steps
You can backup and restore App Service apps using REST API. -->


<!-- IMAGES -->
[ChooseRestoreNow]: ./media/web-sites-restore/02ChooseRestoreNow1.png
[ViewContainers]: ./media/web-sites-restore/03ViewContainers.png
