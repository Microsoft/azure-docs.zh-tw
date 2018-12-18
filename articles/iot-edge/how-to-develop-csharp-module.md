---
title: 針對適用於 Azure IoT Edge 的 C# 模組進行偵錯 | Microsoft Docs
description: 使用 Visual Studio Code 來開發、建置適用於 Azure IoT Edge 的 C# 模組以及針對其進行偵錯
services: iot-edge
keywords: ''
author: shizn
manager: timlt
ms.author: xshi
ms.date: 09/27/2018
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: 88659d31b64b4a98043606a71602f7c29316a31e
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423284"
---
# <a name="use-visual-studio-code-to-develop-and-debug-c-modules-for-azure-iot-edge"></a>使用 Visual Studio Code 來開發適用於 Azure IoT Edge 的 C# 模組以及針對其進行偵錯

您可以將商務邏輯轉換成 Azure IoT Edge 的模組。 本文說明如何使用 Visual Studio Code (VS Code) 作為主要工具，來開發 C# 模組並對其進行偵錯。

## <a name="prerequisites"></a>必要條件

您可以使用執行 Windows、macOS 或 Linux 的電腦或虛擬機器作為開發電腦。 IoT Edge 裝置可以是另一部實體裝置。

有兩種方式可讓您在 VS Code 中對 C# 模組進行偵錯。 第一個方式在模組容器中附加程序，另一個方式則是在偵錯模式中啟動模組程式碼。 如果您不熟悉 Visual Studio Code 的偵錯功能，請參閱[偵錯](https://code.visualstudio.com/Docs/editor/debugging)。

請先安裝 Visual Studio Code，然後新增下列必要的擴充功能：
* [Visual Studio Code](https://code.visualstudio.com/) 
* [Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge) 
* [C# 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) 
* [Docker 擴充功能](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)

若要建立模組，您需要 .NET 來建置專案資料夾，另外還需要 Docker 來建置模組映像，最後需要容器登錄來保存模組映像：
* [.NET Core 2.1 SDK](https://www.microsoft.com/net/download)。
* 開發電腦上的 [Docker Community Edition](https://docs.docker.com/install/)。 
* [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) 或 [Docker 中樞](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)
   * 您可以使用本機 Docker 登錄作為原型並用於測試，而非使用雲端登錄。 

若要設定本機開發環境以偵錯、執行及測試您的 IoT Edge 解決方案，您需要 [Azure IoT EdgeHub Dev 工具](https://pypi.org/project/iotedgehubdev/)。 安裝 [Python (2.7/3.6) 與 Pip](https://www.python.org/)。 然後，在您的終端機中執行下列命令，以安裝 **iotedgehubdev**。

   ```cmd
   pip install --upgrade iotedgehubdev
   ```

若要在裝置上測試您的模組，您需要一個有效的 IoT 中樞，而且該中樞中至少必須已建立一個 IoT Edge 裝置識別碼。 如果您在開發電腦上執行 IoT Edge 精靈，您可能必須先停止 EdgeHub 與 EdgeAgent 後，再移至下一個步驟。 

## <a name="create-a-new-solution-with-c-module"></a>使用 C# 模組建立新的解決方案

採取這些步驟以使用 Visual Studio Code 和 Azure IoT Edge 擴充功能，來建立以 .NET Core 2.1 為基礎的 IoT Edge 模組。 首先要建立解決方案，然後在該解決方案中產生第一個模組。 每個解決方案都可以包含多個模組。 

1. 在 Visual Studio Code 中，選取 [檢視] > [整合式終端機]。
2. 選取 [檢視] > [命令選擇區]。 
3. 在命令選擇區中，輸入並執行命令 **Azure IoT Edge: New IoT Edge Solution**。

   ![執行新的 IoT Edge 解決方案](./media/how-to-develop-csharp-module/new-solution.png)

4. 瀏覽至要用來建立新解決方案的資料夾。 選擇 [選取資料夾]。 
5. 輸入解決方案的名稱。 
6. 選取 [C# 模組] 作為解決方案中第一個模組的範本。
7. 輸入模組的名稱。 選擇容器登錄內唯一的名稱。 
8. 提供模組映像存放庫的名稱。 VS Code 會自動以 **localhost:5000** 填入模組名稱。 請使用您自己的登錄資訊加以取代。 如果您使用本機 Docker 登錄來進行測試，則可以使用 **localhost**。 如果您使用 Azure Container Registry，則請使用登錄設定中的登入伺服器。 登入伺服器看起來像**\<登錄名稱\>.azurecr.io**。 僅取代字串的 localhost 部分即可，不要刪除您的模組名稱。

   ![提供 Docker 映像存放庫](./media/how-to-develop-csharp-module/repository.png)

VS Code 會採用您提供的資訊、建立 IoT Edge 解決方案，然後將其載入至新的視窗。

   ![檢視 IoT Edge 方案](./media/how-to-develop-csharp-module/view-solution.png)

解決方案中有四個項目： 

* 一個包含偵錯組態的 **.vscode** 資料夾。
* 一個 **modules** 資料夾，其中包含每個模組的子資料夾。 此時，您只有一個資料夾。 但是您可以在命令選擇區中使用 **Azure IoT Edge: Add IoT Edge Module** 命令來新增更多項目。 
* 一個 **.env** 檔案，會列出您的環境變數。 如果您的登錄是 Azure Container Registry，您會有 Azure Container Registry 使用者名稱和密碼。 

   >[!NOTE]
   >環境檔案只會在您為模組提供了映像存放庫時才會建立。 如果您接受 localhost 預設值，並且在本機進行測試和偵錯，則不需要宣告環境變數。 

* 一個 **deployment.template.json** 檔案，會列出新的模組以及一個範例 **tempSensor** 模組，此範例模組會模擬可用於測試的資料。 如需部署資訊清單運作方式的詳細資訊，請參閱[了解如何使用部署資訊清單以部署模組和建立路由](module-composition.md)。 

## <a name="develop-your-module"></a>開發您的模組

解決方案隨附的預設 C# 模組程式碼位於 [modules] > ** [您的模組名稱] ** > [Program.cs]。 系統會設定模組和 deployment.template.json 檔案，讓您可以建置解決方案、將其推送至容器登錄，然後部署到裝置以開始測試，而不需要觸碰到任何程式碼。 模組會建置為僅採用來源的輸入 (在此案例中，是會模擬資料的 tempSensor 模組)，並且將其傳送到 IoT 中樞。 

當您準備要使用自己的程式碼自訂 C# 範本時，請使用 [Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md) 以建置模組，該模組會滿足 IoT 解決方案的主要需求，例如安全性、裝置管理和可靠性。 

VS Code 中的 C# 支援已針對跨平台 .NET Core 開發進行最佳化。 深入了解[如何在 VS Code 中使用 C#](https://code.visualstudio.com/docs/languages/csharp)。

## <a name="launch-and-debug-module-code-without-container"></a>啟動並偵錯模組程式碼，而不需要容器

IoT Edge C# 模組是 .Net Core 應用程式。 而且，此模組依存於 Azure IoT C# 裝置 SDK。 IoT C# 模組需要環境設定才能啟動並執行，因此在預設模組程式碼中，您應使用環境設定和輸入名稱來初始化 **ModuleClient**。 您也需要將訊息傳送或路由至輸入通道。 您的預設 C# 模組僅包含一個輸入通道，且其名稱為 **input1**。

### <a name="setup-iot-edge-simulator-for-single-module-app"></a>設定單一模組應用程式的 IoT Edge 模擬器

1. 若要設定並啟動模擬器，請在 VS Code 命令選擇區中輸入並選取 [Azure IoT Edge：啟動單一模組的 IoT Edge 中樞模擬器]。 您也須指定單一模組應用程式的輸入名稱；請輸入 **input1**，然後按 Enter 鍵。 此命令將會觸發 **iotedgehubdev** CLI，並啟動 IoT Edge 模擬器和測試公用程式模組容器。 如果模擬器已在單一模組模式中順利啟動，您就會在整合式終端機中看到下列輸出。 您也可以查看 `curl` 命令，以利傳送訊息。 稍後您將會用到此資訊。

   ![設定單一模組應用程式的 IoT Edge 模擬器](media/how-to-develop-csharp-module/start-simulator-for-single-module.png)

   您可以移至 Docker 總管，並查看模組的執行狀態。

   ![模擬器模組狀態](media/how-to-develop-csharp-module/simulator-status.png)

   **edgeHubDev** 容器是本機 IoT Edge 模擬器的核心。 它無需 IoT Edge 安全性精靈即可在您的開發電腦上執行，並可為您的原生模組應用程式或模組容器提供環境設定。 **輸入**容器公開的 restAPI 可協助橋接器訊息在您的模組上設定輸入通道的目標。

2. 在 VS Code 命令選擇區中輸入並選取 [Azure IoT Edge：將模組認證設定為使用者設定]，在使用者設定中將模組環境設定設為 `azure-iot-edge.EdgeHubConnectionString` 和 `azure-iot-edge.EdgeModuleCACertificateFile`。 您會發現這些環境設定在 **.vscode** > **launch.json** 和 [VS Code 使用者設定](https://code.visualstudio.com/docs/getstarted/settings)中受到參考。

### <a name="build-module-app-and-debug-in-launch-mode"></a>建置模組應用程式並在啟動模式中偵錯

1. 在整合式終端機中，將目錄切換至 **CSharpModule** 資料夾，並執行下列命令以建置 .Net Core 應用程式。

    ```cmd
    dotnet build
    ```

2. 瀏覽至 `program.cs`。 在此檔案中新增中斷點。

3. 瀏覽至 VS Code 偵錯檢視：[檢視] > [偵錯]。 從下拉式清單中選取偵錯組態 **ModuleName 本機偵錯 (.NET Core)**。 

  ![在 VS Code 中進入偵錯模式](media/how-to-develop-csharp-module/debug-view.png)

4. 按一下 [開始偵錯]，或按 **F5**。 您會啟動偵錯工作階段。

   > [!NOTE]
   > 如果您的 .Net Core `TargetFramework` 與 `launch.json` 中的程式路徑不一致， 您必須手動更新 `launch.json` 中的程式路徑，以遵循 .csproj 檔案中的 `TargetFramework`。 如此，VS Code 才能成功啟動此程式。

5. 在 VS Code 整合式終端機中執行下列命令，將 **Hello World** 訊息傳送至您的模組。 這是先前的步驟中在 IoT Edge 模擬器設定成功時所顯示的命令。

    ```cmd
    curl --header "Content-Type: application/json" --request POST --data '{"inputName": "input1","data":"hello world"}' http://localhost:53000/api/v1/messages
    ```

   > [!NOTE]
   > 如果您使用 Windows，請確定 VS Code 整合式終端機的殼層是 **Git Bash** 或 **WSL Bash**。 您無法在 PowerShell 或命令提示字元中執行 `curl` 命令。 
   
   > [!TIP]
   > 您也可以使用 [PostMan](https://www.getpostman.com/) 或其他 API 工具來傳送訊息，而不使用 `curl`。

6. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。 

    ![觀察變數](media/how-to-develop-csharp-module/single-module-variables.png)

7. 若要停止偵錯工作階段中，請按一下 [停止] 按鈕，或按 **Shift + F5**。 在 VS Code 命令選擇區中，輸入並選取 [Azure IoT Edge：停止 IoT Edge 模擬器]，以停止並清除模擬器。

## <a name="build-module-container-for-debugging-and-debug-in-attach-mode"></a>建置要偵錯的模組容器，並在附加模式中偵錯

您的預設解決方案包含兩個模組，一個是模擬溫度感應器模組，另一個則是 C# 管道模組。 模擬溫度感應器會持續將訊息傳送至 C# 管道模組，然後訊息會輸送至 IoT 中樞。 在您已建立的模組資料夾中，會有不同容器類型的多個 Docker 檔案。 請使用任何副檔名為 **.debug** 的檔案來建置測試用模組。 目前，C# 模組僅支援在附加模式中對 Linux amd64 容器進行偵錯。

### <a name="setup-iot-edge-simulator-for-iot-edge-solution"></a>設定 IoT Edge 解決方案的 IoT Edge 模擬器

在您的開發電腦中，您可以藉由啟動 IoT Edge 模擬器來執行 IoT Edge 解決方案，而不安裝 IoT Edge 安全性精靈。 

1. 在左側的裝置總管中，以滑鼠右鍵按一下您的 IoT Edge 裝置識別碼，然後選取 [設定 IoT Edge 模擬器] 以使用裝置連接字串啟動模擬器。

2. 您可以在整合式終端機中看到 IoT Edge 模擬器已成功設定。

### <a name="build-and-run-container-for-debugging-and-debug-in-attach-mode"></a>建置並執行要偵錯的容器，並在附加模式中偵錯

1. 在 VS Code 中，瀏覽至 `deployment.template.json` 檔案。 在結尾加上 **.debug**，以更新您的 C# 模組映像 URL。

   ![請將 **.debug** 新增至映像名稱](./media/how-to-develop-csharp-module/image-debug.png)

2. 瀏覽至 `program.cs`。 在此檔案中新增中斷點。

3. 在 VS Code 檔案總管中，在內容功能表中為您的解決方案選取 `deployment.template.json` 檔案，然後按一下 [在模擬器中建置並執行 IoT Edge 解決方案]。 您可以在相同的視窗中查看所有模組容器記錄。 您也可以瀏覽至 Docker 總管以查看容器狀態。

   ![觀察變數](media/how-to-develop-csharp-module/view-log.png)

4. 瀏覽至 VS Code 偵錯檢視。 為您的模組選取偵錯組態檔。 偵錯選項名稱應該類似 **ModuleName 遠端偵錯 (.NET Core)**

   ![選取組態](media/how-to-develop-csharp-module/debug-config.png)

5. 選取 [開始偵錯] 或選取 **F5**。 選取所要連結的流程。

6. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。

7. 若要停止偵錯工作階段中，請按一下 [停止] 按鈕，或按 **Shift + F5**。 在 VS Code 命令選擇區中，輸入並選取 [Azure IoT Edge：停止 IoT Edge 模擬器]。

    > [!NOTE]
    > 這個範例說明如何針對容器上的 .Net Core IoT Edge 模組進行偵錯。 它是以 `Dockerfile.debug` 的偵錯版本為基礎，其中包含建置它時容器映像中的 Visual Studio .NET Core 命令列偵錯工具 VSDBG。 針對 C# 模組進行偵錯之後，建議您針對已可供生產環境使用的 IoT Edge 模組，直接使用 `Dockerfile` 或將其自訂成不含 VSDBG。


## <a name="next-steps"></a>後續步驟

建置模組之後，請了解如何[從 Visual Studio Code 部署 Azure IoT Edge 模組](how-to-deploy-modules-vscode.md)。

若要為您的 IoT Edge 裝置開發模組，請[了解及使用 Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md)。
