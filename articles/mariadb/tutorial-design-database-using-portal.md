---
title: 教學課程：使用 Azure 入口網站設計適用於 MariaDB 的 Azure 資料庫
description: 本教學課程說明如何使用「Azure 入口網站」來建立和管理「適用於 MariaDB 的 Azure 資料庫」伺服器和資料庫。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: tutorial
ms.date: 09/24/2018
ms.custom: mvc
ms.openlocfilehash: 20714bed5c36cd93d11cb4ade6640798775f5055
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49322051"
---
# <a name="tutorial-design-an-azure-database-for-mariadb-database-by-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站設計適用於 MariaDB 的 Azure 資料庫

適用於 MariaDB 的 Azure 資料庫是一個受控服務，您可用來在雲端執行、管理及調整高可用性 MySQL 資料庫。 使用 Azure 入口網站，您可以輕鬆管理伺服器和設計資料庫。

在本教學課程中，您將使用 Azure 入口網站來學習如何：

> [!div class="checklist"]
> * 建立適用於 MariaDB 的 Azure 資料庫
> * 設定伺服器防火牆
> * 使用 mysql 命令列工具建立資料庫
> * 載入範例資料
> * 查詢資料
> * 更新資料
> * 還原資料

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

在瀏覽器中，移至 [Azure 入口網站](https://portal.azure.com/)。 輸入您的認證來登入此入口網站。 預設檢視是您的服務儀表板。

## <a name="create-an-azure-database-for-mariadb-server"></a>建立適用於 MariaDB 的 Azure 資料庫伺服器

您可使用一組已定義的[計算和儲存體資源](concepts-pricing-tiers.md)來建立適用於 MariaDB 的 Azure 資料庫伺服器。 伺服器會建立在 [Azure 資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)內。

1. 選取入口網站左上角的 [建立資源] 按鈕 (+)。

2. 在搜尋方塊中輸入 [適用於 MariaDB 的 Azure 資料庫]，以尋找此服務。
   
   ![移至 MySQL](./media/tutorial-design-database-using-portal/1-Navigate-to-mariadb.png)

3. 選取 [適用於 MariaDB 的 Azure 資料庫] 圖格，然後選取 [建立]。 輸入或選取必要的資訊。
   
   ![建立表單](./media/tutorial-design-database-using-portal/2-create-form.png)

    設定 | 建議的值 | 欄位描述 
    ---|---|---
    伺服器名稱 | 唯一的伺服器名稱 | 選擇可識別適用於 MariaDB 的 Azure 資料庫伺服器的唯一名稱。 例如，**mydemoserver**。 網域名稱 *.mariadb.database.azure.com* 會附加至您輸入的伺服器名稱。 伺服器名稱只能包含小寫字母、數字及連字號 (-) 字元。 它必須包含 3 到 63 個字元。
    訂用帳戶 | *您的訂用帳戶* | 選取您要用於伺服器的 Azure 訂用帳戶。 如果您有多個訂用帳戶，請選擇資源計費的訂用帳戶。
    資源群組 | **myresourcegroup** | 輸入新的資源群組名稱，或選取現有資源群組。
    選取來源 | 空白 | 選取 [空白]，以建立新的伺服器。 (如果您要從現有「適用於 MariaDB 的 Azure 資料庫」伺服器的異地備份建立伺服器，請選取 [備份])。
    伺服器管理員登入 | myadmin | 連線至伺服器時所要使用的登入帳戶。 系統管理員登入名稱不能是 **azure_superuser**、**admin**、**administrator**、**root**、**guest** 或 **public**。
    密碼 | 您的選擇 | 為伺服器管理帳戶輸入新密碼。 其必須包含 8 到 128 個字元。 您的密碼必須包含下列三個類別的字元：英文大寫字母、英文小寫字母、數字 (0-9) 和非英數字元 (!、$、#、% 等等)。
    確認密碼 | 您的選擇| 確認管理帳戶密碼。
    位置 | 最接近使用者的區域| 選取最靠近使用者或其他 Azure 應用程式的位置。
    版本 | 最新版本| 最新版本 (除非您有使用不同版本的特定需求)。
    定價層 | 請參閱說明。 | 新伺服器的計算、儲存體和備份組態。 選取 [定價層] > [一般用途]。 保留下列設定的預設值：<br><ul><li>**計算產生** (Gen 5)</li><li>**虛擬核心** (2 個虛擬核心)</li><li>**儲存體** (5 GB)</li><li>**備份保留期限** (7 天)</li></ul><br>若要啟用異地備援儲存體中的伺服器備份，請為 [備份備援選項] 選取 [異地備援]。 <br><br>若要儲存此定價層選取項目，請選取 [確定]。 下方螢幕擷取畫面會擷取這些選取項目。
    
   ![定價層](./media/tutorial-design-database-using-portal/3-pricing-tier.png)

4. 選取 [建立] 。 在一兩分鐘內，新的「適用於 MariaDB 的 Azure 資料庫」伺服器就會在雲端執行。 若要監視部署程序，請在工具列上選取 [通知]。

## <a name="configure-the-firewall"></a>設定防火牆

「適用於 MariaDB 的 Azure 資料庫」會受到防火牆保護。 依預設，伺服器與其內部資料庫的所有連線皆會遭拒。 第一次連線到適用於 MariaDB 的 Azure 資料庫之前，請設定防火牆來新增用戶端電腦的公用網路 IP 位址 (或 IP 位址範圍)。

1. 選取您新建立的伺服器，然後選取 [連線安全性]。
   
   ![連接安全性](./media/tutorial-design-database-using-portal/1-Connection-security.png)
2. 您可以在這裡選取 [新增我的 IP]，或設定防火牆規則。 請記得在建立規則後選取 [儲存]。

您現在可以使用 mysql 命令列工具或 MySQL Workbench 來連線到伺服器。

> [!TIP]
> 「適用於 MariaDB 的 Azure 資料庫」伺服器會透過連接埠 3306 進行通訊。 如果您嘗試從公司網路內進行連線，您網路的防火牆可能不允許透過連接埠 3306 的輸出流量。 在此情況下，若要連線至「適用於 MariaDB 的 Azure 資料庫」伺服器，IT 部門必須開啟連接埠 3306。

## <a name="get-connection-information"></a>取得連線資訊

請從 Azure 入口網站取得「適用於 MariaDB 的 Azure 資料庫」伺服器的 [伺服器名稱] (完整) 和 [伺服器管理員登入名稱] 值。 您將使用 mysql 命令列工具搭配此完整伺服器名稱來連線到伺服器。 

1. 在 [Azure 入口網站](https://portal.azure.com/)的左側功能表中，選取 [所有資源]。 輸入伺服器名稱，並搜尋適用於 MariaDB 的 Azure 資料庫伺服器。 選取伺服器名稱以檢視伺服器詳細資料。

2. 在 [概觀] 頁面上，記下 [伺服器名稱] 和 [伺服器管理員登入名稱] 的值。 您也可以選取每個欄位旁邊的 [複製] 按鈕，將其值複製到剪貼簿。

   ![伺服器屬性](./media/tutorial-design-database-using-portal/2-server-properties.png)

在我們的範例中，伺服器名稱為 **mydemoserver.mariadb.database.azure.com**，而伺服器管理員登入名稱為 **myadmin@mydemoserver**。

## <a name="connect-to-the-server-by-using-mysql"></a>使用 mysql 來連線到伺服器

使用 [mysql 命令列工具](https://dev.mysql.com/doc/refman/5.7/en/mysql.html)來建立對「適用於 MariaDB 的 Azure 資料庫」伺服器的連線。 您可以從 Azure Cloud Shell 在瀏覽器中，或從電腦使用本機安裝的 mysql 工具執行 mysql 命令列工具。 若要開啟 Azure Cloud Shell，請選取本文中程式碼區塊上的 [試用] 按鈕，或造訪 Azure 入口網站並按一下右上方工具列中的 **>_** 圖示。 

輸入下列命令以進行連線：

```azurecli-interactive
mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>建立空白資料庫

連線到伺服器時，請建立一個要搭配運作的空白資料庫：

```sql
CREATE DATABASE mysampledb;
```

在提示字元，執行下列命令以將連線切換到新建立的資料庫：

```sql
USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>在資料庫中建立資料表

您已了解如何連線到「適用於 MariaDB 資料庫的 Azure 資料庫」，現在您可以完成一些基本工作。

首先，建立資料表並在其中載入一些資料。 我們將建立一個儲存清查資訊的資料表：

```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-in-the-tables"></a>將資料載入到資料表

既然您已經有資料表，請在其中插入一些資料。 在開啟的命令提示字元視窗中，執行下列查詢以插入幾列資料：

```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

## <a name="query-and-update-the-data-in-the-tables"></a>查詢並更新資料表中的資料

執行下列查詢，以從資料庫資料表中擷取資訊：

```sql
SELECT * FROM inventory;
```

您也可以更新資料表中的資料：

```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

當您擷取資料時，資料列便會更新：

```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>將資料庫還原至先前的時間點

假設您不小心刪除重要的資料庫資料表，而無法輕易地復原資料。 在適用於 MariaDB 的 Azure 資料庫中，您可以在新的伺服器中建立資料庫複本，以將伺服器還原至某個時間點。 您可以使用這個新的伺服器來復原已刪除的資料。 下列步驟會將範例伺服器還原到新增資料表之前的時間點：

1. 在 Azure 入口網站中，找出您的「適用於 MariaDB 的 Azure 資料庫」。 在 [概觀] 頁面上，選取 [還原]。

   ![還原資料庫](./media/tutorial-design-database-using-portal/1-restore-a-db.png)

2. 在 [還原] 頁面上，輸入或選取下列資訊：
   
   ![還原表單](./media/tutorial-design-database-using-portal/2-restore-form.png)
   
   - **還原點**：在所列的時間範圍內，選取您想要還原的時間點。 務必將您的當地時區轉換成 UTC。
   - **還原到新伺服器**︰輸入要作為還原目的地的新伺服器名稱。
   - **位置**︰此區域與來源伺服器相同，無法加以變更。
   - **定價層**︰此定價層與來源伺服器相同，無法加以變更。
   
3. 選取 [確定]，將伺服器還原到資料表刪除之前的時間點[還原至某個時間點](./howto-restore-server-portal.md)。 還原伺服器可在您選取的時間點建立伺服器的新複本。 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您使用 Azure 入口網站來學習如何：

> [!div class="checklist"]
> * 建立適用於 MariaDB 的 Azure 資料庫
> * 設定伺服器防火牆
> * 使用 mysql 命令列工具來建立資料庫
> * 載入範例資料
> * 查詢資料
> * 更新資料
> * 還原資料

> [!div class="nextstepaction"]
> [如何將應用程式連線至適用於 MariaDB 的 Azure 資料庫](./howto-connection-string.md)
