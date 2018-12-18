---
title: 在適用於 MariaDB 的 Azure 資料庫伺服器中建立使用者
description: 本文描述如何建立新的使用者帳戶，來與「適用於 MariaDB 的 Azure 資料庫」伺服器互動。
author: jasonwhowell
ms.author: jasonh
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 50154a7fee63eb3ff9e08155123f9e5962bbfcf0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946103"
---
# <a name="create-users-in-azure-database-for-mariadb"></a>在適用於 MariaDB 的 Azure 資料庫中建立使用者 
本文描述如何在適用於 MariaDB 的 Azure 資料庫中建立使用者。

當您第一次建立適用於 MariaDB 的 Azure 資料庫時，您提供了伺服器管理員登入使用者名稱和密碼。 如需詳細資訊，您可以遵循[快速入門](quickstart-create-mariadb-server-database-using-azure-portal.md)。 您可以從 Azure 入口網站找出您的伺服器管理員登入使用者名稱。

伺服器管理員使用者可取得您伺服器的某些權限，如下所列：SELECT、INSERT、UPDATE、DELETE、CREATE、DROP、RELOAD、PROCESS、REFERENCES、INDEX、ALTER、SHOW DATABASES、CREATE TEMPORARY TABLES、LOCK TABLES、EXECUTE、REPLICATION SLAVE、REPLICATION CLIENT、CREATE VIEW、SHOW VIEW、CREATE ROUTINE、ALTER ROUTINE、CREATE USER、EVENT、TRIGGER

建立適用於 MariaDB 的 Azure 資料庫伺服器之後，您可以使用第一個伺服器管理員使用者帳戶，建立其他使用者並授與其管理員存取權。 此外，伺服器管理員帳戶可以用來建立較低權限的使用者，以存取個別資料庫結構描述。

## <a name="create-additional-admin-users"></a>建立其他管理使用者
1. 取得連線資訊和管理員使用者名稱。
   若要連線到您的資料庫伺服器，您需要完整伺服器名稱和系統管理員登入認證。 您可以從 Azure 入口網站的伺服器 [概觀] 或 [屬性] 頁面輕鬆尋找伺服器名稱和登入資訊。 

2. 使用系統管理員帳戶和密碼來連線到資料庫伺服器。 使用您慣用的用戶端工具，例如 MySQL Workbench、mysql.exe、HeidiSQL 或其他工具。 
   如果您不確定如何連線，請參閱[使用 MySQL Workbench 來連線及查詢資料](./connect-workbench.md)

3. 編輯並執行下列 SQL 程式碼。 將預留位置值 `new_master_user` 替換為您的新使用者名稱。 此語法可將所有資料庫結構描述上列出的特殊權限 (*.*) 授與給使用者名稱 (在此範例中為 new_master_user)。 

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION; 
   
   FLUSH PRIVILEGES;
   ```

4. 確認授與 
   ```sql
   USE sys;
   
   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="create-database-users"></a>建立資料庫使用者

1. 取得連線資訊和管理員使用者名稱。
   若要連線到您的資料庫伺服器，您需要完整伺服器名稱和系統管理員登入認證。 您可以從 Azure 入口網站的伺服器 [概觀] 或 [屬性] 頁面輕鬆尋找伺服器名稱和登入資訊。 

2. 使用系統管理員帳戶和密碼來連線到資料庫伺服器。 使用您慣用的用戶端工具，例如 MySQL Workbench、mysql.exe、HeidiSQL 或其他工具。 
   如果您不確定如何連線，請參閱[使用 MySQL Workbench 來連線及查詢資料](./connect-workbench.md)

3. 編輯並執行下列 SQL 程式碼。 將預留位置值 `db_user` 替換為您預定的新使用者名稱，並將預留位置值 `testdb` 替換為您自己的資料庫名稱。

   這個 sql 程式碼語法會建立名為 testdb 的新資料庫作為範例。 然後在適用於 MariaDB 的 Azure 資料庫服務中建立新的使用者，並將所有特殊權限授與給該使用者的新資料庫結構描述 (testdb.\*)。 

   ```sql
   CREATE DATABASE testdb;
   
   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';
   
   FLUSH PRIVILEGES;
   ```

4. 確認資料庫中的授與。
   ```sql
   USE testdb;
   
   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. 指定委派的資料庫，使用新的使用者名稱和密碼來登入伺服器。 此範例會顯示 mysql 命令列。 執行此命令時，系統會提示您輸入使用者名稱的密碼。 取代您自己的伺服器名稱、資料庫名稱和使用者名稱。

   ```bash
   mysql --host mydemoserver.mariadb.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```
如需使用者帳戶管理的詳細資訊，請參閱 MariaDB 文件中的[使用者帳戶管理](https://mariadb.com/kb/en/library/user-account-management/)、[GRANT 語法](https://mariadb.com/kb/en/library/grant/)和[權限](https://mariadb.com/kb/en/library/grant/#privilege-levels)。

## <a name="next-steps"></a>後續步驟
針對新使用者電腦的 IP 位址開啟防火牆，讓使用者能夠連線：[使用 Azure 入口網站建立及管理適用於 MariaDB 的 Azure 資料庫防火牆規則](howto-manage-firewall-portal.md)  

<!--or [Azure CLI](howto-manage-firewall-using-cli.md).-->
