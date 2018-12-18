---
title: 在 Azure App Service 中建置 Python 和 PostgreSQL Web 應用程式 | Microsoft Docs
description: 了解如何在 Azure 中連線至 PostgreSQL 資料庫，以執行資料驅動的 Python 應用程式。
services: app-service\web
documentationcenter: python
author: cephalin
manager: jeconnoc
ms.service: app-service-web
ms.workload: web
ms.devlang: python
ms.topic: tutorial
ms.date: 09/28/2018
ms.author: beverst;cephalin
ms.custom: mvc
ms.openlocfilehash: f4ce197d541b8573e38fd85dcebb575c8ee99f59
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435703"
---
# <a name="build-a-docker-python-and-postgresql-web-app-in-azure"></a>在 Azure 中建置 Docker Python 和 PostgreSQL Web 應用程式

[Linux 上的 App Service](app-service-linux-intro.md) 提供可高度擴充、自我修復的 Web 主機服務。 本教學課程說明如何以 PostgreSQL 作為資料庫後端，建立資料驅動 Python Web 應用程式。 完成之後，您在 Linux 上的 App Service 中就會有在Docker 容器內執行的 Python Flask 應用程式。

![Linux 上的 App Service 中的 Docker Python Flask 應用程式](./media/tutorial-python-postgresql-app/docker-flask-in-azure.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 PostgreSQL 資料庫
> * 將 Python 應用程式連線至 PostgreSQL
> * 將應用程式部署至 Azure
> * 檢視診斷記錄
> * 將資料模型更新並將應用程式重新部署
> * 在 Azure 入口網站中管理應用程式

您可以在 macOS 上依照本文中的步驟進行。 Linux 和 Windows 指示在大部分情況下都相同，本教學課程對差異不加詳述。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

1. [安裝 Git](https://git-scm.com/)
1. [安裝 Python](https://www.python.org/downloads/)
1. [安裝及執行 PostgreSQL](https://www.postgresql.org/download/)

## <a name="test-local-postgresql-installation-and-create-a-database"></a>測試本機 PostgreSQL 安裝並建立資料庫

在本機終端機視窗中，執行 `psql` 以連線至本機 PostgreSQL 伺服器。

```bash
sudo -u postgres psql postgres
```

如果您收到類似於 `unknown user: postgres` 的錯誤訊息，請使用您的登入使用者名稱來設定 PostgreSQL 安裝。 改為嘗試下列命令。

```bash
psql postgres
```

如果連線成功，則您的 PostgreSQL 資料庫就已在執行中。 如果沒有，請確定本機 PostgresQL 資料庫已遵循 [Downloads - PostgreSQL Core Distribution](https://www.postgresql.org/download/) (下載 - PostgresQL 核心散發) 中針對您作業系統的指示來啟動。

建立名為 eventregistration 的資料庫，並且設定名為 manager、密碼為 supersecretpass 的個別資料庫使用者。

```sql
CREATE DATABASE eventregistration;
CREATE USER manager WITH PASSWORD 'supersecretpass';
GRANT ALL PRIVILEGES ON DATABASE eventregistration TO manager;
```

輸入 `\q` 來結束 PostgreSQL 用戶端。

<a name="step2"></a>

## <a name="create-local-python-app"></a>建立本機 Python 應用程式

在此步驟中，您要設定本機 Python Flask 專案。

### <a name="clone-the-sample-app"></a>複製範例應用程式

開啟終端機視窗，然後 `CD` 至工作目錄。

執行下列命令來複製範例存放庫。

```bash
git clone https://github.com/Azure-Samples/flask-postgresql-app.git
cd flask-postgresql-app
```

此範例存放庫包含 [Flask](http://flask.pocoo.org/) 應用程式。

### <a name="run-the-app-locally"></a>在本機執行應用程式

安裝必要的封裝，然後啟動應用程式。

```bash
# Bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cd app
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask run

# PowerShell
pip install virtualenv
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
cd app
Set-Item Env:FLASK_APP ".\app.py"
DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

當應用程式完全載入時，您會看到類似下列的訊息：

```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 791cd7d80402, empty message
 * Serving Flask app "app"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

在瀏覽器中，瀏覽至 `http://localhost:5000` 。 按一下 [註冊!] 並且建立測試使用者。

![在本機執行的 Python Flask 應用程式](./media/tutorial-python-postgresql-app/local-app.png)

Flask 範例應用程式會將使用者資料儲存於資料庫中。 如果您成功註冊使用者，您的應用程式會將資料寫入本機 PostgreSQL 資料庫。

如需隨時停止 Flask 伺服器，請在終端機上輸入 Ctrl+C。

## <a name="create-a-production-postgresql-database"></a>建立生產環境 PostgreSQL 資料庫

在此步驟中，您要在 Azure 中建立 PostgreSQL 資料庫。 當您的應用程式部署至 Azure 時，它會使用此雲端資料庫。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [Create resource group](../../../includes/app-service-web-create-resource-group-linux-no-h.md)]

### <a name="create-an-azure-database-for-postgresql-server"></a>建立適用於 PostgreSQL 的 Azure 資料庫伺服器

在 Cloud Shell 中使用 [`az postgres server create`](/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-create) 命令建立 PostgreSQL 伺服器。

在下列範例命令中，請將 *\<postgresql_name>* 取代為唯一的伺服器名稱，並將 *\<admin_username>* 和 *\<admin_password>* 取代為所需的使用者認證。 使用者認證是用於資料庫管理員帳戶。 這個伺服器名稱會用來作為 PostgreSQL 端點 (`https://<postgresql_name>.postgres.database.azure.com`) 的一部分，所以在 Azure 的所有伺服器中必須是唯一的名稱。

```azurecli-interactive
az postgres server create --resource-group myResourceGroup --name <postgresql_name> --location "West Europe" --admin-user <admin_username> --admin-password <admin_password> --sku-name B_Gen4_1
```

建立適用於 PostgreSQL 的 Azure 資料庫伺服器後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "administratorLogin": "<admin_username>",
  "fullyQualifiedDomainName": "<postgresql_name>.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/<postgresql_name>",
  "location": "westus",
  "name": "<postgresql_name>",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "capacity": 1,
    "family": "Gen4",
    "name": "B_Gen4_1",
    "size": null,
    "tier": "Basic"
  },
  < JSON data removed for brevity. >
}
```

> [!NOTE]
> 請記住 \<管理員使用者名稱> 和 \<管理員密碼> 以便稍後使用。 您需要這些資訊，才能登入 Postgre 伺服器及其資料庫。


### <a name="create-firewall-rules-for-the-postgresql-server"></a>建立 PostgreSQL 伺服器的防火牆規則

在 Cloud Shell 中執行下列 Azure CLI 命令，以允許從 Azure 資源存取資料庫。

```azurecli-interactive
az postgres server firewall-rule create --resource-group myResourceGroup --server-name <postgresql_name> --start-ip-address=0.0.0.0 --end-ip-address=0.0.0.0 --name AllowAllAzureIPs
```

> [!NOTE]
> 此設定允許來自 Azure 網路內所有 IP 的網路連線。 針對生產用途，請嘗試[僅使用您應用程式所用的輸出 IP 位址](../app-service-ip-addresses.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips)，來設定可能最嚴格的防火牆規則。

在 Cloud Shell 中，將 *\<您的 IP 位址>* 取代為[您的本機 IPv4 IP 位址](http://www.whatsmyip.org/)並再次執行命令，以允許從您的本機電腦進行存取。

```azurecli-interactive
az postgres server firewall-rule create --resource-group myResourceGroup --server-name <postgresql_name> --start-ip-address=<your_ip_address> --end-ip-address=<your_ip_address> --name AllowLocalClient
```

## <a name="connect-python-app-to-production-database"></a>將 Python 應用程式連線至生產資料庫

在此步驟中，您會將 Flask 範例應用程式連線至您所建立的「適用於 PostgreSQL 的 Azure 資料庫」伺服器。

### <a name="create-empty-database-and-user-access"></a>建立空的資料庫和使用者存取

在本機終端機視窗中，執行以下命令以連線至資料庫。 在出現管理員密碼的提示時，請使用您在[建立適用於 PostgreSQL 的 Azure 資料庫伺服器](#create-an-azure-database-for-postgresql-server)中指定的相同密碼。

```bash
psql -h <postgresql_name>.postgres.database.azure.com -U <my_admin_username>@<postgresql_name> postgres
```

如同您的本機 Postgres 伺服器，在 Azure Postgres 伺服器中建立資料庫和使用者。

```bash
CREATE DATABASE eventregistration;
CREATE USER manager WITH PASSWORD 'supersecretpass';
GRANT ALL PRIVILEGES ON DATABASE eventregistration TO manager;
```

輸入 `\q` 來結束 PostgreSQL 用戶端。

> [!NOTE]
> 最佳做法是建立具有特定應用程式限制權限的資料庫使用者，而不是使用管理使用者。 在此範例中，`manager` 使用者「只」具有 `eventregistration` 資料庫的完整權限。

### <a name="test-app-connectivity-to-production-database"></a>測試應用程式與生產資料庫的連線

回到本機終端機視窗並執行下列命令，以執行 Flask 資料庫移轉和 Flask 伺服器。

```bash
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

當應用程式完全載入時，您會看到類似下列的訊息：

```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 791cd7d80402, empty message
 * Serving Flask app "app"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

在瀏覽器中，瀏覽至 http://localhost:5000 。 按一下 [註冊!] 並且建立測試註冊。 現在您要將資料寫入 Azure 中的資料庫。

![在本機執行的 Python Flask 應用程式](./media/tutorial-python-postgresql-app/local-app.png)

## <a name="deploy-to-azure"></a>部署至 Azure

在此步驟中，您會將與 Postgres 連線的 Python 應用程式部署至 Azure App Service。

### <a name="configure-repository"></a>設定存放庫

當存放庫根目錄中有 _application.py_ 時，App Service 中的 Git 部署引擎會叫用 `pip` 自動化。 在本教學課程中，您將讓部署引擎為您執行自動化。 在本機終端機視窗中，巡覽至存放庫根目錄，建立虛擬 _application.py_，然後認可您的變更。

```bash
cd ..
touch application.py
git add .
git commit -m "ensure azure automation"
```

### <a name="configure-a-deployment-user"></a>設定部署使用者

[!INCLUDE [Configure deployment user](../../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>建立應用程式服務方案 

[!INCLUDE [Create app service plan](../../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

### <a name="create-a-web-app"></a>建立 Web 應用程式 

[!INCLUDE [Create web app](../../../includes/app-service-web-create-web-app-python-linux-no-h.md)]

### <a name="configure-environment-variables"></a>設定環境變數

稍早在本教學課程中，您定義了環境變數來連線至 PostgreSQL 資料庫。

在 App Service 中，您可以在 Cloud Shell 中使用 [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令將環境變數設定為「應用程式設定」。

下列範例會指定資料庫連線詳細資料作為應用程式設定。 

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBPASS="supersecretpass" DBNAME="eventregistration"
```

### <a name="push-to-azure-from-git"></a>從 Git 推送至 Azure

[!INCLUDE [app-service-plan-no-h](../../../includes/app-service-web-git-push-to-azure-no-h.md)]

```bash 
Counting objects: 5, done. 
Delta compression using up to 4 threads. 
Compressing objects: 100% (5/5), done. 
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done. 
Total 5 (delta 3), reused 0 (delta 0) 
remote: Updating branch 'master'. 
remote: Updating submodules. 
remote: Preparing deployment for commit id '6c7c716eee'. 
remote: Running custom deployment command... 
remote: Running deployment command... 
remote: Handling node.js deployment. 
. 
. 
. 
remote: Deployment successful. 
To https://<app_name>.scm.azurewebsites.net/<app_name>.git 
 * [new branch]      master -> master 
```  

### <a name="configure-entry-point"></a>設定進入點

根據預設，內建映像會在根目錄中尋找 _wsgi.py_ 或 _application.py_ 作為進入點，但您的進入點是 _app/app.py_。 您稍早新增的 _application.py_ 為空白，而且不會執行任何動作。

在 Cloud Shell 中，執行 [`az webapp config set`](/cli/azure/webapp/config?view=azure-cli-latest#az-webapp-config-set) 命令來設定自訂啟動指令碼。

```azurecli-interactive
az webapp config set --name <app_name> --resource-group myResourceGroup --startup-file "gunicorn '--bind=0.0.0.0' --chdir /home/site/wwwroot/app app:app"
```

`--startup-file` 參數接受自訂命令或包含自訂命令的檔案路徑。 您的自訂命令應有下列格式：

```
gunicorn '--bind=0.0.0.0' --chdir /home/site/wwwroot/<subdirectory> <module>:<variable>
```

在自訂命令中，如果您的進入點不在根目錄中，則需要 `--chdir`，而 `<subdirectory>` 是子目錄。 `<module>` 是 _.py_ 檔案的名稱，而 `<variable>` 是代表您 Web 應用程式的模組中變數。

### <a name="browse-to-the-azure-web-app"></a>瀏覽至 Azure Web 應用程式

瀏覽至已部署的 Web 應用程式。 它需要一些時間才能啟動，因為在第一次要求應用程式時必須下載並執行容器。 如果頁面逾時或顯示錯誤訊息，請稍候幾分鐘並重新整理頁面。

```bash
http://<app_name>.azurewebsites.net
```

您會看到先前已註冊的來賓，儲存至上一個步驟中的 Azure 生產環境資料庫。

![在 Azure 中執行的 Python Flask 應用程式](./media/tutorial-python-postgresql-app/docker-app-deployed.png)

**恭喜！** 您正在適用於 Linux 的 App Service 中執行 Python 應用程式。

## <a name="access-diagnostic-logs"></a>存取診斷記錄檔

因為 Python 應用程式是在容器中執行，所以 Linux 上的 App Service 可讓您存取從容器內產生的主控台記錄。 若要尋找記錄，請巡覽至這個 URL：

```
https://<app_name>.scm.azurewebsites.net/api/logs/docker
```

您應該會看到兩個 JSON 物件，各有一個 `href` 屬性。 一個 `href` 指向 Docker 主控台記錄 (結尾是 `_docker.log`)，另一個 `href` 指向從 Python 容器內產生的主控台記錄。 

```json
[  
   {  
      "machineName":"RD0003FF61ACD0_default",
      "lastUpdated":"2018-09-27T16:48:17Z",
      "size":4766,
      "href":"https://<app_name>.scm.azurewebsites.net/api/vfs/LogFiles/2018_09_27_RD0003FF61ACD0_default_docker.log",
      "path":"/home/LogFiles/2018_09_27_RD0003FF61ACD0_default_docker.log"
   },
   {  
      "machineName":"RD0003FF61ACD0",
      "lastUpdated":"2018-09-27T16:48:19Z",
      "size":2589,
      "href":"https://<app_name>.scm.azurewebsites.net/api/vfs/LogFiles/2018_09_27_RD0003FF61ACD0_docker.log",
      "path":"/home/LogFiles/2018_09_27_RD0003FF61ACD0_docker.log"
   }
]
```

將您想要的 `href` 值複製到瀏覽器視窗以巡覽至記錄。 記錄尚未串流，因此您可能會遇到一些延遲。 若要查看新記錄，請重新整理瀏覽器頁面。

## <a name="update-data-model-and-redeploy"></a>更新資料模型並重新部署

在此步驟中，您會更新 `Guest` 模式，將出席者的人數新增至每個事件註冊，然後將更新重新部署至 Azure。

在本機終端機視窗中，使用下列 git 命令從 `modelChange` 分支簽出檔案：

```bash
git checkout origin/modelChange -- .
```

此簽出已對模型、檢視和控制器進行必要變更。 它也會包含透過 alembic (`flask db migrate`) 產生的資料庫移轉。 您可以透過下列 git 命令查看所有變更：

```bash
git diff master origin/modelChange
```

### <a name="test-your-changes-locally"></a>本機測試您的變更

在本機終端機視窗中執行下列命令，以藉由執行 Flask 伺服器在本機測試您的變更。

```bash
source venv/bin/activate
cd app
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

在瀏覽器中瀏覽至 http://localhost:5000 可檢視變更。 建立測試註冊。

![以 Docker 容器為基礎在本機執行的 Python Flask 應用程式](./media/tutorial-python-postgresql-app/local-app-v2.png)

### <a name="publish-changes-to-azure"></a>將變更發佈至 Azure

在本機終端機視窗中，在 Git 中認可所有變更，然後將程式碼變更推送至 Azure。

```bash 
git add . 
git commit -m "updated data model" 
git push azure master 
``` 

瀏覽至 Azure Web 應用程式，然後再次嘗試執行新功能。 請務必重新整理頁面。

```bash
http://<app_name>.azurewebsites.net
```

![Azure App Service 中的 Docker Python Flask 應用程式](./media/tutorial-python-postgresql-app/docker-flask-in-azure.png)

## <a name="manage-your-web-app-in-the-azure-portal"></a>在 Azure 入口網站管理您的 Web 應用程式

請移至 [Azure 入口網站](https://portal.azure.com)，以查看您所建立的 Web 應用程式。

按一下左側功能表中的 [應用程式服務]，然後按一下 Azure Web 應用程式的名稱。

![入口網站瀏覽至 Azure Web 應用程式](./media/tutorial-python-postgresql-app/app-resource.png)

根據預設，入口網站會顯示 Web 應用程式的 [概觀] 分頁。 此頁面可讓您檢視應用程式的執行方式。 您也可以在這裡執行基本管理工作，像是瀏覽、停止、啟動、重新啟動及刪除。 分頁左側的索引標籤會顯示您可開啟的各種設定分頁。

![Azure 入口網站中的 App Service 頁面](./media/tutorial-python-postgresql-app/app-mgmt.png)

[!INCLUDE [cli-samples-clean-up](../../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立 PostgreSQL 資料庫
> * 將 Python 應用程式連線至 PostgreSQL
> * 將應用程式部署至 Azure
> * 檢視診斷記錄
> * 將資料模型更新並將應用程式重新部署
> * 在 Azure 入口網站中管理應用程式

前往下一個教學課程，了解如何將自訂的 DNS 名稱對應至 Web 應用程式。

> [!div class="nextstepaction"]
> [設定內建 Python 映像](how-to-configure-python.md)

> [!div class="nextstepaction"]
> [將現有的自訂 DNS 名稱對應至 Azure Web Apps](../app-service-web-tutorial-custom-domain.md)

