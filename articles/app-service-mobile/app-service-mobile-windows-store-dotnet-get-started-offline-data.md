---
title: 使用 Mobile Apps 啟用通用 Windows 平台 (UWP) 應用程式的離線同步處理 | Microsoft Docs
description: 了解如何使用 Azure 行動應用程式快取及同步處理通用 Windows 平台 (UWP) 應用程式中的離線資料。
documentationcenter: windows
author: conceptdev
manager: crdun
editor: ''
services: app-service\mobile
ms.assetid: 8fe51773-90de-4014-8a38-41544446d9b5
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 3df6951129f7beda7970e394ffdd32c7e02304dd
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37060302"
---
# <a name="enable-offline-sync-for-your-windows-app"></a>啟用 Windows 應用程式離線同步處理
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## <a name="overview"></a>概觀
本教學課程說明如何使用 Azure 行動應用程式後端在通用 Windows 平台 (UWP) 應用程式中新增離線支援。 離線同步處理可讓使用者與行動應用程式進行互動 (檢視、新增或修改資料)，即使沒有網路連接也可以。 變更會儲存在本機資料庫中。 裝置恢復上線後，這些變更就會與遠端後端進行同步處理。

在本教學課程中，您將會更新[建立 Windows 應用程式]教學課程中的 UWP 應用程式專案，以支援 Azure Mobile Apps 的離線功能。 如果您不要使用下載的快速入門伺服器專案，必須將資料存取擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

若要深入了解離線同步處理功能，請參閱 [Azure Mobile Apps 中的離線資料同步處理]主題。

## <a name="requirements"></a>需求  
本教學課程需要下列必要條件：

* 執行於 Windows 8.1 或更新版本的 Visual Studio 2013。
* 完成 [建立 Windows 應用程式][建立 Windows 應用程式]。
* [Azure 行動服務 SQLite Store][sqlite store nuget]
* [適用於通用 Windows 平台開發的 SQLite](https://marketplace.visualstudio.com/items?itemName=SQLiteDevelopmentTeam.SQLiteforUniversalWindowsPlatform) 

## <a name="update-the-client-app-to-support-offline-features"></a>更新用戶端應用程式以支援離線功能
Azure 行動應用程式的離線功能可讓您在離線狀態時，仍可與本機資料庫互動。 若要在您的應用程式中使用這些功能，您必須將 [SyncContext][synccontext] 初始化至本機存放區。 接著，請透過 [IMobileServiceSyncTable][IMobileServiceSyncTable] 介面參考您的資料表。 SQLite 可做為裝置上的本機存放區。

1. 安裝 [適用於通用 Windows 平台的 SQLite 執行階段](http://sqlite.org/2016/sqlite-uwp-3120200.vsix)。
2. 在 Visual Studio 中，針對您在[建立 Windows 應用程式]教學課程中完成的專案，開啟 NuGet 封裝管理員。
    搜尋並安裝 **Microsoft.Azure.Mobile.Client.SQLiteStore** NuGet 套件。
3. 在 [方案總管] 中，以滑鼠右鍵按一下 [參考] > [新增參考...]> [通用 Windows] >[擴充功能]，然後啟用 [適用於通用 Windows 平台的 SQLite] 和 [適用於通用 Windows 平台應用程式的 Visual C++ 2015 執行階段]。

    ![新增 SQLite UWP 參考][1]
4. 開啟 MainPage.xaml.cs 檔案，並取消註解 `#define OFFLINE_SYNC_ENABLED` 定義。
5. 在 Visual Studio 中按 **F5** 鍵，以重新建置並執行用戶端應用程式。 應用程式的運作方式和啟用離線同步處理之前的方式相同。不過，本機資料庫現在會填入可在離線案例下使用的資料。

## <a name="update-sync"></a>更新應用程式以使其與後端中斷連線
在本節中，您將中斷與行動應用程式後端的連線，以模擬離線狀態。 當您新增資料項目時，您的例外狀況處理常式會指出應用程式處於離線模式。 處於此狀態時，新項目會新增到本機存放區，並且會在推送接下來於連線狀態執行時，同步處理至行動應用程式後端。

1. 編輯共用專案中的 App.xaml.cs。 註解化 **MobileServiceClient** 的初始化，並新增下一行，其中使用無效的行動應用程式 URL：

         public static MobileServiceClient MobileService = new MobileServiceClient("https://your-service.azurewebsites.fail");

    您也可以透過停用裝置的 WiFi 和行動電話通訊網路，或使用飛航模式來示範離線行為。
2. 按 **F5** 以建置並執行應用程式。 請注意，應用程式啟動後同步處理無法重新整理。
3. 輸入新項目，並注意每次您按一下 **[儲存]** 時，推送都會失敗並具有 [CancelledByNetworkError] 狀態。 不過，新的 todo 項目在可推送至行動應用程式後端之前，都會存留在本機存放區中。  在生產應用程式中，如果您隱藏這些例外狀況，用戶端應用程式的行為會如同它仍連線到行動應用程式後端。
4. 關閉應用程式並重新加以開啟，以驗證您所建立的新項目持續存留於本機存放區中。
5. (選擇性) 在 Visual Studio 中，開啟 [伺服器總管] 。 瀏覽至 [Azure]->[SQL Database 中您的資料庫]。 在資料庫上按一下滑鼠右鍵，並選取 [在 SQL Server 物件總管中開啟] 。 現在您可以瀏覽至您的 SQL Database 資料表和其內容。 確認後端資料庫中的資料沒有變更。
6. (選擇性) 使用 REST 工具 (例如 Fiddler 或 Postman) 來查詢您的行動後端 (使用表單 `https://<your-mobile-app-backend-name>.azurewebsites.net/tables/TodoItem`中的 GET 查詢)。

## <a name="update-online-app"></a>更新應用程式以重新連線您的行動應用程式後端
在本節中，您會將應用程式重新連接至行動應用程式後端。 這些變更在應用程式上模擬網路重新連接。

當您第一次執行應用程式時，`OnNavigatedTo` 事件處理常式會呼叫 `InitLocalStoreAsync`。 這個方法會進而呼叫 `SyncAsync` 來同步處理您的本機存放區與後端資料庫。 應用程式會嘗試在啟動時同步處理。

1. 在共用專案中開啟 App.xaml.cs，取消註解先前的 `MobileServiceClient` 初始化，以使用正確的行動應用程式 URL。
2. 按 **F5** 鍵，以重新建置與執行應用程式。 該應用程式會在 `OnNavigatedTo` 事件處理常式執行時，使用推送與提取作業來同步處理您的本機變更與 Azure 行動應用程式後端。
3. (選擇性) 使用 SQL Server 物件總管或 REST 工具 (例如 Fiddler) 檢視已更新的資料。 請注意，Azure 行動應用程式後端資料庫與本機存放區之間的資料已同步處理。
4. 在應用程式中，按一下幾個項目旁邊的核取方塊，以在本機存放區中完成它們。

   `UpdateCheckedTodoItem` 會呼叫 `SyncAsync`，以便與行動應用程式後端同步處理每個已完成的項目。 `SyncAsync` 會同時呼叫推送與提取。 不過，**每當您對用戶端已變更的資料表執行提取時，一定會執行自動推送**。 此行為可確保本機存放區中的所有資料表和關聯性都保持一致。 此行為可能會導致非預期的推送。  如需此行為的詳細資訊，請參閱 [Azure Mobile Apps 中的離線資料同步處理]。

## <a name="api-summary"></a>API 摘要
為了支援行動服務的離線功能，我們使用 [IMobileServiceSyncTable] 介面，並初始化本機 SQLite 資料庫的 [MobileServiceClient.SyncContext][synccontext]。 離線時，Mobile Apps 的一般 CRUD 作業運作方式，就如同應用程式仍處於連線狀態，而作業會對本機存放區執行。 下列方法可用來同步處理本機存放區與伺服器︰

* **[PushAsync]** 因為這個方法是 [IMobileServicesSyncContext] 的成員，因此所有資料表的變更都會推送至後端。 只有具有本機變更的記錄會傳送到伺服器。
* **[PullAsync]** 從 [IMobileServiceSyncTable] 開始提取。 當資料表中有追蹤的變更時，便會執行隱含推送，以確定本機存放區中的所有資料表和關聯性都維持一致。 pushOtherTables  參數會控制是否在隱含推送中推送內容中的其他資料表。 *query* 參數會採用 [IMobileServiceTableQuery<T>][IMobileServiceTableQuery] 或 OData 查詢字串來篩選傳回的資料。 queryId  參數可用來定義增量同步處理。如需詳細資訊，請參閱 [Azure Mobile Apps 中的離線資料同步處理](app-service-mobile-offline-data-sync.md#how-sync-works)。
* **[PurgeAsync]** 您的應用程式應定期呼叫這個方法，以清除本機存放區中的過時資料。 當您需要清除任何尚未同步處理的變更時，請使用 force  參數。

如需這些概念的詳細資訊，請參閱 [Azure Mobile Apps 中的離線資料同步處理](app-service-mobile-offline-data-sync.md#how-sync-works)。

## <a name="more-info"></a>其他資訊
下列主題提供其他關於 Mobile Apps 離線同步處理功能的背景資訊︰

* [Azure Mobile Apps 中的離線資料同步處理]
* [Azure Mobile Apps .NET SDK HOWTO][8]

<!-- Anchors. -->
[Update the app to support offline features]: #enable-offline-app
[Update the sync behavior of the app]: #update-sync
[Update the app to reconnect your Mobile Apps backend]: #update-online-app
[Next Steps]:#next-steps

<!-- Images -->
[1]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/app-service-mobile-add-reference-sqlite-dialog.png
[11]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/app-service-mobile-add-wp81-reference-sqlite-dialog.png
[13]: ./media/app-service-mobile-windows-store-dotnet-get-started-offline-data/cpu-architecture.png


<!-- URLs. -->
[Azure Mobile Apps 中的離線資料同步處理]: app-service-mobile-offline-data-sync.md
[建立 Windows 應用程式]: app-service-mobile-windows-store-dotnet-get-started.md
[SQLite for Windows 8.1]: http://go.microsoft.com/fwlink/?LinkID=716919
[SQLite for Windows Phone 8.1]: http://go.microsoft.com/fwlink/?LinkID=716920
[SQLite for Windows 10]: http://go.microsoft.com/fwlink/?LinkID=716921
[synccontext]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileserviceclient.synccontext(v=azure.10).aspx
[sqlite store nuget]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client.SQLiteStore/
[IMobileServiceSyncTable]: https://msdn.microsoft.com/library/azure/mt691742(v=azure.10).aspx
[IMobileServiceTableQuery]: https://msdn.microsoft.com/library/azure/dn250631(v=azure.10).aspx
[IMobileServicesSyncContext]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.sync.imobileservicesynccontext(v=azure.10).aspx
[MobileServicePushFailedException]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushfailedexception(v=azure.10).aspx
[Status]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushcompletionresult.status(v=azure.10).aspx
[CancelledByNetworkError]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.sync.mobileservicepushstatus(v=azure.10).aspx
[PullAsync]: https://msdn.microsoft.com/library/azure/mt667558(v=azure.10).aspx
[PushAsync]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.mobileservicesynccontextextensions.pushasync(v=azure.10).aspx
[PurgeAsync]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.mobileservices.sync.imobileservicesynctable.purgeasync(v=azure.10).aspx
[8]: app-service-mobile-dotnet-how-to-use-client-library.md
