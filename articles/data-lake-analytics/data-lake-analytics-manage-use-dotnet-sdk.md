---
title: 使用 Azure .NET SDK 管理 Azure Data Lake Analytics
description: 本文說明如何使用 Azure .Net SDK 來撰寫應用程式，以管理 Data Lake Analytics 作業、資料來源和使用者。
services: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 811d172d-9873-4ce9-a6d5-c1a26b374c79
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 06/18/2017
ms.openlocfilehash: 96b449e372417298ee3517d6a45c245d440a01c2
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047386"
---
# <a name="manage-azure-data-lake-analytics-a-net-app"></a>管理 Azure Data Lake Analytics .NET 應用程式
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用以 Azure .NET SDK 所撰寫的應用程式，來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。 

## <a name="prerequisites"></a>必要條件

* **已安裝 Visual Studio 2015、Visual Studio 2013 更新 4，或具有 Visual C++ 的 Visual Studio 2012**。
* **Microsoft Azure SDK for .NET 2.5 版或更新版本**。  使用 [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx)來進行安裝。
* **必要的 NuGet 套件**

### <a name="install-nuget-packages"></a>安裝 NuGet 套件

|Package|版本|
|-------|-------|
|[Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication)| 2.3.1|
|[Microsoft.Azure.Management.DataLake.Analytics](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Analytics)|3.0.0|
|[Microsoft.Azure.Management.DataLake.Store](https://www.nuget.org/packages/Microsoft.Azure.Management.DataLake.Store)|2.2.0|
|[Microsoft.Azure.Management.ResourceManager](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|1.6.0-preview|
|[Microsoft.Azure.Graph.RBAC](https://www.nuget.org/packages/Microsoft.Azure.Management.ResourceManager)|3.4.0-preview|

您可以透過 NuGet 命令列，使用下列命令來安裝這些套件：

```
Install-Package -Id Microsoft.Rest.ClientRuntime.Azure.Authentication  -Version 2.3.1
Install-Package -Id Microsoft.Azure.Management.DataLake.Analytics  -Version 3.0.0
Install-Package -Id Microsoft.Azure.Management.DataLake.Store  -Version 2.2.0
Install-Package -Id Microsoft.Azure.Management.ResourceManager  -Version 1.6.0-preview
Install-Package -Id Microsoft.Azure.Graph.RBAC -Version 3.4.0-preview
```

## <a name="common-variables"></a>常用變數

``` csharp
string subid = "<Subscription ID>"; // Subscription ID (a GUID)
string tenantid = "<Tenant ID>"; // AAD tenant ID or domain. For example, "contoso.onmicrosoft.com"
string rg == "<value>"; // Resource  group name
string clientid = "1950a258-227b-4e31-a9cf-717495945fc2"; // Sample client ID (this will work, but you should pick your own)
```

## <a name="authentication"></a>驗證

您有多個可登入 Azure Data Lake Analytics 的選項。 下列程式碼片段說明使用快顯視窗進行互動式使用者驗證的驗證範例。

``` csharp
using System;
using System.IO;
using System.Threading;
using System.Security.Cryptography.X509Certificates;

using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.DataLake.Analytics;
using Microsoft.Azure.Management.DataLake.Analytics.Models;
using Microsoft.Azure.Management.DataLake.Store;
using Microsoft.Azure.Management.DataLake.Store.Models;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Azure.Graph.RBAC;

public static Program
{
   public static string TENANT = "microsoft.onmicrosoft.com";
   public static string CLIENTID = "1950a258-227b-4e31-a9cf-717495945fc2";
   public static System.Uri ARM_TOKEN_AUDIENCE = new System.Uri( @"https://management.core.windows.net/");
   public static System.Uri ADL_TOKEN_AUDIENCE = new System.Uri( @"https://datalake.azure.net/" );
   public static System.Uri GRAPH_TOKEN_AUDIENCE = new System.Uri( @"https://graph.windows.net/" );

   static void Main(string[] args)
   {
      string MY_DOCUMENTS= System.Environment.GetFolderPath( System.Environment.SpecialFolder.MyDocuments);
      string TOKEN_CACHE_PATH = System.IO.Path.Combine(MY_DOCUMENTS, "my.tokencache");

      var tokenCache = GetTokenCache(TOKEN_CACHE_PATH);
      var armCreds = GetCreds_User_Popup(TENANT, ARM_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var adlCreds = GetCreds_User_Popup(TENANT, ADL_TOKEN_AUDIENCE, CLIENTID, tokenCache);
      var graphCreds = GetCreds_User_Popup(TENANT, GRAPH_TOKEN_AUDIENCE, CLIENTID, tokenCache);
   }
}
```

如需 **GetCreds_User_Popup** 的原始程式碼及其他驗證選項的程式碼，請參閱 [Data Lake Analytics .NET 驗證選項](https://github.com/Azure-Samples/data-lake-analytics-dotnet-auth-options) \(英文\)


## <a name="create-the-client-management-objects"></a>建立用戶端管理物件

``` csharp
var resourceManagementClient = new ResourceManagementClient(armCreds) { SubscriptionId = subid };

var adlaAccountClient = new DataLakeAnalyticsAccountManagementClient(armCreds);
adlaAccountClient.SubscriptionId = subid;

var adlsAccountClient = new DataLakeStoreAccountManagementClient(armCreds);
adlsAccountClient.SubscriptionId = subid;

var adlaCatalogClient = new DataLakeAnalyticsCatalogManagementClient(adlCreds);
var adlaJobClient = new DataLakeAnalyticsJobManagementClient(adlCreds);

var adlsFileSystemClient = new DataLakeStoreFileSystemManagementClient(adlCreds);

var  graphClient = new GraphRbacManagementClient(graphCreds);
graphClient.TenantID = domain;

```

## <a name="manage-accounts"></a>管理帳戶

### <a name="create-an-azure-resource-group"></a>建立 Azure 資源群組

如果您尚未建立，您必須具有 Azure 資源群組來建立 Data Lake Analytics 元件。 您將需要您的驗證認證、訂用帳戶 ID 及一個位置。 下列程式碼示範如何建立資源群組：

``` csharp
var resourceGroup = new ResourceGroup { Location = location };
resourceManagementClient.ResourceGroups.CreateOrUpdate(groupName, rg);
```
如需詳細資訊，請參閱 [Azure 資源群組和 Data Lake Analytics](#Azure-Resource-Groups-and-Data-Lake-Analytics)。

### <a name="create-a-data-lake-store-account"></a>建立 Data Lake Store 帳戶

每個 ADLA 帳戶都需要一個 ADLS 帳戶。 如果您還沒有可用的帳戶，可以使用下列程式碼來建立一個：

``` csharp
var new_adls_params = new DataLakeStoreAccount(location: _location);
adlsAccountClient.Account.Create(rg, adls, new_adls_params);
```

### <a name="create-a-data-lake-analytics-account"></a>建立 Data Lake Analytics 帳戶

下列程式碼會建立一個 ADLS 帳戶

``` csharp
var new_adla_params = new DataLakeAnalyticsAccount()
{
   DefaultDataLakeStoreAccount = adls,
   Location = location
};

adlaClient.Account.Create(rg, adla, new_adla_params);
```

### <a name="list-data-lake-store-accounts"></a>列出 Data Lake Store 帳戶

``` csharp
var adlsAccounts = adlsAccountClient.Account.List().ToList();
foreach (var adls in adlsAccounts)
{
   Console.WriteLine($"ADLS: {0}", adls.Name);
}
```

### <a name="list-data-lake-analytics-accounts"></a>列出 Data Lake Analytics 帳戶

``` csharp
var adlaAccounts = adlaClient.Account.List().ToList();

for (var adla in AdlaAccounts)
{
   Console.WriteLine($"ADLA: {0}, adla.Name");
}
```

### <a name="checking-if-an-account-exists"></a>檢查帳戶是否存在

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
```

### <a name="get-information-about-an-account"></a>取得帳戶的相關資訊

``` csharp
bool exists = adlaClient.Account.Exists(rg, adla));
if (exists)
{
   var adla_accnt = adlaClient.Account.Get(rg, adla);
}
```

### <a name="delete-an-account"></a>刪除帳戶

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
   adlaClient.Account.Delete(rg, adla);
}
```

### <a name="get-the-default-data-lake-store-account"></a>取得預設的 Data Lake Store 帳戶

每個 Data Lake Analytics 帳戶都必須擁有預設 Data Lake Store 帳戶。 您可以使用此程式碼來判斷 Analytics 帳戶的預設 Store 帳戶。

``` csharp
if (adlaClient.Account.Exists(rg, adla))
{
  var adla_accnt = adlaClient.Account.Get(rg, adla);
  string def_adls_account = adla_accnt.DefaultDataLakeStoreAccount;
}
```

## <a name="manage-data-sources"></a>管理資料來源

Data Lake Analytics 目前支援下列資料來源：

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Azure 儲存體帳戶](../storage/common/storage-introduction.md)

### <a name="link-to-an-azure-storage-account"></a>連結至 Azure 儲存體帳戶

您可以建立「Azure 儲存體」帳戶的連結。

``` csharp
string storage_key = "xxxxxxxxxxxxxxxxxxxx";
string storage_account = "mystorageaccount";
var addParams = new AddStorageAccountParameters(storage_key);            
adlaClient.StorageAccounts.Add(rg, adla, storage_account, addParams);
```

### <a name="list-azure-storage-data-sources"></a>列出 Azure 儲存體資料來源

``` csharp
var stg_accounts = adlaAccountClient.StorageAccounts.ListByAccount(rg, adla);

if (stg_accounts != null)
{
  foreach (var stg_account in stg_accounts)
  {
      Console.WriteLine($"Storage account: {0}", stg_account.Name);
  }
}
```

### <a name="list-data-lake-store-data-sources"></a>列出 Data Lake Store 資料來源

``` csharp
var adls_accounts = adlsClient.Account.List();

if (adls_accounts != null)
{
  foreach (var adls_accnt in adls_accounts)
  {
      Console.WriteLine($"ADLS account: {0}", adls_accnt.Name);
  }
}
```

### <a name="upload-and-download-folders-and-files"></a>上傳及下載資料夾和檔案
您可以使用 Data Lake Store 檔案系統用戶端管理物件，透過下列方法，將個別的檔案或資料夾上傳至 Azure 及從 Azure 下載到您的本機電腦：

- UploadFolder
- UploadFile
- DownloadFolder
- DownloadFile

這些方法的第一個參數是 Data Lake Store 帳戶的名稱，後面接著來源路徑和目的地路徑的參數。

下列範例示範如何下載 Data Lake Store 中的資料夾。

``` csharp
adlsFileSystemClient.FileSystem.DownloadFolder(adls, sourcePath, destinationPath);
```

### <a name="create-a-file-in-a-data-lake-store-account"></a>在 Data Lake Store 帳戶中建立檔案

``` csharp
using (var memstream = new MemoryStream())
{
   using (var sw = new StreamWriter(memstream, UTF8Encoding.UTF8))
   {
      sw.WriteLine("Hello World");
      sw.Flush();
      
      memstream.Position = 0;

      adlsFileSystemClient.FileSystem.Create(adls, "/Samples/Output/randombytes.csv", memstream);
   }
}
```

### <a name="verify-azure-storage-account-paths"></a>確認 Azure 儲存體帳戶路徑
下列程式碼會檢查 Azure 儲存體帳戶 (storageAccntName) 是否存在於 Data Lake Analytics 帳戶 (analyticsAccountName)，以及容器 (containerName) 是否存在於 Azure 儲存體帳戶。

``` csharp
string storage_account = "mystorageaccount";
string storage_container = "mycontainer";
bool accountExists = adlaClient.Account.StorageAccountExists(rg, adla, storage_account));
bool containerExists = adlaClient.Account.StorageContainerExists(rg, adla, storage_account, storage_container));
```

## <a name="manage-catalog-and-jobs"></a>管理目錄和作業
DataLakeAnalyticsCatalogManagementClient 物件會提供方法，用以管理為每個 Azure Data Lake Analytics 帳戶提供的 SQL 資料庫。 DataLakeAnalyticsJobManagementClient 會提供方法來提交及管理使用 U-SQL 指令碼在資料庫上執行的作業。

### <a name="list-databases-and-schemas"></a>列出資料庫和結構描述
在您可以列出的幾個項目中，最常見的是資料庫和其結構描述。 下列程式碼會取得資料庫的集合，然後列舉每個資料庫的結構描述。

``` csharp
var databases = adlaCatalogClient.Catalog.ListDatabases(adla);
foreach (var db in databases)
{
  Console.WriteLine($"Database: {db.Name}");
  Console.WriteLine(" - Schemas:");
  var schemas = adlaCatalogClient.Catalog.ListSchemas(adla, db.Name);
  foreach (var schm in schemas)
  {
      Console.WriteLine($"\t{schm.Name}");
  }
}
```

### <a name="list-table-columns"></a>列出資料表資料行
下列程式碼示範如何使用 Data Lake Analytics 目錄管理用戶端存取資料庫，以列出指定資料表的資料行。

``` csharp
var tbl = adlaCatalogClient.Catalog.GetTable(adla, "master", "dbo", "MyTableName");
IEnumerable<USqlTableColumn> columns = tbl.ColumnList;

foreach (USqlTableColumn utc in columns)
{
  Console.WriteLine($"\t{utc.Name}");
}
```

### <a name="submit-a-u-sql-job"></a>提交 U-SQL 作業
下列程式碼示範如何使用 Data Lake Analytics 作業管理用戶端來提交作業。
``` csharp
string scriptPath = "/Samples/Scripts/SearchResults_Wikipedia_Script.txt";
Stream scriptStrm = adlsFileSystemClient.FileSystem.Open(_adlsAccountName, scriptPath);
string scriptTxt = string.Empty;
using (StreamReader sr = new StreamReader(scriptStrm))
{
    scriptTxt = sr.ReadToEnd();
}

var jobName = "SR_Wikipedia";
var jobId = Guid.NewGuid();
var properties = new USqlJobProperties(scriptTxt);
var parameters = new JobInformation(jobName, JobType.USql, properties, priority: 1, degreeOfParallelism: 1, jobId: jobId);
var jobInfo = adlaJobClient.Job.Create(adla, jobId, parameters);
Console.WriteLine($"Job {jobName} submitted.");
```

### <a name="list-failed-jobs"></a>列出失敗的作業
下列程式碼列出失敗作業的相關資訊。

``` csharp
var odq = new ODataQuery<JobInformation> { Filter = "result eq 'Failed'" };
var jobs = adlaJobClient.Job.List(adla, odq);
foreach (var j in jobs)
{
   Console.WriteLine($"{j.Name}\t{j.JobId}\t{j.Type}\t{j.StartTime}\t{j.EndTime}");
}
```

### <a name="list-pipelines"></a>列出管線
下列程式碼會列出提交給帳戶之作業的每個管線相關資訊。

``` csharp
var pipelines = adlaJobClient.Pipeline.List(adla);
foreach (var p in pipelines)
{
   Console.WriteLine($"Pipeline: {p.Name}\t{p.PipelineId}\t{p.LastSubmitTime}");
}
```

### <a name="list-recurrences"></a>列出週期
下列程式碼會列出提交給帳戶之作業的每個週期相關資訊。

``` csharp
var recurrences = adlaJobClient.Recurrence.List(adla);
foreach (var r in recurrences)
{
   Console.WriteLine($"Recurrence: {r.Name}\t{r.RecurrenceId}\t{r.LastSubmitTime}");
}
```

## <a name="common-graph-scenarios"></a>常見的圖形案例

### <a name="look-up-user-in-the-aad-directory"></a>查詢 AAD 目錄中的使用者

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
```

### <a name="get-the-objectid-of-a-user-in-the-aad-directory"></a>取得 AAD 目錄中使用者的 ObjectId

``` csharp
var userinfo = graphClient.Users.Get( "bill@contoso.com" );
Console.WriteLine( userinfo.ObjectId )
```

## <a name="manage-compute-policies"></a>管理計算原則
DataLakeAnalyticsAccountManagementClient 物件會提供方法，用以管理 Data Lake Analytics 帳戶的計算原則。

### <a name="list-compute-policies"></a>列出計算原則
下列程式碼會擷取 Data Lake Analytics 帳戶的計算原則清單。

``` csharp
var policies = adlaAccountClient.ComputePolicies.ListByAccount(rg, adla);
foreach (var p in policies)
{
   Console.WriteLine($"Name: {p.Name}\tType: {p.ObjectType}\tMax AUs / job: {p.MaxDegreeOfParallelismPerJob}\tMin priority / job: {p.MinPriorityPerJob}");
}
```

### <a name="create-a-new-compute-policy"></a>建立新的計算原則
下列程式碼會為 Data Lake Analytics 帳戶建立新的計算原則，其中是將指定使用者可用的 AU 上限設定為 50，而將作業最低優先順序設定為 250。

``` csharp
var userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde";
var newPolicyParams = new ComputePolicyCreateOrUpdateParameters(userAadObjectId, "User", 50, 250);
adlaAccountClient.ComputePolicies.CreateOrUpdate(rg, adla, "GaryMcDaniel", newPolicyParams);
```

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md)
* [使用 Azure 入口網站監視和疑難排解 Azure Data Lake Analytics 作業](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
