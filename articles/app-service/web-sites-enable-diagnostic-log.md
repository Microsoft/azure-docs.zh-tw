---
title: 在 Azure App Service 中針對 Web 應用程式啟用診斷記錄功能。
description: 了解如何啟用診斷記錄，並在您的應用程式中加入診斷工具，以及如何存取 Azure 所記錄的資訊。
services: app-service
documentationcenter: .net
author: cephalin
manager: erikre
editor: jimbe
ms.assetid: c9da27b2-47d4-4c33-a3cb-1819955ee43b
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/06/2016
ms.author: cephalin
ms.openlocfilehash: 5cd56abd02c55dbf72c92ed070f9988fae2b6762
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365249"
---
# <a name="enable-diagnostics-logging-for-web-apps-in-azure-app-service"></a>在 Azure App Service 中針對 Web 應用程式啟用診斷記錄功能。
## <a name="overview"></a>概觀
Azure 提供內建診斷功能，可協助對 [App Service Web 應用程式](http://go.microsoft.com/fwlink/?LinkId=529714)進行偵錯。 您會在本文中了解如何啟用診斷記錄，並在您的應用程式中加入檢測，以及如何存取 Azure 所記錄的資訊。

本文使用 [Azure 入口網站](https://portal.azure.com)、Azure PowerShell 及 Azure 命令列介面 (Azure CLI) 來處理診斷記錄。 如需使用 Visual Studio 來處理診斷記錄的詳細資訊，請參閱 [在 Visual Studio 中疑難排解 Azure](web-sites-dotnet-troubleshoot-visual-studio.md)。

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="whatisdiag"></a>Web 伺服器診斷和應用程式診斷
App Service Web 應用程式會針對來自 Web 伺服器和 Web 應用程式的記錄資訊提供診斷功能。 這些資訊邏輯上可區分為 [Web 伺服器診斷] 與 [應用程式診斷]。

### <a name="web-server-diagnostics"></a>Web 伺服器診斷
您可以啟用或停用下列各種記錄：

* **詳細的錯誤記錄** - 對於表示失敗的 HTTP 狀態碼 (狀態碼 400 或更大) 的詳細錯誤資訊。 它包含的資訊可協助您判斷為何伺服器傳回錯誤碼。
* **失敗的要求追蹤** - 關於失敗要求的詳細資訊，包括用於處理要求的 IIS 元件追蹤，以及每個元件所花費的時間。 如果您嘗試提升網站效能或是想要從傳回的特定 HTTP 錯誤中找到發生原因，這非常實用。
* **Web 伺服器記錄** - 使用 [W3C 擴充記錄檔格式](http://msdn.microsoft.com/library/windows/desktop/aa814385.aspx)的 HTTP 交易相關資訊。 當您需要判斷整體網站指標 (例如，處理的要求數目，或者有多少要求來自特定的 IP 位址) 時，這非常實用。

### <a name="application-diagnostics"></a>應用程式診斷
應用程式診斷功能可讓您擷取 Web 應用程式所產生的資訊。 ASP.NET 應用程式會使用 [System.Diagnostics.Trace](http://msdn.microsoft.com/library/36hhw2t6.aspx) 類別將資訊記錄到應用程式診斷記錄。 例如︰

    System.Diagnostics.Trace.TraceError("If you're seeing this, something bad happened");

您可以在執行階段擷取這些記錄檔，以協助疑難排解。 如需詳細資訊，請參閱 [在 Visual Studio 中疑難排解 Azure Web 應用程式](web-sites-dotnet-troubleshoot-visual-studio.md)。

當您將內容發行至 Web 應用程式時，App Service Web 應用程式也會記錄部署資訊。 此動作會自動發生，因此無須任何組態設定即會記錄部署動作。 部署記錄功能可讓您判斷部署失敗的原因。 例如，如果您是使用自訂的部署指令碼，則您可以使用部署記錄功能來判斷指令碼失敗的原因。

## <a name="enablediag"></a>如何啟用診斷
若要在 [Azure 入口網站](https://portal.azure.com)中啟用診斷，請移至 Web 應用程式的頁面，然後依序按一下 [設定] > [診斷記錄]。

<!-- todo:cleanup dogfood addresses in screenshot -->
![[記錄] 部分](./media/web-sites-enable-diagnostic-log/logspart.png)

啟用 [應用程式診斷] 時，也會選擇 [層級]。 此設定可讓您篩選擷取至**資訊**、**警告**或**錯誤**資訊中的資訊。 將此選項設定為 [詳細資訊] 會記錄由該應用程式產生的所有資訊。

> [!NOTE]
> 與變更 web.config 檔案不同，啟用應用程式診斷或是變更診斷記錄層級並不會回收用來執行應用程式的應用程式網域。
>
>

針對 [應用程式記錄]，您可以暫時開啟檔案系統選項以供偵錯之用。 這個選項會在 12 小時後自動關閉。 您也可以開啟 Blob 儲存體選項，來選取要寫入記錄的目標 Blob 容器。

針對 [Web 伺服器記錄]，您可以選取 [儲存體] 或 [檔案系統]。 選取 [儲存體] 可讓您選取儲存體帳戶，並接著選取可供寫入記錄的 Blob 容器。 

如果您在檔案系統上儲存記錄，這些檔案可透過 FTP 存取，或是使用 Azure PowerShell 或 Azure 命令列介面 (Azure CLI) 下載為 Zip 封存。

根據預設，不會自動刪除記錄 (除了 [應用程式記錄 (檔案系統)] 之外)。 若要自動刪除記錄，請設定 [保留期限 (天)] 欄位。

> [!NOTE]
> 如果您 [重新產生儲存體帳戶的存取金鑰](../storage/common/storage-create-storage-account.md)，您必須重設個別的記錄組態，才能使用更新的金鑰。 作法：
>
> 1. 在 [設定] 索引標籤上，將個別的記錄功能設定為 [關閉]。 儲存您的設定。
> 2. 重新啟用記錄至儲存體帳戶 Blob 或資料表。 儲存您的設定。
>
>

包括檔案系統、資料表儲存體或是 Blob 儲存體都可以任意組合並同時啟用，並個別具有記錄層級組態。 例如，您也許想要將各種錯誤與警告資訊記錄到 Blob 儲存體做為長期的記錄解決方案，同時啟用詳細資訊層級的檔案系統記錄功能。

雖然以上三個儲存位置全都提供相同的基本資訊供您記錄事件，[資料表儲存體] 與 [Blob 儲存體] 會比 [檔案系統] 記錄更多的資訊，例如執行個體識別碼、執行緒識別碼以及更細緻的時間戳記 (刻度格式)。

> [!NOTE]
> 儲存在 [資料表儲存體] 或 [Blob 儲存體] 內的資訊只能透過儲存用戶端，或是能夠直接使用這些儲存系統的應用程式來存取。 例如，Visual Studio 2013 內含的 [儲存體總管] 可用來探索資料表或 Blob 儲存體，而 HDInsight 則可存取儲存在 Blob 儲存體內的資料。 您也可以使用任何一項 [Azure SDK](https://azure.microsoft.com/downloads/)，撰寫可存取 Azure 儲存體的應用程式。
>
> [!NOTE]
> 您也可以在 Azure PowerShell 中使用 **Set-AzureWebsite** Cmdlet 來啟用診斷功能。 如果您尚未安裝 Azure PowerShell，或尚未將其設定為使用 Azure 訂用帳戶，請參閱[安裝並設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.6.0)。
>
>

## <a name="download"></a> 作法：下載記錄
儲存在 Web 應用程式檔案系統中的診斷資訊，可透過 FTP 直接存取。 或是使用 Azure PowerShell 或 Azure 命令列介面下載為 Zip 封存。

儲存這些記錄的目錄結構如下所示：

* **應用程式記錄** - /LogFiles/Application/。 此資料夾內含有一或多個文字檔案，這些檔案涵蓋應用程式記錄所產生的資訊。
* **失敗要求追蹤** - /LogFiles/W3SVC#########/。 此資料夾內含有一個 XSL 檔案和一或多個 XML 檔案。 請確保將 XSL 檔案下載至 XML 檔案所在的相同目錄，因為 XSL 檔案可提供格式化功能，讓您在 Internet Explorer 中檢視時能夠篩選 XML 檔案內容。
* **詳細錯誤記錄** - /LogFiles/DetailedErrors/。 此資料夾包含一或多個 .htm 檔案，內含已經發生的任何 HTTP 錯誤之詳細資訊。
* **Web 伺服器記錄** - /LogFiles/http/RawLogs。 此資料夾包含一或多個運用 [W3C 擴充記錄檔格式](http://msdn.microsoft.com/library/windows/desktop/aa814385.aspx)來格式化的文字檔案。
* **部署記錄** - /LogFiles/Git。 此資料夾包含由內部部署處理序所產生，且可供 Azure Web 應用程式運用的記錄，以及適用於 Git 部署的記錄。 您也可以在 D:\home\site\deployments 下找到部署記錄。

### <a name="ftp"></a>FTP

若要開啟連線到您應用程式 FTP 伺服器的 FTP 連線，請參閱[使用 FTP/S 將您的應用程式部署至 Azure App Service](app-service-deploy-ftp.md)。

一旦連線到您的 Web 應用程式 FTP/S 伺服器後，開啟儲存記錄檔所在的 [LogFiles] 資料夾。

### <a name="download-with-azure-powershell"></a>使用 Azure PowerShell 來下載
若要下載記錄檔，請啟動新的 Azure PowerShell 執行個體並使用下列命令：

    Save-AzureWebSiteLog -Name webappname

此命令會將 **-Name** 參數所指定的 Web 應用程式記錄儲存到目前目錄中名為 **logs.zip** 的檔案中。

> [!NOTE]
> 如果您尚未安裝 Azure PowerShell，或尚未將其設定為使用 Azure 訂用帳戶，請參閱[安裝並設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.6.0)。
>
>

### <a name="download-with-azure-command-line-interface"></a>使用 Azure 命列列介面來下載
若要使用 Azure 命令列介面來下載記錄檔案，請開啟新的命令列提示字元、PowerShell、Bash 或終端機工作階段，然後輸入下列命令：

    az webapp log download --resource-group resourcegroupname --name webappname

此命令會將名為 'webappname' 的 Web 應用程式記錄儲存至目前目錄中名為 **diagnostics.zip** 的檔案。

> [!NOTE]
> 如果您尚未安裝 Azure 命令列介面 (Azure CLI)，或是尚未將其設定為使用您的 Azure 訂用帳戶，請參閱 [如何使用 Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)。
>
>

## <a name="how-to-view-logs-in-application-insights"></a>作法：在 Application Insights 中檢視記錄
Visual Studio Application Insights 提供篩選與搜尋記錄的工具，以及將記錄與要求及其他事件建立相互關聯的工具。

1. 在 Visual Studio 中將 Application Insights SDK 加入至專案。
   * 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後選擇 [加入 Application Insights]。 介面會指導您完成包括建立 Application Insights 資源在內的所有步驟。 [深入了解](../application-insights/app-insights-asp-net.md)
2. 將追蹤接聽套件新增至專案。
   * 以滑鼠右鍵按一下專案，然後選擇 [管理 NuGet 封裝]。 選取 `Microsoft.ApplicationInsights.TraceListener` [深入了解](../application-insights/app-insights-asp-net-trace-logs.md)
3. 上傳您的專案並執行，以產生記錄資料。
4. 在 [Azure 入口網站](https://portal.azure.com/)中，瀏覽至您新的 Application Insights 資源，然後開啟 [搜尋]。 您應該會看到您的記錄資料，以及要求、使用情況及其他遙測。 有些遙測可能需要數分鐘才能抵達：按一下 [重新整理]。 [深入了解](../application-insights/app-insights-diagnostic-search.md)

[深入了解使用 Application Insights 的效能追蹤](../application-insights/app-insights-azure-web-apps.md)

## <a name="streamlogs"></a> 作法：串流記錄
開發應用程式時，如果能夠幾近即時地檢視記錄資訊，通常會很實用。 您可以使用 Azure PowerShell 或 Azure 命令列介面，將記錄資訊串流至開發環境。

> [!NOTE]
> 某些記錄緩衝區類型會寫入記錄檔中，進而造成串流中的事件順序錯亂。 例如，使用者造訪某個網頁所產生的應用程式記錄項目，可能會比頁面要求的對應 HTTP 記錄項目優先顯示在串流中。
>
> [!NOTE]
> 記錄串流也會將串流資訊寫入儲存於 **D:\\home\\LogFiles\\** 資料夾中的任何文字檔。
>
>

### <a name="streaming-with-azure-powershell"></a>使用 Azure PowerShell 來串流
若要串流記錄資訊，請啟動新的 Azure PowerShell 執行個體並使用下列命令：

    Get-AzureWebSiteLog -Name webappname -Tail

此命令會連線至 **-Name** 參數所指定的 Web 應用程式，並在該 Web 應用程式產生記錄事件時，開始將資訊串流至 PowerShell 視窗。 任何寫入副檔名為 .txt、.log 或 .htm 的檔案中並存放在 /LogFiles 目錄 (d:/home/logfiles) 的資訊，都會串流至本機主控台。

若要篩選特定事件，例如錯誤，請使用 **-Message** 參數。 例如︰

    Get-AzureWebSiteLog -Name webappname -Tail -Message Error

若要篩選特定記錄類型，例如 HTTP，請使用 **-Path** 參數。 例如︰

    Get-AzureWebSiteLog -Name webappname -Tail -Path http

若要檢視可用的路徑清單，請使用 -ListPath 參數。

> [!NOTE]
> 如果您尚未安裝 Azure PowerShell，或尚未將其設定為使用 Azure 訂閱，請參閱 [如何使用 Azure PowerShell](http://azure.microsoft.com/develop/nodejs/how-to-guides/powershell-cmdlets/)(英文)。
>
>

### <a name="streaming-with-azure-command-line-interface"></a>使用 Azure 命令列介面來串流
若要串流記錄資訊，請開啟新的命列列提示字元、PowerShell、Bash 或終端機工作階段，然後輸入下列命令：

    az webapp log tail --name webappname --resource-group myResourceGroup

此命令會連線至名為 'webappname' 的 Web 應用程式，並在該 Web 應用程式產生記錄事件時，開始將資訊串流至視窗。 任何寫入副檔名為 .txt、.log 或 .htm 的檔案中並存放在 /LogFiles 目錄 (d:/home/logfiles) 的資訊，都會串流至本機主控台。

若要篩選特定事件，例如錯誤，請使用 **--Filter** 參數。 例如︰

    az webapp log tail --name webappname --resource-group myResourceGroup --filter Error

若要篩選特定記錄類型，例如 HTTP，請使用 **--Path** 參數。 例如︰

    az webapp log tail --name webappname --resource-group myResourceGroup --path http

> [!NOTE]
> 如果您尚未安裝 Azure 命令列介面，或是尚未將其設定為使用您的 Azure 訂用帳戶，請參閱 [如何使用 Azure 命令列介面](../cli-install-nodejs.md)。
>
>

## <a name="understandlogs"></a> 作法：了解診斷記錄
### <a name="application-diagnostics-logs"></a>應用程式診斷記錄
應用程式診斷功能會根據您將記錄儲存至檔案系統、資料表儲存體或 Blob 儲存體的不同，將 .NET 應用程式的資訊儲存為特定格式。 這三種儲存類型所儲存的資料基底集合全都相同，包括事件發生的日期與時間、產生事件的處理序識別碼、事件類型 (資訊、警告與錯誤)，以及事件訊息。

**檔案系統**

每一行記錄到檔案系統的訊息，或是使用串流收到的訊息，皆採用下列格式：

    {Date}  PID[{process ID}] {event type/level} {message}

例如，錯誤事件可能會類似下列範例：

    2014-01-30T16:36:59  PID[3096] Error       Fatal error on the page!

記錄到檔案系統可針對三種可用的方法提供最基本的資訊，亦即僅提供時間、處理序識別碼、事件層級與訊息。

**資料表儲存體**

記錄至資料表儲存體時，會使用其他屬性來協助搜尋儲存在資料表裡的資料，以及更精細的事件資訊。 以下內容 (資料行) 會用於資料表中儲存的每個實體 (列)。

| 屬性名稱 | 值/格式 |
| --- | --- |
| PartitionKey |格式為 yyyyMMddHH 的事件日期/時間 |
| RowKey |可唯一識別此實體的 GUID 值 |
| Timestamp |事件發生的日期與時間 |
| EventTickCount |事件發生的日期與時間 (刻度格式，精準度更高) |
| ApplicationName |Web 應用程式名稱 |
| Level |事件層級 (例如錯誤、警告、資訊) |
| EventId |如果沒有指定<p><p>這個事件的事件識別碼，則預設為 0 |
| InstanceId |發生事件的 Web 應用程式執行個體 |
| Pid |處理序識別碼 |
| Tid |產生事件的執行緒之執行緒識別碼 |
| 訊息 |事件詳細資訊訊息 |

**Blob 儲存體**

登入 Blob 儲存體時，資料會儲存為逗號分隔值 (CSV) 的格式。 其他欄位則會以類似資料表儲存體的作法記錄起來，以提供更精細的事件相關資訊。 CSV 中的每一列會使用以下屬性：

| 屬性名稱 | 值/格式 |
| --- | --- |
| 日期 |事件發生的日期與時間 |
| Level |事件層級 (例如錯誤、警告、資訊) |
| ApplicationName |Web 應用程式名稱 |
| InstanceId |發生事件的 Web 應用程式執行個體 |
| EventTickCount |事件發生的日期與時間 (刻度格式，精準度更高) |
| EventId |如果沒有指定<p><p>這個事件的事件識別碼，則預設為 0 |
| Pid |處理序識別碼 |
| Tid |產生事件的執行緒之執行緒識別碼 |
| 訊息 |事件詳細資訊訊息 |

儲存在 Blob 中的資料類似下列範例：

    date,level,applicationName,instanceId,eventTickCount,eventId,pid,tid,message
    2014-01-30T16:36:52,Error,mywebapp,6ee38a,635266966128818593,0,3096,9,An error occurred

> [!NOTE]
> 記錄的第一行會包含資料行標頭，如此範例所示。
>
>

### <a name="failed-request-traces"></a>失敗要求追蹤
失敗要求追蹤會儲存在名為 **fr######.xml**的 XML 檔案中。 為了讓您輕鬆地檢視記錄資訊，系統會在 XML 檔案所屬的相同目錄中，提供名為 **freb.xsl** 的 XSL 樣式表。 如果您在 Internet Explorer 中開啟其中一個 XML 檔案，Internet Explorer 會使用 XSL 樣式表提供格式化的追蹤資訊顯示，類似下列範例：

![在瀏覽器中檢視的失敗要求](./media/web-sites-enable-diagnostic-log/tws-failedrequestinbrowser.png)

### <a name="detailed-error-logs"></a>詳細錯誤記錄
詳細的錯誤記錄指的是可針對發生的 HTTP 錯誤提供更詳盡資訊的 HTML 文件。 由於它們都是單純的 HTML 文件，因此可以使用網頁瀏覽器來檢視。

### <a name="web-server-logs"></a>Web 伺服器記錄
Web 伺服器記錄使用 [W3C 擴充記錄檔案格式](http://msdn.microsoft.com/library/windows/desktop/aa814385.aspx)來格式化。 此項資訊可透過文字編輯器來讀取，或是運用 [記錄檔剖析器](http://go.microsoft.com/fwlink/?LinkId=246619)(英文) 之類的公用程式來剖析。

> [!NOTE]
> Azure Web 應用程式所產生的記錄並不支援 **s-computername**、**s-ip** 或 **cs-version** 欄位。
>
>

## <a name="nextsteps"></a> 後續步驟
* [如何監視 Web 應用程式](web-sites-monitor.md)
* [在 Visual Studio 中疑難排解 Azure Web App](web-sites-dotnet-troubleshoot-visual-studio.md)
* [在 HDInsight 中分析 Web 應用程式記錄](http://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413)

> [!NOTE]
> 如果您想在註冊 Azure 帳戶前開始使用 Azure App Service，請移至 [試用 App Service](https://azure.microsoft.com/try/app-service/)，即可在 App Service 中立即建立短期入門 Web 應用程式。 不需要信用卡；無需承諾。
>
>
