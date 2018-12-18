---
title: 使用現有的 IoT 中樞搭配裝置模擬解決方案 - Azure | Microsoft Docs
description: 本文說明如何設定「裝置模擬」解決方案加速器以使用現有的 IoT 中樞。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 07/06/2018
ms.topic: conceptual
ms.openlocfilehash: ee96173ca5f36dee0f08c38e4b6e29da6fee804e
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37967487"
---
# <a name="use-an-existing-iot-hub-with-the-device-simulation-solution-accelerator"></a>使用現有的 IoT 中樞搭配裝置模擬解決方案加速器

當您佈建「裝置模擬」解決方案加速器時，可以選擇部署解決方案加速器之資源群組中的 IoT 中樞，以用於模擬中。

如果未選擇部署選用的 IoT 中樞，則必須使用自己的中樞進行任何模擬。 如果您選擇部署選用的 IoT 中樞，您可以選擇使用此選用的中樞或您自己的中樞。

如果您沒有 IoT 中樞，一律可以從 [Azure 入口網站](https://portal.azure.com)建立一個新的 IoT 中樞。

若要使用預先存在的 IoT 中樞，您需要 **iothubowner** 共用存取原則的連接字串。 您可以從 [Azure 入口網站](https://portal.azure.com)取得此連接字串：

1. 在入口網站中的中樞設定頁面上，按一下 [共用存取原則]。
1. 按一下 [iothubowner]。
1. 複製主要連接字串或次要連接字串。

[![取得連接字串](./media/iot-accelerators-device-simulation-choose-hub/connectionstring-inline.png)](./media/iot-accelerators-device-simulation-choose-hub/connectionstring-expanded.png#lightbox)

使用您設定模擬時所複製的連接字串：

[![設定模擬](./media/iot-accelerators-device-simulation-choose-hub/configuresimulation-inline.png)](./media/iot-accelerators-device-simulation-choose-hub/configuresimulation-expanded.png#lightbox)

## <a name="next-steps"></a>後續步驟

在本操作指南中，了解如何在模擬中使用現有的 IoT 中樞。 接著，您可能想要了解如何為模擬[設定自訂裝置型號](iot-accelerators-device-simulation-custom-model.md)。
