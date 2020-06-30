---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 01/12/2020
ms.author: glenga
ms.openlocfilehash: f4af3c202d4f00c4ac3041921175c92226f0db7c
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/26/2020
ms.locfileid: "76964068"
---
## <a name="run-the-function-locally"></a>在本機執行函式

Visual Studio Code 可與 [Azure Functions Core Tools](../articles/azure-functions/functions-run-local.md) 整合，讓您能夠先在本機開發電腦上執行此專案，再發佈至 Azure。

1. 若要呼叫您的函式，請按 F5 以啟動函式應用程式專案。 Core Tools 的輸出會顯示在**終端機**面板中。

1. 若您出現「不支援的偵錯類型'coreclr'」錯誤訊息，請先安裝 C# Extension ，如果您尚未安裝 Azure Functions Core Tools，請在出現提示時選取 [安裝]  。 安裝 Core Tools 後，您的應用程式將會在 [終端機]  面板中啟動。 您可以查看在本機執行的 HTTP 觸發函式的 URL 端點。 

    ![Azure 本機輸出](./media/functions-run-function-test-local-vs-code/functions-vscode-f5.png)

1. 在 Core Tools 執行時，瀏覽至下列 URL 以執行 GET 要求 (其中包含 `?name=Functions` 查詢字串)。

    <http://localhost:7071/api/HttpExample?name=Functions>

1. 此時會傳回回應，其內容在瀏覽器中會顯示如下：

    ![瀏覽器中的函式 localhost 回應](./media/functions-run-function-test-local-vs-code/functions-test-local-browser.png)

1. 要求的相關資訊會顯示在 [終端機]  面板中。

    ![終端機面板中的函式執行](./media/functions-run-function-test-local-vs-code/function-execution-terminal.png)

1. 按 Ctrl + C 以停止 Core Tools，並中斷偵錯工具的連線。
