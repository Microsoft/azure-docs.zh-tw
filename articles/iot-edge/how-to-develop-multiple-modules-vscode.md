---
title: 在 VS Code 中使用多個 Azure IoT Edge 模組 | Microsoft Docs
description: 使用適用於 Visual Studio Code 的 IoT 擴充，以針對 Azure IoT Edge 一次開發多個模組
services: iot-edge
keywords: ''
author: shizn
manager: timlt
ms.author: xshi
ms.date: 06/27/2018
ms.topic: conceptual
ms.service: iot-edge
ms.openlocfilehash: 31fe210b87a052438956d813db0d104e0f2cdb6e
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39041241"
---
# <a name="develop-an-iot-edge-solution-with-multiple-modules-in-visual-studio-code"></a>使用 Visual Studio Code 中的多個模組來開發 IoT Edge 解決方案

您可以使用 Visual Studio Code，利用多個模組來開發 Azure IoT Edge 解決方案。 本文說明如何建立、更新和部署 IoT Edge 解決方案，該解決方案會在 VS Code 中於模擬的 IoT Edge 裝置上透過管道傳送感應器資料。 

## <a name="prerequisites"></a>必要條件

若要完成本文中的步驟，請先準備好下列先決條件︰

- [Visual Studio Code](https://code.visualstudio.com/)
- [適用於 Visual Studio Code 的 Azure IoT Edge 擴充功能](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge)
- [C# for Visual Studio Code (由 OmniSharp 提供技術支援) 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)
- [Docker](https://docs.docker.com/engine/installation/)
- [.NET Core 2.1 SDK](https://www.microsoft.com/net/download)
- 至少具有一部 IoT Edge 裝置的作用中 IoT 中樞

您也需要整合 [適用於 VS Code 的 Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker) 與 Azure IoT 中樞總管，才能管理映像和容器。

## <a name="create-your-iot-edge-solution"></a>建立 IoT Edge 解決方案

1. 在 Visual Studio Code 中，選取 [檢視]  >  [整合式終端機]，以開啟整合式終端機。 

1. 在 VS Code 的 [命令選擇區]  中，輸入並執行命令 **Azure IoT Edge: New IoT Edge solution**。 選取您的工作區資料夾，並提供解決方案名稱 (預設值為 EdgeSolution)。 建立 C# 模組 (具有名稱 **PipeModule**) 作為此解決方案中的第一個使用者模組。 C# 模組的預設範本是管線模組，它會將來自上游的訊息直接管線傳送到下游。 您也需要為第一個模組指定 Docker 映像存放庫。 預設的映像存放庫是以本機 Docker 登錄 (**localhost:5000/<first module name>**) 為基礎。 您可以將它變更為 Azure Container Registry 或 Docker 中樞。 

2. VS Code 視窗會載入您的 IoT Edge 方案工作區。 根資料夾包含 **modules** 資料夾、**.vscode** 資料夾，以及部署資訊清單範本檔案。 偵錯組態位於 .vscode 資料夾中。 所有使用者模組代碼都是 modules 資料夾的子資料夾。 deployment.template.json 檔案是部署資訊清單範本。 此檔案中的部分參數會從 module.json 檔案 (存在於每個模組資料夾中) 進行剖析。

3. 將您的第二個模組新增至此解決方案專案。 有幾種方式可以將新的模組新增至目前解決方案。 輸入並執行 **Azure IoT Edge: Add IoT Edge module** 命令。 選取要更新的部署範本檔案。 或者，在模組資料夾或 deployment.template.json 檔案上按一下滑鼠右鍵，然後選取 [新增 IoT Edge 模組]。 然後會有一個下拉式清單可供選取模組類型。 選取名稱為 **PipeFunction** 的 **Azure Functions - C#** 模組，以及其 Docker 映像存放庫。 C# 函式模組的預設範本是管線模組，它會將來自上游的訊息直接管線傳送到下游。

4. 開啟 deployment.template.json 檔案。 確認該檔案會宣告三個模組和此執行階段。 訊息會產生自 tempSensor 模組。 訊息會透過 SampleModule 和 SampleFunction 模組直接進行管道傳送，然後傳送至您的 IoT 中樞。 

5. 使用下列內容來更新這些模組的路由：

   ```json
        "routes": {
          "SensorToPipeModule": "FROM /messages/modules/tempSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/PipeModule/inputs/input1\")",
          "PipeModuleToPipeFunction": "FROM /messages/modules/PipeModule/outputs/output1 INTO BrokeredEndpoint(\"/modules/PipeFunction/inputs/input1\")",
          "PipeFunctionToIoTHub": "FROM /messages/modules/PipeFunction/outputs/output1 INTO $upstream"
        },
   ```

5. 儲存這個檔案。

## <a name="build-and-deploy-your-iot-edge-solution"></a>建置和部署您的 IoT Edge 方案

1. 在 VS Code 的 [命令選擇區]  中，輸入並執行命令 **Azure IoT Edge: Build IoT Edge solution**。 根據每個模組資料夾中的 module.json 檔案，此命令會開始建置、容器化及推送每個模組 Docker 映像。 此命令會接著將必要值傳遞至 deployment.template.json 檔案，並以 config 資料夾中的資訊產生 deployment.json 檔案。 VS Code 中的整合式終端機會顯示建立進度。 

2. 在 Azure IoT 中樞的 [裝置總管] 中，以滑鼠右鍵按一下 IoT Edge 裝置識別碼，然後選取 [針對 Edge 裝置建立部署] 命令。 選取 config 資料夾中的 deployment.json 檔案。 VS Code 中的整合式終端機顯示已使用部署識別碼順利建立部署。

3. 如果您是在開發機器上模擬 IoT Edge 裝置，您會發現所有模組映像容器都會在數分鐘內啟動。

## <a name="view-the-generated-data"></a>檢視所產生的資料

1. 若要監視送達 IoT 中樞的資料，請選取 [檢視] > [命令選擇區]。 然後選取 [IoT：開始監視 D2C 訊息] 命令。 
2. 若要停止監視資料，請在 [命令選擇區] 中使用 [IoT：停止監視 D2C 訊息] 命令。 

## <a name="next-steps"></a>後續步驟

了解在 Visual Studio Code 中使用 Azure IoT Edge 進行開發的其他案例：

* 在 VS Code 中使用 [C#](how-to-develop-csharp-module.md) 或 [Node.js](how-to-develop-node-module.md) 來開發模組。
* 在 VS Code 中使用 [C#](how-to-develop-csharp-function.md) 來開發 Azure Functions。

若要為您的 IoT Edge 裝置開發模組，請[了解及使用 Azure IoT 中樞 SDK](../iot-hub/iot-hub-devguide-sdks.md)。