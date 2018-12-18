---
title: 在 Azure CLI 建立您的第一個函式
description: 了解如何使用 Azure CLI 與 Azure Functions Core Tools 建立您的第一個適用於無伺服器執行的 Azure 函式。
services: functions
keywords: ''
author: ggailey777
ms.author: glenga
ms.assetid: 674a01a7-fd34-4775-8b69-893182742ae0
ms.date: 09/10/2018
ms.topic: quickstart
ms.service: azure-functions
ms.custom: mvc
ms.devlang: azure-cli
manager: jeconnoc
ms.openlocfilehash: ef5459b2b31b67afe187612ffc1ab079a5045a8c
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49114905"
---
# <a name="create-your-first-function-from-the-command-line"></a>從命令列建立您的第一個函式

此快速入門主題會逐步引導您從命令列或終端機建立您的第一個函式。 您會使用 Azure CLI 來建立函數應用程式，此應用程式是主控函式的[無伺服器](https://azure.microsoft.com/solutions/serverless/)基礎結構。 函式程式碼專案是使用 [Azure Functions Core Tools](functions-run-local.md) (也用來將函式應用程式專案部署到 Azure) 從範本產生的。

您可以使用 Mac、Windows 或 Linux 電腦，依照下面步驟操作。

## <a name="prerequisites"></a>必要條件

在執行此範例之前，您必須具備下列項目︰

+ 安裝 [Azure Core Tools 2.x 版](functions-run-local.md#v2)。

+ 安裝 [Azure CLI]( /cli/azure/install-azure-cli)。 此文章需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找您擁有的版本。 您也可以使用 [Azure Cloud Shell](https://shell.azure.com/bash)。

+ 有效的 Azure 訂用帳戶。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-the-local-function-app-project"></a>建立本機函式應用程式專案

從命令列執行下列命令，以在目前本機目錄的 `MyFunctionProj` 資料夾中建立函式應用程式專案。 系統也會在 `MyFunctionProj` 中建立 GitHub 存放庫。

```bash
func init MyFunctionProj
```

當出現提示時，請使用方向鍵從下列語言選項中選取背景工作角色執行階段：

+ `dotnet`：建立 .NET 類別庫專案 (.csproj)。
+ `node`：建立 JavaScript 專案。

當命令執行時，您會看到如下輸出：

```output
Writing .gitignore
Writing host.json
Writing local.settings.json
Initialized empty Git repository in C:/functions/MyFunctionProj/.git/
```

## <a name="create-a-function"></a>建立函式

下列命令會瀏覽到新的專案，並建立名為 `MyHtpTrigger` 的 HTTP 觸發函式。

```bash
cd MyFunctionProj
func new --name MyHttpTrigger --template "HttpTrigger"
```

當命令執行時，您會看到如下輸出 (JavaScript 函式)：

```output
Writing C:\functions\MyFunctionProj\MyHttpTrigger\index.js
Writing C:\functions\MyFunctionProj\MyHttpTrigger\sample.dat
Writing C:\functions\MyFunctionProj\MyHttpTrigger\function.json
```

## <a name="edit-the-function"></a>編輯函式

根據預設，此範本建立的函式在發出要求時需要函式金鑰。 為了更輕鬆地在 Azure 中測試函式，您需要更新函式以允許匿名存取。 您進行此變更的方法，視您的函式專案語言而有所不同。

### <a name="c"></a>C\#

開啟新函式的 MyHttpTrigger.cs 程式碼檔案，將函式定義中 **AuthorizationLevel** 屬性的值更新為 `anonymous` 並儲存變更。

```csharp
[FunctionName("MyHttpTrigger")]
        public static IActionResult Run([HttpTrigger(AuthorizationLevel.Anonymous, 
            "get", "post", Route = null)]HttpRequest req, ILogger log)
```

### <a name="javascript"></a>JavaScript

開啟新的函式的 function.json 檔案 (在文字編輯器中開啟)、將 **bindings.httpTrigger** 中的 **authLevel** 屬性更新為 `anonymous` 並儲存變更。

```json
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
```

您現在可以在 Azure 中呼叫該函式，而不需要提供函式金鑰。 在本機執行時一律不需要函式金鑰。

## <a name="run-the-function-locally"></a>在本機執行函式

下列命令會啟動函數應用程式。 應用程式使用的 Azure Functions 執行階段和 Azure 中的是相同的。

```bash
func host start --build
```

編譯 C# 專案必須使用 `--build` 選項， JavaScript 專案則不需要。

當 Functions 主機啟動時，它會寫入類似下列輸出的內容 (已截斷來提高可讀性)：

```output

                  %%%%%%
                 %%%%%%
            @   %%%%%%    @
          @@   %%%%%%      @@
       @@@    %%%%%%%%%%%    @@@
     @@      %%%%%%%%%%        @@
       @@         %%%%       @@
         @@      %%%       @@
           @@    %%      @@
                %%
                %

...

Content root path: C:\functions\MyFunctionProj
Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

...

Http Functions:

        HttpTrigger: http://localhost:7071/api/HttpTrigger

[8/27/2018 10:38:27 PM] Host started (29486ms)
[8/27/2018 10:38:27 PM] Job host started
```

從執行階段輸出複製 `HTTPTrigger` 函式的 URL，並將它貼到瀏覽器的網址列。 將查詢字串 `?name=<yourname>` 附加至此 URL 並執行要求。 下圖顯示瀏覽器中對於本機函式傳回之 GET 要求所做出的回應︰

![在瀏覽器中進行本機測試](./media/functions-create-first-azure-function-azure-cli/functions-test-local-browser.png)

您現在已經在本機執行您的函式，您可以開始在 Azure 中建立函數應用程式與其他必要資源。

[!INCLUDE [functions-create-resource-group](../../includes/functions-create-resource-group.md)]

[!INCLUDE [functions-create-storage-account](../../includes/functions-create-storage-account.md)]

## <a name="create-a-function-app"></a>建立函數應用程式

您必須擁有函式應用程式以便主控函式的執行。 函式應用程式會提供環境來讓您的函式程式碼進行無伺服器執行。 它可讓您將多個函式群組為邏輯單位，以方便您管理、部署和共用資源。 使用 [az functionapp create](/cli/azure/functionapp#az-functionapp-create) 命令來建立函式應用程式。 

在下列命令中，使用唯一函式應用程式名稱來替代您看見 `<app_name>` 預留位置的地方，並使用儲存體帳戶名稱來替代 `<storage_name>`。 `<app_name>` 會作為函式應用程式的預設 DNS 網域，所以此名稱在 Azure 的所有應用程式中都必須是唯一的名稱。 _deployment-source-url_ 參數是 GitHub 中的範例存放庫，其中包含 "Hello World" HTTP 觸發函式。

```azurecli-interactive
az functionapp create --resource-group myResourceGroup --consumption-plan-location westeurope \
--name <app_name> --storage-account  <storage_name>  
```

設定 _consumption-plan-location_ 參數，即表示函數應用程式是裝載於取用主控方案。 在這個無伺服器方案中，會根據您的函式，視需要動態新增資源，且只有在函式執行時才需要付費。 如需詳細資訊，請參閱[選擇正確的主控方案](functions-scale.md)。

建立函式應用程式後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "containerSize": 1536,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "quickstart.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "quickstart.azurewebsites.net",
    "quickstart.scm.azurewebsites.net"
  ],
   ....
    // Remaining output has been truncated for readability.
}
```

## <a name="configure-the-function-app"></a>設定函式應用程式

Core Tools 2.x 版會使用 Azure Functions 2.x 執行階段的範本建立專案。 因此，您需要確認 Azure 中是使用 2.x 版執行階段。 將 `FUNCTIONS_WORKER_RUNTIME` 應用程式設定設為 `~2`，讓函數應用程式一律使用最新的 2.x 版。 使用 [az functionapp config appsettings set](https://docs.microsoft.com/cli/azure/functionapp/config/appsettings#set) 命令設定應用程式設定。

在下列 Azure CLI 命令中，`<app_name> 是您函數應用程式的名稱。

```azurecli-interactive
az functionapp config appsettings set --name <app_name> \
--resource-group myResourceGroup \
--settings FUNCTIONS_WORKER_RUNTIME=~2
```

[!INCLUDE [functions-publish-project](../../includes/functions-publish-project.md)]

[!INCLUDE [functions-test-function-code](../../includes/functions-test-function-code.md)]

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

[!INCLUDE [functions-quickstart-next-steps-cli](../../includes/functions-quickstart-next-steps-cli.md)]
