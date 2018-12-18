---
title: Azure 通知中樞：常見問題集 (FAQ) | Microsoft Docs
description: 在通知中樞上設計/實作解決方案的常見問題集
services: notification-hubs
documentationcenter: mobile
author: dimazaid
manager: kpiteira
editor: spelluru
keywords: 推播通知, 推播通知, iOS 推播通知, android 推播通知, ios 推播, android 推播
ms.assetid: 7b385713-ef3b-4f01-8b1f-ffe3690bbd40
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: fbdb52b1b2dee682021fd822d584551f41f723ff
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33778506"
---
# <a name="push-notifications-with-azure-notification-hubs-frequently-asked-questions"></a>使用 Azure 通知中樞推播通知：常見問題集 (FAQ)
## <a name="general"></a>一般
### <a name="what-is-the-resource-structure-of-notification-hubs"></a>通知中樞的資源結構為何？

Azure 通知中樞有兩個資源層級：中樞和命名空間。 中樞是單一推播資源，可包含一個應用程式的跨平台推播資訊。 命名空間是一個區域中多個中樞的集合。

建議對應是一個應用程式搭配一個命名空間。 在命名空間內，您可以有一個生產中樞來與生產應用程式搭配運作，以及一個測試中樞來與測試應用程式搭配運作，依此類推。

### <a name="what-is-the-price-model-for-notification-hubs"></a>「通知中樞」的價格模式為何？
如需最新的價格詳細資訊，請參閱[通知中樞價格]頁面。 通知中樞會以命名空間層級計費。 (如需命名空間的定義，請參閱＜什麼是通知中樞的資源結構？＞) 通知中樞提供三個層級︰

* **免費**：對探索推播功能來說，此層級是很好的起點。 但不建議用於生產應用程式。 每個月可取得 500 個裝置和 1 百萬個推播 (每個命名空間)，不包含服務等級協定 (SLA) 的保證。
* **基本**：較小型的生產應用程式建議使用此層級 (或標準層級)。 每月可取得 200,000 個裝置和 1 千萬個推播 (以每個命名空間為基準)。 其中包含配額成長選項。
* **標準**：中大型的生產應用程式建議使用此層級。 每月可取得 1 千萬個裝置和 1 千萬個推播 (以每個命名空間為基準)。 其中包含配額增加選項和豐富的遙測功能。

標準層級功能：
* **豐富的遙測**：您可以使用每個訊息遙測的通知中樞，來追蹤任何推播要求和平台通知系統的意見反應以進行偵錯。
* **多租用戶**：您可以在命名空間層級上使用平台通知系統認證。 此選項可讓您在相同的命名空間內輕鬆地將租用戶分割成多個中樞。
* **排定推播**：您可以排定在任何時間傳送通知。

### <a name="what-is-the-notification-hubs-sla"></a>什麼是通知中樞 SLA？
以基本和標準通知中樞層級來說，若應用程式已正確設定，則可傳送推播通知或執行註冊管理作業的時間至少有 99.9 %。 若要深入了解 SLA，請前往[通知中樞 SLA](https://azure.microsoft.com/support/legal/sla/notification-hubs/) 頁面。

> [!NOTE]
> 因為推播通知依存於第三方平台通知系統 (如 Apple APNS 和 Google FCM)，因此這些訊息的傳遞沒有 SLA 保證。 通知中樞將批次傳送至平台通知系統 (SLA 保證) 後，就由平台通知系統負責傳遞推播 (無 SLA 保證)。

### <a name="which-customers-are-using-notification-hubs"></a>客戶如何使用通知中樞？
許多客戶都使用通知中樞。 此處列出一些值得注意的︰

* Sochi 2014︰兩周內有數百個感興趣的群組、3 百萬個以上的裝置和 1.5 億個以上的通知需進行分派。 [案例研究：Sochi]
* Skanska：[案例研究：Skanska]
* Seattle Times：[案例研究：Seattle Times]
* Mural.ly：[案例研究：Mural.ly]
* 7Digital：[案例研究：7Digital]
* Bing 應用程式：1 千萬部以上的裝置每日傳送 3 百萬則通知。

### <a name="how-do-i-upgrade-or-downgrade-my-hub-or-namespace-to-a-different-tier"></a>如何將中樞或命名空間升級或降級為不同層級？
移至  **[Azure 入口網站]** > **通知中樞命名空間**或**通知中樞**。 選取您要更新的資源，並移至**定價層**。 請注意下列需求：

* 更新後的定價層會套用至您正在使用的命名空間中所有中樞。
* 如果您要降級至某一層，而您的裝置數量超過該層的限制，則必須要先刪除裝置才能降級。


## <a name="design-and-development"></a>設計與開發
### <a name="which-server-side-platforms-do-you-support"></a>支援哪些伺服器端平台？
伺服器 SDK 適用於 .NET、Java、Node.js、PHP 和 Python。 通知中樞 API 是以 REST 介面為根據，因此如果您要使用不同平台或不想要額外的相依性，則可以直接使用 REST API。 如需詳細資訊，請移至[通知中樞 REST API]頁面。

### <a name="which-client-platforms-do-you-support"></a>支援哪些用戶端平台？
推播通知支援 [iOS](notification-hubs-ios-apple-push-notification-apns-get-started.md)、[Android](notification-hubs-android-push-notification-google-gcm-get-started.md)[Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)[Windows Phone](notification-hubs-windows-mobile-push-notifications-mpns.md)[Kindle](notification-hubs-kindle-amazon-adm-push-notification.md)[Android China (由百度開發)](notification-hubs-baidu-china-android-notifications-get-started.md)、Xamarin ([iOS](xamarin-notification-hubs-ios-push-notification-apns-get-started.md)和 [Android](xamarin-notification-hubs-push-notifications-android-gcm.md))、[Chrome Apps](notification-hubs-chrome-push-notifications-get-started.md)和 [Safari](https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToSafari)。 如需詳細資訊，請移至[通知中樞入門教學課程]頁面。

### <a name="do-you-support-text-message-email-or-web-notifications"></a>你們是否支援簡訊、電子郵件、或 Web 通知？
通知中樞主要是設計來傳送通知給行動裝置應用程式。 不提供電子郵件或簡訊功能。 但是您可以將提供這些功能的第三方平台與「通知中樞」整合，以使用 [Mobile Apps] 來傳送原生推播通知。

通知中樞也不提供現成可用的瀏覽器中推播通知傳送服務。 客戶可以在支援的伺服器端平台上，使用 SignalR 來實作此功能。 如果您想要在 Chrome 沙箱中將通知傳送至瀏覽器應用程式，請參閱 [Chrome 應用程式教學課程]。

### <a name="how-are-mobile-apps-and-azure-notification-hubs-related-and-when-do-i-use-them"></a>Mobile Apps 和 Azure 通知中樞如何相關？以及何時使用這兩者？
如果您有現有的行動裝置應用程式後端，而且您只想要新增傳送推播通知的功能，則您可以使用 Azure 通知中樞。 如果您想要從頭開始設定行動裝置應用程式後端，請考慮使用 Azure App service 的 Mobile Apps功能。 行動裝置應用程式會自動佈建通知中樞，讓您能夠輕鬆地從行動裝置應用程式後端傳送推播通知。 Mobile Apps 的價格包括通知中樞的基本費用。 只有超過包含的推播時才需付費。 如需費用的詳細資訊，請移至[App Service 價格]頁面。

### <a name="how-many-devices-can-i-support-if-i-send-push-notifications-via-notification-hubs"></a>如果透過通知中樞傳送推播通知，可以支援多少個裝置？
如需支援的裝置數目詳細資訊，請參閱[通知中樞價格]頁面。

如果您需要支援超過 1 千萬個已註冊的裝置，請直接 [與我們連絡](https://azure.microsoft.com/overview/contact-us/) ，我們將協助您調整您的解決方案。

### <a name="how-many-push-notifications-can-i-send-out"></a>我可以傳送多少推播通知？
端視選取的層次而定，Azure 通知朱書會自動根據通知數目相應增加透過系統傳送的通知數目。

> [!NOTE]
> 整體使用成本可能會根據目前提供的推播通知數目而增加。 請務必留意[通知中樞價格]頁面上概述的層級限制。
> 
> 

我們的客戶每天使用通知中樞來傳送數百萬則推播通知。 只要使用 Azure 通知中樞，您就不需要採取任何特別動作來調整推播通知可及範圍。

### <a name="how-long-does-it-take-for-sent-push-notifications-to-reach-my-device"></a>送出的推播通知多久才會到達我的裝置？
在一般使用案例中，Azure 通知中樞能在 1 分鐘處理至少 1 百萬個推播通知傳送作業 ，能處理的連入負載數目大致上一樣。 此速率可能視標記數目、連入傳送作業本質和其他外部因子而異。

在預估的傳送時間內，服務能計算每個平台的目標數，並根據已註冊的標記或標記運算式將訊息路由傳送到推播通知服務 (PNS) 。 PNS 負責將通知傳送至裝置。

PNS 不保證任何傳送通知的 SLA。 不過，大部分的推播通知皆會在傳送到通知中樞後的幾分鐘內，傳遞至目標裝置 (通常是 10 分鐘內)。 有些通知可能需要更多時間。

> [!NOTE]
> Azure 通知中樞有一個原則，可捨棄無法在 30 分鐘內傳送到 PNS 的任何推播通知。 發生此延遲的原因有數種，但最常見的原因是 PNS 正在為您的應用程式進行節流處理。
> 
> 

### <a name="is-there-any-latency-guarantee"></a>是否有任何延遲保證？
因為推播通知的本質 (由外部平台特定 PNS 所傳送)，我們無法提供延遲保證。 一般而言，大部分的推播通知會在數分鐘內傳送到目標裝置。

### <a name="what-do-i-need-to-consider-when-designing-a-solution-with-namespaces-and-notification-hubs"></a>設計具有命名空間與通知中樞的解決方案時，必須考量哪些事項？
#### <a name="mobile-appenvironment"></a>行動裝置應用程式/環境
* 每個環境的每個行動裝置應用程式都使用一個通知中樞。
* 在多租用戶案例中，每個租用戶都應該有個別的中樞。
* 絕對不要在生產和測試環境中共用相同的通知中樞。 這種做法可能會在傳送通知時造成問題。 (Apple 提供「沙箱」與「生產推播」端點，而這兩者有個別的認證。)
* 根據預設，您可以透過 Azure 入口網站或 Visual Studio 中的 Azure 整合式元件，傳送測試通知到已註冊的裝置。 臨界值會設定為從註冊集區中隨機選取的 10 個裝置。

> [!NOTE]
> 如果您的中樞原本是使用 Apple 沙箱憑證設定，然後重新設定為使用 Apple 生產憑證，則原本的裝置權杖會變得無效。 無效的權杖會造成推播失敗。 請將生產與測試環境分開，並針對不同的環境使用不同的中樞。
> 
> 

#### <a name="pns-credentials"></a>PNS 認證
當行動裝置應用程式透過平台的開發人員入口網站 (例如 Apple 或 Google) 進行註冊時，系統會傳送應用程式識別碼和安全性權杖。 應用程式後端會將這些權杖提供給平台的 PNS，藉此推播通知就可以傳送至裝置。 安全性權杖可以是憑證的形式 (例如 Apple iOS 或 Windows Phone)，或是安全性金鑰的形式 (例如 Google、Android 或 Windows)。 並且必須在通知中樞加以設定。 設定通常是在通知中樞層級完成，但在多租用戶案例中，這也可以在命名空間層級完成。

#### <a name="namespaces"></a>命名空間
命名空間可以用於部署分組。 在多租用戶案例中，命名空間可以用來代表相同應用程式中所有租用戶的所有通知中樞。

#### <a name="geo-distribution"></a>地理分散
在推播通知案例中，地理分散不一定是關鍵。 傳送推播通知至裝置的不同 PNS (例如 APNS 或 GCM) 並非平均分散。

如果您有一個在全球使用的應用程式，您可以透過在世界上不同的 Azure 區域中使用通知中樞，以在不同命名空間中建立中樞。

> [!NOTE]
> 但我們不建議此安排，因為這會增加您的管理成本，特別是對註冊而言。 建議只在確實有此需求時這麼做。
> 
> 

### <a name="should-i-do-registrations-from-the-app-back-end-or-directly-through-client-devices"></a>我應該從應用程式後端註冊或直接透過用戶端裝置註冊？
如果您必須在建立註冊前進行驗證，則從應用程式後端註冊較為實用。 而且當您必須基於應用程式邏輯，讓應用程式後端建立或修改標記時，這些註冊也十分實用。 如需詳細資訊，請移至[後端註冊指引]與[後端註冊指引 2] 頁面。

### <a name="what-is-the-push-notification-delivery-security-model"></a>什麼是推播通知傳送資訊安全模型？
Azure 通知中樞使用[共用存取簽章](../storage/common/storage-dotnet-shared-access-signature-part-1.md) 型資訊安全模型。 您可以在根命名空間層級或更細微的通知中樞層級使用共用存取簽章權杖。 共用存取簽章權杖可設定為依循不同授權規則，例如傳送訊息權限或接聽通知權限。 如需詳細資訊，請參閱[通知中樞資訊安全模型]文件。

### <a name="how-should-i-handle-sensitive-payload-in-push-notifications"></a>應如何處理推播通知中的機密承載？
所有通知皆是由平台的 PNS 傳送至目標裝置。 通知傳送給 Azure 通知中樞，並在進行處理後散佈至各別的 PNS。

傳送者與 Azure 通知中樞之間的連線以及 Azure 通知中樞與 PNS 之間的連線都使用 HTTPS。

> [!NOTE]
> Azure 通知中樞不會以任何方式記錄訊息的承載。
> 
> 

若要傳送機密乘載，我們建議您使用「安全推播」模式。 傳送者會傳送 Ping 通知與訊息識別碼給裝置 (不含機密承載)。 當裝置上的應用程式收到此承載時，就能夠直接呼叫安全 API 以擷取訊息詳細資料。 您可以在[通知中樞安全推播教學課程]頁面上，取得如何實作此模式的指南。

## <a name="operations"></a>作業
### <a name="what-support-is-provided-for-disaster-recovery"></a>我可以得到哪些災害復原支援？
我們在我們這邊提供中繼資料災害復原範圍 (通知中樞名稱、連接字串和其他重要資訊)。 觸發災害復原案例後，註冊資料是通知中樞基礎結構中唯一會遺失的區段。 您必須實作解決方案，將此資料重新填入到復原後的新中樞：

1. 在不同的資料中心建立次要通知中樞。 我們建議您一開始就建立通知中樞，以避免災害復原事件和影響您的管理功能。 您也可以在災害復原事件發生時建立一個通知中樞。

2. 將來自主要通知中樞的註冊移往次要通知中樞。 我們不建議您嘗試在兩個中樞上維護註冊，並且在收到註冊時讓兩個中樞進行同步。 此做法無法正確運作，因為註冊有在 PNS 端到期的固有傾向。 當我們收到有關已到期或無效註冊的 PNS 回應時，通知中樞會將它們清除。  

我們有兩個應用程式後端的建議︰

* 使用在本身後端維護一組指定註冊的應用程式後端。 然後就可在次要通知中樞執行大量插入作業。

* 使用會從主要通知中樞定期取得註冊傾印作為備份的應用程式後端。 然後就可在次要通知中樞執行大量插入作業。

> [!NOTE]
> [註冊匯出/匯入]文件會說明標準層中提供的註冊匯出/匯入功能。
> 
> 

如果您沒有後端，應用程式在目標裝置上啟動時，會在次要通知中樞中執行新的註冊。 最後，所有作用中的裝置都會在次要通知中樞中進行註冊。

但如果裝置上有未開啟的應用程式時，會有一段期間收不到通知。

### <a name="is-there-audit-log-capability"></a>是否有稽核記錄功能？
所有通知中樞管理作業移至作業記錄，其公開於 [Azure 入口網站]中。

## <a name="monitoring-and-troubleshooting"></a>監視與疑難排解
### <a name="what-troubleshooting-capabilities-are-available"></a>可用的疑難排解功能有哪些？
Azure 通知中樞提供數個功能以進行疑難排解，特別是在已捨棄通知的最常見案例中。 如需詳細資訊，請參閱[通知中樞疑難排解]技術白皮書。

### <a name="what-telemetry-features-are-available"></a>可用的遙測功能有哪些？
您可透過 Azure 通知中樞，在 [Azure 入口網站]中檢視遙測資料。 您可以在[通知中樞計量]頁面上取得計量的詳細資料。

> [!NOTE]
> 成功通知就是表示推播通知已傳送到外部 PNS (例如 Apple 的 APNS 或 Google 的 GCM)。 PNS 負責將通知傳送至目標裝置。 PNS 通常不會向第三方公開計量。  
> 
> 

我們還提供以程式設計方式匯出遙測資料的功能 (在標準層中)。 如需詳細資訊，請參閱[通知中樞計量範例]。

[Azure 入口網站]: https://portal.azure.com
[通知中樞價格]: http://azure.microsoft.com/pricing/details/notification-hubs/
[Notification Hubs SLA]: http://azure.microsoft.com/support/legal/sla/
[案例研究：Sochi]: https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=7942
[案例研究：Skanska]: https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=5847
[案例研究：Seattle Times]: https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=8354
[案例研究：Mural.ly]: https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=11592
[案例研究：7Digital]: https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=3684
[通知中樞 REST API]: https://msdn.microsoft.com/library/azure/dn530746.aspx
[通知中樞入門教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-ios-get-started/
[Chrome 應用程式教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-chrome-get-started/
[Mobile Services Pricing]: http://azure.microsoft.com/pricing/details/mobile-services/
[後端註冊指引]: https://msdn.microsoft.com/library/azure/dn743807.aspx
[後端註冊指引 2]: https://msdn.microsoft.com/library/azure/dn530747.aspx
[通知中樞資訊安全模型]: https://msdn.microsoft.com/library/azure/dn495373.aspx
[通知中樞安全推播教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-aspnet-backend-ios-secure-push/
[通知中樞疑難排解]: http://azure.microsoft.com/documentation/articles/notification-hubs-diagnosing/
[通知中樞計量]: https://msdn.microsoft.com/library/dn458822.aspx
[通知中樞計量範例]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/FetchNHTelemetryInExcel
[註冊匯出/匯入]: https://msdn.microsoft.com/library/dn790624.aspx
[Azure 入口網站]: https://portal.azure.com
[complete samples]: https://github.com/Azure/azure-notificationhubs-samples
[Mobile Apps]: https://azure.microsoft.com/services/app-service/mobile/
[App Service 價格]: https://azure.microsoft.com/pricing/details/app-service/
