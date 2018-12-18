---
title: 開始在 Xamarin.Forms 應用程式中使用 Mobile Apps 的驗證 | Microsoft Docs
description: 了解如何使用 Mobile Apps 透過眾多識別提供者驗證 Xamarin Forms 應用程式使用者，包括 AAD、Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: xamarin
author: panarasi
manager: crdun
editor: ''
ms.assetid: 9c55e192-c761-4ff2-8d88-72260e9f6179
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin
ms.devlang: dotnet
ms.topic: article
ms.date: 09/24/2018
ms.author: panarasi
ms.openlocfilehash: f7e500fb5856c7eec48a371042244b44dd944779
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063760"
---
# <a name="add-authentication-to-your-xamarin-forms-app"></a>將驗證新增至 Xamarin Forms 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

## <a name="overview"></a>概觀
此主題說明如何從用戶端應用程式驗證 App Service 行動應用程式的使用者。 在此教學課程中，您將使用 App Service 支援的識別提供者，將驗證新增至 Xamarin.Forms 快速入門專案。 由行動應用程式成功驗證並授權之後，就會顯示使用者識別碼值，而您也將可以存取受限制的資料庫資料。

## <a name="prerequisites"></a>必要條件
為了讓此教學課程產生最佳結果，建議您先完成 [建立 Xamarin.Forms 應用程式][1]教學課程。 完成此教學課程之後，您將會有一個多平台 TodoList 應用程式的 Xamarin.Forms 專案。

如果您不要使用下載的快速入門伺服器專案，必須將驗證擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱[使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK][2]。

## <a name="register-your-app-for-authentication-and-configure-app-services"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="redirecturl"></a>將您的應用程式新增至允許的外部重新導向 URL

安全的驗證會要求您為應用程式定義新的 URL 配置。 這讓驗證系統能夠在驗證程序完成之後，重新導向回到您的應用程式。 我們會在這整個教學課程中使用 URL 配置 appname。 不過，您可以使用任何您選擇的 URL 結構描述。 它對於您的行動應用程式而言應該是唯一的。 在伺服器端啟用重新導向：

1. 在 [Azure 入口網站][8] 中，選取您的 App Service。

2. 按一下 [驗證/授權] 功能表選項。

3. 在 [允許的外部重新導向 URL] 中，輸入 `url_scheme_of_your_app://easyauth.callback`。  此字串中的 **url_scheme_of_your_app** 是您行動應用程式的 URL 配置。  它必須遵循通訊協定的標準 URL 規格 (只使用字母和數字，並以字母為開頭)。  請記下您選擇的字串，因為您將需要在數個位置中使用該 URL 配置來調整您的行動應用程式程式碼。

4. 按一下 [確定]。

5. 按一下 [檔案] 。

## <a name="restrict-permissions-to-authenticated-users"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

## <a name="add-authentication-to-the-portable-class-library"></a>將驗證加入可攜式類別庫中
Mobile Apps 會使用 [MobileServiceClient][4] 的 [LoginAsync][3] 擴充方法，透過 App Service 驗證將使用者登入。 這個範例使用伺服器管理的驗證流程，在應用程式中顯示提供者的登入介面。 如需詳細資訊，請參閱[伺服器管理的驗證][5]。 若要在實際執行應用程式中提供更好的使用者體驗，您應該考慮改用[用戶端管理的驗證][6]。

為了驗證 Xamarin Forms 專案，請在應用程式的可攜式類別庫中定義 **IAuthenticate** 介面。 然後，將 [登入]  按鈕新增至可攜式類別庫中定義的使用者介面，讓您按一下來開始驗證。 驗證成功後，將會從行動應用程式後端載入資料。

為應用程式支援的每個平台實作 **IAuthenticate** 介面。

1. 在 Visual Studio 或 Xamarin Studio 中，從名稱中有 **Portable** 的專案中開啟 App.cs，也就是可攜式類別庫專案，然後新增下列 `using` 陳述式︰

        using System.Threading.Tasks;
2. 在 App.cs 中，在 `App` 類別定義之前緊接著加入下列 `IAuthenticate` 介面定義。

        public interface IAuthenticate
        {
            Task<bool> Authenticate();
        }
3. 將下列靜態成員新增至 **App** 類別，以使用平台特定實作來初始化介面。

        public static IAuthenticate Authenticator { get; private set; }

        public static void Init(IAuthenticate authenticator)
        {
            Authenticator = authenticator;
        }
4. 從可攜式類別庫專案中開啟 TodoList.xaml，在 buttonsPanel  配置項目中，將下列 *Button* 項目新增至現有按鈕後面︰

          <Button x:Name="loginButton" Text="Sign-in" MinimumHeightRequest="30"
            Clicked="loginButton_Clicked"/>

    此按鈕會向行動應用程式後端觸發伺服器管理的驗證。
5. 從可攜式類別庫專案中開啟 TodoList.xaml.cs，然後將下列欄位加入至 `TodoList` 類別︰

        // Track whether the user has authenticated.
        bool authenticated = false;
6. 使用下列程式碼取代 **OnAppearing** 方法：

        protected override async void OnAppearing()
        {
            base.OnAppearing();

            // Refresh items only when authenticated.
            if (authenticated == true)
            {
                // Set syncItems to true in order to synchronize the data
                // on startup when running in offline mode.
                await RefreshItems(true, syncItems: false);

                // Hide the Sign-in button.
                this.loginButton.IsVisible = false;
            }
        }

    此程式碼可確保只有在您通過驗證之後，才會從服務重新整理資料。
7. 將下列有關 **Clicked** 事件的處理常式新增至 **TodoList** 類別︰

        async void loginButton_Clicked(object sender, EventArgs e)
        {
            if (App.Authenticator != null)
                authenticated = await App.Authenticator.Authenticate();

            // Set syncItems to true to synchronize the data on startup when offline is enabled.
            if (authenticated == true)
                await RefreshItems(true, syncItems: false);
        }
8. 儲存變更，並建置可攜式類別庫專案，以驗證沒有錯誤。

## <a name="add-authentication-to-the-android-app"></a>將驗證加入 Android 應用程式中
本節說明如何在 Android 應用程式專案中實作 **IAuthenticate** 介面。 如果您不要支援 Android 裝置，請略過這一節。

1. 在 Visual Studio 或 Xamarin Studio 中，以滑鼠右鍵按一下 **droid** 專案，然後按一下 [設定為啟始專案]。
2. 按下 F5 在偵錯工具中啟動專案，然後確認在應用程式啟動後，發生狀態代碼 401 (未經授權) 的未處理例外狀況。 產生 401 代碼是因為只有獲授權的使用者才能存取後端。
3. 開啟 Android 專案中的 MainActivity.cs，然後加入下列 `using` 陳述式：

        using Microsoft.WindowsAzure.MobileServices;
        using System.Threading.Tasks;
4. 更新 **MainActivity** 類別來實作 **IAuthenticate** 介面，如下所示︰

        public class MainActivity : global::Xamarin.Forms.Platform.Android.FormsApplicationActivity, IAuthenticate
5. 更新 **MainActivity** 類別，新增 **IAuthenticate** 介面所需的 **MobileServiceUser** 欄位和 **Authenticate** 方法，如下所示︰

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            var success = false;
            var message = string.Empty;
            try
            {
                // Sign in with Facebook login using a server-managed flow.
                user = await TodoItemManager.DefaultManager.CurrentClient.LoginAsync(this, 
                    MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
                if (user != null)
                {
                    message = string.Format("you are now signed-in as {0}.",
                        user.UserId);
                    success = true;
                }
            }
            catch (Exception ex)
            {
                message = ex.Message;
            }

            // Display the success or failure message.
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.SetMessage(message);
            builder.SetTitle("Sign-in result");
            builder.Create().Show();

            return success;
        }

    如果您使用 Facebook 以外的識別提供者，請為 [MobileServiceAuthenticationProvider][7]選擇不同的值。

6. 透過新增下列 `<application>` 元素內部的 XML，更新 **AndroidManifest.xml** 檔案：

    ```xml
    <activity android:name="com.microsoft.windowsazure.mobileservices.authentication.RedirectUrlActivity" android:launchMode="singleTop" android:noHistory="true">
      <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="{url_scheme_of_your_app}" android:host="easyauth.callback" />
      </intent-filter>
    </activity>
    ```
    將 `{url_scheme_of_your_app}` 取代為您的 URL 配置。
7. 在 **MainActivity** 類別的 **OnCreate** 方法中，在呼叫 `LoadApplication()` 之前新增下列程式碼：

        // Initialize the authenticator before loading the app.
        App.Init((IAuthenticate)this);

    此程式碼可確保在應用程式載入之前初始化驗證器。
8. 重新建置應用程式，執行它，然後以您選擇的驗證提供者登入，並確認您能夠以已驗證的使用者身分存取資料表。

### <a name="troubleshooting"></a>疑難排解

**應用程式因 `Java.Lang.NoSuchMethodError: No static method startActivity` 而當機**

在某些情況下，支援套件中的衝突在 Visual studio 中僅顯示為警告，但應用程式在執行階段會因此例外狀況而當機。 在此情況下，您必須確定您專案中所參考的所有支援套件都具有相同版本。 [Azure Mobile Apps NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/) 具有 Android 平台的 `Xamarin.Android.Support.CustomTabs` 相依性，因此若您的專案使用較新的支援套件，您必須直接安裝具有必要版本的此套件以避免發生衝突。

## <a name="add-authentication-to-the-ios-app"></a>將驗證加入 iOS 應用程式中
本節說明如何在 iOS 應用程式專案中實作 **IAuthenticate** 介面。 如果您不要支援 iOS 裝置，請略過這一節。

1. 在 Visual Studio 或 Xamarin Studio 中，以滑鼠右鍵按一下 **iOS** 專案，然後按一下 [設定為啟始專案]。
2. 按下 F5 在偵錯工具中啟動專案，然後確認在應用程式啟動後，發生狀態代碼 401 (未經授權) 的未處理例外狀況。 產生 401 回應是因為只有獲授權的使用者才能存取後端。
3. 開啟 iOS 專案中的 AppDelegate.cs，然後加入下列 `using` 陳述式：

        using Microsoft.WindowsAzure.MobileServices;
        using System.Threading.Tasks;
4. 更新 **AppDelegate** 類別來實作 **IAuthenticate** 介面，如下所示︰

        public partial class AppDelegate : global::Xamarin.Forms.Platform.iOS.FormsApplicationDelegate, IAuthenticate
5. 更新 **AppDelegate** 類別，新增 **IAuthenticate** 介面所需的 **MobileServiceUser** 欄位和 **Authenticate** 方法，如下所示︰

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            var success = false;
            var message = string.Empty;
            try
            {
                // Sign in with Facebook login using a server-managed flow.
                if (user == null)
                {
                    user = await TodoItemManager.DefaultManager.CurrentClient
                        .LoginAsync(UIApplication.SharedApplication.KeyWindow.RootViewController,
                        MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
                    if (user != null)
                    {
                        message = string.Format("You are now signed-in as {0}.", user.UserId);
                        success = true;
                    }
                }
            }
            catch (Exception ex)
            {
               message = ex.Message;
            }

            // Display the success or failure message.
            UIAlertView avAlert = new UIAlertView("Sign-in result", message, null, "OK", null);
            avAlert.Show();

            return success;
        }

    如果您使用 Facebook 以外的識別提供者，請為 [MobileServiceAuthenticationProvider] 選擇不同的值。
    
6. 透過新增 **OpenUrl** 方法多載來更新 **AppDelegate** 類別，如下所示：

        public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
        {
            return TodoItemManager.DefaultManager.CurrentClient.ResumeWithURL(url);
        }
   
7. 在 **FinishedLaunching** 方法中，在呼叫 `LoadApplication()` 之前新增下面這行程式碼：

        App.Init(this);

    此程式碼可確保在應用程式載入之前初始化驗證器。

8. 開啟 Info.plist，並新增 **URL 類型**。 將**識別碼**設定為您選擇的名稱，將 **URL 配置**設為您應用程式的 URL 配置，並將**角色**設為「無」。

9. 重新建置應用程式，執行它，然後以您選擇的驗證提供者登入，並確認您能夠以已驗證的使用者身分存取資料表。

## <a name="add-authentication-to-windows-10-including-phone-app-projects"></a>將驗證新增至 Windows 10 (包括 Phone) 應用程式專案
本節說明如何在 Windows 10 應用程式專案中實作 **IAuthenticate** 介面。 相同的步驟適用於通用 Windows 平台 (UWP) 專案，但使用 **UWP** 專案 (內含已標註的變更)。 如果您不要支援 Windows 裝置，請略過這一節。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 **UWP** 專案，然後按一下 [設定為啟始專案]。
2. 按下 F5 在偵錯工具中啟動專案，然後確認在應用程式啟動後，發生狀態代碼 401 (未經授權) 的未處理例外狀況。 發生 401 回應是因為只有獲授權的使用者才能存取後端。
3. 開啟 Windows 應用程式專案的 MainPage.xaml.cs，然後加入下列 `using` 陳述式：

        using Microsoft.WindowsAzure.MobileServices;
        using System.Threading.Tasks;
        using Windows.UI.Popups;
        using <your_Portable_Class_Library_namespace>;

    以您的可攜式類別庫的命名空間取代 `<your_Portable_Class_Library_namespace>` 。
4. 更新 **MainPage** 類別來實作 **IAuthenticate** 介面，如下所示︰

        public sealed partial class MainPage : IAuthenticate
5. 更新 **MainPage** 類別，新增 **IAuthenticate** 介面所需的 **MobileServiceUser** 欄位和 **Authenticate** 方法，如下所示︰

        // Define a authenticated user.
        private MobileServiceUser user;

        public async Task<bool> Authenticate()
        {
            string message = string.Empty;
            var success = false;

            try
            {
                // Sign in with Facebook login using a server-managed flow.
                if (user == null)
                {
                    user = await TodoItemManager.DefaultManager.CurrentClient
                        .LoginAsync(MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
                    if (user != null)
                    {
                        success = true;
                        message = string.Format("You are now signed-in as {0}.", user.UserId);
                    }
                }

            }
            catch (Exception ex)
            {
                message = string.Format("Authentication Failed: {0}", ex.Message);
            }

            // Display the success or failure message.
            await new MessageDialog(message, "Sign-in result").ShowAsync();

            return success;
        }

    如果您使用 Facebook 以外的識別提供者，請為 [MobileServiceAuthenticationProvider][7]選擇不同的值。

1. 在 **MainPage** 類別的建構函式中，在呼叫 `LoadApplication()` 之前新增下面這行程式碼：

        // Initialize the authenticator before loading the app.
        <your_Portable_Class_Library_namespace>.App.Init(this);

    以您的可攜式類別庫的命名空間取代 `<your_Portable_Class_Library_namespace>` 。

3. 如果您使用 **UWP**，請將下列 **OnActivated** 方法覆寫新增至 **App** 類別：

       protected override void OnActivated(IActivatedEventArgs args)
       {
           base.OnActivated(args);

            if (args.Kind == ActivationKind.Protocol)
            {
                ProtocolActivatedEventArgs protocolArgs = args as ProtocolActivatedEventArgs;
                TodoItemManager.DefaultManager.CurrentClient.ResumeWithURL(protocolArgs.Uri);
            }
       }

3. 開啟 Package.appxmanifest，並新增**通訊協定**宣告。 將**顯示名稱**設定為您選擇的名稱，並將**名稱**設定為您應用程式的 URL 配置。

4. 重新建置應用程式，執行它，然後以您選擇的驗證提供者登入，並確認您能夠以已驗證的使用者身分存取資料表。

## <a name="next-steps"></a>後續步驟
現在您已經完成了這個基本驗證的教學課程，可以考慮繼續進行下列其中一個教學課程：

* [將推播通知新增至應用程式](app-service-mobile-xamarin-forms-get-started-push.md)

  了解如何將推播通知支援新增至應用程式，並設定行動應用程式後端以使用 Azure 通知中樞傳送推播通知。
* [啟用應用程式的離線同步處理](app-service-mobile-xamarin-forms-get-started-offline-data.md)

  了解如何使用行動應用程式後端，將離線支援新增至應用程式。 離線同步處理可讓使用者與行動應用程式進行互動 (檢視、新增或修改資料)，即使沒有網路連接也可以。

<!-- Images. -->

<!-- URLs. -->
[1]: app-service-mobile-xamarin-forms-get-started.md
[2]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[3]: https://msdn.microsoft.com/library/azure/dn268341(v=azure.10).aspx
[4]: https://msdn.microsoft.com/library/azure/JJ553674(v=azure.10).aspx
[5]: app-service-mobile-dotnet-how-to-use-client-library.md#serverflow
[6]: app-service-mobile-dotnet-how-to-use-client-library.md#clientflow
[7]: https://msdn.microsoft.com/library/azure/jj730936(v=azure.10).aspx
[8]: https://portal.azure.com
