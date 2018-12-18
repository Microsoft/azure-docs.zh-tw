---
title: Azure Log Analytics 中的容量和效能解決方案 | Microsoft Docs
description: 使用 Log Analytics 中的容量和效能解決方案，來協助您了解 Hyper-V 伺服器的容量。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 51617a6f-ffdd-4ed2-8b74-1257149ce3d4
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/13/2017
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: 0244673d50e22124e8a0678e202004f06b0bca00
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182615"
---
# <a name="plan-hyper-v-virtual-machine-capacity-with-the-capacity-and-performance-solution-preview"></a>使用容量和效能解決方案來規劃 Hyper-V 虛擬機器容量 (預覽)

![容量與效能符號](./media/log-analytics-capacity/capacity-solution.png)

> [!NOTE]
> 容量與效能解決方案已取代。  已安裝此解決方案的客戶可以繼續使用，但無法將「容量與效能」加入任何新的工作區。

您可以使用 Log Analytics 中的容量和效能解決方案，來協助您了解 Hyper-V 伺服器的容量。 這個解決方案可顯示主機以及在這些 Hyper-V 主機上執行之 VM 的整體使用率 (CPU、記憶體和磁碟)，以讓您深入了解 Hyper-V 環境。 系統會跨所有主機和在其上執行的 VM 來收集 CPU、記憶體和磁碟的計量。

此解決方案：

-   顯示 CPU 和記憶體使用率最高與最低的主機
-   顯示 CPU 和記憶體使用率最高與最低的 VM
-   顯示 IOPS 和輸送量使用率最高與最低的 VM
-   顯示哪些 VM 在哪些主機上執行
-   顯示在叢集共用磁碟區內具有高輸送量、IOPS 和延遲的前幾大磁碟
- 可讓您根據群組進行自訂和篩選

> [!NOTE]
> 舊版的容量和效能解決方案稱為「容量管理」，其同時需要 System Center Operations Manager 和 System Center Virtual Machine Manager。 這個經過更新的解決方案則沒有這些相依性。


## <a name="connected-sources"></a>連接的來源

下表描述此方案支援的連接來源。

| 連接的來源 | 支援 | 說明 |
|---|---|---|
| [Windows 代理程式](log-analytics-windows-agent.md) | 是 | 解決方案會從 Windows 代理程式收集容量和效能資料的資訊。 |
| [Linux 代理程式](log-analytics-linux-agents.md) | 否    | 解決方案不會從直接 Linux 代理程式收集容量和效能資料的資訊。|
| [SCOM 管理群組](log-analytics-om-agents.md) | 是 |解決方案會從已連線之 SCOM 管理群組中的代理程式收集容量和效能資料。 不需要從 SCOM 代理程式直接連接到 Log Analytics。|
| [Azure 儲存體帳戶](log-analytics-azure-storage.md) | 否 | Azure 儲存體不包含容量和效能資料。|

## <a name="prerequisites"></a>必要條件

- Windows 或 Operations Manager 代理程式必須安裝在 Windows Server 2012 或更新版本的 Hyper-V 主機上，而非安裝在虛擬機器上。


## <a name="configuration"></a>組態

執行下列步驟以將容量和效能解決方案新增至您的工作區。

- 使用[從方案庫加入 Log Analytics 方案](log-analytics-add-solutions.md)中所述的程序，將「容量與效能」解決方案新增至您的 Log Analytics 工作區。

## <a name="management-packs"></a>管理組件

如果 SCOM 管理群組已連線到 Log Analytics 工作區，當您加入此方案時，下列管理組件會安裝在 SCOM 中。 這些管理組件不需要任何設定或維護。

- Microsoft.IntelligencePacks.CapacityPerformance

1201 事件類似︰


```
New Management Pack with id:"Microsoft.IntelligencePacks.CapacityPerformance", version:"1.10.3190.0" received.
```

當容量和效能解決方案更新時，其版本號碼也會變更。

如需有關方案管理組件如何更新的詳細資訊，請參閱 [將 Operations Manager 連接到 Log Analytics](log-analytics-om-agents.md)。

## <a name="using-the-solution"></a>使用解決方案

當您將容量和效能解決方案新增至工作區時，[概觀] 儀表板中會新增 [容量和效能]。 此圖格會顯示目前作用中的 Hyper-V 主機數目，以及所選時段內受監視的作用中虛擬機器數目。

![[容量和效能] 圖格](./media/log-analytics-capacity/capacity-tile.png)


### <a name="review-utilization"></a>檢閱使用率

按一下 [容量和效能] 圖格，即可開啟 [容量和效能] 儀表板。 此儀表板包含下表中的資料行。 每個資料行最多會列出十個符合該資料行中指定範圍和時間範圍之準則的項目。 您可以按一下資料行底部的 [查看全部]，或按一下資料行標頭，以執行記錄檔搜尋來傳回所有記錄。

- **主機**
    - **主機 CPU 使用率** 根據所選時段以圖表顯示主機電腦的 CPU 使用率趨勢和主機清單。 將滑鼠暫留在折線圖上，可檢視特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何主機名稱可開啟記錄搜尋並檢視所裝載 VM 的 CPU 計數器詳細資料。
    - **主機記憶體使用率** 根據所選時段以圖表顯示主機電腦的記憶體使用率趨勢和主機清單。 將滑鼠暫留在折線圖上，可檢視特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何主機名稱可開啟記錄搜尋並檢視所裝載 VM 的記憶體計數器詳細資料。
- **虛擬機器**
    - **VM CPU 使用率** 根據所選時段以圖表顯示虛擬機器的 CPU 使用率趨勢和虛擬機器清單。 將滑鼠暫留在折線圖上，可檢視前 3 個 VM 在特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何 VM 名稱可開啟記錄搜尋並檢視 VM 的彙總 CPU 計數器詳細資料。
    - **VM 記憶體使用率**以所選時段作為基礎，用圖表顯示虛擬機器的記憶體使用率趨勢和虛擬機器清單。 將滑鼠暫留在折線圖上，可檢視前 3 個 VM 在特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何 VM 名稱可開啟記錄搜尋並檢視 VM 的彙總記憶體計數器詳細資料。
    - **VM 磁碟 IOPS 總數** 根據所選時段以圖表顯示虛擬機器的磁碟 IOPS 總數趨勢和虛擬機器清單以及各自的 IOPS。 將滑鼠暫留在折線圖上，可檢視前 3 個 VM 在特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何 VM 名稱可開啟記錄搜尋並檢視 VM 的彙總磁碟 IOPS 計數器詳細資料。
    - **VM 磁碟輸送量總數** 根據所選時段以圖表顯示虛擬機器的磁碟輸送量總數趨勢和虛擬機器清單以及各自的磁碟輸送量總數。 將滑鼠暫留在折線圖上，可檢視前 3 個 VM 在特定時間點的詳細資料。 按一下折線圖可在記錄搜尋中檢視更多詳細資料。 按一下任何 VM 名稱可開啟記錄搜尋並檢視 VM 的彙總磁碟輸送量總數計數器詳細資料。
- **叢集共用磁碟區**
    - **輸送量總數** 顯示叢集共用磁碟區上的讀取和寫入數總和。
    - **IOPS 總數** 顯示叢集共用磁碟區上每秒輸入/輸出作業的總和。
    - **延遲總數** 顯示叢集共用磁碟區上的延遲總數。
- **主機密度** 頂端的圖格會顯示解決方案能夠使用的主機和虛擬機器總數。 按一下頂端的圖格，即可在記錄搜尋中檢視其他詳細資料。 其中也會列出所有主機和其所裝載的虛擬機器數目。 按一下主機即可在記錄搜尋中向下鑽研至 VM 結果。


![儀表板主機刀鋒視窗](./media/log-analytics-capacity/dashboard-hosts.png)

![儀表板虛擬機器刀鋒視窗](./media/log-analytics-capacity/dashboard-vms.png)


### <a name="evaluate-performance"></a>評估效能

每個組織的生產運算環境會有明顯不同。 此外，容量和效能的工作負載可能會取決於 VM 的執行方式以及您認為平常的事項。 可協助您測量效能的特定程序很可能不適用於您的環境。 因此，更為一般性的規範指引較適合提供協助。 Microsoft 發佈了各種規範指引文章，以協助您測量效能。

總結來說，解決方案會從各種來源 (包括效能計數器) 收集容量和效能資料。 請使用解決方案中的各種介面所呈現的容量和效能資料，並將您的結果與[測量 Hyper-V 上的效能](https://msdn.microsoft.com/library/cc768535.aspx)文章中的結果做比較。 雖然該文章已發佈一段時間，但其計量、考量和指導方針仍然有效。 該文章包含其他有用資源的連結。


## <a name="sample-log-searches"></a>記錄搜尋範例

下表提供此解決方案所收集和計算之容量和效能資料的記錄搜尋範例。


| 查詢 | 說明 |
|:--- |:--- |
| 所有主機記憶體組態 | Perf &#124; where ObjectName == "Capacity and Performance" and CounterName == "Host Assigned Memory MB" &#124; summarize MB = avg(CounterValue) by InstanceName |
| 所有 VM 記憶體組態 | Perf &#124; where ObjectName == "Capacity and Performance" and CounterName == "VM Assigned Memory MB" &#124; summarize MB = avg(CounterValue) by InstanceName |
| 所有 VM 的磁碟 IOPS 總數細目 | Perf &#124; where ObjectName == "Capacity and Performance" and (CounterName == "VHD Reads/s" or CounterName == "VHD Writes/s") &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), CounterName, InstanceName |
| 所有 VM 的磁碟輸送量總數細目 | Perf &#124; where ObjectName == "Capacity and Performance" and (CounterName == "VHD Read MB/s" or CounterName == "VHD Write MB/s") &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), CounterName, InstanceName |
| 所有 CSV 的 IOPS 總數細目 | Perf &#124; where ObjectName == "Capacity and Performance" and (CounterName == "CSV Reads/s" or CounterName == "CSV Writes/s") &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), CounterName, InstanceName |
| 所有 CSV 的輸送量總數細目 | Perf &#124; where ObjectName == "Capacity and Performance" and (CounterName == "CSV Reads/s" or CounterName == "CSV Writes/s") &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), CounterName, InstanceName |
| 所有 CSV 的延遲總數細目 | Perf &#124; where ObjectName == "Capacity and Performance" and (CounterName == "CSV Read Latency" or CounterName == "CSV Write Latency") &#124; summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1h), CounterName, InstanceName |


## <a name="next-steps"></a>後續步驟
* 使用 [Log Analytics 中的記錄搜尋](log-analytics-log-search.md)，來檢視詳細的容量和效能資料。
