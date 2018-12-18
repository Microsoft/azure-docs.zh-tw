---
title: Azure IoT 中樞移轉至診斷設定 | Microsoft Docs
description: 如何將「Azure IoT 中樞」更新成使用 Azure 診斷設定 (取代作業監視) 來即時監視 IoT 中樞上作業的狀態。
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/10/2017
ms.author: kgremban
ms.openlocfilehash: 85bdb4b4802283c591e4d7a9e8b14ae74fa44e8d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38666717"
---
# <a name="migrate-your-iot-hub-from-operations-monitoring-to-diagnostics-settings"></a>將 IoT 中樞從作業監視移轉至診斷設定

客戶如果使用[作業監視][lnk-opsmon]來追蹤「IoT 中樞」內作業的狀態，可以將該工作流程移轉至 [Azure 診斷設定][lnk-diagnostics-settings]，這是「Azure 監視器」的一項功能。 診斷設定可針對許多 Azure 服務提供資源層級的診斷資訊。

「IoT 中樞」的作業監視功能已被淘汰，未來將會移除。 本文提供將您工作負載從作業監視移至診斷設定的步驟。 如需有關淘汰時間表的詳細資訊，請參閱[使用 Azure 監視器和 Azure 資源健康狀態來監視您的 Azure IoT 解決方案][lnk-blog-announcement]。

## <a name="update-iot-hub"></a>更新 IoT 中樞

若要在 Azure 入口網站中更新您的「IoT 中樞」，請先開啟診斷設定，然後關閉作業監視。  

[!INCLUDE [iot-hub-diagnostics-settings](../../includes/iot-hub-diagnostics-settings.md)]

### <a name="turn-off-operations-monitoring"></a>關閉作業監視

在您已對工作流程測試新的診斷設定之後，便可以關閉作業監視功能。 

1. 在您的「IoT 中樞」功能表上，選取 [作業監視]。
1. 在每個監視類別底下，選取 [無]。
1. 儲存作業監視變更。

## <a name="update-applications-that-use-operations-monitoring"></a>更新使用作業監視的應用程式

作業監視與診斷設定的結構描述有些微差異。 請務必更新目前使用作業監視的應用程式，以對應診斷設定所使用的結構描述。 

此外，診斷設定還提供 5 個新類別的追蹤。 在您針對現有的結構描述更新應用程式之後，請一併新增新的類別：

- 雲端到裝置對應項作業
- 裝置到雲端對應項作業
- 對應項查詢
- 作業的操作
- 直接方法

針對特定的結構描述結構，請參閱[了解診斷設定的結構描述][lnk-diagnostics-schema]。

## <a name="next-steps"></a>後續步驟

- [監視 Azure IoT 中樞的健康情況並快速診斷問題][lnk-monitor]

[lnk-opsmon]: iot-hub-operations-monitoring.md
[lnk-diagnostics-settings]: ../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md
[lnk-diagnostics-schema]: iot-hub-monitor-resource-health.md#understand-the-logs
[lnk-blog-announcement]: https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health/
[lnk-monitor]: iot-hub-monitor-resource-health.md
