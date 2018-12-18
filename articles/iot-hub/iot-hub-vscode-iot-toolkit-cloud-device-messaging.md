---
title: 使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能，管理 Azure IoT 中樞雲端裝置傳訊 | Microsoft Docs
description: 了解如何使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來監視 Azure IoT 中樞的裝置到雲端訊息，以及傳送雲端到裝置訊息。
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 7/20/2018
ms.author: junhan
ms.openlocfilehash: 7bcb6eebdb6ceba6b07aeb19c1a74309fd4227d0
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39206609"
---
# <a name="use-azure-iot-toolkit-extension-for-visual-studio-code-to-send-and-receive-messages-between-your-device-and-iot-hub"></a>使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來傳送和接收裝置與 IoT 中樞之間的訊息

![端對端圖表](media/iot-hub-get-started-e2e-diagram/2.png)

[Azure IoT 工具組](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) \(英文\) 是實用的 Visual Studio Code 擴充功能，讓 IoT 中樞的管理變得更輕鬆。 本文章重點介紹如何使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來傳送和接收裝置與 IoT 中樞之間的訊息。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="what-you-will-learn"></a>學習目標

您將了解如何使用適用於 Visual Studio Code 的 Azure IoT Toolkit 擴充功能來監視 Azure IoT 中樞的裝置到雲端訊息，以及傳送雲端到裝置訊息。 裝置到雲端的訊息可能是您的裝置所收集，然後傳送到 IoT 中樞的感應器資料。 雲端到裝置的訊息可能是 IoT 中樞傳送到裝置以使連接到裝置的 LED 閃爍的命令。

## <a name="what-you-will-do"></a>將執行的作業

- 使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來監視裝置到雲端訊息。
- 使用適用於 Visual Studio Code 的 Azure IoT 工具組擴充功能來傳送雲端到裝置訊息。

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
   
## <a name="monitor-device-to-cloud-messages"></a>監視裝置到雲端的訊息

若要監視從您的裝置傳送到IoT 中樞的訊息，請遵循下列步驟：

1. 在裝置上按滑鼠右鍵，然後選取 [Start Monitoring D2C Message] \(開始監視 D2C 訊息\)。
1. 受監視的訊息會顯示在 [輸出] > [Azure IoT Toolkit] \(Azure IoT 工具組\) 檢視中。
1. 若要停止監視，請在 [輸出] 檢視上按滑鼠右鍵，然後選取 [Stop Monitoring D2C Message] \(停止監視 D2C 訊息\)。

## <a name="send-cloud-to-device-messages"></a>傳送雲端到裝置訊息

若要從您的 IoT 中樞將訊息傳送到裝置，請遵循下列步驟：

1. 以滑鼠右鍵按一下您的裝置，然後選取 [Send C2D Message to Device] \(將 C2D 訊息傳送至裝置\)。 
1. 在輸入方塊中輸入訊息。
1. 結果會顯示在 [輸出] > [Azure IoT Toolkit] \(Azure IoT 工具組\) 檢視中。

## <a name="next-steps"></a>後續步驟

您已了解如何監視裝置到雲端的訊息，以及在 IoT 裝置和 Azure IoT 中樞之間傳送雲端到裝置的訊息。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
