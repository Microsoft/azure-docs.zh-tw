---
title: 安裝 Durable Functions 擴充和範例 - Azure
description: 了解如何安裝 Azure Functions 的 Durable Functions 擴充，供入口網站開發或 Visual Studio 開發使用。
services: functions
author: cgillum
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 03/19/2018
ms.author: azfuncdf
ms.openlocfilehash: 12b5c013a0704eb2d3f7df58011571b902955c75
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42140653"
---
# <a name="install-the-durable-functions-extension-and-samples-azure-functions"></a>安裝 Durable Functions 擴充和範例 (Azure Functions)

NuGet 套件 [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) 中提供 Azure Functions 的 [Durable Functions](durable-functions-overview.md) 擴充。 本文說明如何為下列開發環境安裝套件和一組範例：

* Visual Studio 2017 (建議用於 C#) 
* Visual Studio Code (建議用於 JavaScript)
* Azure 入口網站

## <a name="visual-studio-2017"></a>Visual Studio 2017

使用 Durable Functions 來開發應用程式時，Visual Studio 目前提供最佳體驗。  您的函式可以在本機執行，也可以發佈至 Azure。 您可以從空白專案或一組範例函式開始。

### <a name="prerequisites"></a>必要條件

* 安裝[最新版本的 Visual Studio](https://www.visualstudio.com/downloads/) (15.3 版或更新版本)。 在您的安裝選項中包含 **Azure 開發**工作負載。

### <a name="start-with-sample-functions"></a>從範例函式開始 

1. 下載[適用於 Visual Studio 的範例應用程式 .zip 檔案](https://azure.github.io/azure-functions-durable-extension/files/VSDFSampleApp.zip)。 範例專案中已經有 NuGet 參考，您不需要再新增。
2. 安裝並執行 [Azure 儲存體模擬器](https://docs.microsoft.com/azure/storage/storage-use-emulator) 5.2 版或更新版本。 或者，您可以指定實際的 Azure 儲存體連接字串來更新 *local.appsettings.json*檔案。
3. 在 Visual Studio 2017 中開啟專案。 
4. 如需有關如何執行範例的指示，請從[函式鏈結 - Hello 序列範例](durable-functions-sequence.md)開始。 此範例可以在本機執行，或發佈至 Azure。

### <a name="start-with-an-empty-project"></a>從空白專案開始
 
就像從範例開始一樣，遵循相同的指示，但改為執行下列步驟，而不是下載 *.zip* 檔案：

1. 建立函式應用程式專案。
2. 使用「管理 NuGet 套件」並將它新增至專案來搜尋下列 NuGet 套件參考：Microsoft.Azure.WebJobs.Extensions.DurableTask v1.5.0
   
## <a name="visual-studio-code"></a>Visual Studio Code

Visual Studio Code 提供涵蓋所有主要平台 (Windows、macOS 和 Linux) 的本機開發體驗。  您的函式可以在本機執行，也可以發佈至 Azure。 您可以從空白專案或一組範例函式開始。

### <a name="prerequisites"></a>必要條件

* 安裝[最新版的 Visual Studio Code](https://code.visualstudio.com/Download) 

* 遵循[撰寫 Azure Functions 並在本機進行測試](https://docs.microsoft.com/azure/azure-functions/functions-run-local)上＜安裝 Azure Functions Core Tools＞下方的指示

    >[!IMPORTANT]
    > 如果您已經有 Azure Functions Cross Platform Tools，請將它更新為最新的可用版本。

    >[!IMPORTANT]
    >JavaScript 中的 Durable Functions 需要 2.x 版的 Azure Functions Core Tools。

*  如果您使用 Windows 電腦，請安裝並執行 [Azure 儲存體模擬器](https://docs.microsoft.com/azure/storage/storage-use-emulator) 5.2 版或更新版本。 或者，您可以使用實際的 Azure 儲存體連線來更新 *local.appsettings.json* 檔案。 


### <a name="start-with-sample-functions"></a>從範例函式開始

#### <a name="c"></a>C#

1. 複製 [Durable Functions 存放庫](https://github.com/Azure/azure-functions-durable-extension.git) \(英文\)。
2. 在您的電腦上瀏覽至 [C# 指令碼範例資料夾](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx) \(英文\)。 
3. 在命令提示字元/終端機視窗中執行下列命令來安裝 Azure Functions 永久性擴充功能：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 1.5.0
    ```
4. 在命令提示字元/終端機視窗中執行下列命令來安裝 Azure Functions Twilio 擴充功能：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.Twilio -v 3.0.0-beta5
    ```
5. 執行 Azure 儲存體模擬器，或者使用實際的 Azure 儲存體連線字串來更新 *local.appsettings.json* 檔案。
6. 在 Visual Studio Code 中開啟專案。 
7. 如需有關如何執行範例的指示，請從[函式鏈結 - Hello 序列範例](durable-functions-sequence.md)開始。 此範例可以在本機執行，或發佈至 Azure。
8. 在命令提示字元/終端機中執行下列命令來開始專案：
    ```bash
    func host start
    ```

#### <a name="javascript-functions-v2-only"></a>JavaScript (僅限 Functions v2)

1. 複製 [Durable Functions 存放庫](https://github.com/Azure/azure-functions-durable-extension.git) \(英文\)。
2. 在您的電腦上瀏覽至 [JavaScript 範例資料夾](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/javascript)。 
3. 在命令提示字元/終端機視窗中執行下列命令來安裝 Azure Functions 永久性擴充功能：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 1.5.0
    ```
4. 在命令提示字元 / 終端機視窗中執行下列命令來還原 npm 套件：
    
    ```bash
    npm install
    ``` 
5. 使用實際的 Azure 儲存體連接字串來更新 local.appsettings.json 檔案。
6. 在 Visual Studio Code 中開啟專案。 
7. 如需有關如何執行範例的指示，請從[函式鏈結 - Hello 序列範例](durable-functions-sequence.md)開始。 此範例可以在本機執行，或發佈至 Azure。
8. 在命令提示字元/終端機中執行下列命令來開始專案：
    ```bash
    func host start
    ```

### <a name="start-with-an-empty-project"></a>從空白專案開始
 
1. 在命令提示字元/終端機中，瀏覽至將裝載您函數應用程式的資料夾。
2. 在命令提示字元/終端機視窗中執行下列命令，來安裝 Azure Functions 永久性擴充功能：

    ```bash
    func extensions install -p Microsoft.Azure.WebJobs.Extensions.DurableTask -v 1.5.0
    ```
3. 執行下列命令以建立函數應用程式專案：

    ```bash
    func init
    ``` 
4. 執行 Azure 儲存體模擬器，或者使用實際的 Azure 儲存體連線字串來更新 *local.appsettings.json* 檔案。
5. 接著，執行下列命令來建立新的函式，然後依照精靈步驟執行：

    ```bash
    func new
    ```
    >[!IMPORTANT]
    > 目前並未提供永久性函式範本，但是您可以開始使用其中一個支援的選項，然後修改程式碼。 用來參考適用於[協調流程用戶端](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/HttpStart) \(英文\)、[協調流程觸發程序](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/E1_HelloSequence) \(英文\) 和[活動觸發程序](https://github.com/Azure/azure-functions-durable-extension/tree/master/samples/csx/E1_HelloSequence) \(英文\) 的範例。

6. 在 Visual Studio Code 中開啟專案資料夾，然後繼續修改範本程式碼。 
7. 在命令提示字元/終端機中執行下列命令來開始專案：
    ```bash
    func host start
    ```

## <a name="azure-portal"></a>Azure 入口網站

想要的話，您可以使用 [Azure 入口網站](https://portal.azure.com)進行 Durable Functions 開發。

   > [!NOTE]
   > JavaScript 中的 Durable Functions 尚無法使用於入口網站中。

### <a name="create-an-orchestrator-function"></a>建立協調器函式

1. 在入口網站中建立新的函式應用程式，如[函式快速入門文章](functions-create-first-azure-function.md#create-a-function-app)中所示。

2. 將函式應用程式設定成[使用 2.0 執行階段版本](set-runtime-version.md)。

   Durable Functions 擴充功能可在 1.X 執行階段和 2.0 執行階段上運作，但只有在將目標設為 2.0 執行階段時，才可使用 Azure 入口網站範本。

3. 選取 [建立您自己的自訂函式] 來建立新的函式。

4. 將 [語言] 變更為 [C#]、將 [案例] 變更為 [Durable Functions]，然後選取 [Durable Functions Http 入門 - C#] 範本。

5. 在 [擴充未安裝] 底下，按一下 [安裝]，從 NuGet.org 下載擴充。 

6. 完成安裝之後，繼續建立協調流程用戶端函式：**“HttpStart”**，此函式是透過選取 [Durable Functions Http 入門 - C#] 範本來建立的。

7. 現在，從 [Durable Functions 協調器 - C#] 範本建立協調流程函式 **"HelloSequence"**。

8. 此外，將從 [Durable Functions 活動 - C#] 範本呼叫最後一個函式 **"Hello"**。

9. 移至 **"HttpStart"** 函式，並複製其 URL。

10. 使用 Postman 或 cURL 來呼叫永久性函式。 測試之前，使用協調器函式名稱 **HelloSequence** 來取代 URL 中的 **{functionName}**。  不需要任何資料，只需使用 POST 動詞命令。 

    ```bash
    curl -X POST https://{your function app name}.azurewebsites.net/api/orchestrators/HelloSequence
    ```

11. 然後，呼叫 **"statusQueryGetUri"** 端點，而您會看到永久性函式的目前狀態

    ```json
        {
            "runtimeStatus": "Running",
            "input": null,
            "output": null,
            "createdTime": "2017-12-01T05:37:33Z",
            "lastUpdatedTime": "2017-12-01T05:37:36Z"
        }
    ```

12. 繼續呼叫 **"statusQueryGetUri"** 端點，直到狀態變成 **"Completed"** 為止 

    ```json
    {
            "runtimeStatus": "Completed",
            "input": null,
            "output": [
                "Hello Tokyo!",
                "Hello Seattle!",
                "Hello London!"
            ],
            "createdTime": "2017-12-01T05:38:22Z",
            "lastUpdatedTime": "2017-12-01T05:38:28Z"
        }
    ```

恭喜！ 您的第一個永久性函式已在 Azure 入口網站中運作！

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [執行函式鏈結範例](durable-functions-sequence.md)
