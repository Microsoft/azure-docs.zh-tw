---
title: Azure 流量管理員中的計量與警示 | Microsoft Docs
description: 本文描述在 Azure 中的流量管理員可使用的計量。
services: traffic-manager
documentationcenter: ''
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/11/2018
ms.author: kumud
ms.openlocfilehash: fe4cc3c06af6868396f5db6fd88804022f5f9793
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432469"
---
# <a name="traffic-manager-metrics-and-alerts"></a>流量管理員的計量與警示

流量管理員提供您以 DNS 為基礎的負載平衡，其中包含多個路由方法和端點監視選項。 本文描述客戶可使用的計量與相關聯的警示。 

## <a name="metrics-available-in-traffic-manager"></a>在流量管理員中可用的計量 

流量管理員會依設定檔提供以下計量。客戶可以取用這些計量，進而了解其流量管理員使用情況，以及設定檔下端點的狀態。  

### <a name="queries-by-endpoint-returned"></a>傳回的依端點查詢數
使用[此計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md)來檢視流量管理員設定檔在指定期間內處理的查詢數目。 您也可以在端點的細微層級來檢視相同的資訊，這樣有助於了解端點在來自流量管理員的查詢回應中被傳回幾次。

在下方的範例中，圖 1 顯示流量管理員設定檔傳回的所有查詢回應。 

  
![流量管理員計量 - 所有查詢的彙總檢視](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-queries-aggregate-view.png)

圖 1：所有查詢的彙總檢視
  
圖 2 顯示相同的資訊，但是依端點分割。 因此，您可以查看傳回特定端點的查詢回應量。

![流量管理員計量 - 每個端點的查詢量分割檢視](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-query-volume-per-endpoint.png)

圖 2：顯示所傳回每個端點查詢量的分割檢視

## <a name="endpoint-status-by-endpoint"></a>依端點的端點狀態
使用[此計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftnetworktrafficmanagerprofiles)來了解設定檔中端點的健全狀態。 它採用兩個值：
 - 如果端點已啟動，使用 **1**。
 - 如果端點已關閉，使用 **0**。

此計量可以顯示為代表所有計量狀態的彙總值 (圖 3)，或者也可以分割 (參閱圖 4) 以顯示特定端點狀態。 在前者的情況下，如果彙總層級選取為 [平均]，則此計量值是所有端點狀態的算術平均值。 例如，如果設定檔有兩個端點，且只有一個健康情況良好，則此計量的值會是 **0.50**，如圖 3 中所示。 


![流量管理員計量 - 端點狀態的複合檢視](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-endpoint-status-composite-view.png)

圖 3：端點狀態計量的複合檢視


![流量管理員計量 - 端點狀態的分割檢視](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-endpoint-status-split-view.png)

圖 4：端點狀態計量的分割檢視

您可以透過 [Azure 監視器服務](../monitoring-and-diagnostics/monitoring-supported-metrics.md)的入口網站、[REST API](https://docs.microsoft.com/rest/api/monitor/)、[Azure CLI](https://docs.microsoft.com/cli/azure/monitor)、[Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.insights)，或流量管理員的入口網站體驗來取用這些計量。

## <a name="alerts-on-traffic-manager-metrics"></a>流量管理員計量的警示
除了處理及顯示來自流量管理員的計量，Azure 監視器也能讓客戶設定和接收與計量相關聯的警示。 您可以選擇這些計量中需要滿足什麼條件才會出現警示、監視這些條件的頻率為何，以及傳送警示給您的方式。 如需詳細資訊，請參閱 [Azure 監視器警示文件](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)。

## <a name="next-steps"></a>後續步驟
- 深入了解 [Azure 監視器服務](../monitoring-and-diagnostics/monitoring-supported-metrics.md)
- 深入了解如何[使用 Azure 監視器建立新的圖表](../monitoring-and-diagnostics/monitoring-metric-charts.md#how-do-i-create-a-new-chart)
