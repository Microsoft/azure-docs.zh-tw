---
title: Azure Web 應用程式的應用程式效能常見問題集 | Microsoft Docs
description: 對於 Azure App Service 的 Web 應用程式功能之中的可用性、效能和應用程式問題的常見問題集獲得解答。
services: app-service\web
documentationcenter: ''
author: genlin
manager: cshepard
editor: ''
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: 06fe9f1fd65f70e41d528a513e44e61edb4a7e4f
ms.sourcegitcommit: cfff72e240193b5a802532de12651162c31778b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39307903"
---
# <a name="application-performance-faqs-for-web-apps-in-azure"></a>Azure Web 應用程式的應用程式效能常見問題集

對於 [Azure App Service 的 Web 應用程式功能](https://azure.microsoft.com/services/app-service/web/)的應用程式效能問題，本文提供常見問題集的解答。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-is-my-app-slow"></a>為何我的應用程式變慢？

造成緩慢應用程式效能有許多因素。 如需詳細的疑難排解步驟，請參閱[針對緩慢的 Web 應用程式效能進行疑難排解](app-service-web-troubleshoot-performance-degradation.md)。

## <a name="how-do-i-troubleshoot-a-high-cpu-consumption-scenario"></a>如何針對高 CPU 耗用量案例進行疑難排解？

在某些高 CPU 耗用量案例中，您的應用程式可能真正需要更多運算資源。 在此情況下，請考慮調整為較高的服務層，讓應用程式取得所需的所有資源。 若是其他情況，不正確的迴圈或程式碼撰寫慣例可能會造成高 CPU 耗用量。 深入了解什麼觸發增加的 CPU 耗用量是兩部分的處理序。 首先，建立處理序傾印，然後分析處理序傾印。 如需詳細資訊，請參閱[擷取及分析 Web 應用程式高 CPU 耗用量的傾印檔案](https://blogs.msdn.microsoft.com/asiatech/2016/01/20/how-to-capture-dump-when-intermittent-high-cpu-happens-on-azure-web-app/)。

## <a name="how-do-i-troubleshoot-a-high-memory-consumption-scenario"></a>如何針對高記憶體耗用量案例進行疑難排解？

在某些高記憶體耗用量案例中，您的應用程式可能真正需要更多運算資源。 在此情況下，請考慮調整為較高的服務層，讓應用程式取得所需的所有資源。 若是其他情況，程式碼中的錯誤可能會導致記憶體流失。 程式碼撰寫慣例也可能會增加記憶體耗用量。 深入了解什麼觸發高記憶體耗用量是兩部分的處理序。 首先，建立處理序傾印，然後分析處理序傾印。 Azure 網站擴充功能資源庫的損毀診斷程式可以有效率地執行這兩個步驟。 如需詳細資訊，請參閱[擷取及分析 Web 應用程式間歇高記憶體的傾印檔案](https://blogs.msdn.microsoft.com/asiatech/2016/02/02/how-to-capture-and-analyze-dump-for-intermittent-high-memory-on-azure-web-app/)。

## <a name="how-do-i-automate-app-service-web-apps-by-using-powershell"></a>如何使用 PowerShell 自動化 App Service Web 應用程式？

您可以使用 PowerShell Cmdlet 來管理和維護 App Service Web 應用程式。 在我們的部落格文章[使用 PowerShell 自動化在 Azure App Service 中裝載的 Web 應用程式](https://blogs.msdn.microsoft.com/puneetgupta/2016/03/21/automating-webapps-hosted-in-azure-app-service-through-powershell-arm-way/)中，我們說明如何使用以 Azure Resource Manager 為基礎的 PowerShell Cmdlet 來自動化一般工作。 部落格文章也有各種 Web 應用程式管理工作的範例程式碼。 如需所有 App Service Web 應用程式 Cmdlet 的說明和語法，請參閱 [AzureRM.Websites](https://docs.microsoft.com/powershell/module/azurerm.websites/?view=azurermps-4.0.0)。

## <a name="how-do-i-view-my-web-apps-event-logs"></a>如何檢視我的 Web 應用程式事件記錄？

若要檢視您的 Web 應用程式事件記錄：

1. 登入 [Kudu 網站](https://*yourwebsitename*.scm.azurewebsites.net)。
2. 在功能表中，選取 [偵錯主控台] >  [CMD]。
3. 選取 **LogFiles** 資料夾。
4. 若要檢視事件記錄，請選取 **eventlog.xml** 旁邊的鉛筆圖示。
5. 若要下載記錄，請執行 PowerShell Cmdlet `Save-AzureWebSiteLog -Name webappname`。

## <a name="how-do-i-capture-a-user-mode-memory-dump-of-my-web-app"></a>如何擷取 Web 應用程式的使用者模式記憶體傾印？

若要擷取 Web 應用程式的使用者模式記憶體傾印：

1. 登入 [Kudu 網站](https://*yourwebsitename*.scm.azurewebsites.net)。
2. 選取 [處理序總管] 功能表。
3. 以滑鼠右鍵按一下 **w3wp.exe** 處理序或您的 WebJob 處理序。
4. 選取 [下載記憶體傾印] >  [完整傾印]。

## <a name="how-do-i-view-process-level-info-for-my-web-app"></a>如何檢視 Web 應用程式的處理序層級資訊？

您有兩個選項可以用來檢視 Web 應用程式的處理序層級資訊：

*   在 Azure 入口網站中：
    1. 開啟 Web 應用程式的 [處理序總管]。
    2. 若要查看詳細資料，請選取 **w3wp.exe** 處理序。
*   在 Kudu 主控台中：
    1. 登入 [Kudu 網站](https://*yourwebsitename*.scm.azurewebsites.net)。
    2. 選取 [處理序總管] 功能表。
    3. 針對 **w3wp.exe** 處理序，選取 [屬性]。

## <a name="when-i-browse-to-my-app-i-see-error-403---this-web-app-is-stopped-how-do-i-resolve-this"></a>當我瀏覽至我的應用程式時，我看到「錯誤 403 - 此 Web 應用程式已停止」。 如何解決這個問題？

有三個條件會造成這個錯誤：

* Web 應用程式已達到帳單限制，您的網站已停用。
* Web 應用程式已在入口網站中停止。
* Web 應用程式已達到資源配額限制，限制可能會適用於免費或共用服務方案。

若要查看造成錯誤的原因並且解決問題，請依照 [Web 應用程式：「錯誤 403 – 此 Web 應用程式已停止」](https://blogs.msdn.microsoft.com/waws/2016/01/05/azure-web-apps-error-403-this-web-app-is-stopped/)中的步驟執行。

## <a name="where-can-i-learn-more-about-quotas-and-limits-for-various-app-service-plans"></a>哪裡可以深入了解各種 App Service 方案的配額和限制？

如需配額和限制的詳細資訊，請參閱 [App Service 限制](../azure-subscription-service-limits.md#app-service-limits)。 

## <a name="how-do-i-decrease-the-response-time-for-the-first-request-after-idle-time"></a>如何減少第一個要求閒置時間之後的回應時間？

根據預設，Web 應用程式如果閒置一段時間，就會卸載。 此舉有助於系統保留資源。 缺點是在 Web 應用程式卸載之後對第一個要求的回應時間比較長，以便讓 Web 應用程式載入及啟動服務回應。 在基本和標準服務方案中，您可以開啟 [永遠開啟] 設定以讓應用程式保持載入。 這會排除應用程式閒置之後的較長載入時間。 若要變更 [永遠開啟] 設定：

1. 在 Azure 入口網站中，移至您的 Web 應用程式。
2. 選取 [應用程式設定]。
3. 針對 [永遠開啟]，選取 [開啟]。

## <a name="how-do-i-turn-on-failed-request-tracing"></a>如何開啟失敗要求追蹤？

若要開啟失敗的要求追蹤：

1. 在 Azure 入口網站中，移至您的 Web 應用程式。
3. 選取 [所有設定] >  [診斷記錄]。
4. 針對 [失敗的要求追蹤]，選取 [開啟]。
5. 選取 [ **儲存**]。
6. 在 Web 應用程式刀鋒視窗上，選取 [工具]。
7. 選取 [Visual Studio Online]。
8. 如果設定不是 [開啟]，請選取 [開啟]。
9. 選取 [執行]。
10. 選取 **Web.config**。
11. 在 system.webServer 中，新增這項設定 (擷取特定 URL)：

    ```
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*api*" />
    <add path="*api*">
    <traceAreas>
    <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
12. 若要針對效能緩慢問題進行疑難排解，新增這項設定 (如果擷取要求花費超過 30 秒)：
    ```
    <system.webServer>
    <tracing> <traceFailedRequests>
    <remove path="*" />
    <add path="*">
    <traceAreas> <add provider="ASP" verbosity="Verbose" />
    <add provider="ASPNET" areas="Infrastructure,Module,Page,AppServices" verbosity="Verbose" />
    <add provider="ISAPI Extension" verbosity="Verbose" />
    <add provider="WWW Server" areas="Authentication,Security,Filter,StaticFile,CGI,Compression, Cache,RequestNotifications,Module,FastCGI" verbosity="Verbose" />
    </traceAreas>
    <failureDefinitions timeTaken="00:00:30" statusCodes="200-999" />
    </add> </traceFailedRequests>
    </tracing>
    ```
13. 若要下載失敗的要求追蹤，在[入口網站](https://portal.azure.com)中，移至您的網站。
15. 選取 [工具] >  [Kudu] >  [執行]。
18. 在功能表中，選取 [偵錯主控台] >  [CMD]。
19. 選取 **LogFiles** 資料夾，然後選取名稱開頭為 **W3SVC** 的資料夾。
20. 若要查看 XML 檔案，請選取鉛筆圖示。

## <a name="i-see-the-message-worker-process-requested-recycle-due-to-percent-memory-limit-how-do-i-address-this-issue"></a>我看到訊息「背景工作處理序由於記憶體 % 限制而要求回收」。 如何解決這個問題？

32 位元處理序 (即使是在 64 位元作業系統上) 的記憶體最大可用量是 2 GB。 根據預設，背景工作處理序會在 App Service 中設定為 32 位元 (針對與傳統 Web 應用程式的相容性)。

請考慮切換成 64 位元處理序，以便利用 Web 背景工作角色中可用的額外記憶體。 這會觸發 Web 應用程式重新啟動，請據以排程。

另請注意，64 位元環境需要基本或標準服務方案。 [免費] 與 [共用] 方案一律於 32 位元環境中執行。

如需詳細資訊，請參閱 [在 App Service 中設定 Web 應用程式](web-sites-configure.md)。

## <a name="why-does-my-request-time-out-after-230-seconds"></a>為何我的要求在 230 秒後逾時？

Azure Load Balancer 有預設四分鐘的閒置逾時設定。 這通常對於 Web 要求是合理的回應時間限制。 如果您的 Web 應用程式需要幕後處理，我們建議您使用 Azure WebJobs。 Azure Web 應用程式可以呼叫 WebJobs，並且在幕後處理完成時收到通知。 您可以從使用 WebJobs 的多個方法中選擇，包括佇列和觸發程序。

WebJobs 是設計用來進行幕後處理。 您可以在 WebJob 中進行您想要的任意數量幕後處理。 如需 WebJobs 的詳細資訊，請參閱[使用 WebJobs 執行背景工作](web-sites-create-web-jobs.md)。

## <a name="aspnet-core-applications-that-are-hosted-in-app-service-sometimes-stop-responding-how-do-i-fix-this-issue"></a>裝載於 App Service 中的 ASP.NET Core 應用程式有時候會停止回應。 如何修正此問題？

先前 [Kestrel 版本](https://github.com/aspnet/KestrelHttpServer/issues/1182) 的已知問題可能會造成裝載於 App Service 中的 ASP.NET Core 1.0 應用程式間歇性停止回應。 您也會看到下列訊息：「指定的 CGI 應用程式發生錯誤，伺服器已終止該處理序」。

Kestrel 1.0.2 版已經修正這個問題。 此版本隨附於 ASP.NET Core 1.0.3 更新中。 若要解決此問題，請確定您將應用程式相依性更新為使用 Kestrel 1.0.2。 或者，您可以使用部落格文章 [App Service Web 應用程式中的 ASP.NET Core 1.0 緩慢效能問題](https://blogs.msdn.microsoft.com/waws/2016/12/11/asp-net-core-slow-perf-issues-on-azure-websites)中所述的兩種因應措施其中之一。


## <a name="i-cant-find-my-log-files-in-the-file-structure-of-my-web-app-how-can-i-find-them"></a>我在我的 Web 應用程式檔案結構中找不到我的記錄檔。 如何找到它們？

如果您使用 App Service 的本機快取功能，App Service 執行個體之「記錄檔」和「資料」資料夾的資料夾結構會受到影響。 使用本機快取時，會在儲存體的「記錄檔」和「資料」資料夾中建立子資料夾。 子資料夾會使用「唯一識別碼」+ 時間戳記的命名模式。 每個子資料夾都對應到將要用來執行或已用來執行 Web 應用程式的 VM 執行個體。

若要判斷您是否使用本機快取，請檢查您的 App Service [應用程式設定] 索引標籤。如果使用本機快取，應用程式設定 `WEBSITE_LOCAL_CACHE_OPTION` 會設定為 `Always`。

如果您不是使用本機快取，但是也遇到此問題，請提交支援要求。

## <a name="i-see-the-message-an-attempt-was-made-to-access-a-socket-in-a-way-forbidden-by-its-access-permissions-how-do-i-resolve-this"></a>我看到訊息：「嘗試存取通訊端被拒絕，因為存取權限不足」。 如何解決這個問題？

如果 VM 執行個體上的輸出 TCP 連線都用盡，通常就會發生這個錯誤。 在 App Service 中，會對每個 VM 執行個體可以使用的輸出連線數目上限強制執行限制。 如需詳細資訊，請參閱[跨 VM 的數值限制](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#cross-vm-numerical-limits)。

如果您嘗試從您的應用程式存取本機位址，也可能會發生這個錯誤。 如需詳細資訊，請參閱[本機位址要求](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#local-address-requests)。

如需 Web 應用程式中輸出連線的詳細資訊，請參閱部落格文章有關[到 Azure 網站的輸出連線](http://www.freekpaans.nl/2015/08/starving-outgoing-connections-on-windows-azure-web-sites/)部分。

## <a name="how-do-i-use-visual-studio-to-remote-debug-my-app-service-web-app"></a>如何使用 Visual Studio 遠端偵錯我的 App Service Web 應用程式？

如需示範如何使用 Visual Studio 偵錯 Web 應用程式的詳細逐步解說，請參閱[遠端偵錯您的 App Service Web 應用程式](https://blogs.msdn.microsoft.com/benjaminperkins/2016/09/22/remote-debug-your-azure-app-service-web-app/)。
