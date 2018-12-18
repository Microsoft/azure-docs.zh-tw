---
title: 說明如何使用 Redis 快取建立會使用另行快取模式的 Web 應用程式的教學課程 | Microsoft Docs
description: 了解如何使用 Redis 快取建立會使用另行快取模式的 Web 應用程式
services: redis-cache
documentationcenter: ''
author: wesmc7777
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/30/2018
ms.author: wesmc
ms.openlocfilehash: b0f1ba7f0799fb9f8f58370a39758416eab43dc7
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32195799"
---
# <a name="tutorial-create-a-cache-aside-leaderboard-on-aspnet"></a>教學課程：在 ASP.NET 上建立另行快取排行榜

在本教學課程中，您將會更新 ContosoTeamStats ASP.NET Web 應用程式 (在 [Azure Redis 快取的 ASP.NET 快速入門](cache-web-app-howto.md)中所建立)，以納入會搭配 Azure Redis 快取使用[另行快取模式](https://docs.microsoft.com/azure/architecture/patterns/cache-aside)的排行榜。 此應用程式範例會顯示資料庫中的隊伍統計資料清單，並示範各種使用 Azure Redis 快取在快取中儲存和擷取資料以改進效能的方式。 完成本教學課程時，您將擁有執行中的 Web 應用程式，其可對資料庫進行讀取和寫入、已使用 Azure Redis 快取進行最佳化，並裝載在 Azure 中。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 透過使用 Azure Redis 快取來儲存和擷取資料，以改善資料輸送量並降低資料庫負載。
> * 使用 Redis 的已排序集合來擷取前 5 名的隊伍。
> * 使用 Resource Manager 範本為應用程式佈建 Azure 資源。
> * 使用 Visual Studio 將應用程式發佈至 Azure。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>先決條件

若要完成本教學課程，您必須具備下列先決條件：

* 本教學課程會從 [Azure Redis 快取的 ASP.NET 快速入門](cache-web-app-howto.md)的中斷處接續下去。 如果您尚未進行，請先依照該快速入門進行。
* 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)：
    * ASP.NET 和 Web 開發
    * Azure 開發
    * 使用 SQL Server Express LocalDB 或 [SQL Server 2017 Express Edition](https://www.microsoft.com/sql-server/sql-server-editions-express) 的 .NET 桌面開發。

## <a name="add-a-leaderboard-to-the-project"></a>將排行榜新增至專案

在教學課程的這一節當中，您會為 ContosoTeamStats 專案設定一個排行榜，以報告一份清單中各虛擬隊伍輸、贏、平手的統計資料。

### <a name="add-the-entity-framework-to-the-project"></a>將 Entity Framework 新增至專案

1. 在 Visual Studio 中，開啟您在 [Azure Redis 快取的 ASP.NET 快速入門](cache-web-app-howto.md)中所建立的 ContosoTeamStats 解決方案。
2. 按一下 [工具] > [NuGet 套件管理員] > [套件管理員主控台]。
3. 從 [套件管理員主控台] 視窗中執行下列命令，以安裝 EntityFramework：

    ```powershell
    Install-Package EntityFramework
    ```

如需此套件的詳細資訊，請參閱 [EntityFramework](https://www.nuget.org/packages/EntityFramework/) NuGet 頁面。

### <a name="add-the-team-model"></a>新增隊伍模型

1. 在 [方案總管] 中以滑鼠右鍵按一下 [模型]，並選擇 [新增][類別]。

1. 針對類別名稱輸入 `Team`，然後按一下 [新增]。

    ![新增模型類別](./media/cache-web-app-cache-aside-leaderboard/cache-model-add-class-dialog.png)

1. 將 Team.cs 檔案頂端的 `using` 陳述式替換為下列 `using` 陳述式：

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Data.Entity;
    using System.Data.Entity.SqlServer;
    ```

1. 將 `Team` 類別的定義取代為包含已更新的 `Team` 類別定義以及一些其他 Entity Framework 協助程式類別的下列程式碼片段。 本教學課程會對 Entity Framework 使用 Code First 方法。 這個方法可讓 Entity Framework 從程式碼建立資料庫。 如需本教學課程中使用之 Entity Framework 的 Code First 方法的詳細資訊，請參閱 [Code First 至新的資料庫](https://msdn.microsoft.com/data/jj193542)。

    ```csharp
    public class Team
    {
        public int ID { get; set; }
        public string Name { get; set; }
        public int Wins { get; set; }
        public int Losses { get; set; }
        public int Ties { get; set; }

        static public void PlayGames(IEnumerable<Team> teams)
        {
            // Simple random generation of statistics.
            Random r = new Random();

            foreach (var t in teams)
            {
                t.Wins = r.Next(33);
                t.Losses = r.Next(33);
                t.Ties = r.Next(0, 5);
            }
        }
    }

    public class TeamContext : DbContext
    {
        public TeamContext()
            : base("TeamContext")
        {
        }

        public DbSet<Team> Teams { get; set; }
    }

    public class TeamInitializer : CreateDatabaseIfNotExists<TeamContext>
    {
        protected override void Seed(TeamContext context)
        {
            var teams = new List<Team>
            {
                new Team{Name="Adventure Works Cycles"},
                new Team{Name="Alpine Ski House"},
                new Team{Name="Blue Yonder Airlines"},
                new Team{Name="Coho Vineyard"},
                new Team{Name="Contoso, Ltd."},
                new Team{Name="Fabrikam, Inc."},
                new Team{Name="Lucerne Publishing"},
                new Team{Name="Northwind Traders"},
                new Team{Name="Consolidated Messenger"},
                new Team{Name="Fourth Coffee"},
                new Team{Name="Graphic Design Institute"},
                new Team{Name="Nod Publishers"}
            };

            Team.PlayGames(teams);

            teams.ForEach(t => context.Teams.Add(t));
            context.SaveChanges();
        }
    }

    public class TeamConfiguration : DbConfiguration
    {
        public TeamConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
        }
    }
    ```

1. 在 [方案總管] 中，連按兩下 [web.config] 來加以開啟。

    ![Web.config](./media/cache-web-app-cache-aside-leaderboard/cache-web-config.png)

1. 將下列 `connectionStrings` 區段新增至 `configuration` 區段中。 連接字串的名稱必須符合 Entity Framework 資料庫內容類別的名稱，亦即 `TeamContext`。

    這個連接字串假設您已符合[先決條件](#prerequisites)並已安裝 SQL Server Express LocalDB (這是隨 Visual Studio 2017 一起安裝的 .NET 桌面開發工作負載的一部分)。

    ```xml
    <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    在 `configuration` 區段中，新的 `connectionStrings` 區段緊接在 `configSections` 後面，如下列範例所示：

    ```xml
    <configuration>
        <configSections>
        <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" requirePermission="false" />
        </configSections>
        <connectionStrings>
        <add name="TeamContext" connectionString="Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\Teams.mdf;Integrated Security=True"     providerName="System.Data.SqlClient" />
        </connectionStrings>
        ...
    ```

### <a name="add-the-teamscontroller-and-views"></a>新增 TeamsController 和檢視

1. 在 Visual Studio 中建置專案。 

1. 在 [方案總管] 中，於 [控制器] 資料夾上按一下滑鼠右鍵，然後依序選擇 [新增] 和 [控制器]。

1. 選擇 [使用 Entity Framework，包含檢視的 MVC 5 控制器]，然後按一下 [新增]。 如果您在按一下 [新增] 後收到錯誤，請確定您已先建置專案。

    ![新增控制器類別](./media/cache-web-app-cache-aside-leaderboard/cache-add-controller-class.png)

1. 在 [模型類別] 下拉式清單中選取 [Team (ContosoTeamStats.Models)]。 在 [資料內容類別] 下拉式清單中選取 [TeamContext (ContosoTeamStats.Models)]。 在 [控制器名稱] 文字方塊中輸入 `TeamsController` (如果尚未自動填入)。 按一下 [新增]  以建立控制器類別並新增預設檢視。

    ![設定控制器](./media/cache-web-app-cache-aside-leaderboard/cache-configure-controller.png)

1. 在 [方案總管] 中展開 [Global.asax]，然後按兩下 [Global.asax.cs] 來加以開啟。

    ![Global.asax.cs](./media/cache-web-app-cache-aside-leaderboard/cache-global-asax.png)

1. 在檔案頂端的其他 `using` 陳述式底下新增下列兩個 `using` 陳述式：

    ```csharp
    using System.Data.Entity;
    using ContosoTeamStats.Models;
    ```

1. 在 `Application_Start` 方法的結尾新增下列程式碼行：

    ```csharp
    Database.SetInitializer<TeamContext>(new TeamInitializer());
    ```

1. 在 [方案總管] 中展開 `App_Start`，然後按兩下 `RouteConfig.cs`。

    ![RouteConfig.cs](./media/cache-web-app-cache-aside-leaderboard/cache-RouteConfig-cs.png)

1. 在 `RegisterRoutes` 方法中，將 `Default` 路由中的 `controller = "Home"` 替換為 `controller = "Teams"`，如下列範例所示：

    ```csharp
    routes.MapRoute(
        name: "Default",
        url: "{controller}/{action}/{id}",
        defaults: new { controller = "Teams", action = "Index", id = UrlParameter.Optional }
    );
    ```

### <a name="configure-the-layout-view"></a>設定版面配置檢視

1. 在 [方案總管] 中依序展開 [檢視] 資料夾和 [共用] 資料夾，然後按兩下 **_Layout.cshtml**。 

    ![_Layout.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml.png)

1. 變更 `title` 元素的內容，並將 `My ASP.NET Application` 替換為 `Contoso Team Stats`，如下列範例所示：

    ```html
    <title>@ViewBag.Title - Contoso Team Stats</title>
    ```

1. 在 `body` 區段中，於「Azure Redis 快取測試」的連結正下方，為「Contoso Team Stats」新增下列新的 `Html.ActionLink` 陳述式。

    ```csharp
    @Html.ActionLink("Contoso Team Stats", "Index", "Teams", new { area = "" }, new { @class = "navbar-brand" })`
    ```

    ![程式碼變更](./media/cache-web-app-cache-aside-leaderboard/cache-layout-cshtml-code.png)

1. 按 **Ctrl+F5** 以建置並執行應用程式。 這個版本的應用程式會直接從資料庫讀取結果。 請注意透過 [使用 Entity Framework，包含檢視的 MVC 5 控制器] scaffold 自動新增至應用程式的 [建立新的]、[編輯]、[詳細資料] 和 [刪除] 動作。 在本教學課程的下一節裡，您將會新增 Redis 快取以將資料存取最佳化並為應用程式提供其他功能。

    ![起始應用程式](./media/cache-web-app-cache-aside-leaderboard/cache-starter-application.png)

## <a name="configure-the-app-for-redis-cache"></a>設定 Redis 快取的應用程式

在教學課程的這一節當中，您會設定範例應用程式，讓其使用 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) 快取用戶端在 Azure Redis 快取執行個體中儲存和擷取 Contoso 的隊伍統計資料。

### <a name="add-a-cache-connection-to-the-teams-controller"></a>將快取連線新增至隊伍控制器

您已在快速入門中安裝 StackExchange.Redis 用戶端程式庫套件。 您也已經將「CacheConnection 應用程式設定」設定為要在本機與已發佈的 App Service 搭配使用。 請在 TeamsController 中使用這個相同的用戶端程式庫和 CacheConnection 資訊。

1. 在 [方案總管] 中展開 [控制器] 資料夾，然後按兩下 [TeamsController.cs] 來加以開啟。

    ![隊伍控制器](./media/cache-web-app-cache-aside-leaderboard/cache-teamscontroller.png)

1. 在 **TeamsController.cs** 中新增下列兩個 `using` 陳述式：

    ```csharp
    using System.Configuration;
    using StackExchange.Redis;
    ```

1. 將下列兩個屬性新增至 `TeamsController` 類別：

    ```csharp
    // Redis Connection string info
    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
        return ConnectionMultiplexer.Connect(cacheConnection);
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }
    ```

### <a name="update-the-teamscontroller-to-read-from-the-cache-or-the-database"></a>將 TeamsController 更新為讀取快取或資料庫

在此範例中，隊伍統計資料可以擷取自資料庫或快取。 隊伍統計資料可儲存在快取中做為序列化的 `List<Team>`，也可以使用 Redis 資料類型做為已排序集合。 從已排序集合擷取項目時，您可以擷取部分或所有項目，或是查詢特定項目。 在此範例中，您將查詢已排序集合內獲勝次數排在前 5 名的隊伍。

想要使用 Azure Redis 快取並不需要將隊伍統計資料以多種格式儲存在快取中。 本教學課程之所以使用多種格式，是為了示範某些可用來快取資料的不同方式和不同資料類型。

1. 在 `TeamsController.cs` 檔案頂端新增下列 `using` 陳述式，和其他 `using` 陳述式放在一起：

    ```csharp
    using System.Diagnostics;
    using Newtonsoft.Json;
    ```

1. 將目前的 `public ActionResult Index()` 方法實作替換為下列實作：

    ```csharp
    // GET: Teams
    public ActionResult Index(string actionType, string resultType)
    {
        List<Team> teams = null;

        switch(actionType)
        {
            case "playGames": // Play a new season of games.
                PlayGames();
                break;

            case "clearCache": // Clear the results from the cache.
                ClearCachedTeams();
                break;

            case "rebuildDB": // Rebuild the database with sample data.
                RebuildDB();
                break;
        }

        // Measure the time it takes to retrieve the results.
        Stopwatch sw = Stopwatch.StartNew();

        switch(resultType)
        {
            case "teamsSortedSet": // Retrieve teams from sorted set.
                teams = GetFromSortedSet();
                break;

            case "teamsSortedSetTop5": // Retrieve the top 5 teams from the sorted set.
                teams = GetFromSortedSetTop5();
                break;

            case "teamsList": // Retrieve teams from the cached List<Team>.
                teams = GetFromList();
                break;

            case "fromDB": // Retrieve results from the database.
            default:
                teams = GetFromDB();
                break;
        }

        sw.Stop();
        double ms = sw.ElapsedTicks / (Stopwatch.Frequency / (1000.0));

        // Add the elapsed time of the operation to the ViewBag.msg.
        ViewBag.msg += " MS: " + ms.ToString();

        return View(teams);
    }
    ```

1. 在 `TeamsController` 類別中新增下列三種方法，實作來自先前程式碼片段中新增之 switch 陳述式中的 `playGames`、`clearCache` 和 `rebuildDB` 動作類型。

    `PlayGames` 方法會藉由模擬一個賽季的遊戲來更新隊伍統計資料、將結果儲存至資料庫，並清除快取中現已過時的資料。

    ```csharp
    void PlayGames()
    {
        ViewBag.msg += "Updating team statistics. ";
        // Play a "season" of games.
        var teams = from t in db.Teams
                    select t;

        Team.PlayGames(teams);

        db.SaveChanges();

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    `RebuildDB` 方法會以一組預設的隊伍重新初始化資料庫、為這些隊伍產生統計資料，並清除快取中現已過時的資料。

    ```csharp
    void RebuildDB()
    {
        ViewBag.msg += "Rebuilding DB. ";
        // Delete and re-initialize the database with sample data.
        db.Database.Delete();
        db.Database.Initialize(true);

        // Clear any cached results
        ClearCachedTeams();
    }
    ```

    `ClearCachedTeams` 方法會移除快取中任何已快取過的隊伍統計資料。

    ```csharp
    void ClearCachedTeams()
    {
        IDatabase cache = Connection.GetDatabase();
        cache.KeyDelete("teamsList");
        cache.KeyDelete("teamsSortedSet");
        ViewBag.msg += "Team data removed from cache. ";
    }
    ```

1. 在 `TeamsController` 類別中加入下列四種方法來實作各種從快取和資料庫中擷取隊伍統計資料的方式。 這四種方法皆會傳回 `List<Team>` 以在檢視中顯示。

    `GetFromDB` 方法會從資料庫讀取隊伍統計資料。
    ```csharp
    List<Team> GetFromDB()
    {
        ViewBag.msg += "Results read from DB. ";
        var results = from t in db.Teams
            orderby t.Wins descending
            select t;

        return results.ToList<Team>();
    }
    ```

    `GetFromList` 方法會從快取中讀取序列化 `List<Team>` 形式的隊伍統計資料。 如果快取中沒有統計資料，就會發生快取遺漏。 快取遺漏時，系統會從資料庫讀取隊伍統計資料，然後儲存在快取中以供下一個要求使用。 此範例會使用 JSON.NET 序列化在快取中傳入和傳出序列化的 .NET 物件。 如需詳細資訊，請參閱 [如何在 Azure Redis 快取中使用 .NET 物件](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache)。

    ```csharp
    List<Team> GetFromList()
    {
        List<Team> teams = null;

        IDatabase cache = Connection.GetDatabase();
        string serializedTeams = cache.StringGet("teamsList");
        if (!String.IsNullOrEmpty(serializedTeams))
        {
            teams = JsonConvert.DeserializeObject<List<Team>>(serializedTeams);

            ViewBag.msg += "List read from cache. ";
        }
        else
        {
            ViewBag.msg += "Teams list cache miss. ";
            // Get from database and store in cache
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            cache.StringSet("teamsList", JsonConvert.SerializeObject(teams));
        }
        return teams;
    }
    ```

    `GetFromSortedSet` 方法會從快取的已排序集合讀取隊伍統計資料。 如果發生快取遺漏情形，便會從資料庫讀取隊伍統計資料，然後儲存在快取中做為已排序集合。

    ```csharp
    List<Team> GetFromSortedSet()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();
        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", order: Order.Descending);
        if (teamsSortedSet.Count() > 0)
        {
            ViewBag.msg += "Reading sorted set from cache. ";
            teams = new List<Team>();
            foreach (var t in teamsSortedSet)
            {
                Team tt = JsonConvert.DeserializeObject<Team>(t.Element);
                teams.Add(tt);
            }
        }
        else
        {
            ViewBag.msg += "Teams sorted set cache miss. ";

            // Read from DB
            teams = GetFromDB();

            ViewBag.msg += "Storing results to cache. ";
            foreach (var t in teams)
            {
                Console.WriteLine("Adding to sorted set: {0} - {1}", t.Name, t.Wins);
                cache.SortedSetAdd("teamsSortedSet", JsonConvert.SerializeObject(t), t.Wins);
            }
        }
        return teams;
    }
    ```

    `GetFromSortedSetTop5` 方法會從快取的已排序集合讀取前 5 名的隊伍。 它會先檢查快取中是否有 `teamsSortedSet` 索引鍵。 如果這個索引鍵不存在，便會呼叫 `GetFromSortedSet` 方法來讀取隊伍統計資料，並將資料儲存在快取中。 接著便會查詢快取的已排序集合中傳回的前 5 名隊伍。

    ```csharp
    List<Team> GetFromSortedSetTop5()
    {
        List<Team> teams = null;
        IDatabase cache = Connection.GetDatabase();

        // If the key teamsSortedSet is not present, this method returns a 0 length collection.
        var teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        if(teamsSortedSet.Count() == 0)
        {
            // Load the entire sorted set into the cache.
            GetFromSortedSet();

            // Retrieve the top 5 teams.
            teamsSortedSet = cache.SortedSetRangeByRankWithScores("teamsSortedSet", stop: 4, order: Order.Descending);
        }

        ViewBag.msg += "Retrieving top 5 teams from cache. ";
        // Get the top 5 teams from the sorted set
        teams = new List<Team>();
        foreach (var team in teamsSortedSet)
        {
            teams.Add(JsonConvert.DeserializeObject<Team>(team.Element));
        }
        return teams;
    }
    ```

### <a name="update-the-create-edit-and-delete-methods-to-work-with-the-cache"></a>更新用來使用快取的建立、編輯和刪除方法

此範例進行過程所產生的樣板程式碼包括用來新增、編輯和刪除隊伍的方法。 每當您加入、編輯或移除隊伍時，快取中的資料就會變得過時。 在本節中，您將會修改這三種方法來清除已快取的隊伍，讓快取不會重新整理。

1. 瀏覽至 `TeamsController` 類別中的 `Create(Team team)` 方法。 在 `ClearCachedTeams` 方法中新增呼叫，如下列範例所示：

    ```csharp
    // POST: Teams/Create
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Create([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Teams.Add(team);
            db.SaveChanges();
            // When a team is added, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }

        return View(team);
    }
    ```

2. 瀏覽至 `TeamsController` 類別中的 `Edit(Team team)` 方法。 在 `ClearCachedTeams` 方法中新增呼叫，如下列範例所示：

    ```csharp
    // POST: Teams/Edit/5
    // To protect from overposting attacks, please enable the specific properties you want to bind to, for 
    // more details see http://go.microsoft.com/fwlink/?LinkId=317598.
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Edit([Bind(Include = "ID,Name,Wins,Losses,Ties")] Team team)
    {
        if (ModelState.IsValid)
        {
            db.Entry(team).State = EntityState.Modified;
            db.SaveChanges();
            // When a team is edited, the cache is out of date.
            // Clear the cached teams.
            ClearCachedTeams();
            return RedirectToAction("Index");
        }
        return View(team);
    }
    ```

3. 瀏覽至 `TeamsController` 類別中的 `DeleteConfirmed(int id)` 方法。 在 `ClearCachedTeams` 方法中新增呼叫，如下列範例所示：

    ```csharp
    // POST: Teams/Delete/5
    [HttpPost, ActionName("Delete")]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteConfirmed(int id)
    {
        Team team = db.Teams.Find(id);
        db.Teams.Remove(team);
        db.SaveChanges();
        // When a team is deleted, the cache is out of date.
        // Clear the cached teams.
        ClearCachedTeams();
        return RedirectToAction("Index");
    }
    ```

### <a name="add-caching-methods-to-the-teams-index-view"></a>將快取方法新增至隊伍索引檢視

1. 在 [方案總管] 中依序展開 [檢視] 資料夾和 [隊伍] 資料夾，然後按兩下 [Index.cshtml]。

    ![Index.cshtml](./media/cache-web-app-cache-aside-leaderboard/cache-views-teams-index-cshtml.png)

1. 在檔案頂端附近尋找下列段落元素：

    ![動作資料表](./media/cache-web-app-cache-aside-leaderboard/cache-teams-index-table.png)

    此連結會建立新的隊伍。 請將此段落元素取代為下列資料表。 這個資料表內有動作連結，可供建立新的隊伍、進行新一賽季的遊戲、清除快取、從快取中以數種格式擷取隊伍、從資料庫中擷取隊伍，以及使用全新的範例資料重建資料庫。

    ```html
    <table class="table">
        <tr>
            <td>
                @Html.ActionLink("Create New", "Create")
            </td>
            <td>
                @Html.ActionLink("Play Season", "Index", new { actionType = "playGames" })
            </td>
            <td>
                @Html.ActionLink("Clear Cache", "Index", new { actionType = "clearCache" })
            </td>
            <td>
                @Html.ActionLink("List from Cache", "Index", new { resultType = "teamsList" })
            </td>
            <td>
                @Html.ActionLink("Sorted Set from Cache", "Index", new { resultType = "teamsSortedSet" })
            </td>
            <td>
                @Html.ActionLink("Top 5 Teams from Cache", "Index", new { resultType = "teamsSortedSetTop5" })
            </td>
            <td>
                @Html.ActionLink("Load from DB", "Index", new { resultType = "fromDB" })
            </td>
            <td>
                @Html.ActionLink("Rebuild DB", "Index", new { actionType = "rebuildDB" })
            </td>
        </tr>
    </table>
    ```

1. 捲動至 **Index.cshtml** 檔案底部並新增下列 `tr` 元素，使其成為檔案中最後一個資料表內的最後一個資料列：

    ```html
    <tr><td colspan="5">@ViewBag.Msg</td></tr>
    ```
    這個資料列會顯示 `ViewBag.Msg` 的值，其包含目前作業的相關狀態報告。 當您按一下上一個步驟中的任何一個動作連結時，便會設定 `ViewBag.Msg`。

    ![狀態訊息](./media/cache-web-app-cache-aside-leaderboard/cache-status-message.png)

1. 按 **F6** 來建置專案。

## <a name="run-the-app-locally"></a>在本機執行應用程式

在機器本機上執行應用程式，確認已新增功能來支援隊伍。

在這項測試中，應用程式與資料庫都是在本機執行的。 不過，Redis 快取會遠端裝載於 Azure。 因此，快取的效能可能會比資料庫差一些。 為了獲得最佳效能，用戶端應用程式和 Azure Redis 快取執行個體應該位於相同的位置。 在下一節，您會將所有資源部署到 Azure，以看見使用快取所造成的效能提升。

若要在本機執行應用程式：

1. 按 **CTRL+F5** 執行應用程式。

    ![本機執行的應用程式](./media/cache-web-app-cache-aside-leaderboard/cache-local-application.png)

1. 對新增至檢視的每個新方法進行測試。 在這些測試中，由於快取位於遠端，資料庫的效能應該會比快取好一些。

## <a name="publish-and-run-in-azure"></a>在 Azure 中發佈並執行

### <a name="provision-a-sql-azure-database-for-the-app"></a>佈建應用程式的 SQL Azure 資料庫

在本節中，您將會佈建新的 SQL Azure 資料庫，以供應用程式在裝載於 Azure 時使用。

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 Azure 入口網站左上角的 [建立資源]。

1. 在 [新增] 頁面上，按一下 [資料庫] > [SQL Database]。

1. 針對新的 SQL Database 使用下列設定：

   | 設定       | 建議的值 | 說明 |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **資料庫名稱** | ContosoTeamsDatabase | 如需有效的資料庫名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。 |
   | **訂用帳戶** | *您的訂用帳戶*  | 選取您在建立快取和裝載 App Service 時所使用的相同訂用帳戶。 |
   | **資源群組**  | TestResourceGroup | 按一下 [使用現有的]，並使用您用來放置快取和 App Service 的相同資源群組。 |
   | **選取來源** | **空白資料庫** | 以空白資料庫開始。 |

1. 在 [伺服器] 底下，按一下 [進行必要設定] > [建立新的伺服器] 並提供下列資訊，然後按一下 [選取] 按鈕：

   | 設定       | 建議的值 | 說明 |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **伺服器名稱** | 任何全域唯一名稱 | 如需有效的伺服器名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 |
   | **伺服器管理員登入** | 任何有效名稱 | 如需有效的登入名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。 |
   | **密碼** | 任何有效密碼 | 您的密碼至少要有 8 個字元，而且必須包含下列幾種字元的其中三種︰大寫字元、小寫字元、數字和非英數字元。 |
   | **位置** | 美國東部 | 選取您用來建立快取和 App Service 的相同區域。 |

1. 按一下 [釘選到儀表板]，然後按一下 [建立] 以建立新的資料庫和伺服器。

1. 建立新的資料庫後，按一下 [顯示資料庫連接字串]，然後複製 **ADO.NET** 連接字串。

    ![Show connection strings](./media/cache-web-app-cache-aside-leaderboard/cache-show-connection-strings.png)

1. 在 Azure 入口網站中，瀏覽至 App Service 並按一下 [應用程式設定]，然後在 [連接字串] 區段底下**新增連接字串**。

1. 新增名為 TeamContext 的連接字串以符合 Entity Framework 資料庫內容類別。 貼上新資料庫的連接字串作為值。 請務必取代連接字串中的下列預留位置，然後按一下 [儲存]：

    | Placeholder | 建議的值 |
    | --- | --- |
    | {your_username} | 針對您剛才建立的資料庫伺服器使用**伺服器系統管理員登入**。 |
    | {your_password} | 針對您剛才建立的資料庫伺服器使用密碼。 |

    藉由新增使用者名稱和密碼作為應用程式設定，程式碼中就不會包含使用者名稱和密碼。 這種方法可協助您保護這些認證。

### <a name="publish-the-application-updates-to-azure"></a>將應用程式更新發佈至 Azure

在教學課程的這個步驟中，您會將應用程式更新發佈至 Azure 以在雲端中執行。

1. 在 Visual Studio 中以滑鼠右鍵按一下 [ContosoTeamStats] 專案，然後選擇 [發佈]。

    ![發佈](./media/cache-web-app-cache-aside-leaderboard/cache-publish-app.png)

2. 按一下 [發佈]，以使用您在快速入門中所建立的相同發行設定檔。

3. 發佈完成後，Visual Studio 會在預設網頁瀏覽器中啟動應用程式。

    ![已新增快取](./media/cache-web-app-cache-aside-leaderboard/cache-added-to-application.png)

    下表說明範例應用程式中的每個動作連結：

    | 動作 | 說明 |
    | --- | --- |
    | 建立新的 |建立新的隊伍。 |
    | 遊戲賽季 |玩一個賽季的遊戲、更新隊伍統計資料，並清除快取中任何已過時的隊伍資料。 |
    | 清除快取 |清除快取中的隊伍統計資料。 |
    | 快取中的清單 |擷取快取中的隊伍統計資料。 如果發生快取遺漏情形，則載入資料庫中的統計資料，並儲存至快取中以供下次使用。 |
    | 快取中的已排序集合 |使用已排序集合擷取快取中的隊伍統計資料。 如果發生快取遺漏的情形，請從資料庫中載入統計資料，並使用已排序集合儲存至快取中。 |
    | 快取中的前 5 名隊伍 |使用已排序集合擷取快取中的前 5 名隊伍。 如果發生快取遺漏的情形，請從資料庫中載入統計資料，並使用已排序集合儲存至快取中。 |
    | 從資料庫載入 |擷取資料庫中的隊伍統計資料。 |
    | 重建資料庫 |重建資料庫並在其中重新載入範例隊伍資料。 |
    | 編輯/詳細資料/刪除 |編輯隊伍、檢視隊伍的詳細資料、刪除隊伍。 |

對某些動作按一下，然後試驗從不同來源擷取資料。 請注意從資料庫和快取擷取資料的各種方式若要完成所需花費之時間的差異。

## <a name="clean-up-resources"></a>清除資源

當您完成教學課程的範例應用程式時，您可以刪除使用的 Azure 資源以節省成本和資源。 您的所有資源應該都會包含在相同的資源群組內，您可以透過刪除資源群組在單一作業中將它們一起刪除。 本主題的指示使用的是名為 TestResources 的資源群組。

> [!IMPORTANT]
> 刪除資源群組是無法回復的動作，資源群組和其內的所有資源將會永久刪除。 請確定您不會不小心刪除錯誤的資源群組或資源。 如果您是在包含有需要保留之資源的現有資源群組內，建立用來裝載此範例的資源，則可以從每個資源各自的刀鋒視窗中個別刪除每個資源。
>

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後按一下 [資源群組]。
2. 在 [篩選項目...]  文字方塊中輸入您的資源群組名稱。
3. 按一下資源群組右邊的 [...]，然後按一下 [刪除資源群組]。

    ![刪除](./media/cache-web-app-cache-aside-leaderboard/cache-delete-resource-group.png)

4. 系統將會要求您確認是否刪除資源。 輸入您的資源群組名稱來確認，然後按一下 [刪除]。

    片刻過後，系統便會刪除該資源群組及其所有內含的資源。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [如何調整 Azure Redis 快取](./cache-how-to-scale.md)