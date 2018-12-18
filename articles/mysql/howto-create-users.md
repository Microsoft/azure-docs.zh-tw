---
title: 在適用於 MySQL 的 Azure 資料庫伺服器中建立使用者
description: 本文說明如何建立新的使用者帳戶，來與「適用於 MySQL 的 Azure 資料庫」伺服器互動。
services: mysql
author: jasonwhowell
ms.author: jasonh
editor: jasonwhowell
manager: kfile
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: ee74ea9e114f6401bfcafe44ca3caedfcd0005c5
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265613"
---
# <a name="create-users-in-azure-database-for-mysql-server"></a>在適用於 MySQL 的 Azure 資料庫伺服器中建立使用者 
本主題說明如在「適用於 MySQL 的 Azure 資料庫」伺服器中建立使用者。

當您第一次建立「適用於 MySQL 的 Azure 資料庫」時，您提供了伺服器管理員登入使用者名稱和密碼。 如需詳細資訊，您可以遵循[快速入門](quickstart-create-mysql-server-database-using-azure-portal.md)。 您可以從 Azure 入口網站找出您的伺服器管理員登入使用者名稱。

伺服器管理員使用者可取得您伺服器的某些權限，如下所列：SELECT、INSERT、UPDATE、DELETE、CREATE、DROP、RELOAD、PROCESS、REFERENCES、INDEX、ALTER、SHOW DATABASES、CREATE TEMPORARY TABLES、LOCK TABLES、EXECUTE、REPLICATION SLAVE、REPLICATION CLIENT、CREATE VIEW、SHOW VIEW、CREATE ROUTINE、ALTER ROUTINE、CREATE USER、EVENT、TRIGGER

建立「適用於 MySQL 的 Azure 資料庫」伺服器之後，您可以使用第一個伺服器管理員使用者帳戶，建立其他使用者並授與系統管理員存取權。 此外，伺服器管理員帳戶可以用來建立較低權限的使用者，以存取個別資料庫結構描述。

## <a name="how-to-create-additional-admin-users-in-azure-database-for-mysql"></a>如何在適用於 MySQL 的 Azure 資料庫中建立其他系統管理員使用者
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

## <a name="how-to-create-database-users-in-azure-database-for-mysql"></a>如何在適用於 MySQL 的 Azure 資料庫中建立資料庫使用者

1. 取得連線資訊和管理員使用者名稱。
   若要連線到您的資料庫伺服器，您需要完整伺服器名稱和系統管理員登入認證。 您可以從 Azure 入口網站的伺服器 [概觀] 或 [屬性] 頁面輕鬆尋找伺服器名稱和登入資訊。 

2. 使用系統管理員帳戶和密碼來連線到資料庫伺服器。 使用您慣用的用戶端工具，例如 MySQL Workbench、mysql.exe、HeidiSQL 或其他工具。 
   如果您不確定如何連線，請參閱[使用 MySQL Workbench 來連線及查詢資料](./connect-workbench.md)

3. 編輯並執行下列 SQL 程式碼。 將預留位置值 `db_user` 替換為您預定的新使用者名稱，並將預留位置值 `testdb` 替換為您自己的資料庫名稱。

   這個 sql 程式碼語法會建立名為 testdb 的新資料庫作為範例。 然後在 MySQL 服務中建立新的使用者，並將所有特殊權限授與給該使用者的新資料庫結構描述 (testdb.\*)。 

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

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```

## <a name="next-steps"></a>後續步驟
針對新使用者機器的 IP 位址開啟防火牆，讓使用者能夠連線：[使用 Azure 入口網站建立及管理適用於 MySQL 的 Azure 資料庫防火牆規則](howto-manage-firewall-using-portal.md)或 [Azure CLI](howto-manage-firewall-using-cli.md)。

如需有關使用者帳戶管理的詳細資訊，請參閱 MySQL 產品文件中的[使用者帳戶管理](https://dev.mysql.com/doc/refman/5.7/en/user-account-management.html)、[GRANT 語法](https://dev.mysql.com/doc/refman/5.7/en/grant.html)和[特殊權限](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)。
