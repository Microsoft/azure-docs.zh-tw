---
title: Linux 上 Azure App Service 的 Java 語言支援 | Microsoft Docs
description: 使用 Linux 上的 Azure App Service 部署 Java 應用程式開發人員指南。
keywords: Azure App Service, Web 應用程式, Linux, OSS, Java
services: app-service
author: rloutlaw
manager: angerobe
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: article
ms.date: 08/29/2018
ms.author: routlaw
ms.openlocfilehash: 2b2256ef5802160dbaa66e2a098a798fcdc653d2
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064494"
---
# <a name="java-developers-guide-for-app-service-on-linux"></a>Linux 上 App Service 的 Java 開發人員指南

Linux 上的 Azure App Service 可讓 Java 開發人員在全受控 Linux 服務上，快速建置、部署和調整其 Tomcat 或 Java Standard Edition (SE) 封裝的 Web 應用程式。 從命令列或編輯器 (例如 IntelliJ、Eclipse 或 Visual Studio Code)，使用 Maven 外掛程式部署應用程式。

本指南提供 Java 開發人員在 App Service for Linux 中使用的重要概念和指示。 如果您從未使用過 Azure App Service for Linux，則應該先閱讀 [Java 快速入門](quickstart-java.md)。 [App Service Linux 常見問題集](app-service-linux-faq.md)中回答了非 Java 開發特定的一般 App Service for Linux 使用問題。

## <a name="logging-and-debugging-apps"></a>記錄和偵錯應用程式

透過 Azure 入口網站，可以取得每個應用程式的效能報表、流量視覺化和健康狀態檢查。 如需如何存取和使用這些診斷工具的詳細資訊，請參閱 [Azure App Service 診斷概觀](/azure/app-service/app-service-diagnostics)。

### <a name="ssh-console-access"></a>SSH 主控台存取 

具有執行應用程式之 Linux 環境的 SSH 連線能力。 如需透過網頁瀏覽器或本機終端機連線至 Linux 系統的完整指示，請參閱 [Linux 上的 Azure App Service 支援 SSH](/azure/app-service/containers/app-service-linux-ssh-support)。

### <a name="streaming-logs"></a>串流記錄

如需快速偵錯和疑難排解，您可以使用 Azure CLI 將記錄串流處理到主控台。 使用 `az webapp log config` 設定 CLI，以啟用記錄：

```azurecli-interactive
az webapp log config --name ${WEBAPP_NAME} \
 --resource-group ${RESOURCEGROUP_NAME} \
 --web-server-logging filesystem
```

然後使用 `az webapp log tail` 將記錄串流處理到主控台：

```azurecli-interactive
az webapp log tail --name webappname --resource-group myResourceGroup
```

如需詳細資訊，請參閱[使用 Azure CLI 串流處理記錄](/azure/app-service/web-sites-enable-diagnostic-log#streaming-with-azure-command-line-interface)。

### <a name="app-logging"></a>應用程式記錄

透過 Azure 入口網站或 [Azure CLI](/cli/azure/webapp/log#az-webapp-log-config) 啟用[應用程式記錄](/azure/app-service/web-sites-enable-diagnostic-log#enablediag)，設定 App Service 將應用程式的標準主控台輸出和標準主控台錯誤資料流寫入至本機檔案系統或 Azure Blob 儲存體。 設定後的 12 個小時會停用記錄至本機 App Service 檔案系統執行個體。 如果您需要較長的保留期，則請設定應用程式將輸出寫入至 Blob 儲存體容器。

如果您的應用程式使用 [Logback](https://logback.qos.ch/) 或 [Log4j](https://logging.apache.org/log4j) 追蹤，則您可以使用[在 Application Insights 中探索 Java 追蹤記錄](/azure/application-insights/app-insights-java-trace-logs)中的記錄架構設定指示，將這些要檢閱的追蹤轉送至 Azure Application Insights。 

## <a name="customization-and-tuning"></a>自訂和調整

Azure App Service for Linux 支援透過 Azure 入口網站和 CLI 的預設調整和自訂。 請檢閱下列非 Java 特定 Web 應用程式設定的文章：

- [設定 App Service 設定](/azure/app-service/web-sites-configure?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [設定自訂網域](/azure/app-service/app-service-web-tutorial-custom-domain?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [啟用 SSL](/azure/app-service/app-service-web-tutorial-custom-ssl?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- [新增 CDN](/azure/cdn/cdn-add-to-web-app?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)

### <a name="set-java-runtime-options"></a>設定 Java 執行階段選項

若要在 Tomcat 和 Java SE 環境中設定已配置的記憶體或其他 JVM 執行階段選項，請設定 JAVA_OPTS，如下面[應用程式設定](/azure/app-service/web-sites-configure#app-settings)所示。 App Service Linux 會在啟動時將此設定當成環境變數傳遞至 Java 執行階段。

在 Azure 入口網站中，於 Web 應用程式的 [應用程式設定] 下，建立名為 `JAVA_OPTS` 且包含其他設定的新應用程式設定 (例如 `$JAVA_OPTS -Xms512m -Xmx1204m`)。

若要從 Azure App Service Linux Maven 外掛程式設定應用程式設定，請在 Azure 外掛程式區段中新增設定/值標籤。 下列範例設定特定最小和最大 Java 堆積大小：

```xml
<appSettings> 
    <property> 
        <name>JAVA_OPTS</name> 
        <value>$JAVA_OPTS -Xms512m -Xmx1204m</value> 
    </property> 
</appSettings> 
```

執行具有其 App Service 方案中某個部署位置的單一應用程式開發人員，可以使用下列選項：

- B1 和 S1 執行個體：-Xms1024m -Xmx1024m
- B2 和 S2 執行個體：-Xms3072m -Xmx3072m
- B3 和 S3 執行個體：-Xms6144m -Xmx6144m


調整應用程式堆積設定時，請檢閱 App Service 方案詳細資料，並考慮多個應用程式和部署位置需求以尋找最佳的記憶體配置。

### <a name="turn-on-web-sockets"></a>開啟 Web 通訊端

在應用程式的 [應用程式設定] 中，開啟 Azure 入口網站中的 Web 通訊端支援。 您必須重新啟動應用程式，設定才會生效。

搭配使用 Azure CLI 與下列命令，以開啟 Web 通訊端支援：

```azurecli-interactive
az webapp config set -n ${WEBAPP_NAME} -g ${WEBAPP_RESOURCEGROUP_NAME} --web-sockets-enabled true 
```

然後重新啟動您的應用程式：

```azurecli-interactive
az webapp stop -n ${WEBAPP_NAME} -g ${WEBAPP_RESOURCEGROUP_NAME} 
az webapp start -n ${WEBAPP_NAME} -g ${WEBAPP_RESOURCEGROUP_NAME}  
```

### <a name="set-default-character-encoding"></a>設定預設字元編碼 

在 Azure 入口網站中，於 Web 應用程式的 [應用程式設定] 下，建立名為 `JAVA_OPTS` 且值為 `$JAVA_OPTS -Dfile.encoding=UTF-8` 的新應用程式設定。

或者，您可以使用 App Service Maven 外掛程式來設定應用程式設定。 在外掛程式設定中新增設定名稱和值標籤： 

```xml
<appSettings> 
    <property> 
        <name>JAVA_OPTS</name> 
        <value>$JAVA_OPTS -Dfile.encoding=UTF-8</value> 
    </property> 
</appSettings> 
```

## <a name="secure-application"></a>保護應用程式

在 App Service for Linux 中執行之 Java 應用程式會有一組與其他應用程式相同的[安全性最佳做法](/azure/security/security-paas-applications-using-app-services)。 

### <a name="authenticate-users"></a>驗證使用者

使用 [驗證與授權] 選項，設定 Azure 入口網站中的應用程式驗證。 在這裡，您可以使用 Azure Active Directory 或社交登入 (例如 Facebook、Google 或 GitHub) 來啟用驗證。 只有在設定單一驗證提供者時，Azure 入口網站設定才會運作。  如需詳細資訊，請參閱[設定 App Service 應用程式使用 Azure Active Directory 登入](/azure/app-service/app-service-mobile-how-to-configure-active-directory-authentication)，以及其他身分識別提供者的相關文章。

如果您需要啟用多個登入提供者，請遵循[自訂 App Service 驗證](https://docs.microsoft.com/azure/app-service/app-service-authentication-how-to)一文中的指示。

 Spring Boot 開發人員可以使用 [Azure Active Directory Spring Boot 簡易版](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-azure-active-directory?view=azure-java-stable)，以利用熟悉的 Spring Security 註釋和 API 來保護應用程式。

### <a name="configure-tlsssl"></a>設定 TLS/SSL

請遵循[繫結現有自訂 SSL 憑證](/azure/app-service/app-service-web-tutorial-custom-ssl)中的指示，上傳現有的 SSL 憑證，並將其繫結至您應用程式的網域名稱。 根據預設，您的應用程式仍然允許 HTTP 連線；請遵循教學課程中的特定步驟，以強制執行 SSL 和 TLS。

## <a name="tomcat"></a>Tomcat 

### <a name="connecting-to-data-sources"></a>連線至資料來源

>[!NOTE]
> 如果您的應用程式使用 Spring Framework 或 Spring Boot，則可以設定 Spring Data JPA 的資料庫連線資訊作為環境變數 [在應用程式屬性檔案中]。 然後使用[應用程式設定](/azure/app-service/web-sites-configure#app-settings)，以在 Azure 入口網站或 CLI 中定義您應用程式的這些值。

若要使用 Java Database Connectivity (JDBC) 或 Java Persistence API (JPA) 設定 Tomcat 使用受控資料庫連線，請先自訂 Tomcat 在啟動時所讀入的 CATALINA_OPTS 環境變數。 透過 App Service Maven 外掛程式中的應用程式設定這些值：

```xml
<appSettings> 
    <property> 
        <name>CATALINA_OPTS</name> 
        <value>"$CATALINA_OPTS -Dmysqluser=${mysqluser} -Dmysqlpass=${mysqlpass} -DmysqlURL=${mysqlURL}"</value> 
    </property> 
</appSettings> 
```

或者是 Azure 入口網站中的對等 App Service 設定。

接下來，判斷需要資料來源僅供 App Service 方案上執行的一個應用程式還是所有應用程式使用。

針對應用程式層級資料來源： 

1. 請將 `context.xml` 檔案 (若不存在) 新增至 Web 應用程式，並在建置專案時於其中新增 WAR 檔案的 `META-INF` 目錄。

2. 在此檔案中，新增 `Context` 路徑項目，以將資料來源連結至 JNDI 位址。 此

    ```xml
    <Context>
        <Resource
            name="jdbc/mysqldb" type="javax.sql.DataSource"
            url="${mysqlURL}"
            driverClassName="com.mysql.jdbc.Driver"
            username="${mysqluser}" password="${mysqlpass}"
        />
    </Context>
    ```

3. 更新您應用程式的 `web.xml`，以使用您應用程式中的資料來源。

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/mysqldb</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

針對共用伺服器層級資源：

1. 如果您尚未設定，則會使用 SSH 將 `/usr/local/tomcat/conf` 的內容複製至 App Service Linux 執行個體上的 `/home/tomcat`。

2. 將內容新增至 `server.xml`

    ```xml
    <Context>
        <Resource
            name="jdbc/mysqldb" type="javax.sql.DataSource"
            url="${mysqlURL}"
            driverClassName="com.mysql.jdbc.Driver"
            username="${mysqluser}" password="${mysqlpass}"
        />
    </Context>
    ```

3. 更新您應用程式的 `web.xml`，以使用您應用程式中的資料來源。

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/mysqldb</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

4. 請確定 Tomcat classloader 可以使用 JDBC 驅動程式檔案，方法是將它們放在 `/home/tomcat/lib` 目錄中。 若要將這些檔案上傳至 App Service 執行個體，請執行下列步驟：  
    1. 安裝 Azure App Service webpp 延伸模組：
      ```azurecli-interactive
      az extension add –name webapp
      ```
    2. 執行下列 CLI 命令，以建立從本機系統到 App Service 的 SSH 通道：
      ```azurecli-interactive
      az webapp remote-connection create –g [resource group] -n [app name] -p [local port to open]
      ```
    3. 連線本機通道連接埠與 SFTP 用戶端，並將檔案上傳至 `/home/tomcat/lib`。

5. 重新啟動 App Service Linux 應用程式。 Tomcat 會將 `CATALINA_HOME` 重設為 `/home/tomcat`，並使用已更新的設定和類別。

## <a name="docker-containers"></a>Docker 容器

若要在容器中使用於 App Service 中執行的 Azure 支援 Zulu JDK，請確定您應用程式的 `Dockerfile` 使用 [Java App Service Docker 映像存放庫](https://github.com/Azure-App-Service/java)中的映像。

## <a name="runtime-availability-and-statement-of-support"></a>執行階段可用性和支援聲明

App Service for Linux 支援 Java Web 應用程式受控裝載的兩個執行階段：

- [Tomcat servlet 容器](http://tomcat.apache.org/)，適用於執行封裝為 Web 封存 (WAR) 檔案的應用程式。 支援的版本是 8.5 和 9.0。
- Java SE 執行階段環境，適用於執行封裝為 Java 封存 (JAR) 檔案的應用程式。 唯一支援的主要版本是 Java 8。

## <a name="java-runtime-statement-of-support"></a>Java 執行階段支援聲明 

### <a name="jdk-versions-and-maintenance"></a>JDK 版本和維護

Azure 支援的 Java Development Kit (JDK) 是透過 [Azul Systems](https://www.azul.com/) 提供的 [Zulu](https://www.azul.com/products/zulu-and-zulu-enterprise/)。

透過 Azure App Service for Linux 中的新執行階段選項，將提供主要版本更新。 客戶會更新為設定其 App Service 部署的這些新版 Java，以及負責測試並確定主要更新符合其需求。

每年 1 月、4 月、7 月和 10 月都會以季為單位自動修補支援的 JDK。

### <a name="security-updates"></a>安全性更新

只要主要資訊安全漏洞的修補程式和修正程式在 Azul Systems 中變成可用，就予以發行。 在 [NIST Common Vulnerability Scoring System 第 2 版](https://nvd.nist.gov/cvss.cfm)上，「主要」弱點是由基本分數 9.0 或更大值定義。

### <a name="deprecation-and-retirement"></a>取代和淘汰

如果將淘汰支援的 Java 執行階段，則使用受影響執行階段的 Azure 開發人員將會在執行階段淘汰之前至少六個月收到淘汰通知。

### <a name="local-development"></a>本機開發

開發人員可以從 [Azul 的下載網站](https://www.azul.com/downloads/zulu/)下載實際執行版本的 Azul Zulu Enterprise JDK，以進行本機開發。

### <a name="development-support"></a>開發支援

使用[合格 Azure 支援方案](https://azure.microsoft.com/support/plans/)開發 Azure 或 [Azure Stack](https://azure.microsoft.com/overview/azure-stack/)時，可以使用 Azul Zulu Enterprise JDK 的產品支援。

### <a name="runtime-support"></a>執行階段支援

如果開發人員具有[合格的支援方案](https://azure.microsoft.com/support/plans/)則可以透過 Azure 支援[開立問題](/azure/azure-supportability/how-to-create-azure-support-request)，而問題是有關 App Service Linux Java 執行階段。

## <a name="next-steps"></a>後續步驟

瀏覽[適用於 Java 開發人員的 Azure](/java/azure/) 中心，以找出 Azure 快速入門、教學課程和 Java 參考文件。
