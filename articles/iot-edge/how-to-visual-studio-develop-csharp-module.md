---
title: 在 Visual Studio 2017 中開發適用於 Azure IoT Edge 的 C# 模組並針對其進行偵錯 | Microsoft Docs
description: 使用 Visual Studio 2017 來開發適用於 Azure IoT Edge 的 C# 模組並針對其進行偵錯
services: iot-edge
keywords: ''
author: shizn
manager: timlt
ms.author: xshi
ms.date: 09/24/2018
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: 9cc8e1db577859ad7637902a5ccd5a044efcd033
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978517"
---
# <a name="use-visual-studio-2017-to-develop-and-debug-c-modules-for-azure-iot-edge-preview"></a>使用 Visual Studio 2017 來開發適用於 Azure IoT Edge 的 C# 模組並針對其進行偵錯 (預覽)

您可以將商務邏輯轉換成 Azure IoT Edge 的模組。 此文章說明如何使用 Visual Studio 2017 作為主要工具，來開發 C# 模組並真對其進行偵錯。

Azure IoT Edge Tools for Visual Studio 提供下列優點：

- 在本機開發電腦上建立、編輯、建置、執行 Azure IoT Edge 解決方案和模組並針對其進行偵錯。
- 透過 Azure IoT 中樞將您的 Azure IoT Edge 解決方案部署到 Azure IoT Edge 裝置。
- 以 C# 撰寫 Azure IoT 模組程式碼，同時享有 Visual Studio 開發的所有優點。
- 使用 UI 管理 Azure IoT Edge 裝置與模組。 

此文章說明如何使用 Azure IoT Edge Tools for Visual Studio 2017 以 C# 開發 IoT Edge 模組。 您也將學習如何將專案部署到 Azure IoT Edge 裝置。

## <a name="prerequisites"></a>必要條件
此文章假設您使用執行 Windows 的電腦或虛擬機器作為開發電腦。 您的 IoT Edge 裝置可以是另一部實體裝置。

因為此文章使用 Visual Studio 2017 作為主要開發工具，所以請安裝 Visual Studio。 請確認您的 Visual Studio 2017 安裝中包含 **Azure 開發工作負載**。 您可以[修改 Visual Studio 2017](https://docs.microsoft.com/visualstudio/install/modify-visual-studio?view=vs-2017) 並新增 Azure 開發工作負載。

Visual Studio 2017 準備就緒之後，您還需要：

- 從 Visual Studio Marketplace 下載並安裝 [Azure IoT Edge 延伸模組](https://marketplace.visualstudio.com/items?itemName=vsc-iot.azureiotedgetools)，以在 Visual Studio 2017 中建立 IoT Edge 專案。
- 開發電腦上的 [Docker Community Edition](https://docs.docker.com/install/)，以建置並執行您的模組映像。 您需要適當地設定在 Linux 容器模式或 Windows 容器模式中執行的 Docker CE。
- 若要設定本機開發環境以偵錯、執行及測試您的 IoT Edge 解決方案，您需要 [Azure IoT EdgeHub Dev Tool](https://pypi.org/project/iotedgehubdev/)。 安裝 [Python (2.7/3.6) 與 Pip](https://www.python.org/)。 接著，在您的終端機中執行下列命令，以安裝 **iotedgehubdev**。 確定您的 Azure IoT EdgeHub Dev Tool 版本大於 0.3.0。

   ```cmd
   pip install --upgrade iotedgehubdev
   ```

- [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/) 或 [Docker 中樞](https://docs.docker.com/docker-hub/repos/#viewing-repository-tags)
   - 您可以使用本機 Docker 登錄作為原型並用於測試，而非使用雲端登錄。 

- 若要測試您的模組，您需要一個有效的 IoT 中樞，而且該中樞中至少必須已建立一個 IoT Edge 裝置識別碼。 如果您在開發電腦上執行 IoT Edge 安全性精靈，您可能必須先停止 EdgeHub 與 EdgeAgent，才能開始在 Visual Studio 中進行開發。 

### <a name="check-your-tools-version"></a>檢查您的工具版本

1. 在 [工具] 功能表中，選擇 [擴充功能和更新]。 依序展開 [已安裝] > [工具]，您可以找到 **Azure IoT Edge** 與 **Cloud Explorer**。

2. 記下已安裝的版本。 您可以將此版本與 Visual Studio Marketplace 上的最新版本進行比較 ([Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS)、[Azure IoT Edge](https://marketplace.visualstudio.com/items?itemName=vsc-iot.azureiotedgetools))

3. 如果您的版本較舊，請依照下一節中的說明在 Visual Studio 中更新您的工具。

### <a name="update-your-tools"></a>更新您的工具

1. 在 [擴充功能和更新] 對話方塊中，展開 [更新] > [Visual Studio Marketplace]，選擇 [Azure IoT Edge] 或 [Cloud Explorer]，然後選取 [更新]。

2. 下載工具更新之後，請關閉 Visual Studio，以使用 VSIX 安裝程式觸發工具更新。

3. 在安裝程式中選擇 [確定] 以啟動，然後選擇 [修改] 以更新工具。

4. 更新完成後，請選擇 [關閉] 並重新啟動 Visual Studio。

### <a name="create-an-azure-iot-edge-project"></a>建立 Azure IoT Edge 專案

Visual Studio 中的 Azure IoT Edge 專案範本可建立可以在 Azure IoT 中樞中部署到 Azure IoT Edge 裝置的專案。 首先要建立 Azure IoT Edge 解決方案，然後在該解決方案中產生第一個 C# 模組。 每個 IoT Edge 解決方案都可以包含多個模組。 

1. 在 Visual Studio 中，從 [檔案] 功能表中選取 [新增]  > [專案]。

2. 在 [新增專案] 對話方塊中，選取 [已安裝]，展開 [Visual C#] > [雲端]，選取 [Azure IoT Edge]，輸入專案的 [名稱] 並指定位置，然後按一下 [確定]。 預設專案名稱是 **AzureIoTEdgeApp1**。 

   ![新增專案](./media/how-to-visual-studio-develop-csharp-module/create-new.jpg)

3. 在 [IoT Edge 模組設定] 視窗中，選取 [C# 模組類型]，並指定您的模組名稱與模組映像存放庫。  VS 會自動以 **localhost:5000** 填入模組名稱。 請使用您自己的登錄資訊來取代它。 如果您使用本機 Docker 登錄來進行測試，則可以使用 localhost。 如果您使用 Azure Container Registry，則請使用登錄設定中的登入伺服器。 登入伺服器看起來像 **<registry name>.azurecr.io**。 僅取代字串的 localhost 部分即可，不要刪除您的模組名稱。 預設的模組名稱是 **IoTEdgeModule1**

4. 按一下 [確定] 以使用 C# 模組建立 Azure IoT Edge 專案。

現在，我們的解決方案中有 **AzureIoTEdgeApp1** 專案與 **IoTEdgeModule1** 專案。 **AzureIoTEdgeApp1** 專案具有 **deployment.template.json** 檔案。 此檔案會定義要為 IoT Edge 解決方案建置及部署的模組，並定義模組之間的路由。 預設解決方案具有 **tempSensor** 模組與 **IoTEdgeModule1** 模組。 **tempSensor** 模組會產生 **IoTEdgeModule1** 模組的模擬資料，而 **IoTEdgeModule1** 模組中的預設程式碼是管道訊息模組，它會直接將收到的訊息透過管道傳送到 IoT 中樞。

**IoTEdgeModule1** 專案是 .Net Core 2.1 主控台應用程式。 它包含執行 Windows 容器或 Linux 容器之 IoT Edge 裝置所需的 **Dockerfiles**。 **module.json** 檔案描述模組的中繼資料。 **program.cs** 是實際的模組程式碼，它將 Azure IoT 裝置 SDK 作為相依性。

## <a name="develop-your-module"></a>開發您的模組

解決方案隨附的預設 C# 模組程式碼位於 **IoTEdgeModule1** > **Program.cs**。 系統會設定模組和 deployment.template.json 檔案，讓您可以建置解決方案、將其推送至容器登錄，然後部署到裝置以開始測試，而不需要觸碰到任何程式碼。 模組會建置為僅採用來源的輸入 (在此案例中，是會模擬資料的 tempSensor 模組)，並且將其傳送到 IoT 中樞。 

當您準備要使用自己的程式碼自訂 C# 範本時，請使用 [Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md) 以建置模組，該模組會滿足 IoT 解決方案的主要需求，例如安全性、裝置管理和可靠性。 

## <a name="build-and-debug-single-c-module"></a>建置單一 C# 模組並針對其進行偵錯

一般而言，我們希望先針對每個模組進行測試/偵錯，然後才在使用多個模組的整個解決方案中執行。

1. 在快顯功能表中選取 **IoTEdgeModule1** 作為啟始專案。

   ![設定啟始專案](./media/how-to-visual-studio-develop-csharp-module/module-start-up-project.png)

2. 按 **F5** 或按一下下面的按鈕執行模組，第一次可能需要 10 到 20 秒。

   ![執行模組](./media/how-to-visual-studio-develop-csharp-module/run-module.png)


3. 如果模組已成功初始化，您應該會看到 .Net Core 主控台應用程式已啟動。

   ![執行中的模組](./media/how-to-visual-studio-develop-csharp-module/single-module-run.png)

4. 現在，您可以在 **Program.cs** 中的 **PipeMessage** 中設定中斷點，然後透過在 **Git Bash** 或 **WSL Bash** 中執行下列命令來傳送訊息 (不要在 CMD 或 Powershell 中執行它) (您也可以在 [VS 輸出] 視窗中找到此命令)：

    ```cmd
    curl --header "Content-Type: application/json" --request POST --data '{"inputName": "input1","data":"hello world"}' http://localhost:53000/api/v1/messages
    ```

   ![針對單一模組進行偵錯](./media/how-to-visual-studio-develop-csharp-module/debug-single-module.png)

    中斷點應該已觸發。 您可以在 Visual Studio [區域變數] 視窗中觀察變數。

   > [!TIP]
   > 您也可以使用 [PostMan](https://www.getpostman.com/) 或其他 API 工具來傳送訊息，而不使用 `curl`。

5. 按 **Ctrl + F5** 或按一下 [停止] 按鈕以停止偵錯。

## <a name="build-and-debug-iot-edge-solution-with-multiple-modules"></a>建置使用多個模組的 IoT Edge 解決方案以及針對其進行偵錯

在我們完成開發單一模組之後，接下來，我們想要執行使用多個模組的整個解決方案以及針對其進行偵錯。

1. 將第二個 C# 模組新增到解決方案中。 以滑鼠右鍵按一下 **AzureIoTEdgeApp1**，然後選取 [新增] -> [新 IoT Edge 模組]。 第二個模組的預設名稱是 **IoTEdgeModule2**，而且它仍然是管道模組。

2. 瀏覽到 **deployment.template.json**。 您將在 [模組] 區段中看到已加入 **IoTEdgeModule2**。 以下列程式碼取代 **routes** 區段。 如果您已自訂模組名稱，請確定在取代之後更新下列名稱。

    ```json
        "routes": {
          "IoTEdgeModule1ToIoTHub": "FROM /messages/modules/IoTEdgeModule1/outputs/* INTO $upstream",
          "sensorToIoTEdgeModule1": "FROM /messages/modules/tempSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/IoTEdgeModule1/inputs/input1\")",
          "IoTEdgeModule2ToIoTHub": "FROM /messages/modules/IoTEdgeModule2/outputs/* INTO $upstream",
          "sensorToIoTEdgeModule2": "FROM /messages/modules/tempSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/IoTEdgeModule2/inputs/input1\")"
        },
    ```

3. 在快顯功能表中將 **AzureIoTEdgeApp1** 專案設定為啟始專案。

4. 設定中斷點並按 F5，然後您可以同時執行多個模組並針對其進行偵錯。 您應該會看到多個 .Net Core 主控台應用程式視窗，每個視窗都指出您的 C# 模組。 

   ![針對多個模組進行偵錯](./media/how-to-visual-studio-develop-csharp-module/debug-multiple-modules.png)

5. 按 **Ctrl + F5** 或按一下 [停止] 按鈕以停止偵錯。

## <a name="build-and-push-images"></a>建置及推送映像

1. 確定 **AzureIoTEdgeApp1** 是您的啟始專案。 為要建置的模組映像選取 [偵錯] 或 [發行] 設定。

    > [!NOTE]
    > 當選擇 [偵錯] 時，VS 將使用 `Dockerfile.debug` 來建置 Docker 映像。 這包含建置它時容器映像中的 .NET Core 命令列偵錯工具 VSDBG。 我們建議您改為使用 [發行] 設定，此設定針對生產就緒 IoT Edge 模組使用無 VSDBG 的 `Dockerfile`。

2. 如果您使用的是 Azure 容器登錄之類的私人登錄，請在終端機中使用下列命令執行 Docker 記錄檔。 如果您使用的是本機登錄，您可以[執行本機登錄](https://docs.docker.com/registry/deploying/#run-a-local-registry)。

    ```cmd
    docker login -u <ACR username> -p <ACR password> <ACR login server> 
    ```

3. 如果您使用的是 Azure 容器登錄之類的私人登錄，則需要將登錄使用者名稱與密碼放在執行階段設定下的 `deployment.template.json` 中，並包含下列內容。 請務必使用您的實際系統管理員使用者名稱與密碼取代預留位置。

    ```json
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {
              "registry1": {
                "username": "<username>",
                "password": "<password>",
                "address": "<registry name>.azurecr.io"
              }
            }
          }
    ```

4. 以滑鼠右鍵按一下 **AzureIoTEdgeApp1** 並選擇快顯功能表項目 [建置並推送 Edge 解決方案]，它會建置並推送每個模組的 Docker 映像。

   ![建置並推送映像](./media/how-to-visual-studio-develop-csharp-module/build-and-push.png)


## <a name="deploy-the-solution"></a>部署解決方案

在您用來設定 IoT Edge 裝置的快速入門文章中，您使用 Azure 入口網站部署了模組。 您也可以使用適用於 Visual Studio 的 Cloud Explorer 來部署模組。 您已備妥您的案例所需的部署資訊清單，即 `deployment.json` 檔案。 現在您只需選取要接收部署的裝置即可。

1. 依序按一下 [檢視] > [Cloud Explorer] 以開啟 **Cloud Explorer**。 請確定您已使用您的帳戶登入 Visual Studio 2017。

2. 在 **Cloud Explorer** 中，展開您的訂用帳戶，尋找您的 Azure IoT 中樞與您要部署的 Azure IoT Edge 裝置。

3. 以滑鼠右鍵按一下 IoT Edge 裝置為其建立部署，您必須選擇 `$AzureIoTEdgeApp1\config\Debug|Release\deployment.json` 下的部署資訊清單檔案。

4. 按一下 [重新整理] 按鈕。 您應該會看到新的模組正在與 **TempSensor** 模組以及 **$edgeAgent** 和 **$edgeHub** 一起執行。

## <a name="view-generated-data"></a>檢視產生的資料

1. 若要監視特定裝置的 D2C 訊息，請按一下清單中的裝置，然後按一下 [動作] 視窗中的 [開始監視 D2C 訊息]。

2. 若要停止監視資料，請按一下清單中的裝置，然後在 [動作] 視窗中選取 [停止監視 D2C 訊息]。

## <a name="next-steps"></a>後續步驟

若要為您的 IoT Edge 裝置開發模組，請[了解及使用 Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md)。
