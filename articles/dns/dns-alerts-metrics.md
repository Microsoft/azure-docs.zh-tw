---
title: Azure DNS 計量與警示 | Microsoft Docs
description: 深入了解 Azure DNS 計量與警示。
services: dns
documentationcenter: na
author: vhorne
manager: jennoc
editor: ''
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/17/2018
ms.author: victorh
ms.openlocfilehash: de29c24556522abeaff8d942edc027c7444c3ed3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965007"
---
# <a name="azure-dns-metrics-and-alerts"></a>Azure DNS 計量與警示
Azure DNS 是 DNS 網域的主機服務，採用 Microsoft Azure 基礎結構來提供名稱解析。 本文說明 Azure DNS 服務的計量與警示。

## <a name="azure-dns-metrics"></a>Azure DNS 計量

Azure DNS 為客戶提供了計量，讓客戶監視其在服務中所託管 DNS 的特定層面。 此外，使用 Azure DNS 計量，您可以設定依據感興趣的條件來接收警示。 計量是透過 [Azure 監視器服務](../azure-monitor/index.yml)提供的。 Azure DNS 透過 Azure 監視器為您的 DNS 區域提供下列計量：

-   QueryVolume
-   RecordSetCount
-   RecordSetCapacityUtilization

您也可以在 Azure 監視器文件頁面上查看[這些計量的定義](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftnetworkdnszones)。
>[!NOTE]
> 目前，這些計量只適用於在 Azure DNS 中託管的公用 DNS 區域。 如果您在 Azure DNS 中有託管的私人區域，這些計量將不會提供這些區域的資料。 此外，計量與警示功能只有在 Azure 公用雲端中才受支援。 之後將提供主權雲端的支援。 

這些計量細微的維度層級是 DNS 區域。

### <a name="query-volume"></a>查詢磁碟區

Azure DNS 中的「查詢磁碟區」計量會顯示 Azure DNS 收到 DNS 區域的 DNS 查詢 (查詢流量) 磁碟區。 度量單位是計數，彙總則是一段時間收到的所有查詢的總計。 若要檢視此計量，請從 Azure 入口網站中的 [監視器] 索引標籤選取 [計量] (預覽) 瀏覽器體驗。 從 [資源] 下拉式清單中選取您的 DNS 區域，選取 [查詢磁碟區] 計量，然後選取 [加總為彙總]。 其範例如下列螢幕擷取畫面所示。  如需計量瀏覽器體驗和圖表的詳細資訊，請參閱 [Azure 監視器計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)。

![查詢磁碟區](./media/dns-alerts-metrics/dns-metrics-query-volume.png)

*圖：Azure DNS 查詢磁碟區計量*

### <a name="record-set-count"></a>記錄集計數
「記錄集計數」計量顯示 Azure DNS 中 DNS 區域的記錄集數目。 您的區域中定義的所有記錄集都會計算在內。 度量單位是計數，彙總則是所有記錄集的最大值。 若要檢視此計量，請從 Azure 入口網站中的 [監視器] 索引標籤選取 [計量] (預覽) 瀏覽器體驗。 從 [資源] 下拉式清單中選取您的 DNS 區域，選取 [記錄集計數] 計量，然後選取 [最大值] 作為 [彙總]。 如需計量瀏覽器體驗和圖表的詳細資訊，請參閱 [Azure 監視器計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)。 

![記錄集計數](./media/dns-alerts-metrics/dns-metrics-record-set-count.png)

*圖：Azure DNS 記錄集計數計量*


### <a name="record-set-capacity-utilization"></a>記錄集容量使用率
Azure DNS 中的「記錄集容量使用率」顯示 DNS 區域的記錄集容量使用率百分比。 Azure DNS 中的每個 DNS 區域有一個記錄集限制，定義區域允許的記錄集最大數目 (請參閱 [DNS 限制](dns-zones-records.md#limits))。 因此，此計量會顯示接近記錄集限制的程度。 例如，如果您的 DNS 區域設定了 500 個記錄集，而區域的預設記錄集限制為 5000，RecordSetCapacityUtilization 計量將顯示 10% 的值 (以 500 除以 5000 所得)。 度量單位是「百分比」，「彙總」類型是「最大值」。 若要檢視此計量，請從 Azure 入口網站中的 [監視器] 索引標籤選取 [計量] (預覽) 瀏覽器體驗。 從 [資源] 下拉式清單中選取您的 DNS 區域，選取 [記錄集容量使用率] 計量，然後選取 [最大值] 作為 [彙總]。 其範例如下列螢幕擷取畫面所示。 如需計量瀏覽器體驗和圖表的詳細資訊，請參閱 [Azure 監視器計量瀏覽器](../monitoring-and-diagnostics/monitoring-metric-charts.md)。 

![記錄集計數](./media/dns-alerts-metrics/dns-metrics-record-set-capacity-uitlization.png)

*圖：Azure DNS 記錄集容量使用率計量*

## <a name="alerts-in-azure-dns"></a>Azure DNS 中的警示
Azure 監視器提供針對可用的計量值發出警示的功能。 DNS 計量可用於新的警示設定體驗。 如 [Azure 監視器警示文件](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)所述，您可以選取 DNS 區域作為資源、選擇計量訊號類型，並設定警示邏輯和其他參數，例如「期間」和「頻率」。 您可進一步定義符合警示條件時的[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，以透過選擇的動作傳遞警示。 如需如何設定 Azure 監視器計量警示的詳細資訊，請參閱[使用 Azure 監視器建立、檢視及管理警示](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)。 

## <a name="next-steps"></a>後續步驟
- 深入了解 [ Azure DNS](dns-overview.md)。
