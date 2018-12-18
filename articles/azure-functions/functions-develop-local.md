---
title: 開發 Azure 函數並在本機執行 | Microsoft Docs
description: 在於 Azure Functions 上執行 Azure 函式之前，先了解如何撰寫 Azure 函式並在本機電腦上進行測試。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 09/04/2018
ms.author: glenga
ms.openlocfilehash: 0f1d4fdbb9e5d2b05b8ec43c1393951a4c6a87b8
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44094754"
---
# <a name="code-and-test-azure-functions-locally"></a>撰寫 Azure Functions 並在本機進行測試

雖然您能夠在 [Azure 入口網站]中開發及測試 Azure Functions，但許多開發人員仍偏好本機開發體驗。 Functions 可讓您輕鬆使用最喜愛的程式碼編輯器及開發工具，在本機電腦上建立及測試函式。 您的本機函式可以連線到即時 Azure 服務，而且您可以在本機電腦上使用完整的 Functions 執行階段進行偵錯。

## <a name="local-development-environments"></a>本機開發環境

您在本機電腦上開發函式的方式取決於您的[語言](supported-languages.md)和工具喜好設定。 下表中的環境支援本機開發：

|環境                              |語言         |說明|
|-----------------------------------------|------------|---|
| [命令提示字元或終端機](functions-run-local.md) | [C# (類別程式庫)](functions-dotnet-class-library.md)、[C# script (.csx)](functions-reference-csharp.md)、[JavaScript](functions-reference-node.md) | [Azure Functions Core Tools] 提供核心執行階段和範本來建立函式，以便進行本機開發。 2.x 版支援在 Linux、MacOS 和 Windows 上進行開發。 所有環境都依賴 Core Tools 執行本機 Functions 執行階段。 |
|[Visual Studio Code](functions-create-first-function-vs-code.md)| [C# (類別程式庫)](functions-dotnet-class-library.md)、[C# script (.csx)](functions-reference-csharp.md)、[JavaScript](functions-reference-node.md) | [適用於 VS Code 的 Azure Functions 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)將 Functions 支援新增至 VS Code。 需要 Core Tools。 使用 2.x 版 Core Tools 時，支援在 Linux、MacOS 和 Windows 上進行開發。 若要深入了解，請參閱[使用 Visual Studio Code 建立第一個函式](functions-create-first-function-vs-code.md)。 |
| [Visual Studio 2017](functions-develop-vs.md) | [C# (類別庫)](functions-dotnet-class-library.md) | [Visual Studio 2017 15.5 版](https://www.visualstudio.com/vs/) 與更新版本的 **Azure 開發**工作負載中包含了 Azure Functions 工具。 讓您編譯類別庫中的函式，並將 .dll 發佈至 Azure。 包括用於本機測試的 Core Tools。 若要深入了解，請參閱[使用 Visual Studio 開發 Azure Functions](functions-develop-vs.md)。 |
| [Maven](functions-create-first-java-maven.md) (各種) | [Java](functions-reference-java.md) | 與 Core Tools 整合以便進行 Java 函式開發。 2.x 版支援在 Linux、MacOS 和 Windows 上進行開發。 若要深入了解，請參閱[使用 Java 和 Maven 建立您的第一個函式](functions-create-first-java-maven.md)。 也支援使用 [Eclipse](functions-create-maven-eclipse.md) 和 [IntelliJ IDEA](functions-create-maven-intellij.md) 進行開發 |

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

每個本機開發環境都可讓您建立函式應用程式專案，並使用預先定義的 Functions 範本來建立新的函式。 每個環境都會使用 Core Tool，以便您在自己的電腦上針對實際的 Functions 執行階段進行您的函式測試和偵錯，就如同處理任何其他應用程式一樣。 您也可以從上述任何環境將您的函式應用程式專案發佈至 Azure。  

## <a name="next-steps"></a>後續步驟

+ 若要深入了解如何使用 Visual Studio 2017 在本機開發已編譯的 C# 函式，請參閱[使用 Visual Studio 開發 Azure Functions](functions-develop-vs.md)。
+ 若要深入了解如何在 Mac、Linux 或 Windows 電腦上使用 VS Code 進行函式的本機開發，請參閱[適用於 Azure Functions 的 VS Code 文件](https://code.visualstudio.com/tutorials/functions-extension/getting-started)。
+ 若要深入了解如何從命令提示字元或終端機開發函式，請參閱[使用 Azure Functions Core Tools](functions-run-local.md)。

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Azure 入口網站]: https://portal.azure.com 
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows
