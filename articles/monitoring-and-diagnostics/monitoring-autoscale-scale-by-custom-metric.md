---
title: 使用自訂計量在 Azure 中自動調整規模
description: 了解如何在 Azure 中依自訂計量調整資源的規模。
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/07/2017
ms.author: ancav
ms.component: autoscale
ms.openlocfilehash: 9df587d92b9e35db496c787186ff2945db7965ce
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987810"
---
# <a name="get-started-with-auto-scale-by-custom-metric-in-azure"></a>開始在 Azure 中依自訂計量自動調整規模
本文說明如何在 Azure 入口網站中依自訂計量調整您資源的規模。

Azure 監視器自動調整僅適用於[虛擬機器擴展集](https://azure.microsoft.com/services/virtual-machine-scale-sets/)、[雲端服務](https://azure.microsoft.com/services/cloud-services/)、[App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/)和 [API 管理服務](https://docs.microsoft.com/azure/api-management/api-management-key-concepts)。

# <a name="lets-get-started"></a>開始使用
本文假設您的 Web 應用程式已設定 Application Insights。 如果您還沒有，則可[設定 ASP.NET 網站的 Application Insights][1]

- 開啟 [Azure 入口網站][2]
- 按一下左側導覽窗格中的 [Azure 監視器] 圖示。
  ![啟動 Azure 監視器][3]
- 按一下 [自動調整規模] 設定，以檢視適用於自動調整規模的所有資源及其目前的自動調整規模狀態![探索 Azure 監視器中的自動調整規模][4]
- 開啟 Azure 監視器中的 [自動調整規模] 刀鋒視窗，然後選取您想要調整的資源
> 注意︰下列步驟會使用與已設定 App Insights 之 Web 應用程式相關聯的 App Service 方案。
- 在資源的 [調整規模設定] 刀鋒視窗中，注意目前的執行個體計數為 1。 按一下 [啟用自動調整規模]。
  ![適用於新 Web 應用程式的調整規模設定][5]
- 提供調整規模設定的名稱，然後按一下 [新增規則]。 請注意，調整規模規則選項會在右邊開啟為內容窗格。 預設會將選項設定為如果資源的 CPU 百分比超過 70%，就會將您的執行個體計數相應增加 1。 變更 [Application Insights] 頂端的計量來源、選取 [資源] 下拉式清單中的 App Insights 資源，然後根據您想要調整規模的項目來選取自訂計量。
  ![依自訂計量調整規模][6]
- 與上述步驟類似，新增一個將會相應縮小的調整規模規則，如果自訂計量低於臨界值，即會將調整規模計數減 1。
  ![根據 CPU 調整規模][7]
- 設定您的執行個體限制。 例如，如果您想要根據自訂計量波動，在 2-5 個執行個體之間調整規模，可將 [最小值] 設為 2、將 [最大值] 設為 5，以及將 [預設值] 設為 2
> 注意︰萬一在讀取資源計量時發生問題，而且目前的容量低於預設容量，則為了確保資源的可用性，自動調整規模將會相應放大為預設值。 如果目前的容量已超過預設容量，自動調整規模將不會進行相應縮小。
- 按一下 [儲存]

恭喜！ 您現在已成功建立調整規模設定，可根據自訂計量自動調整 Web 應用程式的規模。

> 注意︰相同的步驟都適用於開始使用 VMSS 或雲端服務角色。

<!--Reference-->
[1]: https://docs.microsoft.com/azure/application-insights/app-insights-asp-net
[2]: https://portal.azure.com
[3]: ./media/monitoring-autoscale-scale-by-custom-metric/azure-monitor-launch.png
[4]: ./media/monitoring-autoscale-scale-by-custom-metric/discover-autoscale-azure-monitor.png
[5]: ./media/monitoring-autoscale-scale-by-custom-metric/scale-setting-new-web-app.png
[6]: ./media/monitoring-autoscale-scale-by-custom-metric/scale-by-custom-metric.png
[7]: ./media/monitoring-autoscale-scale-by-custom-metric/autoscale-setting-custom-metrics-ai.png
