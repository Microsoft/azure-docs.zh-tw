---
title: 包含檔案
description: 包含檔案
services: logic-apps
author: ecfan
ms.service: logic-apps
ms.topic: include
ms.date: 05/15/2018
ms.author: estfan
ms.custom: include file
ms.openlocfilehash: 4ffda692da0ab7b63f7376c36dfab0bec914e334
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37138060"
---
* 如果您是使用 Azure SQL Database，請遵循[連線到 Azure SQL Database](#connect-azure-sql-db) 下的步驟。 

* 如果您是使用 SQL Server，請遵循[連線到 SQL Server](#connect-sql-server) 下的步驟。

<a name="connect-azure-sql-db"></a>

### <a name="connect-to-azure-sql-database"></a>連線到 Azure SQL Database

1. 當 SQL 觸發程序或動作提示您輸入連線資訊時，請遵循下列步驟：

   1. 為您的連線建立一個名稱。

   2. 選取您的 SQL 伺服器，然後選取資料庫。 

      只在您選取 SQL 伺服器之後，才會顯示資料庫清單。
 
   3. 提供伺服器的使用者名稱和密碼。

      您可以在 Azure 入口網站的 SQL 資料庫屬性下，或在您的連接字串中找到此資訊： 
      
      "User ID=<yourUserName>"
      <br>
      "Password=<yourPassword>"

   本範例顯示的是觸發程序的連線資訊，但這些步驟也適用於動作。

   ![建立 Azure SQL Database 連線](./media/connectors-create-api-sqlazure/azure-sql-database-create-connection.png)
   <br>
   星號 (*) 表示必要值。

   | 屬性 | 值 | 詳細資料 | 
   |----------|-------|---------| 
   | 連線名稱 | <*my-sql-connection*> | 您連線的名稱 | 
   | SQL Server 名稱 | <*my-sql-server*> | 您 SQL 伺服器的名稱 |
   | SQL Database 名稱 | <*my-sql-database*>  | 您 SQL 資料庫的名稱 | 
   | 使用者名稱 | <*my-sql-username*> | 用以存取您資料庫的使用者名稱 |
   | 密碼 | <*my-sql-password*> | 用以存取您資料庫的密碼 | 
   |||| 

2. 完成之後，請選擇 [建立]。

3. 您在建立連線之後，請繼續[新增 SQL 觸發程序](#add-sql-trigger)或[新增 SQL 動作](#add-sql-action)。

<a name="connect-sql-server"></a>

### <a name="connect-to-sql-server"></a>連接到 SQL Server

您在選取閘道之前，請確定您已[設定您的資料閘道](https://docs.microsoft.com/azure/logic-apps/logic-apps-gateway-connection)。 這樣一來，當您建立連線時，您的閘道就會出現在閘道清單中。

1. 當 SQL 觸發程序或動作提示您輸入連線資訊時，請遵循下列步驟：

   1. 在觸發程序或動作中，選取**透過內部部署資料閘道連線**，如此就會顯示 SQL 伺服器選項。

   2. 為您的連線建立一個名稱。

   3. 提供您 SQL 伺服器的位址，然後提供您資料庫的名稱。
   
      您可以在連接字串中找到這項資訊： 
      
      * "Server=<*yourServerAddress*>"
      * "Database=<*yourDatabaseName*>"

   4. 提供伺服器的使用者名稱和密碼。

      您可以在連接字串中找到這項資訊： 
      
      * "User ID=<yourUserName>"
      * "Password=<yourPassword>"

   5. 如果您的 SQL 伺服器是使用 Windows 或基本驗證，請選取驗證類型。

   6. 選取您先前所建立的內部部署資料閘道名稱。
   
      如果您的閘道未出現在清單中，請檢查您是否正確[設定閘道](https://docs.microsoft.com/azure/logic-apps/logic-apps-gateway-connection)。

   本範例顯示的是觸發程序的連線資訊，但這些步驟也適用於動作。

   ![建立 SQL Server 連線](./media/connectors-create-api-sqlazure/sql-server-create-connection.png)
   <br>
   星號 (*) 表示必要值。

   | 屬性 | 值 | 詳細資料 | 
   |----------|-------|---------| 
   | 透過內部部署閘道連線 | 請先針對 SQL Server 設定選取此選項。 | | 
   | 連線名稱 | <*my-sql-connection*> | 您連線的名稱 | 
   | SQL Server 名稱 | <*my-sql-server*> | 您 SQL 伺服器的名稱 |
   | SQL Database 名稱 | <*my-sql-database*>  | 您 SQL 資料庫的名稱 |
   | 使用者名稱 | <*my-sql-username*> | 用以存取您資料庫的使用者名稱 |
   | 密碼 | <*my-sql-password*> | 用以存取您資料庫的密碼 | 
   | 驗證類型 | Windows 或基本 | 選用：您 SQL 伺服器所使用的驗證類型 | 
   | 閘道 | <*my-data-gateway*> | 您內部部署資料閘道的名稱 | 
   |||| 

2. 完成之後，請選擇 [建立]。 

3. 您在建立連線之後，請繼續[新增 SQL 觸發程序](#add-sql-trigger)或[新增 SQL 動作](#add-sql-action)。
