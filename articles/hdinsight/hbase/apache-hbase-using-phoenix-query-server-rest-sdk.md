---
title: Phoenix Query Server REST SDK - Azure HDInsight
description: 在 Azure HDInsight 中安裝和使用 Phoenix Query Server 的 REST SDK。
services: hdinsight
ms.service: hdinsight
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 12/04/2017
ms.openlocfilehash: f5be273401fa9fdfdc2c97296f5f2265dbb3c447
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042372"
---
# <a name="phoenix-query-server-rest-sdk"></a>Phoenix Query Server REST SDK

[Apache Phoenix](http://phoenix.apache.org/) 是 [HBase](apache-hbase-overview.md) 上的開放原始碼、大規模平行關聯式資料庫。 Phoenix 可讓您透過 SSH 工具 (例如 [SQLLine](apache-hbase-phoenix-squirrel-linux.md))，搭配 HBase 使用類似 SQL 的查詢。 Phoenix 也提供一個名為 Phoenix Query Server (PQS) 的 HTTP 伺服器，這是一個精簡型用戶端，可支援兩種用戶端通訊傳輸機制：JSON 和 Protocol Buffers。 Protocol Buffers 是預設機制，可提供比 JSON 更有效率的通訊。

本文說明如何使用 PQS REST SDK，藉由 SQL 陳述式來建立資料表、個別或大量更新插入資料列，以及選取資料。 範例會使用[適用於 Apache Phoenix Query Server 的 Microsoft .NET 驅動程式](https://www.nuget.org/packages/Microsoft.Phoenix.Client) \(英文\)。 此 SDK 建置在 [Apache Calcite 的 Avatica](https://calcite.apache.org/avatica/) \(英文\) API 上，這些 API 完全使用 Protocol Buffers 作為序列化格式。

如需詳細資訊，請參閱 [Apache Calcite Avatica Protocol Buffers 參考](https://calcite.apache.org/avatica/docs/protobuf_reference.html) \(英文\)。

## <a name="install-the-sdk"></a>安裝 SDK

適用於 Apache Phoenix Query Server 的 Microsoft .NET 驅動程式是以 NuGet 套件的形式提供使用，您可以使用以下命令，從 Visual Studio **NuGet 套件管理員主控台**安裝此套件：

    Install-Package Microsoft.Phoenix.Client

## <a name="instantiate-new-phoenixclient-object"></a>將新的 PhoenixClient 物件具現化

若要開始使用此程式庫，請將一個新的 `PhoenixClient` 物件具現化，將含有 `Uri` 的 `ClusterCredentials` 傳遞給叢集，並且傳遞叢集的 Hadoop 使用者名稱和密碼。

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net/"), "USERNAME", "PASSWORD");
client = new PhoenixClient(credentials);
```

將 CLUSTERNAME 更改為您的 HDInsight HBase 叢集名稱，並將 USERNAME 和 PASSWORD 更改為建立叢集時指定的 Hadoop 認證。 預設 Hadoop 使用者名稱為 **admin**。

## <a name="generate-unique-connection-identifier"></a>產生唯一連線識別碼

若要將一或多個要求傳送給 PQS，您必須包含唯一連線識別碼以將要求與連線建立關聯。

```csharp
string connId = Guid.NewGuid().ToString();
```

每個範例都會先對 `OpenConnectionRequestAsync` 方法發出呼叫，傳遞唯一連線識別碼。 接著，定義 `ConnectionProperties` 和 `RequestOptions`，將這些物件和產生的連線識別碼傳遞給 `ConnectionSyncRequestAsync` 方法。 PQS 的 `ConnectionSyncRequest` 物件可協助確保用戶端與伺服器都具有一致的資料庫屬性檢視。

## <a name="connectionsyncrequest-and-its-connectionproperties"></a>ConnectionSyncRequest 及其 ConnectionProperties

若要呼叫 `ConnectionSyncRequestAsync`，請傳遞 `ConnectionProperties` 物件。

```csharp
ConnectionProperties connProperties = new ConnectionProperties
{
    HasAutoCommit = true,
    AutoCommit = true,
    HasReadOnly = true,
    ReadOnly = false,
    TransactionIsolation = 0,
    Catalog = "",
    Schema = "",
    IsDirty = true
};
await client.ConnectionSyncRequestAsync(connId, connProperties, options);
```

以下是一些相關的屬性：

| 屬性 | 說明 |
| -- | -- |
| AutoCommit | 布林值，用來表示是否為 Phoenix 交易啟用 `autoCommit`。 |
| 唯讀 | 布林值，用來表示連線是否為唯讀連線。 |
| TransactionIsolation | 整數，用來表示依據 JDBC 規格的交易隔離層級 - 請參閱下表。|
| 目錄 | 擷取連線屬性時所要使用的目錄名稱。 |
| 結構描述 | 擷取連線屬性時所要使用的結構描述名稱。 |
| IsDirty | 布林值，用來表示屬性是否已經更改。 |

以下是 `TransactionIsolation` 值：

| 隔離值 | 說明 |
| -- | -- |
| 0 | 不支援交易。 |
| 1 | 允許進行中途讀取、不可重複的讀取，以及虛設項目讀取。 |
| 2 | 防止中途讀取，但允許進行不可重複的讀取和虛設項目讀取。 |
| 4 | 防止中途讀取和不可重複的讀取，但允許進行虛設項目讀取。 |
| 8 | 全面防止中途讀取、不可重複的讀取，以及虛設項目讀取。 |

## <a name="create-a-new-table"></a>建立新的資料表

HBase 與任何其他 RDBMS 一樣，會將資料儲存在資料表中。 Phoenix 使用標準 SQL 查詢來建立新資料表，同時會定義主索引鍵和資料行類型。

以下範例及後續所有範例皆如[將新的 PhoenixClient 物件具現化](#instantiate-new-phoenixclient-object)中所定義，使用已具現化的 `PhoenixClient` 物件。

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// You can set certain request options, such as timeout in milliseconds:
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
CreateStatementResponse createStatementResponse = null;
OpenConnectionResponse openConnResponse = null;

try
{
    // Opening connection
    var info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Create the statement
    createStatementResponse = client.CreateStatementRequestAsync(connId, options).Result;
    
    // Create the table if it does not exist
    string sql = "CREATE TABLE IF NOT EXISTS Customers (Id varchar(20) PRIMARY KEY, FirstName varchar(50), " +
        "LastName varchar(100), StateProvince char(2), Email varchar(255), Phone varchar(15))";
    await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    Console.WriteLine($"Table \"Customers\" created.");
}
catch (Exception e)
{
    Console.WriteLine(e);
    throw;
}
finally
{
    if (createStatementResponse != null)
    {
        client.CloseStatementRequestAsync(connId, createStatementResponse.StatementId, options).Wait();
        createStatementResponse = null;
    }

    if (openConnResponse != null)
    {
        client.CloseConnectionRequestAsync(connId, options).Wait();
        openConnResponse = null;
    }
}
```

上述範例使用 `IF NOT EXISTS` 選項來建立一個名為 `Customers` 的新資料表。 `CreateStatementRequestAsync` 呼叫會在 Avitica (PQS) 伺服器中建立一個新的陳述式。 `finally` 區塊可關閉所傳回 `CreateStatementResponse` 和 `OpenConnectionResponse` 物件。

## <a name="insert-data-individually"></a>個別插入資料

以下範例示範個別資料插入，其中參考了美國州別和領土縮寫的 `List<string>` 集合：

```csharp
var states = new List<string> { "AL", "AK", "AS", "AZ", "AR", "CA", "CO", "CT", "DE", "DC", "FM", "FL", "GA", "GU", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MH", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "MP", "OH", "OK", "OR", "PW", "PA", "PR", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VI", "VA", "WA", "WV", "WI", "WY" };
```

後續的選取作業將會使用此資料表的 `StateProvince` 資料行值。

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = await client.PrepareRequestAsync(connId, sql, 100, options);
    statementHandle = prepareResponse.Statement;
    
    var r = new Random();

    // Insert 300 rows
    for (int i = 0; i < 300; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0,1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        Console.WriteLine("Inserting customer " + i);

        await client.ExecuteRequestAsync(statementHandle, list, long.MaxValue, true, options);
    }

    await client.CommitRequestAsync(connId, options);

    Console.WriteLine("Upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

執行 insert 陳述式的結構與建立新資料表類似。 請注意，在 `try` 區塊的結尾會明確地認可交易。 此範例會重複執行插入交易 300 次。 以下範例示範一個更有效率的批次插入程序。

## <a name="batch-insert-data"></a>批次插入資料

以下程式碼與個別插入資料的程式碼幾乎相同。 以下範例是在對 `ExecuteBatchRequestAsync` 的呼叫中使用 `UpdateBatch` 物件，而不是使用準備陳述式來重複地呼叫 `ExecuteRequestAsync`。

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();
options.TimeoutMillis = 300000;

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
CreateStatementResponse createStatementResponse = null;
try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);

    // Creating statement
    createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "UPSERT INTO Customers VALUES (?,?,?,?,?,?)";
    PrepareResponse prepareResponse = client.PrepareRequestAsync(connId, sql, long.MaxValue, options).Result;
    var statementHandle = prepareResponse.Statement;
    var updates = new pbc.RepeatedField<UpdateBatch>();

    var r = new Random();

    // Insert 300 rows
    for (int i = 300; i < 600; i++)
    {
        var list = new pbc.RepeatedField<TypedValue>();
        var id = new TypedValue
        {
            StringValue = "id" + i,
            Type = Rep.String
        };
        var firstName = new TypedValue
        {
            StringValue = "first" + i,
            Type = Rep.String
        };
        var lastName = new TypedValue
        {
            StringValue = "last" + i,
            Type = Rep.String
        };
        var state = new TypedValue
        {
            StringValue = states.ElementAt(r.Next(0, 49)),
            Type = Rep.String
        };
        var email = new TypedValue
        {
            StringValue = $"email{1}@junkemail.com",
            Type = Rep.String
        };
        var phone = new TypedValue
        {
            StringValue = $"555-229-341{i.ToString().Substring(0, 1)}",
            Type = Rep.String
        };
        list.Add(id);
        list.Add(firstName);
        list.Add(lastName);
        list.Add(state);
        list.Add(email);
        list.Add(phone);

        var batch = new UpdateBatch
        {
            ParameterValues = list
        };
        updates.Add(batch);

        Console.WriteLine($"Added customer {i} to batch");
    }

    var executeBatchResponse = await client.ExecuteBatchRequestAsync(connId, statementHandle.Id, updates, options);

    Console.WriteLine("Batch upserted customer data");

}
catch (Exception ex)
{

}
finally
{
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

在一個測試環境中，個別插入 300 筆新記錄會花費幾乎 2 分鐘的時間。 相對地，以一個批次插入 300 筆記錄則只需要 6 秒鐘的時間。

## <a name="select-data"></a>選取資料

以下範例示範如何重複使用一個連線來執行多個查詢：

1. 選取所有記錄，然後在傳回預設的最大值 100 之後，擷取剩餘的記錄。
2. 使用總資料列計數 select 陳述式來擷取單一純量結果。
3. 執行一個會依州別或領土傳回客戶總數的 select 陳述式。

```csharp
string connId = Guid.NewGuid().ToString();
RequestOptions options = RequestOptions.GetGatewayDefaultOptions();

// In gateway mode, PQS requests will be https://<cluster dns name>.azurehdinsight.net/hbasephoenix<N>/
// Requests sent to hbasephoenix0/ will be forwarded to PQS on workernode0
options.AlternativeEndpoint = "hbasephoenix0/";
OpenConnectionResponse openConnResponse = null;
StatementHandle statementHandle = null;

try
{
    // Opening connection
    pbc::MapField<string, string> info = new pbc::MapField<string, string>();
    openConnResponse = await client.OpenConnectionRequestAsync(connId, info, options);
    // Syncing connection
    ConnectionProperties connProperties = new ConnectionProperties
    {
        HasAutoCommit = true,
        AutoCommit = true,
        HasReadOnly = true,
        ReadOnly = false,
        TransactionIsolation = 0,
        Catalog = "",
        Schema = "",
        IsDirty = true
    };
    await client.ConnectionSyncRequestAsync(connId, connProperties, options);
    var createStatementResponse = await client.CreateStatementRequestAsync(connId, options);

    string sql = "SELECT * FROM Customers";
    ExecuteResponse executeResponse = await client.PrepareAndExecuteRequestAsync(connId, sql, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> rows = executeResponse.Results[0].FirstFrame.Rows;
    // Loop through all of the returned rows and display the first two columns
    for (int i = 0; i < rows.Count; i++)
    {
        Row row = rows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + " " + row.Value[1].ScalarValue.StringValue);
    }
    
    // 100 is hard-coded on the server side as the default firstframe size
    // FetchRequestAsync is called to get any remaining rows
    Console.WriteLine("");
    Console.WriteLine($"Number of rows: {rows.Count}");

    // Fetch remaining rows, offset is not used, simply set to 0
    // When FetchResponse.Frame.Done is true, all rows were fetched
    FetchResponse fetchResponse = await client.FetchRequestAsync(connId, createStatementResponse.StatementId, 0, int.MaxValue, options);
    Console.WriteLine($"Frame row count: {fetchResponse.Frame.Rows.Count}");
    Console.WriteLine($"Fetch response is done: {fetchResponse.Frame.Done}");
    Console.WriteLine("");

    // Running query 2
    string sql2 = "select count(*) from Customers";
    ExecuteResponse countResponse = await client.PrepareAndExecuteRequestAsync(connId, sql2, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);
    long count = countResponse.Results[0].FirstFrame.Rows[0].Value[0].ScalarValue.NumberValue;

    Console.WriteLine($"Total customer records: {count}");
    Console.WriteLine("");

    // Running query 3
    string sql3 = "select StateProvince, count(*) as Number from Customers group by StateProvince order by Number desc";
    ExecuteResponse groupByResponse = await client.PrepareAndExecuteRequestAsync(connId, sql3, createStatementResponse.StatementId, long.MaxValue, int.MaxValue, options);

    pbc::RepeatedField<Row> stateRows = groupByResponse.Results[0].FirstFrame.Rows;
    for (int i = 0; i < stateRows.Count; i++)
    {
        Row row = stateRows[i];
        Console.WriteLine(row.Value[0].ScalarValue.StringValue + ": " + row.Value[1].ScalarValue.NumberValue);
    }
}
catch (Exception ex)
{

}
finally
{
    if (statementHandle != null)
    {
        await client.CloseStatementRequestAsync(connId, statementHandle.Id, options);
        statementHandle = null;
    }
    if (openConnResponse != null)
    {
        await client.CloseConnectionRequestAsync(connId, options);
        openConnResponse = null;
    }
}
```

`select` 陳述式的輸出應該為以下結果：

```
id0 first0
id1 first1
id10 first10
id100 first100
id101 first101
id102 first102
. . .
id185 first185
id186 first186
id187 first187
id188 first188

Number of rows: 100
Frame row count: 500
Fetch response is done: True

Total customer records: 600

NJ: 21
CA: 19
GU: 17
NC: 16
IN: 16
MA: 16
AZ: 16
ME: 16
IL: 15
OR: 15
. . . 
MO: 10
HI: 10
GA: 10
DC: 9
NM: 9
MD: 9
MP: 9
SC: 7
AR: 7
MH: 6
FM: 5
```

## <a name="next-steps"></a>後續步驟 

* [HDInsight 中的 Phoenix](../hdinsight-phoenix-in-hdinsight.md)
* [使用 HBase REST SDK](apache-hbase-rest-sdk.md)
