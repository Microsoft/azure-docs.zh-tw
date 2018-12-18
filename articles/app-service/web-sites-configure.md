---
title: 在 Azure App Service 中設定 Web 應用程式
description: 如何在 Azure App Service 中設定 Web 應用程式
services: app-service\web
documentationcenter: ''
author: cephalin
manager: erikre
editor: ''
ms.assetid: 9af8a367-7d39-4399-9941-b80cbc5f39a0
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/25/2017
ms.author: cephalin
ms.openlocfilehash: 84bd2019e9586fa008560dba07119323ecb7f02e
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36293711"
---
# <a name="configure-web-apps-in-azure-app-service"></a>在 Azure App Service 中設定 Web 應用程式

本主題說明如何使用 [Azure 入口網站]設定 Web 應用程式。

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="application-settings"></a>應用程式設定
1. 在 [Azure 入口網站]中，開啟 Web 應用程式的刀鋒視窗。
3. 按一下 [應用程式設定] 。

![應用程式設定][configure01]

[ **應用程式設定** ] 刀鋒視窗中的設定會以數種類別來分組。

### <a name="general-settings"></a>一般設定
**Framework 版本**。 如果您的應用程式使用下列任何 Framework，請設定下列選項： 

* **.NET Framework**：設定 .NET framework 版本。 
* **PHP**：設定 PHP 版本，或設為 [關閉] 以停用 PHP。 
* **Java**：選取 Java 版本，或設為 [關閉] 以停用 Java。 使用 [ **Web 容器** ] 選項來選擇 Tomcat 或 Jetty 版本。
* **Python**：選取 Python 版本，或設為 [關閉] 以停用 Python。

在技術上，針對您的應用程式啟用 Java 就會停用 .NET、PHP 與 Python 選項。

<a name="platform"></a>
**平台**。 選取 Web 應用程式是否會在 32 位元或 64 位元環境中執行。 64 位元環境需要 [基本] 或 [標準] 層。 [免費] 與 [共用] 層一律於 32 位元環境中執行。

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

**Web 通訊端**。 設為 [開啟] 以啟用 WebSocket 通訊協定，例如，如果您的 Web 應用程式使用 [ASP.NET SignalR] 或 [socket.io](https://socket.io/)。

<a name="alwayson"></a>
**永遠開啟**。 根據預設，Web 應用程式如果閒置一段時間，就會卸載。 此舉有助於系統保留資源。 在 [基本] 或 [標準] 模式中，您可以啟用 [ **永遠開啟** ]，讓應用程式隨時都能載入。 如果您的應用程式會執行連續的 WebJobs，或執行使用 CRON 運算式觸發的 WebJobs，就應該啟用 [一律開啟]，否則 Web 作業可能無法可靠地執行。

**受控管線版本**。 設定 IIS [管線模式]。 除非您擁有的舊版應用程式需要舊版 IIS，否則請保留 [整合 (預設值)] 的設定。

**HTTP 版本**。 設定為 **2.0** 來啟用 [HTTPS/2](https://wikipedia.org/wiki/HTTP/2) 通訊協定支援。 

> [!NOTE]
> 大部分的新式瀏覽器僅支援透過 TLS 的 HTTP/2 通訊協定，非加密的流量則會繼續使用 HTTP/1.1。 若要確保用戶端瀏覽器會使用 HTTP/2 連線至應用程式，請針對應用程式的自訂網域[購買 App Service 憑證](web-sites-purchase-ssl-web-site.md)或[繫結第三方憑證](app-service-web-tutorial-custom-ssl.md)。

**ARR 同質性**。 在相應放大至多個 VM 執行個體的應用程式中，ARR 同質性 Cookie 可保證用戶端會在工作階段的存留期間路由至相同的執行個體。 若要提升無狀態應用程式的效能，請將此選項設定為 [關閉]。   

**自動交換**。 如果您針對部署位置啟用「自動交換」，當您將更新推送到該位置時，App Service 就會將 Web 應用程式自動交換至生產位置。 如需詳細資訊，請參閱 [將 Azure App Service 中的 Web 應用程式部署至預備位置](web-sites-staged-publishing.md)。

### <a name="debugging"></a>Debugging
**遠端偵錯**。 啟用遠端偵錯。 一經啟用，您就可以使用 Visual Studio 中的遠端偵錯工具，直接連接到您的 Web 應用程式。 遠端偵錯將保持啟用達 48 小時。 

### <a name="app-settings"></a>應用程式設定
這一節包含 Web 應用程式將在啟動時載入的名稱/值組。 

* 如果是 .NET 應用程式，這些設定就會在執行階段插入 .NET 設定 `AppSettings` ，並覆寫現有的設定。 
* PHP、Python、Java 和 Node 應用程式可以在執行階段以環境變數的形式存取這些設定。 系統會為每個應用程式設定建立兩個環境變數，一個變數具有由應用程式設定項目指定的名稱，另一個則具有 APPSETTING_ 前置詞。 這兩個變數都包含相同的值。

應用程式設定在儲存時一律加密 (待用加密)。

### <a name="connection-strings"></a>連接字串
連結資源的連接字串。 

如果是 .NET 應用程式，這些連接字串就會在執行階段插入 .NET 組態 `connectionStrings` 的設定，並覆寫索引鍵相當於所連結資料庫名稱的現有項目。 

如果是 PHP、Python、Java 及 Node 應用程式，您可以在執行階段將這些設定視為環境變數使用，並加上連線類型前置詞。 環境變數首碼如以下所示： 

* SQL Server： `SQLCONNSTR_`
* MySQL： `MYSQLCONNSTR_`
* SQL Database： `SQLAZURECONNSTR_`
* 自訂：`CUSTOMCONNSTR_`

例如，如果 MySql 連接字串命名為 `connectionstring1`，則可透過環境變數 `MYSQLCONNSTR_connectionString1` 加以存取。

連接字串在儲存時一律加密 (待用加密)。

### <a name="default-documents"></a>預設文件
預設文件是顯示於網站根 URL 上的網頁。  系統會使用清單中第一個相符的檔案。 

Web 應用程式可能會使用根據 URL 路由傳送的模組，而非處理靜態內容，若是如此，就不會有這類預設文件。    

### <a name="handler-mappings"></a>處理常式對應
使用此區域可新增自訂指令碼處理器，以處理特定副檔名的要求。 

* **副檔名**。 要處理的副檔名，例如 *.php 或 handler.fcgi。 
* **指令碼處理器路徑**。 指令碼處理器的絕對路徑。 符合該副檔名的檔案要求，將由指令碼處理器來處理。 使用路徑 `D:\home\site\wwwroot` 以指出應用程式的根目錄。
* **其他引數**。 指令碼處理器選用的命令列引數 

### <a name="virtual-applications-and-directories"></a>虛擬應用程式與目錄
若要設定虛擬應用程式與目錄，請指定個別虛擬目錄及其相對於網站根目錄的對應實體路徑。 或者，您可以選取 [ **應用程式** ] 核取方塊，勾選虛擬目錄做為應用程式。

## <a name="enabling-diagnostic-logs"></a>啟用診斷記錄
啟用診斷記錄：

1. 在 Web 應用程式的刀鋒視窗中，按一下 [ **所有設定**]。
2. 按一下 [ **診斷記錄**]。 

從支援記錄功能的 Web 應用程式寫入診斷記錄的選項： 

* **應用程式記錄**。 將應用程式記錄寫入檔案系統。 記錄功能會持續運作 12 小時。 

**層級**。 應用程式記錄功能一經啟用，此選項便會指定要記錄的資訊數量 (錯誤、警告、資訊或詳細資訊)。

**Web 伺服器記錄**。 記錄會儲存為 W3C 延伸的記錄檔格式。 

**詳細的錯誤訊息**。 將詳細的錯誤訊息儲存為 .htm 檔案。 檔案會儲存在 /LogFiles/DetailedErrors 下方。 

**失敗要求的追蹤**。 將失敗的要求記錄於 XML 檔案中。 檔案會儲存在 /LogFiles/W3SVC*xxx*之下，其中 xxx 是唯一識別碼。 此資料夾內含有一個 XSL 檔案和一或多個 XML 檔案。 請務必下載 XSL 檔案，因為 XSL 檔案可提供格式化和篩選 XML 檔案內容的功能。

若要檢視記錄檔，您必須建立 FTP 認證，如下所示：

1. 在 Web 應用程式的刀鋒視窗中，按一下 [ **所有設定**]。
2. 按一下 [ **部署認證**]。
3. 請輸入使用者名稱和密碼。
4. 按一下 [檔案] 。

![設定部署認證][configure03]

完整的 FTP 使用者名稱為 「app\username」，其中 *app* 為您 Web 應用程式的名稱。 username 則列於 Web 應用程式刀鋒視窗的 **Essentials**部分。

![FTP 部署認證][configure02]

## <a name="other-configuration-tasks"></a>其他配置作業
### <a name="ssl"></a>SSL
在 [基本] 或 [標準] 模式中，您可以上傳自訂網域的 SSL 憑證。 如需詳細資訊，請參閱[為 Web 應用程式啟用 HTTPS](app-service-web-tutorial-custom-ssl.md)。 

若要檢視您上傳的憑證，依序按一下  **所有設定** > **自訂網域和 SSL**設定 Web 應用程式。

### <a name="domain-names"></a>網域名稱
新增 Web 應用程式的自訂網域名稱。 如需詳細資訊，請參閱[在 Azure App Service 中設定 Web 應用程式的自訂網域名稱](app-service-web-tutorial-custom-domain.md)。

若要檢視您的網域名稱，依序按一下  **所有設定** > **自訂網域和 SSL**設定 Web 應用程式。

### <a name="deployments"></a>部署
* 設定連續部署。 請參閱 [在 Azure App Service 中使用 Git 來部署 Web Apps](app-service-deploy-local-git.md)。
* 部署位置。 請參閱 [將 Azure App Service 中的 Web Apps 部署至預備環境]。

若要檢視您的部署位置，依序按一下  **所有設定** > **部署位置**設定 Web 應用程式。

### <a name="monitoring"></a>監視
在 [基本] 或 [標準] 模式中，您可以測試 HTTP 或 HTTPS 端點的可用性，最多可測試三個分散的地理位置。 如果 HTTP 回應碼為錯誤 (4xx 或 5xx)，或是當回應時間超過 30 秒時，監視測試即告失敗。 如果所有指定位置上的端點監視測試全都成功，表示該端點可用。 

如需詳細資訊，請參閱 [如何監視 Web 端點狀態]。

> [!NOTE]
> 如果您想在註冊 Azure 帳戶前開始使用 Azure App Service，請移至 [試用 App Service]，即可在 App Service 中立即建立短期入門 Web 應用程式。 不需要信用卡；無需承諾。
> 
> 

## <a name="next-steps"></a>後續步驟
* [在 Azure App Service 中設定自訂網域名稱]
* [針對 Azure App Service 中的 App 啟用 HTTPS]
* [在 Azure App Service 中調整 Web 應用程式規模]
* [在 Azure App Service 中監視 Web 應用程式的基本概念]

<!-- URL List -->

[ASP.NET SignalR]: http://www.asp.net/signalr
[Azure 入口網站]: https://portal.azure.com/
[在 Azure App Service 中設定自訂網域名稱]: ./app-service-web-tutorial-custom-domain.md
[將 Azure App Service 中的 Web Apps 部署至預備環境]: ./web-sites-staged-publishing.md
[針對 Azure App Service 中的 App 啟用 HTTPS]: ./app-service-web-tutorial-custom-ssl.md
[如何監視 Web 端點狀態]: http://go.microsoft.com/fwLink/?LinkID=279906
[在 Azure App Service 中監視 Web 應用程式的基本概念]: ./web-sites-monitor.md
[管線模式]: http://www.iis.net/learn/get-started/introduction-to-iis/introduction-to-iis-architecture#Application
[在 Azure App Service 中調整 Web 應用程式規模]: ./web-sites-scale.md
[試用 App Service]: https://azure.microsoft.com/try/app-service/

<!-- IMG List -->

[configure01]: ./media/web-sites-configure/configure01.png
[configure02]: ./media/web-sites-configure/configure02.png
[configure03]: ./media/web-sites-configure/configure03.png
