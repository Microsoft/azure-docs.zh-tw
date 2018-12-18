---
title: Azure Web 應用程式的開放原始碼技術常見問題集 |Microsoft Docs
description: 獲得有關 Azure App Service 之 Web Apps 功能的開放原始碼技術常見問題解答。
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
ms.openlocfilehash: d65a33dc13d0b91a9ace04dab0be6c37bcd2188f
ms.sourcegitcommit: a62cbb539c056fe9fcd5108d0b63487bd149d5c3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42617619"
---
# <a name="open-source-technologies-faqs-for-web-apps-in-azure"></a>Azure 中的 Web Apps 相關開放原始碼技術常見問題集

針對 [Azure App Service 的 Web Apps 功能](https://azure.microsoft.com/services/app-service/web/)之中的開放原始碼技術相關問題，本文提供常見問題集的解答。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="how-do-i-turn-on-php-logging-to-troubleshoot-php-issues"></a>如何開啟 PHP 記錄進行 PHP 問題的疑難排解？

若要開啟 PHP 記錄：

1. 登入 [Kudu 網站](https://*yourwebsitename*.scm.azurewebsites.net)。
2. 在頂端功能表中，選取 [偵錯主控台] >  > [CMD]。
3. 選取 [網站] 資料夾。
4. 選取 [wwwroot] 資料夾。
5. 選取 **+** 圖示，然後選取 [新增檔案]。
6. 將檔案名稱設定為 **.user.ini**。
7. 選取 **.user.ini** 旁邊的鉛筆圖示。
8. 在檔案中，新增下列程式碼：`log_errors=on`
9. 選取 [ **儲存**]。
10. 選取 **wp-config.php** 旁邊的鉛筆圖示。
11. 將文字變更為下列程式碼：
   ```
   //Enable WP_DEBUG modedefine('WP_DEBUG', true);//Enable debug logging to /wp-content/debug.logdefine('WP_DEBUG_LOG', true);
   //Supress errors and warnings to screendefine('WP_DEBUG_DISPLAY', false);//Supress PHP errors to screenini_set('display_errors', 0);
   ```
12. 在 Azure 入口網站的 Web 應用程式功能表中，重新啟動 Web 應用程式。

如需詳細資訊，請參閱[啟用 WordPress 錯誤記錄檔](https://blogs.msdn.microsoft.com/azureossds/2015/10/09/logging-php-errors-in-wordpress-2/) (英文)。

## <a name="how-do-i-log-python-application-errors-in-apps-that-are-hosted-in-app-service"></a>如何在 App Service 內裝載的應用程式中記錄 Python 應用程式錯誤？
[!INCLUDE [web-sites-python-troubleshooting-wsgi-error-log](../../includes/web-sites-python-troubleshooting-wsgi-error-log.md)]

## <a name="how-do-i-change-the-version-of-the-nodejs-application-that-is-hosted-in-app-service"></a>如何變更 App Service 裝載的 Node.js 應用程式版本？

若要變更 Node.js 應用程式版本，可以使用下列其中一個選項：

*   在 Azure 入口網站中，使用 [應用程式設定]。
    1. 在 Azure 入口網站中，移至您的 Web 應用程式。
    2. 在 [設定] 刀鋒視窗上，選取 [應用程式設定]。
    3. 在 [應用程式設定] 中，您可以包含 WEBSITE_NODE_DEFAULT_VERSION 做為索引鍵，並包含所需的 Node.js 版本做為值。
    4. 移至 [Kudu 主控台](https://*yourwebsitename*.scm.azurewebsites.net)。
    5. 若要檢查 Node.js 版本，請輸入下列命令：  
   ```
   node -v
   ```
*   修改 iisnode.yml 檔案。 在 iisnode.yml 檔案中變更 Node.js 版本只會設定 iisnode 使用的執行階段環境。 Kudu cmd 和其他命令仍然使用在 Azure 入口網站**應用程式設定**中所設的 Node.js 版本。

    若要手動設定 iisnode.yml，請在應用程式根資料夾中建立 iisnode.yml 檔案。 在檔案中，包含下列一行：
   ```
   nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
   ```
   
*   在原始檔控制部署期間使用 package.json，以便設定 iisnode.yml 檔案。
    Azure 原始檔控制部署程序包含下列步驟：
    1. 將內容移至 Azure Web 應用程式。
    2. 如果 Web 應用程式根資料夾中沒有部署指令碼 (deploy.cmd、.deployment 檔案)，則建立預設部署指令碼。
    3. 如果您在 package.json 檔案 > 引擎 `"engines": {"node": "5.9.1","npm": "3.7.3"}` 中提到 Node.js 版本，則執行部署指令碼來建立 iisnode.yml 檔案。
    4. iisnode.yml 檔案具有下列一行程式碼：
        ```
        nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
        ```

## <a name="i-see-the-message-error-establishing-a-database-connection-in-my-wordpress-app-thats-hosted-in-app-service-how-do-i-troubleshoot-this"></a>在 App Service 裝載的 WordPress 應用程式中，顯示「建立資料庫連接時發生錯誤」訊息。 我該如何進行疑難排解？

如果您在 Azure WordPress 應用程式中看到這個錯誤，而要啟用 php_errors.log 和 debug.log，請完成[啟用 WordPress 錯誤記錄檔](https://blogs.msdn.microsoft.com/azureossds/2015/10/09/logging-php-errors-in-wordpress-2/) (英文) 中詳述的步驟。

在記錄檔已啟用時重現錯誤，然後檢查記錄檔查看是否即將用盡連線：
```
[09-Oct-2015 00:03:13 UTC] PHP Warning: mysqli_real_connect(): (HY000/1226): User ‘abcdefghijk79' has exceeded the ‘max_user_connections’ resource (current value: 4) in D:\home\site\wwwroot\wp-includes\wp-db.php on line 1454
```

如果您看見 debug.log 或 php_errors.log 檔案中出現此錯誤，表示應用程式已超出連線數。 如果正在 ClearDB 上裝載，請確認[服務方案](https://www.cleardb.com/pricing.view) (英文) 中可用的連線數。

## <a name="how-do-i-debug-a-nodejs-app-thats-hosted-in-app-service"></a>如何對於 App Service 中裝載的 Node.js 應用程式進行偵錯？

1.  移至 [Kudu 主控台](https://*yourwebsitename*.scm.azurewebsites.net/DebugConsole)。
2.  移至應用程式記錄檔資料夾 (D:\home\LogFiles\Application)。
3.  在 Logging_errors.txt 檔案中檢查內容。

## <a name="how-do-i-install-native-python-modules-in-an-app-service-web-app-or-api-app"></a>如何在 App Service Web 應用程式或 API 應用程式中安裝原生 Python 模組？

某些封裝可能不會使用 Azure 中的 pip 進行安裝。 封裝可能並未出現在 Python 封裝索引，或可能必須使用編譯器 (執行 App Service Web 應用程式的電腦沒有編譯器可用)。 如需在 App Service Web 應用程式及 API 應用程式中安裝原生模組的相關資訊，請參閱[在 App Service 中安裝 Python 模組](https://blogs.msdn.microsoft.com/azureossds/2015/06/29/install-native-python-modules-on-azure-web-apps-api-apps/) (英文)。

## <a name="how-do-i-deploy-a-django-app-to-app-service-by-using-git-and-the-new-version-of-python"></a>如何使用 Git 和新版 Python 將 Django 應用程式部署到 App Service？

如需安裝 Django 的相關資訊，請參閱[將 Django 應用程式部署至 App Service](https://blogs.msdn.microsoft.com/azureossds/2016/08/25/deploying-django-app-to-azure-app-services-using-git-and-new-version-of-python/) (英文)。

## <a name="where-are-the-tomcat-log-files-located"></a>Tomcat 記錄檔位於何處？

對於 Azure Marketplace 和自訂部署：

* 資料夾位置：D:\home\site\wwwroot\bin\apache-tomcat-8.0.33\logs
* 感興趣區域：
    * catalina.*yyyy-mm-dd*.log
    * host-manager.*yyyy-mm-dd*.log
    * localhost.*yyyy-mm-dd*.log
    * manager.*yyyy-mm-dd*.log
    * site_access_log.*yyyy-mm-dd*.log


對於入口網站**應用程式設定**部署：

* 資料夾位置：D:\home\LogFiles
* 感興趣區域：
    * catalina.*yyyy-mm-dd*.log
    * host-manager.*yyyy-mm-dd*.log
    * localhost.*yyyy-mm-dd*.log
    * manager.*yyyy-mm-dd*.log
    * site_access_log.*yyyy-mm-dd*.log

## <a name="how-do-i-troubleshoot-jdbc-driver-connection-errors"></a>如何進行 JDBC 驅動程式連線錯誤的疑難排解？

您可能會在 Tomcat 記錄檔中看見下列訊息：

```
The web application[ROOT] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak,the JDBC Driver has been forcibly unregistered
```

若要解決此錯誤：

1. 從 app/lib 資料夾移除 sqljdbc*.jar 檔案。
2. 如果您使用自訂的 Tomcat 或 Azure Marketplace Tomcat Web 伺服器，請將此 .jar 檔案複製到 Tomcat lib 資料夾。
3. 如果您從 Azure 入口網站啟用 Java (選取 **Java 1.8** > **Tomcat 伺服器**)，請將 sqljdbc.* jar 檔案複製到與應用程式並列的資料夾中。 然後，將下列 classpath 設定新增到 web.config 檔案：

    ```
    <httpPlatform>
    <environmentVariables>
    <environmentVariablename ="JAVA_OPTS" value=" -Djava.net.preferIPv4Stack=true
    -Xms128M -classpath %CLASSPATH%;[Path to the sqljdbc*.jarfile]" />
    </environmentVariables>
    </httpPlatform>
    ```

## <a name="why-do-i-see-errors-when-i-attempt-to-copy-live-log-files"></a>為什麼我嘗試複製即時記錄檔時顯示錯誤？

如果您嘗試複製 Java 應用程式 (例如 Tomcat) 的即時記錄檔，可能會顯示此 FTP 錯誤：

```
Error transferring file [filename] Copying files from remote side failed.
    
The process cannot access the file because it is being used by another process.
```

端視 FTP 用戶端而定，錯誤訊息可能有所不同。

所有的 Java 應用程式都有這個鎖定問題。 只有 Kudu 支援在應用程式執行時下載這個檔案。

停止應用程式即可透過 FTP 存取這些檔案。

另一個解決方法是撰寫依排程執行的 WebJob，並將這些檔案複製到不同的目錄。 如需範例專案，請參閱 [CopyLogsJob](https://github.com/kamilsykora/CopyLogsJob) 專案。

## <a name="where-do-i-find-the-log-files-for-jetty"></a>在哪裡可找到 Jetty 的記錄檔？

對於 Marketplace 和自訂部署，記錄檔位於 D:\home\site\wwwroot\bin\jetty-distribution-9.1.2.v20140210\logs 資料夾。 請注意，資料夾位置取決於您使用的 Jetty 版本。 例如，此處提供的路徑是 Jetty 9.1.2 的路徑。 尋找 jetty_*YYYY_MM_DD*.stderrout.log。

對於入口網站應用程式設定部署，記錄檔位於 D:\home\LogFiles。 尋找 jetty_*YYYY_MM_DD*.stderrout.log

## <a name="can-i-send-email-from-my-azure-web-app"></a>我能否從 Azure Web 應用程式傳送電子郵件？

App Service 沒有內建的電子郵件功能。 如需從應用程式傳送電子郵件的一些可行替代方案，請參閱本篇 [Stack Overflow 討論](http://stackoverflow.com/questions/17666161/sending-email-from-azure) (英文)。

## <a name="why-does-my-wordpress-site-redirect-to-another-url"></a>為什麼我的 WordPress 網站重新導向至其他 URL？

如果您最近移轉至 Azure，WordPress 可能會重新導向至舊網域 URL。 這是 MySQL 資料庫中的設定所致。

WordPress Buddy+ 是 Azure 網站擴充功能，可讓您直接在資料庫中更新重新導向 URL。 如需使用 WordPress Buddy+ 的詳細資訊，請參閱 [WordPress 工具以及使用 WordPress Buddy+ 進行 MySQL 移轉](https://blogs.msdn.microsoft.com/azureossds/2016/12/21/wordpress-tools-and-mysql-migration-with-wordpress-buddy/) (英文)。

或者，如果您想要使用 SQL 查詢或 PHPMyAdmin 手動更新重新導向 URL，請參閱 [WordPress：重新導向至錯誤的 URL](https://blogs.msdn.microsoft.com/azureossds/2016/07/12/wordpress-redirecting-to-wrong-url/) (英文)。

## <a name="how-do-i-change-my-wordpress-sign-in-password"></a>如何變更我的 WordPress 登入密碼？

如果您忘記您的 WordPress 登入密碼，可以使用 WordPress Buddy+ 更新該密碼。 若要重設密碼，請安裝 WordPress Buddy+ Azure 網站擴充功能，然後完成 [WordPress 工具以及使用 WordPress Buddy+ 進行 MySQL 移轉](https://blogs.msdn.microsoft.com/azureossds/2016/12/21/wordpress-tools-and-mysql-migration-with-wordpress-buddy/) (英文) 中所述的步驟。

## <a name="i-cant-sign-in-to-wordpress-how-do-i-resolve-this"></a>我無法登入 WordPress。 如何解決這個問題？

最近安裝外掛程式之後，如果您發現遭鎖定而無法進入 WordPress，則表示外掛程式可能有問題。 WordPress Buddy+ 是 Azure 網站擴充功能，可協助您停用 WordPress 中的外掛程式。 如需詳細資訊，請參閱 [WordPress 工具以及使用 WordPress Buddy+ 進行 MySQL 移轉](https://blogs.msdn.microsoft.com/azureossds/2016/12/21/wordpress-tools-and-mysql-migration-with-wordpress-buddy/) (英文)。

## <a name="how-do-i-migrate-my-wordpress-database"></a>我要如何移轉我的 WordPress 資料庫？

對於移轉連線到 WordPress 網站的 MySQL 資料庫，您有多個選項：

* 開發人員：使用[命令提示字元或 PHPMyAdmin](https://blogs.msdn.microsoft.com/azureossds/2016/03/02/migrating-data-between-mysql-databases-using-kudu-console-azure-app-service/)
* 非開發人員：使用 [WordPress Buddy+](https://blogs.msdn.microsoft.com/azureossds/2016/12/21/wordpress-tools-and-mysql-migration-with-wordpress-buddy/)

## <a name="how-do-i-help-make-wordpress-more-secure"></a>如何使 WordPress 更安全？

若要了解 WordPress 的安全性最佳做法，請參閱 [Azure 的 WordPress 安全性最佳做法](https://blogs.msdn.microsoft.com/azureossds/2016/12/26/best-practices-for-wordpress-security-on-azure/) (英文)。

## <a name="i-am-trying-to-use-phpmyadmin-and-i-see-the-message-access-denied-how-do-i-resolve-this"></a>我嘗試使用 PHPMyAdmin，但顯示「拒絕存取」訊息。 如何解決這個問題？

如果應用程式內 MySQL 的功能並未在此 App Service 執行個體中執行，您可能會遇到此問題。 若要解決此問題，請嘗試存取您的網站。 這將啟動所需的程序，包括應用程式內 MySQL 程序。 若要確認應用程式內 MySQL 正在執行，請在處理序總管確認 mysqld.exe 已列於處理序中。

確定應用程式內 MySQL 執行之後，嘗試使用 PHPMyAdmin。

## <a name="i-get-an-http-403-error-when-i-try-to-import-or-export-my-mysql-in-app-database-by-using-phpmyadmin-how-do-i-resolve-this"></a>我嘗試使用 PHPMyadmin 匯入或匯出應用程式內 MySQL 資料庫時，出現 HTTP 403 錯誤。 如何解決這個問題？

如果您使用舊版 Chrome，您可能會遇到已知的錯誤。 若要解決此問題，請升級為新版 Chrome。 也請嘗試使用無此問題的不同瀏覽器，例如 Internet Explorer 或 Edge。
