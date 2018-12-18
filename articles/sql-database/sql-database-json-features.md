---
title: Azure SQL Database JSON 功能 | Microsoft Docs
description: Azure SQL Database 可讓您剖析、查詢及格式化採用「JavaScript 物件標記法」(JSON) 的資料。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: dfbefd2ff4c2e99268c1e7e84154e93d58f542bb
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47163625"
---
# <a name="getting-started-with-json-features-in-azure-sql-database"></a>開始使用 Azure SQL Database 中的 JSON 功能
Azure SQL Database 可讓您剖析及查詢以「JavaScript 物件標記法」 [(JSON)](http://www.json.org/) 格式表示的資料，然後將您的關聯式資料匯出成 JSON 文字。

JSON 是一種用於在新式的 Web 與行動應用程式中交換資料的常用資料格式。 JSON 也用於將半結構化的資料儲存在記錄檔或 NoSQL 資料庫 (例如 [Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/)) 中。 許多 REST Web 服務都會傳回採用 JSON 文字格式的結果，或是接受採用 JSON 格式的資料。 大多數 Azure 服務 (例如 [Azure 搜尋服務](https://azure.microsoft.com/services/search/)、[Azure 儲存體](https://azure.microsoft.com/services/storage/)、[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/)) 都有會傳回或取用 JSON 的 REST 端點。

Azure SQL Database 可讓您輕鬆使用 JSON 資料，並將資料庫與新式服務整合。

## <a name="overview"></a>概觀
Azure SQL Database 提供下列可與 JSON 資料搭配使用的函數：

![JSON 函數](./media/sql-database-json-features/image_1.png)

如果您有 JSON 文字，您可以透過使用內建的函式 [JSON_VALUE](https://msdn.microsoft.com/library/dn921898.aspx)、[JSON_QUERY](https://msdn.microsoft.com/library/dn921884.aspx) 及 [ISJSON](https://msdn.microsoft.com/library/dn921896.aspx)，從 JSON 擷取資料或確認 JSON 的格式是否正確。 [JSON_MODIFY](https://msdn.microsoft.com/library/dn921892.aspx) 函式可讓您更新 JSON 文字內的值。 針對更進階的查詢和分析， [OPENJSON](https://msdn.microsoft.com/library/dn921885.aspx) 函數可以將 JSON 物件陣列轉換成一組資料列。 您可以在傳回的結果集上執行任何 SQL 查詢。 最後，還有 [FOR JSON](https://msdn.microsoft.com/library/dn921882.aspx) 子句，此子句可讓您將儲存在關聯式資料表中的資料格式化為 JSON 文字。

## <a name="formatting-relational-data-in-json-format"></a>以 JSON 格式將關聯式資料格式化
如果您有會從資料庫層擷取資料並以 JSON 格式提供回應的 Web 服務，或是有會接受以 JSON 格式化之資料的用戶端 JavaScript 架構或程式庫，您就可以直接在 SQL 查詢中將資料庫內容格式化為 JSON。 您不再需要撰寫應用程式程式碼以將來自 Azure SQL Database 的結果格式化為 JSON，或包含一些 JSON 序列化程式庫來轉換表格式查詢結果，然後將物件序列化為 JSON 格式。 取得代之的是，您可以在 Azure SQL Database 中使用 FOR JSON 子句將 SQL 查詢結果格式化為 JSON，然後直接在您的應用程式中使用它。

在下列範例中，會透過使用 FOR JSON 子句，將來自 Sales.Customer 資料表的資料列格式化為 JSON：

```
select CustomerName, PhoneNumber, FaxNumber
from Sales.Customers
FOR JSON PATH
```

FOR JSON PATH 子句會將查詢的結果格式化為 JSON 文字。 資料行名稱會作為索引鍵，而儲存格值則會以 JSON 值的形式產生︰

```
[
{"CustomerName":"Eric Torres","PhoneNumber":"(307) 555-0100","FaxNumber":"(307) 555-0101"},
{"CustomerName":"Cosmina Vlad","PhoneNumber":"(505) 555-0100","FaxNumber":"(505) 555-0101"},
{"CustomerName":"Bala Dixit","PhoneNumber":"(209) 555-0100","FaxNumber":"(209) 555-0101"}
]
```

結果集會格式化為 JSON 陣列，其中每個資料列皆格式化為個別的 JSON 物件。

PATH 表示您可以在資料行別名中使用點標記法來自訂 JSON 結果的輸出格式。 下列查詢會變更輸出 JSON 格式中 "CustomerName" 索引鍵的名稱 ，並將電話及傳真號碼放入 "Contact" 子物件中︰

```
select CustomerName as Name, PhoneNumber as [Contact.Phone], FaxNumber as [Contact.Fax]
from Sales.Customers
where CustomerID = 931
FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
```

此查詢的輸出看起來會像這樣：

```
{
    "Name":"Nada Jovanovic",
    "Contact":{
           "Phone":"(215) 555-0100",
           "Fax":"(215) 555-0101"
    }
}
```

在此範例中，我們透過指定 [WITHOUT_ARRAY_WRAPPER](https://msdn.microsoft.com/library/mt631354.aspx) 選項，傳回了單一 JSON 物件而不是陣列。 如果您知道您要傳回單一物件來作為查詢結果，就可以使用此選項。

FOR JSON 子句的主要價值在於，它可讓您從資料庫傳回格式化為巢狀 JSON 物件或陣列的複雜階層式資料。 下列範例說明如何將屬於 Customer 的 Orders 納入成為巢狀 Orders 陣列：

```
select CustomerName as Name, PhoneNumber as Phone, FaxNumber as Fax,
        Orders.OrderID, Orders.OrderDate, Orders.ExpectedDeliveryDate
from Sales.Customers Customer
    join Sales.Orders Orders
        on Customer.CustomerID = Orders.CustomerID
where Customer.CustomerID = 931
FOR JSON AUTO, WITHOUT_ARRAY_WRAPPER

```

您可以不傳送個別查詢來取得 Customer 資料，然後再擷取相關 Orders 清單，而是透過單一查詢來取得所有必要的資料，如下列範例輸出所示：

```
{
  "Name":"Nada Jovanovic",
  "Phone":"(215) 555-0100",
  "Fax":"(215) 555-0101",
  "Orders":[
    {"OrderID":382,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":395,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":1657,"OrderDate":"2013-01-31","ExpectedDeliveryDate":"2013-02-01"}
]
}
```

## <a name="working-with-json-data"></a>使用 JSON 資料
如果您沒有嚴格結構化的資料，如果您有複雜的子物件、陣列或階層式資料，或如果您的資料結構會隨時間演變，則 JSON 格式可協助您表現任何複雜的資料結構。

JSON 是一種文字格式，您可以在 Azure SQL Database 中使用它，就像使用任何其他字串類型一樣。 您可以用標準 NVARCHAR 形式來傳送或儲存 JSON 資料：

```
CREATE TABLE Products (
  Id int identity primary key,
  Title nvarchar(200),
  Data nvarchar(max)
)
go
CREATE PROCEDURE InsertProduct(@title nvarchar(200), @json nvarchar(max))
AS BEGIN
    insert into Products(Title, Data)
    values(@title, @json)
END
```

在此範例中，是透過使用 NVARCHAR(MAX) 類型來表現所使用的 JSON 資料。 您可以使用標準 Transact-SQL 語法，將 JSON 插入此資料表中或提供來作為預存程序的引數，如下列範例所示：

```
EXEC InsertProduct 'Toy car', '{"Price":50,"Color":"White","tags":["toy","children","games"]}'
```

任何可以與 Azure SQL Database 中的字串資料搭配運作的用戶端語言或程式庫，也將可以與 JSON 資料搭配運作。 JSON 可以儲存在任何支援 NVARCHAR 類型的資料表中，例如記憶體最佳化資料表或由系統控制版本的資料表。 JSON 不會導入任何條件約束，不論是用戶端程式碼中還是在資料庫層中的條件約束。

## <a name="querying-json-data"></a>查詢 JSON 資料
如果您有格式化為 JSON 的資料儲存在 Azure SQL 資料表中，JSON 函數可讓您在任何 SQL 查詢中使用此資料。

Azure SQL Database 中可用的 JSON 函數可讓您將格式化為 JSON 的資料視為任何其他 SQL 資料類型。 您可以輕鬆地從 JSON 文字中擷取值，然後在任何查詢中使用 JSON 資料︰

```
select Id, Title, JSON_VALUE(Data, '$.Color'), JSON_QUERY(Data, '$.tags')
from Products
where JSON_VALUE(Data, '$.Color') = 'White'

update Products
set Data = JSON_MODIFY(Data, '$.Price', 60)
where Id = 1
```

JSON_VALUE 函數會從儲存在 Data 資料行的 JSON 文字中擷取值。 此函數會使用類似 JavaScript 的路徑來參考所要擷取的 JSON 文字中的值。 所擷取的值可以在 SQL 查詢的任何部分中使用。

JSON_QUERY 函數類似於 JSON_VALUE。 與 JSON_VALUE 不同，此函數會擷取複雜的子物件，例如置於 JSON 文字中的陣列或物件。

JSON_MODIFY 函數可讓您指定 JSON 文字中應更新的值路徑，以及將覆寫舊值的新值。 如此一來，您便可以輕鬆更新 JSON 文字，而不需重新剖析整個結構。

由於 JSON 是以標準文字儲存，因此無法保證儲存在文字資料行中的值格式會正確。 您可以使用標準的 Azure SQL Database 檢查條件約束和 ISJSON 函數，來確認儲存在 JSON 資料行中的文字是否格式正確︰

```
ALTER TABLE Products
    ADD CONSTRAINT [Data should be formatted as JSON]
        CHECK (ISJSON(Data) > 0)
```

如果輸入的文字是格式正確的 JSON，ISJSON 函數就會傳回值 1。 在每次插入或更新 JSON 資料行時，這個條件約束都會確認新的文字值不是格式錯誤的 JSON。

## <a name="transforming-json-into-tabular-format"></a>將 JSON 轉換成表格式格式
Azure SQL Database 也可讓您將 JSON 集合轉換成表格式格式，然後再載入或查詢 JSON 資料。

OPENJSON 是一個資料表值函數，可剖析 JSON 文字、找出 JSON 物件陣列、逐一查看陣列的元素，然後在輸出結果中為每個陣列元素傳回一個資料列。

![JSON 表格式](./media/sql-database-json-features/image_2.png)

在上述範例中，我們可以指定要在哪裡尋找應該開啟的 JSON 陣列 (在 $.Orders 路徑中)、應該傳回哪些資料行作為結果，以及要在哪裡尋找將以資料格形式傳回的 JSON 值。

我們可以將 @orders 變數中的 JSON 陣列轉換成一組資料列、分析此結果集，或將資料列插入標準資料表中︰

```
CREATE PROCEDURE InsertOrders(@orders nvarchar(max))
AS BEGIN

    insert into Orders(Number, Date, Customer, Quantity)
    select Number, Date, Customer, Quantity
    OPENJSON (@orders)
     WITH (
            Number varchar(200),
            Date datetime,
            Customer varchar(200),
            Quantity int
     )

END
```
我們可以剖析採用 JSON 陣列格式並作為參數提供給預存程序的訂單集合，然後將它插入 Orders 資料表中。

## <a name="next-steps"></a>後續步驟
若要了解如何將 JSON 整合到您的應用程式中，請參閱下列資源︰

* [TechNet 部落格](https://blogs.technet.microsoft.com/dataplatforminsider/2016/01/05/json-in-sql-server-2016-part-1-of-4/)
* [MSDN 文件](https://msdn.microsoft.com/library/dn921897.aspx)
* [Channel 9 影片](https://channel9.msdn.com/Shows/Data-Exposed/SQL-Server-2016-and-JSON-Support)

若要了解將 JSON 整合到您應用程式中的各種案例，請參閱這段 [Channel 9 影片](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/JSON-as-a-bridge-betwen-NoSQL-and-relational-worlds)中的示範，或從 [JSON 部落格文章](http://blogs.msdn.com/b/sqlserverstorageengine/archive/tags/json/)中尋找符合您使用案例的情況。

