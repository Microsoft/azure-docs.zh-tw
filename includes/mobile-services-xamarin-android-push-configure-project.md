---
author: conceptdev
ms.author: crdun
ms.service: app-service-mobile
ms.topic: include
ms.date: 08/23/2018
ms.openlocfilehash: 42c961b81a254adef5e42c3c8916c9c081f548c8
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809674"
---
1. 在 [方案] 檢視 (或 Visual Studio 中的 [方案總管]) 中，以滑鼠右鍵按一下 [元件] 資料夾，按一下 [取得更多元件]，搜尋 **Google 雲端通訊用戶端**並將其加入至專案。
2. 開啟 ToDoActivity.cs 專案檔案，然後將下列 using 陳述式加入至類別：

    ```csharp
    using Gcm.Client;
    ```

3. 將下列新程式碼加入至 **ToDoActivity** 類別： 

    ```csharp
    // Create a new instance field for this activity.
    static ToDoActivity instance = new ToDoActivity();

    // Return the current activity instance.
    public static ToDoActivity CurrentActivity
    {
        get
        {
            return instance;
        }
    }
    // Return the Mobile Services client.
    public MobileServiceClient CurrentClient
    {
        get
        {
            return client;
        }
    }
    ```

    這樣做可讓您從推播處理常式服務處理程序存取行動服務用戶端執行個體。
4. 在建立 **MobileServiceClient** 之後，將下列程式碼加入至 **OnCreate** 方法：

    ```csharp
    // Set the current instance of TodoActivity.
    instance = this;

    // Make sure the GCM client is set up correctly.
    GcmClient.CheckDevice(this);
    GcmClient.CheckManifest(this);

    // Register the app for push notifications.
    GcmClient.Register(this, ToDoBroadcastReceiver.senderIDs);
    ```

您的 **ToDoActivity** 現在可供新增推送通知。
