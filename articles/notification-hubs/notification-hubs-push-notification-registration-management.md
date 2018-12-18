---
title: 註冊管理
description: 本主題說明如何向通知中樞註冊裝置以接收推播通知。
services: notification-hubs
documentationcenter: .net
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: fd0ee230-132c-4143-b4f9-65cef7f463a1
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 7f9052da066fcc0021151bf3b547484859cf216d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33776918"
---
# <a name="registration-management"></a>註冊管理
## <a name="overview"></a>概觀
本主題說明如何向通知中樞註冊裝置以接收推播通知。 本主題會概要說明註冊，然後介紹註冊裝置的兩個主要模式：直接從裝置向通知中樞註冊，以及透過應用程式後端註冊。 

## <a name="what-is-device-registration"></a>什麼是裝置註冊
向「通知中樞」註冊裝置是藉由使用 [註冊] 或 [安裝] 來完成。

#### <a name="registrations"></a>註冊
註冊會將裝置的「平台通知服務」(PNS) 控制代碼與標記 (以及也可能與範本) 建立關聯。 PNS 控制代碼可能是 ChannelURI、裝置權杖或 GCM 註冊識別碼。標記是用來將通知路由至一組正確的裝置控制代碼。 如需詳細資訊，請參閱 [路由與標記運算式](notification-hubs-tags-segment-push-message.md)。 範本是用來實作每一註冊的轉換。 如需詳細資訊，請參閱 [範本](notification-hubs-templates-cross-platform-push-messages.md)。

#### <a name="installations"></a>安裝
安裝是增強型的註冊，包含一組推播相關的屬性。 它是註冊您的裝置最新最好的方法。 不過，目前用戶端 .NET SDK([適用於後端作業的通知中樞 SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)) 不支援此種安裝。  這表示如果您要從用戶端裝置本身註冊，您必須使用 [通知中樞 REST API](https://msdn.microsoft.com/library/mt621153.aspx) 方法來支援安裝。 如果您使用後端服務，您應該能夠使用 [適用於後端作業的通知中樞 SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)。

以下是使用安裝的一些主要優點：

* 建立或更新安裝是完全等冪的。 因此您可以重試它，而不需顧慮重複註冊的情況。
* 安裝模型可讓您更容易進行個別推播 (以特定裝置為目標)。 在每個安裝型註冊，都會自動新增一個系統標記 **"$InstallationId:[installationId]"** 。 因此，您不需編寫任何額外的程式碼，即可對此標記進行傳送呼叫來以特定裝置做為目標。
* 使用安裝也可讓您進行部分註冊更新。 要求部分安裝更新時，是使用 [JSON-Patch 標準](https://tools.ietf.org/html/rfc6902)以 PATCH 方法來要求。 當您想要更新註冊的相關標記時，這會特別有用。 您不需要移除整個註冊，然後再次重新傳送所有先前的標記。

安裝可以包含下列屬性。 如需完整的安裝屬性清單，請參閱[使用 REST API 建立或覆寫安裝](https://msdn.microsoft.com/library/azure/mt621153.aspx)或[安裝屬性](https://msdn.microsoft.com/library/azure/microsoft.azure.notificationhubs.installation_properties.aspx)。

    // Example installation format to show some supported properties
    {,
        installationId: "",
        expirationTime: "",
        tags: [],
        platform: "",
        pushChannel: "",
        ………
        templates: {
            "templateName1" : {
                body: "",
                tags: [] },
            "templateName2" : {
                body: "",
                // Headers are for Windows Store only
                headers: {
                    "X-WNS-Type": "wns/tile" }
                tags: [] }
        },
        secondaryTiles: {
            "tileId1": {
                pushChannel: "",
                tags: [],
                templates: {
                    "otherTemplate": {
                        bodyTemplate: "",
                        headers: {
                            ... }
                        tags: [] }
                }
            }
        }
    }



請務必注意，註冊與安裝預設不會到期。

註冊與安裝必須包含每個裝置/通道的有效 PNS 控制代碼。 由於 PNS 控制代碼只能在裝置上的用戶端 app 中取得，因此有一種模式是直接在該裝置上使用用戶端 app 進行註冊。 另一方面，與標記相關的安全性考量和商務邏輯可能會需要您在 app 後端管理裝置註冊。 

#### <a name="templates"></a>範本
如果您想要使用[範本](notification-hubs-templates-cross-platform-push-messages.md)，裝置安裝也保有與該裝置關聯且採用 JSON 格式的所有範本 (請參閱上面的範例)。 範本名稱可協助將目標指向相同裝置的不同範本。

每個範本名稱皆對應到一個範本主體和一組選擇性的標記。 此外，每個平台可以有額外的範本屬性。 就 Windows 市集 (使用 WNS) 和 Windows Phone 8 (使用 MPNS) 而言，一組額外的標頭可以是範本的一部分。 如果是 APN，您可以將到期屬性設定為常數或範本運算式。 如需完整的安裝屬性清單，請參閱 [使用 REST 來建立或覆寫安裝](https://msdn.microsoft.com/library/azure/mt621153.aspx) 主題。

#### <a name="secondary-tiles-for-windows-store-apps"></a>Windows 市集應用程式的次要磚
就「Windows 市集」用戶端應用程式而言，將通知傳送給次要磚與將通知傳送給主要磚一樣。 在安裝中也支援此行為。 次要磚具有不同的 ChannelUri，您用戶端 app 上的 SDK 會在背景處理此 ChannelUri。

SecondaryTiles 字典使用的 TileId 會與在「Windows 市集」應用程式中建立 SecondaryTiles 時使用的 TileId 相同。
如同主要 ChannelUri，次要磚的 ChannelUri 也會隨時變更。 為了讓通知中樞內的安裝保持更新，裝置必須以次要磚的 ChannelUri 來重新整理它們。

## <a name="registration-management-from-the-device"></a>從裝置管理註冊
從用戶端 app 管理裝置註冊時，後端只負責傳送通知。 用戶端 app 會讓 PNS 控制代碼保持在最新狀態，並且會註冊標記。 下圖說明這個模式。

![](./media/notification-hubs-registration-management/notification-hubs-registering-on-device.png)

裝置會先從 PNS 抓取 PNS 控制代碼，然後直接向通知中心進行註冊。 註冊成功之後，app 後端即可傳送以該註冊為目標的通知。 如需有關如何傳送通知的詳細資訊，請參閱 [路由與標記運算式](notification-hubs-tags-segment-push-message.md)。
在此情況下，您只使用「接聽」權限，從裝置存取通知中樞。 如需詳細資訊，請參閱[安全性](notification-hubs-push-notification-security.md)。

從裝置註冊是最簡單的方法，但有一些缺點。
第一個缺點是用戶端 app 只有在 app 處於使用中時，才能更新其標記。 舉例來說，如果使用者有兩個註冊球隊相關標記的裝置，當第一個裝置註冊另一個標記 (例如 Seahawks) 時，第二個裝置必須等到第二次執行時，才會收到有關 Seahawks 的通知。 更普遍來說，當標記受多個裝置影響時，從後端管理標記是一個較理想的選項。
從用戶端 app 管理註冊的第二個缺點是，由於 app 可能被入侵，因此如＜標記層級安全性＞一節所述，保護特定標記的註冊必須特別小心。

#### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-an-installation"></a>使用安裝從裝置向通知中樞註冊的範例程式碼
此時，只有使用 [通知中樞 REST API](https://msdn.microsoft.com/library/mt621153.aspx)才支援這種做法。

您也可以使用 [JSON-Patch 標準](https://tools.ietf.org/html/rfc6902) 以 PATCH 方法更新安裝。

    class DeviceInstallation
    {
        public string installationId { get; set; }
        public string platform { get; set; }
        public string pushChannel { get; set; }
        public string[] tags { get; set; }
    }

    private async Task<HttpStatusCode> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation,
         string hubName, string listenConnectionString)
    {
        if (deviceInstallation.installationId == null)
            return HttpStatusCode.BadRequest;

        // Parse connection string (https://msdn.microsoft.com/library/azure/dn495627.aspx)
        ConnectionStringUtility connectionSaSUtil = new ConnectionStringUtility(listenConnectionString);
        string hubResource = "installations/" + deviceInstallation.installationId + "?";
        string apiVersion = "api-version=2015-04";

        // Determine the targetUri that we will sign
        string uri = connectionSaSUtil.Endpoint + hubName + "/" + hubResource + apiVersion;

        //=== Generate SaS Security Token for Authorization header ===
        // See, https://msdn.microsoft.com/library/azure/dn495627.aspx
        string SasToken = connectionSaSUtil.getSaSToken(uri, 60);

        using (var httpClient = new HttpClient())
        {
            string json = JsonConvert.SerializeObject(deviceInstallation);

            httpClient.DefaultRequestHeaders.Add("Authorization", SasToken);

            var response = await httpClient.PutAsync(uri, new StringContent(json, System.Text.Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    string installationId = null;
    var settings = ApplicationData.Current.LocalSettings.Values;

    // If we have not stored a installation id in application data, create and store as application data.
    if (!settings.ContainsKey("__NHInstallationId"))
    {
        installationId = Guid.NewGuid().ToString();
        settings.Add("__NHInstallationId", installationId);
    }

    installationId = (string)settings["__NHInstallationId"];

    var deviceInstallation = new DeviceInstallation
    {
        installationId = installationId,
        platform = "wns",
        pushChannel = channel.Uri,
        //tags = tags.ToArray<string>()
    };

    var statusCode = await CreateOrUpdateInstallationAsync(deviceInstallation, 
                        "<HUBNAME>", "<SHARED LISTEN CONNECTION STRING>");

    if (statusCode != HttpStatusCode.Accepted)
    {
        var dialog = new MessageDialog(statusCode.ToString(), "Registration failed. Installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    else
    {
        var dialog = new MessageDialog("Registration successful using installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }



#### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration"></a>使用註冊從裝置向通知中樞註冊的範例程式碼
這些方法會建立或更新其所在呼叫位置的裝置註冊。 這表示為了更新控制代碼或標記，您必須覆寫整個註冊。 請記住，註冊是暫時性的，因此您應該一律要有一個可靠的存放區，內含特定裝置所需的目前標記。

    // Initialize the Notification Hub
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

    // The Device id from the PNS
    var pushChannel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    // If you are registering from the client itself, then store this registration id in device
    // storage. Then when the app starts, you can check if a registration id already exists or not before
    // creating.
    var settings = ApplicationData.Current.LocalSettings.Values;

    // If we have not stored a registration id in application data, store in application data.
    if (!settings.ContainsKey("__NHRegistrationId"))
    {
        // make sure there are no existing registrations for this push handle (used for iOS and Android)    
        string newRegistrationId = null;
        var registrations = await hub.GetRegistrationsByChannelAsync(pushChannel.Uri, 100);
        foreach (RegistrationDescription registration in registrations)
        {
            if (newRegistrationId == null)
            {
                newRegistrationId = registration.RegistrationId;
            }
            else
            {
                await hub.DeleteRegistrationAsync(registration);
            }
        }

        newRegistrationId = await hub.CreateRegistrationIdAsync();

        settings.Add("__NHRegistrationId", newRegistrationId);
    }

    string regId = (string)settings["__NHRegistrationId"];

    RegistrationDescription registration = new WindowsRegistrationDescription(pushChannel.Uri);
    registration.RegistrationId = regId;
    registration.Tags = new HashSet<string>(YourTags);

    try
    {
        await hub.CreateOrUpdateRegistrationAsync(registration);
    }
    catch (Microsoft.WindowsAzure.Messaging.RegistrationGoneException e)
    {
        settings.Remove("__NHRegistrationId");
    }


## <a name="registration-management-from-a-backend"></a>從後端管理註冊
從後端管理註冊需要撰寫額外的程式碼。 來自裝置的 app 必須在每次 app 啟動時，提供已更新的 PNS 控制代碼給後端 (連同標記和範本)，而後端必須在通知中樞上更新此控制代碼。 下圖說明這個設計。

![](./media/notification-hubs-registration-management/notification-hubs-registering-on-backend.png)

從後端管理註冊的優點包括：即使裝置上對應的 app 不是處於使用中，也能夠修改註冊的標記；以及能夠在將標記新增到用戶端 app 的註冊之前，先驗證該 app。

#### <a name="example-code-to-register-with-a-notification-hub-from-a-backend-using-an-installation"></a>使用安裝從後端向通知中樞註冊的範例程式碼
用戶端裝置仍會如先前一樣取得其 PNS 控制代碼及相關的安裝屬性，然後在可以執行註冊及授權標記等的後端上呼叫自訂 API。後端可以利用[適用於後端作業的通知中樞 SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)。

您也可以使用 [JSON-Patch 標準](https://tools.ietf.org/html/rfc6902) 以 PATCH 方法更新安裝。

    // Initialize the Notification Hub
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

    // Custom API on the backend
    public async Task<HttpResponseMessage> Put(DeviceInstallation deviceUpdate)
    {

        Installation installation = new Installation();
        installation.InstallationId = deviceUpdate.InstallationId;
        installation.PushChannel = deviceUpdate.Handle;
        installation.Tags = deviceUpdate.Tags;

        switch (deviceUpdate.Platform)
        {
            case "mpns":
                installation.Platform = NotificationPlatform.Mpns;
                break;
            case "wns":
                installation.Platform = NotificationPlatform.Wns;
                break;
            case "apns":
                installation.Platform = NotificationPlatform.Apns;
                break;
            case "gcm":
                installation.Platform = NotificationPlatform.Gcm;
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }


        // In the backend we can control if a user is allowed to add tags
        //installation.Tags = new List<string>(deviceUpdate.Tags);
        //installation.Tags.Add("username:" + username);

        await hub.CreateOrUpdateInstallationAsync(installation);

        return Request.CreateResponse(HttpStatusCode.OK);
    }


#### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration-id"></a>使用註冊識別碼從裝置向通知中樞註冊的範例程式碼
您可以從 app 後端，對註冊執行基本 CRUD 作業。 例如︰

    var hub = NotificationHubClient.CreateClientFromConnectionString("{connectionString}", "hubName");

    // create a registration description object of the correct type, e.g.
    var reg = new WindowsRegistrationDescription(channelUri, tags);

    // Create
    await hub.CreateRegistrationAsync(reg);

    // Get by id
    var r = await hub.GetRegistrationAsync<RegistrationDescription>("id");

    // update
    r.Tags.Add("myTag");

    // update on hub
    await hub.UpdateRegistrationAsync(r);

    // delete
    await hub.DeleteRegistrationAsync(r);


後端必須處理註冊更新之間的並行存取。 「服務匯流排」可提供開放式並行存取控制來管理註冊。 在 HTTP 層級，這是藉由在註冊管理作業上使用 ETag 來進行實作。 Microsoft SDK 會在背景使用這項功能，如果因並行存取而導致更新被拒，將會擲回例外狀況。 App 後端會負責處理這些例外狀況，並視需要重試更新。

