---
title: 教學課程：使用 Azure 入口網站來設計適用於 PostgreSQL 的 Azure 資料庫
description: 此教學課程說明如何使用 Azure 入口網站來設計您的第一個「適用於 PostgreSQL 的 Azure 資料庫」。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: tutorial, mvc
ms.topic: tutorial
ms.date: 03/20/2018
ms.openlocfilehash: 6a3241e15cc72d1d50862a9dec14c23712ec4226
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406377"
---
# <a name="tutorial-design-an-azure-database-for-postgresql-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站來設計適用於 PostgreSQL 的 Azure 資料庫

「適用於 PostgreSQL 的 Azure 資料庫」是一個受控服務，可讓您在雲端執行、管理及調整高可用性 PostgreSQL 資料庫。 使用 Azure 入口網站，您可以輕鬆管理伺服器和設計資料庫。

在此教學課程中，您將使用 Azure 入口網站來學習如何：
> [!div class="checklist"]
> * 建立適用於 PostgreSQL 的 Azure 資料庫伺服器
> * 設定伺服器防火牆
> * 使用 [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) 公用程式來建立資料庫
> * 載入範例資料
> * 查詢資料
> * 更新資料
> * 還原資料

## <a name="prerequisites"></a>先決條件
如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站
登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="create-an-azure-database-for-postgresql"></a>建立適用於 PostgreSQL 的 Azure 資料庫

「適用於 PostgreSQL 的 Azure 資料庫」伺服器是以一組已定義的[計算和儲存體資源](./concepts-compute-unit-and-storage.md)所建立。 伺服器會建立在 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)內。

請依照下列步驟來建立「適用於 PostgreSQL 的 Azure 資料庫」伺服器：
1.  按一下 Azure 入口網站左上角的 [建立資源]。
2.  從 [新增] 頁面中選取 [資料庫]，然後從 [資料庫] 頁面中選取 [適用於 PostgreSQL 的 Azure 資料庫]。
  ![適用於 PostgreSQL 的 Azure 資料庫 - 建立資料庫](./media/tutorial-design-database-using-azure-portal/1-create-database.png)

3.  在新伺服器詳細資料表單中填寫下列資訊︰

    ![建立伺服器](./media/tutorial-design-database-using-azure-portal/2-create.png)

    - 伺服器名稱：**mydemoserver** (伺服器的名稱會與 DNS 名稱對應，因此必須是全域唯一的) 
    - 訂用帳戶：如果您有多個訂用帳戶，請選擇資源所在或作為計費對象的適當訂用帳戶。
    - 資源群組︰**myresourcegroup**
    - 您選擇的伺服器管理員登入和密碼
    - 位置
    - PostgreSQL 版本

   > [!IMPORTANT]
   > 必須要有您在此處指定的伺服器系統管理員登入和密碼，稍後在此教學課程中才能登入伺服器及其資料庫。 請記住或記錄此資訊，以供稍後使用。

4.  按一下 [定價層] 以指定新伺服器的定價層。 此教學課程中，選取 [一般用途]、[Gen 4] 計算世代，2 個 [虛擬核心]，5 GB 的 [儲存體]，以及 7 天的 [備份保留期限]。 在備份備援選項中選取 [異地備援]，將您伺服器的自動備份儲存在異地備援儲存體。
 ![適用於 PostgreSQL 的 Azure 資料庫 - 挑選定價層](./media/tutorial-design-database-using-azure-portal/2-pricing-tier.png)

5.  按一下 [確定] 。

6.  按一下 [建立] 以佈建伺服器。 佈建需要幾分鐘的時間。

7.  在工具列上，按一下 [通知] 以監視部署程序。
 ![適用於 PostgreSQL 的 Azure 資料庫 - 查看通知](./media/tutorial-design-database-using-azure-portal/3-notifications.png)

   > [!TIP]
   > 選取 [釘選到儀表板] 選項以輕鬆追蹤部署。

   根據預設，**postgres** 資料庫會建立在您的伺服器底下。 [postgres](https://www.postgresql.org/docs/9.6/static/app-initdb.html) 資料庫是要供使用者、公用程式及第三方應用程式使用的預設資料庫。 

## <a name="configure-a-server-level-firewall-rule"></a>設定伺服器層級防火牆規則

「適用於 PostgreSQL 的 Azure 資料庫」服務會使用伺服器層級的防火牆。 根據預設，除非已建立防火牆規則來針對特定 IP 位址範圍打開防火牆，否則此防火牆會防止所有外部應用程式和工具連線到伺服器及伺服器上的任何資料庫。 

1.  完成部署之後，從左側功能表中按一下 [所有資源]，然後輸入名稱 **mydemoserver** 來搜尋新建立的伺服器。 按一下搜尋結果中列出的伺服器名稱。 伺服器的 [概觀] 頁面隨即開啟，並提供可進行進一步設定的選項。

   ![適用於 PostgreSQL 的 Azure 資料庫 - 搜尋伺服器 ](./media/tutorial-design-database-using-azure-portal/4-locate.png)

2.  在伺服器頁面中，選取 [連線安全性]。 

3.  在 [規則名稱] 底下的文字方塊中按一下，然後新增新的防火牆規則，以將 IP 範圍加入允許清單來獲得連線能力。 輸入您的 IP 範圍。 按一下 [檔案] 。

   ![適用於 PostgreSQL 的 Azure 資料庫 - 建立防火牆規則](./media/tutorial-design-database-using-azure-portal/5-firewall-2.png)

4.  按一下 [儲存]，然後按一下 [X]，以關閉 [連線安全性] 頁面。

   > [!NOTE]
   > Azure PostgreSQL 伺服器會透過連接埠 5432 進行通訊。 如果您嘗試從公司網路內進行連線，您網路的防火牆可能不允許透過連接埠 5432 的輸出流量。 若情況如此，除非 IT 部門開啟連接埠 5432，否則您無法連線至 Azure SQL Database 伺服器。
   >

## <a name="get-the-connection-information"></a>取得連線資訊

您建立「適用於 PostgreSQL 的 Azure 資料庫」伺服器時，會一併建立預設的 **postgres** 資料庫。 若要連線到您的資料庫伺服器，您必須提供主機資訊和存取認證。

1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您剛建立的伺服器。

   ![適用於 PostgreSQL 的 Azure 資料庫 - 搜尋伺服器 ](./media/tutorial-design-database-using-azure-portal/4-locate.png)

2. 按一下伺服器名稱 **mydemoserver**。

3. 選取伺服器的 [概觀] 頁面。 記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。

   ![適用於 PostgreSQL 的 Azure 資料庫 - 伺服器管理員登入](./media/tutorial-design-database-using-azure-portal/6-server-name.png)


## <a name="connect-to-postgresql-database-using-psql-in-cloud-shell"></a>在 Cloud Shell 中使用 psql 來連線到 PostgreSQL 資料庫

現在，我們將使用 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 命令列公用程式來連線到「適用於 PostgreSQL 的 Azure 資料庫」伺服器。 
1. 透過頂端瀏覽窗格上的終端機圖示啟動 Azure Cloud Shell。

   ![適用於 PostgreSQL 的 Azure 資料庫 - Azure Cloud Shell 終端機圖示](./media/tutorial-design-database-using-azure-portal/7-cloud-shell.png)

2. Azure Cloud Shell 會在您的瀏覽器中開啟，讓您能夠輸入 bash 命令。

   ![適用於 PostgreSQL 的 Azure 資料庫 - Azure Shell Bash 提示字元](./media/tutorial-design-database-using-azure-portal/8-bash.png)

3. 在 Cloud Shell 提示字元處，使用 psql 命令來連線到「適用於 PostgreSQL 的 Azure 資料庫」伺服器。 下列格式可用來透過 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 公用程式連線到「適用於 PostgreSQL 的 Azure 資料庫」伺服器：
   ```bash
   psql --host=<myserver> --port=<port> --username=<server admin login> --dbname=<database name>
   ```

   例如，下列命令會使用存取認證，連線到 PostgreSQL 伺服器 **mydemoserver.postgres.database.azure.com** 上名為 **postgres** 的預設資料庫。 在系統提示時輸入您的伺服器管理員密碼。

   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

## <a name="create-a-new-database"></a>建立新資料庫
連線到伺服器之後，在提示字元建立空白資料庫。
```bash
CREATE DATABASE mypgsqldb;
```

在提示字元，執行下列命令以將連線切換到新建立的資料庫 **mypgsqldb**。
```bash
\c mypgsqldb
```
## <a name="create-tables-in-the-database"></a>在資料庫中建立資料表
既然您已知道如何連線到「適用於 PostgreSQL 的 Azure 資料庫」，您可以了解如何完成一些基本工作：

首先，建立資料表並在其中載入一些資料。 我們將使用下列的 SQL 程式碼建立一個追蹤清查資訊的資料表：
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

您現在可以輸入下列命令來查看資料表清單中新建立的資料表：
```sql
\dt
```

## <a name="load-data-into-the-tables"></a>將資料載入到資料表
既然您已經有資料表，請在其中插入一些資料。 在開啟的命令提示字元視窗，執行下列查詢以插入幾列資料。
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

您現在已將兩列範例資料插入到先前建立的清查資料表中。

## <a name="query-and-update-the-data-in-the-tables"></a>查詢並更新資料表中的資料
執行下列查詢，以從清查資料庫資料表中擷取資訊。 
```sql
SELECT * FROM inventory;
```

您也可以更新資料表中的資料。
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

當您擷取資料時，可以看到更新的值。
```sql
SELECT * FROM inventory;
```

## <a name="restore-data-to-a-previous-point-in-time"></a>將資料還原到先前的時間點
想像一下您不小心刪除了這個資料表。 這是您無法輕易復原的情況。 適用於 PostgreSQL 的 Azure 資料庫可讓您返回至伺服器上任何有備份的時間點 (取決於您設定的備份保留期限)，並將此時間點還原至新的伺服器。 您可以使用這個新的伺服器來復原已刪除的資料。 下列步驟會將 **mydemoserver** 伺服器還原到新增清查資料表之前的時間點。

1.  在您伺服器之「適用於 PostgreSQL 的 Azure 資料庫」的 [概觀] 頁面上，按一下工具列上的 [還原]。 [還原] 頁面隨即開啟。

   ![Azure 入口網站 - 還原表單選項](./media/tutorial-design-database-using-azure-portal/9-azure-portal-restore.png)

2.  在 [還原] 表單中填入必要資訊︰

   ![Azure 入口網站 - 還原表單選項](./media/tutorial-design-database-using-azure-portal/10-azure-portal-restore.png)

   - **還原點**：選取在變更伺服器之前的時間點
   - **目標伺服器**︰提供要作為還原目的地的新伺服器名稱
   - **位置**︰您無法選取區域，預設是與來源伺服器相同的區域
   - **定價層**︰還原伺服器時，您無法變更此值。 它與來源伺服器相同。 
3.  按一下 [確定] 以[將伺服器還原到刪除資料表之前的時間點](./howto-restore-server-portal.md)。 如果將伺服器還原到不同的時間點，將會從您指定的時間點 (前提是此時間點在您[定價層](./concepts-pricing-tiers.md)的保留期限內) 開始，建立重複的新伺服器作為原始伺服器。

## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解如何使用 Azure 入口網站和其他公用程式來：
> [!div class="checklist"]
> * 建立適用於 PostgreSQL 的 Azure 資料庫伺服器
> * 設定伺服器防火牆
> * 使用 [**psql**](https://www.postgresql.org/docs/9.6/static/app-psql.html) 公用程式來建立資料庫
> * 載入範例資料
> * 查詢資料
> * 更新資料
> * 還原資料

接著，若要了解如何使用 Azure CLI 來執行類似的工作，請檢閱此教學課程：[使用 Azure CLI 來設計您第一個適用於 PostgreSQL 的 Azure 資料庫](tutorial-design-database-using-azure-cli.md)
