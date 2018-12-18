---
title: 教學課程 - 透過 Azure 入口網站使用發佈/訂閱通道和主題篩選來更新零售庫存商品 | Microsoft Docs
description: 在本教學課程中，您將了解如何從主題和訂用帳戶傳送及接收訊息，以及如何使用 .NET 新增和使用篩選規則
services: service-bus-messaging
author: spelluru
manager: timlt
ms.author: spelluru
ms.date: 09/22/2018
ms.topic: tutorial
ms.service: service-bus-messaging
ms.custom: mvc
ms.openlocfilehash: 357bdcbbee348d8cb89e2d75060a3e7ba05e2c86
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406229"
---
# <a name="tutorial-update-inventory-using-azure-portal-and-topicssubscriptions"></a>教學課程：使用 Azure 入口網站和主題/訂用帳戶來更新庫存

Microsoft Azure 服務匯流排是一項多租用戶雲端傳訊服務，可在應用程式和服務之間傳送資訊。 非同步作業可讓您進行靈活的代理傳訊，搭配結構化的先進先出 (FIFO) 傳訊及發佈/訂閱功能。 本教學課程說明如何透過使用 Azure 入口網站和 .NET 的發佈/訂閱通道，在零售庫存案例中使用服務匯流排主題和訂用帳戶。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 Azure 入口網站建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 .NET 程式碼新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

此案例的範例是多個零售商店的庫存商品更新。 在此案例中，每個 (或每組) 商店都會收到預定要傳送給他們的訊息，用以更新其商品。 本教學課程將說明如何使用訂用帳戶和篩選實作此案例。 首先，您會以 3 個訂用帳戶建立主題、新增的某些規則和篩選條件，然後從主題和訂用帳戶來傳送和接收訊息。

![主題](./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png)

如果您沒有 Azure 訂用帳戶，您可以在開始前建立[免費帳戶][]。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您已安裝︰

- [Visual Studio 2017 Update 3 (版本 15.3, 26730.01)](http://www.visualstudio.com/vs) 或更新版本。
- [.NET Core SDK](https://www.microsoft.com/net/download/windows)，2.0 版或更新版本。

## <a name="service-bus-topics-and-subscriptions"></a>服務匯流排主題和訂用帳戶

[主題的每個訂用帳戶](service-bus-messaging-overview.md#topics)都會接收到每則訊息的複本。 主題在通訊協定和語意上都與服務匯流排佇列完全相容。 服務匯流排主題支援具有篩選條件、並且可透過選用動作來設定或修改訊息屬性的多種選取規則。 每當有規則符合時，就會產生一則訊息。 若要深入了解規則、篩選條件和動作，請進入此[連結](topic-filters.md)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

首先，移至 [Azure 入口網站][Azure portal]，並使用您的 Azure 訂用帳戶登入。 第一個步驟是建立**傳訊**類型的服務匯流排命名空間。

## <a name="create-a-service-bus-namespace"></a>建立服務匯流排命名空間

服務匯流排傳訊命名空間提供唯一的範圍容器 (依其[完整網域名稱][]來參考)，您可以在其中建立一或多個佇列、主題和訂用帳戶。 下列範例會在新的或現有的[資源群組](/azure/azure-resource-manager/resource-group-portal)中建立服務匯流排傳訊命名空間：

1. 在入口網站的左方瀏覽窗格中，依序按一下 [+ 建立資源]、[企業整合] 及 [服務匯流排]。
2. 在 [建立命名空間]  對話方塊中，輸入命名空間名稱。 系統會立即檢查此名稱是否可用。
3. 確定命名空間名稱可用之後，請選擇定價層 ([標準] 或 [進階])。
4. 在 [訂用帳戶]  欄位中，選擇要在其中建立命名空間的 Azure 訂用帳戶。
5. 在 [資源群組]  欄位中，選擇將存留命名空間的現有資源群組，或是建立新的資源群組。      
6. 在 [位置] 中，選擇應裝載您命名空間的國家或地區。
7. 按一下頁面底部的 [新增] 。 此時系統會建立並啟用命名空間。 系統為帳戶提供資源時，您可能需要等幾分鐘。

  ![namespace](./media/service-bus-tutorial-topics-subscriptions-portal/create-namespace.png)

### <a name="obtain-the-management-credentials"></a>取得管理認證

建立新命名空間會自動產生初始共用存取簽章 (SAS) 規則，其利用相關聯的主要和次要金鑰組，分別授與命名空間的所有層面的完全控制權。 若要複製初始規則，請遵循下列步驟：

1. 按一下 [所有資源]，然後按一下新建立的命名空間名稱。
2. 在命名空間視窗中，按一下 [共用存取原則]。
3. 在 [共用存取原則] 畫面中，按一下 **RootManageSharedAccessKey**。
4. 在 [原則: RootManageSharedAccessKey] 視窗中，按一下 [主要連接字串] 旁邊的 [複製] 按鈕，將連接字串複製到剪貼簿以供後續使用。 將此值貼到記事本或一些其他暫存位置。

    ![connection-string][connection-string]
5. 重複前一個步驟，複製 [主要金鑰] 的值並貼到暫存位置以供後續使用。

## <a name="create-a-topic-and-subscriptions"></a>建立主題和訂用帳戶

若要建立服務匯流排主題，請指定您要用來建立主題的命名空間。 下列範例說明如何在入口網站上建立主題：

1. 在入口網站的左方瀏覽窗格中，按一下 [服務匯流排] \(如果您未看見 [服務匯流排]，請按一下 [所有服務])。
2. 按一下要在其中建立主題的命名空間。
3. 在命名空間視窗中按一下 [主題]，然後在 [主題] 視窗中按一下 [+ 主題]。
4. 輸入主題 [名稱]，並且讓其他值保留其預設值。
5. 在視窗底部按一下 [建立]。
6. 記下主題名稱。
7. 選取您剛剛建立的主題。
8. 按一下 [+ 訂用帳戶]，輸入訂用帳戶名稱 **S1**，並將所有其他值保留為預設值。
9. 再重複兩次前述步驟，建立名為 **S2** 和 **S3** 的訂用帳戶。

## <a name="create-filter-rules-on-subscriptions"></a>在訂用帳戶上建立篩選規則

佈建命名空間和主題/訂用帳戶，且您已擁有必要的認證之後，您即可在訂用帳戶上建立篩選規則，然後傳送和接收訊息。 您可以在[此 GitHub 範例資料夾](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/GettingStarted\BasicSendReceiveTutorialwithFilters)中查看程式碼。

### <a name="send-and-receive-messages"></a>傳送及接收訊息

若要執行程式碼，請執行下列動作：

1. 在命令提示字元或 PowerShell 提示字元中發出下列命令，以複製[服務匯流排 GitHub 存放庫](https://github.com/Azure/azure-service-bus/)︰

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. 瀏覽至範例資料夾 `azure-service-bus\samples\DotNet\GettingStarted\BasicSendReceiveTutorialwithFilters`。

3. 取得您先前在本教學課程的[取得管理認證](#obtain-the-management-credentials)一節中複製到 [記事本] 的連接字串。 您也需要在上一節中建立的主題名稱。

4. 在命令提示字元中，輸入下列命令：

   ```shell
   dotnet build
   ```

5. 瀏覽到 `BasicSendReceiveTutorialwithFilters\bin\Debug\netcoreapp2.0` 資料夾。

6. 輸入下列命令以執行程式。 請務必將 `myConnectionString` 取代為您先前取得的值，並將 `myTopicName` 取代為您已建立的主題名稱：

   ```shell
   dotnet BasicSendReceiveTutorialwithFilters.dll -ConnectionString "myConnectionString" -TopicName "myTopicName"
   ``` 
7. 首先，依照主控台中的指示選取篩選建立。 移除預設篩選是建立篩選的一部分。 使用 PowerShell 或 CLI 時，您不需要移除預設篩選，但如果您在程式碼中執行此作業，則必須將其移除。 主控台命令 1 和 3 可協助您對先前建立的訂用帳戶進行篩選管理：

   - 執行 1：移除預設篩選。
   - 執行 2：新增您自己的篩選。
   - 執行 3：選擇性地移除您自己的篩選。 請注意，這並不會重新建立預設篩選。

    ![顯示 2 的輸出](./media/service-bus-tutorial-topics-subscriptions-portal/create-rules.png)

8. 篩選建立後，您可以傳送訊息。 按 4 並觀察 10 個傳送至主題的訊息：

    ![傳送輸出](./media/service-bus-tutorial-topics-subscriptions-portal/send-output.png)

9. 按 5 並觀察接收的訊息。 若未傳回 10 個訊息，請按 "m" 以顯示功能表，然後再按一次 5。

    ![接收輸出](./media/service-bus-tutorial-topics-subscriptions-portal/receive-output.png)

## <a name="clean-up-resources"></a>清除資源

不再需要命名空間和佇列時，請加以刪除。 若要這樣做，請在入口網站上選取這些資源，然後按一下 [刪除]。

## <a name="understand-the-sample-code"></a>了解範例程式碼

本節將詳細說明範例程式碼的功能。

### <a name="get-connection-string-and-topic"></a>取得連接字串和主題

首先，程式碼會宣告一組變數，使程式完成其餘的執行。

```csharp
string ServiceBusConnectionString;
string TopicName;

static string[] Subscriptions = { "S1", "S2", "S3" };
static IDictionary<string, string[]> SubscriptionFilters = new Dictionary<string, string[]> {
    { "S1", new[] { "StoreId IN('Store1', 'Store2', 'Store3')", "StoreId = 'Store4'"} },
    { "S2", new[] { "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'" } },
    { "S3", new[] { "sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')" } }
};
// You can have only have one action per rule and this sample code supports only one action for the first filter, which is used to create the first rule. 
static IDictionary<string, string> SubscriptionAction = new Dictionary<string, string> {
    { "S1", "" },
    { "S2", "" },
    { "S3", "SET sys.Label = 'SalesEvent'"  }
};
static string[] Store = { "Store1", "Store2", "Store3", "Store4", "Store5", "Store6", "Store7", "Store8", "Store9", "Store10" };
static string SysField = "sys.To";
static string CustomField = "StoreId";
static int NrOfMessagesPerStore = 1; // Send at least 1.
```

連接字串和主題名稱會透過命令列參數傳入 (如下所示)，然後以 `Main()` 方法讀取：

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string TopicName = "";

    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i + 1]}");
            ServiceBusConnectionString = args[i + 1]; // Alternatively enter your connection string here.
        }
        else if (args[i] == "-TopicName")
        {
            Console.WriteLine($"TopicName: {args[i + 1]}");
            TopicName = args[i + 1]; // Alternatively enter your queue name here.
        }
    }

    if (ServiceBusConnectionString != "" && TopicName != "")
    {
        Program P = StartProgram(ServiceBusConnectionString, TopicName);
        P.PresentMenu().GetAwaiter().GetResult();
    }
    else
    {
        Console.WriteLine("Specify -Connectionstring and -TopicName to execute the example.");
        Console.ReadKey();
    }
}
```

### <a name="remove-default-filters"></a>移除預設篩選

當您建立訂用帳戶時，服務匯流排會為每個訂用帳戶建立一個預設篩選。 此篩選可用來接收傳送至主題的每個訊息。 如果您想要使用自訂篩選，您可以移除預設篩選，如下列程式碼所示：

```csharp
private async Task RemoveDefaultFilters()
{
    Console.WriteLine($"Starting to remove default filters.");

    try
    {
        foreach (var subscription in Subscriptions)
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, subscription);
            await s.RemoveRuleAsync(RuleDescription.DefaultRuleName);
            Console.WriteLine($"Default filter for {subscription} has been removed.");
            await s.CloseAsync();
        }

        Console.WriteLine("All default Rules have been removed.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="create-filters"></a>建立篩選器

下列程式碼會新增在本教學課程中定義的自訂篩選：

```csharp
private async Task CreateCustomFilters()
{
    try
    {
        for (int i = 0; i < Subscriptions.Length; i++)
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, Subscriptions[i]);
            string[] filters = SubscriptionFilters[Subscriptions[i]];
            if (filters[0] != "")
            {
                int count = 0;
                foreach (var myFilter in filters)
                {
                    count++;

                    string action = SubscriptionAction[Subscriptions[i]];
                    if (action != "")
                    {
                        await s.AddRuleAsync(new RuleDescription
                        {
                            Filter = new SqlFilter(myFilter),
                            Action = new SqlRuleAction(action),
                            Name = $"MyRule{count}"
                        });
                    }
                    else
                    {
                        await s.AddRuleAsync(new RuleDescription
                        {
                            Filter = new SqlFilter(myFilter),
                            Name = $"MyRule{count}"
                        });
                    }
                }
            }

            Console.WriteLine($"Filters and actions for {Subscriptions[i]} have been created.");
        }

        Console.WriteLine("All filters and actions have been created.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="remove-your-custom-created-filters"></a>移除您已建立的自訂篩選

如果您想要移除訂用帳戶上的所有篩選，請參考下列程式碼所說明的操作方式：

```csharp
private async Task CleanUpCustomFilters()
{
    foreach (var subscription in Subscriptions)
    {
        try
        {
            SubscriptionClient s = new SubscriptionClient(ServiceBusConnectionString, TopicName, subscription);
            IEnumerable<RuleDescription> rules = await s.GetRulesAsync();
            foreach (RuleDescription r in rules)
            {
                await s.RemoveRuleAsync(r.Name);
                Console.WriteLine($"Rule {r.Name} has been removed.");
            }
            await s.CloseAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    Console.WriteLine("All default filters have been removed.\n");

    await PresentMenu();
}
```

### <a name="send-messages"></a>傳送訊息

將訊息傳送至主題的程序，與傳送訊息至佇列類似。 此範例說明如何使用工作清單和非同步處理來傳送訊息：

```csharp
public async Task SendMessages()
{
    try
    {
        TopicClient tc = new TopicClient(ServiceBusConnectionString, TopicName);

        var taskList = new List<Task>();
        for (int i = 0; i < Store.Length; i++)
        {
            taskList.Add(SendItems(tc, Store[i]));
        }

        await Task.WhenAll(taskList);
        await tc.CloseAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    Console.WriteLine("\nAll messages sent.\n");
}

private async Task SendItems(TopicClient tc, string store)
{
    for (int i = 0; i < NrOfMessagesPerStore; i++)
    {
        Random r = new Random();
        Item item = new Item(r.Next(5), r.Next(5), r.Next(5));

        // Note the extension class which is serializing an deserializing messages
        Message message = item.AsMessage();
        message.To = store;
        message.UserProperties.Add("StoreId", store);
        message.UserProperties.Add("Price", item.getPrice().ToString());
        message.UserProperties.Add("Color", item.getColor());
        message.UserProperties.Add("Category", item.getItemCategory());

        await tc.SendAsync(message);
        Console.WriteLine($"Sent item to Store {store}. Price={item.getPrice()}, Color={item.getColor()}, Category={item.getItemCategory()}"); ;
    }
}
```

### <a name="receive-messages"></a>接收訊息

訊息同樣會透過工作清單來接收，而程式碼會使用批次。 您可以使用批次進行傳送和接收，但此範例僅說明如何進行批次接收。 在實務上，您不會中斷迴圈，而是會保留迴圈並設定較高的時間範圍，例如一分鐘。 對訊息代理程式的接收呼叫在這段時間內會保持開啟，且訊息在送達時將會立即傳回，並發出新的接收呼叫。 這個概念稱為*長輪詢*。 使用您可以在[快速入門](service-bus-quickstart-portal.md)中和存放庫內的數個其他範例中看到的接收幫浦，是較為常見的選項。

```csharp
public async Task Receive()
{
    var taskList = new List<Task>();
    for (var i = 0; i < Subscriptions.Length; i++)
    {
        taskList.Add(this.ReceiveMessages(Subscriptions[i]));
    }

    await Task.WhenAll(taskList);
}

private async Task ReceiveMessages(string subscription)
{
    var entityPath = EntityNameHelper.FormatSubscriptionPath(TopicName, subscription);
    var receiver = new MessageReceiver(ServiceBusConnectionString, entityPath, ReceiveMode.PeekLock, RetryPolicy.Default, 100);

    while (true)
    {
        try
        {
            IList<Message> messages = await receiver.ReceiveAsync(10, TimeSpan.FromSeconds(2));
            if (messages.Any())
            {
                foreach (var message in messages)
                {
                    lock (Console.Out)
                    {
                        Item item = message.As<Item>();
                        IDictionary<string, object> myUserProperties = message.UserProperties;
                        Console.WriteLine($"StoreId={myUserProperties["StoreId"]}");

                        if (message.Label != null)
                        {
                            Console.WriteLine($"Label={message.Label}");
                        }

                        Console.WriteLine(
                            $"Item data: Price={item.getPrice()}, Color={item.getColor()}, Category={item.getItemCategory()}");
                    }

                    await receiver.CompleteAsync(message.SystemProperties.LockToken);
                }
            }
            else
            {
                break;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }

    await receiver.CloseAsync();
}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 Azure 入口網站佈建資源，然後從服務匯流排主題及其訂用帳戶傳送和接收訊息。 您已了解如何︰

> [!div class="checklist"]
> * 使用 Azure 入口網站建立服務匯流排主題，並建立該主題的一或多個訂用帳戶
> * 使用 .NET 程式碼新增主題篩選
> * 建立含有不同內容的兩個訊息
> * 傳送訊息，並確認它們送達預期的訂用帳戶中
> * 從訂用帳戶接收訊息

如需傳送和接收訊息的範例，請開始使用 [GitHub 上的服務匯流排範例](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted)。

繼續進行下一個教學課程，以深入了解如何使用服務匯流排的發佈/訂閱功能。

> [!div class="nextstepaction"]
> [使用 PowerShell 和主題/訂用帳戶來更新庫存](service-bus-tutorial-topics-subscriptions-powershell.md)

[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[完整網域名稱]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Azure portal]: https://portal.azure.com/

[connection-string]: ./media/service-bus-tutorial-topics-subscriptions-portal/connection-string.png
[service-bus-flow]: ./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png