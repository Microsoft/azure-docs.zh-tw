---
title: Azure 通知中樞卸除的通知診斷
description: 了解如何透過 Azure 通知中樞卸除的通知診斷常見問題。
services: notification-hubs
documentationcenter: Mobile
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: b5c89a2a-63b8-46d2-bbed-924f5a4cce61
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: NA
ms.devlang: multiple
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: bc9ef70560f0485da81c1f54aa955cee76d280ab
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777594"
---
# <a name="diagnose-dropped-notifications-in-notification-hubs"></a>診斷通知中樞卸除的通知

Azure 通知中樞客戶最常遇到的其中一項問題是，用戶端裝置上未出現從應用程式發送的通知時，如何進行疑難排解。 他們想要知道卸除通知的位置和原因，以及如何修正此問題。 本文介紹卸除通知的可能原因，或裝置未收到通知的可能原因。 了解如何分析及判斷根本原因。 

請務必先了解通知中樞服務如何將通知推播至裝置。

![通知中樞架構][0]

一般傳送通知流程中，訊息會從*應用程式後端*傳送至通知中樞。 通知中樞會在所有註冊上進行一些處理。 處理會將設定的標記及標記運算式列入考量，以判斷「目標」。 目標是需要接收推播通知的所有註冊。 這些註冊可以跨越任何或所有我們支援的平台：包括 iOS、Google、Windows、Windows Phone、Kindle 及用於中國 Android 的百度。

透過建立目標，通知中樞服務會將推播通知到裝置平台的*推播通知服務*。 範例包括適用於 Apple 的 Apple Push Notification service (APNs)，以及適用於 Google 的 Firebase Cloud Messaging (FCM)。 通知中樞跨多個批次註冊推播通知。 通知中樞會根據您在 Azure 入口網站中，[設定通知中樞] 下方設定的認證，驗證各個推播通知服務。 隨後，推播通知服務會將通知轉送到各個*用戶端裝置*。 

通知傳遞的最終階段是發生在平台推播通知服務和裝置之間。 推播通知處理程序中四個主要元件的任何一項 (用戶端、應用程式後端、通知中樞和平台推播通知服務)，可能會導致卸除通知。 如需更多通知中樞架構的相關資訊，請參閱[通知中樞概觀]。

在初始測試/暫存階段期間，可能會發生無法傳送通知的情況。 在此階段卸除的通知可能表示設定有問題。 如果在生產環境中無法傳送通知，則可能會卸除部分或全部通知。 在此情況下，表示更深入的應用程式或傳訊模式問題。 

下一節介紹可能會卸除通知的情況，從常見到罕見的案例皆有。

## <a name="notification-hubs-misconfiguration"></a>通知中樞設定錯誤
若要成功地將通知傳送到各個推播通知服務，通知中樞服務需要在開發人員應用程式的內容中自行驗證。 為此，開發人員會使用各個平台 (Google、Apple、Windows 等等) 建立開發人員帳戶。 接著，開發人員會使用其取得認證的平台註冊應用程式。 

您必須將平台認證新增至 Azure 入口網站。 如果裝置未收到任何通知，第一個步驟應是確保在通知中樞已設定正確的認證。 認證必須符合在特定平台開發人員帳戶下建立的應用程式。 

若要完成此程序的逐步指示，請參閱[開始使用 Azure 通知中樞]。

以下是一些常見的錯誤設定檢查：

* **一般**
   
    * 確認通知中樞名稱與以下各項位置相同 (無錯別字)：

        * 從用戶端註冊的位置。
        * 從後端傳送通知的位置。
        * 設定推播通知服務認證的位置。
    
    * 請確定您在用戶端和應用程式後端使用正確的共用存取簽章設定字串。 通常，您必須在用戶端上使用 **DefaultListenSharedAccessSignature**，在應用程式後端上使用 **DefaultFullSharedAccessSignature** (授與將通知傳送到通知中樞的權限)。

* **APNs 設定**
   
    您必須維護兩個不同的中樞：一個供生產環境使用的中樞，另一個供測試使用的中樞。 這表示您必須將於沙箱環境中使用的憑證上傳到該憑證以外的另一個中樞，以及將於生產環境中使用的中樞。 請勿嘗試將不同類型的憑證上傳至相同的中樞。 這可能會導致通知失敗。 
    
    如果不小心將不同類型的憑證上傳至相同的中樞，我們建議您刪除該中樞，並使用新的中樞重新開始。 如果您基於某些原因無法刪除中樞，至少必須刪除中樞內所有現有註冊。 

* **FCM 設定** 
   
    1. 請確認您從 Firebase 取得的*伺服器金鑰*符合您在 Azure 入口網站中註冊的伺服器金鑰。
   
    ![Firebase 伺服器金鑰][3]
   
    2. 確認您已在用戶端上設定**專案識別碼**。 您可以從 Firebase 儀表板取得**專案識別碼**的值。
   
    ![Firebase 專案識別碼][1]

## <a name="application-issues"></a>應用程式問題
* **標記與標記運算式**

    如果您使用標記或標記運算式來區分對象，在傳送通知時，根據在傳送呼叫中指定的標記或標記運算式，可能會找不到任何目標。 
    
    檢閱您的註冊，以確保傳送通知時有相符的標記。 然後，僅驗證具有這些註冊之用戶端的通知回條。 
    
    例如，如果利用通知中樞執行的所有註冊都是使用 "Politics" 標記來完成，並且傳送帶有 "Sports" 標記的通知，則不會將通知傳送到任何裝置。 複雜的案例可能涉及使用 "Tag A" OR "Tag B" 註冊的標記運算式，然而在傳送通知時，您設定的目標為 "Tag A && Tag B"。 在本文稍後的自我診斷秘訣一節中，將示範如何檢閱您的註冊和其標記。 

* **範本問題**

    如果您使用範本，請確定遵循[範本]中所述的指導方針。 

* **無效的註冊**

    如果已正確設定通知中樞，而且已正確地使用任何標記或標記運算式，則會找到有效的目標。 通知應該傳送給這些目標。 然後，通知中樞服務會以並行的方式一連串地傳送數個處理批次。 每個批次會將訊息傳送至一組註冊。 

    > [!NOTE]
    > 因為是並行執行處理，所以不能保證通知的傳遞順序。 

    我們已根據「最多一次」訊息傳遞模式將通知中樞最佳化。 我們會嘗試刪除重複的項目，避免系統將通知多次傳遞給同一部裝置。 為了確保功效，我們會先檢驗註冊並確認系統只會將一則訊息傳送給一個裝置識別碼，然後再將訊息傳送給推播通知服務。 

    由於每個批次會傳送至推播通知服務，接著推播通知服務會接受並驗證註冊，因此推播通知服務可能會偵測到批次中的一或多個註冊錯誤。 在此情況下，推播通知服務會將錯誤傳回通知中樞，並停止處理程序。 推播通知服務會完全卸除該批次。 對於使用 TCP 串流通訊協定的 APNS 來說，這種情況尤其顯著。 

    我們針對「最多一次」傳遞進行最佳化。 但在此情況下，會從資料庫移除失敗的註冊。 然後，我們會針對該批次中的其餘裝置，重試通知傳遞。

    若要針對註冊取得錯誤傳遞嘗試的更多錯誤資訊，您可以使用通知中樞 REST API：[每個訊息遙測︰取得通知訊息遙測](https://msdn.microsoft.com/library/azure/mt608135.aspx)以及 [PNS 回應](https://msdn.microsoft.com/library/azure/mt705560.aspx)。 如需範例程式碼，請參閱[傳送 REST 範例](https://github.com/Azure/azure-notificationhubs-samples/tree/master/dotnet/SendRestExample)。

## <a name="push-notification-service-issues"></a>推播通知服務問題
平台推播通知服務收到通知訊息之後，推播通知服務會負責將通知傳遞給裝置。 此時，通知中樞服務與推播通知服務的運作並不相關，也無法控制它們將通知傳遞給裝置的時間和成功與否。 

由於平台通知服務已臻完備，因此通知經常能在幾秒鐘之內從推播通知服務抵達裝置。 如果推播通知服務正在進行節流，通知中樞會套用指數輪詢策略。 如果推播通知服務在 30 分鐘內仍無法連線，我們已制定一項策略，在到期時會永久卸除這些訊息。 

如果推播通知服務嘗試傳遞通知但裝置已離線，推播通知服務會在一段有限的時間內儲存通知。 並且在可使用裝置時，再行傳遞。 

對於每個應用程式，只會儲存最近的通知。 如果在裝置離線時傳送多則通知，每則新通知都會導致前一則通知遭到捨棄。 僅保留最新通知在 APNs 中稱為*聯合通知*，在 FCM 中稱為*摺疊* (使用摺疊索引鍵之故)。 如果裝置長期處於離線狀態，針對該裝置儲存的所有通知都將遭到捨棄。 如需詳細資訊，請參閱[APNs 概觀]和[關於 FCM 訊息]。

使用 Azure 通知中樞，您可以藉由使用一般 SendNotification API，透過 HTTP 標頭傳遞聯合索引鍵。 例如，對於 .NET SDK，您會使用 **SendNotificationAsync**。 SendNotification API 也會採用依現狀傳遞給各個推播通知服務的 HTTP 標頭。 

## <a name="self-diagnosis-tips"></a>自我診斷秘訣
以下是診斷通知中樞卸除通知之根本原因的路徑：

### <a name="verify-credentials"></a>驗證認證
* **推播通知服務開發人員入口網站**
   
    確認個別推播通知服務開發人員入口網站 (APNs、FCM、Windows 通知服務等等) 中的認證。 如需詳細資訊，請參閱[開始使用 Azure 通知中樞]。

* **Azure 入口網站**
   
    若要檢閱並比對您從推播通知服務開發人員入口網站取得的認證，請在 Azure 入口網站中，移至 [存取原則] 索引標籤。 
   
    ![Azure 入口網站存取原則][4]

### <a name="verify-registrations"></a>驗證註冊
* **Visual Studio**
   
    如果您使用 Visual Studio 來從事開發，可以透過伺服器總管連線至 Azure 來檢視及管理多項 Azure 服務，包括通知中樞。 這能為您的開發/測試環境帶來許多助益。 
   
    ![Visual Studio 伺服器總管][9]
   
    您可檢視及管理中樞內的所有註冊，這些註冊可按照平台、原生或範本註冊、任何標記、推播通知服務識別碼、註冊 ID 及到期日來歸類。 您也可以在此頁面上編輯註冊。 這特別適合編輯標記。 
   
    ![Visual Studio 裝置註冊][8]
   
   > [!NOTE]
   > 僅在開發/測試期間使用 Visual Studio 編輯註冊，且註冊數量有限。 如果您需要大量編輯註冊，請按[大量匯出及修改註冊](https://msdn.microsoft.com/library/dn790624.aspx)的說明，考慮使用匯出及匯入註冊功能。
   > 
   > 
* **服務匯流排總管**
   
    許多客戶使用[服務匯流排總管]來檢視和管理其通知中樞。 服務匯流排總管是開放原始碼專案。 如需範例，請參閱[服務匯流排總管程式碼]。

### <a name="verify-message-notifications"></a>驗證訊息通知
* **Azure 入口網站**
   
    若要將測試通知傳送到您的用戶端，而不需要運作後端服務，請在 [支援 + 疑難排解] 下，選取 [測試傳送]。 
   
    ![測試 Azure 中的傳送功能][7]
* **Visual Studio**
   
    您也可以從 Visual Studio 傳送測試通知。
   
    ![測試 Visual Studio 中的傳送功能][10]
   
    如需更多使用通知中樞與 Visual Studio 伺服器總管的相關資訊，請參閱下列文章： 
   
   * [檢視通知中樞的裝置註冊]
   * [深入探討：Visual Studio 2013 Update 2 RC 和 Azure SDK 2.3]
   * [宣佈發行 Visual Studio 2013 Update 3 和 Azure SDK 2.4]

### <a name="debug-failed-notifications-and-review-notification-outcome"></a>偵錯失敗的通知並檢閱通知結果
**EnableTestSend 屬性**

當您透過通知中樞傳送通知時，一開始，通知會加入通知中樞的佇列等候處理。 通知中樞會確定正確的目標，然後將通知傳送至推播通知服務。 如果您使用 REST API 或任何用戶端 SDK，成功傳回傳送呼叫只代表系統已將訊息順利地加入通知中樞的佇列。 通知中樞最終將訊息傳送至推播通知服務後，您將無法深入瞭解發生的情況。 

如果您的通知未抵達用戶端裝置，當通知中樞嘗試將訊息傳遞給推播通知服務時，可能會發生錯誤。 例如，承載大小可能會超過推播通知服務允許的上限，或通知中樞中設定的認證可能無效。 

若要深入了解推播通知服務的錯誤，您可以使用 [EnableTestSend] 屬性。 當您從入口網站或 Visual Studio 用戶端傳送測試訊息時，會自動啟用該屬性。 您可以使用該屬性查看詳細的偵錯資訊。 您也可以透過 API 使用屬性。 目前，您可以在 .NET SDK 中使用該屬性。 最後，它會加入所有用戶端 SDK。 

若要搭配使用 **EnableTestSend** 屬性與 REST 呼叫，請在傳送呼叫結尾附加名為 *test* 的查詢字串參數。 例如︰ 

    https://mynamespace.servicebus.windows.net/mynotificationhub/messages?api-version=2013-10&test

**範例 (.NET SDK)**

以下是使用 .NET SDK 傳送原生快顯視窗 (快顯) 通知的範例：

```csharp
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName);
    var result = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(result.State);
```

執行結束時，**result.State** 只是表示**加入佇列**。 結果不會提供任何有關推播通知發生情況的深入解析。 

接著，您可以使用 **EnableTestSend** 布林值屬性。 初始化 **NotificationHubClient** 時，請使用 **EnableTestSend** 屬性，以取得關於傳送通知時發生的推播通知服務錯誤詳細狀態。 傳送呼叫需要額外的時間才能傳回，因為只有在通知中樞將通知傳遞給推播通知服務以確定結果後，才會傳回。 

```csharp
    bool enableTestSend = true;
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName, enableTestSend);

    var outcome = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(outcome.State);

    foreach (RegistrationResult result in outcome.Results)
    {
        Console.WriteLine(result.ApplicationPlatform + "\n" + result.RegistrationId + "\n" + result.Outcome);
    }
```

**範例輸出**

    DetailedStateAvailable
    windows
    7619785862101227384-7840974832647865618-3
    The Token obtained from the Token Provider is wrong

此訊息表示通知中樞所設定的認證無效，或中樞的註冊發生問題。 我們建議您刪除此註冊，並讓用戶端先重新建立註冊，然後再傳送訊息。 

> [!NOTE]
> 使用 **EnableTestSend** 屬性會進行大幅度的節流處理。 僅在開發/測試環境中使用此選項，並使用有限的一組註冊。 我們只會傳送偵錯通知給 10 部裝置。 我們的處理中偵錯傳送次數也有每分鐘 10 次的限制。 
> 
> 

### <a name="review-telemetry"></a>檢閱遙測
* **使用 Azure 入口網站**
   
    在入口網站中，您可以取得通知中樞上所有活動的簡要概觀。 
   
    1. 在 [概觀] 索引標籤上，您可以依平台查看註冊、通知和錯誤的彙總檢視。 
   
        ![通知中樞概觀儀表板][5]
   
    2. 在 [監視器] 索引標籤上，您可以新增許多其他平台特定度量以獲得更深入的了解。 您可以專門查看通知中樞服務嘗試將通知傳送至推播通知服務時，傳回的任何推播通知服務相關錯誤。 
   
        ![Azure 入口網站活動記錄][6]
   
    3. 首先，檢閱**內送郵件**、**註冊作業**，以及**成功通知**。 然後，移至每個平台的索引標籤，以檢閱特定於推播通知服務的錯誤。 
   
    4. 如果您通知中樞的驗證設定不正確，會出現 **PNS 驗證錯誤**訊息。 這是提醒您檢查推播通知服務認證的徵兆。 

* **程式設計存取**

    如需程式設計存取的詳細資訊，請參閱下列文章： 

    * [程式設計遙測存取]  
    * [透過 API 進行遙測存取的範例] 

    > [!NOTE]
    > 幾個與遙測有關的功能，例如匯出和匯入註冊，以及透過 API 進行遙測存取，只能在標準服務層上使用。 如果您嘗試從免費層或基本服務層使用這些功能，若您使用 SDK，則會收到例外狀況錯誤，若直接從 REST API 使用這些功能，則會收到 HTTP 403 (禁止) 錯誤。 
    >
    >若要使用遙測相關功能，請先確定在 Azure 入口網站中使用標準服務層。  
> 
> 

<!-- IMAGES -->
[0]: ./media/notification-hubs-diagnosing/Architecture.png
[1]: ./media/notification-hubs-diagnosing/FCMConfigure.png
[3]: ./media/notification-hubs-diagnosing/FCMServerKey.png
[4]: ../../includes/media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png
[5]: ./media/notification-hubs-diagnosing/PortalDashboard.png
[6]: ./media/notification-hubs-diagnosing/PortalAnalytics.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[8]: ./media/notification-hubs-diagnosing/VSRegistrations.png
[9]: ./media/notification-hubs-diagnosing/VSServerExplorer.png
[10]: ./media/notification-hubs-diagnosing/VSTestNotification.png

<!-- LINKS -->
[通知中樞概觀]: notification-hubs-push-notification-overview.md
[開始使用 Azure 通知中樞]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[範本]: https://msdn.microsoft.com/library/dn530748.aspx 
[APNs 概觀]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html
[關於 FCM 訊息]: https://firebase.google.com/docs/cloud-messaging/concept-options
[Export and modify registrations in bulk]: http://msdn.microsoft.com/library/dn790624.aspx
[服務匯流排總管]: https://msdn.microsoft.com/library/dn530751.aspx#sb_explorer
[服務匯流排總管程式碼]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a
[檢視通知中樞的裝置註冊]: http://msdn.microsoft.com/library/windows/apps/xaml/dn792122.aspx 
[深入探討：Visual Studio 2013 Update 2 RC 和 Azure SDK 2.3]: http://azure.microsoft.com/blog/2014/04/09/deep-dive-visual-studio-2013-update-2-rc-and-azure-sdk-2-3/#NotificationHubs 
[宣佈發行 Visual Studio 2013 Update 3 和 Azure SDK 2.4]: http://azure.microsoft.com/blog/2014/08/04/announcing-release-of-visual-studio-2013-update-3-and-azure-sdk-2-4/ 
[EnableTestSend]: https://docs.microsoft.com/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.enabletestsend?view=azure-dotnet
[程式設計遙測存取]: http://msdn.microsoft.com/library/azure/dn458823.aspx
[透過 API 進行遙測存取的範例]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/FetchNHTelemetryInExcel

