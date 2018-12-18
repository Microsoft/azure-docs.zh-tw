---
title: 開始在 Xamarin Android 中使用行動應用程式的驗證
description: 了解如何使用行動應用程式透過眾多識別提供者驗證 Xamarin Android 應用程式使用者，包括 AAD、Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: panarasi
editor: ''
ms.assetid: 570fc12b-46a9-4722-b2e0-0d1c45fb2152
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: article
ms.date: 09/24/2018
ms.author: panarasi
ms.openlocfilehash: d496801894560310a4225eae8a32fced52bcc428
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063535"
---
# <a name="add-authentication-to-your-xamarinandroid-app"></a>將驗證新增至 Xamarin.Android 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

此主題說明如何從用戶端應用程式驗證行動應用程式的使用者。 在此教學課程中，您會使用 Azure 行動應用程式所支援的身分識別提供者將驗證新增至快速入門專案。 在行動應用程式中成功驗證並授權之後，就會顯示使用者識別碼值。

此教學課程以行動應用程式快速入門為基礎。 您也必須先完成 [建立 Xamarin.Android 應用程式教學課程]。 如果您不要使用下載的快速入門伺服器專案，必須將驗證擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

## <a name="register"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="redirecturl"></a>將您的應用程式新增至允許的外部重新導向 URL

安全的驗證會要求您為應用程式定義新的 URL 配置。 這讓驗證系統能夠在驗證程序完成之後，重新導向回到您的應用程式。 我們會在這整個教學課程中使用 URL 配置 appname。 不過，您可以使用任何您選擇的 URL 結構描述。 它對於您的行動應用程式而言應該是唯一的。 在伺服器端啟用重新導向：

1. 在 [Azure 入口網站] 中，選取您的 App Service。

2. 按一下 [驗證/授權] 功能表選項。

3. 在 [允許的外部重新導向 URL] 中，輸入 `url_scheme_of_your_app://easyauth.callback`。  此字串中的 **url_scheme_of_your_app** 是您行動應用程式的 URL 配置。  它必須遵循通訊協定的標準 URL 規格 (只使用字母和數字，並以字母為開頭)。  請記下您選擇的字串，因為您將需要在數個位置中使用該 URL 配置來調整您的行動應用程式程式碼。

4. 按一下 [確定]。

5. 按一下 [檔案] 。

## <a name="permissions"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

在 Visual Studio 或 Xamarin Studio 中，在裝置或模擬器上執行用戶端專案。 確認在應用程式啟動後，發生狀態代碼 401 (未經授權) 的未處理例外狀況。 這是因為應用程式嘗試以未驗證的使用者身分存取您的行動應用程式程式碼。 *TodoItem* 資料表現在需要驗證。

接下來，您將會更新用戶端應用程式，利用已驗證的使用者身分來要求行動應用程式後端的資源。

## <a name="add-authentication"></a>將驗證新增至應用程式
應用程式已更新，要求使用者在資料顯示前點選 [登入]  按鈕並驗證。

1. 將下列程式碼加入 **TodoActivity** 類別：
   
        // Define a authenticated user.
        private MobileServiceUser user;
        private async Task<bool> Authenticate()
        {
                var success = false;
                try
                {
                    // Sign in with Facebook login using a server-managed flow.
                    user = await client.LoginAsync(this,
                        MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
                    CreateAndShowDialog(string.Format("you are now logged in - {0}",
                        user.UserId), "Logged in!");
   
                    success = true;
                }
                catch (Exception ex)
                {
                    CreateAndShowDialog(ex, "Authentication failed");
                }
                return success;
        }
   
        [Java.Interop.Export()]
        public async void LoginUser(View view)
        {
            // Load data only after authentication succeeds.
            if (await Authenticate())
            {
                //Hide the button after authentication succeeds.
                FindViewById<Button>(Resource.Id.buttonLoginUser).Visibility = ViewStates.Gone;
   
                // Load the data.
                OnRefreshItemsSelected();
            }
        }
   
    這會建立一個新方法以驗證使用者，以及建立新 [登入]  按鈕的方法處理常式。 上述範例程式碼中的使用者是使用 Facebook 登入進行驗證。 對話方塊會在驗證後用來顯示使用者識別碼。
   
   > [!NOTE]
   > 如果您使用的身分識別提供者不是 Facebook，請將傳遞給上述 **LoginAsync** 的值變更為下列其中之一：MicrosoftAccount、Twitter、Google 或 WindowsAzureActiveDirectory。
   > 
   > 
2. 在 **OnCreate** 方法中，刪除或註解下列程式碼行：
   
        OnRefreshItemsSelected ();
3. 在 Activity_To_Do.axml 檔案中，在現有的 AddItem 按鈕之前加入下列 LoginUser 按鈕定義：
   
          <Button
            android:id="@+id/buttonLoginUser"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="LoginUser"
            android:text="@string/login_button_text" />
4. 將下列元素新增到 Strings.xml 資源檔：
   
        <string name="login_button_text">Sign in</string>
5. 開啟 AndroidManifest.xml 檔案，在 `<application>` XML 元素內新增下列程式碼：

        <activity android:name="com.microsoft.windowsazure.mobileservices.authentication.RedirectUrlActivity" android:launchMode="singleTop" android:noHistory="true">
          <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="{url_scheme_of_your_app}" android:host="easyauth.callback" />
          </intent-filter>
        </activity>

6. 在 Visual Studio 或 Xamarin Studio 中，在裝置或模擬器上執行用戶端專案，並使用您選擇的身分識別提供者登入。 當您成功登入後，應用程式將會顯示您的登入識別碼以及 todo 項目的清單，您可以對資料進行更新。

## <a name="troubleshooting"></a>疑難排解

**應用程式因 `Java.Lang.NoSuchMethodError: No static method startActivity` 而當機**

在某些情況下，支援套件中的衝突在 Visual studio 中僅顯示為警告，但應用程式在執行階段會因此例外狀況而當機。 在此情況下，您必須確定您專案中所參考的所有支援套件都具有相同版本。 [Azure Mobile Apps NuGet 套件](https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client/) 具有 Android 平台的 `Xamarin.Android.Support.CustomTabs` 相依性，因此若您的專案使用較新的支援套件，您必須直接安裝具有必要版本的此套件以避免發生衝突。

<!-- URLs. -->
[建立 Xamarin.Android 應用程式教學課程]: app-service-mobile-xamarin-android-get-started.md
