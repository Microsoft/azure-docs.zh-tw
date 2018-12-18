---
title: 裝置模擬解決方案中的裝置結構描述 - Azure | Microsoft Docs
description: 本文說明 JSON 結構描述，它會定義裝置模擬解決方案中的模擬裝置。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/29/2018
ms.topic: conceptual
ms.openlocfilehash: b325873819caff139727ec15d6aecd2d4be89c9e
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338150"
---
# <a name="understand-the-device-model-schema"></a>了解裝置機型結構描述

您可以使用裝置模擬解決方案加速器，為使用 IoT 中樞的解決方案產生測試負載。 當您部署裝置模擬解決方案時，會自動佈建模擬裝置集合。 您可以自訂現有的模擬裝置，或建立您自己的裝置。

本文說明裝置機型結構描述，它會指定模擬裝置的功能和行為。 裝置機型會儲存在 JSON 檔案中。

下列文章與目前的文章相關：

* [實作裝置機型行為](iot-accelerators-device-simulation-device-behavior.md)描述您用來實作模擬裝置行為的 JavaScript 檔案。
* [建立新的模擬裝置](iot-accelerators-device-simulation-create-simulated-device.md)會統整並示範如何將新的模擬裝置類型部署到您的解決方案。

在本文中，您將了解：

>[!div class="checklist"]
> * 使用 JSON 檔案以定義模擬裝置機型
> * 指定模擬裝置的屬性
> * 指定模擬裝置傳送的遙測
> * 指定裝置回應的雲端到裝置方法

[!INCLUDE [iot-accelerators-device-schema](../../includes/iot-accelerators-device-schema.md)]

## <a name="next-steps"></a>後續步驟

本文說明如何建立您自己的自訂模擬裝置機型。 本文示範如何：

<!-- Repeat task list from intro -->
>[!div class="checklist"]
> * 使用 JSON 檔案以定義模擬裝置機型
> * 指定模擬裝置的屬性
> * 指定模擬裝置傳送的遙測
> * 指定裝置回應的雲端到裝置方法

現在您已經了解 JSON 結構描述，建議的下一個步驟是了解如何[實作模擬裝置的行為](iot-accelerators-device-simulation-device-behavior.md)。

如需更多有關裝置模擬解決方案的開發人員資訊，請參閱[開發人員參考指南](https://github.com/Azure/device-simulation-dotnet/wiki/Simulation-Service-Developer-Reference-Guide)。
