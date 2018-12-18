---
title: 使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能管理 Azure IoT 裝置 | Microsoft Docs
description: 使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來管理 Azure IoT 中樞裝置，並採用直接方法和對應項所需的屬性管理選項。
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 8/3/2018
ms.author: junhan
ms.openlocfilehash: ef99f4be97f5168add44d373a7e74de62347d110
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39450268"
---
# <a name="use-azure-iot-toolkit-extension-for-visual-studio-code-for-azure-iot-hub-device-management"></a>使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來管理 Azure IoT 中樞裝置

![端對端圖表](media/iot-hub-get-started-e2e-diagram/2.png)

[Azure IoT 工具組](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) \(英文\) 是實用的 Visual Studio Code 擴充功能，讓 IoT 中樞的管理變得更輕鬆。 它隨附的管理選項可供您用來執行各種工作。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

| 管理選項          | Task                                                                                                                            |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| 直接方法             | 使裝置執行動作，例如啟動或停止傳送訊息，或是將裝置重新開機。                                        |
| 讀取裝置對應項           | 取得裝置的報告狀態。 例如，裝置會回報 LED 現在正閃爍不停。                                    |
| 更新裝置對應項         | 讓裝置進入特定狀態，例如將 LED 設定為綠色，或將遙測傳送間隔設定為 30 分鐘。         |
| 雲端到裝置的訊息   | 將通知傳送至裝置。 例如，「今天很可能會下雨。 別忘了帶傘。」              |

如需差異的詳細說明和使用這些選項的相關指引，請參閱[裝置對雲端通訊指引](iot-hub-devguide-d2c-guidance.md)和[雲端對裝置通訊指引](iot-hub-devguide-c2d-guidance.md)。

「裝置對應項」是存放裝置狀態資訊 (中繼資料、組態和條件) 的 JSON 文件。 IoT 中樞會為其連線的每個裝置保存裝置對應項。 如需裝置對應項的詳細資訊，請參閱[開始使用裝置對應項](iot-hub-node-node-twin-getstarted.md)。

## <a name="what-you-learn"></a>您學到什麼

您會學到在開發電腦上使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能搭配各種管理選項。

## <a name="what-you-do"></a>您要做什麼

執行適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能，搭配各種管理選項。

## <a name="what-you-need"></a>您需要什麼

- 有效的 Azure 訂用帳戶。
- 位於您訂用帳戶中的 Azure IoT 中樞。
- [Visual Studio Code](https://code.visualstudio.com/)
- [Azure IoT 工具組](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit)

## <a name="sign-in-to-access-your-iot-hub"></a>登入以存取 IoT 中樞

1. 在 VS Code 的 [總管] 檢視中，展開左下角的 [Azure IoT Hub Devices] \(Azure IoT 中樞裝置\) 部分。
1. 按一下快顯功能表中的 [Select IoT Hub] \(選取 IoT 中樞\)。
1. 快顯視窗會顯示在右下角，讓您第一次登入 Azure。
1. 登入之後，系統會顯示您的 Azure 訂用帳戶清單，然後選取 [Azure 訂用帳戶] 和 [IoT 中樞]。
1. 在幾秒鐘的時間內，裝置清單會顯示在 [Azure IoT Hub Devices] \(Azure IoT 中樞裝置\) 索引標籤中。

   > [!Note]
   > 您也可以選擇 [設定 IoT 中樞連接字串] 來完成設定。 在快顯視窗中，輸入與您的 IoT 裝置連線的 IoT 中樞所使用的連接字串。

## <a name="direct-methods"></a>直接方法

1. 以滑鼠右鍵按一下您的裝置，然後選取 [叫用直接方法]。 
1. 在輸入方塊中輸入方法名稱和承載。
1. 結果會顯示在 [輸出] > [Azure IoT Toolkit] \(Azure IoT 工具組\) 檢視中。

## <a name="read-device-twin"></a>讀取裝置對應項

1. 以滑鼠右鍵按一下您的裝置，然後選取 [編輯裝置對應項]。 
1. **azure-iot-device-twin.json** 檔案與裝置對應項的內容一起開啟。

## <a name="update-device-twin"></a>更新裝置對應項

1. 適度編輯 **tags** 或 **properties.desired** 欄位。
1. 以滑鼠右鍵按一下 **azure-iot-device-twin.json** 檔案。
1. 選取 [更新裝置對應項] 來更新裝置對應項。

## <a name="send-cloud-to-device-messages"></a>傳送雲端到裝置訊息

若要從您的 IoT 中樞將訊息傳送到裝置，請遵循下列步驟：
 
1. 以滑鼠右鍵按一下您的裝置，然後選取 [Send C2D Message to Device] \(將 C2D 訊息傳送至裝置\)。 
1. 在輸入方塊中輸入訊息。
1. 結果會顯示在 [輸出] > [Azure IoT Toolkit] \(Azure IoT 工具組\) 檢視中。

## <a name="next-steps"></a>後續步驟

您已了解如何使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能搭配各種管理選項。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
