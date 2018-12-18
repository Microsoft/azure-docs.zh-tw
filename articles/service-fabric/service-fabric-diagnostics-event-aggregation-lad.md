---
title: 使用 Linux Azure 診斷的 Azure Service Fabric 事件彙總 | Microsoft Docs
description: 了解如何使用 LAD 彙總及收集事件，來監視和診斷 Azure Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: dekapur
ms.openlocfilehash: c7eb98eb2dbff05e67b6a60c413932ba51fdfdf7
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45573751"
---
# <a name="event-aggregation-and-collection-using-linux-azure-diagnostics"></a>使用 Linux Azure 診斷的事件彙總和收集
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-event-aggregation-wad.md)
> * [Linux](service-fabric-diagnostics-event-aggregation-lad.md)
>
>

當您執行 Azure Service Fabric 叢集時，最好從中央位置的所有節點收集記錄檔。 將記錄檔集中在中央位置，可協助您分析並針對叢集或該叢集中執行之應用程式與服務的問題進行疑難排解。

上傳和收集記錄的其中一種方式就是使用 Linux Azure 診斷 (LAD) 延伸模組，此延伸模組可將記錄上傳到 Azure 儲存體，也可以選擇將記錄傳送至 Azure Application Insights 或事件中樞。 您也可以使用外部程序來讀取儲存體中的事件，然後將它們放在 [Log Analytics](../log-analytics/log-analytics-service-fabric.md) 這類的分析平台產品或其他記錄剖析解決方案中。

## <a name="log-and-event-sources"></a>記錄和事件來源

### <a name="service-fabric-platform-events"></a>Service Fabric 平台事件
Service Fabric 會透過 [LTTng](http://lttng.org) 發出少數的現成記錄，包括運作事件或執行階段事件。 這些記錄檔會儲存在叢集的 Resource Manager 範本所指定的位置。 若要取得或設定儲存體帳戶的詳細資訊，請搜尋 **AzureTableWinFabETWQueryable** 標籤並尋找 **StoreConnectionString**。

### <a name="application-events"></a>應用程式事件
 檢測軟體時，會如您在應用程式和服務的程式碼中所指定的，發出事件。 您可以使用任何撰寫文字型記錄檔的記錄解決方案，例如 LTTng。 如需詳細資訊，請參閱 LTTng 文件中關於追蹤您應用程式的內容。

[監視和診斷本機開發設定中的服務](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)。

## <a name="deploy-the-diagnostics-extension"></a>部署診斷擴充功能
收集記錄檔的第一個步驟是將診斷擴充功能部署在 Service Fabric 叢集的每個 WM 上。 診斷擴充功能會收集每個 VM 上的記錄檔，並將它們上傳至您指定的儲存體帳戶。 

若要在建立叢集過程中將診斷擴充功能部署至叢集中的 VM，請將 [診斷] 設為 [開啟]。 建立叢集之後，您無法使用入口網站變更這項設定，因此，您將必須在 Resource Manager 範本中進行適當的變更。

這將會設定 LAD 代理程式來監視指定的記錄檔。 每當新的一行附加至檔案時，系統就會建立 syslog 項目並傳送至您所指定的儲存體 (表格)。


## <a name="next-steps"></a>後續步驟

1. 若要更仔細了解您在進行問題的疑難排解時應該調查哪些事件，請參閱 [LTTng 文件](http://lttng.org/docs)和[使用 LAD](https://docs.microsoft.com/azure/virtual-machines/extensions/diagnostics-linux)。
2. [設定 Log Analytics 代理程式](service-fabric-diagnostics-event-analysis-oms.md)以幫助收集度量、監視部署在您的叢集上的容器，以及視覺化您的記錄檔 
