---
title: 使用 Go 將事件傳送到 Azure 事件中樞 | Microsoft Docs
description: 開始使用 Go 將事件傳送到事件中樞
services: event-hubs
author: ShubhaVijayasarathy
manager: kamalb
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.date: 07/23/2018
ms.author: shvija
ms.openlocfilehash: 40b3aa82c3e9e8ab9a30362c0a41998877655725
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005592"
---
# <a name="send-events-to-event-hubs-using-go"></a>使用 Go 將事件傳送至事件中樞

Azure 事件中樞是可高度擴充的事件管理系統，每秒可以處理數以百萬計的事件，讓應用程式能處理及分析已連線裝置和其他系統所產生的大量資料。 收集到事件中樞之後，您就可以使用處理序內處理常式來接收及處理事件，或將事件轉送到期他分析系統。

若要深入了解事件中樞，請參閱[事件中樞概觀][Event Hubs overview]。

此教學課程說明如何將事件從以 Go 撰寫的應用程式傳送到事件中樞。 若要接收事件，請使用 **Go eph** (事件處理器主機) 套件，如[對應的接收文章](event-hubs-go-get-started-receive-eph.md)中所述。

此教學課程中的程式碼取自[這些 GitHub 範例](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/eventhubs) \(英文\)， 您可以檢查它們以查看完整的可運作應用程式，包括重要陳述式與變數宣告。

您可以在[事件中樞套件存放庫](https://github.com/Azure/azure-event-hubs-go/tree/master/_examples) \(英文\) 中找到其他範例。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* 安裝在本機的 Go。 如果有需要，請依照[這些指示](https://golang.org/doc/install) \(英文\) 執行。
* 使用中的 Azure 帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶][]。
* 現有的事件中樞命名空間和事件中樞。 請依照[此文章](event-hubs-create.md)中的指示來建立這些實體。

## <a name="install-go-package"></a>安裝 Go 套件

使用 `go get` 或 `dep` 取得適用於事件中樞的 Go 套件。 例如︰

```bash
go get -u github.com/Azure/azure-event-hubs-go
go get -u github.com/Azure/azure-amqp-common-go/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
```

## <a name="import-packages-in-your-code-file"></a>在您的程式碼檔案中匯入套件

若要匯入 Go 套件，請使用下列程式碼範例：

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
)
```

## <a name="create-service-principal"></a>建立服務主體

依照[使用Azure CLI 2.0 來建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)中的指示建立新的服務主體。 使用下列名稱將提供的認證儲存在您的環境中。 Azure SDK for Go 與事件中樞套件預先設定為尋找這些變數名稱：

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

現在，為使用這些認證的事件中樞用戶端建立授權提供者：

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

## <a name="create-event-hubs-client"></a>建立事件中樞用戶端

下列程式碼會建立事件中樞用戶端：

```go
hub, err := eventhubs.NewHub("namespaceName", "hubName", tokenProvider)
ctx := context.WithTimeout(context.Background(), 10 * time.Second)
defer hub.Close(ctx)
if err != nil {
    log.Fatalf("failed to get hub %s\n", err)
}
```

## <a name="send-messages"></a>傳送訊息

在下列程式碼片段中，使用 (1) 從終端機以互動方式傳送訊息，或使用 (2) 在您的程式內傳送訊息：

```go
// 1. send messages at the terminal
ctx = context.Background()
reader := bufio.NewReader(os.Stdin)
for {
    fmt.Printf("Input a message to send: ")
    text, _ := reader.ReadString('\n')
    hub.Send(ctx, eventhubs.NewEventFromString(text))
}

// 2. send messages within program
ctx = context.Background()
hub.Send(ctx, eventhubs.NewEventFromString("hello Azure!")
```

## <a name="extras"></a>額外項目

取得事件中樞內的資料分割識別碼：

```go
info, err := hub.GetRuntimeInformation(ctx)
if err != nil {
    log.Fatalf("failed to get runtime info: %s\n", err)
}
log.Printf("got partition IDs: %s\n, info.PartitionIDs)
```

## <a name="next-steps"></a>後續步驟

請瀏覽下列頁面以深入了解事件中樞：

* [使用 EventProcessorHost 接收事件](event-hubs-go-get-started-receive-eph.md)
* [事件中樞概觀][Event Hubs overview]
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
