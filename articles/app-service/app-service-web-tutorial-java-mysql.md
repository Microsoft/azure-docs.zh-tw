---
title: 在 Azure 中建置 Java 和 MySQL Web 應用程式
description: 了解如何取得連線至在 Azure Appservice 中運作的 Azure MySQL 資料庫服務之 Java 應用程式。
services: app-service\web
documentationcenter: Java
author: bbenz
manager: jeffsand
editor: jasonwhowell
ms.assetid: ''
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: tutorial
ms.date: 05/22/2017
ms.author: bbenz
ms.custom: mvc
ms.openlocfilehash: 0baab86c0cb76bfeecb30cdb62c968a476e402b9
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44296766"
---
# <a name="tutorial-build-a-java-and-mysql-web-app-in-azure"></a>教學課程：在 Azure 中建置 Java 和 MySQL Web 應用程式

> [!NOTE]
> 本文會將應用程式部署至 Windows 上的 App Service。 若要在 Linux 上部署至 App Service，請參閱[將容器化的 Spring Boot 應用程式部署至 Azure](/java/azure/spring-framework/deploy-containerized-spring-boot-java-app-with-maven-plugin)。
>

本教學課程示範如何在 Azure 中建立 Java Web 應用程式，並將它連線到 MySQL 資料庫。 當您完成後，在 [Azure App Service Web Apps](app-service-web-overview.md) 中執行的 [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview) 會有一個儲存資料的 [Spring Boot](https://projects.spring.io/spring-boot/)應用程式。

![在 Azure Appservice 中執行的 Java 應用程式](./media/app-service-web-tutorial-java-mysql/appservice-web-app.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 MySQL 資料庫
> * 將範例應用程式連線至資料庫
> * 將應用程式部署至 Azure
> * 更新和重新部署應用程式
> * 來自 Azure 的串流診斷記錄
> * 在 Azure 入口網站中監視應用程式

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

1. [下載並安裝 Git](https://git-scm.com/)
1. [下載並安裝 Java 7 JDK 或更高版本](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
1. [下載、安裝並啟動 MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html) 

## <a name="prepare-local-mysql"></a>準備本機 MySQL 

在此步驟中，您可以在本機 MySQL 伺服器中建立資料庫，供您在本機電腦上測試應用程式。

### <a name="connect-to-mysql-server"></a>連線至 MySQL 伺服器

在終端機視窗中，連線到您的本機 MySQL 伺服器。 您可使用這個終端機視窗來執行本教學課程中的所有命令。

```bash
mysql -u root -p
```

如果系統提示您輸入密碼，請輸入 `root` 帳戶的密碼。 如果您不記得根帳戶密碼，請參閱 [MySQL︰如何重設根密碼](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)。

如果您的命令執行成功，表示您的 MySQL 伺服器已經在執行中。 如果沒有，請遵循 [MySQL 後續安裝步驟](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html)，確定您已啟動本機 MySQL 伺服器。

### <a name="create-a-database"></a>建立資料庫 

在 `mysql` 提示中，建立待辦事項的資料庫和資料表。

```sql
CREATE DATABASE tododb;
```

輸入 `quit` 即可結束您的伺服器連線。

```sql
quit
```

## <a name="create-and-run-the-sample-app"></a>建立和執行範例應用程式 

在此步驟中，您可以複製範例 Spring Boot 應用程式、將它設定為使用本機 MySQL 資料庫，並在您的電腦上執行。 

### <a name="clone-the-sample"></a>複製範例

在終端機視窗中，瀏覽到工作目錄並複製範例存放庫。 

```bash
git clone https://github.com/azure-samples/mysql-spring-boot-todo
```

### <a name="configure-the-app-to-use-the-mysql-database"></a>將應用程式設定為使用 MySQL 資料庫

更新 `spring.datasource.password` 以及 *spring-boot-mysql-todo/src/main/resources/application.properties* 中的值，以用來開啟 MySQL 命令提示字元相同的根密碼加以取代：

```
spring.datasource.password=mysqlpass
```

### <a name="build-and-run-the-sample"></a>建置並執行範例

使用存放庫中包含的 Maven 包裝函式建置並執行範例：

```bash
cd spring-boot-mysql-todo
mvnw package spring-boot:run
```

開啟瀏覽器至 `http://localhost:8080`，查看範例如何運作。 在清單中新增工作時，請使用 MySQL 命令提示字元中的下列 SQL 命令，以檢視 MySQL 中儲存的資料。

```SQL
use testdb;
select * from todo_item;
```

按下終端機中的 `Ctrl`+`C` 以停止應用程式。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-an-azure-mysql-database"></a>建立 Azure MySQL 資料庫

在此步驟中，您會使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 建立 [Azure Database for MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-cli.md) 執行個體。 也會設定範例應用程式，以便稍後可以在教學課程中使用此資料庫。

### <a name="create-a-resource-group"></a>建立資源群組

使用 [`az group create`](/cli/azure/group#az-group-create) 命令來建立[資源群組](../azure-resource-manager/resource-group-overview.md)。 Azure 資源群組是一個邏輯容器，可在其中部署與管理例如 Web 應用程式、資料庫和儲存體帳戶等相關資源。 

下列範例會在北歐區域中建立一個資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location "North Europe"
```    

若要查看您可用於 `--location` 的可能值，請使用 [`az appservice list-locations`](/cli/azure/appservice#list-locations) 命令。

### <a name="create-a-mysql-server"></a>建立 MySQL 伺服器

在 Cloud Shell 中，使用 [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create) 命令在適用於 MySQL 的 Azure 資料庫中建立伺服器。

在下列命令中，使用唯一的伺服器名稱取代 \<mysql_server_name> 預留位置，使用使用者名稱取代 \<admin_user> 預留位置，並使用密碼取代 \<admin_password> 預留位置。 這個伺服器名稱會用來作為 PostgreSQL 端點 (`https://<mysql_server_name>.mysql.database.azure.com`) 的一部分，所以在 Azure 的所有伺服器中必須是唯一的名稱。

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql_server_name>--location "West Europe" --admin-user <admin_user> --admin-password <server_admin_password> --sku-name GP_Gen4_2
```

> [!NOTE]
> 由於在本教學課程有數個認證需要談討，以避免產生混淆，`--admin-user` 和 `--admin-password` 會設為空值。 在生產環境中，為 Azure MySQL 伺服器選擇良好的使用者名稱和密碼時，請遵循安全性最佳做法。
>
>

建立 MySQL 伺服器後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "location": "westeurope",
  "name": "<mysql_server_name>",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "additionalProperties": {},
    "capacity": 2,
    "family": "Gen4",
    "name": "GP_Gen4_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  ...   +  
  -  < Output has been truncated for readability >
}
```

### <a name="configure-server-firewall"></a>設定伺服器防火牆

在 Cloud Shell 中，使用 [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-create) 命令，建立 MySQL 伺服器的防火牆規則來允許用戶端連線。 當起始 IP 和結束 IP 都設為 0.0.0.0 時，防火牆只會為其他 Azure 資源開啟。 

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql_server_name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP] 
> [僅使用您的應用程式所用的輸出 IP 位址](app-service-ip-addresses.md#find-outbound-ips)，讓您的防火牆規則更具限制性。
>

## <a name="configure-the-azure-mysql-database"></a>設定 Azure MySQL 資料庫

在本機終端機視窗中，連線至 Azure 中的 MySQL 伺服器。 使用您先前為 _&lt;mysql_server_name>_ 指定的值。 當系統提示您輸入密碼時，請使用您在 Azure 中建立資料庫時指定的密碼。

```bash
mysql -u <admin_user>@<mysql_server_name> -h <mysql_server_name>.mysql.database.azure.com -P 3306 -p
```

### <a name="create-a-database"></a>建立資料庫 

在 `mysql` 提示中，建立待辦事項的資料庫和資料表。

```sql
CREATE DATABASE tododb;
```

### <a name="create-a-user-with-permissions"></a>建立具有權限的使用者

建立資料庫使用者，並將 `tododb` 資料庫中所有的權限授權給它。 將 `<Javaapp_user>` 和 `<Javaapp_password>` 預留位置取代您自己的唯一應用程式名稱。

```sql
CREATE USER '<Javaapp_user>' IDENTIFIED BY '<Javaapp_password>'; 
GRANT ALL PRIVILEGES ON tododb.* TO '<Javaapp_user>';
```

輸入 `quit` 即可結束您的伺服器連線。

```sql
quit
```

## <a name="deploy-the-sample-to-azure-app-service"></a>將範例部署到 Azure App Service

使用 [`az appservice plan create`](/cli/azure/appservice/plan#az-appservice-plan-create) CLI 命令來建立具有**免費**定價層的 Azure App Service 方案。 Appservice 方案會定義用來託管應用程式的實體資源。 所有指派給 Appservice 方案的應用程式都會共用這些資源，從而讓您節省託管多個應用程式的成本。 

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

方案準備妥當時，Azure CLI 會顯示類似下列範例的輸出：

```json
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "North Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "North Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  ...
  < Output has been truncated for readability >
} 
``` 

### <a name="create-an-azure-web-app"></a>建立 Azure Web 應用程式

在 Cloud Shell 中，使用 [`az webapp create`](/cli/azure/webapp#az-webapp-create) CLI 命令，在 `myAppServicePlan` App Service 方案中建立 Web 應用程式定義。 Web 應用程式定義會提供一個 URL 以存取您的應用程式，並設定數個選項將您的程式碼部署至 Azure。 

```azurecli-interactive
az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan
```

使用您自己的唯一應用程式名稱來取代 `<app_name>` 預留位置。 這個唯一名稱會是 Web 應用程式預設網域名稱的一部分，因此，這個名稱在 Azure 的所有應用程式中必須是唯一的。 您可以先將自訂的網域名稱項目對應至 Web 應用程式，再將它公開給使用者。

Web 應用程式定義備妥之後，Azure CLI 會顯示類似下列範例的資訊： 

```json 
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
   ...
  < Output has been truncated for readability >
}
```

### <a name="configure-java"></a>設定 Java 

在 Cloud Shell 中，使用 [`az webapp config set`](/cli/azure/webapp/config#az-webapp-config-set) 命令來設定您應用程式需要的 Java 執行階段組態。

下列命令會將 Web 應用程式設定為在最新的 Java 8 JDK 和 [Apache Tomcat](http://tomcat.apache.org/) 8.0 上執行。

```azurecli-interactive
az webapp config set --name <app_name> --resource-group myResourceGroup --java-version 1.8 --java-container Tomcat --java-container-version 8.0
```

### <a name="configure-the-app-to-use-the-azure-sql-database"></a>將應用程式設定為使用 Azure SQL 資料庫

請先將 Web 應用程式上的應用程式設定設為使用在 Azure 中建立的 Azure MySQL 資料庫，再執行範例應用程式。 這些屬性會公開至 Web 應用程式作為環境變數，並覆寫已封裝之 Web 應用程式中 application.properties 所設定的值。 

在 Cloud Shell 中，使用 CLI 中的 [`az webapp config appsettings`](https://docs.microsoft.com/cli/azure/webapp/config/appsettings) 來設定應用程式設定：

```azurecli-interactive
az webapp config appsettings set --settings SPRING_DATASOURCE_URL="jdbc:mysql://<mysql_server_name>.mysql.database.azure.com:3306/tododb?verifyServerCertificate=true&useSSL=true&requireSSL=false" --resource-group myResourceGroup --name <app_name>
```

```azurecli-interactive
az webapp config appsettings set --settings SPRING_DATASOURCE_USERNAME=Javaapp_user@mysql_server_name --resource-group myResourceGroup --name <app_name>
```

```azurecli-interactive
az webapp config appsettings set --settings SPRING_DATASOURCE_PASSWORD=Javaapp_password --resource-group myResourceGroup --name <app_name>
```

### <a name="get-ftp-deployment-credentials"></a>取得 FTP 部署認證 
您可以使用各種方式來將應用程式部署至 Azure Appservice，包括 FTP、本機 Git、GitHub、Azure DevOps 和 BitBucket。 在此範例中，使用 FTP 將先前在您本機電腦上建置的 .WAR 檔案部署至 Azure App Service。

若要判斷哪些認證要在 ftp 命令中傳遞至 Web 應用程式，請在 Cloud Shell 中使用 [`az appservice web deployment list-publishing-profiles`](https://docs.microsoft.com/cli/azure/webapp/deployment#az-appservice-web-deployment-list-publishing-profiles) 命令： 

```azurecli-interactive
az webapp deployment list-publishing-profiles --name <app_name> --resource-group myResourceGroup --query "[?publishMethod=='FTP'].{URL:publishUrl, Username:userName,Password:userPWD}" --output json
```

```JSON
[
  {
    "Password": "aBcDeFgHiJkLmNoPqRsTuVwXyZ",
    "URL": "ftp://waws-prod-blu-069.ftp.azurewebsites.windows.net/site/wwwroot",
    "Username": "app_name\\$app_name"
  }
]
```

### <a name="upload-the-app-using-ftp"></a>使用 FTP 上傳應用程式

使用您喜愛的 FTP 工具將 .WAR 檔案部署至 */site/wwwroot/webapps* 資料夾；，該資料夾位於先前命令中 `URL` 欄位的伺服器位址上。 移除現有的預設 (ROOT) 應用程式目錄，並以先前在教學課程中建置的 .WAR 檔案取代現有的 ROOT.war。

```bash
ftp waws-prod-blu-069.ftp.azurewebsites.windows.net
Connected to waws-prod-blu-069.drip.azurewebsites.windows.net.
220 Microsoft FTP Service
Name (waws-prod-blu-069.ftp.azurewebsites.windows.net:raisa): app_name\$app_name
331 Password required
Password:
cd /site/wwwroot/webapps
mdelete -i ROOT/*
rmdir ROOT/
put target/TodoDemo-0.0.1-SNAPSHOT.war ROOT.war
```

### <a name="test-the-web-app"></a>測試 Web 應用程式

瀏覽至 `http://<app_name>.azurewebsites.net/` 並將幾項工作新增至清單。 

![在 Azure Appservice 中執行的 Java 應用程式](./media/app-service-web-tutorial-java-mysql/appservice-web-app.png)

**恭喜！** 您正在 Azure App Service 中執行資料驅動的 Java 應用程式。

## <a name="update-the-app-and-redeploy"></a>更新應用程式並重新部署

更新應用程式，以在待辦事項清單中包含額外的資料行，用來記錄項目的建立日期。 Spring Boot 會在資料模型變更時為您更新資料庫結構描述，無需更改您現有的資料庫記錄。

1. 在您的本機系統中，開啟 *src/main/java/com/example/fabrikam/TodoItem.java* 並在類別中新增下列匯入項目：   

    ```java
    import java.text.SimpleDateFormat;
    import java.util.Calendar;
    ```

2. 在 *src/main/java/com/example/fabrikam/TodoItem.java* 中新增 `String` 屬性 `timeCreated`，並以建立物件時使用的時間戳記加以起始。 在編輯此檔案時，為新的 `timeCreated` 屬性新增 getter/setter。

    ```java
    private String name;
    private boolean complete;
    private String timeCreated;
    ...

    public TodoItem(String category, String name) {
       this.category = category;
       this.name = name;
       this.complete = false;
       this.timeCreated = new SimpleDateFormat("MMMM dd, YYYY").format(Calendar.getInstance().getTime());
    }
    ...
    public void setTimeCreated(String timeCreated) {
       this.timeCreated = timeCreated;
    }

    public String getTimeCreated() {
        return timeCreated;
    }
    ```

3. 以 `updateTodo` 方法中的一行更新 *src/main/java/com/example/fabrikam/TodoDemoController.java*，以設定時間戳記：

    ```java
    item.setComplete(requestItem.isComplete());
    item.setId(requestItem.getId());
    item.setTimeCreated(requestItem.getTimeCreated());
    repository.save(item);
    ```

4. 在 `Thymeleaf` 範本中加入新欄位的支援。 以適用於時間戳記的新資料表標頭來更新 *src/main/resources/templates/index.html*，並以新欄位來顯示每個資料表資料列中的時間戳記值。

    ```html
    <th>Name</th>
    <th>Category</th>
    <th>Time Created</th>
    <th>Complete</th>
    ...
    <td th:text="${item.category}">item_category</td><input type="hidden" th:field="*{todoList[__${i.index}__].category}"/>
    <td th:text="${item.timeCreated}">item_time_created</td><input type="hidden" th:field="*{todoList[__${i.index}__].timeCreated}"/>
    <td><input type="checkbox" th:checked="${item.complete} == true" th:field="*{todoList[__${i.index}__].complete}"/></td>
    ```

5. 重新建置應用程式：

    ```bash
    mvnw clean package 
    ```

6. 一如往常透過 FTP 處理已更新的 .WAR、移除現有的 *site/wwwroot/webapps/ROOT* 目錄和 *ROOT.war*，接著上傳更新的 .WAR 檔案以取代 ROOT.war。 

當您重新整理應用程式時，會看到**建立時間**的資料行。 當您新增工作時，應用程式會自動植入時間戳記。 您現有的工作會維持不變，即使基礎資料模型已變更，仍可搭配使用該應用程式。 

![搭配新資料行的 Java 應用程式更新](./media/app-service-web-tutorial-java-mysql/appservice-updates-java.png)
      
## <a name="stream-diagnostic-logs"></a>資料流診斷記錄 

在 Azure App Service 中執行您的 Java 應用程式時，可以將主控台記錄直接傳送至終端機。 這樣一來，您就能取得相同的診斷訊息，以協助您偵錯應用程式錯誤。

若要開始記錄資料流，請在 Cloud Shell 中使用 [`az webapp log tail`](/cli/azure/webapp/log?view=azure-cli-latest#az-webapp-log-tail) 命令。

```azurecli-interactive 
az webapp log tail --name <app_name> --resource-group myResourceGroup 
``` 

## <a name="manage-your-azure-web-app"></a>管理您的 Azure Web 應用程式

請移至 [Azure 入口網站](https://portal.azure.com)，以查看您所建立的 Web 應用程式。

按一下左側功能表中的 [App Service]，然後按一下 Azure Web 應用程式的名稱。

![入口網站瀏覽至 Azure Web 應用程式](./media/app-service-web-tutorial-java-mysql/access-portal.png)

根據預設，Web 應用程式頁面會顯示 [概觀] 頁面。 此頁面可讓您檢視應用程式的執行方式。 您也可以在這裡執行像是停止、啟動、重新啟動及刪除等管理工作。 分頁左側的索引標籤會顯示您可開啟的各種設定分頁。

![Azure 入口網站中的 App Service 頁面](./media/app-service-web-tutorial-java-mysql/web-app-blade.png)

頁面中的索引標籤會顯示您可以新增至 Web 應用程式的許多強大功能。 下表提供幾個可能性︰
* 對應自訂 DNS 名稱
* 繫結自訂 SSL 憑證
* 設定連續部署
* 相應增加和相應放大
* 新增使用者驗證

## <a name="clean-up-resources"></a>清除資源

如果您不需要這些資源來進行其他教學課程 (請參閱[後續步驟](#next))，您可以在 Cloud Shell 中執行下列命令來將這些資源刪除︰ 
  
```azurecli-interactive
az group delete --name myResourceGroup 
``` 

<a name="next"></a>

## <a name="next-steps"></a>後續步驟

> [!div class="checklist"]
> * 在 Azure 中建立 MySQL 資料庫
> * 將範例 Java 應用程式連線至 MySQL
> * 將應用程式部署至 Azure
> * 更新和重新部署應用程式
> * 來自 Azure 的串流診斷記錄
> * 在 Azure 入口網站中管理應用程式

前往下一個教學課程，了解如何將自訂的 DNS 名稱對應至該應用程式。

> [!div class="nextstepaction"] 
> [將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)
