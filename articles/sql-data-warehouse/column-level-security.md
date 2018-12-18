---
title: Azure SQL 資料倉儲的資料行層級安全性 | Microsoft Docs
description: 資料行層級安全性 (CLS) 讓客戶能夠根據使用者的執行內容或其群組成員資格來控制資料庫資料表資料行的存取。 CLS 可簡化應用程式中安全性的設計和編碼。 CLS 可讓您實作資料行存取的限制。
services: sql-data-warehouse
author: KavithaJonnakuti
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 06/15/2018
ms.author: kavithaj
ms.reviewer: igorstan, carlrab
ms.openlocfilehash: 1765c92ad10fa35af98e7c7314eb44c3a119f422
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301050"
---
# <a name="column-level-security"></a>資料行層級安全性 
資料行層級安全性 (CLS) 讓客戶能夠根據使用者的執行內容或其群組成員資格來控制資料庫資料表資料行的存取。  

> [!VIDEO https://www.youtube.com/embed/OU_ESg0g8r8]

CLS 可簡化應用程式中安全性的設計和編碼。 CLS 可讓您實作資料行存取的限制來保護機密資料。 例如，確保特定使用者只能存取其部門相關資料表的特定資料行。 存取限制邏輯位於資料庫層，而不是遠離另一個應用程式層中的資料。 資料庫會在每次嘗試從任何層級存取該資料時套用存取限制。 此限制可藉由縮小整個安全性系統的介面區，讓安全性系統更加可靠和健全。 此外，CLS 也不需引進檢視以篩選出可用來將存取限制加諸於使用者的資料行。 

您可以使用 [GRANT](https://docs.microsoft.com/sql/t-sql/statements/grant-transact-sql) T-SQL 陳述式來實作 CLS。 透過這個機制，即可支援 SQL 和 Azure Active Directory (AAD) 驗證。

![cls](./media/column-level-security/cls.png)

## <a name="syntax"></a>語法 

```sql
GRANT <permission> [ ,...n ] ON    
    [ OBJECT :: ][ schema_name ]. object_name [ ( column [ ,...n ] ) ]   
    TO <database_principal> [ ,...n ]    
    [ WITH GRANT OPTION ]   
    [ AS <database_principal> ]   
<permission> ::=   
    SELECT 
  | UPDATE  
<database_principal> ::=    
      Database_user    
    | Database_role    
    | Database_user_mapped_to_Windows_User    
    | Database_user_mapped_to_Windows_Group    
```

## <a name="example"></a>範例 
下列範例示範如何限制 'TestUser' 存取 ‘Membership’ 資料表的 'SSN' 資料行： 

建立 ‘Membership’ 資料表，其中含有用來儲存社會安全號碼的 SSN 資料行：

```sql
CREATE TABLE Membership   
  (MemberID int IDENTITY,   
   FirstName varchar(100) NULL,   
   SSN char(9) NOT NULL, 
   LastName varchar(100) NOT NULL,   
   Phone varchar(12) NULL,   
   Email varchar(100) NULL);  
```

允許 'TestUser' 存取含機密資料之 SSN 資料行以外的所有資料行： 

```sql  
GRANT SELECT ON Membership(MemberID, FirstName, LastName, Phone, Email) TO TestUser;   
``` 

如果以 ‘TestUser’ 執行的查詢中包含 SSN 資料行，則它們將會失敗：

```sql  
SELECT * FROM Membership;

Msg 230, Level 14, State 1, Line 12
The SELECT permission was denied on the column 'SSN' of the object 'Membership', database 'CLS_TestDW', schema 'dbo'. 
``` 

## <a name="use-cases"></a>使用案例
一些目前使用 CLS 的範例： 
- 某家金融服務公司只允許帳戶管理員能夠存取客戶社會安全號碼 (SSN)、電話號碼和其他個人識別資訊 (PII)。
- 某家醫療保健供應商允許醫生和護士存取機密的醫療記錄，而不允許帳務部門的成員檢視此資料。
