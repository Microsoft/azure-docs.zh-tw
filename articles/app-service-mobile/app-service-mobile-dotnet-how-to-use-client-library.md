---
title: 使用 App Service Mobile Apps 受控用戶端程式庫 (Windows | Microsoft Docs
description: 了解如何搭配 Windows 和 Xamarin 應用程式針對 Azure App Service Mobile Apps 使用 .NET 用戶端。
services: app-service\mobile
documentationcenter: ''
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 0280785c-e027-4e0d-aaf2-6f155e5a6197
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 09/24/2018
ms.author: crdun
ms.openlocfilehash: 5e399a237fe533b46997365c16b75dce14149dec
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064317"
---
# <a name="how-to-use-the-managed-client-for-azure-mobile-apps"></a>如何針對 Azure Mobile Apps 使用受控用戶端
[!INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

## <a name="overview"></a>概觀
本指南將示範如何在 Windows 和 Xamarin 應用程式中針對 Azure App Service Mobile Apps 使用受控用戶端程式庫，來執行一般案例。 如果您不熟悉 Mobile Apps，應該考慮先完成 [Azure Mobile Apps 快速入門][1]教學課程。 在本指南中，我們會著重於用戶端受控 SDK。 若要深入了解 Mobile Apps 的伺服器端 SDK，請參閱 [.NET 伺服器 SDK][2] 或 [Node.js 伺服器 SDK][3] 的文件。

## <a name="reference-documentation"></a>參考文件
用戶端 SDK 的參考文件位於此處：[Azure Mobile Apps .NET 用戶端參考資料][4]。
您也可以在 [Azure 範例 GitHub 存放庫][5]中找到數個用戶端範例。

## <a name="supported-platforms"></a>支援的平台
.NET 平台支援下列平台︰

* 適用於 API 19 到 24 (KitKat 到 Nougat) 的 Xamarin Android 版本
* 適用於 iOS 8.0 版和更新版本的 Xamarin iOS 版本
* 通用 Windows 平台
* Windows Phone 8.1
* Windows Phone 8.0 (Silverlight 應用程式除外)

「伺服器流程」驗證在呈現的 UI 中使用 WebView。  如果裝置無法呈現 WebView UI，您需要其他驗證方法。  因此，此 SDK 不適用於手錶類型或受到類似限制的裝置。

## <a name="setup"></a>設定和必要條件
我們假設您已建立並發佈您的行動應用程式後端專案 (至少包含一個資料表)。  在此主題使用的程式碼中，資料表的名稱為 `TodoItem`，且其具有下列資料行：`Id`、`Text` 和 `Complete`。 此資料表與您完成 [Azure Mobile Apps 快速入門][1]時所建立的資料表相同。

C# 中對應的具類型用戶端類型為下列類別：

```
public class TodoItem
{
    public string Id { get; set; }

    [JsonProperty(PropertyName = "text")]
    public string Text { get; set; }

    [JsonProperty(PropertyName = "complete")]
    public bool Complete { get; set; }
}
```

[JsonPropertyAttribute][6] 是用來定義用戶端欄位與資料表欄位之間的 *PropertyName* 對應。

若要了解如何在 Mobile Apps 後端中建立資料表，請參閱 [.NET 伺服器 SDK 主題][7] 或 [Node.js 伺服器 SDK 主題][8]。 如果您已使用＜快速入門＞在 Azure 入口網站中建立行動應用程式後端，也可以使用 **Azure 入口網站** 中的 [Azure 入口網站]設定。

### <a name="how-to-install-the-managed-client-sdk-package"></a>做法︰安裝受控用戶端 SDK 套件
使用下列其中一種方法，從 [NuGet][9] 安裝適用於 Mobile Apps 的受控用戶端 SDK 套件：

* **Visual Studio** 以滑鼠右鍵按一下您的專案、按一下 [管理 NuGet 套件]，搜尋 `Microsoft.Azure.Mobile.Client` 套件，然後按一下 [安裝]。
* **Xamarin Studio** 以滑鼠右鍵按一下您的專案、按一下 [新增] > [新增 NuGet 套件]，搜尋 `Microsoft.Azure.Mobile.Client ` 套件，然後按一下 [新增套件]。

在您的主要活動檔案中，記得加入下列 **using** 陳述式：

```
using Microsoft.WindowsAzure.MobileServices;
```

> [!NOTE]
> 請注意，您 Android 專案中所參考的所有支援套件都必須具有相同版本。 此 SDK 具有 Android 平台的 `Xamarin.Android.Support.CustomTabs` 相依性，因此若您的專案使用較新的支援套件，您必須直接安裝具有必要版本的此套件以避免發生衝突。

### <a name="symbolsource"></a>做法︰使用 Visual Studio 中的偵錯符號
您可以從 [SymbolSource][10] 取得適用於 Microsoft.Azure.Mobile 命名空間的符號。  若要將 SymbolSource 與 Visual Studio 整合，請參閱 [SymbolSource 指示][11]。

## <a name="create-client"></a>建立 Mobile Apps 用戶端
下列程式碼會建立用來存取「行動應用程式」後端的 [MobileServiceClient][12] 物件。

```
var client = new MobileServiceClient("MOBILE_APP_URL");
```

在上述程式碼中，以行動應用程式後端 URL 取代 `MOBILE_APP_URL` ，這位於 [Azure 入口網站]的行動應用程式後端刀鋒視窗中。 MobileServiceClient 物件應該是單一的。

## <a name="work-with-tables"></a>使用資料表
下一節將詳細說明如何搜尋和擷取記錄，以及修改資料表中的資料。  此文章涵蓋下列主題：

* [建立資料表參考](#instantiating)
* [查詢資料](#querying)
* [篩選傳回的資料](#filtering)
* [排序傳回的資料](#sorting)
* [以分頁方式傳回資料](#paging)
* [選取特定資料欄](#selecting)
* [依據識別碼查詢記錄](#lookingup)
* [處理不具類型的查詢](#untypedqueries)
* [插入資料](#inserting)
* [更新資料](#updating)
* [刪除資料](#deleting)
* [衝突解決和開放式並行存取](#optimisticconcurrency)
* [繫結至 Windows 使用者介面](#binding)
* [變更頁面大小](#pagesize)

### <a name="instantiating"></a>作法：建立資料表參考
存取或修改後端資料表中資料的所有程式碼，都會在 `MobileServiceTable` 物件上呼叫函數。 透過呼叫 [GetTable] 方法來取得資料表的參考，如下所示：

```
IMobileServiceTable<TodoItem> todoTable = client.GetTable<TodoItem>();
```

傳回的物件會使用具類型的序列化模型。 也支援不具類型的序列化模型。 下列範例會[建立不具類型的資料表參考]：

```
// Get an untyped table reference
IMobileServiceTable untypedTodoTable = client.GetTable("TodoItem");
```

在不具類型的查詢中，您必須指定基礎 OData 查詢字串。

### <a name="querying"></a>如何：查詢行動應用程式中的資料
本節將說明如何對行動應用程式後端發出查詢，包括下列功能：

* [篩選傳回的資料](#filtering)
* [排序傳回的資料](#sorting)
* [以分頁方式傳回資料](#paging)
* [選取特定資料欄](#selecting)
* [按識別碼查詢資料](#lookingup)

> [!NOTE]
> 系統會強制使用伺服器控制的頁面大小，以防止傳回所有資料列。  分頁可避免大型資料集的預設要求對服務造成負面影響。  若要傳回超過 50 個資料列，請依照[以分頁方式傳回資料](#paging)所述，使用 `Skip` 和 `Take` 方法。

### <a name="filtering"></a>作法：篩選傳回的資料
下列程式碼說明如何在查詢中包含 `Where` 子句，以篩選資料。 它會從 `todoTable` 傳回其 `Complete` 屬性等於 `false` 的所有項目。 [Where] 函數會套用資料列篩選述語來查詢資料表。

```
// This query filters out completed TodoItems and items without a timestamp.
List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .ToListAsync();
```

您可以使用訊息檢查軟體 (例如瀏覽器開發人員工具或 [Fiddler]) 來檢視傳送至後端的要求 URI。 如果您查看要求 URI，會發現查詢字串已經過修改：

```
GET /tables/todoitem?$filter=(complete+eq+false) HTTP/1.1
```

伺服器 SDK 會將此 OData 要求轉譯成 SQL 查詢︰

```
SELECT *
    FROM TodoItem
    WHERE ISNULL(complete, 0) = 0
```

傳遞至 `Where` 方法的函式可以有任意數目的條件。

```
// This query filters out completed TodoItems where Text isn't null
List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false && todoItem.Text != null)
    .ToListAsync();
```

伺服器 SDK 會將此範例轉譯成 SQL 查詢︰

```
SELECT *
    FROM TodoItem
    WHERE ISNULL(complete, 0) = 0
          AND ISNULL(text, 0) = 0
```

此查詢也可以分割成多個子句︰

```
List<TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .Where(todoItem => todoItem.Text != null)
    .ToListAsync();
```

這兩種方法是相同的，而且可以交換使用。  舊有的選項&mdash;在一個查詢中串連多個述詞&mdash;較為精簡，也是建議的選項。

`Where` 子句可支援被轉譯成 OData 子集的作業。 這些作業包括︰

* 關係運算子 (==、!=、<、<=、>、>=)、
* 算術運算子 (+、-、/、*、%)、
* 精確度位數 (Math.Floor、Math.Ceiling)、
* 字串函數 (Length、Substring、Replace、IndexOf、StartsWith、EndsWith)、
* 日期屬性 (Year、Month、Day、Hour、Minute、Second)、
* 物件的存取屬性，及
* 結合上述任何運算的運算式。

在考慮伺服器 SDK 支援的作業時，您可以考慮 [OData v3 文件]。

### <a name="sorting"></a>作法：排序傳回的資料
下列程式碼將說明如何透過在查詢中加上 [OrderBy] 或 [OrderByDescending] 函式來排序資料。 它會從 `todoTable` 傳回項目，並依據 `Text` 欄位以遞增順序排列。

```
// Sort items in ascending order by Text field
MobileServiceTableQuery<TodoItem> query = todoTable
                .OrderBy(todoItem => todoItem.Text)
List<TodoItem> items = await query.ToListAsync();

// Sort items in descending order by Text field
MobileServiceTableQuery<TodoItem> query = todoTable
                .OrderByDescending(todoItem => todoItem.Text)
List<TodoItem> items = await query.ToListAsync();
```

### <a name="paging"></a>作法：以分頁方式傳回資料
依預設，後端僅會傳回前 50 筆資料列。 您可以提高傳回的資料列數，方法是呼叫 [Take] 方法。 使用 `Take` 搭配 [Skip] 方法來要求查詢傳回之總資料集的特定「頁面」。 執行下列查詢時，會傳回資料表中的前三個項目。

```
// Define a filtered query that returns the top 3 items.
MobileServiceTableQuery<TodoItem> query = todoTable.Take(3);
List<TodoItem> items = await query.ToListAsync();
```

下列已修訂查詢會略過前三個結果，並傳回接下來的三個結果。 此查詢會產生第二「頁」資料，頁面大小為三個項目。

```
// Define a filtered query that skips the top 3 items and returns the next 3 items.
MobileServiceTableQuery<TodoItem> query = todoTable.Skip(3).Take(3);
List<TodoItem> items = await query.ToListAsync();
```

[IncludeTotalCount] 方法會要求已傳回之 *all* 記錄的總數，忽略指定的任何分頁/限制子句：

```
query = query.IncludeTotalCount();
```

在實際的應用程式中，您可以搭配頁面巡覽區控制項或類似的 UI 來使用類似上述範例的查詢，以導覽各個頁面。

> [!NOTE]
> 若要覆寫行動應用程式後端中的 50 個資料列限制，您也必須將 [EnableQueryAttribute] 套用到公用的 GET 方法並指定分頁行為。 套用到方法時，下列設定最多傳回 1000 個資料列：
>
> `[EnableQuery(MaxTop=1000)]`


### <a name="selecting"></a>作法：選取特定資料欄
您可以指定結果中要包含的屬性集，方法是在查詢中加上 [Select] 子句。 例如，下列程式碼將示範如何只選取一個欄位以及如何選取及格式化多個欄位：

```
// Select one field -- just the Text
MobileServiceTableQuery<TodoItem> query = todoTable
                .Select(todoItem => todoItem.Text);
List<string> items = await query.ToListAsync();

// Select multiple fields -- both Complete and Text info
MobileServiceTableQuery<TodoItem> query = todoTable
                .Select(todoItem => string.Format("{0} -- {1}",
                    todoItem.Text.PadRight(30), todoItem.Complete ?
                    "Now complete!" : "Incomplete!"));
List<string> items = await query.ToListAsync();
```

目前為止說明的所有函式只是不斷添加，因此只要一直鏈結即可。 每一個鏈結的呼叫都會更加影響查詢。 再提供一個範例：

```
MobileServiceTableQuery<TodoItem> query = todoTable
                .Where(todoItem => todoItem.Complete == false)
                .Select(todoItem => todoItem.Text)
                .Skip(3).
                .Take(3);
List<string> items = await query.ToListAsync();
```

### <a name="lookingup"></a>作法：按識別碼查詢資料
[LookupAsync] 函數可用來查詢具有特定 ID 的資料庫物件。

```
// This query filters out the item with the ID of 37BBF396-11F0-4B39-85C8-B319C729AF6D
TodoItem item = await todoTable.LookupAsync("37BBF396-11F0-4B39-85C8-B319C729AF6D");
```

### <a name="untypedqueries"></a>如何：執行不具類型的查詢
使用不具類型的資料表物件執行查詢時，您必須透過呼叫 [ReadAsync]來明確指定 OData 查詢字串，如下列範例所示：

```
// Lookup untyped data using OData
JToken untypedItems = await untypedTodoTable.ReadAsync("$filter=complete eq 0&$orderby=text");
```

您將收到可用作屬性包的 JSON 值。 如需有關 JToken 和 Newtonsoft Json.NET 的詳細資訊，請參閱 [Json.NET] 網站。

### <a name="inserting"></a>如何：將資料插入行動應用程式後端
所有用戶端類型都必須包含名為 **Id**的成員，其預設為字串。 需要有此 **Id** 才能執行 CRUD 作業和離線同步處理。下列程式碼將說明如何使用 [InsertAsync] 方法，將新的資料列插入資料表。 參數包含要作為 .NET 物件插入的資料。

```
await todoTable.InsertAsync(todoItem);
```

如果插入期間沒有在 `todoItem` 包含唯一的自訂識別碼值，則會由伺服器產生 GUID。
在呼叫傳回之後，您可以透過檢查物件來擷取產生的識別碼。

若要插入不具類型的資料，您可以充份利用 Json.NET：

```
JObject jo = new JObject();
jo.Add("Text", "Hello World");
jo.Add("Complete", false);
var inserted = await table.InsertAsync(jo);
```

以下是使用電子郵件地址作為唯一字串 id 的範例：

```
JObject jo = new JObject();
jo.Add("id", "myemail@emaildomain.com");
jo.Add("Text", "Hello World");
jo.Add("Complete", false);
var inserted = await table.InsertAsync(jo);
```

### <a name="working-with-id-values"></a>使用識別碼值
Mobile Apps 支援資料表的 **id** 資料行使用唯一自訂字串值。 字串值可讓應用程式使用自訂的值，例如電子郵件地址或使用者名稱作為識別碼。  字串識別碼為您提供下列優點：

* 不需要往返存取資料庫就能產生識別碼。
* 輕鬆合併不同資料表或資料庫的記錄。
* 識別碼值與應用程式邏輯的整合更理想。

當插入的記錄上未設定字串識別碼值時，行動應用程式後端會產生唯一值做為識別碼。 您可以使用 [Guid.NewGuid] 方法，在用戶端上或在後端中產生您自己的識別碼值。

```
JObject jo = new JObject();
jo.Add("id", Guid.NewGuid().ToString("N"));
```

### <a name="modifying"></a>如何：修改行動應用程式後端中的資料
下列程式碼將說明如何使用 [UpdateAsync] 方法，利用新資訊來更新具相同識別碼的現有記錄。 參數包含要作為 .NET 物件更新的資料。

```
await todoTable.UpdateAsync(todoItem);
```

若要更新不具類型的資料，您可以充份利用 [Json.NET] ，如下所示：

```
JObject jo = new JObject();
jo.Add("id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
jo.Add("Text", "Hello World");
jo.Add("Complete", false);
var inserted = await table.UpdateAsync(jo);
```

更新時，您必須指定 `id` 欄位。 後端會使用 `id` 欄位來識別要更新哪一個資料列。 您可以從 `InsertAsync` 呼叫的結果取得 `id` 欄位。 當您嘗試更新項目但未提供 `id` 值時，就會引發 `ArgumentException`。

### <a name="deleting"></a>如何：刪除行動應用程式後端中的資料
下列程式碼將說明如何使用 [DeleteAsync] 方法，刪除現有的執行個體。 您可以透過 `todoItem` 上設定的 `id` 欄位來識別執行個體。

```
await todoTable.DeleteAsync(todoItem);
```

若要刪除不具類型的資料，您可以充份利用 Json.NET，如下所示：

```
JObject jo = new JObject();
jo.Add("id", "37BBF396-11F0-4B39-85C8-B319C729AF6D");
await table.DeleteAsync(jo);
```

提出刪除要求時，必須指定 ID。 其他屬性不會傳遞至服務，或者服務會將它們忽略。 `DeleteAsync` 呼叫的結果通常會是 `null`。 您可以從 `InsertAsync` 呼叫的結果取得所要傳入的 ID。 當您嘗試刪除項目但未指定 `id` 欄位時，會擲回 `MobileServiceInvalidOperationException`。

### <a name="optimisticconcurrency"></a>做法：使用開放式並行存取來解決衝突
兩個或多個用戶端可能會同時對相同項目寫入變更。 在沒有偵測到衝突的情況下，最後寫入將覆寫任何先前的更新。  會假設每筆交易都可以認可，因此不會使用任何資源鎖定。  在認可交易之前，開放式並行存取控制項會驗證沒有其他交易已修改此資料。 如果資料已修改，則會復原認可的交易。

Mobile Apps 支援開放式並行存取控制項，方法是使用 `version` 系統屬性資料行來追蹤對每個項目的變更，該資料行是針對行動應用程式後端中的每個資料表所定義的。 每當更新記錄時，Mobile Apps 會將該筆記錄的 `version` 屬性設定為新值。 在每次更新要求期間，要求所提供的該筆記錄 `version` 屬性會與伺服器上該筆記錄的相同屬性進行比對。 如果隨著要求傳遞的版本與後端不符，則用戶端程式庫會引發 `MobileServicePreconditionFailedException<T>` 例外狀況。 例外狀況所提供的類型是來自包含該記錄之伺服器版本的後端記錄。 接著應用程式可以使用此資訊，來決定是否要針對後端的正確 `version` 值來執行更新要求以認可變更。

在 `version` 系統屬性的資料表類別上定義資料行，以啟用開放式並行存取。 例如︰

```
public class TodoItem
{
    public string Id { get; set; }

    [JsonProperty(PropertyName = "text")]
    public string Text { get; set; }

    [JsonProperty(PropertyName = "complete")]
    public bool Complete { get; set; }

    // *** Enable Optimistic Concurrency *** //
    [JsonProperty(PropertyName = "version")]
    public string Version { set; get; }
}
```

使用不具類型之資料表的應用程式會在資料表的 `SystemProperties` 上設定 `Version` 旗標，以啟用開放式並行存取，如下所示。

```
//Enable optimistic concurrency by retrieving version
todoTable.SystemProperties |= MobileServiceSystemProperties.Version;
```

除了啟用開放式並行存取之外，您還必須在呼叫 [UpdateAsync] 時攔截程式碼中的 `MobileServicePreconditionFailedException<T>` 例外狀況。  將正確的 `version` 套用到更新的記錄，並利用解決的記錄呼叫 [UpdateAsync] ，藉此解決衝突。 下列程式碼將示範如何在偵測到寫入衝突之後立即解決：

```
private async void UpdateToDoItem(TodoItem item)
{
    MobileServicePreconditionFailedException<TodoItem> exception = null;

    try
    {
        //update at the remote table
        await todoTable.UpdateAsync(item);
    }
    catch (MobileServicePreconditionFailedException<TodoItem> writeException)
    {
        exception = writeException;
    }

    if (exception != null)
    {
        // Conflict detected, the item has changed since the last query
        // Resolve the conflict between the local and server item
        await ResolveConflict(item, exception.Item);
    }
}


private async Task ResolveConflict(TodoItem localItem, TodoItem serverItem)
{
    //Ask user to choose the resolution between versions
    MessageDialog msgDialog = new MessageDialog(
        String.Format("Server Text: \"{0}\" \nLocal Text: \"{1}\"\n",
        serverItem.Text, localItem.Text),
        "CONFLICT DETECTED - Select a resolution:");

    UICommand localBtn = new UICommand("Commit Local Text");
    UICommand ServerBtn = new UICommand("Leave Server Text");
    msgDialog.Commands.Add(localBtn);
    msgDialog.Commands.Add(ServerBtn);

    localBtn.Invoked = async (IUICommand command) =>
    {
        // To resolve the conflict, update the version of the item being committed. Otherwise, you will keep
        // catching a MobileServicePreConditionFailedException.
        localItem.Version = serverItem.Version;

        // Updating recursively here just in case another change happened while the user was making a decision
        UpdateToDoItem(localItem);
    };

    ServerBtn.Invoked = async (IUICommand command) =>
    {
        RefreshTodoItems();
    };

    await msgDialog.ShowAsync();
}
```

如需詳細資訊，請參閱 [Azure Mobile Apps 中的離線資料同步處理] 主題。

### <a name="binding"></a>如何：將 Mobile Apps 資料繫結至 Windows 使用者介面
本節說明如何在 Windows app 中使用 UI 元素來顯示傳回的資料物件。  下列範例程式碼繫結至具有未完成項目之查詢的清單來源。 [MobileServiceCollection] 會建立 Mobile Apps 感知的繫結集合。

```
// This query filters out completed TodoItems.
MobileServiceCollection<TodoItem, TodoItem> items = await todoTable
    .Where(todoItem => todoItem.Complete == false)
    .ToCollectionAsync();

// itemsControl is an IEnumerable that could be bound to a UI list control
IEnumerable itemsControl  = items;

// Bind this to a ListBox
ListBox lb = new ListBox();
lb.ItemsSource = items;
```

受控執行階段中的部分控制項支援名為 [ISupportIncrementalLoading]的介面。 此介面允許控制項在使用者捲動時要求額外資料。 通用 Windows 應用程式可透過會自動處理控制項呼叫的 [MobileServiceIncrementalLoadingCollection]，內建支援此介面。 在 Windows 應用程式中使用 `MobileServiceIncrementalLoadingCollection` ，如下所示︰

```
MobileServiceIncrementalLoadingCollection<TodoItem,TodoItem> items;
items = todoTable.Where(todoItem => todoItem.Complete == false).ToIncrementalLoadingCollection();

ListBox lb = new ListBox();
lb.ItemsSource = items;
```

若要在 Windows Phone 8 和 「Silverlight」應用程式上使用新的集合，請在 `IMobileServiceTableQuery<T>` 與 `IMobileServiceTable<T>` 上使用 `ToCollection` 擴充方法。 若要載入資料，請呼叫 `LoadMoreItemsAsync()`。

```
MobileServiceCollection<TodoItem, TodoItem> items = todoTable.Where(todoItem => todoItem.Complete==false).ToCollection();
await items.LoadMoreItemsAsync();
```

當您使用透過呼叫 `ToCollectionAsync` 或 `ToCollection` 來建立的集合時，您會取得可繫結至 UI 控制項的集合。  此集合有分頁感知功能。  因為集合會從網路中載入資料，因此載入有時會失敗。 若要處理這類失敗，請覆寫 `MobileServiceIncrementalLoadingCollection` 上的 `OnException` 方法，以處理呼叫 `LoadMoreItemsAsync` 時所造成的例外狀況。

請思考一下如果您的資料表有許多欄位，但您只想要在控制項中顯示其中部分欄位。 您可以使用上述[選取特定資料欄](#selecting)一節中的指導方針，以選取要在 UI 中顯示的特定資料欄。

### <a name="pagesize"></a>變更頁面大小
Azure Mobile Apps 預設針對每個要求最多會傳回 50 個項目。  您可以增加用戶端和伺服器上的頁面大小上限，以變更分頁大小。  若要增加要求的頁面大小，請在使用 `PullAsync()`時指定 `PullOptions`：

```
PullOptions pullOptions = new PullOptions
    {
        MaxPageSize = 100
    };
```

假設您已經在伺服器中讓 `PageSize` 等於或大於 100，要求最多會傳回 100 個項目。

## <a name="#offlinesync"></a>使用離線資料表
離線資料表會使用本機 SQLite 存放區來儲存資料供離線時使用。  所有資料表作業都是針對本機 SQLite 存放區而非遠端伺服器存放區完成。  若要建立離線資料表，先準備您的專案：

1. 在 Visual Studio 中，以滑鼠右鍵按一下方案 > [管理方案的 NuGet 套件...]，然後為方案中的所有專案，尋找並安裝 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 套件。
2. (選擇性) 若要支援 Windows 裝置，請安裝下列其中一個 SQLite 執行階段套件︰

   * **Windows 8.1 執行階段：** 安裝[適用於 Windows 8.1 的 SQLite][3]。
   * **Windows Phone 8.1：** 安裝[適用於 Windows Phone 8.1 的 SQLite][4]。
   * **通用 Windows 平台：** 安裝[適用於通用 Windows 平台的 SQLite][5]。
3. (選擇性)。 針對 Windows 裝置，請按一下 [參考]  > [加入參考]，展開 **Windows** 資料夾 > [擴充功能]，然後啟用適當的 **SQLite for Windows** SDK 及 **Visual C++ 2013 Runtime for Windows** SDK。
    每個 Windows 平台的 SQLite SDK 名稱稍有差異。

建立資料表參考之前，必須準備本機存放區：

```
var store = new MobileServiceSQLiteStore(Constants.OfflineDbPath);
store.DefineTable<TodoItem>();

//Initializes the SyncContext using the default IMobileServiceSyncHandler.
await this.client.SyncContext.InitializeAsync(store);
```

存放區初始化通常是在建立用戶端之後隨即完成。  **OfflineDbPath** 應該是適合在您支援的所有平台上使用的檔案名稱。  如果路徑是完整路徑 (也就是開頭為斜線)，則會使用該路徑。  如果路徑不是完整路徑，則檔案會放在平台特定的位置。

* 針對 iOS 和 Android 裝置，預設路徑是「個人檔案」資料夾。
* 針對 Windows 裝置，預設路徑是應用程式特定的 "AppData" 資料夾。

您可以使用 `GetSyncTable<>` 方法來取得資料表參考：

```
var table = client.GetSyncTable<TodoItem>();
```

使用離線資料表不需要取得驗證。  只需要在與後端服務通訊時進行驗證。

### <a name="syncoffline"></a>同步處理離線資料表
預設情況下，不會對後端同步處理離線資料表。  同步處理會分割成兩個部分。  您可以透過下載新的項目，個別推送變更。  以下是典型的同步處理方法：

```
public async Task SyncAsync()
{
    ReadOnlyCollection<MobileServiceTableOperationError> syncErrors = null;

    try
    {
        await this.client.SyncContext.PushAsync();

        await this.todoTable.PullAsync(
            //The first parameter is a query name that is used internally by the client SDK to implement incremental sync.
            //Use a different query name for each unique query in your program
            "allTodoItems",
            this.todoTable.CreateQuery());
    }
    catch (MobileServicePushFailedException exc)
    {
        if (exc.PushResult != null)
        {
            syncErrors = exc.PushResult.Errors;
        }
    }

    // Simple error/conflict handling. A real application would handle the various errors like network conditions,
    // server conflicts and others via the IMobileServiceSyncHandler.
    if (syncErrors != null)
    {
        foreach (var error in syncErrors)
        {
            if (error.OperationKind == MobileServiceTableOperationKind.Update && error.Result != null)
            {
                //Update failed, reverting to server's copy.
                await error.CancelAndUpdateItemAsync(error.Result);
            }
            else
            {
                // Discard local change.
                await error.CancelAndDiscardItemAsync();
            }

            Debug.WriteLine(@"Error executing sync operation. Item: {0} ({1}). Operation discarded.", error.TableName, error.Item["id"]);
        }
    }
}
```

如果 `PullAsync` 的第一個引數為 null，則不會使用增量同步處理。  每個同步處理作業會擷取所有記錄。

SDK 會在提取記錄之前執行隱含 `PushAsync()`。

衝突處理會發生在 `PullAsync()` 方法。  您可以使用與線上資料表相同的方式處理衝突。  衝突是在呼叫 `PullAsync()` 時而不是在插入、更新或刪除期間產生。 如果發生多個衝突，則會將它們會搭配至單一 MobileServicePushFailedException。  個別處理每一個失敗。

## <a name="#customapi"></a>使用自訂 API
自訂 API 可讓您定義自訂端點，並用來公開無法對應插入、更新、刪除或讀取等操作的伺服器功能。 透過使用自訂 API，您可以進一步控制訊息，包括讀取與設定 HTTP 訊息標頭，並定義除了 JSON 以外的訊息內文格式。

若要呼叫自訂 API，您可以呼叫用戶端上的其中一個 [InvokeApiAsync] 方法。 例如，下列程式碼行會將 POST 要求傳送至後端的 **completeAll** API：

```
var result = await client.InvokeApiAsync<MarkAllResult>("completeAll", System.Net.Http.HttpMethod.Post, null);
```

此形式是具類型的方法呼叫，並且會要求定義 **MarkAllResult** 傳回類型。 具類型的和不具類型的方法皆受支援。

InvokeApiAsync() 方法會在您想要呼叫的 API 前面加上 '/api/'，除非該 API 開頭是 '/'。
例如︰

* `InvokeApiAsync("completeAll",...)` 會呼叫後端上的 /api/completeAll
* `InvokeApiAsync("/.auth/me",...)` 會呼叫後端上的 /.auth/me

您可以使用 InvokeApiAsync 來呼叫任何 WebAPI，包括未隨著 Azure Mobile Apps 一起定義的 WebAPI。  當您使用 InvokeApiAsync() 時，會隨著要求一起傳送適當的標頭，包括驗證標頭。

## <a name="authentication"></a>驗證使用者
Mobile Apps 支援使用各種外部識別提供者 (Facebook、Google、Microsoft 帳戶、Twitter 以及 Azure Active Directory)，來驗證與授權應用程式使用者。 您可以在資料表上設定權限，以限制僅有通過驗證使用者可以存取特定操作。 您也可以使用通過驗證使用者的身分識別來實作伺服器指令碼中的授權規則。 如需詳細資訊，請參閱 [將驗證新增至您的應用程式]教學課程。

支援兩種驗證流程：*用戶端管理*和*伺服器管理*流程。 由於伺服器管理流程採用提供者的 Web 驗證介面，因此所提供的驗證體驗也最為簡單。 因為用戶端管理流程採用提供者特定的裝置特定 SDK，因此可允許與裝置特定功能的深入整合。

> [!NOTE]
> 我們建議在生產應用程式中使用用戶端管理的流程。

若要設定驗證，您必須向一或多個識別提供者註冊您的應用程式。  識別提供者會產生應用程式的用戶端識別碼和用戶端密碼。  這些值接著會設定於後端中以啟用 Azure App Service 驗證/授權。  如需詳細資訊，請依照教學課程 [將驗證新增至您的應用程式]中的詳細指示操作。

這一節涵蓋下列主題：

* [用戶端管理的驗證](#clientflow)
* [伺服器管理的驗證](#serverflow)
* [快取驗證權杖](#caching)

### <a name="clientflow"></a>用戶端管理的驗證
您的應用程式可以個別連絡識別提供者，然後在用您的後端登入期間提供傳回的權杖。 此用戶端流程可讓您為使用者提供單一登入體驗，或從識別提供者擷取其他使用者資料。 用戶端流程驗證比較適合使用伺服器流程，因為識別提供者 SDK 提供更原生的 UX 風格，並可允許進行其他自訂。

已提供下列用戶端流程驗證模式的範例︰

* [Active Directory Authentication Library](#adal)
* [Facebook 或 Google](#client-facebook)
* [Live SDK](#client-livesdk)

#### <a name="adal"></a>使用 Active Directory Authentication Library 驗證使用者
您可以使用 Active Directory Authentication Library (ADAL)，從使用 Azure Active Directory 驗證的用戶端起始使用者驗證。

1. 依照[如何針對 Active Directory 登入設定 App Service]教學課程的說明，設定您的行動應用程式後端來進行 AAD 登入。 請務必完成註冊原生用戶端應用程式的選擇性步驟。
2. 在 Visual Studio 或 Xamarin Studio 中，開啟您的專案，然後新增對 `Microsoft.IdentityModel.CLients.ActiveDirectory` NuGet 套件的參考。 搜尋時，包含發行前版本。
3. 根據您使用的平台，將下列程式碼新增至您的應用程式。 在每個程式碼中，進行下列取代：

   * 以您佈建應用程式的租用戶名稱取代 **INSERT-AUTHORITY-HERE** 。 格式應為 https://login.microsoftonline.com/contoso.onmicrosoft.com。 您可以從 [Azure 入口網站]之 Azure Active Directory 的 [網域] 索引標籤中複製這個值。
   * 以您行動應用程式後端的用戶端識別碼取代 INSERT-RESOURCE-ID-HERE  。 您可以從入口網站 [Azure Active Directory 設定] 底下的 [進階] 索引標籤取得用戶端識別碼。
   * 以您從原生用戶端應用程式中複製的用戶端識別碼取代 INSERT-CLIENT-ID-HERE  。
   * 使用 HTTPS 配置，以您網站的 **/.auth/login/done** 端點取代 *INSERT-REDIRECT-URI-HERE* 。 此值應該類似 https://contoso.azurewebsites.net/.auth/login/done。

     每個平台所需的程式碼如下：

     **Windows：**

    ```
    private MobileServiceUser user;
    private async Task AuthenticateAsync()
    {

        string authority = "INSERT-AUTHORITY-HERE";
        string resourceId = "INSERT-RESOURCE-ID-HERE";
        string clientId = "INSERT-CLIENT-ID-HERE";
        string redirectUri = "INSERT-REDIRECT-URI-HERE";
        while (user == null)
        {
            string message;
            try
            {
                AuthenticationContext ac = new AuthenticationContext(authority);
                AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId,
                    new Uri(redirectUri), new PlatformParameters(PromptBehavior.Auto, false) );
                JObject payload = new JObject();
                payload["access_token"] = ar.AccessToken;
                user = await App.MobileService.LoginAsync(
                    MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
                message = string.Format("You are now logged in - {0}", user.UserId);
            }
            catch (InvalidOperationException)
            {
                message = "You must log in. Login Required";
            }
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```

     **Xamarin.iOS**

    ```
    private MobileServiceUser user;
    private async Task AuthenticateAsync(UIViewController view)
    {

        string authority = "INSERT-AUTHORITY-HERE";
        string resourceId = "INSERT-RESOURCE-ID-HERE";
        string clientId = "INSERT-CLIENT-ID-HERE";
        string redirectUri = "INSERT-REDIRECT-URI-HERE";
        try
        {
            AuthenticationContext ac = new AuthenticationContext(authority);
            AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId,
                new Uri(redirectUri), new PlatformParameters(view));
            JObject payload = new JObject();
            payload["access_token"] = ar.AccessToken;
            user = await client.LoginAsync(
                MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
        }
        catch (Exception ex)
        {
            Console.Error.WriteLine(@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
        }
    }
    ```

     **Xamarin.Android**

    ```
    private MobileServiceUser user;
    private async Task AuthenticateAsync()
    {

        string authority = "INSERT-AUTHORITY-HERE";
        string resourceId = "INSERT-RESOURCE-ID-HERE";
        string clientId = "INSERT-CLIENT-ID-HERE";
        string redirectUri = "INSERT-REDIRECT-URI-HERE";
        try
        {
            AuthenticationContext ac = new AuthenticationContext(authority);
            AuthenticationResult ar = await ac.AcquireTokenAsync(resourceId, clientId,
                new Uri(redirectUri), new PlatformParameters(this));
            JObject payload = new JObject();
            payload["access_token"] = ar.AccessToken;
            user = await client.LoginAsync(
                MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory, payload);
        }
        catch (Exception ex)
        {
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.SetMessage(ex.Message);
            builder.SetTitle("You must log in. Login Required");
            builder.Create().Show();
        }
    }
    protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
    {

        base.OnActivityResult(requestCode, resultCode, data);
        AuthenticationAgentContinuationHelper.SetAuthenticationAgentContinuationEventArgs(requestCode, resultCode, data);
    }
    ```

#### <a name="client-facebook"></a>使用來自 Facebook 或 Google 的權杖單一登入
您可以使用用戶端流程，如以下 Facebook 或 Google 的程式碼片段中所示。

```
var token = new JObject();
// Replace access_token_value with actual value of your access token obtained
// using the Facebook or Google SDK.
token.Add("access_token", "access_token_value");

private MobileServiceUser user;
private async Task AuthenticateAsync()
{
    while (user == null)
    {
        string message;
        try
        {
            // Change MobileServiceAuthenticationProvider.Facebook
            // to MobileServiceAuthenticationProvider.Google if using Google auth.
            user = await client.LoginAsync(MobileServiceAuthenticationProvider.Facebook, token);
            message = string.Format("You are now logged in - {0}", user.UserId);
        }
        catch (InvalidOperationException)
        {
            message = "You must log in. Login Required";
        }

        var dialog = new MessageDialog(message);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
}
```

#### <a name="client-livesdk"></a>使用 Microsoft 帳戶搭配 Live SDK 進行單一登入
若要驗證使用者，您必須在 Microsoft 帳戶開發人員中心註冊您的應用程式。 請在行動應用程式後端設定註冊詳細資料。 若要建立 Microsoft 帳戶註冊，並將其連接到您的行動應用程式後端，請完成 [註冊您的應用程式以使用 Microsoft 帳戶登入]中的步驟。 如果您的應用程式同時有 Microsoft Store 與 Windows Phone 8/Silverlight 版本，請先註冊 Microsoft Store 版本。

以下程式碼會使用 Live SDK 進行驗證，並使用傳回的權杖來登入您的「行動應用程式」後端。

```
private LiveConnectSession session;
    //private static string clientId = "<microsoft-account-client-id>";
private async System.Threading.Tasks.Task AuthenticateAsync()
{

    // Get the URL the Mobile App backend.
    var serviceUrl = App.MobileService.ApplicationUri.AbsoluteUri;

    // Create the authentication client for Microsoft Store using the service URL.
    LiveAuthClient liveIdClient = new LiveAuthClient(serviceUrl);
    //// Create the authentication client for Windows Phone using the client ID of the registration.
    //LiveAuthClient liveIdClient = new LiveAuthClient(clientId);

    while (session == null)
    {
        // Request the authentication token from the Live authentication service.
        // The wl.basic scope should always be requested.  Other scopes can be added
        LiveLoginResult result = await liveIdClient.LoginAsync(new string[] { "wl.basic" });
        if (result.Status == LiveConnectSessionStatus.Connected)
        {
            session = result.Session;

            // Get information about the logged-in user.
            LiveConnectClient client = new LiveConnectClient(session);
            LiveOperationResult meResult = await client.GetAsync("me");

            // Use the Microsoft account auth token to sign in to App Service.
            MobileServiceUser loginResult = await App.MobileService
                .LoginWithMicrosoftAccountAsync(result.Session.AuthenticationToken);

            // Display a personalized sign-in greeting.
            string title = string.Format("Welcome {0}!", meResult.Result["first_name"]);
            var message = string.Format("You are now logged in - {0}", loginResult.UserId);
            var dialog = new MessageDialog(message, title);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
        else
        {
            session = null;
            var dialog = new MessageDialog("You must log in.", "Login Required");
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
}
```

如需詳細資訊，請參閱 [Windows Live SDK] 文件。

### <a name="serverflow"></a>伺服器管理的驗證
註冊識別提供者之後，使用提供者的 [LoginAsync] 值，在 [MobileServiceClient] 上呼叫 [LoginAsync] 方法。 例如，下列程式碼將透過使用 Facebook 來初始化伺服器流程登入。

```
private MobileServiceUser user;
private async System.Threading.Tasks.Task Authenticate()
{
    while (user == null)
    {
        string message;
        try
        {
            user = await client
                .LoginAsync(MobileServiceAuthenticationProvider.Facebook);
            message =
                string.Format("You are now logged in - {0}", user.UserId);
        }
        catch (InvalidOperationException)
        {
            message = "You must log in. Login Required";
        }

        var dialog = new MessageDialog(message);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
}
```

如果您打算使用除了 Facebook 以外的識別提供者，請將上方的 [LoginAsync] 值變更成您提供者。

在伺服器流程中，Azure App Service 透過顯示所選提供者的登入頁面，來管理 OAuth 驗證流程。  在識別提供者傳回後，Azure App Service 會產生 App Service 驗證權杖。 [LoginAsync] 方法 會傳回 [MobileServiceUser]，並提供通過驗證使用者的 [UserId] 和 [MobileServiceAuthenticationToken]，以作為 JSON Web 權杖 (JWT)。 您可以快取並重複使用此權杖，直到它到期為止。 如需詳細資訊，請參閱 [快取驗證權杖](#caching)。

### <a name="caching"></a>快取驗證權杖
在某些情況下，儲存來自提供者的驗證權杖即可避免在第一次成功驗證後呼叫登入方法。  Microsoft Store 和 UWP app 可以使用 [PasswordVault] ，在成功登入後快取目前的驗證權杖，如下所示：

```
await client.LoginAsync(MobileServiceAuthenticationProvider.Facebook);

PasswordVault vault = new PasswordVault();
vault.Add(new PasswordCredential("Facebook", client.currentUser.UserId,
    client.currentUser.MobileServiceAuthenticationToken));
```

UserId 值會儲存為認證的 UserName，而權杖會儲存為 Password。 在後續的啟動時，您可以檢查 **PasswordVault** 中的快取認證。 下列範例會使用找到的快取認證，否則會嘗試再次向後端進行驗證︰

```
// Try to retrieve stored credentials.
var creds = vault.FindAllByResource("Facebook").FirstOrDefault();
if (creds != null)
{
    // Create the current user from the stored credentials.
    client.currentUser = new MobileServiceUser(creds.UserName);
    client.currentUser.MobileServiceAuthenticationToken =
        vault.Retrieve("Facebook", creds.UserName).Password;
}
else
{
    // Regular login flow and cache the token as shown above.
}
```

當您登出使用者時，您也必須移除預存的認證，如下所示︰

```
client.Logout();
vault.Remove(vault.Retrieve("Facebook", client.currentUser.UserId));
```

Xamarin 應用程式會使用 [Xamarin.Auth] API，將憑證安全地儲存在 [帳戶] 物件中。 如需使用這些 API 的範例，請參閱 [ContosoMoments 照片分享範例](https://github.com/azure-appservice-samples/ContosoMoments)中的 [AuthStore.cs] 程式碼檔案。

當您使用用戶端管理的驗證時，您也可以快取向您的提供者 (例如 Facebook 或 Twitter) 取得的存取權杖。 您可以提供此權杖，以向後端要求新的驗證權杖，如下所示︰

```
var token = new JObject();
// Replace <your_access_token_value> with actual value of your access token
token.Add("access_token", "<your_access_token_value>");

// Authenticate using the access token.
await client.LoginAsync(MobileServiceAuthenticationProvider.Facebook, token);
```

## <a name="pushnotifications"></a>推播通知
下列主題涵蓋推播通知︰

* [註冊推播通知](#register-for-push)
* [取得 Microsoft Store 套件 SID](#package-sid)
* [利用跨平台範本進行註冊](#register-xplat)

### <a name="register-for-push"></a>做法：註冊推播通知
Mobile Apps 用戶端可讓您向 Azure 通知中樞註冊推播通知。 註冊時，您會取得從特定平台「推送通知服務 (PNS)」取得的控制代碼。 然後您就可以在建立註冊時提供此值以及任何標記。 下列程式碼會為您的 Windows 應用程式向 Windows 通知服務 (WNS) 註冊推播通知：

```
private async void InitNotificationsAsync()
{
    // Request a push notification channel.
    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    // Register for notifications using the new channel.
    await MobileService.GetPush().RegisterNativeAsync(channel.Uri, null);
}
```

如果您要推送到 WNS，則必須[取得 Microsoft Store 套件 SID](#package-sid)。  如需 Windows 應用程式的詳細資訊，包括如何註冊範本，請參閱 [將推播通知新增至您的應用程式]。

不支援從用戶端要求標記。  註冊時會自動捨棄標記要求。
如果您想要利用標記註冊裝置，請建立自訂 API，以使用通知中樞 API 代替您執行註冊。  [呼叫自訂 API](#customapi) 而不是 `RegisterNativeAsync()` 方法。

### <a name="package-sid"></a>如何：取得 Microsoft Store 套件 SID
在 Microsoft Store 應用程式中啟用推播通知需有套件 SID。  若要收到套件 SID，請向 Microsoft Store 註冊應用程式。

若要取得這個值：

1. 在 Visual Studio 方案總管中，以滑鼠右鍵按一下 Microsoft Store 應用程式專案，然後按一下 [市集] > [將應用程式與市集建立關聯...]。
2. 在精靈中按 [下一步]，使用 Microsoft 帳戶登入，在 [保留新的應用程式名稱] 中輸入您應用程式的名稱，然後按一下 [保留]。
3. 成功建立應用程式註冊之後，選取應用程式名稱，按 [下一步]，然後按一下 [關聯]。
4. 使用您的 Microsoft 帳戶登入 [Windows 開發人員中心] 。 在 [我的應用程式] 底下，按一下您建立的應用程式註冊。
5. 按一下 [應用程式管理]  >  [應用程式身分識別]，然後向下捲動找到您的 [套件 SID]。

許多使用套件 SID 的情況會將其視為 URI，在這種情況下，您必須使用 *ms-app://* 作為配置。 記下您套件 SID 的版本，套件 SID 是由串連這個值作為首碼所形成。

Xamarin 應用程式需要一些額外的程式碼，才能註冊執行於 iOS 或 Android 平台的應用程式。 如需詳細資訊，請參閱平台適用的主題：

* [Xamarin.Android](app-service-mobile-xamarin-android-get-started-push.md#add-push)
* [Xamarin.iOS](app-service-mobile-xamarin-ios-get-started-push.md#add-push-notifications-to-your-app)

### <a name="register-xplat"></a>作法：註冊推送範本以傳送跨平台通知
若要註冊範本，請使用 `RegisterAsync()` 方法搭配範本，如下所示：

```
JObject templates = myTemplates();
MobileService.GetPush().RegisterAsync(channel.Uri, templates);
```

您的範本應該是 `JObject` 類型，並且可能包含下列 JSON 格式的多個範本：

```
public JObject myTemplates()
{
    // single template for Windows Notification Service toast
    var template = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(message)</text></binding></visual></toast>";

    var templates = new JObject
    {
        ["generic-message"] = new JObject
        {
            ["body"] = template,
            ["headers"] = new JObject
            {
                ["X-WNS-Type"] = "wns/toast"
            },
            ["tags"] = new JArray()
        },
        ["more-templates"] = new JObject {...}
    };
    return templates;
}
```

方法 **RegisterAsync()** 也接受次要磚：

```
MobileService.GetPush().RegisterAsync(string channelUri, JObject templates, JObject secondaryTiles);
```

為確保安全，註冊期間會移除所有標籤。 若要在安裝中將標籤新增至安裝或範本，請參閱 [使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK]。

若要利用這些已註冊的範本傳送通知，請參閱 [通知中樞 API]。

## <a name="misc"></a>其他主題
### <a name="errors"></a>作法：處理錯誤
當後端發生錯誤時，用戶端 SDK 會引發 `MobileServiceInvalidOperationException`。  下列範例示範如何處理後端所傳回的例外狀況：

```
private async void InsertTodoItem(TodoItem todoItem)
{
    // This code inserts a new TodoItem into the database. When the operation completes
    // and App Service has assigned an Id, the item is added to the CollectionView
    try
    {
        await todoTable.InsertAsync(todoItem);
        items.Add(todoItem);
    }
    catch (MobileServiceInvalidOperationException e)
    {
        // Handle error
    }
}
```

如需另一個處理錯誤狀況的範例，請造訪 [Mobile Apps 檔案範例]。 [LoggingHandler] 範例會提供記錄委派處理常式，以記錄向後端提出的要求。

### <a name="headers"></a>作法：自訂要求標頭
若要支援您的特定應用程式案例，您可能需要自訂與行動應用程式後端的通訊。 例如，您可能會想要在每封連出要求上新增自訂標頭，或甚至變更回應狀態碼。 您可以使用自訂的 [DelegatingHandler]，如下列範例所示：

```
public async Task CallClientWithHandler()
{
    MobileServiceClient client = new MobileServiceClient("AppUrl", new MyHandler());
    IMobileServiceTable<TodoItem> todoTable = client.GetTable<TodoItem>();
    var newItem = new TodoItem { Text = "Hello world", Complete = false };
    await todoTable.InsertAsync(newItem);
}

public class MyHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage>
        SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        // Change the request-side here based on the HttpRequestMessage
        request.Headers.Add("x-my-header", "my value");

        // Do the request
        var response = await base.SendAsync(request, cancellationToken);

        // Change the response-side here based on the HttpResponseMessage

        // Return the modified response
        return response;
    }
}
```


<!-- Anchors. -->


<!-- Images. -->

<!-- URLs. -->
[1]: app-service-mobile-windows-store-dotnet-get-started.md
[2]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[3]: app-service-mobile-node-backend-how-to-use-server-sdk.md
[4]: https://msdn.microsoft.com/library/azure/mt419521(v=azure.10).aspx
[5]: https://github.com/Azure-Samples
[6]: http://www.newtonsoft.com/json/help/html/Properties_T_Newtonsoft_Json_JsonPropertyAttribute.htm
[7]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#define-table-controller
[8]: app-service-mobile-node-backend-how-to-use-server-sdk.md#TableOperations
[9]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/
[10]: http://www.symbolsource.org/
[11]: http://www.symbolsource.org/Public/Wiki/Using
[12]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient(v=azure.10).aspx

[將驗證新增至您的應用程式]: app-service-mobile-windows-store-dotnet-get-started-users.md
[Azure Mobile Apps 中的離線資料同步處理]: app-service-mobile-offline-data-sync.md
[將推播通知新增至您的應用程式]: app-service-mobile-windows-store-dotnet-get-started-push.md
[註冊您的應用程式以使用 Microsoft 帳戶登入]: ../app-service/app-service-mobile-how-to-configure-microsoft-authentication.md
[如何針對 Active Directory 登入設定 App Service]: ../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md

<!-- Microsoft URLs. -->
[MobileServiceCollection]: https://msdn.microsoft.com/library/azure/dn250636(v=azure.10).aspx
[MobileServiceIncrementalLoadingCollection]: https://msdn.microsoft.com/library/azure/dn268408(v=azure.10).aspx
[LoginAsync]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceauthenticationprovider(v=azure.10).aspx
[MobileServiceUser]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser(v=azure.10).aspx
[MobileServiceAuthenticationToken]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.mobileserviceauthenticationtoken(v=azure.10).aspx
[GetTable]: https://msdn.microsoft.com/library/azure/jj554275(v=azure.10).aspx
[建立不具類型的資料表參考]: https://msdn.microsoft.com/library/azure/jj554278(v=azure.10).aspx
[DeleteAsync]: https://msdn.microsoft.com/library/azure/dn296407(v=azure.10).aspx
[IncludeTotalCount]: https://msdn.microsoft.com/library/azure/dn250560(v=azure.10).aspx
[InsertAsync]: https://msdn.microsoft.com/library/azure/dn296400(v=azure.10).aspx
[InvokeApiAsync]: https://msdn.microsoft.com/library/azure/dn268343(v=azure.10).aspx
[LoginAsync]: https://msdn.microsoft.com/library/azure/dn296411(v=azure.10).aspx
[LookupAsync]: https://msdn.microsoft.com/library/azure/jj871654(v=azure.10).aspx
[OrderBy]: https://msdn.microsoft.com/library/azure/dn250572(v=azure.10).aspx
[OrderByDescending]: https://msdn.microsoft.com/library/azure/dn250568(v=azure.10).aspx
[ReadAsync]: https://msdn.microsoft.com/library/azure/mt691741(v=azure.10).aspx
[Take]: https://msdn.microsoft.com/library/azure/dn250574(v=azure.10).aspx
[Select]: https://msdn.microsoft.com/library/azure/dn250569(v=azure.10).aspx
[Skip]: https://msdn.microsoft.com/library/azure/dn250573(v=azure.10).aspx
[UpdateAsync]: https://msdn.microsoft.com/library/azure/dn250536.(v=azure.10)aspx
[UserID]: http://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.mobileservices.mobileserviceuser.userid(v=azure.10).aspx
[Where]: https://msdn.microsoft.com/library/azure/dn250579(v=azure.10).aspx
[Azure 入口網站]: https://portal.azure.com/
[EnableQueryAttribute]: https://msdn.microsoft.com/library/system.web.http.odata.enablequeryattribute.aspx
[Guid.NewGuid]: https://msdn.microsoft.com/library/system.guid.newguid(v=vs.110).aspx
[ISupportIncrementalLoading]: http://msdn.microsoft.com/library/windows/apps/Hh701916.aspx
[Windows 開發人員中心]: https://dev.windows.com/overview
[DelegatingHandler]: https://msdn.microsoft.com/library/system.net.http.delegatinghandler(v=vs.110).aspx
[Windows Live SDK]: https://msdn.microsoft.com/library/bb404787.aspx
[PasswordVault]: http://msdn.microsoft.com/library/windows/apps/windows.security.credentials.passwordvault.aspx
[ProtectedData]: http://msdn.microsoft.com/library/system.security.cryptography.protecteddata%28VS.95%29.aspx
[通知中樞 API]: https://msdn.microsoft.com/library/azure/dn495101.aspx
[Mobile Apps 檔案範例]: https://github.com/Azure-Samples/app-service-mobile-dotnet-todo-list-files
[LoggingHandler]: https://github.com/Azure-Samples/app-service-mobile-dotnet-todo-list-files/blob/master/src/client/MobileAppsFilesSample/Helpers/LoggingHandler.cs#L63

<!-- External URLs -->
[OData v3 文件]: http://www.odata.org/documentation/odata-version-3-0/
[Fiddler]: http://www.telerik.com/fiddler
[Json.NET]: http://www.newtonsoft.com/json
[Xamarin.Auth]: https://components.xamarin.com/view/xamarin.auth/
[AuthStore.cs]: https://github.com/azure-appservice-samples/ContosoMoments
[ContosoMoments photo sharing sample]: https://github.com/azure-appservice-samples/ContosoMoments
