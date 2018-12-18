---
title: 將驗證新增至您的通用 Windows 平台 (UWP) 應用程式 | Microsoft Docs
description: 了解如何使用 Azure App Service Mobile Apps，透過眾多識別提供者驗證通用 Windows 平台 (UWP) 應用程式使用者，包括 AAD、Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: windows
author: conceptdev
manager: panarasi
editor: ''
ms.assetid: 6cffd951-893e-4ce5-97ac-86e3f5ad9466
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: article
ms.date: 07/05/2017
ms.author: panarasi
ms.openlocfilehash: 4cc597f8aca13445034c8a1691b41018d4d9bc4b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38306569"
---
# <a name="add-authentication-to-your-windows-app"></a>將驗證新增至您的 Windows 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

本主題說明如何將雲端式驗證加入到您的行動應用程式。 在本教學課程中，您會使用 Azure App Service 所支援的身分識別提供者，將驗證新增至 Mobile Apps 的通用 Windows 平台 (UWP) 快速入門專案。 由行動應用程式後端成功驗證並授權之後，就會顯示使用者識別碼值。

本教學課程以 Azure Mobile Apps 快速入門為基礎。 您必須先完成 [開始使用 Mobile Apps](app-service-mobile-windows-store-dotnet-get-started.md)教學課程。

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

現在，您可以驗證是否已停用後端的匿名存取。 將 UWP 應用程式專案設定為啟始專案，部署並執行應用程式；應用程式啟動後，確認會引發狀態碼為 401 (未經授權) 的未處理例外狀況。 這是因為應用程式嘗試以未驗證的使用者身分存取您的行動應用程式程式碼，但 *TodoItem* 資料表現在需要驗證。

接下來，您要將應用程式更新為在要求 App Service 的資源之前必須驗證使用者。

## <a name="add-authentication"></a>將驗證新增至應用程式
1. 在 UWP 應用程式專案檔案 MainPage.cs 中，新增下列程式碼片段：
   
        // Define a member variable for storing the signed-in user. 
        private MobileServiceUser user;
   
        // Define a method that performs the authentication process
        // using a Facebook sign-in. 
        private async System.Threading.Tasks.Task<bool> AuthenticateAsync()
        {
            string message;
            bool success = false;
            try
            {
                // Change 'MobileService' to the name of your MobileServiceClient instance.
                // Sign-in using Facebook authentication.
                user = await App.MobileService
                    .LoginAsync(MobileServiceAuthenticationProvider.Facebook, "{url_scheme_of_your_app}");
                message =
                    string.Format("You are now signed in - {0}", user.UserId);
   
                success = true;
            }
            catch (InvalidOperationException)
            {
                message = "You must log in. Login Required";
            }
   
            var dialog = new MessageDialog(message);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
            return success;
        }
   
    此程式碼會使用 Facebook 登入來驗證使用者。 如果您打算使用除了 Facebook 以外的識別提供者，請將上方的 **MobileServiceAuthenticationProvider** 值變更成您提供者。
2. 取代 MainPage.xaml.cs 中的 **OnNavigatedTo()** 方法。 接下來，將 [登入]  按鈕新增至觸發驗證的應用程式。

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            if (e.Parameter is Uri)
            {
                App.MobileService.ResumeWithURL(e.Parameter as Uri);
            }
        }

3. 將下列程式碼片段新增至 MainPage.xaml.cs：
   
        private async void ButtonLogin_Click(object sender, RoutedEventArgs e)
        {
            // Login the user and then load data from the mobile app.
            if (await AuthenticateAsync())
            {
                // Switch the buttons and load items from the mobile app.
                ButtonLogin.Visibility = Visibility.Collapsed;
                ButtonSave.Visibility = Visibility.Visible;
                //await InitLocalStoreAsync(); //offline sync support.
                await RefreshTodoItems();
            }
        }
4. 開啟 MainPage.xaml 專案檔案，找到定義 **儲存** 按鈕的項目，並將它取代為下列程式碼︰
   
        <Button Name="ButtonSave" Visibility="Collapsed" Margin="0,8,8,0" 
                Click="ButtonSave_Click">
            <StackPanel Orientation="Horizontal">
                <SymbolIcon Symbol="Add"/>
                <TextBlock Margin="5">Save</TextBlock>
            </StackPanel>
        </Button>
        <Button Name="ButtonLogin" Visibility="Visible" Margin="0,8,8,0" 
                Click="ButtonLogin_Click" TabIndex="0">
            <StackPanel Orientation="Horizontal">
                <SymbolIcon Symbol="Permissions"/>
                <TextBlock Margin="5">Sign in</TextBlock> 
            </StackPanel>
        </Button>
5. 將下列程式碼片段新增至 App.xaml.cs：

        protected override void OnActivated(IActivatedEventArgs args)
        {
            if (args.Kind == ActivationKind.Protocol)
            {
                ProtocolActivatedEventArgs protocolArgs = args as ProtocolActivatedEventArgs;
                Frame content = Window.Current.Content as Frame;
                if (content.Content.GetType() == typeof(MainPage))
                {
                    content.Navigate(typeof(MainPage), protocolArgs.Uri);
                }
            }
            Window.Current.Activate();
            base.OnActivated(args);
        }
6. 開啟 Package.appxmanifest 檔案，瀏覽至 [宣告]，在 [可用宣告] 下拉式清單中選取 [通訊協定] 並按一下 [新增] 按鈕。 立即設定 [通訊協定] 宣告的 [屬性]。 在 [顯示名稱] 中，新增您要對應用程式使用者顯示的名稱。 在 [名稱] 中，新增您的 {url_scheme_of_your_app}。
7. 按 F5 鍵以執行應用程式、按一下 [登入]  按鈕，然後以您選擇的身分識別提供者登入應用程式。 當您成功登入之後，應用程式應會正確無誤地執行，而且您應能夠查詢後端並更新資料。

## <a name="tokens"></a>將驗證權杖儲存在用戶端上
先前範例所示範的標準登入，在每次應用程式啟動時，皆需要用戶端連絡身分識別提供者和 App Service。 這個方法不只效率不彰，而且如果同時有許多用戶試圖啟用您的應用程式時，還可能遇到使用量相關的問題。 更好的方法就是快取 App Service 傳回的授權權杖，然後嘗試在使用提供者形式登入前先使用此方法。

> [!NOTE]
> 無論您使用用戶端管理型或服務管理型驗證，皆可以快取 App Services 發行的權杖。 本教學課程使用服務受控驗證。
> 
> 

[!INCLUDE [mobile-windows-universal-dotnet-authenticate-app-with-token](../../includes/mobile-windows-universal-dotnet-authenticate-app-with-token.md)]

## <a name="next-steps"></a>後續步驟
現在您已經完成了這個基本驗證的教學課程，可以考慮繼續進行下列其中一個教學課程：

* [將推播通知新增至應用程式](app-service-mobile-windows-store-dotnet-get-started-push.md)  
  了解如何將推播通知支援新增至應用程式，並設定行動應用程式後端以使用 Azure 通知中樞傳送推播通知。
* [啟用應用程式的離線同步處理](app-service-mobile-windows-store-dotnet-get-started-offline-data.md)  
  ：了解如何使用行動應用程式後端，將離線支援加入至應用程式。 離線同步處理可讓使用者與行動應用程式進行互動&mdash;檢視、新增或修改資料&mdash;即使沒有網路連線進也可行。

<!-- URLs. -->
[Get started with your mobile app]: app-service-mobile-windows-store-dotnet-get-started.md
