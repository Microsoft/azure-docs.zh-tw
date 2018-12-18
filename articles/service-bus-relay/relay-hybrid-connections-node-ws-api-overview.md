---
title: Azure 轉送 Node API 概觀 | Microsoft Docs
description: 轉送 Node API 概觀
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: b7d6e822-7c32-4cb5-a4b8-df7d009bdc85
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/23/2018
ms.author: spelluru
ms.openlocfilehash: bf0173f9c9802be689f7f3a893d381a251a2b16a
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701131"
---
# <a name="relay-hybrid-connections-node-api-overview"></a>轉送混合式連線 Node API 概觀

## <a name="overview"></a>概觀

[`hyco-ws`](https://www.npmjs.com/package/hyco-ws) Azure 轉送混合式連線的 Node 封裝建置於並擴充 ['ws'](https://www.npmjs.com/package/ws) NPM 封裝。 此封裝重新匯出該基本封裝的所有匯出，並新增新的匯出，可啟用與 Azure 轉送服務混合式連線功能整合。 

現有的應用程式 `require('ws')` 可以改為搭配使用這個套件與 `require('hyco-ws')`，這也會啟用混合式案例，其中應用程式可以從「防火牆內部」及透過混合式連線，在本機接聽 WebSocket 連線，全部都在相同的時間。
  
## <a name="documentation"></a>文件

API [記載於主要的 'ws' 封裝中](https://github.com/websockets/ws/blob/master/doc/ws.md)。 本文說明此套件與該基準不同之處。 

基本封裝和此 ' hyco-ws' 之間的主要差異在於此 'hyco-ws' 新增了新的伺服器類別，透過 `require('hyco-ws').RelayedServer` 匯出，和一些協助程式方法。

### <a name="package-helper-methods"></a>套件協助程式方法

封裝匯出上有數個公用程式方法可供您參考，如下所示︰

```JavaScript
const WebSocket = require('hyco-ws');

var listenUri = WebSocket.createRelayListenUri('namespace.servicebus.windows.net', 'path');
listenUri = WebSocket.appendRelayToken(listenUri, 'ruleName', '...key...')
...

```

協助程式方法適用於與此封裝搭配使用，但也可供 Node 伺服器用來啟用 web 或裝置用戶端以建立接聽程式或傳送程式。 伺服器會將內嵌短期權杖的 URI 傳遞給它們來使用這些方法。 這些 URI 也可以搭配使用一般不支援設定 WebSocket 信號交換之 HTTP 標頭的 WebSocket 堆疊。 將授權權杖內嵌至 URI 主要目的是為了支援這些程式庫外部使用方式案例。 

#### <a name="createrelaylistenuri"></a>createRelayListenUri

```JavaScript
var uri = createRelayListenUri([namespaceName], [path], [[token]], [[id]])
```

建立指定命名空間和路徑的有效 Azure 轉送混合式連線接聽程式 URI。 此 URI 便可搭配使用轉送版本的 WebSocketServer 類別。

- `namespaceName` (必要) - 要使用的 Azure 轉送命名空間之網域限定名稱。
- `path` (必要) - 該命名空間中現有的 Azure 轉送混合式連線名稱。
- `token` (選用) - 內嵌在接聽程式 URI 的先前發行轉送存取權杖 (請參閱下列範例)。
- `id` (選用) - 可啟用端對端診斷追蹤要求的追蹤識別碼。

`token` 值是選擇性的，且應該僅用於無法傳送 HTTP 標題與 WebSocket 交握作業時，在此情況下為 W3C WebSocket 堆疊。                  


#### <a name="createrelaysenduri"></a>createRelaySendUri
 
```JavaScript
var uri = createRelaySendUri([namespaceName], [path], [[token]], [[id]])
```

建立指定命名空間和路徑的有效 Azure 轉送混合式連線傳送 URI。 這個 URI 可以搭配使用任何 WebSocket 用戶端。

- `namespaceName` (必要) - 要使用的 Azure 轉送命名空間之網域限定名稱。
- `path` (必要) - 該命名空間中現有的 Azure 轉送混合式連線名稱。
- `token` (選用) - 內嵌在傳送 URI 的先前發行轉送存取權杖 (請參閱下列範例)。
- `id` (選用) - 可啟用端對端診斷追蹤要求的追蹤識別碼。

`token` 值是選擇性的，且應該僅用於無法傳送 HTTP 標題與 WebSocket 交握作業時，在此情況下為 W3C WebSocket 堆疊。                   


#### <a name="createrelaytoken"></a>createRelayToken 

```JavaScript
var token = createRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

建立指定目標 URI、SAS 規則和 SAS 規則金鑰的 Azure 轉送共用存取簽章 (SAS) 權杖，如果省略到期引數，則其從目前即刻起指定的秒數或一小時為有效。

- `uri` (必要) - 要發行權杖的 URI。 URI 會正規化為使用 HTTP 配置，且會移除查詢字串資訊。
- `ruleName` (必要) - 指定 URI 所代表之實體或 URI 主機部分所代表之命名空間的 SAS 規則名稱。
- `key` (必要) - SAS 規則的有效金鑰。 
- `expirationSeconds` (選用) - 所產生權杖應過期前的秒數。 如果未指定，則預設值是 1 小時 (3600)。

發行的權杖會授與指定持續時間與所指定之 SAS 規則相關聯的權限。

#### <a name="appendrelaytoken"></a>appendRelayToken

```JavaScript
var uri = appendRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

此方法在功能上相當於先前所記載的 `createRelayToken` 方法，但傳回的權杖正確地附加至輸入 URI。

### <a name="class-wsrelayedserver"></a>類別 ws.RelayedServer

`hycows.RelayedServer` 類別是替代 `ws.Server` 類別，其不會接聽本機網路，但會委派接聽 Azure 轉送服務。

這兩個類別大部分是合約相容，也就是說使用 `ws.Server` 類別的現有應用程式可以輕鬆地變更為使用轉送的版本。 主要差異是在建構函式以及可用的選項中。

#### <a name="constructor"></a>建構函式  

```JavaScript 
var ws = require('hyco-ws');
var server = ws.RelayedServer;

var wss = new server(
    {
        server: ws.createRelayListenUri(ns, path),
        token: function() { return ws.createRelayToken('http://' + ns, keyrule, key); }
    });
```

`RelayedServer` 建構函式支援與 `Server` 不同的一組引數，因為它並非獨立接聽程式，也不能內嵌到現有的 HTTP 接聽程式架構。 可用選項也較少，因為 WebSocket 管理多半會委派至轉送服務。

建構函式引數︰

- `server` (必要) - 要接聽之混合式連線名稱的完整 URI，通常是使用 WebSocket.createRelayListenUri() 協助程式方法所建構。
- `token` (必要) - 這個引數包含先前發行的權杖字串或可以呼叫以取得權杖字串的回呼函式。 回呼是慣用選項，因為它可讓權杖更新。

#### <a name="events"></a>活動

`RelayedServer` 執行個體會發出三個事件，可讓您處理傳入要求、建立連線，以及偵測錯誤狀況。 您必須訂閱 `connect` 事件以處理訊息。 

##### <a name="headers"></a>headers

```JavaScript 
function(headers)
```

接受連入連線前才會引發 `headers` 事件，讓修改標題傳送給用戶端。 

##### <a name="connection"></a>connection

```JavaScript
function(socket)
```

接受新的 WebSocket 連線時就會發出。 物件類型是 `ws.WebSocket`，與基本封裝相同。


##### <a name="error"></a>錯誤

```JavaScript
function(error)
```

如果基礎伺服器發出錯誤，它會在這裡轉送。  

#### <a name="helpers"></a>協助程式

若要簡化啟動轉送的伺服器，並立即訂閱連入連線，封裝會公開簡單的協助程式函式，也會在範例中使用，如下所示︰

##### <a name="createrelayedlistener"></a>createRelayedListener

```JavaScript
var WebSocket = require('hyco-ws');

var wss = WebSocket.createRelayedServer(
    {
        server: WebSocket.createRelayListenUri(ns, path),
        token: function() { return WebSocket.createRelayToken('http://' + ns, keyrule, key); }
    }, 
    function (ws) {
        console.log('connection accepted');
        ws.onmessage = function (event) {
            console.log(JSON.parse(event.data));
        };
        ws.on('close', function () {
            console.log('connection closed');
        });       
});
``` 

##### <a name="createrelayedserver"></a>createRelayedServer

```javascript
var server = createRelayedServer([options], [connectCallback] )
```

這個方法會呼叫建構函式來建立 RelayedServer 的新執行個體，然後訂閱 'connection' 事件所提供的回呼。
 
##### <a name="relayedconnect"></a>relayedConnect

只要鏡像函式中的 `createRelayedServer`協助程式，`relayedConnect` 會建立用戶端連線，並在產生的通訊端上訂閱 'open' 事件。

```JavaScript
var uri = WebSocket.createRelaySendUri(ns, path);
WebSocket.relayedConnect(
    uri,
    WebSocket.createRelayToken(uri, keyrule, key),
    function (socket) {
        ...
    }
);
```

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure 轉送，請造訪下列連結：
* [什麼是 Azure 轉送？](relay-what-is-it.md)
* [可用的轉送 API](relay-api-overview.md)
