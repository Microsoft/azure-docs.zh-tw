---
title: 如何搭配使用通知中樞與 PHP
description: 了解如何從 PHP 後端使用 Azure 通知中樞。
services: notification-hubs
documentationcenter: ''
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 0156f994-96d0-4878-b07b-49b7be4fd856
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: php
ms.devlang: php
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 930da7cca312ac6233b337dd7ddac478c3bbee7b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
---
# <a name="how-to-use-notification-hubs-from-php"></a>如何從 PHP 使用通知中樞
[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

您可以使用通知中心 REST 介面，存取 Java/PHP/Ruby 後端的所有通知中心功能，如 MSDN 主題 [通知中心 REST API](http://msdn.microsoft.com/library/dn223264.aspx)中所述。

在本主題中，我們將說明如何：

* 在 PHP 中建置通知中心功能的 REST 用戶端；
* 依照 [開始使用教學課程](notification-hubs-ios-apple-push-notification-apns-get-started.md) ，針對您所選的行動平台，在 PHP 中實作後端部分。

## <a name="client-interface"></a>用戶端介面
主要用戶端介面可提供與 [.NET 通知中心 SDK](http://msdn.microsoft.com/library/jj933431.aspx)提供的相同方法，這可讓您直接轉譯本網站上目前所提供以及網際網路上社群所貢獻的所有教學課程和範例。

您可在 [PHP REST 包裝函式範例]中找到所有可用的程式碼。

例如，若要建立用戶端：

    $hub = new NotificationHub("connection string", "hubname");    

若要傳送 iOS 原生通知：

    $notification = new Notification("apple", '{"aps":{"alert": "Hello!"}}');
    $hub->sendNotification($notification, null);

## <a name="implementation"></a>實作
如果尚未這麼做，請遵循[開始使用教學課程]中的指示，一路進行到您必須實作後端的最後一節。
另外，若有需要，您可以使用 [PHP REST 包裝函式範例]的程式碼，並直接移至[完成教學課程](#complete-tutorial)一節。

您可以在 [MSDN](http://msdn.microsoft.com/library/dn530746.aspx)上找到所有實作完整 REST 包裝函式的詳細資料。 在本節中，我們將針對存取通知中心 REST 端點所需主要步驟的 PHP 實作進行說明：

1. 解析連接字串
2. 產生授權權杖
3. 執行 HTTP 呼叫

### <a name="parse-the-connection-string"></a>解析連接字串
以下是實作其建構函式可解析連接字串之用戶端的主要類別：

    class NotificationHub {
        const API_VERSION = "?api-version=2013-10";

        private $endpoint;
        private $hubPath;
        private $sasKeyName;
        private $sasKeyValue;

        function __construct($connectionString, $hubPath) {
            $this->hubPath = $hubPath;

            $this->parseConnectionString($connectionString);
        }

        private function parseConnectionString($connectionString) {
            $parts = explode(";", $connectionString);
            if (sizeof($parts) != 3) {
                throw new Exception("Error parsing connection string: " . $connectionString);
            }

            foreach ($parts as $part) {
                if (strpos($part, "Endpoint") === 0) {
                    $this->endpoint = "https" . substr($part, 11);
                } else if (strpos($part, "SharedAccessKeyName") === 0) {
                    $this->sasKeyName = substr($part, 20);
                } else if (strpos($part, "SharedAccessKey") === 0) {
                    $this->sasKeyValue = substr($part, 16);
                }
            }
        }
    }


### <a name="create-security-token"></a>建立安全性權杖
您可以在 [此處](http://msdn.microsoft.com/library/dn495627.aspx)找到建立安全性權杖的詳細資料。
已將下列方法新增至 **NotificationHub** 類別，以根據目前要求的 URI 和從連接字串中擷取的認證來建立權杖。

    private function generateSasToken($uri) {
        $targetUri = strtolower(rawurlencode(strtolower($uri)));

        $expires = time();
        $expiresInMins = 60;
        $expires = $expires + $expiresInMins * 60;
        $toSign = $targetUri . "\n" . $expires;

        $signature = rawurlencode(base64_encode(hash_hmac('sha256', $toSign, $this->sasKeyValue, TRUE)));

        $token = "SharedAccessSignature sr=" . $targetUri . "&sig="
                    . $signature . "&se=" . $expires . "&skn=" . $this->sasKeyName;

        return $token;
    }

### <a name="send-a-notification"></a>傳送通知
首先，讓我們先定義代表通知的類別。

    class Notification {
        public $format;
        public $payload;

        # array with keynames for headers
        # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
        # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
        public $headers;

        function __construct($format, $payload) {
            if (!in_array($format, ["template", "apple", "windows", "gcm", "windowsphone"])) {
                throw new Exception('Invalid format: ' . $format);
            }

            $this->format = $format;
            $this->payload = $payload;
        }
    }

此類別是原生通知主體的容器，或是一組範本通知案例的屬性，及一組包含格式 (原生平台或範本) 的標頭，以及平台特定屬性 (如 Apple 到期屬性和 WNS 標頭)。

請參閱[通知中樞 REST API 文件](http://msdn.microsoft.com/library/dn495827.aspx)及特定通知平台的格式，以取得所有可用選項。

有了此類別之後，我們現在可以在 **NotificationHub** 類別內寫入傳送通知方法。

    public function sendNotification($notification, $tagsOrTagExpression="") {
        if (is_array($tagsOrTagExpression)) {
            $tagExpression = implode(" || ", $tagsOrTagExpression);
        } else {
            $tagExpression = $tagsOrTagExpression;
        }

        # build uri
        $uri = $this->endpoint . $this->hubPath . "/messages" . NotificationHub::API_VERSION;
        $ch = curl_init($uri);

        if (in_array($notification->format, ["template", "apple", "gcm"])) {
            $contentType = "application/json";
        } else {
            $contentType = "application/xml";
        }

        $token = $this->generateSasToken($uri);

        $headers = [
            'Authorization: '.$token,
            'Content-Type: '.$contentType,
            'ServiceBusNotification-Format: '.$notification->format
        ];

        if ("" !== $tagExpression) {
            $headers[] = 'ServiceBusNotification-Tags: '.$tagExpression;
        }

        # add headers for other platforms
        if (is_array($notification->headers)) {
            $headers = array_merge($headers, $notification->headers);
        }

        curl_setopt_array($ch, array(
            CURLOPT_POST => TRUE,
            CURLOPT_RETURNTRANSFER => TRUE,
            CURLOPT_SSL_VERIFYPEER => FALSE,
            CURLOPT_HTTPHEADER => $headers,
            CURLOPT_POSTFIELDS => $notification->payload
        ));

        // Send the request
        $response = curl_exec($ch);

        // Check for errors
        if($response === FALSE){
            throw new Exception(curl_error($ch));
        }

        $info = curl_getinfo($ch);

        if ($info['http_code'] <> 201) {
            throw new Exception('Error sending notificaiton: '. $info['http_code'] . ' msg: ' . $response);
        }
    } 

上述方法會傳送 HTTP POST 要求至通知中心的 /messages 端點，並使用正確的主體和標頭傳送通知。

## <a name="complete-tutorial"></a>完成教學課程
現在您可以透過從 PHP 後端傳送通知，來完成開始使用教學課程。

初始化您的通知中樞用戶端 (請依 [開始使用教學課程]中的指示替換連接字串和中心名稱)：

    $hub = new NotificationHub("connection string", "hubname");    

然後根據您的目標行動平台新增傳送程式碼。

### <a name="windows-store-and-windows-phone-81-non-silverlight"></a>Windows 市集和 Windows Phone 8.1 (非 Silverlight)
    $toast = '<toast><visual><binding template="ToastText01"><text id="1">Hello from PHP!</text></binding></visual></toast>';
    $notification = new Notification("windows", $toast);
    $notification->headers[] = 'X-WNS-Type: wns/toast';
    $hub->sendNotification($notification, null);

### <a name="ios"></a>iOS
    $alert = '{"aps":{"alert":"Hello from PHP!"}}';
    $notification = new Notification("apple", $alert);
    $hub->sendNotification($notification, null);

### <a name="android"></a>Android
    $message = '{"data":{"msg":"Hello from PHP!"}}';
    $notification = new Notification("gcm", $message);
    $hub->sendNotification($notification, null);

### <a name="windows-phone-80-and-81-silverlight"></a>Windows Phone 8.0 和 8.1 Silverlight
    $toast = '<?xml version="1.0" encoding="utf-8"?>' .
                '<wp:Notification xmlns:wp="WPNotification">' .
                   '<wp:Toast>' .
                        '<wp:Text1>Hello from PHP!</wp:Text1>' .
                   '</wp:Toast> ' .
                '</wp:Notification>';
    $notification = new Notification("windowsphone", $toast);
    $notification->headers[] = 'X-WindowsPhone-Target : toast';
    $notification->headers[] = 'X-NotificationClass : 2';
    $hub->sendNotification($notification, null);


### <a name="kindle-fire"></a>Kindle Fire
    $message = '{"data":{"msg":"Hello from PHP!"}}';
    $notification = new Notification("adm", $message);
    $hub->sendNotification($notification, null);

執行 PHP 程式碼現在應會產生一則顯示於目標裝置的通知。

## <a name="next-steps"></a>後續步驟
在本主題中，我們會說明如何為通知中心建立簡單的 Java REST 用戶端。 您可以在這裡執行下列動作：

* 下載完整的 [PHP REST 包裝函式範例]，其中包含上述所有程式碼。
* 繼續了解 [即時新聞教學課程] 中的通知中心標記功能
* 了解 [通知使用者教學課程] 中的推播通知給個人使用者

如需詳細資訊，另請參閱 [PHP 開發人員中心](/develop/php/)。

[PHP REST 包裝函式範例]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-php
[開始使用教學課程]: http://azure.microsoft.com/documentation/articles/notification-hubs-ios-get-started/

