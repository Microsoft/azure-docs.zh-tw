---
title: 使用 HBase .NET SDK - Azure HDInsight
description: 使用 HBase .NET SDK 建立和刪除資料表，以及讀取和寫入資料。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 12/13/2017
ms.author: ashishth
ms.openlocfilehash: af3b87fbe79624143b6c2b7e0a3c50852e532524
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042116"
---
# <a name="use-the-hbase-net-sdk"></a>使用 HBase .NET SDK

[HBase](apache-hbase-overview.md) 提供兩大資料處理選擇：[Hive 查詢，以及呼叫 HBase 的 RESTful API](apache-hbase-tutorial-get-started-linux.md)。 您可以使用 `curl` 命令或類似公用程式，直接使用 REST API。

對於 C# 和 .NET 應用程式，[Microsoft HBase REST Client Library for .NET](https://www.nuget.org/packages/Microsoft.HBase.Client/) 還提供了 HBase REST API 以外的用戶端程式庫。

## <a name="install-the-sdk"></a>安裝 SDK

HBase .NET SDK 以 NuGet 封裝的形式提供，能以 Visual Studio **NuGet 套件管理員主控台**使用下列命令來安裝：

    Install-Package Microsoft.HBase.Client

## <a name="instantiate-a-new-hbaseclient-object"></a>實例化新的 HBaseClient 物件

若要使用 SDK，請將一個新的 `HBaseClient` 物件具現化，在由通向您的叢集的 `Uri` 以及 Hadoop 使用者名稱和密碼組成的 `ClusterCredentials` 中傳遞。

```csharp
var credentials = new ClusterCredentials(new Uri("https://CLUSTERNAME.azurehdinsight.net"), "USERNAME", "PASSWORD");
client = new HBaseClient(credentials);
```

將 CLUSTERNAME 更改為您的 HDInsight HBase 叢集名稱，並將 USERNAME 和 PASSWORD 更改為建立叢集時指定的 Hadoop 認證。 預設 Hadoop 使用者名稱為 **admin**。

## <a name="create-a-new-table"></a>建立新的資料表

HBase 會將資料儲存在資料表中。 資料表包含 *Rowkey*、主索引鍵，以及名為*資料行系列*的一或多個資料行群組。 每個資料表中的資料會根據 Rowkey 範圍水平均分至*區域*。 每個區域都有開始和結束索引鍵。 一個資料表可以有一或多個區域。 隨著資料表中的資料成長，HBase 會將大型區域分割成較小的區域。 區域會儲存在*區域伺服器*中，一個區域伺服器可儲存多個區域。

資料實際儲存在 *HFiles*。 單一 HFile 包含一個資料表、一個區域和一個資料行系列的資料。 HFile 中的資料列會在 Rowkey 上依序儲存。 每個 HFile 都具有 *B+ 樹狀結構*索引，以利快速擷取資料列。

若要建立新的資料表，請指定 `TableSchema` 和資料行。 下列程式碼會檢查資料表「RestSDKTable」是否已經存在，如果不存在，即會建立資料表。

```csharp
if (!client.ListTablesAsync().Result.name.Contains("RestSDKTable"))
{
    // Create the table
    var newTableSchema = new TableSchema {name = "RestSDKTable" };
    newTableSchema.columns.Add(new ColumnSchema() {name = "t1"});
    newTableSchema.columns.Add(new ColumnSchema() {name = "t2"});

    await client.CreateTableAsync(newTableSchema);
}
```

這個新的資料表有 t1 和 t2 兩個資料行系列。 因為資料行系列分別儲存在不同的 HFiles，因此，經常查詢的資料具有個別的資料行系列不無道理。 在下列[插入資料](#insert-data)範例中，是將資料行新增至 t1 資料行系列。

## <a name="delete-a-table"></a>刪除資料表

刪除資料表：

```csharp
await client.DeleteTableAsync("RestSDKTable");
```

## <a name="insert-data"></a>插入資料

若要插入資料，您可以將一個唯一的資料列索引鍵指定為資料列識別碼。 所有資料都儲存在 `byte[]` 陣列中。 下列程式碼定義了 `title`、`director`、`release_date` 資料行，並將其新增至 t1 資料行系列，因為這些是最常存取的資料行。 `description` 和 `tagline` 資料行則會新增至 t2 資料行系列。 您可以視需要將資料分割成資料行系列。

```csharp
var key = "fifth_element";
var row = new CellSet.Row { key = Encoding.UTF8.GetBytes(key) };
var value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:title"),
    data = Encoding.UTF8.GetBytes("The Fifth Element")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:director"),
    data = Encoding.UTF8.GetBytes("Luc Besson")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t1:release_date"),
    data = Encoding.UTF8.GetBytes("1997")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:description"),
    data = Encoding.UTF8.GetBytes("In the colorful future, a cab driver unwittingly becomes the central figure in the search for a legendary cosmic weapon to keep Evil and Mr Zorg at bay.")
};
row.values.Add(value);
value = new Cell
{
    column = Encoding.UTF8.GetBytes("t2:tagline"),
    data = Encoding.UTF8.GetBytes("The Fifth is life")
};
row.values.Add(value);

var set = new CellSet();
set.rows.Add(row);

await client.StoreCellsAsync("RestSDKTable", set);
```

HBase 會實作 BigTable，因此資料格式如下所示：

![具有 [叢集使用者] 角色的使用者](./media/apache-hbase-rest-sdk/table.png)

## <a name="select-data"></a>選取資料

若要讀取 HBase 資料表的資料，請將資料表名稱和資料列索引鍵傳遞至 `GetCellsAsync` 方法，以傳回 `CellSet`。

```csharp
var key = "fifth_element";

var cells = await client.GetCellsAsync("RestSDKTable", key);
// Get the first value from the row.
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values[0].data));
// Get the value for t1:title
Console.WriteLine(Encoding.UTF8.GetString(cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:title").data));
// With the previous insert, it should yield: "The Fifth Element"
```

在此情況下，程式碼只會傳回最初相符的資料列，因為唯一的索引鍵應該只有一個資料列。 傳回的值會從 `byte[]` 陣列變更為 `string` 格式。 您也可以將值轉換成其他類型，例如電影上映日期可以使用整數：

```csharp
var releaseDateField = cells.rows[0].values
    .Find(c => Encoding.UTF8.GetString(c.column) == "t1:release_date");
int releaseDate = 0;

if (releaseDateField != null)
{
    releaseDate = Convert.ToInt32(Encoding.UTF8.GetString(releaseDateField.data));
}
Console.WriteLine(releaseDate);
// Should return 1997
```

## <a name="scan-over-rows"></a>掃描資料列

HBase 使用 `scan` 擷取一或多個資料列。 這個範例會以 10 個批次要求多個資料列，並擷取其索引鍵的值介於 25 和 35 之間的資料。 擷取所有資料列之後，請刪除掃描器以清除資源。

```csharp
var tableName = "mytablename";

// Assume the table has integer keys and we want data between keys 25 and 35
var scanSettings = new Scanner()
{
    batch = 10,
    startRow = BitConverter.GetBytes(25),
    endRow = BitConverter.GetBytes(35)
};
RequestOptions scanOptions = RequestOptions.GetDefaultOptions();
scanOptions.AlternativeEndpoint = "hbaserest0/";
ScannerInformation scannerInfo = null;
try
{
    scannerInfo = await client.CreateScannerAsync(tableName, scanSettings, scanOptions);
    CellSet next = null;
    while ((next = client.ScannerGetNextAsync(scannerInfo, scanOptions).Result) != null)
    {
        foreach (var row in next.rows)
        {
            // ... read the rows
        }
    }
}
finally
{
    if (scannerInfo != null)
    {
        await client.DeleteScannerAsync(tableName, scannerInfo, scanOptions);
    }
}
```

## <a name="next-steps"></a>後續步驟

* [開始使用 HDInsight 中的 Apache HBase 範例](apache-hbase-tutorial-get-started-linux.md)
* 透過[使用 HBase 分析即時 Twitter 情感](../hdinsight-hbase-analyze-twitter-sentiment.md)建立端對端應用程式
