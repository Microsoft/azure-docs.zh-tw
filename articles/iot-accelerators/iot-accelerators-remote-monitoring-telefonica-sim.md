---
title: 在遠端監視解決方案中整合 SIM 資料 - Azure | Microsoft Docs
description: 本文描述如何將 Telefónica SIM 資料整合到遠端監視解決方案。
author: hegate
manager: ''
ms.author: hegate
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 05/15/2018
ms.topic: conceptual
ms.openlocfilehash: c453998eea2a747b2cb608482f0ef9c1ee197ee0
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39185426"
---
# <a name="integrate-sim-data-in-the-remote-monitoring-solution"></a>在遠端監視解決方案中整合 SIM 資料

IoT 裝置通常會使用 SIM 卡連線到雲端，讓裝置能從任何地方傳送資料流。 Azure IoT 遠端監視解決方案能夠整合 IoT 管理的連線資料，讓操作員也可透過 IoT SIM 提供的資料追蹤裝置的健康情況。

遠端監視提供與 Telefónica IoT 連線的既有整合，讓客戶能夠使用它的 IoT 連線平台，將其裝置 SIM 連線資料同步處理到解決方案。 這個解決方案可透過 GitHub [存放庫](http://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet)進行擴充，以支援其他 IoT 連線。

在本教學課程中，您了解如何：

* 將 Telefónica IoT SIM 資料整合到遠端監視解決方案
* 檢視即時遙測
* 檢視 SIM 資料

## <a name="telefnica-iot-integration-setup"></a>Telefónica IoT 整合設定

### <a name="prerequisites"></a>必要條件

這項額外的遠端監視功能目前為預覽狀態。 若要將連線資料同步處理到 Azure 遠端監視解決方案，請遵循下列步驟：

1. 在 [Telefónica 的網站](https://iot.telefonica.com/contact)上填寫要求，然後選取 [Azure 遠端監視]，其中包括您的連絡資料。
2. Telefónica 會啟動您的帳戶。
3. 如果您還不是 Telefónica 用戶端，但是您想要享有這項或其他 IoT 連線雲端就緒服務，請造訪 [Telefónica 的網站](https://iot.telefonica.com/)，並選取 [連線] 選項。

### <a name="telefnica-sim-setup"></a>Telefónica SIM 設定
Telefónica SIM 和 Azure Twin 裝置識別碼關聯是以 Telefónica IoT SIM「別名」屬性為依據。 

瀏覽至[Telefónica IoT 連線平台入口網站](https://m2m-movistar-es.telefonica.com/) > [SIM 清查] > 選取 SIM，然後使用您所需的 Twin deviceID 更新每個 SIM「別名」。 也能夠以大量模式完成這項工作 (請參閱 Telefónica IoT 連線平台使用者手冊)。

也能夠以大量模式完成這項工作 (請參閱 Telefónica IoT 連線平台使用者手冊)

![Telefónica 更新](./media/iot-accelerators-remote-monitoring-telefonica-sim/telefonica_site.png)

若要將裝置連線到遠端監視，您可以使用 [C](iot-accelerators-connecting-devices-linux.md) 或 [Node](iot-accelerators-connecting-devices-node.md) 來遵循這些教學課程。 

## <a name="view-device-telemetry-and-sim-properties"></a>檢視裝置遙測和 SIM 屬性

一旦正確設定 Telefónica 帳戶且將裝置連線之後，您就能檢視裝置詳細資料和 SIM 資料。

您會發佈下列連線參數：

* ICCID
* IP
* 網路目前狀態
* SIM 狀態
* 網路位置
* 已取用的資料流量

![儀表板](./media/iot-accelerators-remote-monitoring-telefonica-sim/dashboard.png)

## <a name="next-steps"></a>後續步驟

現在您已概略了解如何將 SIM 資料整合到 Azure IoT 遠端監視，以下是對於解決方案加速器所建議的後續步驟：

* [操作 Azure IoT 遠端監視解決方案](quickstart-remote-monitoring-deploy.md)
* [執行進階監視](iot-accelerators-remote-monitoring-monitor.md)
* [管理您的裝置](iot-accelerators-remote-monitoring-manage.md)
* [針對裝置問題進行疑難排解](iot-accelerators-remote-monitoring-maintain.md)

