---
title: Azure Mobile Apps 中的離線資料同步處理 | Microsoft Docs
description: Azure 行動應用程式離線資料同步處理功能的概念參考與概觀
documentationcenter: windows
author: conceptdev
manager: crdun
editor: ''
services: app-service\mobile
ms.assetid: 982fb683-8884-40da-96e6-77eeca2500e3
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 10/30/2016
ms.author: crdun
ms.openlocfilehash: 403b3289a6135a59e888c4a7bf1cd9cbe40c8ac0
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34597972"
---
# <a name="offline-data-sync-in-azure-mobile-apps"></a>Azure 行動應用程式中的離線資料同步處理
## <a name="what-is-offline-data-sync"></a>什麼是離線資料同步處理？
離線資料同步處理是 Azure 行動應用程式的用戶端和伺服器 SDK 功能，可讓開發人員建立不需要網路連線就能運作的應用程式。

當您的應用程式處於離線模式時，您仍然可以建立和修改資料，所做的變更會儲存至本機存放區。 當應用程式回到線上時，即可將本機變更與您的 Azure 行動應用程式後端同步處理。 此功能也包含偵測衝突的支援，即當同一筆記錄在用戶端和後端都發生變更。 衝突可以在伺服器或用戶端處理。

離線同步處理有幾項優點：

* 在裝置上本機快取伺服器資料，以改善應用程式回應性
* 建立當網路有問題時仍可以使用的健全應用程式。
* 即使無法存取網路，使用者也能建立和修改資料，支援連線不佳或無連線的情況
* 同步多個裝置之間的資料，並在兩個裝置修改相同的記錄時偵測衝突
* 高延遲或計量付費網路的網路使用限制。

下列教學課程說明如何使用 Azure 行動應用程式將離線同步處理新增至您的行動用戶端：

* [Android：啟用離線同步處理]
* [Apache Cordova︰啟用離線同步處理](app-service-mobile-cordova-get-started-offline-data.md)
* [iOS：啟用離線同步處理]
* [Xamarin iOS：啟用離線同步處理]
* [Xamarin Android：啟用離線同步處理]
* [Xamarin.Forms：啟用離線同步處理](app-service-mobile-xamarin-forms-get-started-offline-data.md)
* [通用 Windows 平台︰啟用離線同步處理]

## <a name="what-is-a-sync-table"></a>什麼是同步處理資料表？
若要存取 "/tables" 端點，Azure 行動用戶端 SDK 提供 `IMobileServiceTable` (.NET 用戶端 SDK) 或 `MSTable` (iOS 用戶端) 等介面。 這些 API 直接連接至 Azure 行動應用程式後端，如果用戶端裝置沒有網路連接，則會失敗。

若要支援離線使用，您的應用程式應改為使用「同步處理資料表」API，例如 `IMobileServiceSyncTable` (.NET 用戶端 SDK) 或 `MSSyncTable` (iOS 用戶端)。 所有相同的 CRUD 作業 (Create、Read、Update、Delete) 適用於同步資料表 API，但現在會讀取或寫入「本機存放區」。 必須先初始化本機存放區，才能執行任何同步處理資料表作業。

## <a name="what-is-a-local-store"></a>什麼是本機存放區？
本機存放區是用戶端裝置上的資料持續層。 Azure 行動應用程式用戶端 SDK 提供預設的本機存放區實作。 在 Windows、Xamarin 和 Android 上，它是以 SQLite 為基礎。 在 iOS 上，它是以 Core Data 為基礎。

若要在 Windows Phone 或 Microsoft Store 上使用 SQLite 為基礎的實作，您需要安裝 SQLite 擴充。 如需詳細資訊，請參閱[通用 Windows 平台︰啟用離線同步處理]。Android 與 iOS 裝置的作業系統本身即包含 SQLite 版本，因此您不需要再參考自己的 SQLite 版本。

開發人員也可以實作自己的本機存放區。 例如，如果您希望將資料以加密格式儲存在行動用戶端上，則您可以定義使用 SQLCipher 進行加密的本機存放區。

## <a name="what-is-a-sync-context"></a>什麼是同步處理內容？
「同步處理內容」會與行動用戶端物件相關聯 (例如 `IMobileServiceClient` 或 `MSClient`)，並且追蹤對同步處理資料表所做的變更。 同步處理內容會維護「作業佇列」，其中會保留 CUD 作業 (Create、Update、Delete) 的排序清單，這些作業稍後會傳送至伺服器。

本機存放區會使用初始化方法 (例如 [.NET 用戶端 SDK] 中的 `IMobileServicesSyncContext.InitializeAsync(localstore)`)，來與同步處理內容相關聯。

## <a name="how-sync-works"></a>離線同步處理如何運作
使用同步處理資料表時，您的用戶端程式碼需要控制本機變更與 Azure 行動應用程式後端同步處理的時機。 在有呼叫要「推送」( *push* ) 變更之前不會傳送任何項目到後端。 同樣地，只當有呼叫要「提取」( *pull* ) 時才會將新資料填入本機存放區。

* **推送**：推送是同步處理內容的作業，會傳送自上一次推送之後的所有 CUD 變更。 請注意，您無法只傳送個別資料表的變更，因為這樣作業傳送順序可能會發生錯誤。 推送會對 Azure 行動應用程式後端執行一系列的 REST 呼叫，再由後端修改伺服器資料庫。
* **提取**：提取會以各資料表為基礎執行，並且可以使用佇列來自訂，以抓取伺服器資料的特定子集。 然後 Azure 行動用戶端 SDK 會將該結果資料插入本機存放區。
* **隱含推送**：如果對有擱置中本機更新的資料表執行提取，則提取會先在同步處理內容上執行 `push()`。 此推送有助於將已排入佇列的變更與來自伺服器的新資料之間的衝突降到最低。
* **增量同步處理**：提取作業的第一個參數是「查詢名稱」，此參數只在用戶端使用。 如果您使用非 null 的查詢名稱，Azure 行動 SDK 會執行「增量同步處理」 。每當提取作業傳回結果集，該結果集中最新的 `updatedAt` 時間戳記就會儲存在 SDK 本機系統資料表。 後續的提取作業只會擷取該時間戳記之後的記錄。

  若要使用增量同步處理，您的伺服器必須傳回有意義的 `updatedAt` 值，也必須支援依據此欄位排序。 不過，由於 SDK 會在 updatedAt 欄位上加入自己的排序，所以您不能使用本身具備 `orderBy` 子句的提取查詢。

  查詢名稱可以是您選擇的任何字串，但它在應用程式中的每個邏輯查詢都必須是唯一的。
  否則，不同的提取作業可能會覆寫相同的增量同步處理時間戳記，您的查詢可能因此傳回不正確的結果。

  如果查詢具有參數，一個建立唯一查詢名稱的方法是納入該參數值。
  例如，如果您要篩選 userid，您的查詢名稱可能如下 (在 C# 中)：

        await todoTable.PullAsync("todoItems" + userid,
            syncTable.Where(u => u.UserId == userid));

  如果您想選擇不要增量同步處理，則傳遞 `null` 做為查詢識別碼。 在此情況下，每次呼叫 `PullAsync` 時都會擷取所有的記錄，這樣可能沒有效率。
* **清除**：您可以使用 `IMobileServiceSyncTable.PurgeAsync` 來清除本機存放區的內容。
  如果用戶端資料庫中有過時資料，或者您想要捨棄所有擱置的變更，可能就需要清除。

  清除作業會從本機存放區清除資料表。 如果有正在等待與伺服器資料庫同步處理的作業，則清除會擲回例外狀況，除非設定 *force purge* 參數。

  舉例說明用戶端上過時資料，假設在 "todo list" 範例中，Device1 只會提取未完成的項目。 有其他裝置在伺服器上將 "Buy milk" todoitem 標記為已完成。 不過，Device1 在本機存放區仍有 "Buy milk" todoitem，因為它只提取未標記為已完成的項目。 清除作業會清除這個過時項目。

## <a name="next-steps"></a>後續步驟
* [iOS：啟用離線同步處理]
* [Xamarin iOS：啟用離線同步處理]
* [Xamarin Android：啟用離線同步處理]
* [通用 Windows 平台︰啟用離線同步處理]

<!-- Links -->
[.NET 用戶端 SDK]: app-service-mobile-dotnet-how-to-use-client-library.md
[Android：啟用離線同步處理]: app-service-mobile-android-get-started-offline-data.md
[iOS：啟用離線同步處理]: app-service-mobile-ios-get-started-offline-data.md
[Xamarin iOS：啟用離線同步處理]: app-service-mobile-xamarin-ios-get-started-offline-data.md
[Xamarin Android：啟用離線同步處理]: app-service-mobile-xamarin-android-get-started-offline-data.md
[通用 Windows 平台︰啟用離線同步處理]: app-service-mobile-windows-store-dotnet-get-started-offline-data.md
