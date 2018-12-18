---
title: 使用 Azure Functions 和認知服務建立 IoT DevKit 翻譯工具 | Microsoft Docs
description: 使用 IoT DevKit 上的麥克風來接收語音訊息，然後使用 Azure 認知服務將其處理為以英文表示的翻譯文字
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 02/28/2018
ms.author: liydu
ms.openlocfilehash: cd67e612dd020ba600e33ac8baf77bc094d8afd3
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42141519"
---
# <a name="use-iot-devkit-az3166-with-azure-functions-and-cognitive-services-to-make-a-language-translator"></a>使用 IoT DevKit AZ3166 搭配 Azure Functions 和認知服務製作語言翻譯工具

在本文中，您可了解如何藉由使用 [Azure 認知服務](https://azure.microsoft.com/services/cognitive-services/)，讓 IoT DevKit 成為語言翻譯工具。 它會記錄您的語音，並將它轉譯成在 DevKit 畫面上顯示的英文文字。

[MXChip IoT DevKit](https://aka.ms/iot-devkit) 是全方位 Arduino 相容板，具有豐富的週邊設備和感應器。 您可以使用[適用於 Arduino 的 Visual Studio Code 延伸模組](https://aka.ms/arduino)來對它進行開發。 它隨附不斷增長的[專案目錄](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/)，以引導您設定運用 Microsoft Azure 服務的物聯網 (IoT) 解決方案原型。

## <a name="what-you-need"></a>您需要什麼

完成[使用者入門指南](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started)以便：

* 讓 DevKit 連線至 Wi-Fi
* 準備開發環境

有效的 Azure 訂用帳戶。 如果沒有，您可以透過下列兩種方法之一來註冊：

* 啟動 [30 天免費試用 Microsoft Azure 帳戶](https://azure.microsoft.com/free/)
* 如果您是 MSDN 或 Visual Studio 訂閱者，請認領您的 [Azure 點數](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)

## <a name="open-the-project-folder"></a>開啟專案資料夾

首先，開啟專案資料夾。 

### <a name="start-vs-code"></a>啟動 VS Code

- 確定 DevKit 已連接到電腦。

- 啟動 VS Code。

- 將 DevKit 連接到您的電腦。

### <a name="open-the-arduino-examples-folder"></a>開啟 Arduino 範例資料夾

展開左邊的 [ARDUINO 範例 > MXCHIP AZ3166 的範例 > AzureIoT]，然後選取 [DevKitTranslator]。 新的 VS Code 視窗隨即開啟，其中顯示專案資料夾。 如果您看不到 Examples for MXCHIP AZ3166 區段，請確定裝置已正確連接並重新啟動 VS Code。  

![IoT DevKit 範例](media/iot-hub-arduino-iot-devkit-az3166-translator/vscode_examples.png)

您也可以從命令選擇區開啟範例。 請使用 `Ctrl+Shift+P` (macOS：`Cmd+Shift+P`) 來開啟命令選擇區，輸入 **Arduino**，然後尋找並選取 [Arduino: Examples] \(Arduino: 範例\)。

## <a name="provision-azure-services"></a>佈建 Azure 服務

在解決方案視窗中，輸入 `Ctrl+P` (macOS: `Cmd+P`) 然後輸入 `task cloud-provision`。

在 VS Code 終端機中，互動式命令列將引導您佈建所有必要的 Azure 服務：

![雲端佈建工作](media/iot-hub-arduino-iot-devkit-az3166-translator/cloud-provision.png)

## <a name="deploy-the-azure-function"></a>部署 Azure Function

使用 `Ctrl+P` (macOS: `Cmd+P`) 來執行 `task cloud-deploy`，以部署 Azure Functions 程式碼。 此程序通常需要 2 到 5 分鐘才能完成。

![雲端部署工作](media/iot-hub-arduino-iot-devkit-az3166-translator/cloud-deploy.png)

Azure Function 部署成功之後，請在 azure_config.h 檔案中填入函式應用程式名稱。 您可以瀏覽至 [Azure 入口網站](https://portal.azure.com/)以找到該檔案：

![尋找 Azure Function 應用程式名稱](media/iot-hub-arduino-iot-devkit-az3166-translator/azure-function.png)

> [!NOTE]
> 如果 Azure Function 未正常運作，請查看 [IoT DevKit 常見問題集中的「Azure Function 的複雜功能錯誤」頁面](https://microsoft.github.io/azure-iot-developer-kit/docs/faq#compilation-error-for-azure-function)，以解決問題。

## <a name="build-and-upload-the-device-code"></a>組建並上傳裝置程式碼

1. 使用 `Ctrl+P` (macOS: `Cmd+P`) 來執行 `task config-device-connection`。

2. 終端機會詢問您是否要使用擷取自 `task cloud-provision` 步驟的連接字串。 您也可以選取 [建立新的...] 來輸入自己的裝置連接字串

3. 終端機會提示您進入設定模式。 若要這樣做，請按住按鈕 A，然後按下並放開 [重設] 按鈕。 畫面會顯示 DevKit 識別碼和 [設定]。

   ![驗證和上傳 Arduino 草圖](media/iot-hub-arduino-iot-devkit-az3166-translator/config-device-connection.png)

4. `task config-device-connection` 完成之後，按一下 `F1` 以載入 VS Code 命令，並選取 `Arduino: Upload`，然後 VS Code 會開始驗證和上傳 Arduino 草稿碼。

   ![驗證和上傳 Arduino 草圖](media/iot-hub-arduino-iot-devkit-az3166-translator/arduino-upload.png)

DevKit 會重新開機，然後開始執行程式碼。

## <a name="test-the-project"></a>測試專案

在應用程式初始化之後，依照 DevKit 畫面上的指示操作。 預設來源語言是中文。

若要選取另一種語言進行翻譯：

1. 按下按鈕 A 進入設定模式。

2. 按下按鈕 B 捲動所有支援的來源語言。

3. 按下按鈕 A 確認您選擇的來源語言。

4. 在說話時按住按鈕 B，然後放開按鈕 B 開始翻譯。

5. 以英文表示的翻譯文字會顯示在畫面上。

![捲動以選取語言](media/iot-hub-arduino-iot-devkit-az3166-translator/select-language.jpg)

![翻譯結果](media/iot-hub-arduino-iot-devkit-az3166-translator/translation-result.jpg)

在翻譯結果畫面上，您可以：

- 按下按鈕 A 和 B 以捲動並選取來源語言。

- 按 B 按鈕進行交談。 若要傳送語音和取得翻譯文字，請放開 B 按鈕。

## <a name="how-it-works"></a>運作方式

![mini-solution-voice-to-tweet-diagram](media/iot-hub-arduino-iot-devkit-az3166-translator/diagram.png)

Arduino 草稿碼會記錄您的語音，然後發佈 HTTP 要求以觸發 Azure Function。 Azure Function 會呼叫認知服務語音翻譯工具 API 來進行翻譯。 在 Azure Function 取得翻譯文字之後，它會將 C2D (雲端到裝置) 訊息傳送到裝置。 然後翻譯會顯示在畫面上。

## <a name="problems-and-feedback"></a>問題與意見反應

如果發生問題，請參閱 [IoT DevKit 常見問題集](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/)，或使用下列管道與我們連絡：

* [Gitter.im](http://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stackoverflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>後續步驟

您已了解如何使用 Azure Functions 和認知服務讓 IoT DevKit 成為翻譯工具。 在此操作說明中，您已了解如何：

> [!div class="checklist"]
> * 使用 Visual Studio Code 工作自動進行雲端佈建
> * 設定 Azure IoT 裝置連接字串
> * 部署 Azure Function
> * 測試語音訊息翻譯

請前進到其他教學課程來了解如何：

> [!div class="nextstepaction"]
> [將 IoT DevKit AZ3166 連線到 Azure IoT 遠端監視解決方案加速器](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring)
