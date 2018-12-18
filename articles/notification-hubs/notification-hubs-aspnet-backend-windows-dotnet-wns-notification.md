---
title: 使用 Azure 通知中樞將通知傳送給特定使用者 | Microsoft Docs
description: 了解如何使用通用 Windows 平台 (UWP) 應用程式將通知傳送給特定使用者。
documentationcenter: windows
author: dimazaid
manager: kpiteira
editor: spelluru
services: notification-hubs
ms.assetid: 012529f2-fdbc-43c4-8634-2698164b5880
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/04/2018
ms.author: dimazaid
ms.openlocfilehash: ee0e78402515a733731e6faf21b09886674cda51
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38452408"
---
# <a name="tutorial-send-notifications-to-specific-users-by-using-azure-notification-hubs"></a>教學課程：使用 Azure 通知中樞將通知傳送給特定使用者
[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

## <a name="overview"></a>概觀
本教學課程將示範如何使用 Azure 通知中心，來將推播通知傳送到特定裝置上的特定應用程式使用者。 ASP.NET WebAPI 後端是用來驗證用戶端。 後端在驗證用戶端應用程式使用者時，會自動將標記新增至通知登錄。 後端會使用此標記將通知傳送給特定使用者。 

> [!NOTE]
> 您可以在 [GitHub](https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/NotifyUsers) 上找到本教學課程的完整程式碼。 

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立 WebAPI 專案
> * 對 WebAPI 後端驗證用戶端
> * 使用 WebAPI 後端來註冊通知
> * 從 WebAPI 後端傳送通知
> * 發佈新的 WebAPI 後端
> * 更新用戶端專案的程式碼
> * 測試應用程式


## <a name="prerequisites"></a>先決條件
本教學課程的建置基礎，是通知中樞以及您在[教學課程：使用 Azure 通知中樞將通知傳送至通用 Windows 平台應用程式](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)教學課程中建立的 Visual Studio 專案。 因此，請先加以完成，再開始進行本教學課程。 

> [!NOTE]
> 如果您使用 Azure App Service 中的 Mobile Apps 作為後端服務，請參閱本教學課程的 [Mobile Apps 版本](../app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push.md) 。


&nbsp;

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="update-the-code-for-the-client-project"></a>更新用戶端專案的程式碼
在本節中，您會更新您為[教學課程：使用 Azure 通知中樞將通知傳送至通用 Windows 平台應用程式](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)教學課程完成之專案中的程式碼。 此專案應已與 Windows 市集產生關聯。 此外它也應設定成使用您的通知中樞。 在本節中，您會新增程式碼以呼叫新的 WebAPI 後端，並使用它來註冊和傳送通知。

1. 在 Visual Studio 中，開啟您為[教學課程：使用 Azure 通知中樞將通知傳送至通用 Windows 平台應用程式](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)建立的解決方案。
2. 在方案總管中，以滑鼠右鍵按一下 **WindowsApp** 專案，然後按一下 [管理 NuGet 套件]。
3. 在左側，按一下 [線上] 。
4. 在 [搜尋] 方塊中，輸入 **Http Client**。
5. 在 [結果] 清單中按一下 **System.Net.Http**，然後按一下 [安裝]。 完成安裝。
6. 回到 NuGet [搜尋] 方塊，輸入 **Json.net**。 安裝 **Newtonsoft.json** 套件，然後關閉 [NuGet 套件管理員] 視窗。
8. 在方案總管的 **WindowsApp** 專案中按兩下 **MainPage.xaml**，在 Visual Studio 編輯器中開啟該檔案。
9. 在 **MainPage.xaml** XML 程式碼中，將 `<Grid>` 區段取代為下列程式碼：此程式碼會新增使用者用於驗證的使用者名稱和密碼文字方塊。 它也會加入通知訊息的文字方塊，以及應接收通知的使用者名稱標記：

    ```xml   
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0" Text="Notify Users" HorizontalAlignment="Center" FontSize="48"/>

        <StackPanel Grid.Row="1" VerticalAlignment="Center">
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <TextBlock Grid.Row="0" Grid.ColumnSpan="3" Text="Username" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="UsernameTextBox" Grid.Row="1" Grid.ColumnSpan="3" Margin="20,0,20,0"/>
                <TextBlock Grid.Row="2" Grid.ColumnSpan="3" Text="Password" FontSize="24" Margin="20,0,20,0" />
                <PasswordBox Name="PasswordTextBox" Grid.Row="3" Grid.ColumnSpan="3" Margin="20,0,20,0"/>

                <Button Grid.Row="4" Grid.ColumnSpan="3" HorizontalAlignment="Center" VerticalAlignment="Center"
                            Content="1. Login and register" Click="LoginAndRegisterClick" Margin="0,0,0,20"/>

                <ToggleButton Name="toggleWNS" Grid.Row="5" Grid.Column="0" HorizontalAlignment="Right" Content="WNS" IsChecked="True" />
                <ToggleButton Name="toggleGCM" Grid.Row="5" Grid.Column="1" HorizontalAlignment="Center" Content="GCM" />
                <ToggleButton Name="toggleAPNS" Grid.Row="5" Grid.Column="2" HorizontalAlignment="Left" Content="APNS" />

                <TextBlock Grid.Row="6" Grid.ColumnSpan="3" Text="Username Tag To Send To" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="ToUserTagTextBox" Grid.Row="7" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <TextBlock Grid.Row="8" Grid.ColumnSpan="3" Text="Enter Notification Message" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="NotificationMessageTextBox" Grid.Row="9" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <Button Grid.Row="10" Grid.ColumnSpan="3" HorizontalAlignment="Center" Content="2. Send push" Click="PushClick" Name="SendPushButton" />
            </Grid>
        </StackPanel>
    </Grid>
    ```
11. 在 [方案總管] 中，開啟 [(Windows 8.1)] 和 [(Windows Phone 8.1)] 專案的 **MainPage.xaml.cs** 檔案。 在這兩個檔案頂端加入下列 `using` 陳述式：

    ```csharp    
    using System.Net.Http;
    using Windows.Storage;
    using System.Net.Http.Headers;
    using Windows.Networking.PushNotifications;
    using Windows.UI.Popups;
    using System.Threading.Tasks;
    ```
12. 在 **WindowsApp** 專案的 **MainPage.xaml.cs** 中，將下列成員新增至 `MainPage` 類別。 請務必將 `<Enter Your Backend Endpoint>` 取代為您先前取得的實際後端端點。 例如： `http://mybackend.azurewebsites.net`。
    
    ```csharp
    private static string BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";
    ```
13. 將下面的程式碼新增到 [(Windows 8.1)] 和 [(Windows Phone 8.1)] 專案之 **MainPage.xaml.cs** 中的 MainPage 類別。
    
    `PushClick` 方法是 [傳送推播] 按鈕的 click 處理常式。 它會呼叫後端以觸發所有裝置的通知，而所有裝置都具有符合 `to_tag` 參數的使用者名稱標記。 通知訊息會以要求主體的 JSON 內容形式傳送。
    
    `LoginAndRegisterClick` 方法是 [登入並註冊] 按鈕的 click 處理常式。 它會在本機儲存體中儲存基本驗證權杖 (代表驗證配置使用的任何權杖)，然後使用 `RegisterClient` 來註冊使用後端的通知。

    ```csharp
    private async void PushClick(object sender, RoutedEventArgs e)
    {
        if (toggleWNS.IsChecked.Value)
        {
            await sendPush("wns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleGCM.IsChecked.Value)
        {
            await sendPush("gcm", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleAPNS.IsChecked.Value)
        {
            await sendPush("apns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);

        }
    }

    private async Task sendPush(string pns, string userTag, string message)
    {
        var POST_URL = BACKEND_ENDPOINT + "/api/notifications?pns=" +
            pns + "&to_tag=" + userTag;

        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

            try
            {
                await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                    System.Text.Encoding.UTF8, "application/json"));
            }
            catch (Exception ex)
            {
                MessageDialog alert = new MessageDialog(ex.Message, "Failed to send " + pns + " message");
                alert.ShowAsync();
            }
        }
    }

    private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
    {
        SetAuthenticationTokenInLocalStorage();

        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        // The "username:<user name>" tag gets automatically added by the message handler in the backend.
        // The tag passed here can be whatever other tags you may want to use.
        try
        {
            // The device handle used is different depending on the device and PNS. 
            // Windows devices use the channel uri as the PNS handle.
            await new RegisterClient(BACKEND_ENDPOINT).RegisterAsync(channel.Uri, new string[] { "myTag" });

            var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
            SendPushButton.IsEnabled = true;
        }
        catch (Exception ex)
        {
            MessageDialog alert = new MessageDialog(ex.Message, "Failed to register with RegisterClient");
            alert.ShowAsync();
        }
    }

    private void SetAuthenticationTokenInLocalStorage()
    {
        string username = UsernameTextBox.Text;
        string password = PasswordTextBox.Password;

        var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
        ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
    }
    ```
1. 開啟 **App.xaml.cs** 檔案。 在 `InitNotificationsAsync()` in the `OnLaunched()` 的呼叫。 取消註解或刪除對 `InitNotificationsAsync()`的呼叫。 此按鈕處理常式會初始化通知註冊。

    ```csharp
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        //InitNotificationsAsync();
    ```
1. 以滑鼠右鍵按一下 **WindowsApp** 專案、按一下 [新增]，然後按一下 [類別]。 將類別命名為 **RegisterClient.cs**，然後按一下 [確定] 以產生類別。
   
   為了註冊推播通知，此類別會包裝連絡應用程式後端所需的 REST 呼叫。 它也會在本機儲存通知中心所建立的 *registrationIds* ，如 [從您的應用程式後端註冊](http://msdn.microsoft.com/library/dn743807.aspx)中的詳細說明。 當您按一下 [登入並註冊] 按鈕時，系統會使用儲存在本機儲存體中的授權權杖。
2. 在 RegisterClient.cs 檔案開頭加入下列 `using` 陳述式：

    ```csharp   
    using Windows.Storage;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Newtonsoft.Json;
    using System.Threading.Tasks;
    using System.Linq;
    ```
3. 在 `RegisterClient` 類別定義中加入下列程式碼。
   
    ```csharp
    private string POST_URL;

    private class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    public RegisterClient(string backendEndpoint)
    {
        POST_URL = backendEndpoint + "/api/register";
    }

    public async Task RegisterAsync(string handle, IEnumerable<string> tags)
    {
        var regId = await RetrieveRegistrationIdOrRequestNewOneAsync();

        var deviceRegistration = new DeviceRegistration
        {
            Platform = "wns",
            Handle = handle,
            Tags = tags.ToArray<string>()
        };

        var statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);

        if (statusCode == HttpStatusCode.Gone)
        {
            // regId is expired, deleting from local storage & recreating
            var settings = ApplicationData.Current.LocalSettings.Values;
            settings.Remove("__NHRegistrationId");
            regId = await RetrieveRegistrationIdOrRequestNewOneAsync();
            statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);
        }

        if (statusCode != HttpStatusCode.Accepted && statusCode != HttpStatusCode.OK)
        {
            // log or throw
            throw new System.Net.WebException(statusCode.ToString());
        }
    }

    private async Task<HttpStatusCode> UpdateRegistrationAsync(string regId, DeviceRegistration deviceRegistration)
    {
        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

            var putUri = POST_URL + "/" + regId;

            string json = JsonConvert.SerializeObject(deviceRegistration);
                            var response = await httpClient.PutAsync(putUri, new StringContent(json, Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    private async Task<string> RetrieveRegistrationIdOrRequestNewOneAsync()
    {
        var settings = ApplicationData.Current.LocalSettings.Values;
        if (!settings.ContainsKey("__NHRegistrationId"))
        {
            using (var httpClient = new HttpClient())
            {
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                var response = await httpClient.PostAsync(POST_URL, new StringContent(""));
                if (response.IsSuccessStatusCode)
                {
                    string regId = await response.Content.ReadAsStringAsync();
                    regId = regId.Substring(1, regId.Length - 2);
                    settings.Add("__NHRegistrationId", regId);
                }
                else
                {
                    throw new System.Net.WebException(response.StatusCode.ToString());
                }
            }
        }
        return (string)settings["__NHRegistrationId"];

    }
    ```
4. 儲存您的所有變更。

## <a name="test-the-application"></a>測試應用程式
1. 在這兩種 Windows 系統上啟動應用程式。
2. 輸入 [使用者名稱]和 [密碼]，如下列畫面所示。 它應該與您在 Windows Phone 上輸入的使用者名稱和密碼不同。
3. 按一下 [登入並註冊]  ，並確認顯示您已登入的對話方塊。 此程式碼也會啟用 [傳送推播] 按鈕。
   
    ![][14]
5. 然後，在 [收件者使用者名稱標記] 欄位中，輸入已註冊的使用者名稱。 輸入通知訊息，然後按一下 [傳送推播] 。
6. 只有已經使用相符使用者名稱標記所註冊的裝置才會收到通知訊息。
   
    ![][15]

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已學會如何針對具有與其註冊相關聯標記的使用者，將通知推送至這些特定使用者。 若要了解如何推送以位置為基礎的通知，請繼續進行下列教學課程： 

> [!div class="nextstepaction"]
>[推送以位置為基礎的通知](notification-hubs-push-bing-spartial-data-geofencing-notification.md)

[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-ui.png
[14]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-windows-instance-username.png
[15]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-notification-received.png
[16]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-send-message.png



<!-- URLs. -->
[Get started with Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Secure Push]: notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Notification Hubs Guidance]: http://msdn.microsoft.com/library/jj927170.aspx
