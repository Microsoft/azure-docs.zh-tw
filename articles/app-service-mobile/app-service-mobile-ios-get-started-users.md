---
title: 在具有 Azure Mobile Apps 的 iOS 上新增驗證
description: 了解如何使用 Azure Mobile Apps 透過眾多身分識別提供者驗證 iOS 應用程式使用者，包括 Google、Facebook、Twitter 和 Microsoft。
services: app-service\mobile
documentationcenter: ios
author: conceptdev
manager: crdun
editor: ''
ms.assetid: ef3d3cbe-e7ca-45f9-987f-80c44209dc06
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: dotnet
ms.topic: article
ms.date: 01/23/2017
ms.author: crdun
ms.openlocfilehash: e0eeee05ebad2e8148752f988bbbc2f6a0d7c296
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27592689"
---
# <a name="add-authentication-to-your-ios-app"></a>將驗證新增至您的 iOS 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

在本教學課程中，您可以使用支援的身分識別提供者，將驗證加入 [iOS 快速入門] 專案。 本教學課程以 [iOS 快速入門] 教學課程為基礎，您必須先完成該教學課程。

## <a name="register"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="redirecturl"></a>將您的應用程式新增至允許的外部重新導向 URL

安全的驗證會要求您為應用程式定義新的 URL 配置。  這讓驗證系統能夠在驗證程序完成之後，重新導向回到您的應用程式。  我們會在這整個教學課程中使用 URL 配置 appname。  不過，您可以使用任何您選擇的 URL 結構描述。  它對於您的行動應用程式而言應該是唯一的。  若要在伺服器端啟用重新導向：

1. 在 [Azure 入口網站]中，選取您的 App Service。

2. 按一下 [驗證/授權] 功能表選項。

3. 按一下 [驗證提供者] 區段下方的 [Azure Active Directory]。

4. 將 [管理模式] 設定為 [進階]。

5. 在 [允許的外部重新導向 URL] 中，輸入 `appname://easyauth.callback`。  此字串中的 appname 是您行動應用程式的 URL 配置。  它必須遵循通訊協定的標準 URL 規格 (只使用字母和數字，並以字母為開頭)。  請記下您選擇的字串，因為您將需要在數個位置中使用該 URL 配置來調整您的行動應用程式程式碼。

6. 按一下 [SERVICEPRINCIPAL] 。

7. 按一下 [檔案] 。

## <a name="permissions"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

在 Xcode 中，按下 [Run]  以啟動應用程式。 因為應用程式會嘗試以未經驗證的使用者身分來存取後端，但 TodoItem 資料表現在需要驗證，所以會引發例外狀況。

## <a name="add-authentication"></a>將驗證加入應用程式
**Objective-C**：

1. 在 Mac 上，開啟 Xcode 中的 QSTodoListViewController.m，並新增下列方法：

    ```Objective-C
    - (void)loginAndGetData
    {
        QSAppDelegate *appDelegate = (QSAppDelegate *)[UIApplication sharedApplication].delegate;
        appDelegate.qsTodoService = self.todoService;

        [self.todoService.client loginWithProvider:@"google" urlScheme:@"appname" controller:self animated:YES completion:^(MSUser * _Nullable user, NSError * _Nullable error) {
            if (error) {
                NSLog(@"Login failed with error: %@, %@", error, [error userInfo]);
            }
            else {
                self.todoService.client.currentUser = user;
                NSLog(@"User logged in: %@", user.userId);

                [self refresh];
            }
        }];
    }
    ```

    如果您不使用 Google 作為識別提供者，請將 google 變更為 microsoftaccount、twitter、facebook 或 windowsazureactivedirectory。 如果您使用 Facebook，則必須在應用程式中[將 Facebook 網域列入允許清單][1]。

    使用您應用程式的唯一名稱來取代 **urlScheme**。  urlScheme 必須與您在 Azure 入口網站中的 [允許的外部重新導向 URL] 欄位中指定的 URL 配置通訊協定相同。 urlScheme 可在完成驗證要求之後，供驗證回呼用來切換回您的應用程式。

2. 使用以下程式碼來取代 QSTodoListViewController.m 內 `viewDidLoad` 中的 `[self refresh]`：

    ```Objective-C
    [self loginAndGetData];
    ```

3. 開啟 `QSAppDelegate.h` 檔案並新增下列程式碼：

    ```Objective-C
    #import "QSTodoService.h"

    @property (strong, nonatomic) QSTodoService *qsTodoService;
    ```

4. 開啟 `QSAppDelegate.m` 檔案並新增下列程式碼：

    ```Objective-C
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
    {
        if ([[url.scheme lowercaseString] isEqualToString:@"appname"]) {
            // Resume login flow
            return [self.qsTodoService.client resumeWithURL:url];
        }
        else {
            return NO;
        }
    }
    ```

   將下列程式碼新增於 `#pragma mark - Core Data stack` 行的正前方。  利用您在步驟 1 中所使用的 urlScheme 值來取代appname。

5. 開啟 `AppName-Info.plist` 檔案 (使用您的應用程式名稱來取代 AppName)，並新增下列程式碼：

    ```XML
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLName</key>
            <string>com.microsoft.azure.zumo</string>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>appname</string>
            </array>
        </dict>
    </array>
    ```

    此程式碼應該置於 `<dict>` 元素內部。  使用您在步驟 1 中選擇應的用程式名稱來取代 appname 字串 (在 **CFBundleURLSchemes** 的陣列內)。  您也可以在 plist 編輯器中進行這些變更 - 按一下 XCode 中的 `AppName-Info.plist` 檔案以開啟 plist 編輯器。

    使用您的 Apple 套件組合識別碼，來取代 **CFBundleURLName** 的 `com.microsoft.azure.zumo` 字串。

6. 按下 [執行] 以啟動應用程式，然後登入。 當您登入時，應該能夠檢視待辦事項清單並進行更新。

**Swift**：

1. 在 Mac 上，開啟 Xcode 中的 ToDoTableViewController.swift，並新增下列方法：

    ```swift
    func loginAndGetData() {

        guard let client = self.table?.client, client.currentUser == nil else {
            return
        }

        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        appDelegate.todoTableViewController = self

        let loginBlock: MSClientLoginBlock = {(user, error) -> Void in
            if (error != nil) {
                print("Error: \(error?.localizedDescription)")
            }
            else {
                client.currentUser = user
                print("User logged in: \(user?.userId)")
            }
        }

        client.login(withProvider:"google", urlScheme: "appname", controller: self, animated: true, completion: loginBlock)

    }
    ```

    如果您不使用 Google 作為識別提供者，請將 google 變更為 microsoftaccount、twitter、facebook 或 windowsazureactivedirectory。 如果您使用 Facebook，則必須在應用程式中[將 Facebook 網域列入允許清單][1]。

    使用您應用程式的唯一名稱來取代 **urlScheme**。  urlScheme 必須與您在 Azure 入口網站中的 [允許的外部重新導向 URL] 欄位中指定的 URL 配置通訊協定相同。 urlScheme 可在完成驗證要求之後，供驗證回呼用來切換回您的應用程式。

2. 移除 ToDoTableViewController.swift 中 `viewDidLoad()` 結尾處的 `self.refreshControl?.beginRefreshing()` 和 `self.onRefresh(self.refreshControl)` 行。 在其位置新增呼叫至 `loginAndGetData()` ：

    ```swift
    loginAndGetData()
    ```

3. 開啟 `AppDelegate.swift` 檔案，並將下列程式碼行新增至 `AppDelegate` 類別：

    ```swift
    var todoTableViewController: ToDoTableViewController?

    func application(_ application: UIApplication, openURL url: NSURL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
        if url.scheme?.lowercased() == "appname" {
            return (todoTableViewController!.table?.client.resume(with: url as URL))!
        }
        else {
            return false
        }
    }
    ```

    利用您在步驟 1 中所使用的 urlScheme 值來取代appname。

4. 開啟 `AppName-Info.plist` 檔案 (使用您的應用程式名稱來取代 AppName)，並新增下列程式碼：

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLName</key>
            <string>com.microsoft.azure.zumo</string>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>appname</string>
            </array>
        </dict>
    </array>
    ```

    此程式碼應該置於 `<dict>` 元素內部。  使用您在步驟 1 中選擇應的用程式名稱來取代 appname 字串 (在 **CFBundleURLSchemes** 的陣列內)。  您也可以在 plist 編輯器中進行這些變更 - 按一下 XCode 中的 `AppName-Info.plist` 檔案以開啟 plist 編輯器。

    使用您的 Apple 套件組合識別碼，來取代 **CFBundleURLName** 的 `com.microsoft.azure.zumo` 字串。

5. 按下 [執行] 以啟動應用程式，然後登入。 當您登入時，應該能夠檢視待辦事項清單並進行更新。

App Service 驗證會使用 Apples Inter-App Communication。  如需有關這個主題的詳細資訊，請參閱 [Apple 文件][2]
<!-- URLs. -->

[1]: https://developers.facebook.com/docs/ios/ios9#whitelist
[2]: https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html
[Azure 入口網站]: https://portal.azure.com

[iOS 快速入門]: app-service-mobile-ios-get-started.md

