---
title: 使用 Azure 通知中樞將當地語系化的通知傳送至 Windows 應用程式 | Microsoft Docs
description: 了解如何使用 Azure 通知中樞傳送本地化即時新聞通知。
services: notification-hubs
documentationcenter: windows
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: c454f5a3-a06b-45ac-91c7-f91210889b25
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 0e78b00e49b2ef468e693abfdcde8a138313ba63
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "41918312"
---
# <a name="tutorial-push-localized-notifications-to-windows-apps-by-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將當地語系化的通知推送至 Windows 應用程式
> [!div class="op_single_selector"]
> * [Windows 市集 C#](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

## <a name="overview"></a>概觀
本教學課程說明如何將當地語系化的通知推送至已向通知中樞服務註冊的行動裝置。 在此教學課程中，您會更新在[教學課程：將通知傳送至特定裝置 (通用 Windows 平台)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) 中建立的應用程式，以支援下列案例： 

- Windows 市集應用程式允許用戶端裝置指定語言，以及訂閱不同的即時新聞類別。
- 後端應用程式使用 Azure 通知中樞的**標記**和**範本**功能來廣播通知。

當您完成此教學課程後，行動應用程式將可讓您註冊您感興趣的類別，以及指定用來接收通知的語言。 後端應用程式會傳送依語言和裝置當地語系化的通知。 

在本教學課程中，您了解如何： 

> [!div class="checklist"]
> * 更新 Windows 應用程式以支援地區設定資訊
> * 更新後端應用程式以傳送當地語系化的通知
> * 測試應用程式

## <a name="prerequisites"></a>必要條件
完成[教學課程：將通知傳送至特定裝置 (通用 Windows 平台)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md)。 

在[教學課程：將通知傳送至特定裝置 (通用 Windows 平台)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) 中，您建置了使用**標記**來訂閱不同新聞**類別**之通知的應用程式。 在本教學課程中，您會使用通知中樞的**範本**功能，輕鬆地傳遞**當地語系化的**即時新聞通知。

概括而言，範本是可用來指定特定裝置應以何種格式接收通知的方式。 範本可參照您的應用程式後端所傳送的訊息中包含的屬性，藉以指定確切的裝載格式。 在本教學課程中，後端應用程式會傳送地區設定無從驗證、且包含所有支援語言的訊息：

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

裝置會註冊參照正確屬性的範本。 例如，要以英文接收快顯通知訊息的 Windows 市集應用程式，會註冊下列包含任何對應標記的範本：

```xml
<toast>
    <visual>
    <binding template=\"ToastText01\">
        <text id=\"1\">$(News_English)</text>
    </binding>
    </visual>
</toast>
```

若要深入了解範本，請參閱[範本](notification-hubs-templates-cross-platform-push-messages.md)一文。 

## <a name="update-windows-app-to-support-locale-information"></a>更新 Windows 應用程式以支援地區設定資訊

1. 開啟您為[教學課程：將通知傳送至特定裝置 (通用 Windows 平台)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) 建立的 Visual Studio 解決方案。 
2. 更新 **MainPage.xaml** 以加入地區設定下拉式方塊：

    ```xml
    <Grid Margin="120, 58, 120, 80"  
            Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top"/>
        <ComboBox Name="Locale" HorizontalAlignment="Left" VerticalAlignment="Center" Width="200" Grid.Row="1" Grid.Column="0">
            <x:String>English</x:String>
            <x:String>French</x:String>
            <x:String>Mandarin</x:String>
        </ComboBox>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="2" Grid.Column="0"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="3" Grid.Column="0"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="4" Grid.Column="0"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="2" Grid.Column="1"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="3" Grid.Column="1"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="4" Grid.Column="1"/>
        <Button Content="Subscribe" HorizontalAlignment="Center" Grid.Row="5" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
    </Grid>
    ```
2. 在您的 **Notifications** 類別中，將地區設定參數新增至 **StoreCategoriesAndSubscribe** 和 **SubscribeToCateories** 方法。

    ```csharp   
    public async Task<Registration> StoreCategoriesAndSubscribe(string locale, IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        ApplicationData.Current.LocalSettings.Values["locale"] = locale;
        return await SubscribeToCategories(categories);
    }

    public async Task<Registration> SubscribeToCategories(string locale, IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration. This makes supporting notifications across other platforms much easier.
        // Using the localized tags based on locale selected.
        string templateBodyWNS = String.Format("<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(News_{0})</text></binding></visual></toast>", locale);

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "localizedWNSTemplateExample", categories);
    }
    ```

    您會呼叫 *RegisterTemplateAsync* 方法，而非 *RegisterNativeAsync* 方法。 您會註冊範本取決於地區設定的特定通知格式。 您也會提供範本名稱 (例如 "localizedWNSTemplateExample")，因為您可能想要註冊多個範本 (例如，一個用於快顯通知，另一個用於圖格)。 您也需加以命名，以便將其更新或刪除。
   
    如果有裝置註冊了多個使用相同標記的範本，一個以該標記為目標的傳入訊息，將會使多個通知傳遞至裝置 (每個範本各一個)。 此行為在相同的邏輯訊息必須產生多個視覺化通知時將有所幫助，例如，在一個 Windows 市集應用程式中同時顯示徽章和快顯通知。
3. 新增下列方法，以擷取已儲存的地區設定：
   
    ```csharp
    public string RetrieveLocale()
    {
        var locale = (string) ApplicationData.Current.LocalSettings.Values["locale"];
        return locale != null ? locale : "English";
    }
    ```

4. 在您的 **MainPage.xaml.cs** 中，擷取地區設定下拉式方塊的現行值，並將其提供給 Notifications 類別的呼叫，以更新您的按鈕 click 處理常式，如下所示：
   
    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var locale = (string)Locale.SelectedItem;

        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(locale,
                categories);

        var dialog = new MessageDialog("Locale: " + locale + " Subscribed to: " + 
            string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```
4. 最後，在 App.xaml.cs 檔案中，請務必更新 `InitNotificationsAsync` 方法來擷取地區設定，並在訂閱時使用它：

    ```csharp   
    private async void InitNotificationsAsync()
    {
        var result = await notifications.SubscribeToCategories(notifications.RetrieveLocale());

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```

## <a name="send-localized-notifications-from-your-back-end"></a>從後端傳送已當地語系化的通知
傳送範本通知時，您只需提供一組屬性。在本教學課程中，後端應用程式會傳送一組包含已當地語系化版本的目前新聞的屬性，例如：

```json
{
    "News_English": "World News in English!",
    "News_French": "World News in French!",
    "News_Mandarin": "World News in Mandarin!"
}
```

在本節中，您會更新解決方案中的主控台應用程式專案。 在您先前建立的主控台應用程式中，使用下列程式碼修改 `SendTemplateNotificationAsync` 方法： 

> [!IMPORTANT]
> 請在程式碼中指定對您的通知中樞具有完整存取權的名稱和連接字串。 


```csharp
private static async void SendTemplateNotificationAsync()
{
    // Define the notification hub.
    NotificationHubClient hub = 
        NotificationHubClient.CreateClientFromConnectionString(
            "<connection string with full access>", "<hub name>");

    // Sending the notification as a template notification. All template registrations that contain 
    // "messageParam" or "News_<local selected>" and the proper tags will receive the notifications. 
    // This includes APNS, GCM, WNS, and MPNS template registrations.
    Dictionary<string, string> templateParams = new Dictionary<string, string>();

    // Create an array of breaking news categories.
    var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};
    var locales = new string[] { "English", "French", "Mandarin" };

    foreach (var category in categories)
    {
        templateParams["messageParam"] = "Breaking " + category + " News!";

        // Sending localized News for each tag too...
        foreach( var locale in locales)
        {
            string key = "News_" + locale;

            // Your real localized news content would go here.
            templateParams[key] = "Breaking " + category + " News in " + locale + "!";
        }

        await hub.SendTemplateNotificationAsync(templateParams, category);
    }
}
```

此簡單呼叫會將當地語系化的新聞片段傳送至您**所有的** 裝置 (不論平台為何)，因為您的通知中樞會建立並傳遞正確的原生承載給訂閱特定標記的所有裝置。

## <a name="test-the-app"></a>測試應用程式
1. 執行通用 Windows 市集應用程式。 請靜待 [註冊成功] 訊息顯示。

    ![行動應用程式和註冊](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/registration-successful.png)
1. 選取 [類別] 和 [地區設定]，然後按一下 [訂閱]。 應用程式會將選取的類別轉換成標籤，並在通知中心內為選取的標籤要求新裝置註冊。

    ![行動應用程式](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/mobile-app.png)
2.  您會看到關於**訂用帳戶**的**確認**訊息。 

    ![訂用帳戶訊息](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/subscription-message.png)
1. 收到確認後，請執行**主控台應用程式**，請以各個支援的語言傳送各種類別的通知。 請確認您只會收到所訂閱之類別的通知，且訊息是屬於您所選取的地區設定。 

    ![通知訊息](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/notifications.png)
 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何將當地語系化的通知推送至具有與其註冊相關聯之標記的特定裝置。 若要了解如何將通知推送給可能使用多項裝置的特定使用者，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[將通知推送給特定使用者](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)


<!-- Anchors. -->
[Template concepts]: #concepts
[The app user interface]: #ui
[Building the Windows Store client app]: #building-client
[Send notifications from your back-end]: #send
[Next Steps]:#next-steps

<!-- Images. -->

<!-- URLs. -->
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs/notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md

[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-dotnet
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-dotnet
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-dotnet
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-app-users-dotnet
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-dotnet
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js

[wns object]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
[Notification Hubs How-To for iOS]: http://msdn.microsoft.com/library/jj927168.aspx
[Notification Hubs How-To for Windows Store]: http://msdn.microsoft.com/library/jj927172.aspx
