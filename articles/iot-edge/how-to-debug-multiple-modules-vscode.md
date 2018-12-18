---
title: 在 VS Code 中偵測 Azure IoT Edge 的多個模組 | Microsoft Docs
description: 使用 Visual Studio Code 偵錯 Azure IoT Edge 的多個模組
author: shizn
manager: ''
ms.author: xshi
ms.date: 06/27/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 38c66129ac8244d18b0f94c1485c785015e29ab4
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37049586"
---
# <a name="use-visual-studio-code-to-debug-multiple-modules-with-azure-iot-edge"></a>使用 Visual Studio Code 偵錯 Azure IoT Edge 的多個模組
本文提供使用 [Visual Studio (VS) Code](https://code.visualstudio.com/) 偵錯 IoT Edge 上多個模組的詳細指示。

## <a name="prerequisites"></a>先決條件
完成[在 Visual Studio Code 中開發具有多個模組的 IoT Edge 解決方案](tutorial-multiple-modules-in-vscode.md)教學課程，並確定 IoT Edge 裝置上至少有兩個模組正在執行中。

## <a name="multi-target-and-remote-debugging-in-vs-code"></a>VS Code 中的多目標與遠端偵錯
透過 VS Code 與 Azure IoT Edge 擴充功能，您可以連結容器中的模組程序，無論容器正在開發電腦上或遠端實體 IoT Edge 裝置中執行。 偵錯正在容器中執行的多個模組，實際上會連結不同容器中的多個處理序。 偵錯多個模組時，可使用 VS Code [多目標偵錯](https://code.visualstudio.com/docs/editor/debugging#_multitarget-debugging)。

   > [!TIP]
   > 如果您的模組容器正在遠端實體 IoT Edge 裝置中執行，您可能需要設定 [Docker 機器](https://docs.docker.com/machine/overview/) ，開發電腦上的 Docker 引擎才能與遠端 Docker 主機通訊。

### <a name="build-your-iot-edge-modules-for-debugging-purpose"></a>建置用於偵錯的 IoT Edge 模組
1. 若要開始多模組偵錯，您必須使用 **Dockerfile.amd64.debug** 重建 docker 映像，並再次部署 Edge 解決方案。 在 VS Code 總管中，瀏覽至 `deployment.template.json` 檔案。 在結尾加上 `.debug` 以更新映像 URL。 您必須至少有兩個具有 `.debug` 的模組映像。 如果您是使用先前教學課程中的解決方案，您應該會有 C# 函式模組與 C# 模組。 在尾端加入 `.debug` 並儲存此檔案，以更新至少兩個映像 URL。 
2. 重建您的解決方案。 在 VS Code 命令選擇區中，輸入命令 **Azure IoT Edge: Build IoT Edge solution** 並執行。
3. 在 Azure IoT 中樞裝置總管中，以滑鼠右鍵按一下 IoT Edge 裝置識別碼，然後選取 [Create deployment for Edge device] \(針對 Edge 裝置建立部署\)。 選取 `config` 資料夾下的 `deployment.json` 檔案。 接著，您可以看到已順利建立部署，而且 VS Code 整合式終端機中會有部署識別碼。

您可以在 VS Code Docker 總管中檢查您的容器狀態，或透過在終端機中執行 `docker ps` 命令來檢查。

### <a name="start-debugging-c-function-in-vs-code"></a>在 VS Code 中開始針對 C# 函式進行偵錯
1. VS Code 會將偵錯設定資訊保留在您工作區中之 `.vscode` 資料夾的 `launch.json` 檔案中。 這個 `launch.json` 檔案在您建立新 IoT Edge 解決方案時產生。 每當您新增支援偵錯的模組時，此檔案都會更新。 瀏覽到偵錯檢視，並為 C# 函式模組遠端偵錯選取對應的偵錯設定檔。
2. 瀏覽至 `run.csx`。 在函式中新增中斷點。
3. 按一下 [開始偵錯] 按鈕或按 **F5**，然後選取要附加至的處理序。
4. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。 

### <a name="start-debugging-c-module-at-the-same-time-in-vs-code"></a>同時開始在 VS Code 中針對 C# 模組進行偵錯
1. 在 VS Code 命令選擇區中，輸入「Workspace: Duplicate Workspace in New Window」命令並執行。 這會開啟一個新的 VS Code 視窗，當中顯示相同的工作區。
2. 瀏覽到偵錯檢視，並為 C# 模組遠端偵錯選取對應的偵錯組態檔案。
3. 瀏覽至 `program.cs`。 在 C# 模組中新增中斷點。
4. 按一下 [開始偵錯] 按鈕或按 **F5**，然後選取要附加至的處理序。
5. 在 VS Code 偵錯檢視中，您可以在左面板中看到變數。 

### <a name="see-variables-in-multiple-debugging-windows"></a>在多個偵錯視窗中檢視變數
1. 現在您有至少兩個偵錯工作階段在兩個 VS Code 視窗中執行。 其中一個中斷點應已遭點擊。
2. 按 `F10`，或按一下**偵錯工具列** 中的 [不進入函式] 按鈕。
3. 另一個 VS Code 視窗中的中斷點應已遭點擊。 
4. 繼續上述兩個步驟，您會在多個 VS Code 偵錯視窗中看見多個模組的變數。

> [!NOTE]
> 以上範例顯示如何偵錯 Azure IoT Edge 的多個模組。 它是以 `Dockerfile.amd64.debug` 的偵錯版本為基礎，其中包含建置時容器映像中的 VSDBG (.NET Core 命令列偵錯工具)。 完成 C# 函式偵錯之後，我們建議您直接使用，或針對生產就緒 IoT Edge 函式自訂無 VSDBG 的 `Dockerfile`。

## <a name="next-steps"></a>後續步驟

建置模組之後，了解如何[從 Visual Studio Code 部署 Azure IoT Edge 模組](how-to-deploy-modules-vscode.md)
