---
title: 啟用 Azure 行動應用程式的離線同步處理 (Android)
description: 了解如何使用 App Service Mobile Apps 來快取和同步處理離線資料
documentationcenter: android
author: conceptdev
manager: crdun
services: app-service\mobile
ms.assetid: 32a8a079-9b3c-4faf-8588-ccff02097224
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 152702bed0ea061c3cb86e2ff6f88bf204f9d243
ms.sourcegitcommit: df4ddc55b42b593f165d56531f591fdb1e689686
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/04/2018
ms.locfileid: "27594183"
---
# <a name="enable-offline-sync-for-your-android-mobile-app"></a>啟用 Android 行動應用程式的離線同步處理
[!INCLUDE [app-service-mobile-selector-offline](../../includes/app-service-mobile-selector-offline.md)]

## <a name="overview"></a>概觀
本教學課程說明 Android 之 Azure Mobile Apps 的離線同步處理功能。 離線同步處理可讓使用者與行動應用程式進行互動&mdash;檢視、新增或修改資料&mdash;即使沒有網路連接進也可行。 變更會儲存在本機資料庫中。 裝置恢復上線後，這些變更就會與遠端後端進行同步處理。

如果這是您第一次接觸 Azure Mobile Apps，您應先完成 [建立 Android 應用程式]教學課程。 如果您不要使用下載的快速入門伺服器專案，必須將資料存取擴充套件新增至您的專案。 如需伺服器擴充套件的詳細資訊，請參閱 [使用 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md)。

若要深入了解離線同步處理功能，請參閱 [Azure Mobile Apps 中的離線資料同步處理]主題。

## <a name="update-the-app-to-support-offline-sync"></a>更新應用程式以支援離線同步
利用離線同步讀取和寫入*同步資料表* (使用 *IMobileServiceSyncTable* 介面)，這是您裝置上 **SQLite** 資料庫的一部分。

若要推送和提取裝置與 Azure 行動服務之間的變更，您可使用*同步處理內容* (*MobileServiceClient.SyncContext*)，這是您用來在本機儲存資料的本機資料庫所初始化。

1. 在 `TodoActivity.java` 中，將 `mToDoTable` 的現有定義註解化，並取消註解同步資料表版本：
   
        private MobileServiceSyncTable<ToDoItem> mToDoTable;
2. 在 `onCreate` 方法中，將 `mToDoTable` 的現有初始化註解化，並取消註解此定義：
   
        mToDoTable = mClient.getSyncTable("ToDoItem", ToDoItem.class);
3. 在 `refreshItemsFromTable` 中，將 `results` 的定義註解化，並取消註解此定義：
   
        // Offline Sync
        final List<ToDoItem> results = refreshItemsFromMobileServiceTableSyncTable();
4. 註解化 `refreshItemsFromMobileServiceTable`的定義。
5. 取消註解 `refreshItemsFromMobileServiceTableSyncTable`的定義：
   
        private List<ToDoItem> refreshItemsFromMobileServiceTableSyncTable() throws ExecutionException, InterruptedException {
            //sync the data
            sync().get();
            Query query = QueryOperations.field("complete").
                    eq(val(false));
            return mToDoTable.read(query).get();
        }
6. 取消註解 `sync`的定義：
   
        private AsyncTask<Void, Void, Void> sync() {
            AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
                @Override
                protected Void doInBackground(Void... params) {
                    try {
                        MobileServiceSyncContext syncContext = mClient.getSyncContext();
                        syncContext.push().get();
                        mToDoTable.pull(null).get();
                    } catch (final Exception e) {
                        createAndShowDialogFromTask(e, "Error");
                    }
                    return null;
                }
            };
            return runAsyncTask(task);
        }

## <a name="test-the-app"></a>測試應用程式
在本節中，您會在開啟 WiFi 的情況下測試行為，然後關閉 WiFi 以建立離線案例。

當您新增資料項目時，這些項目會存放在本機 SQLite 存放區中，但直到您按 [重新整理]  按鈕時才會同步處理至行動服務。 對於何時需要同步處理資料，其他應用程式可能會有不同的需求，但是為了示範目的，本教學課程讓使用者明確要求。

當您按下該按鈕時，新的背景工作會啟動。 它會使用同步處理內容先推送對本機存放區做的所有變更，然後將所有變更的資料從 Azure 提取至本機資料表。

### <a name="offline-testing"></a>離線測試
1. 讓裝置或模擬器處於「飛航模式」 。 這會建立離線案例。
2. 新增一些 *ToDo* 項目，或將某些項目標示為完成。 結束裝置或模擬器 (或強制關閉應用程式)，然後重新啟動。 請確認您的變更已保存在裝置上，因為它們會保留在本機 SQLite 存放區中。
3. 使用 SQL 工具 (如 *SQL Server Management Studio*) 或 REST 用戶端 (如 *Fiddler* 或 *Postman*) 檢視 Azure *TodoItem* 資料表的內容。 請確認新項目「尚未」  同步處理到伺服器
   
       + 若為 Node.js 後端，請移至 [Azure 入口網站](https://portal.azure.com/)，在您的行動應用程式後端中按一下 [簡單資料表] > [TodoItem]，檢視 `TodoItem` 資料表的內容。
       + 若為 .NET 後端，請使用 SQL 工具 (例如 *SQL Server Management Studio*) 或 REST 用戶端 (例如 *Fiddler* 或 *Postman*) 檢視資料表內容。
4. 在裝置或模擬器中開啟 WiFi。 接著，按 [重新整理]  按鈕。
5. 在 Azure 入口網站中，再次檢視 TodoItem 資料。 新的和變更的 TodoItems 現在應該會出現。

## <a name="additional-resources"></a>其他資源
* [Azure Mobile Apps 中的離線資料同步處理]
* [雲端報導：Azure Mobile Services 中的離線同步處理] \(注意：影片位於 Mobile Services 上，但離線同步處理的運作方式類似在 Azure Mobile Apps 中的方式\)

<!-- URLs. -->

[Azure Mobile Apps 中的離線資料同步處理]: app-service-mobile-offline-data-sync.md

[建立 Android 應用程式]: app-service-mobile-android-get-started.md

[雲端報導：Azure Mobile Services 中的離線同步處理]: http://channel9.msdn.com/Shows/Cloud+Cover/Episode-155-Offline-Storage-with-Donna-Malayeri
[Azure Friday: Offline-enabled apps in Azure Mobile Services]: http://azure.microsoft.com/documentation/videos/azure-mobile-services-offline-enabled-apps-with-donna-malayeri/

