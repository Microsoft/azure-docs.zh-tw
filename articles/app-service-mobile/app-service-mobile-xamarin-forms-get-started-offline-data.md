---
title: 啟用 Azure 行動應用程式的離線同步處理 (Xamarin.Forms) | Microsoft Docs
description: 了解如何在 Xamarin.Forms 應用程式中使用 App Service 行動應用程式快取和同步離線資料
documentationcenter: xamarin
author: conceptdev
manager: yochayk
editor: ''
services: app-service\mobile
ms.assetid: acf0f874-3ea5-4410-bd22-b0e72140f3b5
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: article
ms.date: 10/04/2016
ms.author: crdun
ms.openlocfilehash: f88e6a4037bcca54982359742cdc6021f020882d
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27594712"
---
# <a name="enable-offline-sync-for-your-xamarinforms-mobile-app"></a>啟用 Xamarin.Forms 行動應用程式的離線同步處理
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## <a name="overview"></a>概觀
此教學課程介紹適用於 Xamarin.Forms 之 Azure 行Mobile Apps 的離線同步處理功能。 離線同步處理可讓使用者與行動應用程式進行互動--檢視、新增或修改資料--即使沒有網路連線進也可行。 變更會儲存在本機資料庫中。 裝置恢復上線後，這些變更就會與遠端服務進行同步處理。

本教學課程是根據您完成教學課程 [建立 Xamarin iOS 應用程式] 時所建立之 Mobile Apps 的 Xamarin.Forms 快速入門方案。 Xamarin.Forms 的快速入門方案包含程式碼來支援離線同步處理，只需要啟用即可。 在本教學課程中，您將會更新快速入門方案來開啟 Azure Mobile Apps 的離線功能。 我們也會在應用程式中反白顯示離線特有的程式碼。 如果您不要使用下載的快速入門方案，則必須將資料存取擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱[使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK][1]。

若要深入了解離線同步處理功能，請參閱 [Azure Mobile Apps 中的離線資料同步處理][2]主題。

## <a name="enable-offline-sync-functionality-in-the-quickstart-solution"></a>啟用快速入門方案中的離線同步處理功能
專案中使用 C# 前置處理器指示詞來包含離線同步處理程式碼。 定義 **OFFLINE\_SYNC\_ENABLED** 符號後，組建中會包含這些程式碼路徑。 針對 Windows 應用程式，您也必須安裝 SQLite 平台。

1. 在 Visual Studio 中，以滑鼠右鍵按一下方案 > [管理方案的 NuGet 套件...]，然後為方案中的所有專案，尋找並安裝 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 套件。
2. 在 [方案總管] 中，從名稱中有 **Portable** 的專案中開啟 TodoItemManager.cs 檔案，也就是可攜式類別庫專案，然後取消註解下列前置處理器指示詞︰

        #define OFFLINE_SYNC_ENABLED
3. (選擇性) 若要支援 Windows 裝置，請安裝下列其中一個 SQLite 執行階段封裝︰

   * **Windows 8.1 執行階段：** 安裝[適用於 Windows 8.1 的 SQLite][3]。
   * **Windows Phone 8.1：** 安裝[適用於 Windows Phone 8.1 的 SQLite][4]。
   * **通用 Windows 平台：** 安裝 [適用於通用 Windows 平台的 SQLite][5]。

     雖然快速入門中未包含通用 Windows 專案，使用 Xamarin Forms 可支援通用 Windows 平台。
4. (選擇性) 在每個 Windows 應用程式專案中，以滑鼠右鍵按一下 [參考] >  [新增參考...]，展開 [Windows] 資料夾 > [擴充]。
    啟用適當的 **SQLite for Windows** SDK 及 **Visual C++ 2013 Runtime for Windows** SDK。
    每個 Windows 平台的 SQLite SDK 名稱稍有差異。

## <a name="review-the-client-sync-code"></a>檢閱用戶端同步處理程式碼
以下是教學課程程式碼的 `#if OFFLINE_SYNC_ENABLED` 指示詞內已包含之程式碼的簡要概觀。 離線同步處理功能位於可攜式類別庫專案的 TodoItemManager.cs 專案檔中。 如需此功能的概念性概觀，請參閱 [Azure Mobile Apps 中的離線資料同步處理][2]。

* 必須先初始化本機存放區，才可以執行資料表作業。 **TodoItemManager** 類別建構函式中使用下列程式碼來初始化本機存放區資料庫︰

        var store = new MobileServiceSQLiteStore(OfflineDbPath);
        store.DefineTable<TodoItem>();

        //Initializes the SyncContext using the default IMobileServiceSyncHandler.
        this.client.SyncContext.InitializeAsync(store);

        this.todoTable = client.GetSyncTable<TodoItem>();

    此代碼會使用 **MobileServiceSQLiteStore** 類別建立新的本機 SQLite 資料庫。

    **DefineTable** 方法會在本機存放區中建立一個與所提供之類型的欄位相符的資料表。  該類型不必包含遠端資料庫中的所有資料行。 可以儲存資料行的子集。
* **TodoItemManager** 中的 **todoTable** 欄位是 **IMobileServiceSyncTable** 類型，而非 **IMobileServiceTable**。 此類別使用本機資料庫來進行所有建立、讀取、更新和刪除 (CRUD) 資料表作業。 您可以在 **IMobileServiceSyncContext** 上呼叫 **PushAsync**，以決定何時將這些變更推送到行動應用程式後端。 當呼叫 **PushAsync** 時，透過追蹤和推送所有用戶端應用程式修改之資料表中的變更，同步處理內容可協助保存資料表關聯性。

    呼叫下列 **SyncAsync** 方法可以與行動應用程式後端同步處理︰

        public async Task SyncAsync()
        {
            ReadOnlyCollection<MobileServiceTableOperationError> syncErrors = null;

            try
            {
                await this.client.SyncContext.PushAsync();

                await this.todoTable.PullAsync(
                    "allTodoItems",
                    this.todoTable.CreateQuery());
            }
            catch (MobileServicePushFailedException exc)
            {
                if (exc.PushResult != null)
                {
                    syncErrors = exc.PushResult.Errors;
                }
            }

            // Simple error/conflict handling.
            if (syncErrors != null)
            {
                foreach (var error in syncErrors)
                {
                    if (error.OperationKind == MobileServiceTableOperationKind.Update && error.Result != null)
                    {
                        //Update failed, reverting to server's copy.
                        await error.CancelAndUpdateItemAsync(error.Result);
                    }
                    else
                    {
                        // Discard local change.
                        await error.CancelAndDiscardItemAsync();
                    }

                    Debug.WriteLine(@"Error executing sync operation. Item: {0} ({1}). Operation discarded.",
                        error.TableName, error.Item["id"]);
                }
            }
        }

    這個範例使用簡單的錯誤處理及預設同步處理常式。 實際的應用程式會使用自訂 **IMobileServiceSyncHandler** 實作來處理各種錯誤，例如網路狀況和伺服器衝突。

## <a name="offline-sync-considerations"></a>離線同步處理考量
在範例中，只會在開始時和特別要求同步處理時呼叫 **SyncAsync** 方法。  若要在 Android 或 iOS 應用程式中起始同步處理，請在項目清單上向下拉。若是 Windows，請使用 [同步] 按鈕。 在實際的應用程式中，您也可以在網路狀態變更時觸發此同步處理。

針對具有內容追蹤之擱置中本機更新的資料表執行提取，該提取作業將自動觸發先前的內容推送。 在此範例中重新整理、新增和完成項目時，您可以省略明確的 **PushAsync** 呼叫。

在提供的程式碼中，遠端 TodoItem 資料表中的所有記錄都會進行查詢，但是也可能透過將查詢識別碼與查詢傳遞至 **PushAsync**來篩選記錄。 如需詳細資訊，請參閱 [Azure Mobile Apps 中的離線資料同步處理][2]中的「增量同步處理」一節。

## <a name="run-the-client-app"></a>執行用戶端應用程式
離線同步現在已啟用，請在每個平台執行用戶端應用程式至少一次，以填入本機存放區資料庫。 稍後，模擬一個離線案例，並在應用程式離線時修改本機存放區中的資料。

## <a name="update-the-sync-behavior-of-the-client-app"></a>更新用戶端應用程式的同步處理行為
在本節中，您將透過使用無效的後端應用程式 URL，修改用戶端專案來模擬離線案例。 或者，您可以將裝置切換至「飛航模式」來關閉網路連線。  當您新增或變更資料項目時，這些變更會存放在本機存放區，但不會同步處理到後端資料存放區，直到重新建立連線為止。

1. 在 [方案總管] 中，從 **Portable** 專案開啟 Constants.cs 專案檔案，然後變更 `ApplicationURL` 的值以指向無效的 URL：

        public static string ApplicationURL = @"https://your-service.azurewebsites.net/";
2. 從 **Portable** 專案開啟 TodoItemManager.cs 檔案，然後將基底 **Exception** 類別的 **catch** 新增至 **SyncAsync** 中的 **try…catch** 區塊。 這個 **catch** 區塊會將例外狀況訊息寫入主控台，如下所示︰

            catch (Exception ex)
            {
                Console.Error.WriteLine(@"Exception: {0}", ex.Message);
            }
3. 建置並執行用戶端應用程式。  新增一些新項目。 請注意，每次嘗試與後端同步時，主控台都會記錄例外狀況。 這些新的項目在可推送至行動後端之前，都只會存留在本機存放區中。 用戶端應用程式的行為，會如同它已連接到支援所有建立、讀取、更新、刪除 (CRUD) 作業的後端。
4. 關閉應用程式並重新加以開啟，以驗證您所建立的新項目持續存留於本機存放區中。
5. (選擇性) 使用 Visual Studio 檢視您的 Azure SQL Database 資料表，以查看後端資料庫中的資料並無變更。

    在 Visual Studio 中，開啟 [伺服器總管] 。 瀏覽至 [Azure]->[SQL Database 中您的資料庫]。 在資料庫上按一下滑鼠右鍵，並選取 [在 SQL Server 物件總管中開啟] 。 現在您可以瀏覽至您的 SQL Database 資料表和其內容。

## <a name="update-the-client-app-to-reconnect-your-mobile-backend"></a>更新用戶端應用程式，重新連接您的行動後端
在本節中，會將應用程式重新連接到行動後端，而模擬回到線上狀態的應用程式。 當您執行重新整理動作時，資料會同步處理至您的行動後端中。

1. 重新開啟 Constants.cs。 更正 `applicationURL` 以指向正確的 URL。
2. 重建並執行用戶端應用程式。 應用程式在啟動後會嘗試與行動應用程式後端同步處理。 請確認偵錯主控台沒有記錄任何例外狀況。
3. (選擇性) 使用 SQL Server 物件總管或 REST 工具 (例如 Fiddler 或 [Postman][6]) 檢視已更新的資料。 請注意，後端資料庫與本機存放區之間尚未同步處理資料。

    請注意，資料庫與本機存放區之間的資料已同步處理，並包含應用程式中斷連接時您所新增的項目。

## <a name="additional-resources"></a>其他資源
* [Azure Mobile Apps 中的離線資料同步處理][2]
* [Azure Mobile Apps .NET SDK HOWTO][8]

<!-- URLs. -->
[1]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[2]: app-service-mobile-offline-data-sync.md
[3]: http://go.microsoft.com/fwlink/p/?LinkID=716919
[4]: http://go.microsoft.com/fwlink/p/?LinkID=716920
[5]: http://sqlite.org/2016/sqlite-uwp-3120200.vsix
[6]: https://www.getpostman.com/
[7]: http://www.telerik.com/fiddler
[8]: app-service-mobile-dotnet-how-to-use-client-library.md
