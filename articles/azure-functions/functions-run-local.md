---
title: 使用 Azure Functions Core Tools | Microsoft Docs
description: 在 Azure Functions 上執行 Azure 函式之前，先了解如何從命令提示字元或終端機在本機電腦上撰寫 Azure 函式並進行測試。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
editor: ''
ms.assetid: 242736be-ec66-4114-924b-31795fd18884
ms.service: functions
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: multiple
ms.topic: article
ms.date: 08/14/2018
ms.author: glenga
ms.openlocfilehash: cb336d6742aab10e1fd8305fd52f1376bb4f2598
ms.sourcegitcommit: 76797c962fa04d8af9a7b9153eaa042cf74b2699
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42140096"
---
# <a name="work-with-azure-functions-core-tools"></a>使用 Azure Functions Core Tools

Azure Functions Core Tools 可讓您從命令提示字元或終端機，在本機電腦上開發及測試您的函式。 您的本機函式可以連線到即時 Azure 服務，而且您可以在本機電腦上使用完整的 Functions 執行階段進行您的函式偵錯。 您甚至可以將函式應用程式部署至您的 Azure 訂用帳戶。

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

## <a name="core-tools-versions"></a>Core Tools 版本

Azure Functions Core Tools 有兩個版本。 您使用的版本取決於您的本機開發環境，選擇的語言，以及所需的支援層級：

+ [1.x 版](#v1)：支援 1.x 版的執行階段，這是正式上市 (GA) 版本。 這個版本的工具只有在 Windows 電腦上提供支援，並且從 [npm 套件](https://docs.npmjs.com/getting-started/what-is-npm)進行安裝。 使用此版本，您可以未正式支援的實驗性語言建立函式。 如需詳細資訊，請參閱 [Azure Functions 中支援的語言](supported-languages.md)。

+ [版本 2.x](#v2)：支援[版本 2.x 的執行階段](functions-versions.md)。 此版本支援 [Windows](#windows-npm)、[macOS](#brew) 和 [Linux](#linux)。 使用平台專屬的套件管理員或 npm 進行安裝。 如同 2.x 執行階段，這個版本的 Core Tools 目前為預覽狀態。 

除非另外註明，否則本文中的範例適用於 2.x 版。 若要接收 2.x 版的重要更新 (包括重大變更通知)，請隨時留意 [Azure App Service 通知](https://github.com/Azure/app-service-announcements/issues)存放庫。

## <a name="install-the-azure-functions-core-tools"></a>安裝 Azure Functions Core Tools

[Azure Functions Core Tools] 包含相同的執行階段版本，以支援您可在本機開發電腦上執行的 Azure Functions 執行階段。 它也提供命令來建立函式、連線到 Azure，以及部署函式專案。

### <a name="v1"></a>版本 1.x

工具的原始版本會使用 Functions 1.x 執行階段。 這個版本使用 .NET Framework (4.7.1)，並且只有在 Windows 電腦上提供支援。 安裝版本 1.x 工具之前，您必須先[安裝 NodeJS](https://docs.npmjs.com/getting-started/installing-node) (內含 npm)。

使用下列命令來安裝 1.x 版工具：

```bash
npm install -g azure-functions-core-tools
```

### <a name="v2"></a>版本 2.x

>[!NOTE]
> Azure Functions 執行階段 2.0 為預覽版本，目前尚未完全支援 Azure Functions 的所有功能。 如需詳細資訊，請參閱 [Azure Functions 版本](functions-versions.md)。 

工具的 2.x 版會使用以 .NET Core 為建置基礎的 Azure Functions 執行階段 2.x。 .NET Core 2.x 支援的所有平台都支援這個版本，包括 [Windows](#windows-npm)、[macOS](#brew) 和 [Linux](#linux)。

#### <a name="windows-npm"></a>Windows

下列步驟使用 npm 在 Windows 上安裝 Core Tools。 您也可以使用 [Chocolatey](https://chocolatey.org/)。 如需詳細資訊，請參閱 [Core Tools 讀我檔案](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#windows)。

1. 安裝[適用於 Windows 的 .NET Core 2.1](https://www.microsoft.com/net/download/windows)。

2. 安裝 [Node.js] (內含 npm)。 針對 2.x 版的工具，只支援 Node.js 8.5 和更新版本。

3. 安裝 Core Tools 套件：

    ```bash
    npm install -g azure-functions-core-tools@core
    ```

#### <a name="brew"></a>採用 Homebrew 的 MacOS

下列步驟使用 Homebrew 在 macOS 上安裝 Core Tools。

1. 安裝[適用於 macOS 的 .NET Core 2.1](https://www.microsoft.com/net/download/macos)。

2. 如果尚未安裝 [Homebrew](https://brew.sh/)，請加以安裝。

3. 安裝 Core Tools 套件：

    ```bash
    brew tap azure/functions
    brew install azure-functions-core-tools 
    ```

#### <a name="linux"></a> 採用 APT 的 Linux (Ubuntu/Debian)

下列步驟使用 [APT](https://wiki.debian.org/Apt) 在 Ubuntu/Debian Linux 散發套件上安裝 Core Tools。 若為其他 Linux 散發套件，請參閱 [Core Tools 讀我檔案](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#linux)。

1. 安裝[適用於 Linux 的 .NET Core 2.1](https://www.microsoft.com/net/download/linux)。

2. 將 Microsoft 產品金鑰註冊為可信任：

    ```bash
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
    ```

3. 請確認您的 Ubuntu 伺服器正在執行下表其中一個適當的版本。 若要新增 apt 來源，請執行：

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
    sudo apt-get update
    ```

    | Linux 散發套件 | 版本 |
    | --------------- | ----------- |
    | Ubuntu 17.10    | `artful`    |
    | Ubuntu 17.04    | `zesty`     |
    | Ubuntu 16.04/Linux Mint 18    | `xenial`  |

4. 安裝 Core Tools 套件：

    ```bash
    sudo apt-get install azure-functions-core-tools
    ```

## <a name="create-a-local-functions-project"></a>建立本機的 Functions 專案

Functions 專案目錄包含 [host.json](functions-host-json.md) 和 [local.settings.json](#local-settings-file) 檔案，以及包含個別函式程式碼的子資料夾。 此目錄相當於 Azure 中的函式應用程式。 若要深入了解 Functions 的資料夾結構，請參閱 [Azure Functions 的開發人員指南](functions-reference.md#folder-structure)。

2.x 版要求您在初始化時為您的專案選取預設語言，而所有新增的函式會使用預設語言範本。 在 1.x 版中，您會在每次建立函式時指定語言。

在終端機視窗或命令提示字元中，執行下列命令來建立專案和本機 Git 存放庫：

```bash
func init MyFunctionProj
```

當您提供專案名稱時，系統會建立具有該名稱的新資料夾，並將其初始化。 否則，會將目前的資料夾初始化。  
在 2.x 版中，當您執行命令時，您必須為您的專案選擇執行階段。 如果您打算開發 JavaScript 函式，請選擇**節點**：

```output
Select a worker runtime:
dotnet
node
```

使用向上/向下鍵來選擇語言，然後按 Enter。 JavaScript 專案的輸出看起來會像下列範例：

```output
Select a worker runtime: node
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing C:\myfunctions\myMyFunctionProj\.vscode\extensions.json
Initialized empty Git repository in C:/myfunctions/myMyFunctionProj/.git/
```

若要建立不含本機 Git 存放庫的專案，請使用 `--no-source-control [-n]` 選項。

> [!IMPORTANT]
> 根據預設，2.x 版的 Core Tools 會建立適用於 .NET 執行階段的函數應用程式專案作為 [C# 類別專案](functions-dotnet-class-library.md) (.csproj)。 這些 C# 專案可以與 Visual Studio 2017 或 Visual Studio Code 搭配使用，並在測試期間以及發佈至 Azure 時進行編譯。 如果您想要改為建立和使用在 1.x 版中以及在入口網站中建立的相同 C# 指令碼 (.csx) 檔案，當您建立及部署函式時，必須包含 `--csx` 參數。

## <a name="register-extensions"></a>註冊延伸模組

在 2.x 版的 Azure Functions 執行階段中，您必須明確註冊您在函式應用程式中使用的繫結延伸模組 (繫結類型)。

[!INCLUDE [Register extensions](../../includes/functions-core-tools-install-extension.md)]

如需詳細資訊，請參閱 [Azure Functions 觸發程序和繫結概念](functions-triggers-bindings.md#register-binding-extensions)。

## <a name="local-settings-file"></a>本機設定檔

local.settings.json 檔案會儲存應用程式設定、連接字串和 Azure Functions Core Tools 的設定。 其結構如下：

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "<connection-string>",
    "AzureWebJobsDashboard": "<connection-string>",
    "MyBindingConnection": "<binding-connection-string>"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*"
  },
  "ConnectionStrings": {
    "SQLConnectionString": "<sqlclient-connection-string>"
  }
}
```

| 設定      | 說明                            |
| ------------ | -------------------------------------- |
| **IsEncrypted** | 設定為 **true** 時，所有的值都會使用本機電腦金鑰加密。 需搭配 `func settings` 命令使用。 預設值為 **false**。 |
| **值** | 於本機執行時使用的應用程式設定集合與連接字串。 這些值會對應至 Azure 中函數應用程式的應用程式設定，例如 **AzureWebJobsStorage** 和 **AzureWebJobsDashboard**。 許多觸發程序和繫結都有參考連結字串應用程式設定的屬性，例如 [Blob 儲存體觸發程序](functions-bindings-storage-blob.md#trigger---configuration)的**連線**屬性。 對於這類屬性，您需要在 [值] 陣列中定義應用程式設定。 <br/>**AzureWebJobsStorage** 是 HTTP 以外之觸發程序的必要應用程式設定。 當您在本機安裝了 [Azure 儲存體模擬器](../storage/common/storage-use-emulator.md)，便可以將 **AzureWebJobsStorage** 設定至 `UseDevelopmentStorage=true` 與使用模擬器的核心工具。 此功能在開發期間非常實用，但您應在部署前先透過實際的儲存體連接進行測試。 |
| **Host** | 此區段中的設定能自訂於本機執行的 Functions 主機處理序。 |
| **LocalHttpPort** | 設定於執行本機 Functions 主機 (`func host start` 和 `func run`) 時所使用的預設連接埠。 `--port` 命令列選項的優先順序高於此值。 |
| **CORS** | 定義針對[跨來源資源共享 (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) 所允許的來源。 來源是以不含空格的逗號分隔清單提供。 支援萬用字元值 (\*)，它允許來自任何來源的要求。 |
| **ConnectionStrings** | 請勿將此集合用於您函式繫結所使用的連接字串。 此集合僅供架構使用，通常會從設定檔的 **ConnectionStrings** 區段取得連接字串，例如 [Entity Framework](https://msdn.microsoft.com/library/aa937723(v=vs.113).aspx)。 此物件中的連接字串會新增至具有 [System.Data.SqlClient](https://msdn.microsoft.com/library/system.data.sqlclient(v=vs.110).aspx) 提供者類型的環境。 此集合中的項目不會發佈至具備其他應用程式設定的 Azure。 您必須明確地將這些值新增到函數應用程式設定的**連接字串**集合中。 如果您要在函式程式碼中建立 [SqlConnection](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection(v=vs.110).aspx) \(英文\)，您應該將連接字串值以及您的其他連線一起儲存於**應用程式設定**中。 |

這些函數應用程式設定值在您的程式碼中也可以做為環境變數加以讀取。 如需詳細資訊，請參閱這些特定語言參考主題的「環境變數」章節：

+ [先行編譯 C#](functions-dotnet-class-library.md#environment-variables)
+ [C# 指令碼 (.csx)](functions-reference-csharp.md#environment-variables)
+ [F#](functions-reference-fsharp.md#environment-variables)
+ [Java](functions-reference-java.md#environment-variables) 
+ [JavaScript](functions-reference-node.md#environment-variables)

local.settings.json 檔案中的值，只會由於本機執行的 Functions 工具使用。 根據預設，在專案發佈至 Azure 時，這些設定將不會自動移轉。 請在[發佈時](#publish)使用 `--publish-local-settings` 參數，以確保這些設定會新增至 Azure 中的函式應用程式。 **ConnectionStrings** 中的值永遠不會發佈。

若沒有針對 **AzureWebJobsStorage** 設定有效的儲存體連接字串，而且未使用模擬器時，系統會顯示下列錯誤訊息：  

>在 local.settings.json 中遺失 AzureWebJobsStorage 的值。 這對 HTTP 以外的所有觸發程序是必要的。 您可以執行 'func azure functionapp fetch-app-settings <functionAppName>'，或指定 local.settings.json 中的連接字串。

### <a name="get-your-storage-connection-strings"></a>取得您的儲存體連接字串

即使使用儲存體模擬器進行開發，您可能想要透過實際的儲存體連接進行測試。 假設您已經[建立了儲存體帳戶](../storage/common/storage-create-storage-account.md)，您可以透過下列其中一種方式取得有效的儲存體連接字串：

+ 從 [Azure 入口網站]。 瀏覽至您的儲存體帳戶，並在 **[設定]** 中選取 **[存取金鑰]**，然後複製其中一個**連接字串**值。

  ![從 Azure 入口網站複製連接字串](./media/functions-run-local/copy-storage-connection-portal.png)

+ 使用 [Azure 儲存體總管](http://storageexplorer.com/)以連接至您的 Azure 帳戶。 在**總管**中展開您的訂閱，選取您的儲存體帳戶，並複製主要或次要連接字串。 

  ![透過儲存體總管複製連接字串](./media/functions-run-local/storage-explorer.png)

+ 使用核心工具，並藉由以下其中一個命令從 Azure 下載連接字串：

    + 從現有的函數應用程式下載所有設定：

    ```bash
    func azure functionapp fetch-app-settings <FunctionAppName>
    ```
    + 取得特定儲存體帳戶的連接字串：

    ```bash
    func azure storage fetch-connection-string <StorageAccountName>
    ```
    
    如果您尚未登入 Azure，系統會提示您這麼做。

## <a name="create-func"></a>建立函式

若要建立函式，請執行下列命令：

```bash
func new
```

在 2.x 版中，當您執行 `func new` 時，系統會提示您選擇採用函式應用程式預設語言的範本，然後系統也會提示您為您的函式選擇名稱。 在 1.x 版中，系統也會提示您選擇語言。

```output
Select a language: Select a template:
Blob trigger
Cosmos DB trigger
Event Grid trigger
HTTP trigger
Queue trigger
SendGrid
Service Bus Queue trigger
Service Bus Topic trigger
Timer trigger
```

如您在下列佇列觸發程序輸出中所見，函式程式碼會在子資料夾中產生並採用所提供的函式名稱：

```output
Select a language: Select a template: Queue trigger
Function name: [QueueTriggerJS] MyQueueTrigger
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\index.js
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\readme.md
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\sample.dat
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\function.json
```

您也可以使用下列引數，在命令中指定這些選項：

| 引數     | 說明                            |
| ------------------------------------------ | -------------------------------------- |
| **`--language -l`**| 範本程式語言，例如 C#、F# 或 JavaScript。 這是 1.x 版中的必要選項。 在 2.x 版中，請勿使用這個選項，或選擇您專案的預設語言。 |
| **`--template -t`** | 使用 `func templates list` 命令，以針對每個支援的語言查看可用範本的完整清單。   |
| **`--name -n`** | 函式名稱。 |
| **`--csx`** | (2.x 版) 產生 1.x 版中和入口網站中所使用的相同 C# 指令碼 (.csx) 範本。 |

例如，若要在單一命令中建立 JavaScript HTTP 觸發程序，請執行：

```bash
func new --template "Http Trigger" --name MyHttpTrigger
```

若要在單一命令中建立佇列所觸發的函式，請執行：

```bash
func new --template "Queue Trigger" --name QueueTriggerJS
```

## <a name="start"></a>在本機執行函式

若要執行 Functions 專案，請執行 Functions 主機。 主機可允許專案中所有函式的觸發程序：

```bash
func host start
```
1.x 版才需要 `host` 命令。

`func host start` 支援下列選項：

| 選項     | 說明                            |
| ------------ | -------------------------------------- |
| **`--cors`** | 以逗號分隔的 CORS 來源清單，不含空格。 |
| **`--debug <type>`** | 啟動已開啟偵錯連接埠的主機，以便您從 [Visual Studio Code](https://code.visualstudio.com/tutorials/functions-extension/getting-started) 或 [Visual Studio 2017](functions-dotnet-class-library.md) 連結至 **func.exe** 程序。 *\<type\>* 選項為 `VSCode` 和 `VS`。  |
| **`--port -p`** | 要接聽的本機連接埠。 預設值：7071。 |
| **`--timeout -t`** | Functions 主機要啟動的逾時 (以秒為單位)。 預設值：20 秒。|
| **`--useHttps`** | 繫結至 `https://localhost:{port}` 而不是 `http://localhost:{port}` 。 根據預設，此選項會在您的電腦上建立受信任的憑證。|
| **`--build`** | 執行前先建置目前的專案。 僅限 2.x 版和 C# 專案。 |
| **`--cert`** | 包含私密金鑰的 .pfx 檔案路徑。 僅能與 `--useHttps` 搭配使用。 僅限 2.x 版。 | 
| **`--password`** | 密碼或包含 .pfx 檔案密碼的檔案。 僅能與 `--cert` 搭配使用。 僅限 2.x 版。 |
| **`--language-worker`** | 用來設定語言背景工作角色的引數。 僅限 2.x 版。 |
| **`--nodeDebugPort -n`** | 要使用的節點偵錯工具連接埠。 預設值：Launch.json 中的值或 5858。 僅限 1.x 版。 |

針對 C# 類別庫專案 (.csproj)，您必須包含 `--build` 選項才能產生程式庫 .dll。

Functions 主機啟動時，它會輸出 HTTP 觸發函式的 URL：

```bash
Found the following functions:
Host.Functions.MyHttpTrigger

Job host started
Http Function MyHttpTrigger: http://localhost:7071/api/MyHttpTrigger
```

### <a name="passing-test-data-to-a-function"></a>將測試資料傳遞至函式

若要在本機測試您的函式，您要使用 HTTP 要求在本機伺服器上[啟動 Functions 主機](#start)並呼叫端點。 您呼叫的端點取決於函式的類型。

>[!NOTE]  
> 本主題中的範例使用 cURL 工具，從終端機或命令提示字元傳送 HTTP 要求。 您可以使用您選擇的工具，將 HTTP 要求傳送至本機伺服器。 以 Linux 為基礎的系統預設可以使用 cURL 工具。 在 Windows 上，您必須先下載並安裝 [cURL 工具 (英文)](https://curl.haxx.se/)。

如需測試函式的更多一般資訊，請參閱[在 Azure Functions 中測試程式碼的策略](functions-test-a-function.md)。

#### <a name="http-and-webhook-triggered-functions"></a>HTTP 和 Webhook 觸發的函式

您要呼叫下列端點，以在本機執行 HTTP 和 Webhook 觸發的函式：

    http://localhost:{port}/api/{function_name}

請務必使用 Functions 主機接聽的相同伺服器名稱和連接埠。 啟動 Function 主機時，您會在產生的輸出中看到下列內容。 您可以使用觸發程序支援的任何 HTTP 方法呼叫此 URL。 

下列 cURL 命令從 GET 要求在查詢字串中傳遞 _name_ 參數，觸發 `MyHttpTrigger` 快速入門函式。 

```bash
curl --get http://localhost:7071/api/MyHttpTrigger?name=Azure%20Rocks
```
下列範例與從 POST 要求在要求本文中傳遞 _name_ 所呼叫的函式相同：

```bash
curl --request POST http://localhost:7071/api/MyHttpTrigger --data '{"name":"Azure Rocks"}'
```

您可以從瀏覽器在查詢字串中傳遞資料來進行 GET 要求。 對於所有其他 HTTP 方法，您必須使用 cURL、Fiddler、Postman 或類似的 HTTP 測試工具。  

#### <a name="non-http-triggered-functions"></a>非 HTTP 觸發函式

對於 HTTP 觸發程序和 Webhook 以外的所有函式類型，您可以呼叫管理端點在本機測試函式。 在本機伺服器上使用 HTTP POST 要求來呼叫此端點會觸發函式。 您可以在 POST 要求本文中選擇性地傳遞測試資料到執行程序。 這項功能類似於 Azure 入口網站中的 [測試] 索引標籤。  

您可以呼叫下列系統管理員端點來觸發非 HTTP 函式：

    http://localhost:{port}/admin/functions/{function_name}

若要將測試資料傳遞至函式的管理員端點，您必須在 POST 要求訊息的本文中提供資料。 訊息本文必須具備下列 JSON 格式：

```JSON
{
    "input": "<trigger_input>"
}
````

`<trigger_input>` 值包含函式預期格式的資料。 下列 cURL 範例是 POST 到 `QueueTriggerJS` 函式。 在此情況下，輸入是一個相當於在佇列中預期找到的訊息字串。

```bash
curl --request POST -H "Content-Type:application/json" --data '{"input":"sample queue data"}' http://localhost:7071/admin/functions/QueueTriggerJS
```

#### <a name="using-the-func-run-command-in-version-1x"></a>使用版本 1.x 中的 `func run` 命令

>[!IMPORTANT]  
> 工具的版本 2.x 不支援 `func run` 命令。 如需詳細資訊，請參閱[如何設定 Azure Functions 執行階段版本目標](set-runtime-version.md)主題。

您也可以使用 `func run <FunctionName>` 直接叫用函式，並為函式提供輸入資料。 此命令類似於使用 Azure 入口網站中的 [測試] 索引標籤執行函式。 

`func run` 支援下列選項：

| 選項     | 說明                            |
| ------------ | -------------------------------------- |
| **`--content -c`** | 內嵌內容。 |
| **`--debug -d`** | 在執行函式之前，請先將偵錯工具附加到主機處理序。|
| **`--timeout -t`** | 本機 Functions 主機就緒前的等候時間 (以秒為單位)。|
| **`--file -f`** | 要用來作為內容的檔案名稱。|
| **`--no-interactive`** | 不會提示輸入。 適用於自動化情節。|

例如，若要呼叫 HTTP 觸發的函式並傳遞內容的內文，請執行下列命令：

```bash
func run MyHttpTrigger -c '{\"name\": \"Azure\"}'
```

### <a name="viewing-log-files-locally"></a>在本機檢視記錄檔

[!INCLUDE [functions-local-logs-location](../../includes/functions-local-logs-location.md)]

## <a name="publish"></a>發佈至 Azure

若要將 Functions 專案發佈至 Azure 中的函式應用程式，請使用 `publish` 命令：

```bash
func azure functionapp publish <FunctionAppName>
```

您可以使用下列選項：

| 選項     | 說明                            |
| ------------ | -------------------------------------- |
| **`--publish-local-settings -i`** |  將 local.settings.json 中的設定發佈至 Azure，若設定已經存在，則提示進行覆寫。 如果您使用儲存體模擬器，請將應用程式設定變更為[實際的儲存體連接](#get-your-storage-connection-strings)。 |
| **`--overwrite-settings -y`** | 必須與 `-i` 搭配使用。 使用本機值在 Azure 中覆寫 AppSettings (如果不同)。 預設值為提示。|

此命令會發行至 Azure 中的現有函式應用程式。 訂用帳戶中沒有 `<FunctionAppName>` 時，會發生錯誤。 若要了解如何使用 Azure CLI 從命令提示字元或終端機視窗建立函式應用程式，請參閱[建立無伺服器也可執行的函式應用程式](./scripts/functions-cli-create-serverless.md)。

`publish` 命令會將 Functions 專案目錄的內容上傳。 如果您在本機將檔案刪除，`publish` 命令並不會從 Azure 刪除它們。 您可以使用 [Azure 入口網站] 中的 [Kudu 工具](functions-how-to-use-azure-function-app-settings.md#kudu)來刪除 Azure 中的檔案。  

>[!IMPORTANT]  
> 當您在 Azure 中建立函式應用程式時，預設會使用 1.x 版的 Function 執行階段。 若要讓函式應用程式使用 2.x 版的執行階段，請新增應用程式設定 `FUNCTIONS_EXTENSION_VERSION=beta`。  
使用下列 Azure CLI 程式碼，將這個設定新增至函式應用程式：

```azurecli-interactive
az functionapp config appsettings set --name <function_app> \
--resource-group myResourceGroup \
--settings FUNCTIONS_EXTENSION_VERSION=beta   
```

## <a name="next-steps"></a>後續步驟

Azure Functions Core Tools 是[開放原始碼且裝載於 GitHub 上](https://github.com/azure/azure-functions-cli)。  
若要提出錯誤或功能要求，[請開啟 GitHub 問題](https://github.com/azure/azure-functions-cli/issues)。

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Azure 入口網站]: https://portal.azure.com 
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows
