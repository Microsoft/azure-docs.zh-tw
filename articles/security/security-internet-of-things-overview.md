---
title: 在 Azure 中保護您的物聯網 (IoT) | Microsoft Docs
description: " Azure 物聯網 (IoT) 服務提供廣泛的功能。 本文章協助您了解如何在 Azure 中保護您的 IoT 解決方案。 "
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TomSh
ms.assetid: 1473c8dd-8669-48fb-86db-b3c50e2eaf59
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: terrylan
ms.openlocfilehash: 41e422d0808cafb45b182c5f0a6bb7176a35516f
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35758615"
---
# <a name="internet-of-things-security-overview"></a>物聯網安全性概觀
Azure 物聯網 (IoT) 服務提供廣泛的功能。 這些企業等級的服務可讓您：

* 從裝置收集資料
* 分析移動中的資料串流
* 儲存和查詢大型資料集
* 視覺化即時和歷程記錄資料
* 與後端辦公室系統整合

為了提供這些功能，Azure IoT 解決方案加速器將多個 Azure 服務與自訂擴充功能封裝在一起作為預先設定的解決方案。 這些預先設定的解決方案是常見 IoT 解決方案模式的基礎實作，可幫助您減少實行 IoT 解決方案所花費的時間。 透過使用 IoT 軟體開發套件，您將可自訂和擴充這些解決方案來滿足您自己的需求。 您也可以使用這些解決方案作為開發新 IoT 解決方案時的範例或範本。

Azure IoT 解決方案加速器是一組符合 IoT 需求的強大解決方案。 不過，最重要的是您的 IoT 解決方案一開始就是以安全性為考量而設計的。 因為有大量的 IoT 裝置，所以任何安全性事件可能很快就會造成有嚴重後果的大規模事件。

為了協助您了解如何保護您的 IoT 解決方案，我們提供下列資訊。

## <a name="security-architecture"></a>安全性架構
在設計系統時，我們應了解該系統的潛在威脅並加入適當的防禦機制，以妥善設計系統與其架構。 在一開始設計產品時就先考量安全性是很重要的，因為了解攻擊者可能如何破壞系統，有助於從一開始就備妥適當的安全防護功能。

您可以了解 IoT 安全性架構，方法是閱讀 [物聯網安全性架構](/azure/iot-fundamentals/iot-security-architecture)。

本文章討論下列主題：

* [保障安全從威脅模型開始](/azure/iot-fundamentals/iot-security-architecture#security-starts-with-a-threat-model)
* [IoT 中的安全性](/azure/iot-fundamentals/iot-security-architecture#security-in-iot)
* [為 Azure IoT 參考架構進行威脅模型化作業](/azure/iot-fundamentals/iot-security-architecture#threat-modeling-the-azure-iot-reference-architecture)

## <a name="security-from-the-ground-up"></a>從頭建立安全性
IoT 使得全球企業面臨獨特的安全性、隱私權及相容性挑戰。 不同於傳統網路技術 (這類問題是以軟體及其實作方式為中心)，IoT 在意的是當網路與實體世界交會時會發生什麼事。 保護 IoT 解決方案要求確保安全佈建裝置，保護這些裝置與雲端之間的連接，以及在處理和儲存期間保護雲端中資料保護的安全。 但是，會針對這類功能運作的是資源受限的裝置、根據地理位置分佈的部署，以及解決方案中的許多裝置。

您可以藉由閱讀 [徹底保護物聯網安全性](/azure/iot-fundamentals/iot-security-ground-up)，了解如何處理這些區域中的安全性。

本文章討論下列主題：

* [徹底保護基礎結構的安全](/azure/iot-fundamentals/iot-security-ground-up#secure-infrastructure-from-the-ground-up)
* [Microsoft Azure - 適用於貴公司的安全 IoT 基礎結構](/azure/iot-fundamentals/iot-security-ground-up#microsoft-azure---secure-iot-infrastructure-for-your-business)

## <a name="best-practices"></a>最佳做法
保護 IoT 基礎結構需要嚴格的深度安全性防禦策略。 從保護雲端資料的安全、保護資料在公用網際網路上傳輸時的資料完整性，到安全佈建裝置，在整體基礎結構的每一階層建置更強大的安全性保證。

您可以藉由閱讀 [物聯網安全性最佳作法](/azure/iot-fundamentals/iot-security-best-practices)，了解物聯網安全性最佳作法。

本文章討論下列主題：

* [IoT 硬體製造商/整合者](/azure/iot-fundamentals/iot-security-best-practices#iot-hardware-manufacturerintegrator)
* [IoT 解決方案開發人員](/azure/iot-fundamentals/iot-security-best-practices#iot-solution-developer)
* [IoT 解決方案部署人員](/azure/iot-fundamentals/iot-security-best-practices#iot-solution-deployer)
* [IoT 解決方案操作人員](/azure/iot-fundamentals/iot-security-best-practices#iot-solution-operator)
