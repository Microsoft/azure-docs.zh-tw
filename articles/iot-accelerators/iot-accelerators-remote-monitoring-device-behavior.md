---
title: 遠端監視解決方案中的模擬裝置行為 - Azure | Microsoft Docs
description: 本文說明如何使用 JavaScript 定義模擬裝置在遠端監視解決方案中的行為。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/29/2018
ms.topic: conceptual
ms.openlocfilehash: a983c7307308534140ab8999593ac4c8c6992a42
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338501"
---
# <a name="implement-the-device-model-behavior"></a>實作裝置模型行為

[了解裝置模型結構描述](iot-accelerators-remote-monitoring-device-schema.md)一文說明了定義模擬裝置模型的結構描述。 該文參考兩種類型的 JavaScript 檔案，這兩種類型都可以實作模擬裝置的行為：

- **State**：JavaScript 檔案，可在固定的間隔執行，以更新裝置的內部狀態。
- **Method**：JavaScript 檔案，可當方案在裝置上叫用方法時執行。

在本文中，您將了解：

>[!div class="checklist"]
> * 控制模擬裝置的狀態
> * 定義模擬裝置從遠端監視解決方案回應方法呼叫的方式
> * 為指令碼偵錯

[!INCLUDE [iot-accelerators-device-schema](../../includes/iot-accelerators-device-schema.md)]

## <a name="next-steps"></a>後續步驟

本文說明如何定義您自己的自訂模擬裝置模型行為。 本文示範如何：

<!-- Repeat task list from intro -->
>[!div class="checklist"]
> * 控制模擬裝置的狀態
> * 定義模擬裝置從遠端監視解決方案回應方法呼叫的方式
> * 為指令碼偵錯

現在您已經學會如何指定模擬裝置的行為，建議的下一個步驟是了解如何[建立模擬的裝置](iot-accelerators-remote-monitoring-create-simulated-device.md)。

如需關於遠端監視解決方案的開發人員詳細資訊，請參閱：

* [開發人員參考指南](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide)
* [開發人員疑難排解指南](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Troubleshooting-Guide)

<!-- Next tutorials in the sequence -->
