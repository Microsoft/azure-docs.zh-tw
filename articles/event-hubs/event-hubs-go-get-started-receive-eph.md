---
title: 使用 Go 從 Azure 事件中樞接收事件 | Microsoft Docs
description: 開始使用 Go 從事件中樞接收事件
services: event-hubs
author: ShubhaVijayasarathy
manager: kamalb
ms.service: event-hubs
ms.workload: core
ms.topic: article
ms.date: 07/23/2018
ms.author: shvija
ms.openlocfilehash: eaea6adbaef7baf9bb1e617ba0a709cf14edf781
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005573"
---
# <a name="receive-events-from-event-hubs-using-go"></a>使用 Go 從事件中樞接收事件

Azure 事件中樞是可高度擴充的事件管理系統，每秒可以處理數以百萬計的事件，讓應用程式能處理及分析已連線裝置和其他系統所產生的大量資料。 收集到事件中樞之後，您就可以使用處理序內處理常式來接收及處理事件，或將事件轉送到期他分析系統。

若要深入了解事件中樞，請參閱[事件中樞概觀][Event Hubs overview]。

本教學說明如何在 Go 應用程式中接收來自事件中樞的事件。 若要了解如何傳送事件，請參閱[相對應的傳送文章](event-hubs-go-get-started-send.md)。

此教學課程中的程式碼取自[這些 GitHub 範例](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/eventhubs) \(英文\)，您可以檢查它們以查看完整的可運作應用程式，包括重要陳述式與變數宣告。

您可以在[事件中樞套件存放庫](https://github.com/Azure/azure-event-hubs-go/tree/master/_examples) \(英文\) 中找到其他範例。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要下列必要條件：

* 安裝在本機的 Go。 如果有需要，請依照[這些指示](https://golang.org/doc/install) \(英文\) 執行。
* 使用中的 Azure 帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶][]。
* 若要接收訊息，目標事件中樞中必須有訊息。 若要了解如何傳送訊息，請參閱[傳送教學課程](event-hubs-go-get-started-send.md)。
* 現有的事件中樞 (請參閱下節內容)。
* 現有的儲存體帳戶和容器 (請參閱下下節的內容)。

### <a name="create-an-event-hub"></a>建立事件中樞

本教學課程會以現有的事件中樞命名空間和事件中樞作為開始。 請依照[此文章](event-hubs-create.md)中的指示來建立這些實體。

### <a name="create-a-storage-account-and-container"></a>建立儲存體帳戶和容器

分割區上的租用和事件資料流中的檢查點等狀態，都會使用 Azure 儲存體容器在接收者之間共用。 您可以搭配 Go SDK 建立儲存體帳戶和容器，但您也可以依照[關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)中的指示建立它們。

搭配 Go SDK 建立儲存體構件的範例，可於 [Go 範例存放庫](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/storage) \(英文\) 和本教學課程所對應的範例中取得。

## <a name="receive-messages"></a>接收訊息

若要接收訊息，請透過 `go get` 或 `dep` 取得適用於事件中樞的 Go 套件：

```bash
go get -u github.com/Azure/azure-event-hubs-go/...
go get -u github.com/Azure/azure-amqp-common-go/...
go get -u github.com/Azure/go-autorest/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
dep ensure -add github.com/Azure/go-autorest
```

## <a name="import-packages-in-your-code-file"></a>在您的程式碼檔案中匯入套件

若要匯入 Go 套件，請使用下列程式碼範例：

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
    eph "github.com/Azure/azure-event-hubs-go/eph"
    storageLeaser "github.com/Azure/azure-event-hubs-go/storage"
    azure "github.com/Azure/go-autorest/autorest/azure"
)
```

## <a name="create-service-principal"></a>建立服務主體

依照[使用Azure CLI 2.0 來建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)中的指示建立新的服務主體。 使用下列名稱將提供的認證儲存在您的環境中。 Azure SDK for Go 與事件中樞套件皆已預先設定為會尋找這些變數名稱。

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

接下來，為使用這些認證的事件中樞用戶端建立授權提供者：

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

## <a name="get-metadata-struct"></a>取得中繼資料結構

使用 Azure Go SDK 取得具有關於您 Azure 環境之中繼資料的結構。 後續作業會使用此結構來尋找正確的端點。

```go
azureEnv, err := azure.EnvironmentFromName("AzurePublicCloud")
if err != nil {
    log.Fatalf("could not get azure.Environment struct: %s\n", err)
}
```

## <a name="create-credential-helper"></a>建立認證協助程式 

建立會使用先前的 Azure Active Directory (AAD) 認證來建立適用於儲存體的共用存取簽章 (SAS) 認證的認證協助程式。 最後一個參數會告訴此建構函式應使用和先前相同的環境變數：

```go
cred, err := storageLeaser.NewAADSASCredential(
    subscriptionID,
    resourceGroupName,
    storageAccountName,
    storageContainerName,
    storageLeaser.AADSASCredentialWithEnvironmentVars())
if err != nil {
    log.Fatalf("could not prepare a storage credential: %s\n", err)
}
```

## <a name="create-checkpointer-and-leaser"></a>建立 Checkpointer 和 Leaser 

建立負責將分割區租用給特定接收者的 **Leaser**，以及負責為訊息資料流撰寫檢查點，使其他接收者可以從正確的位移開始讀取的 **Checkpointer**。

目前有提供單一的 **StorageLeaserCheckpointer**，能使用相同的儲存體容器來管理租用和檢查點。 除了儲存體帳戶和容器名稱之外，**StorageLeaserCheckpointer** 也需要在上個步驟中所建立的認證及 Azure 環境結構以正確存取容器。

```go
leaserCheckpointer, err := storageLeaser.NewStorageLeaserCheckpointer(
    cred,
    storageAccountName,
    storageContainerName,
    azureEnv)
if err != nil {
    log.Fatalf("could not prepare a storage leaserCheckpointer: %s\n", err)
}
```

## <a name="construct-event-processor-host"></a>建構事件處理器主機

您現在已具備以下列方式建構 EventProcessorHost 所需的所有項目。 如先前所述，相同的 **StorageLeaserCheckpointer** 會被用來同時作為 Leaser 和 Checkpointer：

```go
ctx := context.Background()
p, err := eph.New(
    ctx,
    nsName,
    hubName,
    tokenProvider,
    leaserCheckpointer,
    leaserCheckpointer)
if err != nil {
    log.Fatalf("failed to create EPH: %s\n", err)
}
defer p.Close(context.Background())
```

## <a name="create-handler"></a>建立處理常式 

現在請建立處理常式，並向事件處理器主機註冊它。 當主機啟動時，它會將此處理常式及任何其他指定的處理常式套用到傳入的訊息：

```go
handler := func(ctx context.Context, event *eventhubs.Event) error {
    fmt.Printf("received: %s\n", string(event.Data))
    return nil
}

// register the handler with the EPH
_, err := p.RegisterHandler(ctx, handler)
if err != nil {
    log.Fatalf("failed to register handler: %s\n", err)
}
```

## <a name="receive-messages"></a>接收訊息

一切都設定好之後，您便可以搭配 `Start(context)` 啟動事件處理器主機來使它永久執行，或是搭配 `StartNonBlocking(context)` 來僅在有可用訊息的情況下執行。

本教學課程會以下列方式啟動並執行；請參閱 GitHub 範例以取得使用 `StartNonBlocking` 的範例：

```go
ctx := context.Background()
err = p.Start()
if err != nil {
    log.Fatalf("failed to start EPH: %s\n", err)
}
```

## <a name="notes"></a>注意

本教學課程使用單一 **EventProcessorHost**執行個體。 若要提升輸送量和可靠性，您應該在不同的系統上執行多個 **EventProcessorHost** 執行個體。 Leaser 系統能確保在指定時間內，單一接收者只會和指定的分割區關聯，並從之接收訊息。

## <a name="next-steps"></a>後續步驟

請造訪這些網頁以深入了解事件中樞：

* [搭配 Go 傳送事件](event-hubs-go-get-started-send.md)
* [事件中心概觀](event-hubs-about.md)
* [建立事件中樞](event-hubs-create.md)
* [事件中樞常見問題集](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[免費帳戶]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
