---
title: Azure 地圖服務中的耗用模型 | Microsoft Docs
description: 深入了解 Azure 地圖服務中的耗用模型
author: subbarayudukamma
ms.author: skamma
ms.date: 05/08/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: cf20c7dbfbf7cd3f09579b03b835148c1c295137
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34600624"
---
# <a name="consumption-model"></a>耗用模型

連線路由會提供一組參數，以詳細說明載具特定的耗用模型。
視 **vehicleEngineType** 的值而定，支援兩種主要的耗用模型：_燃燒_和_電子_。 在同一要求中指定屬於不同模型的參數，會發生錯誤。
耗用模型不能搭配 **travelMode** 值 _bicycle_ 和 _pedestrian_。

## <a name="parameter-constraints-for-consumption-model"></a>耗用模型的參數限制式

在這兩個耗用模型中，要明確指定一些參數必須同時指定其他參數。 這些相依性如下：

* 所有參數都需要由使用者指定 **constantSpeedConsumption**。 如果未指定 **constantSpeedConsumption**\*，指定任何其他耗用模型參數會發生錯誤，除了 **vehicleWeight**。
* **accelerationEfficiency** 和 **decelerationEfficiency** 一律必須指定為一對 (也就是兩個並存或無)。
* 如果指定 **accelerationEfficiency** 和 **decelerationEfficiency**，其值的產品必不能大於 1 (以防永恆運動)。
* **uphillEfficiency** 和 **downhillEfficiency** 一律必須指定為一對 (也就是兩個並存或無)。
* 如果指定 **uphillEfficiency** 和 **downhillEfficiency**，其值的產品必不能大於 1 (以防永恆運動)。
* 如果使用者指定 \***Efficiency** 參數，則也必須指定 **vehicleWeight**。 若 **vehicleEngineType** 為 _combustion_，也必須指定 **fuelEnergyDensityInMJoulesPerLiter**。
* **maxChargeInkWh** 和 **currentChargeInkWh** 一律必須指定為一對 (也就是兩個並存或無)。

> [!NOTE]
> 如果僅指定 **constantSpeedConsumption**，沒有為耗用計算考慮其他耗用層面，如坡度和載具加速。

## <a name="combustion-consumption-model"></a>燃燒耗用模型

**vehicleEngineType** 設為_燃燒_時，會使用燃燒耗用模型。
屬於此模型的參數清單如下。 請參閱 [參數] 區段，取得詳細說明。

* constantSpeedConsumptionInLitersPerHundredkm
* vehicleWeight
* currentFuelInLiters
* auxiliaryPowerInLitersPerHour
* fuelEnergyDensityInMJoulesPerLiter
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="electric-consumption-model"></a>電子耗用模型

**vehicleEngineType** 設為_電子_時，會使用電子耗用模型。
屬於此模型的參數清單如下。 請參閱 [參數] 區段，取得詳細說明。

* constantSpeedConsumptionInkWhPerHundredkm
* vehicleWeight
* currentChargeInkWh
* maxChargeInkWh
* auxiliaryPowerInkW
* accelerationEfficiency
* decelerationEfficiency
* uphillEfficiency
* downhillEfficiency

## <a name="sensible-values-of-consumption-parameters"></a>合理的耗用參數值

一組特定的耗用參數可能遭拒絕，即使它可能達到了上述指定的所有明確需求。 若特定參數的值或數個參數的值組合，被視為會造成不合理的耗用值範圍，即會發生這種情況。 如果發生這種情況，最有可能表示輸入錯誤，因為原先已適當處理以容納耗用參數的所有合理值。 如果已拒絕一組特定的耗用參數，則伴隨而來的錯誤訊息會包含原因的文字說明。
參數的詳細說明含有兩種模型合理值的範例。
