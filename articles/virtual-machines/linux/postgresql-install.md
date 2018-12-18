---
title: 設定 Linux VM 上的 PostgreSQL | Microsoft Docs
description: 了解如何在 Azure 中的 Linux 虛擬機器上安裝和設定 PostgreSQL
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: 1a747363-0cc5-4ba3-9be7-084dfeb04651
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: cynthn
ms.openlocfilehash: 903e94cfa932ddd93a931caa8888d93f1bdfe365
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932726"
---
# <a name="install-and-configure-postgresql-on-azure"></a>安裝和設定 Azure 上的 PostgreSQL
PostgreSQL 是與 Oracle 和 DB2 類似的進階開放原始碼資料庫。 它包含企業用功能，例如完整的 ACID 的相容性、可靠的交易式程序，以及多版本的並行控制。 它也支援標準，例如 ANSI SQL 和 SQL/MED (包括 Oracle、MySQL、MongoDB 和許多其他項目的外部資料包裝函式)。 其高度可擴充性支援超過 12 種程序性語言、GIN 和 GiST 索引、空間資料支援和多個類似 NoSQL 的功能，適用於 JSON 或以索引鍵-值為基礎的應用程式。

在本文中，您將學習如何在執行 Linux 的 Azure 虛擬機器上安裝和設定 PostgreSQL。

[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="install-postgresql"></a>安裝 PostgreSQL
> [!NOTE]
> 您必須已經具有執行 Linux 的 Azure 虛擬機器，才能完成本教學課程。 若要建立並設定 Linux VM 再繼續進行，請參閱 [Azure Linux VM 教學課程](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> 
> 

在此情況下，使用連接埠 1999 做為 PostgreSQL 連接埠。  

連接到您透過 PuTTY 建立的 Linux VM。 如果這是您第一次使用 Azure Linux VM，請參閱[如何搭配 Azure 上的 Linux 使用 SSH](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 以了解如何使用 PuTTY 來連接到 Linux VM。

1. 執行下列命令來切換至 root (admin)：
   
        # sudo su -
2. 某些散發套件有相依性，您必須先安裝這些相依性再安裝 PostgreSQL。 檢查此清單中的 distro 並執行適當的命令：
   
   * Red Hat 基底 Linux：
     
           # yum install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam  libxslt-devel tcl-devel python-devel -y  
   * Debian 基底 Linux：
     
            # apt-get install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam libxslt-devel tcl-devel python-devel -y  
   * SUSE Linux：
     
           # zypper install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam  libxslt-devel tcl-devel python-devel -y  
3. 下載 PostgreSQL 到根目錄，接著將封裝解壓縮：
   
        # wget https://ftp.postgresql.org/pub/source/v9.3.5/postgresql-9.3.5.tar.bz2 -P /root/
   
        # tar jxvf  postgresql-9.3.5.tar.bz2
   
    以上是範例。 您可以在 [Index of /pub/source/](https://ftp.postgresql.org/pub/source/)中找到更詳細的下載位址。
4. 若要啟動組建，請執行以下命令：
   
        # cd postgresql-9.3.5
   
        # ./configure --prefix=/opt/postgresql-9.3.5
5. 如果您想要建置所有可以建置的項目 (包括文件 (HTML 和 man 頁面) 和其他模組 (contrib))，請改為執行下列命令：
   
        # gmake install-world
   
    您會收到下列確認訊息：
   
        PostgreSQL, contrib, and documentation successfully made. Ready to install.

## <a name="configure-postgresql"></a>設定 PostgreSQL
1. (選擇性) 建立符號連結來縮短 PostgreSQL 參考，使其不包含版本號碼：
   
        # ln -s /opt/pgsql9.3.5 /opt/pgsql
2. 建立資料庫的目錄：
   
        # mkdir -p /opt/pgsql_data
3. 建立非根使用者，並修改該使用者的設定檔。 然後切換到這個新的使用者 (在我們的範例中稱為 *postgres* )：
   
        # useradd postgres
   
        # chown -R postgres.postgres /opt/pgsql_data
   
        # su - postgres
   
   > [!NOTE]
   > 基於安全性理由，PostgreSQL 會使用非根使用者初始化、啟動或關閉資料庫。
   > 
   > 
4. 輸入下列命令以編輯 bash_profile 檔。 這幾行將會加入至 bash_profile 檔案的結尾：
   
        cat >> ~/.bash_profile <<EOF
        export PGPORT=1999
        export PGDATA=/opt/pgsql_data
        export LANG=en_US.utf8
        export PGHOME=/opt/pgsql
        export PATH=\$PATH:\$PGHOME/bin
        export MANPATH=\$MANPATH:\$PGHOME/share/man
        export DATA=`date +"%Y%m%d%H%M"`
        export PGUSER=postgres
        alias rm='rm -i'
        alias ll='ls -lh'
        EOF
5. 執行 bash_profile 檔案：
   
        $ source .bash_profile
6. 利用下列命令驗證安裝：
   
        $ which psql
   
    如果您成功安裝，您將會看見下列回應：
   
        /opt/pgsql/bin/psql
7. 您也可以檢查 PostgreSQL 版本：
   
        $ psql -V
8. 初始化資料庫：
   
        $ initdb -D $PGDATA -E UTF8 --locale=C -U postgres -W
   
    您應該會收到下列輸出：

![映像](./media/postgresql-install/no1.png)

## <a name="set-up-postgresql"></a>設定 PostgreSQL
<!--    [postgres@ test ~]$ exit -->

執行下列命令：

    # cd /root/postgresql-9.3.5/contrib/start-scripts

    # cp linux /etc/init.d/postgresql

修改 /etc/init.d/postgresql 檔案中的兩個變數。 前置詞設為 PostgreSQL 的安裝路徑： **/opt/pgsql**。 PGDATA 設為 PostgreSQL 的資料儲存路徑：**/opt/pgsql_data**。

    # sed -i '32s#usr/local#opt#' /etc/init.d/postgresql

    # sed -i '35s#usr/local/pgsql/data#opt/pgsql_data#' /etc/init.d/postgresql

![映像](./media/postgresql-install/no2.png)

變更檔案，使其可執行：

    # chmod +x /etc/init.d/postgresql

啟動 PostgreSQL：

    # /etc/init.d/postgresql start

查看 PostgreSQL 端點是否位於：

    # netstat -tunlp|grep 1999

您應該會看見下列輸出：

![映像](./media/postgresql-install/no3.png)

## <a name="connect-to-the-postgres-database"></a>連接到 Postgres 資料庫
再一次切換到 postgres 使用者：

    # su - postgres

建立 Postgres 資料庫：

    $ createdb events

連接到您剛建立的事件資料庫：

    $ psql -d events

## <a name="create-and-delete-a-postgres-table"></a>建立和刪除 Postgres 資料表
既然您已經連接到資料庫，可以在其中建立資料表。

例如，利用下列命令建立新的範例 Postgres 資料表：

    CREATE TABLE potluck (name VARCHAR(20),    food VARCHAR(30),    confirmed CHAR(1), signup_date DATE);

您現在已經利用下列資料行名稱和限制設定了 4 個資料行的資料表：

1. “name” 資料行已經由 VARCHAR 命令限制在 20 個字元以下。
2. "Food" 資料行指出每個人將會攜帶的食物項目。 VARCHAR 將此文字限制在 30 個字元以下。
3. “confirmed” 資料行記錄該人員是否具有聚會的 RSVP'd。 可接受的值為 "Y" 和 "N"。
4. “date” 資料行顯示他們註冊此事件的時間。 Postgres 的必要日期格式為 yyyy-mm-dd。

如果您成功建立資料表，您會看到下列內容：

![映像](./media/postgresql-install/no4.png)

您也可以利用下列命令檢查資料表結構：

![映像](./media/postgresql-install/no5.png)

### <a name="add-data-to-a-table"></a>將資料新增至資料表
首先，將資料插入資料列：

    INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('John', 'Casserole', 'Y', '2012-04-11');

您應該會看見此輸出：

![映像](./media/postgresql-install/no6.png)

您也可以將更多人員新增至資料表。 以下是一些選項，或者您可以建立自己的選項：

    INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('Sandy', 'Key Lime Tarts', 'N', '2012-04-14');

    INSERT INTO potluck (name, food, confirmed, signup_date) VALUES ('Tom', 'BBQ','Y', '2012-04-18');

    INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('Tina', 'Salad', 'Y', '2012-04-18');

### <a name="show-tables"></a>顯示資料表
使用下列命令來顯示資料表：

    select * from potluck;

輸出如下：

![映像](./media/postgresql-install/no7.png)

### <a name="delete-data-in-a-table"></a>刪除資料表中的資料
使用下列命令刪除資料表中的資料：

    delete from potluck where name=’John’;

這會刪除 "John" 資料列中的所有資訊。 輸出如下：

![映像](./media/postgresql-install/no8.png)

### <a name="update-data-in-a-table"></a>更新資料表中的資料
使用下列命令更新資料表中的資料。 對於此項目，Sandy 已確認她會參加，所以我們要將她的 RSVP 從 "N" 變更為 "Y"：

     UPDATE potluck set confirmed = 'Y' WHERE name = 'Sandy';


## <a name="get-more-information-about-postgresql"></a>取得 PostgreSQL 的詳細資訊
既然您已完成在 Azure Linux VM 中的 PostgreSQL 安裝，您可以在 Azure 中享用它。 若要深入了解 PostgreSQL，請造訪 [PostgreSQL 網站](http://www.postgresql.org/)。

