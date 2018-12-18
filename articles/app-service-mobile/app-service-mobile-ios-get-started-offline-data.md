---
title: 啟用 iOS Mobile Apps 的離線同步處理 | Microsoft Docs
description: 了解如何使用 Azure App Service Mobile Apps 來快取及同步處理 iOS 應用程式中的離線資料。
documentationcenter: ios
author: conceptdev
manager: crdun
editor: ''
services: app-service\mobile
ms.assetid: eb5b9520-0f39-4a09-940a-dadb6d940db8
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 2f415f1886c654f3bdd880cdccaadc7aa3e69892
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32156490"
---
# <a name="enable-offline-syncing-with-ios-mobile-apps"></a>啟用 iOS Mobile Apps 的離線同步處理
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## <a name="overview"></a>概觀
本教學課程涵蓋適用於 iOS 的 Azure App Service Mobile Apps 功能的離線同步處理說明。 透過離線同步處理，終端使用者即使沒有網路連線，也可以和行動裝置 App 互動以檢視、新增、修改資料。 變更會儲存在本機資料庫中。 裝置恢復上線後，這些變更就會與遠端後端進行同步處理。

如果這是您第一次接觸 Mobile Apps，請先完成[建立 iOS 應用程式] 教學課程。 如果您不使用下載的快速入門伺服器專案，則必須將資料存取擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

若要深入了解離線同步處理功能，請參閱 [Mobile Apps 中的離線資料同步處理]。

## <a name="review-sync"></a>檢閱用戶端同步程式碼
您針對[建立 iOS 應用程式]教學課程下載的用戶端專案，已經包含了使用本機核心資料式資料庫支援離線同步處理的程式碼。 本節將摘要說明已包含在教學課程程式碼中的內容。 如需此功能的概念性概觀，請參閱 [Mobile Apps 中的離線資料同步處理]。

Mobile Apps 的離線資料同步處理功能可讓終端使用者在無法存取網路時，仍可與本機資料庫互動。 若要在您的應用程式中使用這些功能，您可初始化 `MSClient` 的同步處理內容以及參考本機存放區。 然後透過 **MSSyncTable** 介面參考您的資料表。

在 **QSTodoService.m** (Objective-C) 或 **ToDoTableViewController.swift** (Swift) 中，注意到成員 **syncTable** 的類型為 **MSSyncTable**。 離線同步處理會使用此同步處理資料表介面，而不是 **MSTable**。 使用同步處理資料表時，所有作業都會移至本機存放區，而且只會與具有明確推送和提取作業的遠端後端同步處理。

 若要取得同步處理資料表的參考，請在 `MSClient` 上使用 **syncTableWithName** 方法。 若要移除離線同步處理功能，請改用 **tableWithName**。

必須先初始化本機存放區，才可以執行資料表作業。 以下是相關的程式碼：

* **Objective-C**。 在 **QSTodoService.init** 方法中：

   ```objc
   MSCoreDataStore *store = [[MSCoreDataStore alloc] initWithManagedObjectContext:context];
   self.client.syncContext = [[MSSyncContext alloc] initWithDelegate:nil dataSource:store callback:nil];
   ```    
* **Swift**。 在 **ToDoTableViewController.viewDidLoad** 方法中︰

   ```swift
   let client = MSClient(applicationURLString: "http:// ...") // URI of the Mobile App
   let managedObjectContext = (UIApplication.sharedApplication().delegate as! AppDelegate).managedObjectContext!
   self.store = MSCoreDataStore(managedObjectContext: managedObjectContext)
   client.syncContext = MSSyncContext(delegate: nil, dataSource: self.store, callback: nil)
   ```
   這方法會以 Mobile Apps SDK 提供的 `MSCoreDataStore` 介面建立一個本機存放區。 或者，您也可以實作 `MSSyncContextDataSource` 通訊協定，改為提供不同的本機存放區。 此外，**MSSyncContext** 的第一個參數是用來指定衝突處理常式。 因為已傳遞 `nil`，所以我們會取得預設衝突處理常式，該處理常式在任何衝突時都會失敗。

現在，讓我們執行實際的同步處理作業，並從遠端後端取得資料：

* **Objective-C**。 `syncData` 先推送新的變更，然後呼叫 **pullData** 以從遠端後端取得資料。 最後，**pullData** 方法會取得符合查詢的新資料︰

   ```objc
   -(void)syncData:(QSCompletionBlock)completion
   {
       // Push all changes in the sync context, and then pull new data.
       [self.client.syncContext pushWithCompletion:^(NSError *error) {
           [self logErrorIfNotNil:error];
           [self pullData:completion];
       }];
   }

   -(void)pullData:(QSCompletionBlock)completion
   {
       MSQuery *query = [self.syncTable query];

       // Pulls data from the remote server into the local table.
       // We're pulling all items and filtering in the view.
       // Query ID is used for incremental sync.
       [self.syncTable pullWithQuery:query queryId:@"allTodoItems" completion:^(NSError *error) {
           [self logErrorIfNotNil:error];

           // Lets the caller know that we have finished.
           if (completion != nil) {
               dispatch_async(dispatch_get_main_queue(), completion);
           }
       }];
   }
   ```
* **Swift**：
   ```swift
   func onRefresh(sender: UIRefreshControl!) {
      UIApplication.sharedApplication().networkActivityIndicatorVisible = true

      self.table!.pullWithQuery(self.table?.query(), queryId: "AllRecords") {
          (error) -> Void in

          UIApplication.sharedApplication().networkActivityIndicatorVisible = false

          if error != nil {
              // A real application would handle various errors like network conditions,
              // server conflicts, etc via the MSSyncContextDelegate
              print("Error: \(error!.description)")

              // We will discard our changes and keep the server's copy for simplicity
              if let opErrors = error!.userInfo[MSErrorPushResultKey] as? Array<MSTableOperationError> {
                  for opError in opErrors {
                      print("Attempted operation to item \(opError.itemId)")
                      if (opError.operation == .Insert || opError.operation == .Delete) {
                          print("Insert/Delete, failed discarding changes")
                          opError.cancelOperationAndDiscardItemWithCompletion(nil)
                      } else {
                          print("Update failed, reverting to server's copy")
                          opError.cancelOperationAndUpdateItem(opError.serverItem!, completion: nil)
                      }
                  }
              }
          }
          self.refreshControl?.endRefreshing()
      }
   }
   ```

在 Objective-C 版本中，於 `syncData`，我們會先在同步處理內容上呼叫 **pushWithCompletion**。 此方法是 `MSSyncContext` 的成員 (而非同步處理資料表本身)，因為它會將變更推送至所有資料表。 只有以某種方式在本機上修改過的記錄 (透過 CUD 作業)，才會傳送至伺服器。 接著會呼叫 **pullData** 協助程式，該程式會呼叫 **MSSyncTable.pullWithQuery** 來擷取遠端資料並存放在本機資料庫中。

在 Swift 版本中，因為推送作業不是絕對必要，所以並沒有呼叫 **pushWithCompletion**。 如果同步處理內容中正在進行推送作業的資料表有任何變更擱置，則提取一律會先發出推送。 不過，如果您有一個以上的同步處理資料表，最好能明確呼叫推送，以確保所有的相關資料表都能一致。

在 Objective-C 與 Swift 版本中，您可以使用 **pullWithQuery** 方法來指定查詢，以篩選想要擷取的記錄。 在此範例中，查詢會擷取遠端 `TodoItem` 資料表中的所有記錄。

**pullWithQuery** 的第二個參數是用於「增量同步處理」的查詢識別碼。增量同步處理會使用記錄的 `UpdatedAt` 時間戳記 (在本機存放區中稱為 `updatedAt`)，僅取出自上次同步處理後修改的記錄。對您應用程式中的每個邏輯查詢而言，查詢識別碼應該是唯一的描述性字串。 若選擇不要增量同步處理，請傳遞 `nil` 做為查詢識別碼。 此方法可能效率不佳，因為它會在每次提取作業擷取所有記錄。

當您修改或新增資料、使用者執行重新整理動作及啟動時，Objective-C 應用程式就會同步處理。

當使用者執行重新整理動作及啟動時，Swift 應用程式就會同步處理。

因為每當資料修改 (Objective-C) 或 App 啟動 (Objective-C 和 Swift) 時，App 就會同步處理，故 App 假設使用者在線上。 在後續章節中，您將會更新 App，讓使用者即使是離線狀態也能進行編輯。

## <a name="review-core-data"></a>檢閱核心資料模型
在使用「核心資料離線」存放區時，您必須在資料模型中定義特定資料表和欄位。 範例應用程式已經包含具有正確格式的資料模型。 在這一節中，我們會逐步介紹這些資料表並示範其使用方式。

開啟 **QSDataModel.xcdatamodeld**。 已定義四個資料表--其中三個由 SDK 使用，而一個是用於 To-do 項目本身：
  * MS_TableOperations：追蹤需要與伺服器同步的項目。
  * MS_TableOperationErrors：追蹤在離線同步處理期間發生的任何錯誤。
  * MS_TableConfig：追蹤所有提取作業的最後一次同步處理作業的上次更新時間。
  * TodoItem：儲存 To-do 項目。 系統資料行 **createdAt**、**updatedAt** 和 **version** 為選擇性系統屬性。

> [!NOTE]
> Mobile Apps SDK 會保留以 "**``**" 為開頭的資料行名稱。 請不要將此前置詞用於系統資料行以外的項目。 否則，當您使用遠端後端時，系統會修改您的資料行名稱。
>
>

當您使用離線同步處理功能時，請定義三個系統資料表，以及資料資料表。

### <a name="system-tables"></a>系統資料表

**MS_TableOperations**  

![MS_TableOperations 資料表屬性][defining-core-data-tableoperations-entity]

| 屬性 | 類型 |
| --- | --- |
| id | 整數 64 |
| itemId | 字串 |
| properties | 二進位資料 |
| 資料表 | 字串 |
| tableKind | 整數 16 |


**MS_TableOperationErrors**

 ![MS_TableOperationErrors 資料表屬性][defining-core-data-tableoperationerrors-entity]

| 屬性 | 類型 |
| --- | --- |
| id |字串 |
| operationId |整數 64 |
| properties |二進位資料 |
| tableKind |整數 16 |

 **MS_TableConfig**

 ![][defining-core-data-tableconfig-entity]

| 屬性 | 類型 |
| --- | --- |
| id |字串 |
| 索引鍵 |字串 |
| keyType |整數 64 |
| 資料表 |字串 |
| value |字串 |

### <a name="data-table"></a>資料表

**TodoItem**

| 屬性 | 類型 | 附註 |
| --- | --- | --- |
| id | 字串 (標示為必要) |遠端存放區中的主索引鍵 |
| 完成 | BOOLEAN | To-do 項目欄位 |
| text |字串 |To-do 項目欄位 |
| 建立時間 | 日期 | (選擇性) 對應至 **createdAt** 系統屬性 |
| 更新時間 | 日期 | (選擇性) 對應至 **updatedAt** 系統屬性 |
| version | 字串 | (選擇性) 用來偵測衝突，對應至版本 |

## <a name="setup-sync"></a>變更應用程式的同步處理行為
在本節中，您將修改 App，使它在啟動或有使用者插入並更新項目時不會同步處理。 只有在執行重新整理動作按鈕時，它才會同步。

**Objective-C**：

1. 在 **QSTodoListViewController.m** 中，變更 **viewDidLoad** 方法以移除在方法結尾的 `[self refresh]` 呼叫。 資料現在已經不會在 App 啟動時同步。 反之，它是與本機存放區的內容同步。
2. 在 **QSTodoService.m** 中，修改 `addItem` 的定義，使其不會在插入項目後同步處理。 移除 `self syncData` 區塊並以下列項目取代：

   ```objc
   if (completion != nil) {
       dispatch_async(dispatch_get_main_queue(), completion);
   }
   ```
3. 如先前所述修改 `completeItem` 的定義。 移除 `self syncData` 的區塊並以下列項目取代：
   ```objc
   if (completion != nil) {
       dispatch_async(dispatch_get_main_queue(), completion);
   }
   ```

**Swift**：

在 **ToDoTableViewController.swift** 的 `viewDidLoad` 中，註解化這兩行以停止在 App 啟動時同步處理。 在本文撰寫期間，當某人新增或完成項目時，Swift Todo App 不會更新服務。 只有在 App 啟動時它才會更新。

   ```swift
  self.refreshControl?.beginRefreshing()
  self.onRefresh(self.refreshControl)
```

## <a name="test-app"></a>測試應用程式
在本節中，您將連接至無效的 URL，以模擬離線情況。 當您新增資料項目時，這些項目會存放在本機核心資料存放區，但不會同步到行動裝置 App 後端。

1. 將 **QSTodoService.m** 中的行動裝置 App URL 變更為無效的 URL，然後再次執行該 App：

   **Objective-C**。 在 QSTodoService.m 中：
   ```objc
   self.client = [MSClient clientWithApplicationURLString:@"https://sitename.azurewebsites.net.fail"];
   ```
   **Swift**。 在 ToDoTableViewController.swift 中：
   ```swift
   let client = MSClient(applicationURLString: "https://sitename.azurewebsites.net.fail")
   ```
2. 新增一些 To-do 項目。 結束模擬器 (或強制關閉 App)，然後重新啟動它。 確認已保存您的變更。

3. 檢視遠端 **TodoItem** 資料表的內容：
   * 針對 Node.js 後端，請移至 [Azure 入口網站](https://portal.azure.com/)，在您的行動裝置 App 後端中按一下 [簡易表]  >  [TodoItem]。  
   * 針對 .NET 後端，請使用 SQL 工具 (例如 SQL Server Management Studio) 或 REST 用戶端 (例如 Fiddler 或 Postman)。  

4. 請確認新項目「尚未」同步處理到伺服器。

5. 請將 **QSTodoService.m** 中的 URL 變更回正確的 URL，然後重新執行 App。

6. 將項目清單往下拉，執行重新整理動作。  
會出現旋轉進度指示器。

7. 再次檢視 **TodoItem** 資料。 其中會顯示已變更的新 To-do 項目。

## <a name="summary"></a>總結
為了支援離線同步處理功能，我們使用了 `MSSyncTable` 介面，並對本機存放區初始化 `MSClient.syncContext`。 在此案例中，本機存放區是以核心資料為基礎的資料庫。

使用核心資料本機存放區時，您必須使用[正確的系統屬性](#review-core-data)定義數個資料表。

針對行動裝置 App 的建立、讀取、更新及刪除 (CRUD) 等一般作業，會以 App 有如處於連線狀態之下執行，但所有的作業都是對本機存放區執行。

我們使用 **MSSyncTable.pullWithQuery** 方法來同步處理本機存放區與伺服器。

## <a name="additional-resources"></a>其他資源
* [Mobile Apps 中的離線資料同步處理]
* [雲端報導：Azure Mobile Services 中的離線同步處理] \(雖然影片是關於 Mobile Services，但 Mobile Apps 也是以類似的方式進行離線同步處理。\)

<!-- URLs. -->


[建立 iOS 應用程式]: app-service-mobile-ios-get-started.md
[Mobile Apps 中的離線資料同步處理]: app-service-mobile-offline-data-sync.md

[defining-core-data-tableoperationerrors-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperationerrors-entity.png
[defining-core-data-tableoperations-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableoperations-entity.png
[defining-core-data-tableconfig-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-tableconfig-entity.png
[defining-core-data-todoitem-entity]: ./media/app-service-mobile-ios-get-started-offline-data/defining-core-data-todoitem-entity.png

[雲端報導：Azure Mobile Services 中的離線同步處理]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Azure Friday: Offline-enabled apps in Azure Mobile Services]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/
