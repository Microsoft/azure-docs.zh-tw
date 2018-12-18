---
title: 如何使用適用於 Azure Mobile Apps 的 iOS SDK
description: 如何使用適用於 Azure Mobile Apps 的 iOS SDK
services: app-service\mobile
documentationcenter: ios
author: conceptdev
editor: ''
ms.assetid: 4e8e45df-c36a-4a60-9ad4-393ec10b7eb9
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 0de561b177a1474b0ce4f0f203803e8265db5e7a
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818341"
---
# <a name="how-to-use-ios-client-library-for-azure-mobile-apps"></a>如何使用適用於 Azure Mobile Apps 的 iOS 用戶端程式庫

[!INCLUDE [app-service-mobile-selector-client-library](../../includes/app-service-mobile-selector-client-library.md)]

本指南說明如何使用最新的 [Azure Mobile Apps iOS SDK][1] 執行常見案例。 如果您是 Azure Mobile Apps 的新手，請先完成 [Azure Mobile Apps 快速啟動] 以建立後端、建立資料表及下載預先建置的 iOS Xcode 專案。 在本指南中，我們會著重於用戶端 iOS SDK。 若要深入了解後端的伺服器端 SDK，請參閱伺服器 SDK 做法。

## <a name="reference-documentation"></a>參考文件

iOS 用戶端 SDK 的參考文件位於此處：[Azure Mobile Apps iOS 用戶端參考資料][2]。

## <a name="supported-platforms"></a>支援的平台

iOS SDK 支援 Objective-C 專案、Swift 2.2 專案，以及適用於 iOS 8.0 版或更新版本的 Swift 2.3 專案。

「伺服器流程」驗證在呈現的 UI 中使用 WebView。  如果裝置無法呈現 WebView UI，您需要本產品無法提供的其他驗證方法。  
因此，此 SDK 不適用於手錶類型或受到類似限制的裝置。

## <a name="Setup"></a>設定和必要條件

本指南假設您已建立包含資料表的後端。 本指南假設資料表的結構描述與這些教學課程中的資料表相同。 本指南也假設您在程式碼中，參考了 `MicrosoftAzureMobile.framework` 並匯入了 `MicrosoftAzureMobile/MicrosoftAzureMobile.h`。

## <a name="create-client"></a>作法：建立用戶端

若要在專案中存取 Azure Mobile Apps 後端，請建立 `MSClient`。 以應用程式 URL 取代 `AppUrl` 。 您可以將 `gatewayURLString` 和 `applicationKey` 留白。 如果您設定驗證的閘道器，請將 `gatewayURLString` 填入閘道器 URL。

**Objective-C**：

```objc
MSClient *client = [MSClient clientWithApplicationURLString:@"AppUrl"];
```

**Swift**：

```swift
let client = MSClient(applicationURLString: "AppUrl")
```

## <a name="table-reference"></a>作法：建立資料表參考

若要存取或更新資料，請建立後端資料表的參考。 以您的資料表名稱取代 `TodoItem`

**Objective-C**：

```objc
MSTable *table = [client tableWithName:@"TodoItem"];
```

**Swift**：

```swift
let table = client.tableWithName("TodoItem")
```

## <a name="querying"></a>作法：查詢資料

若要建立資料庫查詢，請查詢 `MSTable` 物件。 下列查詢會取得 `TodoItem` 中的所有項目並記錄每個項目的文字。

**Objective-C**：

```objc
[table readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) { // error is nil if no error occured
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) { // items is NSArray of records that match query
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**：

```swift
table.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="filtering"></a>作法：篩選傳回的資料

若要篩選結果，有許多可用的選項。

若要使用述詞篩選，請使用 `NSPredicate` 和 `readWithPredicate`。 下列篩選器傳回的資料只尋找未完成的待辦事項。

**Objective-C**：

```objc
// Create a predicate that finds items where complete is false
NSPredicate * predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
// Query the TodoItem table
[table readWithPredicate:predicate completion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**：

```swift
// Create a predicate that finds items where complete is false
let predicate =  NSPredicate(format: "complete == NO")
// Query the TodoItem table
table.readWithPredicate(predicate) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="query-object"></a>作法：使用 MSQuery

若要執行複雜的查詢 (包括排序和分頁)，請使用述詞直接建立 `MSQuery` 物件：

**Objective-C**：

```objc
MSQuery *query = [table query];
MSQuery *query = [table queryWithPredicate: [NSPredicate predicateWithFormat:@"complete == NO"]];
```

**Swift**：

```swift
let query = table.query()
let query = table.queryWithPredicate(NSPredicate(format: "complete == NO"))
```

`MSQuery` 可讓您控制幾種查詢行為。

* 指定結果的順序
* 限制要傳回的欄位
* 限制要傳回的記錄數
* 指定回應中的總計數
* 在要求中指定自訂查詢字串參數
* 套用其他函式

在物件上呼叫 `readWithCompletion` 以執行 `MSQuery` 查詢。

## <a name="sorting"></a>做法：使用 MSQuery 排序資料

我們來看一下範例如何排序結果。 若要根據 'text' 欄位依照遞增順序排序，然後再根據 'complete' 欄位依照遞減順序排序，請叫用 `MSQuery` ，如下所示︰

**Objective-C**：

```objc
[query orderByAscending:@"text"];
[query orderByDescending:@"complete"];
[query readWithCompletion:^(MSQueryResult *result, NSError *error) {
        if(error) {
                NSLog(@"ERROR %@", error);
        } else {
                for(NSDictionary *item in result.items) {
                        NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
                }
        }
}];
```

**Swift**：

```swift
query.orderByAscending("text")
query.orderByDescending("complete")
query.readWithCompletion { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let items = result?.items {
        for item in items {
            print("Todo Item: ", item["text"])
        }
    }
}
```

## <a name="selecting"></a><a name="parameters"></a>作法：使用 MSQuery 限制欄位和展開查詢字串參數

若要限制在查詢中傳回的欄位，請在 **selectFields** 屬性中指定欄位的名稱。 本範例僅會傳回文字和已完成欄位：

**Objective-C**：

```objc
query.selectFields = @[@"text", @"complete"];
```

**Swift**：

```swift
query.selectFields = ["text", "complete"]
```

若要在伺服器要求中包含額外的查詢字串參數 (例如有某個自訂的伺服器端指令碼使用這些參數)，請如下填入 `query.parameters` ：

**Objective-C**：

```objc
query.parameters = @{
    @"myKey1" : @"value1",
    @"myKey2" : @"value2",
};
```

**Swift**：

```swift
query.parameters = ["myKey1": "value1", "myKey2": "value2"]
```

## <a name="paging"></a>如何：設定頁面大小

使用 Azure Mobile Apps，頁面大小會控制從後端資料表一次提取的記錄數目。 然後對 `pull`資料的呼叫會根據此頁面大小將資料分批，直到沒有更多要提取的記錄為止。

您可以使用 **MSPullSettings** 設定頁面大小，如下所示。 預設頁面大小為 50，而下列範例將它變更為 3。

您可以基於效能的考量設定不同的頁面大小。 如果您有大量的小型資料記錄時，較高的分頁大小可減少伺服器的來回行程。

此設定只能控制用戶端上的頁面大小。 如果用戶端要求較 Mobile Apps 所支援更大的頁面大小，頁面大小會設為後端設定可支援的最大值。

此設定也是資料記錄的*數目*，而不是*位元組大小*。

如果您增加用戶端頁面大小，也應該增加伺服器上的頁面大小。 相關步驟請參閱[作法︰調整資料表分頁大小」](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

**Objective-C**：

```objc
  MSPullSettings *pullSettings = [[MSPullSettings alloc] initWithPageSize:3];
  [table  pullWithQuery:query queryId:@nil settings:pullSettings
                        completion:^(NSError * _Nullable error) {
                               if(error) {
                    NSLog(@"ERROR %@", error);
                }
                           }];
```

**Swift**：

```swift
let pullSettings = MSPullSettings(pageSize: 3)
table.pullWithQuery(query, queryId:nil, settings: pullSettings) { (error) in
    if let err = error {
        print("ERROR ", err)
    }
}
```

## <a name="inserting"></a>作法：插入資料

若要插入新的資料表資料列，請建立 `NSDictionary` 並叫用 `table insert`。 如果[動態結構描述]已啟用，Azure App Service 行動後端會根據 `NSDictionary` 自動產生新的資料欄。

如果未提供 `id` ，則後端會自動產生新的唯一識別碼。 提供您自己的 `id` ，以使用電子郵件地址、使用者名稱或您自己自訂的值作為識別碼。 提供您自己的識別碼可以讓聯結和商務導向的資料庫邏輯變得更容易。

`result` 含有先前插入的新項目。 視您的伺服器邏輯而定，相較於傳遞給伺服器的項目，它可能會含有其他或已修改的資料。

**Objective-C**：

```objc
NSDictionary *newItem = @{@"id": @"custom-id", @"text": @"my new item", @"complete" : @NO};
[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**：

```swift
let newItem = ["id": "custom-id", "text": "my new item", "complete": false]
table.insert(newItem) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

## <a name="modifying"></a>作法：修改資料

若要更新現有的資料列，請修改項目並呼叫 `update`：

**Objective-C**：

```objc
NSMutableDictionary *newItem = [oldItem mutableCopy]; // oldItem is NSDictionary
[newItem setValue:@"Updated text" forKey:@"text"];
[table update:newItem completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**：

```swift
if let newItem = oldItem.mutableCopy() as? NSMutableDictionary {
    newItem["text"] = "Updated text"
    table2.update(newItem as [NSObject: AnyObject], completion: { (result, error) -> Void in
        if let err = error {
            print("ERROR ", err)
        } else if let item = result {
            print("Todo Item: ", item["text"])
        }
    })
}
```

或者，提供資料列識別碼和更新的欄位：

**Objective-C**：

```objc
[table update:@{@"id":@"custom-id", @"text":"my EDITED item"} completion:^(NSDictionary *result, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item: %@", [result objectForKey:@"text"]);
    }
}];
```

**Swift**：

```swift
table.update(["id": "custom-id", "text": "my EDITED item"]) { (result, error) in
    if let err = error {
        print("ERROR ", err)
    } else if let item = result {
        print("Todo Item: ", item["text"])
    }
}
```

進行更新時，至少必須設定 `id` 屬性。

## <a name="deleting"></a>作法：刪除資料

若要刪除項目，請叫用 `delete` 搭配項目：

**Objective-C**：

```objc
[table delete:item completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];
```

**Swift**：

```swift
table.delete(newItem as [NSObject: AnyObject]) { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

或者，提供資料列識別碼來進行刪除：

**Objective-C**：

```objc
[table deleteWithId:@"37BBF396-11F0-4B39-85C8-B319C729AF6D" completion:^(id itemId, NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    } else {
        NSLog(@"Todo Item ID: %@", itemId);
    }
}];
```

**Swift**：

```swift
table.deleteWithId("37BBF396-11F0-4B39-85C8-B319C729AF6D") { (itemId, error) in
    if let err = error {
        print("ERROR ", err)
    } else {
        print("Todo Item ID: ", itemId)
    }
}
```

進行刪除時，至少必須設定 `id` 屬性。

## <a name="customapi"></a>如何：呼叫自訂 API

使用自訂 API，您可以公開任何後端功能。 它不必對應至資料表作業。 您不僅能進一步控制訊息，甚至還可以讀取或設定標頭，並變更回應內文格式。 若要了解如何在後端上建立自訂 API，請閱讀 [自訂 API](app-service-mobile-node-backend-how-to-use-server-sdk.md#work-easy-apis)

若要呼叫自訂 API，請呼叫 `MSClient.invokeAPI`。 要求和回應內容會被視為 JSON。 若要使用其他媒體類型，[請使用 `invokeAPI`] 的其他多載[5]。  若要進行 `GET` 要求而不是 `POST` 要求，請將參數 `HTTPMethod` 設為 `"GET"`，以及將參數 `body` 設為 `nil` (因為 GET 要求沒有訊息內文)。如果您的自訂 API 支援其他 HTTP 動詞命令，請適當地變更 `HTTPMethod`。

**Objective-C**：

```objc
[self.client invokeAPI:@"sendEmail"
                  body:@{ @"contents": @"Hello world!" }
            HTTPMethod:@"POST"
            parameters:@{ @"to": @"bill@contoso.com", @"subject" : @"Hi!" }
               headers:nil
            completion: ^(NSData *result, NSHTTPURLResponse *response, NSError *error) {
                if(error) {
                    NSLog(@"ERROR %@", error);
                } else {
                    // Do something with result
                }
            }];
```

**Swift**：

```swift
client.invokeAPI("sendEmail",
            body: [ "contents": "Hello World" ],
            HTTPMethod: "POST",
            parameters: [ "to": "bill@contoso.com", "subject" : "Hi!" ],
            headers: nil)
            {
                (result, response, error) -> Void in
                if let err = error {
                    print("ERROR ", err)
                } else if let res = result {
                          // Do something with result
                }
        }
```

## <a name="templates"></a>作法：註冊推送範本以傳送跨平台通知

若要註冊範本，請在用戶端應用程式中利用 **client.push registerDeviceToken** 方法傳遞範本。

**Objective-C**：

```objc
[client.push registerDeviceToken:deviceToken template:iOSTemplate completion:^(NSError *error) {
    if(error) {
        NSLog(@"ERROR %@", error);
    }
}];
```

**Swift**：

```swift
client.push?.registerDeviceToken(NSData(), template: iOSTemplate, completion: { (error) in
    if let err = error {
        print("ERROR ", err)
    }
})
```

您的範本類型為 NSDictionary，並且可能包含多個下列格式的範本：

**Objective-C**：

```objc
NSDictionary *iOSTemplate = @{ @"templateName": @{ @"body": @{ @"aps": @{ @"alert": @"$(message)" } } } };
```

**Swift**：

```swift
let iOSTemplate = ["templateName": ["body": ["aps": ["alert": "$(message)"]]]]
```

所有標記都將因安全性而移除。  若要將標籤新增至安裝中或安裝內的範本，請參閱[使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK][4]。  若要使用這些已註冊的範本來傳送通知，請使用[通知中樞 API][3]。

## <a name="errors"></a>作法：處理錯誤

呼叫 Azure App Service行動後端時，completion 區塊會包含 `NSError` 參數。 發生錯誤時，此參數便會傳回非 Nil。 您應檢查程式碼中的此參數，並視需要處理錯誤，如上述的程式碼片段所示。

檔案 [`<WindowsAzureMobileServices/MSError.h>`][6] 定義常數 `MSErrorResponseKey`、`MSErrorRequestKey` 和 `MSErrorServerItemKey`。 若要取得與錯誤相關的詳細資料︰

**Objective-C**：

```objc
NSDictionary *serverItem = [error.userInfo objectForKey:MSErrorServerItemKey];
```

**Swift**：

```swift
let serverItem = error.userInfo[MSErrorServerItemKey]
```

此外，檔案也定義每個錯誤代碼的常數：

**Objective-C**：

```objc
if (error.code == MSErrorPreconditionFailed) {
```

**Swift**：

```swift
if (error.code == MSErrorPreconditionFailed) {
```

## <a name="adal"></a>如何：使用 Active Directory Authentication Library 驗證使用者

您可以使用 Active Directory Authentication Library (ADAL)，利用 Azure Active Directory 將使用者登入應用程式。 相較於使用 `loginWithProvider:completion:` 方法，較建議使用身分識別提供者 SDK 的用戶端流程驗證。  用戶端流程驗證能提供較原生的 UX 風格，並允許進行其他自訂。

1. 依照[如何設定 App Service 來進行 Active Directory 登入][7]教學課程的說明，設定您的行動應用程式後端來進行 AAD 登入。 請務必完成註冊原生用戶端應用程式的選擇性步驟。 若是 iOS，我們建議採用 `<app-scheme>://<bundle-id>` 形式的重新導向 URI。 如需詳細資訊，請參閱 [ADAL iOS 快速入門][8]。
2. 使用 Cocoapods 安裝 ADAL。 編輯您的 Podfile 以納入下列定義，並以您的 Xcode 專案名稱取代 **YOUR-PROJECT** ：

        source 'https://github.com/CocoaPods/Specs.git'
        link_with ['YOUR-PROJECT']
        xcodeproj 'YOUR-PROJECT'

   以及 Pod：

        pod 'ADALiOS'

3. 使用終端機，從包含您專案的目錄執行 `pod install`，然後開啟產生的 Xcode 工作區 (而不是專案)。
4. 根據您使用的語言，將下列程式碼新增至您的應用程式。 取代每個程式碼的以下項目：

   * 以您佈建應用程式的租用戶名稱取代 **INSERT-AUTHORITY-HERE** 。 格式應為 https://login.microsoftonline.com/contoso.onmicrosoft.com。 您可以從 [Azure 入口網站]之 Azure Active Directory 的 [網域] 索引標籤中複製這個值。
   * 以您行動應用程式後端的用戶端識別碼取代 INSERT-RESOURCE-ID-HERE  。 您可以從入口網站 [Azure Active Directory 設定] 底下的 [進階] 索引標籤取得用戶端識別碼。
   * 以您從原生用戶端應用程式中複製的用戶端識別碼取代 INSERT-CLIENT-ID-HERE  。
   * 使用 HTTPS 配置，以您網站的 **/.auth/login/done** 端點取代 *INSERT-REDIRECT-URI-HERE* 。 此值應該類似 https://contoso.azurewebsites.net/.auth/login/done。

**Objective-C**：

```objc
#import <ADALiOS/ADAuthenticationContext.h>
#import <ADALiOS/ADAuthenticationSettings.h>
// ...
- (void) authenticate:(UIViewController*) parent
            completion:(void (^) (MSUser*, NSError*))completionBlock;
{
    NSString *authority = @"INSERT-AUTHORITY-HERE";
    NSString *resourceId = @"INSERT-RESOURCE-ID-HERE";
    NSString *clientId = @"INSERT-CLIENT-ID-HERE";
    NSURL *redirectUri = [[NSURL alloc]initWithString:@"INSERT-REDIRECT-URI-HERE"];
    ADAuthenticationError *error;
    ADAuthenticationContext *authContext = [ADAuthenticationContext authenticationContextWithAuthority:authority error:&error];
    authContext.parentController = parent;
    [ADAuthenticationSettings sharedInstance].enableFullScreen = YES;
    [authContext acquireTokenWithResource:resourceId
                                    clientId:clientId
                                redirectUri:redirectUri
                            completionBlock:^(ADAuthenticationResult *result) {
                                if (result.status != AD_SUCCEEDED)
                                {
                                    completionBlock(nil, result.error);;
                                }
                                else
                                {
                                    NSDictionary *payload = @{
                                                            @"access_token" : result.tokenCacheStoreItem.accessToken
                                                            };
                                    [client loginWithProvider:@"aad" token:payload completion:completionBlock];
                                }
                            }];
}
```

**Swift**：

```swift
// add the following imports to your bridging header:
//        #import <ADALiOS/ADAuthenticationContext.h>
//        #import <ADALiOS/ADAuthenticationSettings.h>

func authenticate(parent: UIViewController, completion: (MSUser?, NSError?) -> Void) {
    let authority = "INSERT-AUTHORITY-HERE"
    let resourceId = "INSERT-RESOURCE-ID-HERE"
    let clientId = "INSERT-CLIENT-ID-HERE"
    let redirectUri = NSURL(string: "INSERT-REDIRECT-URI-HERE")
    var error: AutoreleasingUnsafeMutablePointer<ADAuthenticationError?> = nil
    let authContext = ADAuthenticationContext(authority: authority, error: error)
    authContext.parentController = parent
    ADAuthenticationSettings.sharedInstance().enableFullScreen = true
    authContext.acquireTokenWithResource(resourceId, clientId: clientId, redirectUri: redirectUri) { (result) in
            if result.status != AD_SUCCEEDED {
                completion(nil, result.error)
            }
            else {
                let payload: [String: String] = ["access_token": result.tokenCacheStoreItem.accessToken]
                client.loginWithProvider("aad", token: payload, completion: completion)
            }
        }
}
```

## <a name="facebook-sdk"></a>作法：使用 Facebook SDK for iOS 來驗證使用者

您可以使用 Facebook SDK for iOS，利用 Facebook 將使用者登入應用程式。  相較於使用 `loginWithProvider:completion:` 方法，較建議使用用戶端流程驗證。  用戶端流程驗證能提供較原生的 UX 風格，並允許進行其他自訂。

1. 依照[如何設定 App Service 來進行 Facebook 登入][9]教學課程的說明，設定您的行動應用程式後端來進行 Facebook 登入。
2. 依照 [Facebook SDK for iOS - 開始使用][10]文件來安裝 Facebook SDK for iOS。 您可以在現有註冊中新增 iOS 平台，而不必建立應用程式。
3. Facebook 的文件包含應用程式委派中的某些 Objective-C 程式碼。 如果您要使用 **Swift**，您可以使用 AppDelegate.swift 的下列轉譯：

    ```swift
    // Add the following import to your bridging header:
    //        #import <FBSDKCoreKit/FBSDKCoreKit.h>

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) -> Bool {
        FBSDKApplicationDelegate.sharedInstance().application(application, didFinishLaunchingWithOptions: launchOptions)
        // Add any custom logic here.
        return true
    }

    func application(application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject?) -> Bool {
        let handled = FBSDKApplicationDelegate.sharedInstance().application(application, openURL: url, sourceApplication: sourceApplication, annotation: annotation)
        // Add any custom logic here.
        return handled
    }
    ```
4. 除了在專案中新增 `FBSDKCoreKit.framework`，也請以相同方式新增 `FBSDKLoginKit.framework` 的參考。
5. 根據您使用的語言，將下列程式碼新增至您的應用程式。

    **Objective-C**：

    ```objc
    #import <FBSDKLoginKit/FBSDKLoginKit.h>
    #import <FBSDKCoreKit/FBSDKAccessToken.h>
    // ...
    - (void) authenticate:(UIViewController*) parent
                completion:(void (^) (MSUser*, NSError*)) completionBlock;
    {
        FBSDKLoginManager *loginManager = [[FBSDKLoginManager alloc] init];
        [loginManager
            logInWithReadPermissions: @[@"public_profile"]
            fromViewController:parent
            handler:^(FBSDKLoginManagerLoginResult *result, NSError *error) {
                if (error) {
                    completionBlock(nil, error);
                } else if (result.isCancelled) {
                    completionBlock(nil, error);
                } else {
                    NSDictionary *payload = @{
                                            @"access_token":result.token.tokenString
                                            };
                    [client loginWithProvider:@"facebook" token:payload completion:completionBlock];
                }
            }];
    }
    ```

    **Swift**：

    ```swift
    // Add the following imports to your bridging header:
    //        #import <FBSDKLoginKit/FBSDKLoginKit.h>
    //        #import <FBSDKCoreKit/FBSDKAccessToken.h>

    func authenticate(parent: UIViewController, completion: (MSUser?, NSError?) -> Void) {
        let loginManager = FBSDKLoginManager()
        loginManager.logInWithReadPermissions(["public_profile"], fromViewController: parent) { (result, error) in
            if (error != nil) {
                completion(nil, error)
            }
            else if result.isCancelled {
                completion(nil, error)
            }
            else {
                let payload: [String: String] = ["access_token": result.token.tokenString]
                client.loginWithProvider("facebook", token: payload, completion: completion)
            }
        }
    }
    ```

## <a name="twitter-fabric"></a>作法：使用 Twitter Fabric for iOS 來驗證使用者

您可以使用 Fabric for iOS，利用 Twitter 將使用者登入應用程式。 與使用 `loginWithProvider:completion:` 方法相比，較建議使用用戶端流程驗證，因為它提供更原生的 UX 風格，並可允許進行其他自訂。

1. 依照 [如何設定 App Service 來進行 Twitter 登入](../app-service/app-service-mobile-how-to-configure-twitter-authentication.md) 教學課程的說明，設定您的行動應用程式後端來進行 Twitter 登入。
2. 依照 [Fabric for iOS - 開始使用]文件並設定 TwitterKit，在專案中新增網狀架構。

   > [!NOTE]
   > 根據預設，網狀架構會為您建立 Twitter 應用程式。 您可以使用下列程式碼片段，註冊您稍早所建立的取用者金鑰和取用者密碼，以避免建立應用程式。    或者，您可以使用您在 [網狀架構儀表板]中看到的值，取代您提供給 App Service 的取用者金鑰和取用者密碼值。 如果您選擇此選項，請務必將回呼 URL 設定為預留位置值，例如 `https://<yoursitename>.azurewebsites.net/.auth/login/twitter/callback`。

    如果您選擇使用稍早所建立的密碼，請在應用程式委派中新增下列程式碼︰

    **Objective-C**：

    ```objc
    #import <Fabric/Fabric.h>
    #import <TwitterKit/TwitterKit.h>
    // ...
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        [[Twitter sharedInstance] startWithConsumerKey:@"your_key" consumerSecret:@"your_secret"];
        [Fabric with:@[[Twitter class]]];
        // Add any custom logic here.
        return YES;
    }
    ```

    **Swift**：

    ```swift
    import Fabric
    import TwitterKit
    // ...
    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject : AnyObject]?) -> Bool {
        Twitter.sharedInstance().startWithConsumerKey("your_key", consumerSecret: "your_secret")
        Fabric.with([Twitter.self])
        // Add any custom logic here.
        return true
    }
    ```

3. 根據您使用的語言，將下列程式碼新增至您的應用程式。

    **Objective-C**：

    ```objc
    #import <TwitterKit/TwitterKit.h>
    // ...
    - (void)authenticate:(UIViewController*)parent completion:(void (^) (MSUser*, NSError*))completionBlock
    {
        [[Twitter sharedInstance] logInWithCompletion:^(TWTRSession *session, NSError *error) {
            if (session) {
                NSDictionary *payload = @{
                                            @"access_token":session.authToken,
                                            @"access_token_secret":session.authTokenSecret
                                        };
                [client loginWithProvider:@"twitter" token:payload completion:completionBlock];
            } else {
                completionBlock(nil, error);
            }
        }];
    }
    ```

    **Swift**：

    ```swift
    import TwitterKit
    // ...
    func authenticate(parent: UIViewController, completion: (MSUser?, NSError?) -> Void) {
        let client = self.table!.client
        Twitter.sharedInstance().logInWithCompletion { session, error in
            if (session != nil) {
                let payload: [String: String] = ["access_token": session!.authToken, "access_token_secret": session!.authTokenSecret]
                client.loginWithProvider("twitter", token: payload, completion: completion)
            } else {
                completion(nil, error)
            }
        }
    }
    ```

## <a name="google-sdk"></a>作法：使用 Google Sign-In SDK for iOS 來驗證使用者

您可以使用 Google Sign-In SDK for iOS，利用 Google 帳戶將使用者登入應用程式。  近期內，Google 宣布他們的 OAuth 安全性原則變更。  這些原則變更要求您未來必須使用 Google SDK。

1. 依照 [如何設定 App Service 來進行 Google 登入](../app-service/app-service-mobile-how-to-configure-google-authentication.md) 教學課程的說明，設定您的行動應用程式後端來進行 Google 登入。
2. 請依照 [Google Sign-In for iOS - Start integrating](https://developers.google.com/identity/sign-in/ios/start-integrating) 文件安裝 Google SDK for iOS。 您可以略過＜使用後端伺服器進行驗證＞一節。
3. 請根據您使用的語言，將下列內容新增到委派的 `signIn:didSignInForUser:withError:` 方法。

    **Objective-C**：
    ```objc
    NSDictionary *payload = @{
                                @"id_token":user.authentication.idToken,
                                @"authorization_code":user.serverAuthCode
                                };

    [client loginWithProvider:@"google" token:payload completion:^(MSUser *user, NSError *error) {
        // ...
    }];
    ```

    **Swift**：

    ```swift
    let payload: [String: String] = ["id_token": user.authentication.idToken, "authorization_code": user.serverAuthCode]
    client.loginWithProvider("google", token: payload) { (user, error) in
        // ...
    }
    ```

4. 務必也將下列內容新增到應用程式委派中的 `application:didFinishLaunchingWithOptions:`，將 "SERVER_CLIENT_ID" 取代為您用來在步驟 1 中設定 App Service 的相同識別碼。

    **Objective-C**：

    ```objc
    [GIDSignIn sharedInstance].serverClientID = @"SERVER_CLIENT_ID";
    ```

     **Swift**：

    ```swift
    GIDSignIn.sharedInstance().serverClientID = "SERVER_CLIENT_ID"
    ```

5. 根據您所使用的語言，將下列程式碼新增到應用程式的 UIViewController 中以實作 `GIDSignInUIDelegate` 通訊協定。  系統會先將您登出，然後再將您登入；雖然不需要再次輸入認證，不過您會看到同意對話方塊。  請只在工作階段權杖過期時才呼叫這個方法。

   **Objective-C**：

    ```objc
    #import <Google/SignIn.h>
    // ...
    - (void)authenticate
    {
            [GIDSignIn sharedInstance].uiDelegate = self;
            [[GIDSignIn sharedInstance] signOut];
            [[GIDSignIn sharedInstance] signIn];
    }
    ```

   **Swift**：

    ```swift
    // ...
    func authenticate() {
        GIDSignIn.sharedInstance().uiDelegate = self
        GIDSignIn.sharedInstance().signOut()
        GIDSignIn.sharedInstance().signIn()
    }
    ```

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[Setup and Prerequisites]: #Setup
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #table-reference
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Bind data to the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching-tokens
[How to: Upload images and large files]: #blobs
[How to: Handle errors]: #errors
[How to: Design unit tests]: #unit-testing
[How to: Customize the client]: #customizing
[Customize request headers]: #custom-headers
[Customize data type serialization]: #custom-serialization
[Next Steps]: #next-steps
[How to: Use MSQuery]: #query-object

<!-- Images. -->

<!-- URLs. -->
[Azure Mobile Apps 快速啟動]: app-service-mobile-ios-get-started.md

[Add Mobile Services to Existing App]: /develop/mobile/tutorials/get-started-data
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-ios
[Validate and modify data in Mobile Services by using server scripts]: /develop/mobile/tutorials/validate-modify-and-augment-data-ios
[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[iOS SDK]: https://developer.apple.com/xcode
[Azure 入口網站]: https://portal.azure.com/
[Handling Expired Tokens]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[Permissions]: http://msdn.microsoft.com/library/windowsazure/jj193161.aspx
[Service-side Authorization]: mobile-services-javascript-backend-service-side-authorization.md
[Use scripts to authorize users]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[動態結構描述]: http://go.microsoft.com/fwlink/p/?LinkId=296271
[How to: access custom parameters]: /develop/mobile/how-to-guides/work-with-server-scripts#access-headers
[Create a table]: http://msdn.microsoft.com/library/windowsazure/jj193162.aspx
[NSDictionary object]: http://go.microsoft.com/fwlink/p/?LinkId=301965
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[CLI to manage Mobile Services tables]: /cli/azure/get-started-with-az-cli2
[Conflict-Handler]: mobile-services-ios-handling-conflicts-offline-data.md#add-conflict-handling

[網狀架構儀表板]: https://www.fabric.io/home
[Fabric for iOS - 開始使用]: https://docs.fabric.io/ios/fabric/getting-started.html
[1]: https://github.com/Azure/azure-mobile-apps-ios-client/blob/master/README.md#ios-client-sdk
[2]: http://azure.github.io/azure-mobile-apps-ios-client/
[3]: https://msdn.microsoft.com/library/azure/dn495101.aspx
[4]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#tags
[5]: http://azure.github.io/azure-mobile-services/iOS/v3/Classes/MSClient.html#//api/name/invokeAPI:data:HTTPMethod:parameters:headers:completion:
[6]: https://github.com/Azure/azure-mobile-services/blob/master/sdk/iOS/src/MSError.h
[7]: ../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md
[8]:../active-directory/develop/quickstart-v1-ios.md
[9]: ../app-service/app-service-mobile-how-to-configure-facebook-authentication.md
[10]: https://developers.facebook.com/docs/ios/getting-started
