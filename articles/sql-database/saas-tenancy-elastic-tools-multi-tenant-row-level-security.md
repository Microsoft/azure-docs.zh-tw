---
title: 使用 RLS 與彈性資料庫工具的多租用戶應用程式 | Microsoft Docs
description: 使用彈性資料庫工具搭配資料列層級安全性，建置具有可高度擴充性資料層的應用程式。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: tmullaney
ms.author: thmullan
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 94430d3b72bb5b8e8bde0e9e2e9fb2eb2b0c3632
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056279"
---
# <a name="multi-tenant-applications-with-elastic-database-tools-and-row-level-security"></a>使用彈性資料庫工具和資料列層級安全性的多租用戶應用程式

[彈性資料庫工具](sql-database-elastic-scale-get-started.md)和[資料列層級安全性 (RLS)][rls] 共同運作，讓您可以透過 Azure SQL Database 調整多租用戶應用程式的資料層。 綜合這些技術可協助您建置具有高擴充性資料層的應用程式。 資料層支援多租用戶分區，並使用 **ADO.NET SqlClient** 或 **Entity Framework**。 如需詳細資訊，請參閱[多租用戶 SaaS 應用程式與 Azure SQL Database 的設計模式](saas-tenancy-app-design-patterns.md)。

- **彈性資料庫工具**可讓開發人員透過使用 .NET 程式庫和 Azure 服務範本的業界標準分區化作法，相應放大資料層。 使用[彈性資料庫用戶端程式庫][s-d-elastic-database-client-library]管理分區，有助於自動化及簡化許多常與分區化相關的基礎結構工作。
- **資料列層級安全性**可讓開發人員在相同資料庫中安全地儲存多個租用戶的資料。 RLS 安全性原則會篩選掉不屬於執行查詢之租用戶的資料列。 將篩選邏輯集中在資料庫中，簡化了維護工作，並且降低安全性錯誤的風險。 只仰賴用戶端程式碼來強制執行安全性是種極具風險的替代方案。

透過搭配使用這些功能，應用程式可以在同一個分區資料庫中儲存多個租用戶的資料。 當租用戶共用資料庫時，每個租用戶所花費的成本更少。 然而，相同的應用程式也可以為進階租用戶提供付費選項，讓它們取得專屬的單一租用戶分區。 隔離出單一租用戶的一個好處在於能保證較穩固的效能， 因為在單一租用戶的資料庫中，不會有其他租用戶競爭資源。

目標是要使用彈性資料庫用戶端程式庫[資料依存路由](sql-database-elastic-scale-data-dependent-routing.md) API，來將每個指定的租用戶自動連線到正確的分區資料庫。 只有一個分區包含指定租用戶的特定 TenantId 值。 TenantId 是「分區索引鍵」。 建立連線之後，資料庫中的 RLS 安全性原則可確保指定的租用戶只能存取包含其 TenantId 的資料列。

> [!NOTE]
> 租用戶識別碼可能由多個資料行組成。 為了方便此討論進行，我們直接假設使用單一資料行的 TenantId。

![部落格應用程式架構][1]

## <a name="download-the-sample-project"></a>下載範例專案

### <a name="prerequisites"></a>先決條件

- 使用 Visual Studio (2012 或更新版本) 
- 建立三個 Azure SQL 資料庫 
- 下載範例專案：[Azure SQL 的彈性資料庫工具：多租用戶分區](http://go.microsoft.com/?linkid=9888163)
  - 在 **Program.cs** 開頭填寫您的資料庫資訊 

此專案會新增對多租用戶分區資料庫的支援，藉此擴充 [Azure SQL 的彈性資料庫工具：Entity Framework 整合](sql-database-elastic-scale-use-entity-framework-applications-visual-studio.md) 中所述的項目。 專案會建置用於建立部落格和貼文的簡易主控台應用程式。 專案包含四個租用戶，再加上兩個多租用戶分區資料庫。 上圖說明此組態。 

建置並執行應用程式。 這會執行彈性資料庫工具的分區對應管理員啟動程序，並執行下列測試： 

1. 使用 Entity Framework 和 LINQ，建立新的部落格，然後顯示每個租用戶的所有部落格
2. 使用 ADO.NET SqlClient，顯示某個租用戶的所有部落格
3. 嘗試插入錯誤的租用戶部落格，以確認是否擲回錯誤  

請注意，因為分區資料庫中尚未啟用 RLS，所以這些測試都會顯現出一個問題：租用戶能夠查看不屬於自己的部落格，且應用程式無法阻止插入錯誤的租用戶部落格。 本文的其餘部分會說明，如何藉由 RLS 強制執行租用戶隔離來解決這些問題。 有兩個步驟： 

1. **應用程式層**：將應用程式程式碼修改成一律會在連線開啟後，於 SESSION\_CONTEXT 中設定目前的 TenantId。 範例專案已透過此方式設定 TenantId。 
2. **資料層**：在每個分區資料庫中建立 RLS 安全性原則，以根據儲存在 SESSION\_CONTEXT 中的 TenantId 來篩選資料列。 為每個分區資料庫建立原則，否則不會篩選多租用戶分區中的資料列。 

## <a name="1-application-tier-set-tenantid-in-the-sessioncontext"></a>1.應用程式層：設定 SESSION\_CONTEXT 中的 TenantId

首先，使用彈性資料庫用戶端程式庫的資料依存路由 API 來連線到分區資料庫。 應用程式仍必須告訴資料庫正在使用連線的 TenantId 為何。 TenantId 會告訴 RLS 安全性原則必須篩選掉哪些屬於其他租用戶的資料列。 將目前的 TenantId 儲存在連線的 [SESSION\_CONTEXT](https://docs.microsoft.com/sql/t-sql/functions/session-context-transact-sql) 中。

SESSION\_CONTEXT 的替代方式是使用 [CONTEXT\_INFO](https://docs.microsoft.com/sql/t-sql/functions/context-info-transact-sql)。 但 SESSION\_CONTEXT 是比較好的選項。 SESSION\_CONTEXT 較容易使用，它預設會傳回 NULL，而且它支援機碼值組。

### <a name="entity-framework"></a>Entity Framework

對於使用 Entity Framework 的應用程式，最簡單的方法是在[使用 EF DbContext 的資料相依路由](sql-database-elastic-scale-use-entity-framework-applications-visual-studio.md#data-dependent-routing-using-ef-dbcontext)一文中所述的 ElasticScaleContext 覆寫中設定 SESSION\_CONTEXT。 建立並執行 SqlCommand，以將 SESSION\_CONTEXT 中的 TenantId 設為連線的指定 shardingKey。 然後傳回透過資料依存路由代理的連線。 如此一來，您只需要撰寫程式碼一次，就能設定 SESSION\_CONTEXT。 

```csharp
// ElasticScaleContext.cs 
// Constructor for data-dependent routing.
// This call opens a validated connection that is routed to the
// proper shard by the shard map manager.
// Note that the base class constructor call fails for an open connection 
// if migrations need to be done and SQL credentials are used.
// This is the reason for the separation of constructors.
// ...
public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
    : base(
        OpenDDRConnection(shardMap, shardingKey, connectionStr),
        true)  // contextOwnsConnection
{
}

public static SqlConnection OpenDDRConnection(
    ShardMap shardMap,
    T shardingKey,
    string connectionStr)
{
    // No initialization.
    Database.SetInitializer<ElasticScaleContext<T>>(null);

    // Ask shard map to broker a validated connection for the given key.
    SqlConnection conn = null;
    try
    {
        conn = shardMap.OpenConnectionForKey(
            shardingKey,
            connectionStr,
            ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", shardingKey);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
} 
// ... 
```

現在，每次叫用 ElasticScaleContext 時，就會自動使用指定的 TenantId 來設定 SESSION\_CONTEXT： 

```csharp
// Program.cs 
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() => 
{   
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))   
    {     
        var query = from b in db.Blogs
                    orderby b.Name
                    select b;

        Console.WriteLine("All blogs for TenantId {0}:", tenantId);     
        foreach (var item in query)     
        {       
            Console.WriteLine(item.Name);     
        }   
    } 
}); 
```

### <a name="adonet-sqlclient"></a>ADO.NET SqlClient

針對使用 ADO.NET SqlClient 的應用程式，在方法 ShardMap.OpenConnectionForKey 周圍建立包裝函式。 使包裝函式在連線傳回之前，自動將 SESSION\_CONTEXT 中的 TenantId 設定成目前的 TenantId。 若要確保 SESSION\_CONTEXT 一律是設定好的，請務必只使用此包裝函式來開啟連線。

```csharp
// Program.cs
// Wrapper function for ShardMap.OpenConnectionForKey() that
// automatically sets SESSION_CONTEXT with the correct
// tenantId before returning a connection.
// As a best practice, you should only open connections using this method
// to ensure that SESSION_CONTEXT is always set before executing a query.
// ...
public static SqlConnection OpenConnectionForTenant(
    ShardMap shardMap, int tenantId, string connectionStr)
{
    SqlConnection conn = null;
    try
    {
        // Ask shard map to broker a validated connection for the given key.
        conn = shardMap.OpenConnectionForKey(
            tenantId, connectionStr, ConnectionOptions.Validate);

        // Set TenantId in SESSION_CONTEXT to shardingKey
        // to enable Row-Level Security filtering.
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText =
            @"exec sp_set_session_context
                @key=N'TenantId', @value=@shardingKey";
        cmd.Parameters.AddWithValue("@shardingKey", tenantId);
        cmd.ExecuteNonQuery();

        return conn;
    }
    catch (Exception)
    {
        if (conn != null)
        {
            conn.Dispose();
        }
        throw;
    }
}

// ...

// Example query via ADO.NET SqlClient.
// If row-level security is enabled, only Tenant 4's blogs are listed.
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (SqlConnection conn = OpenConnectionForTenant(
        sharding.ShardMap, tenantId4, connStrBldr.ConnectionString))
    {
        SqlCommand cmd = conn.CreateCommand();
        cmd.CommandText = @"SELECT * FROM Blogs";

        Console.WriteLine(@"--
All blogs for TenantId {0} (using ADO.NET SqlClient):", tenantId4);

        SqlDataReader reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine("{0}", reader["Name"]);
        }
    }
});

```

## <a name="2-data-tier-create-row-level-security-policy"></a>2.資料層：建立資料列層級安全性原則

### <a name="create-a-security-policy-to-filter-the-rows-each-tenant-can-access"></a>建立安全性原則來篩選每個租用戶可以存取的資料列

現在應用程式會在查詢之前，使用目前的 TenantId 來設定 SESSION\_CONTEXT，RLS 安全性原則可篩選查詢並排除具有不同 TenantId 的資料列。  

RLS 已實作於 Transact-SQL 中。 使用者定義的函式會定義存取邏輯，而安全性原則會將此函式繫結至任意數目的資料表。 針對此專案：

1. 此函式會確認應用程式已連線至資料庫，且儲存在 SESSION\_CONTEXT 中的 TenantId 符合指定之資料列的 TenantId。
    - 已與應用程式建立連線，而不是與其他 SQL 使用者。

2. FILTER 述詞讓符合 TenantId 篩選條件的資料列能夠通過 SELECT、UPDATE 和 DELETE 查詢。
    - BLOCK 述詞會防止系統在被篩選條件篩選掉的資料列上執行 INSERT 或 UPDATE 操作。
    - 如果尚未設定 SESSION\_CONTEXT，則函式會傳回 NULL，而且看不見或無法插入任何資料列。 

若要為所有分區啟用 RLS，請使用 Visual Studio (SSDT)、SSMS 或專案中包含的 PowerShell 指令碼來執行下列 T-SQL。 或者，如果您是使用[彈性資料庫工作](sql-database-elastic-jobs-overview.md)，則可以將此 T-SQL 在所有分區上的執行自動化。

```sql
CREATE SCHEMA rls; -- Separate schema to organize RLS objects.
GO

CREATE FUNCTION rls.fn_tenantAccessPredicate(@TenantId int)     
    RETURNS TABLE     
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult
        -- Use the user in your application’s connection string.
        -- Here we use 'dbo' only for demo purposes!
        WHERE DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo')
        AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId;
GO

CREATE SECURITY POLICY rls.tenantAccessPolicy
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Blogs,
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.Posts;
GO 
```

> [!TIP]
> 在複雜的專案中，您可能需要將述詞新增到數百個資料表，這麼做很費時。 協助程式預存程序可以自動產生安全性原則，並為結構描述中的所有資料表加入述詞。 如需詳細資訊，請參閱部落格文章[將資料列層級安全性套用至所有資料表 - 協助程式指令碼 (部落格)](http://blogs.msdn.com/b/sqlsecurity/archive/2015/03/31/apply-row-level-security-to-all-tables-helper-script) \(英文\)。

現在，如果您再次執行範例應用程式，租用戶只會看到屬於自己的資料列。 此外，應用程式無法插入目前沒有連線到分區資料庫的租用戶所屬的資料列。 此外，應用程式無法更新它可以看到之任何資料列中的 TenantId。 如果應用程式嘗試執行任一作業，就會引發 DbUpdateException。

如果您是在之後加入新的資料表，請變更 (ALTER) 安全性原則，並在新的資料表上加入 FILTER 和 BLOCK 述詞。

```sql
ALTER SECURITY POLICY rls.tenantAccessPolicy     
    ADD FILTER PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable,
    ADD BLOCK  PREDICATE rls.fn_tenantAccessPredicate(TenantId) ON dbo.MyNewTable;
GO 
```

### <a name="add-default-constraints-to-automatically-populate-tenantid-for-inserts"></a>加入預設條件約束來為插入自動填入 TenantId

您可以在每個資料表上放置預設條件約束，以在插入資料列時，以 SESSION\_CONTEXT 中目前儲存的值自動填入 TenantId。 範例如下。 

```sql
-- Create default constraints to auto-populate TenantId with the
-- value of SESSION_CONTEXT for inserts.
ALTER TABLE Blogs     
    ADD CONSTRAINT df_TenantId_Blogs      
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO

ALTER TABLE Posts     
    ADD CONSTRAINT df_TenantId_Posts      
    DEFAULT CAST(SESSION_CONTEXT(N'TenantId') AS int) FOR TenantId;
GO 
```

現在應用程式不需要在插入資料列時指定 TenantId： 

```csharp
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() => 
{   
    using (var db = new ElasticScaleContext<int>(
        sharding.ShardMap, tenantId, connStrBldr.ConnectionString))
    {
        // The default constraint sets TenantId automatically!
        var blog = new Blog { Name = name };
        db.Blogs.Add(blog);     
        db.SaveChanges();   
    } 
}); 
```

> [!NOTE]
> 如果您針對 Entity Framework 專案使用預設條件約束，建議您「不要」在 EF 資料模型中包含 TenantId 資料行。 此建議的原因是 Entity Framework 查詢會自動提供預設值，以覆寫 T-SQL 中使用 SESSION\_CONTEXT 建立的預設條件約束。
> 若要使用範例專案中的預設條件約束，舉例來說，您可以從 DataClasses.cs 移除 TenantId (並在 Package Manager Console 中執行 Add-Migration)，然後使用 T-SQL 確保欄位只存在於資料庫資料表中。 如此一來，EF 就不會在插入資料時，自動提供不正確的預設值。

### <a name="optional-enable-a-superuser-to-access-all-rows"></a>(選擇性) 啟用「進階使用者」來存取所有資料列

某些應用程式可能會想要建立能存取所有資料列的「進階使用者」。 進階使用者可以啟用跨所有分區上之租用戶的報告。 進階使用者也可以對涉及在資料庫之間移動資料列的分區執行分割合併作業。

若要啟用進階使用者，請在每個分區資料庫中建立新的 SQL 使用者 (在此範例中為 `superuser`)。 然後使用新的述詞函式修改安全性原則，允許此使用者存取所有資料列。 此類函式如下。

```sql
-- New predicate function that adds superuser logic.
CREATE FUNCTION rls.fn_tenantAccessPredicateWithSuperUser(@TenantId int)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_accessResult 
        WHERE 
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('dbo') -- Replace 'dbo'.
            AND CAST(SESSION_CONTEXT(N'TenantId') AS int) = @TenantId
        ) 
        OR
        (
            DATABASE_PRINCIPAL_ID() = DATABASE_PRINCIPAL_ID('superuser')
        );
GO

-- Atomically swap in the new predicate function on each table.
ALTER SECURITY POLICY rls.tenantAccessPolicy
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Blogs,
    ALTER FILTER PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts,
    ALTER BLOCK  PREDICATE rls.fn_tenantAccessPredicateWithSuperUser(TenantId) ON dbo.Posts;
GO
```


### <a name="maintenance"></a>維護 

- **新增分區**：執行 T-SQL 指令碼來啟用所有新分區上的 RLS，否則系統不會篩選這些分區的查詢。
- **新增資料表**：在每次建立新資料表時，將 FILTER 和 BLOCK 述詞新增到所有分區上的安全性原則。 否則，系統不會篩選針對新資料表的查詢。 如[自動將資料列層級安全性套用至新建立的資料表 (部落格)](http://blogs.msdn.com/b/sqlsecurity/archive/2015/05/22/apply-row-level-security-automatically-to-newly-created-tables.aspx) \(英文\) 中所述，此新增動作可以使用 DDL 觸發程序來自動執行。

## <a name="summary"></a>摘要

您可以將彈性資料庫工具與資料列層級安全性搭配使用，以支援多租用戶和單一租用戶的分區，藉此向外延展應用程式的資料層。 多租用戶分區可以用來更有效率地儲存資料。 當大量租用戶只有少量資料列的資料時，此效率特別顯著。 單一租用戶分區可支援效能和隔離需求更嚴格的進階租用戶。  如需詳細資訊，請參閱[資料列層級安全性參考資料][rls]。

## <a name="additional-resources"></a>其他資源

- [什麼是 Azure 彈性集區？](sql-database-elastic-pool.md)
- [使用 Azure SQL Database 相應放大](sql-database-elastic-scale-introduction.md)
- [多租用戶 SaaS 應用程式與 Azure SQL Database 的設計模式](saas-tenancy-app-design-patterns.md)
- [使用 Azure AD 和 OpenID Connect 的多租用戶應用程式驗證](../guidance/guidance-multitenant-identity-authenticate.md)
- [Tailspin Surveys 應用程式](../guidance/guidance-multitenant-identity-tailspin.md)

## <a name="questions-and-feature-requests"></a>問題和功能要求

如有問題，您可以在 [SQL Database 論壇](http://social.msdn.microsoft.com/forums/azure/home?forum=ssdsgetstarted) 上連絡我們。 在 [SQL Database 意見反應論壇](https://feedback.azure.com/forums/217321-sql-database/) \(英文\) 中提出功能要求。


<!--Image references-->
[1]: ./media/saas-tenancy-elastic-tools-multi-tenant-row-level-security/blogging-app.png
<!--anchors-->
[rls]: https://docs.microsoft.com/sql/relational-databases/security/row-level-security
[s-d-elastic-database-client-library]: sql-database-elastic-database-client-library.md

