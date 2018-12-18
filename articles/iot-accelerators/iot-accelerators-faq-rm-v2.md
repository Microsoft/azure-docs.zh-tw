---
title: 遠端監視解決方案加速器的常見問題集 | Microsoft Docs
description: 遠端監視解決方案加速器的常見問題集
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 02/15/2018
ms.author: dobett
ms.openlocfilehash: ba0d81761904be74632f8f0025488b7f501bd715
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39185994"
---
# <a name="frequently-asked-questions-for-remote-monitoring-solution-accelerator"></a>遠端監視解決方案加速器的常見問題集

另請參閱，一般[常見問題集](iot-accelerators-faq.md)。

### <a name="how-much-does-it-cost-to-provision-the-new-remote-monitoring-solution"></a>佈建新的遠端監視解決方案需要多少成本？

新的解決方案加速器提供兩種部署選項：

* 基本選項，是針對尋求降低開發成本的開發人員，或需要建置示範或概念證明的客戶所設計。
* 標準選項，是針對需要部署備妥基礎結構的企業而設計。

### <a name="how-can-i-ensure-i-keep-my-costs-down-while-i-develop-my-solution"></a>如何確定我在開發解決方案的同時保持降低成本？

除了提供兩種不同的部署之外，新的遠端監視解決方案還有一項設定，可隨選啟用或停用所有模擬的裝置。 停用模擬會減少解決方案中內嵌的資料，並因而降低整體成本。

### <a name="what-is-the-difference-between-the-basic-and-standard-deployment-options-how-do-i-decide-between-the-two-deployment-options"></a>基本和標準部署選項之間的差異為何？ 如何在兩個部署選項之間做決定？

每個部署選項都會對應到不同的需求。 基本部署是針對入門及開發 PoC 及小型試驗而設計。 它可以最低需求的資源和較低的成本來提供簡化的架構。 標準部署是針對建置和自訂備妥解決方案而設計，且能提供具有實現所需必要元素的部署。 對於可靠性與調整，應用程式微服務會建置作為 Docker 容器，並使用協調器 (依預設為 Kubernetes) 加以部署。 協調器會負責部署、調整及管理應用程式。 您應該以目前的需求作為基礎選擇一個選項。 您可以根據您的專案階段，使用其中一個、另一個或兩者的組合。

### <a name="how-do-i-configure-a-dynamic-map-on-the-dashboard"></a>如何在儀表板上設定動態地圖？

如需詳細資訊，請參閱[升級地圖索引鍵以在動態地圖上顯示裝置](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide#upgrade-map-key-to-see-devices-on-a-dynamic-map) \(英文\)。

### <a name="next-steps"></a>後續步驟

您也可以探索 IoT 解決方案加速器的一些其他特性與功能：

* [探索遠端監視解決方案加速器的功能](quickstart-remote-monitoring-deploy.md)
* [預測性維護解決方案加速器概觀](iot-accelerators-predictive-overview.md)
* [部署連線的處理站解決方案加速器](quickstart-connected-factory-deploy.md) \(英文\)
* [從頭建立 IoT 安全性](/azure/iot-fundamentals/iot-security-ground-up)
