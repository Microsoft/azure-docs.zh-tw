---
title: 對適用於 Azure IoT Edge 的 Java 模組進行偵錯 | Microsoft Docs
description: 使用 Visual Studio Code 來開發適用於 Azure IoT Edge 的 Java 模組以及對其進行偵錯
services: iot-edge
keywords: ''
author: shizn
manager: timlt
ms.author: xshi
ms.date: 09/21/2018
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: a5e86b7c93344a6c61fecaa41b98bc1daa20f36f
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423216"
---
# <a name="use-visual-studio-code-to-develop-and-debug-java-modules-for-azure-iot-edge"></a>使用 Visual Studio Code 來開發適用於 Azure IoT Edge 的 Java 模組以及對其進行偵錯

您可以將商務邏輯轉換成 Azure IoT Edge 的模組，以將商務邏輯傳送到 Edge 運作。 本文提供詳細指示，說明如何使用 Visual Studio Code (VS Code) 作為主要開發工具來開發和偵錯 Java 模組。

## <a name="prerequisites"></a>必要條件
本文假設您使用執行 Windows、macOS 或 Linux 的電腦或虛擬機器作為開發電腦。 您的 IoT Edge 裝置可以是另一部實體裝置。

> [!NOTE]
> 此偵錯文章會示範兩種在 VS Code 中對您的 Java 模組進行偵錯的常用方式。 其中一個方式在模組容器中附加程序，而另一個則是在偵錯模式中啟動模組程式碼。 如果您不熟悉 Visual Studio Code 的偵錯功能，請參閱[偵錯](https://code.visualstudio.com/Docs/editor/debugging)。

因為本文使用 Visual Studio Code 作為主要開發工具，因此請安裝 VS Code，然後新增必要的擴充：
* [Visual Studio Code](https://code.visualstudio.com/) 
* [Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge) 
* [適用於 Visual Studio Code 的 Java 擴充套件](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
* [Docker 擴充功能](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)

若要建立模組，您必須以 Java 和 Maven 來建置和執行模組程式碼、以 Docker 建置模組映像，並且以容器登錄來保存模組映像：
* [Java SE 開發套件 10](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (英文)，並[設定 `JAVA_HOME` 環境變數](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/) (英文) 指向 JDK 安裝。
* [Maven](https://maven.apache.org/)
* [Docker](https://docs.docker.com/engine/installation/)
* [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) 或 [Docker 中樞](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)
   * 您可以使用本機 Docker 登錄作為原型並用於測試，而非使用雲端登錄。 

若要設定本機開發環境以偵錯、執行及測試您的 IoT Edge 解決方案，您需要 [Azure IoT EdgeHub Dev 工具](https://pypi.org/project/iotedgehubdev/)。 安裝 [Python (2.7/3.6) 與 Pip](https://www.python.org/)。 然後，在您的終端機中執行下列命令，以安裝 **iotedgehubdev**。

   ```cmd
   pip install --upgrade iotedgehubdev
   ```

若要在裝置上測試您的模組，您需要一個有效的 IoT 中樞，而且該中樞中至少必須已建立一個 IoT Edge 裝置識別碼。 如果您在開發電腦上執行 IoT Edge 精靈，您可能必須先停止 EdgeHub 與 EdgeAgent 後，再移至下一個步驟。 


## <a name="create-a-new-solution-template"></a>建立新的解決方案範本

下列步驟示範如何使用 Visual Studio Code 和 Azure IoT Edge 延伸模組，來建立以 Java 為基礎的 IoT Edge 模組。 您開始建立解決方案，然後在該解決方案中產生第一個模組。 每個解決方案都可以包含多個模組。 

1. 在 Visual Studio Code 中，選取 [檢視] > [整合式終端機]。

3. 在 Visual Studio Code 中，選取 [檢視] > [命令選擇區]。 
4. 在命令選擇區中，輸入並執行命令 **Azure IoT Edge: New IoT Edge Solution**。

   ![執行新的 IoT Edge 解決方案](./media/how-to-develop-csharp-module/new-solution.png)

5. 瀏覽至您要建立新解決方案所在的資料夾，然後按一下 [選取資料夾]。 
6. 提供解決方案的名稱。 
7. 選擇 [Java 模組] 作為解決方案中第一個模組的範本。
8. 提供模組的名稱。 選擇容器登錄內唯一的名稱。 
8. 提供 groupId 的值，或接受預設值 **com.edgemodule**。
9. 提供模組的映像存放庫。 VS Code 會自動填寫模組名稱，因此您只需要將 **localhost:5000** 取代為自己的登錄資訊即可。 如果您使用本機 Docker 登錄來進行測試，則可以使用 localhost。 如果您使用 Azure Container Registry，則請使用登錄設定中的登入伺服器。 登入伺服器看起來像**\<登錄名稱\>.azurecr.io**。 僅取代字串的 localhost 部分即可，不要刪除您的模組名稱。

   ![提供 Docker 映像存放庫](./media/how-to-develop-node-module/repository.png)

VS Code 會採用您提供的資訊、建立 IoT Edge 解決方案，然後將它載入至新的視窗。

在解決方案中，您有三個項目： 
* 一個包含偵錯組態的 **.vscode** 資料夾。
* 一個 **modules** 資料夾，它包含每個模組的子資料夾。 現在您只有一個模組資料夾，但是可以在命令選擇區中，使用命令 **Azure IoT Edge: Add IoT Edge Module** 來新增更多模組資料夾。 
* 一個 **.env** 檔案，它會列出您的環境變數。 如果您的登錄是 Azure Container Registry，您會有 Azure Container Registry 使用者名稱和密碼。

   >[!NOTE]
   >環境檔案只會在您為模組提供了映像存放庫時才會建立。 如果您接受 localhost 預設值，並且在本機進行測試和偵錯，則不需要宣告環境變數。 

* 一個 **deployment.template.json** 檔案，它會列出新的模組以及一個範例 **tempSensor** 模組，此範例模組會模擬可用於測試的資料。 如需部署資訊清單運作方式的詳細資訊，請參閱[了解如何使用、設定以及重複使用 IoT Edge 模組](module-composition.md)。

## <a name="develop-your-module"></a>開發您的模組

解決方案隨附的預設 Java 程式碼位於：**modules > [您的模組名稱] > src > main > java > com > edgemodulemodules > App.java**。 系統會設定模組和 deployment.template.json 檔案，讓您可以建置解決方案、將其推送至容器登錄，然後部署到裝置以開始測試，而不需要觸碰到任何程式碼。 模組會建置為僅採用來源的輸入 (在此案例中，是會模擬資料的 tempSensor 模組)，並且將其傳送到 IoT 中樞。 

當您準備要使用自己的程式碼自訂 Java 範本時，請使用 [Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md) 以建置模組，該模組會滿足 IoT 解決方案的主要需求，例如安全性、裝置管理和可靠性。 

Visual Studio Code 支援 Java。 深入了解[如何在 VS Code 中使用 Java](https://code.visualstudio.com/docs/java/java-tutorial)。

## <a name="launch-and-debug-module-code-without-container"></a>啟動並偵錯模組程式碼，而不需要容器
IoT Edge Java 模組相依於 Azure IoT Java 裝置 SDK。 在預設模組程式碼中，您應使用環境設定和輸入名稱來初始化 **ModuleClient**，這表示 IoT Edge Java 模組需要環境設定才能啟動並執行，而且您也需要將訊息傳送或路由至輸入通道。 您的預設 Java 模組僅包含一個輸入通道，且其名稱為 **input1**。

### <a name="setup-iot-edge-simulator-for-single-module-app"></a>設定單一模組應用程式的 IoT Edge 模擬器

1. 若要設定並啟動模擬器，請在 VS Code 命令選擇區中輸入並選取 [Azure IoT Edge：啟動單一模組的 IoT Edge 中樞模擬器]。 您也須指定單一模組應用程式的輸入名稱；請輸入 **input1**，然後按 Enter 鍵。 此命令將會觸發 **iotedgehubdev** CLI，並啟動 IoT Edge 模擬器和測試公用程式模組容器。 如果模擬器已在單一模組模式中順利啟動，您就會在整合式終端機中看到下列輸出。 您也可以查看 `curl` 命令，以利傳送訊息。 稍後您將會用到此資訊。

   ![設定單一模組應用程式的 IoT Edge 模擬器](media/how-to-develop-csharp-module/start-simulator-for-single-module.png)

   您可以移至 Docker 總管，並查看模組的執行狀態。

   ![模擬器模組狀態](media/how-to-develop-csharp-module/simulator-status.png)

   **edgeHubDev** 容器是本機 IoT Edge 模擬器的核心。 它無需 IoT Edge 安全性精靈即可在您的開發電腦上執行，並可為您的原生模組應用程式或模組容器提供環境設定。 **輸入**容器公開的 restAPI 可協助橋接器訊息在您的模組上設定輸入通道的目標。

2. 在 VS Code 命令選擇區中輸入並選取 [Azure IoT Edge：將模組認證設定為使用者設定]，在使用者設定中將模組環境設定設為 `azure-iot-edge.EdgeHubConnectionString` 和 `azure-iot-edge.EdgeModuleCACertificateFile`。 您會發現這些環境設定在 **.vscode** > **launch.json** 和 [VS Code 使用者設定](https://code.visualstudio.com/docs/getstarted/settings)中受到參考。

### <a name="debug-java-module-in-launch-mode"></a>在啟動模式中對 Java 模組進行偵錯

2. 瀏覽至 `App.java`。 在此檔案中新增中斷點。

3. 瀏覽至 VS Code 偵錯檢視。 選取偵錯組態 **ModuleName 本機偵錯 (Java)**。 

4. 按一下 [開始偵錯]，或按 **F5**。 您會啟動偵錯工作階段。

5. 在 VS Code 整合式終端機中執行下列命令，將 **Hello World** 訊息傳送至您的模組。 這是先前的步驟中在 IoT Edge 模擬器設定成功時所顯示的命令。 如果目前的終端機被封鎖，您可能需要建立或切換至另一個整合式終端機。

    ```cmd
    curl --header "Content-Type: application/json" --request POST --data '{"inputName": "input1","data":"hello world"}' http://localhost:53000/api/v1/messages
    ```

   > [!NOTE]
   > 如果您使用 Windows，請確定 VS Code 整合式終端機的殼層是 **Git Bash** 或 **WSL Bash**。 您無法在 PowerShell 或命令提示字元中執行 `curl` 命令。 
   
   > [!TIP]
   > 您也可以使用 [PostMan](https://www.getpostman.com/) 或其他 API 工具來傳送訊息，而不使用 `curl`。

6. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。 

7. 若要停止偵錯工作階段中，請按一下 [停止] 按鈕，或按 **Shift + F5**。 在 VS Code 命令選擇區中，輸入並選取 [Azure IoT Edge：停止 IoT Edge 模擬器]，以停止並清除模擬器。


## <a name="build-module-container-for-debugging-and-debug-in-attach-mode"></a>建置要偵錯的模組容器，並在附加模式中偵錯

您的預設解決方案包含兩個模組，一個是模擬溫度感應器模組，另一個則是 Java 管道模組。 模擬溫度感應器會持續將訊息傳送至 Java 管道模組，然後訊息會輸送至 IoT 中樞。 在您已建立的模組資料夾中，會有不同容器類型的多個 Docker 檔案。 請使用任何副檔名為 **.debug** 的檔案來建置測試用模組。 Java 模組目前只支援在 linux-amd64 及 linux-arm32v7 容器中進行偵錯。

### <a name="setup-iot-edge-simulator-for-iot-edge-solution"></a>設定 IoT Edge 解決方案的 IoT Edge 模擬器

在您的開發電腦中，您可以藉由啟動 IoT Edge 模擬器來執行 IoT Edge 解決方案，而不安裝 IoT Edge 安全性精靈。 

1. 在左側的裝置總管中，以滑鼠右鍵按一下您的 IoT Edge 裝置識別碼，然後選取 [設定 IoT Edge 模擬器] 以使用裝置連接字串啟動模擬器。

2. 您可以在整合式終端機中看到 IoT Edge 模擬器已成功設定。

### <a name="build-and-run-container-for-debugging-and-debug-in-attach-mode"></a>建置並執行要偵錯的容器，並在附加模式中偵錯

1. 在 VS Code 中，瀏覽至 `deployment.template.json` 檔案。 在結尾加上 **.debug**，以更新您的模組映像 URL。

2. 將 **deployment.template.json** 中的 Java 模組 createOptions 取代為以下內容並儲存此檔案： 
    ```json
    "createOptions":"{\"HostConfig\":{\"PortBindings\":{\"5005/tcp\":[{\"HostPort\":\"5005\"}]}}}"
    ```

5. 瀏覽至 VS Code 偵錯檢視。 為您的模組選取偵錯組態檔。 偵錯選項名稱應該類似 **ModuleName 遠端偵錯 (Java)**。

6. 選取 [開始偵錯] 或選取 **F5**。 選取所要連結的流程。

7. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。

8. 若要停止偵錯工作階段中，請按一下 [停止] 按鈕，或按 **Shift + F5**。 在 VS Code 命令選擇區中，輸入並選取 [Azure IoT Edge：停止 IoT Edge 模擬器]。

> [!NOTE]
> 上述範例說明如何針對容器上的 Java IoT Edge 模組進行偵錯。 這樣會在您的模組容器 createOptions 中新增公開連接埠。 完成 Java 模組偵錯之後，建議您針對已可供生產環境使用的 IoT Edge 模組，移除這些公開連接埠。

## <a name="next-steps"></a>後續步驟

建置模組之後，了解如何[從 Visual Studio Code 部署 Azure IoT Edge 模組](how-to-deploy-modules-vscode.md)

若要為您的 IoT Edge 裝置開發模組，請[了解及使用 Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md)。
