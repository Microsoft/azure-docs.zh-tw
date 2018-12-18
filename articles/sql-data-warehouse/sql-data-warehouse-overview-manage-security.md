---
title: 保護 SQL 資料倉儲中的資料庫 | Microsoft Docs
description: 保護 Azure SQL 資料倉儲中的資料庫以便開發解決方案的秘訣。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 8abb40b0c1a5b9cd3f8d1e23124090c00e8cfadb
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302534"
---
# <a name="secure-a-database-in-sql-data-warehouse"></a>保護 SQL 資料倉儲中的資料庫
> [!div class="op_single_selector"]
> * [安全性概觀](sql-data-warehouse-overview-manage-security.md)
> * [驗證](sql-data-warehouse-authentication.md)
> * [加密 (入口網站)](sql-data-warehouse-encryption-tde.md)
> * [加密 (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)
> 
> 

本文逐步解說保護 Azure SQL 資料倉儲資料庫的基本概念。 本文尤其著重於協助您開始利用資源，以在資料庫上限制存取、保護資料，以及監視活動。

## <a name="connection-security"></a>連接安全性
「連線安全性」是指如何使用防火牆規則和連線加密，限制和保護資料庫的連線。

伺服器和資料庫兩者都可使用防火牆規則，拒絕來自未明確設為允許清單的 IP 位址的連線嘗試。 若要允許來自應用程式或用戶端機器之公用 IP 位址的連線，您必須先使用 Azure 入口網站、REST API 或 PowerShell 建立伺服器層級的防火牆規則。 最好的作法是，您應該盡可能限制允許穿透您伺服器防火牆的 IP 位址範圍。  若要從您的本機電腦存取 Azure SQL 資料倉儲，請確定您的網路和本機電腦上的防火牆允許 TCP 連接埠 1433 上的傳出通訊。  

SQL 資料倉儲會使用伺服器層級的防火牆規則。 它不支援資料庫層級的防火牆規則。 如需詳細資訊，請參閱 [Azure SQL Database 防火牆][Azure SQL Database firewall]、[sp_set_firewall_rule][sp_set_firewall_rule]。

預設會加密與 SQL 資料倉儲的連線。  停用加密的修改連線設定會被忽略。

## <a name="authentication"></a>驗證
「驗證」是指連線到資料庫時如何證明身分識別。 SQL 資料倉儲目前支援採用使用者名稱和密碼，以及 Azure Active Directory 的 SQL Server 驗證。 

當您為資料庫建立邏輯伺服器時，採取使用者名稱和密碼指定了「伺服器管理員」登入。 使用這些認證，您就可以透過 SQL Server 驗證，使用資料庫擁有者或 "dbo" 的身分驗證該伺服器上的任何資料庫。

不過，最佳作法是，貴組織的使用者應該使用不同的帳戶來驗證。 因為萬一應用程式的程式碼容易受到 SQL 插入式攻擊，您就可以限制授與應用程式的權限，並降低惡意活動的風險。 

若要建立 SQL Server 驗證使用者，請使用伺服器管理員登入連接到您伺服器上的 **master** 資料庫，並建立新的伺服器登入。  此外，在主要資料庫中建立一個使用者做為 Azure SQL 資料倉儲使用者是不錯的主意。 在主要資料庫中建立使用者，可讓使用者使用類似 SSMS 的工具登入，而不用指定資料庫名稱。  它也可讓使用者使用物件總管來檢視 SQL Server 上的所有資料庫。

```sql
-- Connect to master database and create a login
CREATE LOGIN ApplicationLogin WITH PASSWORD = 'Str0ng_password';
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

然後，使用您的伺服器管理員登入連線到 **SQL 資料倉儲資料庫** ，並根據您所建立的伺服器登入建立資料庫使用者。

```sql
-- Connect to SQL DW database and create a database user
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

若要提供使用者執行其他作業 (例如建立登入或建立新資料庫) 的權限，請為他們指派主要資料庫中的 `Loginmanager` 和 `dbmanager` 角色。 如需有關這些額外角色及向 SQL Database 驗證的詳細資訊，請參閱[管理 Azure SQL Database 的資料庫和登入][Managing databases and logins in Azure SQL Database]。  如需詳細資訊，請參閱[使用 Azure Active Directory 驗證連線到 SQL 資料倉儲][Connecting to SQL Data Warehouse By Using Azure Active Directory Authentication]。

## <a name="authorization"></a>Authorization
授權是指您在 Azure SQL 資料倉儲資料庫內可以執行的動作。 授權權限取決於角色成員資格和權限。 最好的作法是，您應該授與使用者所需的最低權限。 若要管理角色，您可以使用下列預存程序：

```sql
EXEC sp_addrolemember 'db_datareader', 'ApplicationUser'; -- allows ApplicationUser to read data
EXEC sp_addrolemember 'db_datawriter', 'ApplicationUser'; -- allows ApplicationUser to write data
```

您所連線的伺服器管理員帳戶是 db_owner 的成員，有權限在資料庫中執行任何動作。 請儲存此帳戶，以便部署結構描述升級及其他管理作業。 請使用具更多有限權限的 "ApplicationUser" 帳戶，從應用程式連線到具應用程式所需之最低權限的資料庫。

有許多方式可以進一步限制使用者透過 Azure SQL 資料倉儲可以執行的動作：

* 細微的[權限][Permissions]可讓您控制可對資料庫中的個別資料行、資料表、檢視、結構描述、程序和其他物件執行哪些作業。 使用細微權限，以擁有最大控制權，並授與所需的最小權限。 
* 除了 db_datareader 和 db_datawriter 以外，[資料庫角色][Database roles]均可以用來建立權力較大的應用程式使用者帳戶，或權力較小的管理帳戶。 內建固定資料庫角色提供簡單的方式來授與權限，但可能會導致授與的權限多於所需的權限。
* [預存程序][Stored procedures]可用來限制對資料庫可採取的動作。

下列範例會針對使用者定義結構描述授予讀取權限。
```sql
--CREATE SCHEMA Test
GRANT SELECT ON SCHEMA::Test to ApplicationUser
```

從 Azure 入口網站或使用 Azure Resource Manager API 管理資料庫和邏輯伺服器的能力，是由您入口網站使用者帳戶的角色指派所控制。 如需詳細資訊，請參閱 [Azure 入口網站中的角色型存取控制][Role-based access control in Azure portal]。

## <a name="encryption"></a>加密
Azure SQL 資料倉儲透明資料加密 (TDE) 可以對待用資料加密和解密，協助防止惡意活動的威脅。  當您加密資料庫時，相關聯的備份和交易記錄檔就會加密，完全不需要變更您的應用程式。 TDE 會使用稱為資料庫加密金鑰的對稱金鑰來加密整個資料庫的儲存體。 

在 SQL Database 中，資料庫加密金鑰是由內建伺服器憑證保護。 內建伺服器憑證對每個 SQL Database 伺服器都是唯一的。 Microsoft 至少每 90 天會自動替換這些憑證。 「SQL 資料倉儲」使用的加密演算法是 AES-256。 如需 TDE 的一般描述，請參閱[透明資料加密][Transparent Data Encryption]。

您可以使用 [Azure 入口網站][Encryption with Portal]或 [T-SQL][Encryption with TSQL] 將資料庫加密。

## <a name="next-steps"></a>後續步驟
如需使用不同通訊協定連接到您的 SQL 資料倉儲的詳細資訊和範例，請參閱[連接到 SQL 資料倉儲][Connect to SQL Data Warehouse]。

<!--Image references-->

<!--Article references-->
[Connect to SQL Data Warehouse]: ./sql-data-warehouse-connect-overview.md
[Encryption with Portal]: ./sql-data-warehouse-encryption-tde.md
[Encryption with TSQL]: ./sql-data-warehouse-encryption-tde-tsql.md
[Connecting to SQL Data Warehouse By Using Azure Active Directory Authentication]: ./sql-data-warehouse-authentication.md

<!--MSDN references-->
[Azure SQL Database firewall]: https://msdn.microsoft.com/library/ee621782.aspx
[sp_set_firewall_rule]: https://msdn.microsoft.com/library/dn270017.aspx
[sp_set_database_firewall_rule]: https://msdn.microsoft.com/library/dn270010.aspx
[Database roles]: https://msdn.microsoft.com/library/ms189121.aspx
[Managing databases and logins in Azure SQL Database]: https://msdn.microsoft.com/library/ee336235.aspx
[Permissions]: https://msdn.microsoft.com/library/ms191291.aspx
[Stored procedures]: https://msdn.microsoft.com/library/ms190782.aspx
[Transparent Data Encryption]: https://msdn.microsoft.com/library/bb934049.aspx
[Azure portal]: https://portal.azure.com/

<!--Other Web references-->
[Role-based access control in Azure portal]: https://azure.microsoft.com/documentation/articles/role-based-access-control-configure
