---
title: 裝置模擬與 IoT 遠端監視 - Azure | Microsoft Docs
description: 本操作指南示範如何搭配遠端監視解決方案加速器使用裝置模擬器。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 09/28/2018
ms.topic: conceptual
ms.openlocfilehash: d741677252ba6787701b9ee9da84ebd38528d70e
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432149"
---
# <a name="create-and-test-a-new-simulated-device"></a>建立及測試新模擬裝置

遠端監視解決方案加速器可讓您定義自己的模擬裝置。 本文會示範如何定義新的模擬燈泡裝置，然後在本機進行測試。 解決方案加速器中包含冷卻器和卡車等模擬裝置。 不過，您可以定義自己的模擬裝置，以在部署實際裝置之前，先測試您的 IoT 解決方案。

本操作指南會示範如何自訂裝置模擬微服務。 此微服務會是遠端監視解決方案加速器的一部份。 為說明裝置模擬的功能，本操作指南會使用 Contoso IoT 應用程式中的兩個情節：

[!INCLUDE [iot-solution-accelerators-create-device](../../includes/iot-solution-accelerators-create-device.md)]

## <a name="next-steps"></a>後續步驟

本指南已示範如何建立自訂的模擬裝置類型，並藉由在本機執行裝置模擬微服務來進行測試。

接下來，建議您了解如何將自訂模擬裝置類型部署至[遠端監視解決方案加速器](iot-accelerators-remote-monitoring-deploy-simulated-device.md)。
