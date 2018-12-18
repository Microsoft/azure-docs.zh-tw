---
title: Azure Functions 執行階段版本概觀
description: Azure Functions 支援多個執行階段版本。 了解其間的差異以及如何選擇最適合您的版本。
services: functions
documentationcenter: ''
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.topic: conceptual
ms.date: 09/14/2018
ms.author: glenga
ms.openlocfilehash: a601ea42549abad84d6cab5c429cf94147776436
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978619"
---
# <a name="azure-functions-runtime-versions-overview"></a>Azure Functions 執行階段版本概觀

 Azure Functions 執行階段有兩個主要版本：1.x 和 2.x。 兩個版本都支援生產案例，但可用的新功能和改進功能都在目前的版本 2.x 推出。  下面詳述兩個版本間的差異、如何建立每個版本，以及如何從 1.x 升級到 2.x。

> [!NOTE] 
> 本文所介紹的是雲端服務 Azure Functions。 如需可讓您執行 Azure Functions 內部部署之預覽產品的詳細資訊，請參閱 [Azure Functions 執行階段概觀](functions-runtime-overview.md)。

## <a name="creating-1x-apps"></a>建立 1.x 應用程式

在 Azure 入口網站中建立的新應用程式預設會設定為 2.x，這是最新的版本且新功能都在此版本推出。  不過，您仍然可以執行下列步驟來建立 1.x 版應用程式。

1. 從 Azure 入口網站建立 Azure 函式
1. 開啟建立的應用程式，並且在它還是空白時，開啟 [函式設定]
1. 將版本從 ~2 變更為 ~1。  *如果您的應用程式中有函式，則系統會停用此切換開關*。
1. 按一下儲存並重新啟動應用程式。  所有範本現在應該都會在 1.x 中建立並執行。

## <a name="cross-platform-development"></a>跨平台開發

執行階段 1.x 僅支援在入口網站或 Windows 上進行開發和裝載。 執行階段 2.x 會在 .NET Core 2 上執行，這表示它可以在受到 .NET Core 支援的所有平台上執行，包括 macOS 與 Linux。 這樣可支援跨平台開發及裝載案例。

## <a name="languages"></a>語言

執行階段 2.x 使用新的語言擴充性模型。 此外，為了改善工具和效能，2.x 中的每個應用程式都只能有單一語言的函式。 2.x 中目前支援的語言為 C#、F#、JavaScript 與 Java。 Azure Functions 1.x 實驗語言尚未更新為使用新模型，因此在 2.x 中不受支援。 下表說明列出每個執行階段版本所支援的程式設計語言。

[!INCLUDE [functions-supported-languages](../../includes/functions-supported-languages.md)]

如需詳細資訊，請參閱[支援的語言](supported-languages.md)。

## <a name="migrating-from-1x-to-2x"></a>從 1.x 移轉到 2.x

您可能希望將現有以 1.x 撰寫的應用程式移轉到 2.x。  在版本之間移轉所需的大部分考量，都是關於上述的語言執行階段變更 (例如 C# 從 .NET Framework 4.7 移轉到 .NET Core 2)。  您必須確認您的程式和程式庫與所使用的語言執行階段相容。  也請務必記下下面強調的觸發程序、繫結和功能的變更。

### <a name="changes-in-triggers-and-bindings"></a>觸發程序與繫結中的變更

雖然大部分的觸發程序與繫結屬性及設定在版本之間都保持相同，在 2.x 中您仍然需要將任何觸發程序或繫結安裝到應用程式。 此狀況的唯一例外是 HTTP 與計時器觸發程序。  請參閱[註冊並安裝繫結延伸模組](./functions-triggers-bindings.md#register-binding-extensions)。  請注意，`function.json` 或函式的屬性在版本之間可能也有所變更 (例如 CosmosDB `connection` 屬性現在為 `ConnectionStringSetting`)。  請參考[現有的繫結表格](#bindings)，以取得每個繫結的文件連結。

### <a name="changes-in-features-available"></a>可用的功能變更

除了語言與繫結的變更，有些功能在版本之間已經移除、更新或被取代。  在使用 1.x 之後要開始使用 2.x 時，一些主要的建議考量事項如下。  下列是在 2.x 版中的變更：

* 用於呼叫函式的索引鍵一律會儲存在加密的 Blob 儲存體中。 在 1.x 中，根據預設它們是在檔案儲存體中，而且如果啟用位置之類的功能，就可以移動到 Blob 中。  如果將 1.x 應用程式升級到 2.x，且祕密是在檔案儲存體中，則系統目前會將它們重設。
* 為了改善效能，"webhook" 類型觸發程序已被移除並由 "HTTP" 觸發程序取代。
* 主機設定(`host.json`) 應該為空白或針對其中一個屬性包含設定值為 `2.0` 的 `version`。
* 為了改進監視功能和可檢視性，WebJobs 儀表板 (`AzureWebJobsDashboard`) 已由 [Azure Application Insights](functions-monitoring.md) (`APPINSIGHTS_INSTRUMENTATIONKEY`) 取代
* 應用程式設定 (`local.settings.json`) 需要屬性 `FUNCTIONS_WORKER_RUNTIME` 的值對應至應用程式的語言 `dotnet | node | java | python`。
    * 為了改善使用量和啟動時間，應用程式被限制為單一語言。 您可以發佈多個應用程式，使函式針對相同的解決方案有不同語言的版本。
* App Service 方案中函式的預設逾時為 30 分鐘。  您仍然可以手動將它設定成無限制。
* [由於.NET Core 限制](https://github.com/Azure/azure-functions-host/issues/3414) \(英文\)，已經移除 F# 函式的 `.fsx` 指令碼。 仍然支援已編譯的 F# 函式。
* Webhook 型觸發程序 (如事件方格) 的格式已經變更為 `https://{app}/runtime/webhooks/{triggerName}`

### <a name="upgrading-a-locally-developed-application"></a>升級在本機開發的應用程式

如果您的 1.x 版應用程式是在本機開發，您可以對應用程式或專案進行變更，使它與 2 版相容。  建議您建立新的應用程式，並將程式碼移植到新的應用程式。  雖然有使現有應用程式執行就地升級的變更，1 版和 2 版之間還是有數個其他改進功能是舊版程式碼可能無法利用的 (例如 C# 中的 `TraceWriter` 變更為 `ILogger`)。  

建議的路徑是從其中一個 2 版範本開始，然後將程式碼移動到新的專案或應用程式中。

#### <a name="visual-studio-runtime-versions"></a>Visual Studio 執行階段版本

當您在 Visual Studio 中建立專案時，您要選取執行階段版本。  Visual Studio 有兩個主要版本的位元，而且可以針對專案動態地使用正確的版本。  這些設定衍生自 `.csproj` 檔案。  針對 1.x 應用程式，專案有下列屬性

```xml
<TargetFramework>net461</TargetFramework>
<AzureFunctionsVersion>v1</AzureFunctionsVersion>
```

在 2 版中，專案屬性為

```xml
<TargetFramework>netstandard2.0</TargetFramework>
<AzureFunctionsVersion>v2</AzureFunctionsVersion>
```

按一下 [偵錯] 或 [發行] 會正確設定專案設定的正確版本。

#### <a name="vs-code-and-azure-functions-core-tools"></a>VS Code 與 Azure Functions Core Tools

其他本機工具依賴 Azure Functions Core Tools。  這些工具安裝在電腦上，而且通常在開發電腦上一次只會安裝一個版本。  請參閱[如何安裝特定版本核心工具的指示](./functions-run-local.md)。

針對 VS Code，您可能也必須更新 `azureFunctions.projectRuntime` 的使用者設定，以符合所安裝工具的版本。  這也會更新在新應用程式建立期間所呈現的範本與語言。

### <a name="changing-version-of-apps-in-azure"></a>在 Azure 中變更應用程式版本

已發佈應用程式版本是透過應用程式設定 `FUNCTIONS_RUNTIME_VERSION` 所設定。  設定為 `~2` 代表 2 版應用程式，而 `~1` 代表 1 版應用程式。  如果應用程式中已有現有函式發佈至其中，則除非一併變更那些函式的程式碼，否則強烈建議不要變更應用程式的執行階段版本。  建議的路徑是建立新的函式應用程式並設定適當的版本、測試變更，然後停用或刪除先前的應用程式。

## <a name="bindings"></a>繫結 

執行階段 2.x 使用新的[繫結擴充性模型](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview)，該模型提供下列優點：

* 支援第三方繫結延伸模組。
* 分開處理執行階段和繫結。 這可讓繫結延伸模組個別建立版本和發行。 舉例來說，您可以選擇升級至依賴較新版基礎 SDK 的延伸模組版本。
* 較輕便的執行環境，執行階段只會知道及載入使用中的繫結。

所有內建 Azure Functions 繫結均已採用此模型，除了計時器觸發程序和 HTTP 觸發程序以外，依預設不會再包含這些繫結。 當您透過 Visual Studio 等工具或透過入口網站建立函式時，這些延伸模組會自動安裝。

下表說明每個執行階段版本所支援的繫結。

[!INCLUDE [Full bindings table](../../includes/functions-bindings.md)]

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列資源：

* [撰寫 Azure Functions 並在本機進行測試](functions-run-local.md)
* [如何設定 Azure Functions 執行階段目標版本](set-runtime-version.md)
* [版本資訊](https://github.com/Azure/azure-functions-host/releases)
