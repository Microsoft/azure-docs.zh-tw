---
title: Azure Data Factory 中的可重複複製 | Microsoft Docs
description: 了解如何在即使將複製資料的配量執行多次的情況下，也能避免產生重複的項目。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 8b1d1cf62dad94ca6141ce33783c0b16b50c931c
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34622630"
---
# <a name="repeatable-copy-in-azure-data-factory"></a>Azure Data Factory 中的可重複複製

## <a name="repeatable-read-from-relational-sources"></a>從關聯式來源進行可重複的讀取
從關聯式資料存放區複製資料時，請將可重複性謹記在心，以避免產生非預期的結果。 在 Azure Data Factory 中，您可以手動重新執行配量。 您也可以為資料集設定重試原則，使得在發生失敗時，重新執行配量。 以上述任一方式重新執行配量時，您必須確保不論將配量執行多少次，都會讀取相同的資料。  
 
> [!NOTE]
> 下面的範例是針對 Azure SQL，但也適用於任何支援矩形資料集的資料存放區。 您可能必須針對資料存放區調整來源的**類型**和 **query** 屬性 (例如：query 而不是 sqlReaderQuery)。   

通常從關聯式存放區進行讀取時，您會希望只讀取與該配量對應的資料。 有一個可達到此目的的方法，就是使用 Azure Data Factory 中提供的 WindowStart 和 WindowEnd 系統變數。 若要了解 Azure Data Factory 中的變數與函式，請參閱 [Azure Data Factory - 函式與系統變數](data-factory-functions-variables.md)一文。 範例： 

```json
"source": {
    "type": "SqlSource",
    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm\\'', WindowStart, WindowEnd)"
},
```
此查詢會從 MyTable 資料表中，讀取配量持續時間範圍內 (WindowStart -> WindowEnd) 的資料。 重新執行此配量也會一律確保讀取相同的資料。 

在其他情況下，您可能會想要閱讀整份資料表，而可以依下列方式定義 sqlReaderQuery：

```json
"source": 
{            
    "type": "SqlSource",
    "sqlReaderQuery": "select * from MyTable"
},
```

## <a name="repeatable-write-to-sqlsink"></a>對 SqlSink 進行可重複的寫入
將資料從其他資料存放區複製到 **Azure SQL/SQL Server** 時，您需將可重複性謹記在心，以避免產生非預期的結果。 

將資料複製到「Azure SQL/SQL Server 資料庫」時，複製活動預設會將資料附加至接收資料表。 假設您要將資料從包含兩筆記錄的 CSV (逗點分隔值) 檔案複製到「Azure SQL/SQL Server 資料庫」中的下列資料表。 當配量執行時，會將該兩筆記錄複製到 SQL 資料表中。 

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
```

假設您在來源檔案中發現錯誤，並將 Down Tube 的數量從 2 更新為 4。 如果您手動重新執行該期間的資料配量，您將會發現有兩筆新記錄附加到「Azure SQL/SQL Server 資料庫」。 此範例是假設資料表中的所有資料行都沒有主索引鍵條件約束。

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    2            2015-05-01 00:00:00
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

若要避免此行為，您必須使用以下兩個機制其中之一來指定 UPSERT 語意：

### <a name="mechanism-1-using-sqlwritercleanupscript"></a>機制 1：使用 sqlWriterCleanupScript
您可以在執行配量時，先使用 **sqlWriterCleanupScript** 屬性將資料從接收資料表中清除，然後才插入資料。 

```json
"sink":  
{ 
  "type": "SqlSink", 
  "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM table WHERE ModifiedDate >= \\'{0:yyyy-MM-dd HH:mm}\\' AND ModifiedDate < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
}
```

當配量執行時，會先執行清除指令碼，從 SQL 資料表中刪除與該配量對應的資料。 複製活動會接著將資料插入到 SQL 資料表中。 如果重新執行配量，將會視需要更新數量。

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
6    Flat Washer    3            2015-05-01 00:00:00
7     Down Tube    4            2015-05-01 00:00:00
```

假設原始 csv 中的 Flat Washer 記錄被移除。 則重新執行配量會產生以下結果： 

```
ID    Product        Quantity    ModifiedDate
...    ...            ...            ...
7     Down Tube    4            2015-05-01 00:00:00
```

複製活動執行清除指令碼來刪除該配量的對應資料。 接著，它會從 csv (只包含一筆記錄) 讀取輸入，並將它插入到資料表中。 

### <a name="mechanism-2-using-sliceidentifiercolumnname"></a>機制 2：使用 sliceIdentifierColumnName
> [!IMPORTANT]
> 目前「Azure SQL 資料倉儲」並不支援 sliceIdentifierColumnName。 

達成可重複性的第二個機制是在目標資料表中擁有一個專用的資料行 (sliceIdentifierColumnName)。 Azure Data Factory 會使用這個資料行以確保來源和目的地保持同步。 當目的地 SQL 資料表結構描述可彈性變更或定義，就可以使用這種方法。 

Azure Data Factory 會基於可重複性目的使用此資料行，且在過程中 Azure Data Factory 不會對資料表進行任何結構描述變更。 如何使用這個方法：

1. 在目的地 SQL 資料表中定義一個「二進位 (32)」類型的資料行。 此資料行不應該有任何條件約束。 讓我們針對此範例將這個資料行命名為 AdfSliceIdentifier。


    來源資料表：

    ```sql
    CREATE TABLE [dbo].[Student](
       [Id] [varchar](32) NOT NULL,
       [Name] [nvarchar](256) NOT NULL
    )
    ```

    目的地資料表： 

    ```sql
    CREATE TABLE [dbo].[Student](
       [Id] [varchar](32) NOT NULL,
       [Name] [nvarchar](256) NOT NULL,
       [AdfSliceIdentifier] [binary](32) NULL
    )
    ```

2. 在複製活動中使用它，如下所示：
   
    ```json
    "sink":  
    { 
   
        "type": "SqlSink", 
        "sliceIdentifierColumnName": "AdfSliceIdentifier"
    }
    ```

Azure Data Factory 會根據其需求來填入此資料行，以確保來源和目的地保持同步。 此資料行的值不應用於此內容之外。 

與機制 1 類似，「複製活動」也會自動從目的地 SQL 資料表中清除所指定配量的資料。 接著，它會將來自來源的資料插入到目的地資料表中。 

## <a name="next-steps"></a>後續步驟
如需完整的 JSON 範例，請檢閱下列連接器文章： 

- [Azure SQL Database](data-factory-azure-sql-connector.md)
- [Azure SQL 資料倉儲](data-factory-azure-sql-data-warehouse-connector.md)
- [SQL Server](data-factory-sqlserver-connector.md)
