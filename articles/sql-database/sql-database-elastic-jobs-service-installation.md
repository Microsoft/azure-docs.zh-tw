---
title: 安裝彈性資料庫工作 | Microsoft Docs
description: 逐步解說如何安裝彈性工作功能。
services: sql-database
ms.service: sql-database
subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 37052c86736a05012b9bb996f07bbf2568d7c879
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47163153"
---
# <a name="installing-elastic-database-jobs-overview"></a>安裝彈性資料庫工作概觀

[!INCLUDE [elastic-database-jobs-deprecation](../../includes/sql-database-elastic-jobs-deprecate.md)]



您可以經由 PowerShell 或透過 Azure 入口網站安裝[**彈性資料庫工作**](sql-database-elastic-jobs-overview.md)。只有在安裝 PowerShell 套件的情況下，您才能取得可使用 PowerShell API 來建立和管理工作的存取權。 此外，PowerShell API 目前比入口網站提供更多的功能。

如果您已透過「入口網站」從現有的「彈性集區」安裝「彈性資料庫工作」，最新的 Powershell 預覽版包含可升級現有安裝的指令碼。 強烈建議將安裝升級至最新的 **彈性資料庫工作** 元件，以利用透過 PowerShell API 公開的新功能。

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶。 如需免費試用版，請參閱 [免費試用版](https://azure.microsoft.com/pricing/free-trial/)。
* Azure PowerShell。 使用 [Web Platform Installer](http://go.microsoft.com/fwlink/p/?linkid=320376)安裝最新版本。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。
* [NuGet 命令列公用程式](https://nuget.org/nuget.exe) 可用來安裝彈性資料庫工作封裝。 如需詳細資訊，請參閱 http://docs.nuget.org/docs/start-here/installing-nuget。

## <a name="download-and-import-the-elastic-database-jobs-powershell-package"></a>下載並匯入彈性資料庫工作 PowerShell 封裝
1. 啟動 Microsoft Azure PowerShell 命令視窗，並瀏覽至您下載 NuGet 命令列公用程式 (nuget.exe) 的目錄。
2. 利用下列命令，將 **彈性資料庫工作** 封裝下載並匯入到目前的目錄：
   
        PS C:\>.\nuget install Microsoft.Azure.SqlDatabase.Jobs -prerelease
   
    「彈性資料庫工作」檔案會放在本機目錄中名為 **Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x** 的資料夾中，其中 *x.x.xxxx.x* 反映的是版本號碼。 PowerShell Cmdlet (包括必要的用戶端 .dll 檔) 位於 **tools\ElasticDatabaseJobs** 子目錄中，而要安裝、升級及解除安裝的 PowerShell 指令碼也位於 [tools] 子目錄中。
3. 例如，輸入  cd tools，瀏覽至 Microsoft.Azure.SqlDatabase.Jobs.x.x.xxx.x 資料夾下的 tools 子目錄：
   
        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*>cd tools

4. 執行 .\InstallElasticDatabaseJobsCmdlets.ps1 script to copy the ElasticDatabaseJobs directory into $home\Documents\WindowsPowerShell\Modules。 這也會自動匯入模組，以供使用，例如：
   
        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobsCmdlets.ps1
        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>.\InstallElasticDatabaseJobsCmdlets.ps1

## <a name="install-the-elastic-database-jobs-components-using-powershell"></a>使用 PowerShell 安裝彈性資料庫工作元件
1. 啟動 Microsoft Azure PowerShell 命令視窗，並且瀏覽至 Microsoft.Azure.SqlDatabase.Jobs.x.x.xxx.x 資料夾底下的 \tools 子目錄：輸入 cd \tools
   
        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*>cd tools

2. 執行.\InstallElasticDatabaseJobs.ps1 PowerShell 指令碼，並提供其所要求之變數的值。 此指令碼將依 [彈性資料庫工作元件和定價](sql-database-elastic-jobs-overview.md#components-and-pricing) 所述建立元件，以及將 Azure 雲端服務設定為適當地使用相依元件。

        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobs.ps1
        PS C:\*Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools>.\InstallElasticDatabaseJobs.ps1

當您執行此命令時，會開啟一個向您詢問「使用者名稱」和「密碼」的視窗。 這不是您的 Azure 認證，請輸入要用於新伺服器之系統管理員認證的使用者名稱和密碼。

在此範例引動過程上提供的參數可以進行修改，以適用於您所需的設定。 下列提供每個參數行為的詳細資訊：

<table style="width:100%">
  <tr>
    <th>參數</th>
    <th>說明</th>
  </tr>

<tr>
    <td>resourceGroupName</td>
    <td>提供建立的 Azure 資源群組名稱，以包含新建的 Azure 元件。 此參數預設為 “__ElasticDatabaseJob”。 不建議變更此值。</td>
    </tr>

</tr>

    <tr>
    <td>ResourceGroupLocation</td>
    <td>提供要用於新建的 Azure 元件的 Azure 位置。 此參數預設為美國中部位置。</td>
</tr>

<tr>
    <td>ServiceWorkerCount</td>
    <td>提供要安裝的服務背景工作數目。 此參數預設為 1。 可以使用更多的背景工作來相應放大服務，並提供高可用性。 建議針對需要服務的高可用性的部署使用 "2"。</td>
    </tr>

</tr>
    <tr>
    <td>ServiceVmSize</td>
    <td>提供在雲端服務內使用的 VM 大小。 此參數預設為 A0。 接受 A0/A1/A2/A3 的參數值，這會導致背景工作角色分別使用 ExtraSmall/Small/Medium/Large 大小。 如需有關背景工作角色大小的詳細資訊，請參閱[彈性資料庫工作元件和價格](sql-database-elastic-jobs-overview.md#components-and-pricing)。</td>
</tr>

</tr>
    <tr>
    <td>SqlServerDatabaseSlo</td>
    <td>提供標準版的計算大小。 此參數預設為 S0。 接受 S0/S1/S2/S3/S4/S6/S9/S12 的參數值，這會導致 Azure SQL Database 使用各自的計算大小。 如需有關 SQL Database 計算大小的詳細資訊，請參閱[彈性資料庫工作元件和價格](sql-database-elastic-jobs-overview.md#components-and-pricing)。</td>
</tr>

</tr>
    <tr>
    <td>SqlServerAdministratorUserName</td>
    <td>提供新建的 Azure SQL Database 伺服器的系統管理員使用者名稱。 未指定時，將開啟 PowerShell 認證視窗，提示輸入認證。</td>
</tr>

</tr>
    <tr>
    <td>SqlServerAdministratorPassword</td>
    <td>提供新建的 Azure SQL Database 伺服器的系統管理員密碼。 未提供時，將開啟 PowerShell 認證視窗，提示輸入認證。</td>
</tr>
</table>

對於將對大量資料庫具有大量平行執行的工作設為目標的系統，建議指定如下參數：-ServiceWorkerCount 2 -ServiceVmSize A2 -SqlServerDatabaseSlo S2.

    PS C:\*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>Unblock-File .\InstallElasticDatabaseJobs.ps1
    PS C:\*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>.\InstallElasticDatabaseJobs.ps1 -ServiceWorkerCount 2 -ServiceVmSize A2 -SqlServerDatabaseSlo S2

## <a name="update-an-existing-elastic-database-jobs-components-installation-using-powershell"></a>使用 PowerShell 更新現有的彈性資料庫工作元件安裝
**彈性資料庫工作** 可以在現有的安裝中更新，以進行擴充和高可用性。 此程序允許未來的服務程式碼升級，而不需捨棄並重新建立控制資料庫。 此程序也可在相同版本內用來修改服務的 VM 大小或伺服器的背景工作計數。

若要更新安裝的 VM 大小，請執行下列指令碼，並將參數更新為您選擇的值。

    PS C:\*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>Unblock-File .\UpdateElasticDatabaseJobs.ps1
    PS C:\*Microsoft.Azure.SqlDatabase.Jobs.dll.x.x.xxx.x*\tools>.\UpdateElasticDatabaseJobs.ps1 -ServiceVmSize A1 -ServiceWorkerCount 2

<table style="width:100%">
  <tr>
  <th>參數</th>
  <th>說明</th>
</tr>

  <tr>
    <td>resourceGroupName</td>
    <td>識別一開始安裝彈性資料庫工作元件時所使用的 Azure 資源群組名稱。 此參數預設為 “__ElasticDatabaseJob”。 不建議變更此值，因為您應該不必指定此參數。</td>
    </tr>
</tr>

</tr>

  <tr>
    <td>ServiceWorkerCount</td>
    <td>提供要安裝的服務背景工作數目。  此參數預設為 1。  可以使用更多的背景工作來相應放大服務，並提供高可用性。  建議針對需要服務的高可用性的部署使用 "2"。</td>
</tr>

</tr>

    <tr>
    <td>ServiceVmSize</td>
    <td>提供在雲端服務內使用的 VM 大小。 此參數預設為 A0。 接受 A0/A1/A2/A3 的參數值，這會導致背景工作角色分別使用 ExtraSmall/Small/Medium/Large 大小。 如需有關背景工作角色大小的詳細資訊，請參閱[彈性資料庫工作元件和價格](sql-database-elastic-jobs-overview.md#components-and-pricing)。</td>
</tr>

</table>

## <a name="install-the-elastic-database-jobs-components-using-the-portal"></a>使用入口網站安裝彈性資料庫工作元件
在[建立彈性資料庫集區](sql-database-elastic-pool-manage-portal.md)之後，您可以安裝「彈性資料庫工作」元件，以便能夠對彈性集區中的每個資料庫執行系統管理工作。 不同於使用 **彈性資料庫工作** PowerShell API 時，入口網站介面目前限制為只針對現有的集區執行。

**預估完成時間：** 10 分鐘。

1. 透過 [Azure 入口網站](https://portal.azure.com/#)，從彈性集區的儀表板檢視按一下 [建立工作]。
2. 如果您是第一次建立工作，則必須按一下 [預覽條款] 來安裝「彈性資料庫工作」。
3. 然後按一下核取方塊接受條款。
4. 在 [安裝服務] 檢視中，按一下 [工作認證] 。
   
    ![安裝服務][1]
5. 輸入資料庫管理員的使用者名稱和密碼。在安裝過程中，會建立新的 Azure SQL Database 伺服器。 在這個新的伺服器內，會建立稱為控制資料庫的新資料庫，並用來包含彈性資料庫工作的中繼資料。 這裡建立的使用者名稱和密碼用於登入控制資料庫。 個別的認證用於對集區內的資料庫執行指令碼。
   
    ![建立使用者名稱和密碼][2]
6. 按一下 [確定] 按鈕。 幾分鐘內，就會在新的[資源群組](../azure-resource-manager/resource-group-overview.md)中為您建立元件。 新的資源群組已釘選到「開始面板」，如下所示。 建立後，會在群組中建立所有彈性資料庫工作 (雲端服務、SQL Database、服務匯流排和儲存體)。
   
    ![「開始面板」中的資源群組][3]
7. 如果您嘗試在安裝彈性資料庫工作時建立或管理工作，您會在提供 **認證** 時看到下列訊息。
   
    ![部署仍在進行中][4]

如果需要解除安裝，請刪除資源群組。 請參閱 [如何解除安裝彈性資料庫工作元件](sql-database-elastic-jobs-uninstall.md)。

## <a name="next-steps"></a>後續步驟
確定在群組中的每個資料庫上建立具備適當指令碼執行權限的認證，如需詳細資訊，請參閱[保護您的 SQL Database](sql-database-manage-logins.md)。
請參閱[建立和管理彈性資料庫工作](sql-database-elastic-jobs-create-and-manage.md)以開始進行。

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-service-installation/screen-1.png
[2]: ./media/sql-database-elastic-jobs-service-installation/credentials.png
[3]: ./media/sql-database-elastic-jobs-service-installation/start-board.png
[4]: ./media/sql-database-elastic-jobs-service-installation/not-done.png
