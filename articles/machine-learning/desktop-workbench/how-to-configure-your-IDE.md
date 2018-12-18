---
title: 如何設定 Azure Machine Learning Workbench 搭配 IDE？  | Microsoft Docs
description: 設定 Azure Machine Learning Workbench 搭配 IDE 的指南。
services: machine-learning
author: svankam
ms.author: svankam
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 02/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 8158d6faee5ec4d28f0c7e16963fc3d78392b857
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978948"
---
# <a name="how-to-configure-azure-machine-learning-workbench-to-work-with-an-ide"></a>如何設定 Azure Machine Learning Workbench 搭配 IDE 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

Azure Machine Learning Workbench 可設定為搭配熱門的 Python IDE (整合式開發環境)。 它能夠在資料準備、程式碼撰寫、執行追蹤及作業化之間，提供順暢的資料科學開發體驗。 目前支援的 IDE 如下：
- Microsoft Visual Studio Code 
- JetBrain PyCharm 

## <a name="configure-workbench"></a>設定 Workbench
1. 按一下應用程式左上角的 [檔案] 功能表。 
2. 從飛出視窗中選取 [設定專案 IDE] 選項 
3. 在 [名稱] 欄位 (名稱為任意指定) 中輸入 `VS Code` 或 `PyCharm`
4. 在 [執行路徑] 中輸入 IDE 可執行檔的位置 (含執行檔名稱與附檔名)

### <a name="default-install-path-for-visual-studio-code"></a>Visual Studio Code 的預設安裝路徑  

* Windows 32 位元：`C:\Program Files (x86)\Microsoft VS Code\Code.exe`
* Windows 64 位元：`C:\Program Files\Microsoft VS Code\Code.exe`
* macOS：選取 .app 路徑 (例如 `/Applications/Visual Studio Code.app`)，而應用程式會為您附加路徑的其餘部分。 根據預設，可執行檔的完整路徑是 `/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code`。 如果您已在 VS Code 中執行 `Shell Command: Install 'code' command in PATH` 命令，則也可以參考 `/usr/local/bin/code` 中的 VS Code 指令碼

### <a name="default-install-path-for-pycharm"></a>PyCharm 的預設安裝路徑 

* Windows 32 位元：`C:\Program Files (x86)\JetBrains\PyCharm Community Edition 2017.2.1\bin\pycharm.exe`。 
* Windows 64 位元：`C:\Program Files\JetBrains\PyCharm Community Edition 2017.2.1\bin\pycharm64.exe`。
* macOS：選取 .app 路徑 (例如 “/Applications/PyCharm CE.app”)，而應用程式會為您附加路徑的其餘部分。 根據預設，可執行檔的完整路徑是 `/Applications/PyCharm CE.app/Contents/MacOS/pycharm`。 您也能在 bin 資料夾 `/usr/local/bin/charm` 中找到 PyCharm

## <a name="open-project-in-ide"></a>在 IDE 中開啟專案 
設定完成之後，您可以在 Azure Machine Learning Workbench 中開啟 [檔案] 功能表，然後按一下 [開啟專案 (<IDE_Name>)]，來開啟 Azure Machine Learning 專案。 此動作會在所設定的 IDE 中開啟目前作用中的專案。 _注意：如果您不在專案中，[開啟專案 (<IDE_Name>)]將會是停用狀態。_

## <a name="configuring-the-integrated-terminal-in-visual-studio-code"></a>在 Visual Studio Code 中設定整合的終端機

### <a name="windows"></a>Windows 
我們已將預設殼層覆寫為 cmd，而不是 PowerShell。 按一下 [開啟專案 (<IDE_Name>)]，就會看到提示： 

您允許在終端機中啟動殼層：`C:\windows\System32\cmd.exe` (已定義為工作區設定) 嗎？

回答 `yes`，便能將殼層設為可與 Azure ML Workbench 命令列介面無縫運作。

### <a name="mac"></a>Mac
若要在 Mac 上使用 Visual Studio Code 的整合終端機來執行 `az` 命令，您必須手動將 `PATH` 的值，設定為與專案之 `.vscode/settings.json` 檔案中 `terminal.integrated.env.osx` 機碼底下的 `PATH` 值相同。 若要這麼做，您可以在終端機中執行下列命令：`PATH=<PATH in .vscode/settings>`
