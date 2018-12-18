---
title: 如何透過 Python 使用通知中樞
description: 了解如何透過 Python 後端使用 Azure 通知中樞。
services: notification-hubs
documentationcenter: ''
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 5640dd4a-a91e-4aa0-a833-93615bde49b4
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: python
ms.devlang: php
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: b0aa61c86b08a4fe2eddb5eb1fb5b33c4cb901a0
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49386582"
---
# <a name="how-to-use-notification-hubs-from-python"></a>如何透過 Python 使用通知中樞
[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

您可以使用通知中樞 REST 介面，透過 Java/PHP/Python/Ruby 後端來存取所有通知中樞功能，如 MSDN 文章[通知中樞 REST API](http://msdn.microsoft.com/library/dn223264.aspx)所述。

> [!NOTE]
> 這是在 Python 實作通知傳送的範例參考實作，並非正式支援的通知中樞 Python SDK。
> 
> 這個範例是使用 Python 3.4 撰寫的。
> 
> 

本文將說明如何：

* 在 Python 中建置通知中樞功能的 REST 用戶端。
* 使用 Python 介面傳送通知到通知中樞 REST API。 
* 取得 HTTP REST 要求/回應的傾印以用於偵錯/教學用途。 

您可以遵循選定行動平台的 [開始使用教學課程](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md) 來實作 Python 的後端部分。

> [!NOTE]
> 此範例的範圍僅限於傳送通知，不會執行任何註冊管理。
> 
> 

## <a name="client-interface"></a>用戶端介面
主要用戶端介面提供的方法與 [.NET 通知中樞 SDK](http://msdn.microsoft.com/library/jj933431.aspx)中的方法相同。 這個介面可讓您直接轉換此網站目前所提供，以及由網際網路社群所貢獻的所有教學課程和範例。

您可在 [Python REST 包裝函式範例]中找到所有可用的程式碼。

例如，若要建立用戶端：

    isDebug = True
    hub = NotificationHub("myConnectionString", "myNotificationHubName", isDebug)

傳送 Windows 快顯通知：

    wns_payload = """<toast><visual><binding template=\"ToastText01\"><text id=\"1\">Hello world!</text></binding></visual></toast>"""
    hub.send_windows_notification(wns_payload)

## <a name="implementation"></a>實作
如果尚未這麼做，請遵循[開始使用教學課程]中的指示，一路進行到您必須實作後端的最後一節。

您可以在 [MSDN](http://msdn.microsoft.com/library/dn530746.aspx)上找到所有實作完整 REST 包裝函式的詳細資料。 本節將針對存取通知中樞 REST 端點和傳送通知所需主要步驟的 Python 實作進行說明

1. 解析連接字串
2. 產生授權權杖
3. 使用 HTTP REST API 傳送通知

### <a name="parse-the-connection-string"></a>解析連接字串
以下是實作其建構函式可解析連接字串之用戶端的主要類別：

    class NotificationHub:
        API_VERSION = "?api-version=2013-10"
        DEBUG_SEND = "&test"

        def __init__(self, connection_string=None, hub_name=None, debug=0):
            self.HubName = hub_name
            self.Debug = debug

            # Parse connection string
            parts = connection_string.split(';')
            if len(parts) != 3:
                raise Exception("Invalid ConnectionString.")

            for part in parts:
                if part.startswith('Endpoint'):
                    self.Endpoint = 'https' + part[11:]
                if part.startswith('SharedAccessKeyName'):
                    self.SasKeyName = part[20:]
                if part.startswith('SharedAccessKey'):
                    self.SasKeyValue = part[16:]


### <a name="create-security-token"></a>建立安全性權杖
您可以在 [此處](http://msdn.microsoft.com/library/dn495627.aspx)找到建立安全性權杖的詳細資料。
將下列方法新增至 **NotificationHub** 類別，以依據目前要求的 URI，以及從連接字串擷取的認證來建立權杖。

    @staticmethod
    def get_expiry():
        # By default returns an expiration of 5 minutes (=300 seconds) from now
        return int(round(time.time() + 300))

    @staticmethod
    def encode_base64(data):
        return base64.b64encode(data)

    def sign_string(self, to_sign):
        key = self.SasKeyValue.encode('utf-8')
        to_sign = to_sign.encode('utf-8')
        signed_hmac_sha256 = hmac.HMAC(key, to_sign, hashlib.sha256)
        digest = signed_hmac_sha256.digest()
        encoded_digest = self.encode_base64(digest)
        return encoded_digest

    def generate_sas_token(self):
        target_uri = self.Endpoint + self.HubName
        my_uri = urllib.parse.quote(target_uri, '').lower()
        expiry = str(self.get_expiry())
        to_sign = my_uri + '\n' + expiry
        signature = urllib.parse.quote(self.sign_string(to_sign))
        auth_format = 'SharedAccessSignature sig={0}&se={1}&skn={2}&sr={3}'
        sas_token = auth_format.format(signature, expiry, self.SasKeyName, my_uri)
        return sas_token

### <a name="send-a-notification-using-http-rest-api"></a>使用 HTTP REST API 傳送通知
首先，讓我們先定義呈現通知的類別。

    class Notification:
        def __init__(self, notification_format=None, payload=None, debug=0):
            valid_formats = ['template', 'apple', 'gcm', 'windows', 'windowsphone', "adm", "baidu"]
            if not any(x in notification_format for x in valid_formats):
                raise Exception(
                    "Invalid Notification format. " +
                    "Must be one of the following - 'template', 'apple', 'gcm', 'windows', 'windowsphone', 'adm', 'baidu'")

            self.format = notification_format
            self.payload = payload

            # array with keynames for headers
            # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
            # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry
            # in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
            self.headers = None

此類別是範本通知的原生通知主體或一組屬性的容器；為標頭集合，其包含格式 (原生平台或範本)，以及平台特定屬性 (如 Apple 到期屬性和 WNS 標頭)。

請參閱[通知中樞 REST API 文件](http://msdn.microsoft.com/library/dn495827.aspx)及特定通知平台的格式，以取得所有可用選項。

透過此類別，現在可以在 **NotificationHub** 類別內寫入傳送通知方法。

    def make_http_request(self, url, payload, headers):
        parsed_url = urllib.parse.urlparse(url)
        connection = http.client.HTTPSConnection(parsed_url.hostname, parsed_url.port)

        if self.Debug > 0:
            connection.set_debuglevel(self.Debug)
            # adding this querystring parameter gets detailed information about the PNS send notification outcome
            url += self.DEBUG_SEND
            print("--- REQUEST ---")
            print("URI: " + url)
            print("Headers: " + json.dumps(headers, sort_keys=True, indent=4, separators=(' ', ': ')))
            print("--- END REQUEST ---\n")

        connection.request('POST', url, payload, headers)
        response = connection.getresponse()

        if self.Debug > 0:
            # print out detailed response information for debugging purpose
            print("\n\n--- RESPONSE ---")
            print(str(response.status) + " " + response.reason)
            print(response.msg)
            print(response.read())
            print("--- END RESPONSE ---")

        elif response.status != 201:
            # Successful outcome of send message is HTTP 201 - Created
            raise Exception(
                "Error sending notification. Received HTTP code " + str(response.status) + " " + response.reason)

        connection.close()

    def send_notification(self, notification, tag_or_tag_expression=None):
        url = self.Endpoint + self.HubName + '/messages' + self.API_VERSION

        json_platforms = ['template', 'apple', 'gcm', 'adm', 'baidu']

        if any(x in notification.format for x in json_platforms):
            content_type = "application/json"
            payload_to_send = json.dumps(notification.payload)
        else:
            content_type = "application/xml"
            payload_to_send = notification.payload

        headers = {
            'Content-type': content_type,
            'Authorization': self.generate_sas_token(),
            'ServiceBusNotification-Format': notification.format
        }

        if isinstance(tag_or_tag_expression, set):
            tag_list = ' || '.join(tag_or_tag_expression)
        else:
            tag_list = tag_or_tag_expression

        # add the tags/tag expressions to the headers collection
        if tag_list != "":
            headers.update({'ServiceBusNotification-Tags': tag_list})

        # add any custom headers to the headers collection that the user may have added
        if notification.headers is not None:
            headers.update(notification.headers)

        self.make_http_request(url, payload_to_send, headers)

    def send_apple_notification(self, payload, tags=""):
        nh = Notification("apple", payload)
        self.send_notification(nh, tags)

    def send_gcm_notification(self, payload, tags=""):
        nh = Notification("gcm", payload)
        self.send_notification(nh, tags)

    def send_adm_notification(self, payload, tags=""):
        nh = Notification("adm", payload)
        self.send_notification(nh, tags)

    def send_baidu_notification(self, payload, tags=""):
        nh = Notification("baidu", payload)
        self.send_notification(nh, tags)

    def send_mpns_notification(self, payload, tags=""):
        nh = Notification("windowsphone", payload)

        if "<wp:Toast>" in payload:
            nh.headers = {'X-WindowsPhone-Target': 'toast', 'X-NotificationClass': '2'}
        elif "<wp:Tile>" in payload:
            nh.headers = {'X-WindowsPhone-Target': 'tile', 'X-NotificationClass': '1'}

        self.send_notification(nh, tags)

    def send_windows_notification(self, payload, tags=""):
        nh = Notification("windows", payload)

        if "<toast>" in payload:
            nh.headers = {'X-WNS-Type': 'wns/toast'}
        elif "<tile>" in payload:
            nh.headers = {'X-WNS-Type': 'wns/tile'}
        elif "<badge>" in payload:
            nh.headers = {'X-WNS-Type': 'wns/badge'}

        self.send_notification(nh, tags)

    def send_template_notification(self, properties, tags=""):
        nh = Notification("template", properties)
        self.send_notification(nh, tags)

這些方法會傳送 HTTP POST 要求至通知中樞的 /messages 端點，並使用正確的主體和標頭來傳送通知。

### <a name="using-debug-property-to-enable-detailed-logging"></a>使用偵錯屬性啟用詳細的記錄
在初始化通知中樞時啟用偵錯屬性會寫出關於 HTTP 要求和回應傾印的詳細記錄資訊，以及詳細的通知訊息傳送結果。 [通知中樞 TestSend 屬性](https://docs.microsoft.com/previous-versions/azure/reference/dn495827(v=azure.100)) \(英文\) 會傳回關於通知傳送結果的詳細資訊。 若要使用它，請使用下列程式碼來初始化：

    hub = NotificationHub("myConnectionString", "myNotificationHubName", isDebug)

通知中樞傳送要求 HTTP URL 會附加 "test" 查詢字串做為結果。 

## <a name="complete-tutorial"></a>完成教學課程
現在您可以透過從 Python 後端傳送通知，來完成開始使用教學課程。

初始化您的通知中樞用戶端 (請依 [開始使用教學課程]中的指示替換連接字串和中心名稱)：

    hub = NotificationHub("myConnectionString", "myNotificationHubName")

然後根據您的目標行動平台新增傳送程式碼。 此範例也會新增更高層級的方法，以依據平台傳送通知，例如 send_windows_notification (適用於 Windows)、send_apple_notification (適用於 Apple) 等。 

### <a name="windows-store-and-windows-phone-81-non-silverlight"></a>Windows 市集和 Windows Phone 8.1 (非 Silverlight)
    wns_payload = """<toast><visual><binding template=\"ToastText01\"><text id=\"1\">Test</text></binding></visual></toast>"""
    hub.send_windows_notification(wns_payload)

### <a name="windows-phone-80-and-81-silverlight"></a>Windows Phone 8.0 和 8.1 Silverlight
    hub.send_mpns_notification(toast)

### <a name="ios"></a>iOS
    alert_payload = {
        'data':
            {
                'msg': 'Hello!'
            }
    }
    hub.send_apple_notification(alert_payload)

### <a name="android"></a>Android
    gcm_payload = {
        'data':
            {
                'msg': 'Hello!'
            }
    }
    hub.send_gcm_notification(gcm_payload)

### <a name="kindle-fire"></a>Kindle Fire
    adm_payload = {
        'data':
            {
                'msg': 'Hello!'
            }
    }
    hub.send_adm_notification(adm_payload)

### <a name="baidu"></a>Baidu
    baidu_payload = {
        'data':
            {
                'msg': 'Hello!'
            }
    }
    hub.send_baidu_notification(baidu_payload)

執行 Python 程式碼應會產生一則顯示於目標裝置的通知。

## <a name="examples"></a>範例：
### <a name="enabling-debug-property"></a>啟用偵錯屬性
若在初始化 NotificationHub 時啟用偵錯旗標，您會看到詳細的 HTTP 要求和回應傾印，還有類似以下的 NotificationOutcome，您可從中了解在要求中傳送的 HTTP 標頭，以及從通知中樞接收到的 HTTP 回應：![][1]

例如，您會看到詳細的通知中樞結果。 

* 訊息成功傳送至推播通知服務的時間。 
  
        <Outcome>The Notification was successfully sent to the Push Notification System</Outcome>
* 如果找不到任何推播通知的目標，您可能會看到下列輸出作為回應 (這表示找不到可傳遞通知的註冊，可能是因為註冊有一些不相符的標記)
  
        '<NotificationOutcome xmlns="http://schemas.microsoft.com/netservices/2010/10/servicebus/connect" xmlns:i="http://www.w3.org/2001/XMLSchema-instance"><Success>0</Success><Failure>0</Failure><Results i:nil="true"/></NotificationOutcome>'

### <a name="broadcast-toast-notification-to-windows"></a>廣播快顯通知給 Windows
請注意當您傳送廣播快顯通知給 Windows 用戶端時所送出的標頭。 

    hub.send_windows_notification(wns_payload)

![][2]

### <a name="send-notification-specifying-a-tag-or-tag-expression"></a>傳送指定標記 (或標記運算式) 的通知
請注意到新增至 HTTP 要求的 Tags HTTP 標頭 (在下列範例中，通知只會被傳送至含有 'sports' 承載的註冊)

    hub.send_windows_notification(wns_payload, "sports")

![][3]

### <a name="send-notification-specifying-multiple-tags"></a>傳送指定多個標記的通知
請注意傳送多個標記時的 Tags HTTP 標頭如何變更 

    tags = {'sports', 'politics'}
    hub.send_windows_notification(wns_payload, tags)

![][4]

### <a name="templated-notification"></a>樣板化通知
請注意，Format HTTP 標頭會變更，承載主體則會傳送為 HTTP 要求主體的一部分：

**用戶端 - 註冊的範本**

        var template =
                        @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(greeting_en)</text></binding></visual></toast>";

**伺服器端 - 傳送承載**

        template_payload = {'greeting_en': 'Hello', 'greeting_fr': 'Salut'}
        hub.send_template_notification(template_payload)

![][5]

## <a name="next-steps"></a>後續步驟
本主題會說明如何針對通知中樞建立 Python REST 用戶端。 您可以在這裡執行下列動作：

* 下載完整的 [Python REST 包裝函式範例]，其中包含本文章的所有程式碼。
* 繼續了解 [即時新聞教學課程]
* 繼續了解 [當地語系化新聞教學課程]

<!-- URLs -->
[Python REST 包裝函式範例]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-python
[開始使用教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-windows-store-dotnet-get-started/
[即時新聞教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-windows-store-dotnet-send-breaking-news/
[當地語系化新聞教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-windows-store-dotnet-send-localized-breaking-news/

<!-- Images. -->
[1]: ./media/notification-hubs-python-backend-how-to/DetailedLoggingInfo.png
[2]: ./media/notification-hubs-python-backend-how-to/BroadcastScenario.png
[3]: ./media/notification-hubs-python-backend-how-to/SendWithOneTag.png
[4]: ./media/notification-hubs-python-backend-how-to/SendWithMultipleTags.png
[5]: ./media/notification-hubs-python-backend-how-to/TemplatedNotification.png

