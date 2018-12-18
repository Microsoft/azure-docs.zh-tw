---
title: 設定自訂裝置型號 - Azure | Microsoft Docs
description: 本文說明如何在「裝置模擬」解決方案加速器中設定自訂裝置型號。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 07/06/2018
ms.topic: conceptual
ms.openlocfilehash: 61a7c5314df541b4b44a286efe982f4bf93256e3
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37967469"
---
# <a name="device-models"></a>裝置型號

設定模擬時，您可以選擇將現有裝置型號和預先定義的一組感應器搭配使用，或使用您選擇的模擬感應器建立自訂裝置型號。 自訂感應器可讓您更接近模擬真實裝置。

## <a name="pre-configured-device-models"></a>預先設定的裝置型號

裝置模擬解決方案加速器提供三個預先設定的裝置型號：Chillers、Elevators 和 Trucks。

預先設定的裝置型號有多個感應器與 JavaScript 檔案中定義的進階行為。 您無法在 Web UI 中設定這些行為。

下表針對每個預先設定的裝置型號，顯示設定資料表：

| 裝置型號  | 感應器           | 單位  |
| ------------- | ---------------- | ----- |
| Chiller       | 溼度         | %     |
|               | pressure         | psig  |
|               | 溫度      | F     |
| 電梯      | 樓層            |       |
|               | 震動        | mm    |
|               | 溫度      | F     |
| 卡車         | 緯度         |       |
|               | 經度        |       |
|               | 速度            | 英哩/小時   |
|               | cargotemperature | F     |

## <a name="custom-device-models"></a>自訂裝置型號

自訂裝置型號可讓您製作更接近您自己裝置的感應器型號。 自訂裝置最多可以擁有 10 個自訂感應器。

當您選取自訂裝置型號類型時，可以按一下 [+ 新增感應器] 來加入新的感應器。

[![設定感應器](./media/iot-accelerators-device-simulation-custom-model/configuresensors-inline.png)](./media/iot-accelerators-device-simulation-custom-model/configuresensors-expanded.png#lightbox)

自訂感應器具有下列屬性：

| 欄位     | 說明 |
| --------- | ----------- |
| 感應器名稱 | 易記的感應器名稱，例如**溫度**或**速度**。  |
| 行為  | 行為可讓遙測資料根據不同的訊息而變化，以模擬真實世界的資料。 **遞增**可在傳送的每個訊息中將其值加 1 (從最小值開始)。 一旦符合最大值，就會再從最小值開始。 **遞減**的運作方式與**遞增**相同，但是為倒數。 **隨機**行為會產生一個介於最小值到最大值之間的隨機值。 |
| 最小值 | 可接受範圍的最小數字。 |
| 最大值 | 可接受範圍的最大數字。 |
| 單位      | 感應器的度量單位，例如 °F 或英哩/小時。 |

## <a name="next-steps"></a>後續步驟

在本使用說明指南中，您已了解如何設定模擬的自訂裝置型號。 接下來，您可能想要探索一些其他 [IoT 解決方案加速器](about-iot-accelerators.md)。