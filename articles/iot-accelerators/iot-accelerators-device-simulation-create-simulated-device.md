---
title: 裝置模擬與 IoT 解決方案加速器 - Azure | Microsoft Docs
description: 本操作指南示範如何搭配使用裝置模擬器和裝置模擬解決方案加速器。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 08/16/2018
ms.topic: conceptual
ms.openlocfilehash: 6fe6421cea584a84c76e8c70c2ae90475b613036
ms.sourcegitcommit: e45b2aa85063d33853560ec4bc867f230c1c18ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43371528"
---
# <a name="create-and-test-a-new-simulated-device"></a>建立及測試新模擬裝置

裝置模擬解決方案加速器可讓您定義自己的模擬裝置。 本文會示範如何定義新的模擬燈泡裝置，然後在本機進行測試。 解決方案加速器中包含冷卻器和卡車等模擬裝置。 不過，您可以定義自己的模擬裝置，以在部署實際裝置之前，先測試您的 IoT 解決方案。

本操作指南會示範如何自訂裝置模擬微服務。 此微服務會是裝置模擬解決方案加速器的一部份。 為說明裝置模擬的功能，本操作指南會使用 Contoso IoT 應用程式中的兩個情節：

[!INCLUDE [iot-solution-accelerators-create-device](../../includes/iot-solution-accelerators-create-device.md)]

## <a name="next-steps"></a>後續步驟

本指南已示範如何建立自訂的模擬裝置類型，並藉由在本機執行裝置模擬微服務來進行測試。

接下來，建議您了解如何將自訂模擬裝置類型部署至[裝置模擬解決方案加速器](iot-accelerators-device-simulation-deploy-simulated-device.md)。
