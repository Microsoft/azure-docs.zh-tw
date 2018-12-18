---
title: 使用 Azure 通知中樞將通知推送至特定 Windows Phone | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 通知中樞，將通知推送至已向應用程式後端註冊的特定 (並非所有) Windows Phone 8 或 Windows Phone 8.1 裝置。
services: notification-hubs
documentationcenter: windows
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 42726bf5-cc82-438d-9eaa-238da3322d80
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-phone
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: fb408765a1185ac64a664cee458432bfce061c91
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41920523"
---
# <a name="tutorial-push-notifications-to-specific-windows-phone-devices-by-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知推送至特定 Windows Phone 裝置
[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

本教學課程說明如何使用 Azure 通知中樞將推播通知傳送至特定 Windows Phone 8 或 Windows Phone 8.1 裝置。 如果您的目標是 Windows Phone 8.1 (非 Silverlight)，請參閱本教學課程的 [Windows 通用](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)版本。

您可以透過在通知中樞內建立註冊時包含一或多個*標記*，來啟用此案例。 當標籤收到通知時，所有已註冊此標籤的裝置都會收到通知。 如需標記的詳細資訊，請參閱[註冊中的標記](notification-hubs-tags-segment-push-message.md)。

> [!NOTE]
> 通知中樞 Windows Phone SDK 不支援將 Windows 推播通知服務 (WNS) 與 Windows Phone 8.1 Silverlight app 搭配使用。 若要將 WNS (而非 MPNS) 與 Windows Phone 8.1 Silverlight 應用程式搭配使用，請遵循使用 REST API 的 [通知中樞 - Windows Phone Silverlight 教學課程]。

在本教學課程中，您了解如何： 

> [!div class="checklist"]
> * 在行動應用程式中新增類別選項
> * 註冊附有標記的通知
> * 傳送加註標記的通知
> * 測試應用程式

## <a name="prerequisites"></a>必要條件
完成[教學課程：使用 Azure 通知中樞將通知推送至 Windows Phone 應用程式](notification-hubs-windows-mobile-push-notifications-mpns.md)。 在本教學課程中，您會更新行動應用程式，以便能夠註冊您所感興趣的即時新聞類別，並僅接收這些類別的推播通知。 

## <a name="add-category-selection-to-the-mobile-app"></a>在行動應用程式中新增類別選項
第一個步驟是在您的現有主頁面上新增 UI 元素，以便使用者選取要註冊的類別。 使用者所選取的類別會儲存在裝置上。 啟動應用程式時，您的通知中心內會建立以所選取類別作為標籤的裝置註冊。

1. 開啟 MainPage.xaml 專案檔，然後使用下列程式碼來取代名為 `TitlePanel` 和 `ContentPanel` 的 **Grid** 元素：
   
        <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
            <TextBlock Text="Breaking News" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/>
            <TextBlock Text="Categories" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
        </StackPanel>
   
        <Grid Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
            <Grid.RowDefinitions>
                <RowDefinition Height="auto"/>
                <RowDefinition Height="auto" />
                <RowDefinition Height="auto" />
                <RowDefinition Height="auto" />
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
            </Grid.ColumnDefinitions>
            <CheckBox Name="WorldCheckBox" Grid.Row="0" Grid.Column="0">World</CheckBox>
            <CheckBox Name="PoliticsCheckBox" Grid.Row="1" Grid.Column="0">Politics</CheckBox>
            <CheckBox Name="BusinessCheckBox" Grid.Row="2" Grid.Column="0">Business</CheckBox>
            <CheckBox Name="TechnologyCheckBox" Grid.Row="0" Grid.Column="1">Technology</CheckBox>
            <CheckBox Name="ScienceCheckBox" Grid.Row="1" Grid.Column="1">Science</CheckBox>
            <CheckBox Name="SportsCheckBox" Grid.Row="2" Grid.Column="1">Sports</CheckBox>
            <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="3" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
        </Grid>
2. 將名為**通知**的類別新增至專案。 將**公用**修飾詞新增至類別定義。 然後，將下列 **using** 陳述式新增至新的程式碼檔案：
   
    ```csharp
    using Microsoft.Phone.Notification;
    using Microsoft.WindowsAzure.Messaging;
    using System.IO.IsolatedStorage;
    using System.Windows;
    ```
1. 將下列程式碼複製到新的 **Notifications** 類別：
   
    ```csharp
    private NotificationHub hub;

    // Registration task to complete registration in the ChannelUriUpdated event handler
    private TaskCompletionSource<Registration> registrationTask;

    public Notifications(string hubName, string listenConnectionString)
    {
        hub = new NotificationHub(hubName, listenConnectionString);
    }

    public IEnumerable<string> RetrieveCategories()
    {
        var categories = (string)IsolatedStorageSettings.ApplicationSettings["categories"];
        return categories != null ? categories.Split(',') : new string[0];
    }

    public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
    {
        var categoriesAsString = string.Join(",", categories);
        var settings = IsolatedStorageSettings.ApplicationSettings;
        if (!settings.Contains("categories"))
        {
            settings.Add("categories", categoriesAsString);
        }
        else
        {
            settings["categories"] = categoriesAsString;
        }
        settings.Save();

        return await SubscribeToCategories();
    }

    public async Task<Registration> SubscribeToCategories()
    {
        registrationTask = new TaskCompletionSource<Registration>();

        var channel = HttpNotificationChannel.Find("MyPushChannel");

        if (channel == null)
        {
            channel = new HttpNotificationChannel("MyPushChannel");
            channel.Open();
            channel.BindToShellToast();
            channel.ChannelUriUpdated += channel_ChannelUriUpdated;

            // This is optional, used to receive notifications while the app is running.
            channel.ShellToastNotificationReceived += channel_ShellToastNotificationReceived;
        }

        // If channel.ChannelUri is not null, complete the registrationTask here.  
        // If it is null, the registrationTask will be completed in the ChannelUriUpdated event handler.
        if (channel.ChannelUri != null)
        {
            await RegisterTemplate(channel.ChannelUri);
        }

        return await registrationTask.Task;
    }

    async void channel_ChannelUriUpdated(object sender, NotificationChannelUriEventArgs e)
    {
        await RegisterTemplate(e.ChannelUri);
    }

    async Task<Registration> RegisterTemplate(Uri channelUri)
    {
        // Using a template registration to support notifications across platforms.
        // Any template notifications that contain messageParam and a corresponding tag expression
        // will be delivered for this registration.

        const string templateBodyMPNS = "<wp:Notification xmlns:wp=\"WPNotification\">" +
                                            "<wp:Toast>" +
                                                "<wp:Text1>$(messageParam)</wp:Text1>" +
                                            "</wp:Toast>" +
                                        "</wp:Notification>";

        // The stored categories tags are passed with the template registration.

        registrationTask.SetResult(await hub.RegisterTemplateAsync(channelUri.ToString(), 
            templateBodyMPNS, "simpleMPNSTemplateExample", this.RetrieveCategories()));

        return await registrationTask.Task;
    }

    // This is optional. It is used to receive notifications while the app is running.
    void channel_ShellToastNotificationReceived(object sender, NotificationEventArgs e)
    {
        StringBuilder message = new StringBuilder();
        string relativeUri = string.Empty;

        message.AppendFormat("Received Toast {0}:\n", DateTime.Now.ToShortTimeString());

        // Parse out the information that was part of the message.
        foreach (string key in e.Collection.Keys)
        {
            message.AppendFormat("{0}: {1}\n", key, e.Collection[key]);

            if (string.Compare(
                key,
                "wp:Param",
                System.Globalization.CultureInfo.InvariantCulture,
                System.Globalization.CompareOptions.IgnoreCase) == 0)
            {
                relativeUri = e.Collection[key];
            }
        }

        // Display a dialog of all the fields in the toast.
        System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() => 
        { 
            MessageBox.Show(message.ToString()); 
        });
    }
    ```
    
    本類別使用獨立儲存體，來儲存此裝置接收的新聞類別。 它也包含使用 [範本](notification-hubs-templates-cross-platform-push-messages.md) 通知來註冊這些類別的方法。
1. 在 App.xaml.cs 專案檔案中，新增下列屬性至 **App** 類別。 以您的通知中樞名稱與先前取得的 *DefaultListenSharedAccessSignature* 連接字串，取代 `<hub name>` 和 `<connection string with listen access>` 預留位置。
   
    ```csharp
    public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");
    ```

   > [!NOTE]
   > 因為隨用戶端應用程式散佈的憑證通常不安全，您應只將接聽存取權的金鑰隨用戶端應用程式散佈。 您的應用程式可透過接聽存取權來註冊通知，但無法修改現有的註冊或無法傳送通知。 在安全的後端服務中，會使用完整存取金鑰來傳送通知和變更現有的註冊。
   > 
   > 
2. 在 MainPage.xaml.cs 中新增下列程式碼行：
   
    ```csharp
    using Windows.UI.Popups;
    ```
1. 在 MainPage.xaml.cs 專案檔案中新增下列方法：
   
    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var categories = new HashSet<string>();
        if (WorldCheckBox.IsChecked == true) categories.Add("World");
        if (PoliticsCheckBox.IsChecked == true) categories.Add("Politics");
        if (BusinessCheckBox.IsChecked == true) categories.Add("Business");
        if (TechnologyCheckBox.IsChecked == true) categories.Add("Technology");
        if (ScienceCheckBox.IsChecked == true) categories.Add("Science");
        if (SportsCheckBox.IsChecked == true) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

        MessageBox.Show("Subscribed to: " + string.Join(",", categories) + " on registration id : " +
            result.RegistrationId);
    }
    ```
   
    此方法會建立一份類別清單，並使用 **Notifications** 類別在本機儲存體中儲存清單，並在通知中心註冊對應標籤。 變更類別時，系統會使用新類別重新建立註冊。

您的應用程式現在可以在裝置上的本機儲存體中儲存一組類別，並在使用者每次變更類別選項時在通知中心註冊。

## <a name="register-for-notifications"></a>註冊通知
這些步驟會在啟動時，使用已儲存在本機儲存體中的類別在通知中心註冊。

> [!NOTE]
> 由於 Microsoft 推播通知服務 (WPNS) 所指派的通道 URI 可能隨時會變更，因此您應經常註冊通知以避免通知失敗。 此範例會在應用程式每次啟動時註冊通知。 若是經常執行 (一天多次) 的應用程式，如果距離上次註冊的時間不到一天，則您可能可以略過註冊以保留頻寬。

1. 開啟 App.xaml.cs 檔案，並將 **async** 修飾詞新增至 **Application_Launching** 方法，同時使用以下代碼，取代您新增至[開始使用通知中樞]中的通知中樞註冊代碼：
   
    ```csharp
    private async void Application_Launching(object sender, LaunchingEventArgs e)
    {
        var result = await notifications.SubscribeToCategories();
    
        if (result != null)
            System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() =>
            {
                MessageBox.Show("Registration Id :" + result.RegistrationId, "Registered", MessageBoxButton.OK);
            });
    }
    ```
   
    此程式碼可確保應用程式每次啟動時都會從本機儲存體擷取類別，並要求這些類別的註冊。
2. 在 MainPage.xaml.cs 專案檔案中，新增下列實作 **OnNavigatedTo** 方法的程式碼：
   
    ```csharp
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        var categories = ((App)Application.Current).notifications.RetrieveCategories();
    
        if (categories.Contains("World")) WorldCheckBox.IsChecked = true;
        if (categories.Contains("Politics")) PoliticsCheckBox.IsChecked = true;
        if (categories.Contains("Business")) BusinessCheckBox.IsChecked = true;
        if (categories.Contains("Technology")) TechnologyCheckBox.IsChecked = true;
        if (categories.Contains("Science")) ScienceCheckBox.IsChecked = true;
        if (categories.Contains("Sports")) SportsCheckBox.IsChecked = true;
    }
    ```
   
    此程式碼會根據先前儲存之類別的狀態來更新主頁面。

現在已完成此應用程式，且可在裝置本機儲存體中儲存一組類別，以供每次使用者變更類別選項在通知中心註冊時使用。 接著，定義可將類別通知傳送至此應用程式的後端。

## <a name="send-tagged-notifications"></a>傳送加註標記的通知
[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="test-the-app"></a>測試應用程式
1. 在 Visual Studio 中，按 F5 以編譯並啟動應用程式。
   
    ![具有類別的行動應用程式][1]
   
    應用程式 UI 提供一組切換，可讓您選擇要訂閱的類別。
2. 啟用一或多個類別切換，然後按一下 [訂閱] 。
   
    應用程式會將選取的類別轉換成標籤，並在通知中心內為選取的標籤要求新裝置註冊。 系統會傳回已註冊類別並顯示在對話方塊中。
   
    ![已訂閱的訊息][2]
3. 收到您各項類別的訂閱已完成的確認後，請執行主控台應用程式，以傳送各種類別的通知。 請確認您只會收到所訂閱類別的通知。
   
    ![通知訊息][3]

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何將通知推送至具有與其註冊相關聯之標記的特定裝置。 若要了解如何將通知推送給可能使用多項裝置的特定使用者，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[將通知推送給特定使用者](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)


<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[1]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-breakingnews.png
[2]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-registration.png
[3]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-toast.png



<!-- URLs.-->
[開始使用通知中樞]: notification-hubs-windows-mobile-push-notifications-mpns.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for Windows Phone]: ??

