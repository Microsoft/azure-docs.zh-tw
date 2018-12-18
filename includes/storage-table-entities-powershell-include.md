<!--created by Robin Shahan to go in the articles for table storage w/powershell.
    There is one for Azure Table Storage and one for Azure Cosmos DB Table API -->

## <a name="managing-table-entities"></a>管理資料表實體

現在，您有一個資料表，讓我們看看如何管理資料表中的實體 (或稱資料列)。 

實體最多可有 255 個屬性，包括 3 個系統屬性：**PartitionKey**、**RowKey** 和 **Timestamp**。 您需負責插入及更新 **PartitionKey** 和 **RowKey** 的值。 伺服器會管理 **Timestamp**的值，該值無法予以修改。 **PartitionKey** 和 **RowKey** 的組合可唯一識別資料表內的每個實體。

* **PartitionKey**：決定儲存實體的資料分割。
* **RowKey**：在資料分割內唯一地識別實體。

您可以為每個實體最多定義 252 個自訂屬性。 

### <a name="add-table-entities"></a>新增資料表實體

使用 **Add-StorageTableRow** 將實體加入至資料表。 這些範例使用分割區索引鍵，其值為 "partition1" 和 "partition2"，資料列索引鍵等於州名縮寫。 每個實體中的屬性為使用者名稱和使用者識別碼。 

```powershell
$partitionKey1 = "partition1"
$partitionKey2 = "partition2"

# add four rows 
Add-StorageTableRow `
    -table $storageTable `
    -partitionKey $partitionKey1 `
    -rowKey ("CA") -property @{"username"="Chris";"userid"=1}

Add-StorageTableRow `
    -table $storageTable `
    -partitionKey $partitionKey2 `
    -rowKey ("NM") -property @{"username"="Jessie";"userid"=2}

Add-StorageTableRow `
    -table $storageTable `
    -partitionKey $partitionKey1 `
    -rowKey ("WA") -property @{"username"="Christine";"userid"=3}

Add-StorageTableRow `
    -table $storageTable `
    -partitionKey $partitionKey2 `
    -rowKey ("TX") -property @{"username"="Steven";"userid"=4}
```

### <a name="query-the-table-entities"></a>查詢資料表實體

有幾種不同方式可用來查詢資料表中的實體。

#### <a name="retrieve-all-entities"></a>擷取所有實體

若要擷取所有實體，請使用 **Get-AzureStorageTableRowAll**。

```powershell
Get-AzureStorageTableRowAll -table $storageTable | ft
```

此命令會產生類似下列資料表的結果：

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |
| 2 | Jessie | partition2 | NM |
| 4 | Steven | partition2 | TX |

#### <a name="retrieve-entities-for-a-specific-partition"></a>擷取特定分割區的實體

若要擷取特定分割區中的所有實體，請使用 **Get-AzureStorageTableRowByPartitionKey**。

```powershell
Get-AzureStorageTableRowByPartitionKey -table $storageTable -partitionKey $partitionKey1 | ft
```
結果會類似下列資料表：

| userid | username | partition | rowkey |
|----|---------|---------------|----|
| 1 | Chris | partition1 | CA |
| 3 | Christine | partition1 | WA |

#### <a name="retrieve-entities-for-a-specific-value-in-a-specific-column"></a>擷取特定資料行中特定值的實體

若要擷取特定資料行中的值等於特定值的實體，請使用 **Get-AzureStorageTableRowByColumnName**。

```powershell
Get-AzureStorageTableRowByColumnName -table $storageTable `
    -columnName "username" `
    -value "Chris" `
    -operator Equal
```

此查詢會擷取一筆記錄。

|field|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

#### <a name="retrieve-entities-using-a-custom-filter"></a>使用自訂篩選條件擷取實體 

若要使用自訂篩選器擷取實體，請使用 **Get-AzureStorageTableRowByCustomFilter**.

```powershell
Get-AzureStorageTableRowByCustomFilter `
    -table $storageTable `
    -customFilter "(userid eq 1)"
```

此查詢會擷取一筆記錄。

|field|value|
|----|----|
| userid | 1 |
| username | Chris |
| PartitionKey | partition1 |
| RowKey      | CA |

### <a name="updating-entities"></a>更新實體 

更新實體有三個步驟。 首先，取出要變更的實體。 第二，進行變更。 第三，使用 **Update-AzureStorageTableRow** 認可變更。

更新使用者名稱 = 'Jessie' 的實體，讓使用者名稱 = 'Jessie2'。 此範例也顯示了另一種使用 .NET 類型建立自訂篩選條件的方法。 

```powershell
# Create a filter and get the entity to be updated.
[string]$filter = `
    [Microsoft.WindowsAzure.Storage.Table.TableQuery]::GenerateFilterCondition("username",`
    [Microsoft.WindowsAzure.Storage.Table.QueryComparisons]::Equal,"Jessie")
$user = Get-AzureStorageTableRowByCustomFilter `
    -table $storageTable `
    -customFilter $filter

# Change the entity.
$user.username = "Jessie2" 

# To commit the change, pipe the updated record into the update cmdlet.
$user | Update-AzureStorageTableRow -table $storageTable 

# To see the new record, query the table.
Get-AzureStorageTableRowByCustomFilter -table $storageTable `
    -customFilter "(username eq 'Jessie2')"
```

結果會顯示 Jessie2 記錄。

|field|value|
|----|----|
| userid | 2 |
| username | Jessie2 |
| PartitionKey | partition2 |
| RowKey      | NM |

### <a name="deleting-table-entities"></a>刪除資料表實體

您可以刪除資料表中的一個實體或所有實體。

#### <a name="deleting-one-entity"></a>刪除單一實體

若要刪除單一實體，請取得該實體的參考，並使用管線將它傳遞到 **Remove-AzureStorageTableRow**。

```powershell
# Set filter.
[string]$filter = `
  [Microsoft.WindowsAzure.Storage.Table.TableQuery]::GenerateFilterCondition("username",`
  [Microsoft.WindowsAzure.Storage.Table.QueryComparisons]::Equal,"Jessie2")

# Retrieve entity to be deleted, then pipe it into the remove cmdlet.
$userToDelete = Get-AzureStorageTableRowByCustomFilter `
    -table $storageTable `
    -customFilter $filter
$userToDelete | Remove-AzureStorageTableRow -table $storageTable 

# Retrieve entities from table and see that Jessie2 has been deleted.
Get-AzureStorageTableRowAll -table $storageTable | ft
```

#### <a name="delete-all-entities-in-the-table"></a>刪除資料表中的所有實體 

若要刪除資料表中的所有實體，請將它們取出，並使用管線將它們傳遞到 remove Cmdlet。 

```powershell
# Get all rows and pipe the result into the remove cmdlet.
Get-AzureStorageTableRowAll `
    -table $storageTable | Remove-AzureStorageTableRow -table $storageTable 

# List entities in the table (there won't be any).
Get-AzureStorageTableRowAll -table $storageTable | ft
```
