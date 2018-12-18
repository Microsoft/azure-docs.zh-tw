---
title: 快速入門 - Azure SignalR 服務 REST API | Microsoft Docs
description: 使用 Azure SignalR 服務 REST API 的快速入門。
services: signalr
documentationcenter: ''
author: sffamily
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.tgt_pltfrm: ASP.NET
ms.workload: tbd
ms.date: 06/13/2018
ms.author: zhshang
ms.openlocfilehash: 40d5a02f83188330facc82701abdfb950585781c
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49310379"
---
# <a name="quickstart-broadcast-real-time-messages-from-console-app"></a>快速入門：從主控台應用程式廣播即時訊息

Azure SignalR 服務能提供 [REST API](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md) 以支援如廣播等的伺服器至用戶端通訊案例。 您可以選擇任何可進行 REST API 呼叫的程式設計語言。 您可以將訊息張貼到所有已連線的用戶端、依名稱指定的特定用戶端，或是某個用戶端群組。

在此快速入門中，您將會了解如何以 C# 從命令列應用程式將訊息傳送至已連線的用戶端應用程式。

## <a name="prerequisites"></a>必要條件

此快速入門可以在 macOS、Windows 或 Linux 上執行。
* [.NET Core SDK](https://www.microsoft.com/net/download/core)
* 您偏好的文字編輯器或程式碼編輯器。


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="sign-in-to-azure"></a>登入 Azure

使用您的 Azure 帳戶登入 Azure 入口網站 (<https://portal.azure.com/>)。


[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

## <a name="clone-the-sample-application"></a>複製範例應用程式

在服務部署期間，讓我們來準備程式碼。 複製 [GitHub 上的範例應用程式](https://github.com/aspnet/AzureSignalR-samples.git) \(英文\)，設定 SignalR 服務連接字串，然後在本機執行該應用程式。

1. 開啟 git 終端機視窗。 變更至您想要複製範例專案的資料夾。

1. 執行下列命令來複製範例存放庫。 此命令會在您的電腦上建立範例應用程式副本。

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```

## <a name="build-and-run-the-sample"></a>建置並執行範例

此範例為示範 Azure SignalR 服務之用途的主控台應用程式。 它提供兩個模式：

- 伺服器模式：使用簡單的命令來呼叫 Azure SignalR 服務 REST API。
- 用戶端模式：連線至 Azure SignalR 服務並接收來自伺服器的訊息。

此外，您也可以了解如何產生存取權杖以向 Azure SignalR 服務進行驗證。

### <a name="build-the-executable-file"></a>建置可執行檔

我們會使用 macOS osx.10.13-x64 作為範例。 您可以尋找[參考](https://docs.microsoft.com/dotnet/core/rid-catalog)以了解如何在其他平台上建置。
```bash
cd AzureSignalR-samples/samples/Serverless/

dotnet publish -c Release -r osx.10.13-x64
```

### <a name="start-a-client"></a>啟動用戶端

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="start-a-server"></a>啟動伺服器

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless server -c "<ConnectionString>" -h <HubName>
```

## <a name="run-the-sample-without-publishing"></a>在不發行的情況下執行範例

您也可以執行下列命令來啟動伺服器或用戶端

```bash
# Start a server
dotnet run -- server -c "<ConnectionString>" -h <HubName>

# Start a client
dotnet run -- client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="use-user-secrets-to-specify-connection-string"></a>使用使用者祕密來指定連接字串

您可以在範例的根目錄中執行 `dotnet user-secrets set Azure:SignalR:ConnectionString "<ConnectionString>"`。 在那之後，您便不再需要 `-c "<ConnectionString>"` 這個選項。

## <a name="usage"></a>使用量

在伺服器啟動之後，使用下列命令來傳送訊息

```
send user <User Id>
send users <User List>
send group <Group Name>
send groups <Group List>
broadcast
```

您可以啟動具不同用戶端名稱的多個用戶端。

## <a name="usage"> </a> 與第三方服務整合
Azure SignalR 服務可讓第三方服務與系統整合。
### <a name="usage"> </a> 技術規格的定義
下表顯示目前所支援的所有 REST API 版本。 您也可以找到每個特定版本的定義檔
版本 | API 狀態 | 門 | 特定
--- | --- | --- | ---
`1.0-preview` | 可用 | 5002 | [Swagger] (https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1-preview.json)
`1.0` | 可用 | 標準 | [Swagger] (https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1.json)
下列清單提供每個特定版本的可用 API 清單。
API | `1.0-preview` | `1.0`
--- | --- | ---
[廣播到所有目標] (# broadcast) | : heavy_check_mark: | : Heavy_check_mark:
[廣播到群組] (# broadcast-group) | : heavy_check_mark: | : Heavy_check_mark:
廣播到部分群組 | : heavy_check_mark: (已淘汰) | `N / A`
[傳送給特定使用者] (# send-user) | : heavy_check_mark: | : Heavy_check_mark:
傳送給部分使用者 | : heavy_check_mark: (已淘汰) | `N / A`
[將使用者新增至群組] (# add-user-to-group) | `N / A` | : Heavy_check_mark:
[從群組中移除使用者] (# remove-user-from-group) | `N / A` | : Heavy_check_mark:
<a name="broadcast"> </a>
### <a name="broadcast-to-everyone"></a>廣播給所有人
版本 | API HTTP 方法 | 要求 URL | Request body
--- | --- | --- | ---
`1.0-preview` | `POST` | `https: // <instance-name> .service.signalr.net: 5002 / api / v1-preview / hub / <hub-name>` | `{" target ":" <method-name> "," arguments ": [...]}`
`1.0` | `POST` | `https: // <instance-name> .service.signalr.net / api / v1 / hubs / <hub-name>` | 同上
<a name="broadcast-group"> </a>
### <a name="broadcast-to-a-group"></a>廣播到群組
版本 | API HTTP 方法 | 要求 URL | Request body
--- | --- | --- | ---
`1.0-preview` | `POST` | `https: // <instance-name> .service.signalr.net: 5002 / api / v1-preview / hub / <hub-name> / group / <group-name>` | `{" target ":" <method-name> "," arguments ": [...]}`
`1.0` | `POST` | `https: // <instance-name> .service.signalr.net / api / v1 / hubs / <hub-name> / groups / <group-name>` | 同上
<a name="send-user"> </a>
### <a name="sending-to-specific-users"></a>傳送給特定使用者
版本 | API HTTP 方法 | 要求 URL | Request body
--- | --- | --- | ---
`1.0-preview` | `POST` | `https: // <instance-name> .service.signalr.net: 5002 / api / v1-preview / hub / <hub-name> / user / <user-id>` | `{" target ":" <method-name> "," arguments ": [...]}`
`1.0` | `POST` | `https: // <instance-name> .service.signalr.net / api / v1 / hubs / <hub-name> / users / <user-id>` | 同上
<a name="add-user-to-group"> </a>
### <a name="adding-a-user-to-a-group"></a>將使用者新增至群組
版本 | API HTTP 方法 | 要求 URL
--- | --- | ---
`1.0` | `PUT` | `Https: // <instance-name> .service.signalr.net / api / v1 / hubs / <hub-name> / groups / <group-name> / users / <userid>`
<a name="remove-user-from-group"> </a>
### <a name="removing-a-user-from-a-group"></a>從群組中移除使用者
版本 | API HTTP 方法 | 要求 URL
--- | --- | ---
`1.0` | `DELETE` | `Https: // <instance-name> .service.signalr.net / api / v1 / hubs / <hub-name> / groups / <group-name> / users / <userid>`

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]
