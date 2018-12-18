---
title: Azure 服務匯流排和事件中樞的 AMQP 1.0 通訊協定指南 | Microsoft Docs
description: Azure 服務匯流排和事件中樞的 AMQP 1.0 運算式和說明的通訊協定指南
services: service-bus-messaging,event-hubs
documentationcenter: .net
author: clemensv
manager: timlt
editor: ''
ms.assetid: d2d3d540-8760-426a-ad10-d5128ce0ae24
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/26/2018
ms.author: clemensv
ms.openlocfilehash: 75c6b5c34559ad17f662c895352bff5a58da00d4
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395843"
---
# <a name="amqp-10-in-azure-service-bus-and-event-hubs-protocol-guide"></a>Azure 服務匯流排和事件中樞的 AMQP 1.0 通訊協定指南

進階訊息佇列通訊協定 1.0 是標準化框架處理和傳輸通訊協定，可以非同步、安全且可靠的方式傳輸兩方之間的訊息。 它是 Azure 服務匯流排訊息和 Azure 事件中樞的主要通訊協定。 這兩項服務也支援 HTTPS。 同時支援的專屬 SBMP 通訊協定會慢慢被淘汰，以利採用 AMQP。

AMQP 1.0 是產業共同作業的結果，由中介軟體廠商 (例如 Microsoft 和 Red Hat) 與許多傳訊中介軟體使用者 (例如代表金融服務產業的 JP Morgan Chase) 攜手合作。 OASIS 是 AMQP 通訊協定和擴充規格的技術標準化論壇，它已獲得 ISO/IEC 19494 國際標準的正式核准。

## <a name="goals"></a>目標

此文章簡短摘要說明 AMQP 1.0 訊息規格的核心概念以及目前正由 OASIS AMQP 技術委員會定案的一小組草稿擴充規格，並說明 Azure 服務匯流排如何根據這些規格進行實作和建置。

目的是要讓在任何平台上使用任何現有 AMQP 1.0 用戶端堆疊的開發人員，能夠透過 AMQP 1.0 與 Azure 服務匯流排互動。

常見的一般用途 AMQP 1.0 堆疊 (例如 Apache Proton 或 AMQP.NET Lite) 已經實作所有核心 AMQP 1.0 通訊協定。 這些基本軌跡有時會以更高層級的 API 包裝；Apache Proton 甚至會提供命令式 Messenger API 和反應式 Reactor API 等兩個 API 包裝。

在以下討論中，我們假設 AMQP 連線、工作階段和連結的管理，以及框架傳輸和流量控制都是由個別堆疊 (例如 Apache PROTON-C) 處理，而不需要應用程式開發人員特別注意。 我們會以抽象方式假設存在一些 API 原始物件，像是連線的能力，以及建立某種形式的「傳送者」和「接收者」抽象物件的能力，然後分別會有某種形式的 `send()` 和 `receive()` 作業。

討論 Azure 服務匯流排的進階功能 (例如訊息瀏覽或工作階段管理) 時，將會以 AMQP 詞彙說明這些功能，但也可作為以這個假設 API 抽象概念為基礎的多層式虛擬實作。

## <a name="what-is-amqp"></a>AMQP 是什麼？

AMQP 是框架處理和傳輸通訊協定。 框架處理表示它會為以網路連線的任一方向流入的二進位資料串流提供結構。 此結構會針對要在已連線方之間交換的不同資料區塊 (稱為框架) 提供略圖。 傳輸功能會確定通訊雙方都可以對於何時應傳送框架以及傳輸何時應視為完成，建立起共識。

與 AMQP 工作群組所產生且稍早過期的草稿版本 (仍有一些訊息代理程式在使用) 不同，工作群組的最終標準化 AMQP 1.0 通訊協定並未規定要存在訊息代理程式或訊息代理人內實體的任何特定拓撲。

此通訊協定可用於對稱的對等通訊，以便與支援佇列及發佈/訂閱實體的訊息代理程式互動，如 Azure 服務匯流排所為。 也可以用來與傳訊基礎結構互動，其互動模式不同於一般佇列 (和 Azure 事件中樞的情況一樣)。 當事件傳送至事件中樞時，事件中樞的作用如同佇列，但從中讀取事件時，其作用比較像是序列儲存體服務；它有點類似磁帶機。 用戶端會挑選可用資料串流的位移，然後取得從該位移至最新可用的所有事件。

AMQP 1.0 通訊協定是設計為可延伸的，允許進一步規格以增強其功能。 我們在此文件中討論的三個擴充規格會說明這點。 在透過現有 HTTPS/WebSockets 基礎結構進行的通訊中，設定原生 AMQP TCP 連接埠可能很困難。 繫結規格會定義如何透過 WebSockets 將 AMQP 分層。 以要求/回應方式與傳訊基礎結構互動，以便進行管理或提供進階功能時，AMQP 管理規格會定義必要的基本互動原始物件。 在同盟授權模型整合中，AMQP 宣告型安全性規格會定義如何產生關聯並更新與連結相關聯的授權權杖。

## <a name="basic-amqp-scenarios"></a>基本 AMQP 案例

本節說明 AMQP 1.0 與 Azure 服務匯流排的基本使用方式，其中包括建立連線、工作階段和連結，以及往返於服務匯流排實體 (例如佇列、主題和訂用帳戶) 傳輸訊息。

了解 AMQP 運作方式的最可靠來源是 AMQP 1.0 規格，但此規格是為了精確引導實作而撰寫，而非用以指導通訊協定。 本節著重於盡可能介紹描述服務匯流排如何使用 AMQP 1.0 的術語。 如需 AMQP 的更完整介紹，以及 AMQP 1.0 的更廣泛討論，您可以觀看[此影片課程][this video course]。

### <a name="connections-and-sessions"></a>連線和工作階段

AMQP 會將通訊程式稱為「容器」；其中包含「節點」，也就是這些容器內的通訊實體。 佇列就屬於這類節點。 AMQP 允許多工處理，所以單一連線可以用於節點之間的許多通訊路徑；例如，應用程式用戶端可以同時從一個佇列接收，並透過相同的網路連線傳送到另一個佇列。

![][1]

網路連線因此會錨定在容器上。 它是由採用用戶端角色的容器起始，對採用接收者角色的容器進行輸出 TCP 通訊端連線，以接聽和接受輸入 TCP 連線。 連線交握包括交涉通訊協定版本，宣告或交涉傳輸層安全性 (TLS/SSL) 的使用，以及以 SASL 為基礎之連線範圍的驗證/授權交握。

Azure 服務匯流排隨時都需要使用 TLS。 它支援透過 TCP 連接埠 5671 的連線，因此 TCP 連線會在進入 AMQP 通訊協定交握前先與 TLS 覆疊，而且還支援透過 TCP 連接埠 5672 的連線，因此伺服器會使用 AMQP 規定的模型，立即提供強制的 TLS 連線升級。 AMQP WebSockets 繫結會建立透過 TCP 連接埠 443 的通道，其相當於 AMQP 5671 連線。

在設定連線和 TLS 之後，服務匯流排會提供兩個 SASL 機制選項︰

* SASL PLAIN 常用於將使用者名稱和密碼認證傳送至伺服器。 服務匯流排沒有帳戶，但是有具名的[共用存取安全性規則](service-bus-sas.md)，這些規則可授予權限並與某個金鑰相關聯。 規則名稱可做為使用者名稱，而金鑰 (如 base64 編碼文字) 可做為密碼。 與所選規則相關聯的權限會控管允許在連接上進行的作業。
* 當用戶端想要使用稍後會說明的宣告型安全性 (CBS) 模型時，可用 SASL ANONYMOU 來略過 SASL 授權。 使用此選項，即可以匿名方式建立短期的用戶端連線，在此連線期間，用戶端只能與 CBS 端點互動，而且 CBS 交握必須完成。

建立傳輸連線之後，容器會各自宣告它們願意處理的框架大小上限，而在閒置逾時後，如果連線上沒有任何活動，它們就會單方面中斷連線。

他們也會宣告支援的並行通道數量。 通道是以連線為基礎的單向、輸出、虛擬傳輸路徑。 工作階段會從每個互連的容器取得通道，以形成雙向通訊路徑。

工作階段具有以時段為基礎的流量控制模型；建立工作階段時，每一方會宣告它願意在接收時段內接受的框架數目。 當各方交換框架時，已傳輸的框架會填滿該時段且傳輸會在時段已滿時停止，直到該時段使用「流程展演」進行重設或擴展為止 (「展演」是 AMQP 用語，表示在雙方之間交換的通訊協定層級軌跡)。

這個以時段為基礎的模型大致上類似於 TCP 以時段為基礎的流量控制概念，但屬於通訊端內的工作階段層級。 通訊協定具有允許多個並行工作階段的概念，因此高優先順序的流量可能會衝過節流的正常流量，就像在高速公路快速車道一樣。

Azure 服務匯流排目前只對每個連線使用一個工作階段。 服務匯流排標準和事件中樞的服務匯流排框架大小上限為 262,144 個位元組 (256 KB)。 進階服務匯流排則為 1,048,576 (1 MB)。 服務匯流排不會強加任何特定工作階段層級節流時段，但是會在連結層級流量控制中定期重設時段 (請參閱[下一節](#links))。

連線、通道和工作階段都是暫時的。 如果基礎連線摺疊，則必須重新建立連線、TLS 通道、SASL 授權內容及工作階段。

### <a name="links"></a>連結

AMQP 會透過連結傳輸訊息。 連結是在能以單一方向傳輸訊息的工作階段中建立的通訊路徑；傳輸狀態交涉會透過連結在已連線方之間雙向進行。

![][2]

任一容器可以在現有的工作階段中隨時建立連結，這使 AMQP 不同於其他許多通訊協定 (包括 HTTP 和 MQTT)，其中起始傳輸和傳輸路徑是建立通訊端連線之一方的獨佔權限。

起始連結的容器會要求相對的容器接受連結，而且選擇傳送者或接收者的角色。 因此，任一容器均可起始建立單向或雙向通訊路徑，而後者會模型化為成對連結。

連結會以節點命名並產生關聯。 如一開始所述，節點是容器內的通訊實體。

在服務匯流排中，節點直接等同於佇列、主題、訂用帳戶或佇列或訂用帳戶的寄不出信件子佇列。 AMQP 中使用的節點名稱因此是服務匯流排命名空間內實體的相對名稱。 如果佇列名為 `myqueue`，這也是它的 AMQP 節點名稱。 主題訂用帳戶會依循 HTTP API 慣例歸類為 “subscriptions” 資源集合，因此訂用帳戶 **sub** 或主題 **mytopic** 具有 AMQP 節點名稱 **mytopic/subscriptions/sub**。

正在連線的用戶端也必須使用本機節點名稱來建立連結；服務匯流排不會規範這些節點名稱，而且不會解譯它們。 AMQP 1.0 用戶端堆疊通常會使用配置，以確保這些暫時節點名稱是用戶端範圍內的唯一名稱。

### <a name="transfers"></a>傳輸

建立連結後，即可透過該連結傳輸訊息。 在 AMQP 中，會使用明確的通訊協定軌跡執行傳輸 (「傳輸」展演)，以透過連結將訊息從傳送者移到接收者。 傳輸會在「安置好」時完成，這表示雙方已建立該傳輸結果的共識。

![][3]

在最簡單的情況下，傳送者可以選擇傳送「預先安置」的訊息，這表示用戶端對結果不感興趣，而且接收者不會提供任何有關作業結果的意見反應。 這個模式由服務匯流排在 AMQP 通訊協定層級支援，但不會顯現在任何用戶端 API 中。

一般情況是傳送未安置好的訊息，然後接收者會使用「處置」展演表示接受或拒絕。 拒絕會發生於接收者因為任何原因而無法接受訊息時，而拒絕訊息會包含原因相關資訊，這是 AMQP 所定義的錯誤結構。 如果訊息因為服務匯流排內部的錯誤而被拒絕，則服務會傳回該結構內的額外資訊，而如果您提出支援要求，該資訊即可用來提供診斷提示給支援人員。 稍後您會了解錯誤的詳細資訊。

「已解除」狀態是一種特殊形式的拒絕，表示接收者對傳輸沒有任何技術異議，但是對於安置傳輸也不感興趣。 該情況的確存在，例如，當訊息傳遞至服務匯流排用戶端，而用戶端因為無法執行處理訊息所產生的工作 (儘管訊息傳遞本身並未出錯) 而選擇「放棄」訊息時。 該狀態的變化是「已修改」狀態，該狀態允許在訊息解除後進行變更。 服務匯流排目前不會使用該狀態。

AMQP 1.0 規格會定義稱為「已接收」的進一步處置狀態稱，其對於處理連結復原特別有用。 當先前的連線和工作階段遺失時，連結復原允許重組連結的狀態，以及新連線和工作階段的任何擱置傳遞。

服務匯流排不支援連結復原；如果用戶端失去對服務匯流排的連線，而且未安置的訊息傳輸已擱置，該訊息傳輸便會遺失，而用戶端必須重新連線、重建連結，以及重試傳輸。

同樣地，服務匯流排和事件中樞都支援「至少一次」傳輸，其中的傳送者可確保訊息已儲存並接受，但是不支援在 AMQP 層級的「剛好一次」傳輸，其中的系統會嘗試復原連結並繼續交涉傳遞狀態，以避免重複傳輸訊息。

為了彌補可能的重複傳送，服務匯流排支援重複偵測佇列和主題 (選用功能)。 重複偵測會在使用者定義的時段內記錄所有內送訊息的訊息識別碼，然後以無訊息方式放棄在該相同時段內使用相同訊息識別碼傳送的所有訊息。

### <a name="flow-control"></a>流量控制

除了先前討論過的工作階段層級流量控制模型以外，每個連結都有自己的流量控制模型。 工作階段層級流量控制可防止容器必須一次處理太多框架，連結層級流量控制會讓應用程式負責控制它想要從連結處理的訊息數目以及時機。

![][4]

在連結上，傳輸只會發生於傳送者有足夠的連結信用額度時。 連結信用額度是接收者使用「流程」展演所設定的計數器，其範圍是連結。 將連結信用額度指派給傳送者時，將會透過傳遞訊息來嘗試用完該信用額度。 每個訊息傳遞會使剩餘的連結信用額外遞減 1。 當連結信用額度用完時，便會停止傳遞。

當服務匯流排採用接收者角色時，則會立即將充足的連結信用額度提供給傳送者，以便立即傳送訊息。 使用連結信用額度時，服務匯流排偶爾會傳送流程展演給傳送者，以更新連結信用額度餘額。

在傳送者角色中，服務匯流排會立即傳送訊息，以用完任何未償付的連結信用額度。

在 API 層級的「接收」呼叫會轉譯成由用戶端傳送至服務匯流排的流程展演，而服務匯流排會取用佇列中第一個可用、已解除鎖定的訊息，將它鎖定並傳輸，以使用該信用額度。 如果沒有可立即傳遞的訊息，由任何連結使用該特定實體建立的任何未償付信用額度仍會以抵達順序記錄，而訊息會遭到鎖定，並且在能夠使用任何未償付信用額度時進行傳輸。

當傳輸進入其中一種終止狀態「已接受」、「已拒絕」或「已解除」時，訊息鎖定就會解除。 終止狀態為「已接受」時，就會從服務匯流排中移除訊息。 它會保留在服務匯流排中，並將在傳輸達到任何其他狀態時，傳遞給下一個接收者。 服務匯流排會在因為重複拒絕或解除而達到實體所允許的最大傳遞計數時，自動將訊息移到實體寄不出的信件佇列中。

現在，即使是服務匯流排 API 也不會直接公開這種選項，較低層級的 AMQP 通訊協定用戶端可以使用連結信用額度模型，透過核發大量的連結信用額度，將針對每個接收要求核發一單位信用額度的「提取式」模型變成「推送式」模型，然後接收可用的訊息，而不需要任何進一步的互動。 推送是透過 [MessagingFactory.PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_PrefetchCount) 或 [MessageReceiver.PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagereceiver#Microsoft_ServiceBus_Messaging_MessageReceiver_PrefetchCount) 屬性設定來支援。 如果兩者均不為零，則 AMQP 用戶端會使用它做為連結信用額度。

在此內容中，務必了解實體內訊息鎖定的到期時鐘會在從實體取得訊息時啟動，而不是在訊息放在網路上時啟動。 每當用戶端透過核發連結信用額度來表示接收訊息的整備性，因此預期會主動提取網路上的訊息並準備好處理它們。 否則訊息鎖定可能會在訊息傳遞之前過期。 使用連結信用流量控制應直接反映出可立即準備處理分派給接收者的可用訊息。

總而言之，下列各節提供在不同 API 互動期間展演流程的圖解概觀。 每一節會說明不同的邏輯作業。 其中一些互動可能很「緩慢」，這表示它們可能只會在有需要時執行。 直到傳送或要求第一個訊息，建立訊息傳送者才會造成網路互動。

下表中的箭號會顯示展演流程方向。

#### <a name="create-message-receiver"></a>建立訊息接收者

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={entity name},<br/>target={client link ID}<br/>) |用戶端會以接收者身分附加至實體 |
| 附加至連結結尾的服務匯流排回覆 |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={entity name},<br/>target={client link ID}<br/>) |

#### <a name="create-message-sender"></a>建立訊息傳送者

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link ID},<br/>target={entity name}<br/>) |沒有動作 |
| 沒有動作 |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={client link ID},<br/>target={entity name}<br/>) |

#### <a name="create-message-sender-error"></a>建立訊息傳送者 (錯誤)

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link ID},<br/>target={entity name}<br/>) |沒有動作 |
| 沒有動作 |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source=null,<br/>target=null<br/>)<br/><br/><-- detach(<br/>handle={numeric handle},<br/>closed=**true**,<br/>error={error info}<br/>) |

#### <a name="close-message-receiversender"></a>將訊息接收者/傳送者關閉

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) |沒有動作 |
| 沒有動作 |<-- detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) |

#### <a name="send-success"></a>傳送 (成功)

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |沒有動作 |
| 沒有動作 |<-- disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**accepted**<br/>) |

#### <a name="send-error"></a>傳送 (錯誤)

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |沒有動作 |
| 沒有動作 |<-- disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**rejected**(<br/>error={error info}<br/>)<br/>) |

#### <a name="receive"></a>接收

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> flow(<br/>link-credit=1<br/>) |沒有動作 |
| 沒有動作 |< transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=**receiver**,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**accepted**<br/>) |沒有動作 |

#### <a name="multi-message-receive"></a>多訊息接收

| 用戶端 | 服務匯流排 |
| --- | --- |
| --> flow(<br/>link-credit=3<br/>) |沒有動作 |
| 沒有動作 |< transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| 沒有動作 |< transfer(<br/>delivery-id={numeric handle+1},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| 沒有動作 |< transfer(<br/>delivery-id={numeric handle+2},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID+2},<br/>settled=**true**,<br/>state=**accepted**<br/>) |沒有動作 |

### <a name="messages"></a>訊息

下列各節說明服務匯流排會使用標準 AMQP 訊息區段中的哪些屬性，以及它們如何對應到服務匯流排 API 集。

應用程式需要定義的屬性應對應至 AMQP 的 `application-properties` 對應。

#### <a name="header"></a>頁首

| 欄位名稱 | 使用量 | API 名稱 |
| --- | --- | --- |
| 持久 |- |- |
| 優先順序 |- |- |
| ttl |此訊息的存留時間 |[TimeToLive](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_TimeToLive) |
| first-acquirer |- |- |
| delivery-count |- |[DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_DeliveryCount) |

#### <a name="properties"></a>properties

| 欄位名稱 | 使用量 | API 名稱 |
| --- | --- | --- |
| message-id |應用程式為此訊息定義的自由格式識別碼。 用於重複偵測。 |[MessageId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId) |
| user-id |應用程式定義的使用者識別碼，服務匯流排無法解譯。 |無法透過服務匯流排 API 存取。 |
| to |應用程式定義的目的地識別碼，服務匯流排無法解譯。 |[To](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_To) |
| 主旨 |應用程式定義的訊息用途識別碼，服務匯流排無法解譯。 |[Label](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Label) |
| reply-to |應用程式定義的回覆路徑指示器，服務匯流排無法解譯。 |[ReplyTo](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_ReplyTo) |
| correlation-id |應用程式定義的相互關聯識別碼，服務匯流排無法解譯。 |[CorrelationId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_CorrelationId) |
| Content-Type |應用程式為內文定義的內容類型識別碼，服務匯流排無法解譯。 |[ContentType](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_ContentType) |
| content-encoding |應用程式為內文定義的內容編碼識別碼，服務匯流排無法解譯。 |無法透過服務匯流排 API 存取。 |
| absolute-expiry-time |宣告訊息會過期的絕對立即時刻。 在輸入時忽略 (觀察到標頭 TTL)，在輸出時授權具權威性。 |[ExpiresAtUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_ExpiresAtUtc) |
| creation-time |宣告訊息的建立時間。 服務匯流排未使用 |無法透過服務匯流排 API 存取。 |
| group-id |應用程式為一組相關訊息所定義的識別碼。 用於服務匯流排工作階段。 |[SessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId) |
| group-sequence |用以識別訊息在工作階段內的相對序號的計數器。 服務匯流排會忽略。 |無法透過服務匯流排 API 存取。 |
| reply-to-group-id |- |[ReplyToSessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_ReplyToSessionId) |

#### <a name="message-annotations"></a>訊息註解

有幾個其他服務匯流排訊息屬性不屬於 AMQP 訊息屬性，而且會隨著 `MessageAnnotations` 在訊息上傳遞。

| 註解對應索引鍵 | 使用量 | API 名稱 |
| --- | --- | --- |
| x-opt-scheduled-enqueue-time | 宣告訊息應出現在實體上的時間 |[ScheduledEnqueueTime](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.scheduledenqueuetimeutc?view=azure-dotnet) |
| x-opt-partition-key | 應用程式定義的索引鍵，指出訊息應落在哪個資料分割中。 | [PartitionKey](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.partitionkey?view=azure-dotnet) |
| x-opt-via-partition-key | 當交易用於透過傳輸佇列傳送訊息時，應用程式所定義的資料分割索引鍵值。 | [ViaPartitionKey](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.viapartitionkey?view=azure-dotnet) |
| x-opt-enqueued-time | 由服務定義的 UTC 時間，表示將訊息加入佇列的實際時間。 輸入時忽略。 | [EnqueuedTimeUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc?view=azure-dotnet) |
| x-opt-sequence-number | 由服務定義並指派給訊息的唯一編號。 | [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber?view=azure-dotnet) |
| x-opt-offset | 由服務定義並已加入佇列的訊息序號。 | [EnqueuedSequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedsequencenumber?view=azure-dotnet) |
| x-opt-locked-until | 由服務定義。 訊息在佇列/訂用帳戶中鎖定的日期和時間。 | [LockedUntilUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.lockeduntilutc?view=azure-dotnet) |
| x-opt-deadletter-source | 由服務定義。 從無效信件佇列收到訊息時，原始訊息的來源。 | [DeadLetterSource](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deadlettersource?view=azure-dotnet) |

### <a name="transaction-capability"></a>交易功能

交易會將兩個以上的作業一起分組到執行範圍。 本質上，這類交易必須確定屬於指定作業群組的所有作業都一起成功或失敗。
作業會依識別碼 `txn-id` 分組。

針對交易式互動，用戶端會當作 `transaction controller` 來控制應分在一組的作業。 服務匯流排服務會當作 `transactional resource` 並依照 `transaction controller` 的要求執行工作。

用戶端與服務會透過用戶端所建立的 `control link` 通訊。 控制器會透過控制連結傳送 `declare` 和 `discharge` 訊息，分別配置及完成交易 (它們不代表交易式工作的區分)。 實際傳送/接收並非在此連結上執行。 每個交易式作業都能以所需的 `txn-id` 明確識別，因此在連線時可能會發生在任何連結上。 如果控制連結已關閉，但存在它所建立的非釋出交易，然後會立即復原所有這類交易，而嘗試在其上執行進一步交易式工作將會導致失敗。 不能預先決定控制連結上的訊息。

每次連線都必須起始自己的控制連結，才能夠開始和結束交易。 服務會定義當作 `coordinator` 的特殊目標。 用戶端/控制站會建立此目標的控制連結。 控制連結超出實體的界限，也就是說，相同的控制連結可用來起始和釋出多個實體的交易。

#### <a name="starting-a-transaction"></a>開始交易

若要開始進行交易式工作： 控制器必須從協調器取得 `txn-id`。 它會透過傳送 `declare` 類型訊息來執行此作業。 如果宣告成功，協調器就會以具有已指派 `txn-id` 的處置結果回應。

| 用戶端 (控制站) | | 服務匯流排 (協調器) |
| --- | --- | --- |
| attach(<br/>name={link name},<br/>... ,<br/>role=**sender**,<br/>target=**Coordinator**<br/>) | ------> |  |
|  | <------ | attach(<br/>name={link name},<br/>... ,<br/>target=Coordinator()<br/>) |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (**Declare()**)}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=**Declared**(<br/>**txn-id**={transaction ID}<br/>))|

#### <a name="discharging-a-transaction"></a>釋出交易

控制器會將 `discharge` 訊息傳送至協調器，結束交易式工作。 控制器會在 `fail` 釋出主體上設定旗標，表示它想要認可或復原交易式工作。 如果協調器無法完成釋出，則會以帶有 `transaction-error` 的這個結果拒絕訊息。

> 注意：fail=true 意指「交易復原」，而 fail=false 意指「認可」。

| 用戶端 (控制站) | | 服務匯流排 (協調器) |
| --- | --- | --- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| | . . . <br/>交易式工作<br/>其他連結上<br/> . . . |
| transfer(<br/>delivery-id=57, ...)<br/>{ AmqpValue (<br/>**Discharge(txn-id=0,<br/>fail=false)**)}| ------> |  |
| | <------ | disposition( <br/> first=57, last=57, <br/>state=**Accepted()**)|

#### <a name="sending-a-message-in-a-transaction"></a>在交易中傳送訊息

所有交易式工作完成時的交易式傳遞狀態都是 `transactional-state`，其帶有 txn-id。在傳送訊息的情況下，交易式狀態是由訊息的傳輸框架傳送。 

| 用戶端 (控制站) | | 服務匯流排 (協調器) |
| --- | --- | --- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| transfer(<br/>handle=1,<br/>delivery-id=1, <br/>**state=<br/>TransactionalState(<br/>txn-id=0)**)<br/>{ payload }| ------> |  |
| | <------ | disposition( <br/> first=1, last=1, <br/>state=**TransactionalState(<br/>txn-id=0,<br/>outcome=Accepted()**))|

#### <a name="disposing-a-message-in-a-transaction"></a>在交易中處置訊息

訊息處置包含 `Complete` / `Abandon` / `DeadLetter` / `Defer` 等作業。 若要在交易內執行這些作業，請隨著處置傳遞 `transactional-state`。

| 用戶端 (控制站) | | 服務匯流排 (協調器) |
| --- | --- | --- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| | <------ |transfer(<br/>handle=2,<br/>delivery-id=11, <br/>state=null)<br/>{ payload }|  
| disposition( <br/> first=11, last=11, <br/>state=**TransactionalState(<br/>txn-id=0,<br/>outcome=Accepted()**))| ------> |


## <a name="advanced-service-bus-capabilities"></a>進階服務匯流排功能

本節涵蓋 Azure 服務匯流排的進階功能，而這些功能是以目前正在 AMQP 的 OASIS Technical Committee 中開發的 AMQP 草稿延伸模組為基礎。 服務匯流排會實作這些草稿的最新版本，而且會採用這些草稿達到標準狀態時所引進的變更。

> [!NOTE]
> 服務匯流排訊息進階作業透過要求/回應模式受到支援。 如需這些作業的詳細資料，請參閱[服務匯流排中的 AMQP 1.0：要求/回應架構作業](service-bus-amqp-request-response.md)一文中的說明。
> 
> 

### <a name="amqp-management"></a>AMQP 管理

AMQP 管理規格是此文章討論的第一個延伸模組草稿。 此規格會定義一組分層於 AMQP 通訊協定之上的通訊協定，以便透過 AMQP 進行傳訊基礎結構的管理互動。 此規格定義泛型作業 (例如建立、讀取、更新和刪除)，以便管理傳訊基礎結構內的實體和一組查詢作業。

上述所有軌跡都需要用戶端與傳訊基礎結構之間的要求/回應互動，因此此規格會定義如何製作 AMQP 上互動模式的模型︰用戶端會連線到傳訊基礎結構、起始工作階段，然後建立一組連結。 在某一個連結上，用戶端會扮演傳送者，而在其他連結上扮演接收者，因此建立一組可做為雙向通道的連結。

| 邏輯作業 | 用戶端 | 服務匯流排 |
| --- | --- | --- |
| 建立要求回應路徑 |--> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=**null**,<br/>target=”myentity/$management”<br/>) |沒有動作 |
| 建立要求回應路徑 |沒有動作 |\<-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=null,<br/>target=”myentity”<br/>) |
| 建立要求回應路徑 |--> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=”myentity/$management”,<br/>target=”myclient$id”<br/>) | |
| 建立要求回應路徑 |沒有動作 |\<-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=”myentity”,<br/>target=”myclient$id”<br/>) |

備妥該組連結，要求/回應實作就相當簡單︰要求是傳送至傳訊基礎結構內了解此模式之實體的訊息。 在該要求訊息中，*properties* 區段中的 *reply-to* 欄位會設定為回應會傳遞至之連結的 *target* 識別碼。 處理實體會處理此要求，然後透過 *target* 識別碼符合所指示 *reply-to* 識別碼的連結傳遞回覆。

此模式顯然要求回覆目的地的用戶端容器和用戶端產生的識別碼在所有用戶端中是唯一的，而且基於安全性理由，還要難以預測。

用於管理通訊協定和所有其他使用相同模組之的訊協定的訊息交換會發生於應用程式層級；它們不會定義新的 AMQP 通訊協定層級軌跡。 這是刻意設計的，以便應用程式立即利用符合 AMQP 1.0 堆疊規範的這些擴充功能。

服務匯流排目前不會實作管理規格的任何核心功能，但是對宣告型安全性功能以及我們在下列各節中討論的幾乎所有進階功能而言，管理規格所定義的要求/回應模式是基本的：

### <a name="claims-based-authorization"></a>宣告型授權

AMQP 宣告型授權 (CBS) 規格草稿是以管理規格的要求/回應模式為基礎，主要說明如何搭配使用同盟安全性權杖與 AMQP 的廣義模型。

簡介中所討論的 AMQP 預設安全性模型是以 SASL 為基礎，並與 AMQP 連接交握整合。 使用 SASL 的好處是它會提供已定義一組機制的可延伸模型，任何正式依賴 SASL 的通訊協定均可受惠。 在這些機制之中，“PLAIN” 用於傳輸使用者名稱和密碼，“EXTERNAL” 用於繫結至 TLS 層級安全性，“ANONYMOUS” 用於表示缺少明確的驗證/授權，還有其他各種機制能夠用來傳送驗證和/或授權認證或權杖。

AMQP 的 SASL 整合有兩個缺點︰

* 所有認證與權杖的範圍都只限於連線。 傳訊基礎結構可能會以每個實體方式提供不同的存取控制；例如，允許權杖的持有者傳送至佇列 A，而不是至佇列 B。使用錨定在連線上的授權內容，就不可能使用單一連線並且對佇列 A 和 B 使用不同的存取權杖。
* 存取權杖的有效時間通常有限。 此有效性會要求使用者定期重新取得權杖，而如果使用者的存取權限已變更，權杖的簽發者便有機會拒絕簽發全新的權杖。 AMQP 連線可能會持續很長的時間。 SASL 模型只提供一個機會在連線時設定權杖，這表示傳訊基礎結構必須在權杖過期時中斷用戶端的連線，或者必須接受允許與存取權限可能已在其間撤銷的用戶端持續通訊的風險。

服務匯流排所實作的 AMQP CBS 規格，可讓這兩個問題獲得圓滿的解決︰它可讓用戶端建立存取權杖與每個節點的關聯，以及在這些權杖過期前進行更新，而不需中斷訊息流程。

CBS 會定義由傳訊基礎結構所提供的虛擬管理節點 (名為 *$cbs*)。 管理節點可代表傳訊基礎結構中的任何其他節點接受權杖。

通訊協定軌跡是由管理規格所定義的要求/回覆交換。 這表示用戶端會使用 *$cbs* 節點建立一組連結，在輸出連結上傳送要求，然後在輸入連結上等候回應。

要求訊息具有下列應用程式屬性︰

| Key | 選用 | 值類型 | 值內容 |
| --- | --- | --- | --- |
| operation |否 |字串 |**put-token** |
| type |否 |字串 |正在放置的權杖類型。 |
| name |否 |字串 |套用權杖的「對象」。 |
| expiration |是 |timestamp |權杖的到期時間。 |

*name* 屬性會識別應與權杖相關聯的實體。 在服務匯流排中，這是佇列或主題/訂用帳戶的路徑。 *type* 屬性會識別權杖類型︰

| 權杖類型 | 權杖描述 | 主體類型 | 注意 |
| --- | --- | --- | --- |
| amqp:jwt |JSON Web 權杖 (JWT) |AMQP 值 (字串) |尚未提供。 |
| amqp:swt |簡單 Web 權杖 (SWT) |AMQP 值 (字串) |僅支援 AAD/ACS 所簽發的 SWT 權杖 |
| servicebus.windows.net:sastoken |服務匯流排 SAS 權杖 |AMQP 值 (字串) |- |

權杖會賦予權限。 服務匯流排知道三個基本權限：「傳送」可進行傳送、「接聽」可進行接收，以「管理」可進行管理實體。 AAD/ACS 所簽發的 SWT 權杖會明確將這些權限納入為宣告。 服務匯流排 SAS 權杖會參考在命名空間或實體上設定的規則，而這些規則是使用權限來設定。 使用與該規則相關聯的金鑰來簽署權杖，以此方式讓權杖表達各自的權限。 與使用 put-token 之實體相關聯的權杖，將允許已連線的用戶端依照每個權杖權限來與實體互動。 用戶端承擔 sender 角色的連結需要「傳送」權限，而承擔 receiver 角色的連結則需要「接聽」權限。

回覆訊息具有下列 *application-properties* 值

| Key | 選用 | 值類型 | 值內容 |
| --- | --- | --- | --- |
| status-code |否 |int |HTTP 回應碼 **[RFC2616]**。 |
| status-description |是 |字串 |狀態的描述。 |

用戶端可以針對傳訊基礎結構中的任何實體重複呼叫 *put-token*。 權杖的範圍為目前用戶端且錨點為目前連線，這表示伺服器會在連線捨棄時捨棄所有保留的權杖。

目前的服務匯流排實作只允許 CBS 搭配SASL 方法 “ANONYMOUS”。 SSL/TLS 連線必須一律存在於 SASL 交握之前。

因此所選的 AMQP 1.0 用戶端必須支援 ANONYMOUS 機制。 匿名存取表示發生初始連線交握 (包括建立初始工作階段)，而服務匯流排不知道誰正在建立此連線。

建立連線和工作階段後，將連結附加至 *$cbs* 節點及傳送 *put-token* 要求是唯一允許的作業。 必須在建立連線後的 20 秒內使用對某個實體節點的 *put-token* 要求成功設定有效的權杖，否則服務匯流排會單方面中斷連線。

用戶端後續會負責追蹤權杖到期。 權杖到期時，服務匯流排會立即卸除個別實體連線上的所有連結。 若要避免發生問題，用戶端可以透過具有相同 *put-token* 軌跡的虛擬 *$cbs* 管理節點隨時使用新的權杖來取代節點的權杖，但不會干擾在不同連結上流動的承載流量。

### <a name="send-via-functionality"></a>傳送方式功能

[傳送方式 / 傳輸傳送者](service-bus-transactions.md#transfers-and-send-via)功能，可讓服務匯流排透過另一個實體將指定的訊息轉寄至目的地實體。 此功能用於在單一交易中執行跨實體的作業。

使用此功能，您可建立傳送者並建立 `via-entity` 的連結。 建立連結時，系統會傳遞其他資訊，以在此連結上建立訊息/傳輸的真正目的地。 一旦連結成功，在此連結上傳送的所有訊息都會透過 *via-entity* 自動轉寄到 *destination-entity*。 

> 注意：建立這個連結之前，必須對 via-entity 和 destination-entity 執行驗證。

| 用戶端 | | 服務匯流排 |
| --- | --- | --- |
| attach(<br/>name={link name},<br/>role=sender,<br/>source={client link ID},<br/>target=**{via-entity}**,<br/>**properties=map [(<br/>com.microsoft:transfer-destination-address=<br/>{destination-entity} )]** ) | ------> | |
| | <------ | attach(<br/>name={link name},<br/>role=receiver,<br/>source={client link ID},<br/>target={via-entity},<br/>properties=map [(<br/>com.microsoft:transfer-destination-address=<br/>{destination-entity} )] ) |

## <a name="next-steps"></a>後續步驟

若要深入了解 AMQP，請造訪下列連結：

* [服務匯流排 AMQP 概觀]
* [適用於服務匯流排分割的佇列和主題的 AMQP 1.0 支援]
* [Windows Server 服務匯流排中的 AMQP]

[this video course]: https://www.youtube.com/playlist?list=PLmE4bZU0qx-wAP02i0I7PJWvDWoCytEjD
[1]: ./media/service-bus-amqp-protocol-guide/amqp1.png
[2]: ./media/service-bus-amqp-protocol-guide/amqp2.png
[3]: ./media/service-bus-amqp-protocol-guide/amqp3.png
[4]: ./media/service-bus-amqp-protocol-guide/amqp4.png

[服務匯流排 AMQP 概觀]: service-bus-amqp-overview.md
[適用於服務匯流排分割的佇列和主題的 AMQP 1.0 支援]: service-bus-partitioned-queues-and-topics-amqp-overview.md
[Windows Server 服務匯流排中的 AMQP]: https://msdn.microsoft.com/library/dn574799.aspx
