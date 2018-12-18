---
title: 開始在 Xamarin iOS 中使用行動應用程式的驗證
description: 了解如何使用行動應用程式透過眾多識別提供者驗證 Xamarin iOS 應用程式使用者，包括 AAD、Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 180cc61b-19c5-48bf-a16c-7181aef3eacc
ms.service: app-service-mobile
ms.workload: na
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: article
ms.date: 07/05/2017
ms.author: crdun
ms.openlocfilehash: be6ee88f43254ec3075a64299005d3597af968e7
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39224760"
---
# <a name="add-authentication-to-your-xamarinios-app"></a>將驗證新增至 Xamarin.iOS 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

本主題說明如何從用戶端應用程式驗證 App Service 行動應用程式的使用者。 在本教學課程中，您將使用 App Service 支援的身分識別提供者，將驗證新增至 Xamarin.iOS 快速入門專案。 由行動應用程式成功驗證並授權之後，就會顯示使用者識別碼值，而您也將可以存取受限制的資料庫資料。

您必須先完成 [建立 Xamarin.iOS 應用程式]教學課程。 如果您不要使用下載的快速入門伺服器專案，必須將驗證擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure 行動應用程式的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

## <a name="register-your-app-for-authentication-and-configure-app-services"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="add-your-app-to-the-allowed-external-redirect-urls"></a>將您的應用程式新增至允許的外部重新導向 URL

安全的驗證會要求您為應用程式定義新的 URL 配置。 這讓驗證系統能夠在驗證程序完成之後，重新導向回到您的應用程式。 我們會在這整個教學課程中使用 URL 配置 appname。 不過，您可以使用任何您選擇的 URL 結構描述。 它對於您的行動應用程式而言應該是唯一的。 在伺服器端啟用重新導向：

1. 在 [Azure 入口網站] 中，選取您的 App Service。

2. 按一下 [驗證/授權] 功能表選項。

3. 在 [允許的外部重新導向 URL] 中，輸入 `url_scheme_of_your_app://easyauth.callback`。  此字串中的 **url_scheme_of_your_app** 是您行動應用程式的 URL 配置。  它必須遵循通訊協定的標準 URL 規格 (只使用字母和數字，並以字母為開頭)。  請記下您選擇的字串，因為您將需要在數個位置中使用該 URL 配置來調整您的行動應用程式程式碼。

4. 按一下 [確定]。

5. 按一下 [檔案] 。

## <a name="restrict-permissions-to-authenticated-users"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

&nbsp;&nbsp;4. 在 Visual Studio 或 Xamarin Studio 中，在裝置或模擬器上執行用戶端專案。 確認在應用程式啟動後，發生狀態代碼 401 (未經授權) 的未處理例外狀況。 失敗都會記錄到偵錯工具主控台。 因此在 Visual Studio 中，您應該會在 [輸出] 視窗中看到失敗。

&nbsp;&nbsp;應用程式嘗試以未經驗證的使用者身分存取您的行動應用程式後端，因此發生此未經授權的失敗。 *TodoItem* 資料表現在需要驗證。

接下來，您將會更新用戶端應用程式，利用已驗證的使用者身分來要求行動應用程式後端的資源。

## <a name="add-authentication-to-the-app"></a>將驗證新增至應用程式
在本節中您將修改應用程式，以先顯示登入畫面再顯示資料。 應用程式在啟動時將不會連線到您的 App Service，且不會顯示任何資料。 在使用者第一次執行重新整理動作後，登入畫面將會出現；在成功登入後，將會顯示 todo 項目清單。

1. 在用戶端專案中開啟檔案 **QSTodoService.cs**，將下列 using 陳述式和具有存取子的 `MobileServiceUser`新增至 QSTodoService 類別：
 
        using UIKit;
       
        // Logged in user
        private MobileServiceUser user;
        public MobileServiceUser User { get { return user; } }
2. 使用下列定義，將名為 **Authenticate** 的新方法新增至 **QSTodoService**：

        public async Task Authenticate(UIViewController view)
        {
            try
            {
                AppDelegate.ResumeWithURL = url => url.Scheme == "zumoe2etestapp" && client.ResumeWithURL(url);
                user = await client.LoginAsync(view, MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

    >[AZURE.NOTE] 如果您使用的身分識別提供者不是 Facebook，請將傳給上述 **LoginAsync** 的值變更為下列其中一個：MicrosoftAccount、Twitter、Google 或 WindowsAzureActiveDirectory。

3. 開啟 **QSTodoListViewController.cs**。 修改 **ViewDidLoad** 的方法定義，移除結尾附近的 **RefreshAsync()** 呼叫：
   
        public override async void ViewDidLoad ()
        {
            base.ViewDidLoad ();
   
            todoService = QSTodoService.DefaultService;
            await todoService.InitializeStoreAsync();
   
            RefreshControl.ValueChanged += async (sender, e) => {
                await RefreshAsync();
            }
   
            // Comment out the call to RefreshAsync
            // await RefreshAsync();
        }
4. 修改方法 **RefreshAsync**，以驗證 **User** 屬性是否為 null。 在方法定義最上方新增下列程式碼：
   
        // start of RefreshAsync method
        if (todoService.User == null) {
            await QSTodoService.DefaultService.Authenticate(this);
            if (todoService.User == null) {
                Console.WriteLine("couldn't login!!");
                return;
            }
        }
        // rest of RefreshAsync method
5. 開啟 **AppDelegate.cs**，新增下列方法：

        public static Func<NSUrl, bool> ResumeWithURL;

        public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
        {
            return ResumeWithURL != null && ResumeWithURL(url);
        }
6. 開啟 **Info.plist** 檔案，瀏覽至 [進階] 區段中的 [URL 類型]。 立即設定 URL 類型的 [識別碼] 和 [URL 配置]，然後按一下 [新增 URL 類型]。 [URL 配置] 應該與您的 {url_scheme_of_your_app} 相同。
7. 在 Visual Studio 中，連接到 Mac 主機或 Visual Studio for Mac，執行以裝置或模擬器為目標的用戶端專案。 確認應用程式未顯示資料。
   
    將項目清單往下拉以執行重新整理動作，這會使登入畫面出現。 在您成功輸入有效認證後，應用程式將會顯示 todo 項目清單，且您可以對資料進行更新。

<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[建立 Xamarin.iOS 應用程式]: app-service-mobile-xamarin-ios-get-started.md
