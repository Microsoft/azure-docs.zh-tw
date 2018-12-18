---
title: 在 Azure 中建置 PHP 和 MySQL Web 應用程式 | Microsoft Docs
description: 了解如何取得在 Azure 中運作的 PHP 應用程式，並連線至 Azure 中的 MySQL 資料庫。
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: ''
ms.assetid: 14feb4f3-5095-496e-9a40-690e1414bd73
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 10/20/2017
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: b14163bfb9a5e6265158db39e98cb9b31ccef021
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39494103"
---
# <a name="tutorial-build-a-php-and-mysql-web-app-in-azure"></a>教學課程：在 Azure 中建置 PHP 和 MySQL Web 應用程式

> [!NOTE]
> 本文會將應用程式部署至 Windows 上的 App Service。 若要在 Linux 上部署至 App Service，請參閱[在 Linux 上的 Azure App Service 中建置 PHP 和 MySQL 應用程式](./containers/tutorial-php-mysql-app.md)。
>

[Azure Web Apps](app-service-web-overview.md) 提供可高度擴充、自我修復的 Web 主機服務。 本教學課程示範如何在 Azure 中建立 PHP Web 應用程式，並將它連線到 MySQL 資料庫。 完成後，您將有一個在 Azure App Service Web Apps 上執行的 [Laravel](https://laravel.com/) 應用程式。

![在 Azure App Service 中執行的 PHP 應用程式](./media/app-service-web-tutorial-php-mysql/complete-checkbox-published.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 MySQL 資料庫
> * 將 PHP 應用程式連線至 MySQL
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 來自 Azure 的串流診斷記錄
> * 在 Azure 入口網站中管理應用程式

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

* [安裝 Git](https://git-scm.com/)
* [安裝 PHP 5.6.4 或更新版本](http://php.net/downloads.php)
* [安裝編輯器](https://getcomposer.org/doc/00-intro.md)
* 啟用下列 PHP 擴充功能 Laravel 需求︰OpenSSL、PDO-MySQL、Mbstring、Tokenizer、XML
* [下載並啟動 MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html) 

## <a name="prepare-local-mysql"></a>準備本機 MySQL

在此步驟中，您可以在本機 MySQL 伺服器中建立資料庫，供您在本教學課程中使用。

### <a name="connect-to-local-mysql-server"></a>連線至本機 MySQL 伺服器

在終端機視窗中，連線到您的本機 MySQL 伺服器。 您可使用這個終端機視窗來執行本教學課程中的所有命令。

```bash
mysql -u root -p
```

如果系統提示您輸入密碼，請輸入 `root` 帳戶的密碼。 如果您不記得根帳戶密碼，請參閱 [MySQL︰如何重設根密碼](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)。

如果您的命令執行成功，表示您的 MySQL 伺服器正在執行。 如果沒有，請遵循 [MySQL 後續安裝步驟](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html)，確定您已啟動本機 MySQL 伺服器。

### <a name="create-a-database-locally"></a>在本機建立資料庫

在 `mysql` 提示中，建立資料庫。

```sql 
CREATE DATABASE sampledb;
```

輸入 `quit` 即可結束您的伺服器連線。

```sql
quit
```

<a name="step2"></a>

## <a name="create-a-php-app-locally"></a>在本機建立 PHP 應用程式
在此步驟中，您會取得 Laravel 範例應用程式、設定其資料庫連線，並在本機執行。 

### <a name="clone-the-sample"></a>複製範例

在終端機視窗中，使用 `cd` 移至工作目錄。

執行下列命令來複製範例存放庫。

```bash
git clone https://github.com/Azure-Samples/laravel-tasks
```

使用 `cd` 移至您複製的目錄。
安裝必要的套件。

```bash
cd laravel-tasks
composer install
```

### <a name="configure-mysql-connection"></a>設定 MySQL 連線

在儲存機制根路徑中，建立名為 *.env* 的文字檔案。 將下列變數複製到 *.env* 檔案。 將 _&lt;root_password>_ 預留位置取代為 MySQL 根使用者的密碼。

```
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=sampledb
DB_USERNAME=root
DB_PASSWORD=<root_password>
```

如需有關 Laravel 如何使用 _.env_ 檔案的資訊，請參閱 [Laravel 環境設定](https://laravel.com/docs/5.4/configuration#environment-configuration)。

### <a name="run-the-sample-locally"></a>在本機執行範例

執行 [Laravel 資料庫移轉](https://laravel.com/docs/5.4/migrations)來建立應用程式所需的資料表。 若要查看移轉中會建立哪些資料表，請參閱 Git 存放庫中的 _database/migrations_ 目錄。

```bash
php artisan migrate
```

產生新的 Laravel 應用程式金鑰。

```bash
php artisan key:generate
```

執行應用程式。

```bash
php artisan serve
```

在瀏覽器中，瀏覽至 `http://localhost:8000` 。 在頁面中新增幾項工作。

![PHP 成功連線至 MySQL](./media/app-service-web-tutorial-php-mysql/mysql-connect-success.png)

若要停止 PHP 伺服器，請在終端機中輸入`Ctrl + C`。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-mysql-in-azure"></a>在 Azure 中建立 MySQL

在此步驟中，您會在[適用於 MySQL 的 Azure 資料庫](/azure/mysql)中建立 MySQL 資料庫。 稍後，您要將 PHP 應用程式設定為連線至此資料庫。

### <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)] 

### <a name="create-a-mysql-server"></a>建立 MySQL 伺服器

在 Cloud Shell 中，使用 [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create) 命令在適用於 MySQL 的 Azure 資料庫中建立伺服器。

在下列命令中，使用唯一的伺服器名稱取代 \<mysql_server_name> 預留位置，使用使用者名稱取代 \<admin_user> 預留位置，並使用密碼取代 \<admin_password> 預留位置。 這個伺服器名稱會用來作為 MySQL 端點 (`https://<mysql_server_name>.mysql.database.azure.com`) 的一部分，所以在 Azure 的所有伺服器中必須是唯一的名稱。

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql_server_name> --location "West Europe" --admin-user <admin_user> --admin-password <server_admin_password> --sku-name GP_Gen4_2
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

在 Cloud Shell 中，使用 [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule?view=azure-cli-latest#az-mysql-server-firewall-rule-create) 命令，建立 MySQL 伺服器的防火牆規則來允許用戶端連線。 當起始 IP 和結束 IP 都設為 0.0.0.0 時，防火牆只會為其他 Azure 資源開啟。 

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql_server_name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP] 
> [僅使用您的應用程式所用的輸出 IP 位址](app-service-ip-addresses.md#find-outbound-ips)，讓您的防火牆規則更具限制性。
>

### <a name="connect-to-production-mysql-server-locally"></a>在本機連線到生產環境 MySQL 伺服器

在本機終端機視窗中，連線至 Azure 中的 MySQL 伺服器。 使用您先前為 _&lt;mysql_server_name>_ 指定的值。 當系統提示您輸入密碼時，請使用您在 Azure 中建立資料庫時指定的密碼。

```bash
mysql -u <admin_user>@<mysql_server_name> -h <mysql_server_name>.mysql.database.azure.com-P 3306 -p
```

### <a name="create-a-production-database"></a>建立生產環境資料庫

在 `mysql` 提示中，建立資料庫。

```sql
CREATE DATABASE sampledb;
```

### <a name="create-a-user-with-permissions"></a>建立具有權限的使用者

建立名為 _phpappuser_ 的資料庫使用者，並將 `sampledb` 資料庫中所有的權限授權給它。 同樣地，為了簡單起見，請使用_MySQLAzure2017_做為密碼。

```sql
CREATE USER 'phpappuser' IDENTIFIED BY 'MySQLAzure2017'; 
GRANT ALL PRIVILEGES ON sampledb.* TO 'phpappuser';
```

輸入 `quit` 結束伺服器連線。

```sql
quit
```

## <a name="connect-app-to-azure-mysql"></a>將應用程式連線至 Azure MySQL

在此步驟中，您會將 PHP 應用程式連線至您在適用於 MySQL 的 Azure 資料庫中建立的 MySQL 資料庫。

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>設定資料庫連接

在存放庫根目錄中，建立 _.env.production_ 檔案，並將下列變數複製到檔案中。 在 *DB_HOST* 和 *DB_USERNAME* 中，將預留位置取代 _&lt;mysql_server_name >_。

```
APP_ENV=production
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=<mysql_server_name>.mysql.database.azure.com
DB_DATABASE=sampledb
DB_USERNAME=phpappuser@<mysql_server_name>
DB_PASSWORD=MySQLAzure2017
MYSQL_SSL=true
```

儲存變更。

> [!TIP]
> 為了保護您的 MySQL 連接資訊，Git 存放庫中已經排除此檔案 (請看 _.gitignore_ 存放庫的根目錄)。 稍後，您將了解如何設定 App Service 中的環境變數，以連線至適用於 MySQL 的 Azure 資料庫。 使用環境變數，您在 App Service 中就不需要 *.env* 檔案。
>

### <a name="configure-ssl-certificate"></a>設定 SSL 憑證

根據預設，用於 MySQL 的 Azure 資料庫會強制用戶端使用 SSL 連線。 若要連線至 Azure 中的 MySQL 資料庫，您必須使用[適用於 MySQL 的 Azure 資料庫所提供的 _.pem_ 憑證](../mysql/howto-configure-ssl.md)。

開啟 _config/database.php_，在 `connections.mysql` 中新增 `sslmode` 和 `options` 參數，如下列程式碼所示。

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/BaltimoreCyberTrustRoot.crt.pem', 
    ] : []
],
```

為了方便起見，本教學課程中的 `BaltimoreCyberTrustRoot.crt.pem` 憑證會在存放庫中提供。 

### <a name="test-the-application-locally"></a>在本機測試應用程式

使用 _.env.production_ 作為環境檔案以執行 Laravel 資料庫移轉，可在適用於 MySQL 的 Azure 資料庫中建立資料表。 請記住，_.env.production_ 中有連線至您在 Azure 中的 MySQL 資料庫的連線資訊。

```bash
php artisan migrate --env=production --force
```

_.env.production_ 沒有有效的應用程式金鑰。 在終端機中為它產生一個新的。

```bash
php artisan key:generate --env=production --force
```

使用 _.env.production_ 作為環境檔案來執行範例應用程式。

```bash
php artisan serve --env=production
```

瀏覽至 `http://localhost:8000`。 如果頁面載入無誤，PHP 應用程式就會連線至 Azure 中的 MySQL 資料庫。

在頁面中新增幾項工作。

![PHP 順利連線至適用於 MySQL 的 Azure 資料庫](./media/app-service-web-tutorial-php-mysql/mysql-connect-success.png)

若要停止 PHP，請在終端機中輸入 `Ctrl + C`。

### <a name="commit-your-changes"></a>認可變更

執行下列的 Git 命令認可您的變更：

```bash
git add .
git commit -m "database.php updates"
```

您的應用程式已準備好進行部署。

## <a name="deploy-to-azure"></a>部署至 Azure

在此步驟中，您要將已與 MySQL 連線的 PHP 應用程式部署至 Azure App Service。

### <a name="configure-a-deployment-user"></a>設定部署使用者

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>建立應用程式服務方案

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-no-h.md)]

<a name="create"></a>
### <a name="create-a-web-app"></a>建立 Web 應用程式

[!INCLUDE [Create web app no h](../../includes/app-service-web-create-web-app-php-no-h.md)] 

### <a name="configure-database-settings"></a>設定資料庫設定

如先前所指出，您可以使用 App Service 中的環境變數，連線至 Azure MySQL 資料庫。

在 Cloud Shell 中，您可以使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令將環境變數設定為「應用程式設定」。

下列命令會設定 `DB_HOST`、`DB_DATABASE`、`DB_USERNAME`、`DB_PASSWORD` 應用程式設定。 取代預留位置 _&lt;appname>_ 和 _&lt;mysql_server_name>_。

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings DB_HOST="<mysql_server_name>.mysql.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="phpappuser@<mysql_server_name>" DB_PASSWORD="MySQLAzure2017" MYSQL_SSL="true"
```

您可以使用 PHP [getenv](http://www.php.net/manual/function.getenv.php) 方法來存取這些設定。 Laravel 程式碼會透過 PHP `getenv` 使用 [env](https://laravel.com/docs/5.4/helpers#method-env) 包裝函式。 例如，在 _config/database.php_ 中的 MySQL 設定看起來像這樣︰

```php
'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'),
    'username'  => env('DB_USERNAME', 'forge'),
    'password'  => env('DB_PASSWORD', ''),
    ...
],
```

### <a name="configure-laravel-environment-variables"></a>設定 Laravel 環境變數

Laravel 在 App Service 中需要應用程式金鑰。 您可以使用應用程式設定加以設定。

在本機終端機視窗中，使用 `php artisan` 產生新的應用程式金鑰，而不將它儲存為 _.env_。

```bash
php artisan key:generate --show
```

在 Cloud Shell 中，使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令，在 App Service Web 應用程式中設定應用程式金鑰。 取代 _&lt;appname>_ 和 _&lt;outputofphpartisankey:generate>_ 預留位置。

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings APP_KEY="<output_of_php_artisan_key:generate>" APP_DEBUG="true"
```

當部署的 Web 應用程式遇到錯誤，`APP_DEBUG="true"` 會告訴 Laravel 傳回偵錯資訊。 執行生產環境應用程式時，將它設為 `false` 會更安全。

### <a name="set-the-virtual-application-path"></a>設定虛擬應用程式路徑

設定 Web 應用程式的虛擬應用程式路徑。 需要執行此步驟，是因為 [Laravel 應用程式生命週期](https://laravel.com/docs/5.4/lifecycle)是在「公用」目錄中啟動，而不是在應用程式的根目錄。 在根目錄中啟動生命週期的其他 PHP 架構，不需要手動設定虛擬應用程式路徑即可運作。

在 Cloud Shell 中，使用 [`az resource update`](/cli/azure/resource#az-resource-update) 命令來設定虛擬應用程式路徑。 取代 _&lt;appname>_ 預留位置。

```azurecli-interactive
az resource update --name web --resource-group myResourceGroup --namespace Microsoft.Web --resource-type config --parent sites/<app_name> --set properties.virtualApplications[0].physicalPath="site\wwwroot\public" --api-version 2015-06-01
```

依預設，Azure App Service 會將根虛擬應用程式路徑 (_/_) 指向已部署應用程式檔案的根目錄 (_sites\wwwroot_)。

### <a name="push-to-azure-from-git"></a>從 Git 推送至 Azure

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

```bash
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
< Output has been truncated for readability >
```

> [!NOTE]
> 您可能會注意到，部署程序會在結尾安裝 [Composer](https://getcomposer.org/) 套件。 App Service 不會在預設部署期間執行這些自動化，因此，這個範例存放庫在其根目錄中有三個其他檔案可啟用它：
>
> - `.deployment` - 此檔案會告訴 App Service，執行 `bash deploy.sh` 以做為自訂部署指令碼。
> - `deploy.sh` - 自訂部署指令碼。 如果您檢閱檔案，您將看到它會在 `npm install` 之後執行 `php composer.phar install`。
> - `composer.phar` - Composer 套件管理員。
>
> 您可以使用這種方法，將您任何以 Git 為基礎的部署步驟新增至 App Service。 如需相關資訊，請參閱[自訂部署指令碼](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script)。
>

### <a name="browse-to-the-azure-web-app"></a>瀏覽至 Azure Web 應用程式

瀏覽至 `http://<app_name>.azurewebsites.net` 並將幾項工作新增至清單。

![在 Azure App Service 中執行的 PHP 應用程式](./media/app-service-web-tutorial-php-mysql/php-mysql-in-azure.png)

恭喜，您正在 Azure App Service 中執行資料驅動的 PHP 應用程式。

## <a name="update-model-locally-and-redeploy"></a>在本機更新模型並重新部署

在此步驟中，您會簡單變更 `task` 資料模型和 webapp，然後將更新發行至 Azure。

在此工作案例中，您將修改應用程式，讓您可以將工作標示為完成。

### <a name="add-a-column"></a>新增資料行

在本機終端機視窗中，瀏覽至 Git 存放庫的根目錄。

為 `tasks` 資料表產生新的資料庫移轉：

```bash
php artisan make:migration add_complete_column --table=tasks
```

此命令會顯示所產生的移轉檔案名稱。 在 _database/migrations_ 中找到這個檔案並開啟它。

以下列程式碼取代 `up` 方法：

```php
public function up()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->boolean('complete')->default(False);
    });
}
```

上方的程式碼會在名為 `complete` 的 `tasks` 資料表中新增布林值資料行。

將 `down` 方法取代為下列程式碼以進行復原動作︰

```php
public function down()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('complete');
    });
}
```

在本機終端機視窗中，執行 Laravel 資料庫移轉，以在本機資料庫中進行變更。

```bash
php artisan migrate
```

根據 [Laravel 命名慣例](https://laravel.com/docs/5.4/eloquent#defining-models)，依預設 `Task` 模型 (請看 app/Task.php) 會對應至 `tasks` 資料表。

### <a name="update-application-logic"></a>更新應用程式邏輯

開啟 *routes/web.php* 檔案。 應用程式會在這裡定義其路由和商務邏輯。

在檔案的結尾，使用下列程式碼新增路由︰

```php
/**
 * Toggle Task completeness
 */
Route::post('/task/{id}', function ($id) {
    error_log('INFO: post /task/'.$id);
    $task = Task::findOrFail($id);

    $task->complete = !$task->complete;
    $task->save();

    return redirect('/');
});
```

上方的程式碼會切換 `complete` 的值，對資料模型進行簡單的更新。

### <a name="update-the-view"></a>更新檢視

開啟 *resources/views/tasks.blade.php* 檔案。 搜尋 `<tr>` 開頭標記，並將它取代為︰

```html
<tr class="{{ $task->complete ? 'success' : 'active' }}" >
```

上方的程式碼會視工作是否完成來變更資料列色彩。

在下一行中，您會有下列程式碼︰

```html
<td class="table-text"><div>{{ $task->name }}</div></td>
```

將這一整行取代為下列程式碼：

```html
<td>
    <form action="{{ url('task/'.$task->id) }}" method="POST">
        {{ csrf_field() }}

        <button type="submit" class="btn btn-xs">
            <i class="fa {{$task->complete ? 'fa-check-square-o' : 'fa-square-o'}}"></i>
        </button>
        {{ $task->name }}
    </form>
</td>
```

上方的程式碼新增的提交按鈕會參考您稍早定義的路由。

### <a name="test-the-changes-locally"></a>本機測試變更

在本機終端機視窗中，從 Git 儲存機制的根目錄，執行程式開發伺服器。

```bash
php artisan serve
```

若要查看工作狀態的變更，瀏覽至 `http://localhost:8000`，然後選取核取方塊。

![已將核取方塊新增至工作](./media/app-service-web-tutorial-php-mysql/complete-checkbox.png)

若要停止 PHP，請在終端機中輸入 `Ctrl + C`。

### <a name="publish-changes-to-azure"></a>將變更發佈至 Azure

在本機終端機視窗中，使用生產環境連接字串執行 Laravel 資料庫移轉，以在 Azure 資料庫中進行變更。

```bash
php artisan migrate --env=production --force
```

在 Git 中認可所有變更，然後將程式碼變更推送至 Azure。

```bash
git add .
git commit -m "added complete checkbox"
git push azure master
```

完成 `git push` 之後，瀏覽至 Azure Web 應用程式，然後測試新功能。

![發佈至 Azure 的模型和資料庫變更](media/app-service-web-tutorial-php-mysql/complete-checkbox-published.png)

如果您新增任何工作，它們會保留在資料庫中。 對資料結構描述進行的更新，現有資料會原封不動。

## <a name="stream-diagnostic-logs"></a>資料流診斷記錄

當 PHP 應用程式在 Azure App Service 中執行時，您可以使用管線將主控台記錄傳送至終端機。 這樣一來，您就能取得相同的診斷訊息，以協助您偵錯應用程式錯誤。

若要開始記錄資料流，請在 Cloud Shell 中使用 [`az webapp log tail`](/cli/azure/webapp/log?view=azure-cli-latest#az-webapp-log-tail) 命令。

```azurecli-interactive
az webapp log tail --name <app_name> --resource-group myResourceGroup
```

開始記錄資料流之後，重新整理瀏覽器中的 Azure Web 應用程式，以取得部分 Web 流量。 您現在會看到使用管線傳送到終端機的主控台記錄。 如果您沒有立即看到主控台記錄，請在 30 秒後再查看。

若要隨時停止記錄資料流，輸入 `Ctrl`+`C`。

> [!TIP]
> PHP 應用程式可以使用標準 [error_log()](http://php.net/manual/function.error-log.php) 輸出到主控台。 範例應用程式會在 _app/Http/routes.php_ 中使用這種方法。
>
> 如同 web 架構，[Laravel 會使用 Monolog](https://laravel.com/docs/5.4/errors) 作為記錄提供者。 若要了解如何使 Monolog 將訊息輸出至主控台，請參閱 [PHP︰如何使用 Monolog 記錄至主控台 (php://out)](http://stackoverflow.com/questions/25787258/php-how-to-use-monolog-to-log-to-console-php-out)。
>
>

## <a name="manage-the-azure-web-app"></a>管理 Azure Web 應用程式

請移至 [Azure 入口網站](https://portal.azure.com)，以管理您所建立的 Web 應用程式。

按一下左側功能表中的 [應用程式服務]，然後按一下 Azure Web 應用程式的名稱。

![入口網站瀏覽至 Azure Web 應用程式](./media/app-service-web-tutorial-php-mysql/access-portal.png)

您會看到 Web 應用程式的 [概觀] 頁面。 您可以在這裡執行基本管理工作，像是停止、啟動、重新啟動、瀏覽、刪除。

左側功能表提供的頁面可用來設定您的應用程式。

![Azure 入口網站中的 App Service 頁面](./media/app-service-web-tutorial-php-mysql/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 MySQL 資料庫
> * 將 PHP 應用程式連線至 MySQL
> * 將應用程式部署至 Azure
> * 將資料模型更新並將應用程式重新部署
> * 來自 Azure 的串流診斷記錄
> * 在 Azure 入口網站中管理應用程式

前往下一個教學課程，了解如何將自訂的 DNS 名稱對應至 Web 應用程式。

> [!div class="nextstepaction"]
> [將現有的自訂 DNS 名稱對應至 Azure Web Apps](app-service-web-tutorial-custom-domain.md)
