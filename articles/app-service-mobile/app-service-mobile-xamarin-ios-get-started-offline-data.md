---
title: 啟用 Azure 行動應用程式 (Xamarin iOS) 的離線同步處理
description: 了解如何在 Xamarin iOS 應用程式中使用應用程式服務行動應用程式快取和同步離線資料
documentationcenter: xamarin
author: conceptdev
manager: cfowler
editor: ''
services: app-service\mobile
ms.assetid: 828a287c-5d58-4540-9527-1309ebb0f32b
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 287977d55656a4b2bc42d90730d16f522fae9b9e
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27594661"
---
# <a name="enable-offline-sync-for-your-xamarinios-mobile-app"></a>啟用 Xamarin iOS 行動應用程式的離線同步處理
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## <a name="overview"></a>概觀
此教學課程介紹適用於 Xamarin.iOS 之 Azure 行動應用程式的離線同步處理功能。 離線同步處理可讓使用者與行動應用程式進行互動--檢視、新增或修改資料--即使沒有網路連線進也可行。 變更會儲存在本機資料庫中。 裝置恢復上線後，這些變更就會與遠端服務進行同步處理。

在本教學課程中，更新[建立 Xamarin iOS 應用程式]中的 Xamarin.iOS 應用程式專案，來支援 Azure Mobile Apps 的離線功能。 如果您不要使用下載的快速入門伺服器專案，必須將資料存取擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

若要深入了解離線同步處理功能，請參閱 [Azure 行動應用程式中的離線資料同步處理]主題。

## <a name="update-the-client-app-to-support-offline-features"></a>更新用戶端應用程式以支援離線功能
Azure 行動應用程式的離線功能可讓您在離線狀態時，仍可與本機資料庫互動。 若要在您的應用程式中使用這些功能，您必須將 [SyncContext] 初始化至本機存放區。 透過 [IMobileServiceSyncTable] 介面參考您的資料表。 SQLite 可做為裝置上的本機存放區。

1. 在您於[建立 Xamarin iOS 應用程式]教學課程所完成的專案中，開啟 NuGet 封裝管理員，然後搜尋並安裝 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 套件。
2. 開啟 QSTodoService.cs 檔案，並取消註解 `#define OFFLINE_SYNC_ENABLED` 定義。
3. 重建並執行用戶端應用程式。 應用程式的運作方式和啟用離線同步處理之前的方式相同。不過，本機資料庫現在會填入可在離線案例下使用的資料。

## <a name="update-sync"></a>更新應用程式以使其與後端中斷連線
在本節中，您將中斷與行動應用程式後端的連線，以模擬離線狀態。 當您新增資料項目時，您的例外狀況處理常式會指出應用程式處於離線模式。 處於此狀態時，新項目會新增到本機存放區，並且會在推送接下來於連線狀態執行時，同步處理至行動應用程式後端。

1. 編輯共用專案中的 QSToDoService.cs。 變更 **applicationURL** 以指向無效的 URL：

         const string applicationURL = @"https://your-service.azurewebsites.fail";

    您也可以透過停用裝置的 WiFi 和行動電話通訊網路，或使用飛航模式來示範離線行為。
2. 建置並執行應用程式。 請注意，應用程式啟動後同步處理無法重新整理。
3. 輸入新項目，並注意每次您按一下 [儲存]  時，推送都會失敗並具有 [CancelledByNetworkError] 狀態。 不過，新的 todo 項目在可推送至行動應用程式後端之前，都會存留在本機存放區中。  在生產應用程式中，如果您隱藏這些例外狀況，用戶端應用程式的行為會如同它仍連線到行動應用程式後端。
4. 關閉應用程式並重新加以開啟，以驗證您所建立的新項目持續存留於本機存放區中。
5. (選擇性) 如果您已在電腦上安裝 Visual Studio，請開啟 [伺服器總管]。 瀏覽至 [Azure]-> [SQL Database 中您的資料庫]。 在資料庫上按一下滑鼠右鍵，並選取 [在 SQL Server 物件總管中開啟] 。 現在您可以瀏覽至您的 SQL Database 資料表和其內容。 確認後端資料庫中的資料沒有變更。
6. (選擇性) 使用 REST 工具 (例如 Fiddler 或 Postman) 來查詢您的行動後端 (使用表單 `https://<your-mobile-app-backend-name>.azurewebsites.net/tables/TodoItem`中的 GET 查詢)。

## <a name="update-online-app"></a>更新應用程式以重新連線您的行動應用程式後端
在本節中，您會將應用程式重新連接至行動應用程式後端。 您將藉此模擬應用程式在行動應用程式後端中，從離線狀態恢復為線上狀態的情境。   如果您關閉網路連線來模擬網路中斷情況，則不需要變更任何程式碼。
重新開啟網路。  第一次執行應用程式時會呼叫 `RefreshDataAsync` 方法。 這會進而呼叫 `SyncAsync` 來同步處理您的本機存放區與後端資料庫。

1. 在共用專案中，開啟 QSToDoService.cs，並還原您對 **applicationURL** 屬性所做的變更。
2. 重新建置並執行應用程式。 該應用程式會在 `OnRefreshItemsSelected` 方法執行時使用推送與提取作業，同步處理您的本機變更與 Azure 行動應用程式後端。
3. (選擇性) 使用 SQL Server 物件總管或 REST 工具 (例如 Fiddler) 檢視已更新的資料。 請注意，Azure 行動應用程式後端資料庫與本機存放區之間的資料已同步處理。
4. 在應用程式中，按一下幾個項目旁邊的核取方塊，以在本機存放區中完成它們。

   `CompleteItemAsync` 會呼叫 `SyncAsync`，以便與行動應用程式後端同步處理每個已完成的項目。 `SyncAsync` 會同時呼叫推送與提取。
   **每當您針對用戶端進行變更的資料表執行提取時，用戶端同步處理內容上的推送一律會先自動執行**。 隱含推送可確保本機存放區中的所有資料表和關聯性都保持一致。 如需此行為的詳細資訊，請參閱 [Azure 行動應用程式中的離線資料同步處理]。

## <a name="review-the-client-sync-code"></a>檢閱用戶端同步處理程式碼
您完成 [建立 Xamarin iOS 應用程式] 教學課程時所下載的 Xamarin 用戶端專案，已經包含了使用本機 SQLite 資料庫支援離線同步處理的程式碼。 以下是已經包含在教學課程程式碼中之內容的簡要概觀。 如需此功能的概念性概觀，請參閱 [Azure 行動應用程式中的離線資料同步處理]。

* 必須先初始化本機存放區，才可以執行資料表作業。 當 `QSTodoListViewController.ViewDidLoad()` 執行 `QSTodoService.InitializeStoreAsync()` 時會初始化本機存放區資料庫。 這個方法會使用 Azure 行動應用程式用戶端 SDK 所提供的 `MobileServiceSQLiteStore` 類別，建立新的本機 SQLite 資料庫。

    `DefineTable` 方法會在本機存放區中建立與給定類型中的欄位相符的資料表，在此案例中為 `ToDoItem`。 該類型不必包含遠端資料庫中的所有資料行。 可以只儲存資料行的子集。

        // QSTodoService.cs

        public async Task InitializeStoreAsync()
        {
            var store = new MobileServiceSQLiteStore(localDbPath);
            store.DefineTable<ToDoItem>();

            // Uses the default conflict handler, which fails on conflict
            await client.SyncContext.InitializeAsync(store);
        }
* `QSTodoService` 的 `todoTable` 成員屬於 `IMobileServiceSyncTable` 類型而非 `IMobileServiceTable`。 IMobileServiceSyncTable 會將所有建立、讀取、更新和刪除 (CRUD) 資料表作業導向至本機存放區資料庫。

    您可以呼叫 `IMobileServiceSyncContext.PushAsync()`，以決定何時將那些變更推送至 Azure 行動應用程式後端。 當呼叫 `PushAsync` 時，透過追蹤和推送所有用戶端應用程式修改之資料表中的變更，同步處理內容可協助保存資料表關聯性。

    每當重新整理 todoitem 清單或是已新增或完成 todoitem 時，提供的程式碼會呼叫 `QSTodoService.SyncAsync()` 來進行同步處理。 每次本機變更之後，應用程式會同步處理。 如果針對具有內容追蹤之擱置中本機更新的資料表執行提取，該提取作業會先自動觸發內容推送。

    在提供的程式碼中，遠端 `TodoItem` 資料表中的所有記錄都會進行查詢，但是也可能透過將查詢識別碼與查詢傳遞至 `PushAsync` 來篩選記錄。 如需詳細資訊，請參閱 [Azure 行動應用程式中的離線資料同步處理]中的*增量同步處理*一節。

        // QSTodoService.cs
        public async Task SyncAsync()
        {
            try
            {
                await client.SyncContext.PushAsync();
                await todoTable.PullAsync("allTodoItems", todoTable.CreateQuery()); // query ID is used for incremental sync
            }

            catch (MobileServiceInvalidOperationException e)
            {
                Console.Error.WriteLine(@"Sync Failed: {0}", e.Message);
            }
        }

## <a name="additional-resources"></a>其他資源
* [Azure 行動應用程式中的離線資料同步處理]
* [Azure Mobile Apps .NET SDK HOWTO][8]

<!-- Images -->

<!-- URLs. -->
[建立 Xamarin iOS 應用程式]: app-service-mobile-xamarin-ios-get-started.md
[Azure 行動應用程式中的離線資料同步處理]: app-service-mobile-offline-data-sync.md
[SyncContext]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.synccontext(v=azure.10).aspx
[8]: app-service-mobile-dotnet-how-to-use-client-library.md
